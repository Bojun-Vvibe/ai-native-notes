# Schema Drift in Tool Definitions Across Model Upgrades

Every team that has been running an agent in production for more than a quarter has a story like this: a new model version drops, marketing suggests it is across-the-board better, you flip the version string in your config, and within an hour your tool calls start failing in shapes you have never seen before. The error rate is not catastrophic — maybe two or three percent — and the failures are concentrated in a handful of specific tools. By the end of the day you have rolled back. By the end of the week you have a postmortem that says, in essence, "the new model is stricter about JSON schemas and our tool definitions were sloppy in ways the old model tolerated."

This is schema drift, and it is one of the least-discussed and most reliably painful aspects of running a long-lived agent stack. The model providers do not advertise it because, from their point of view, getting stricter about input validation is unambiguously a quality improvement. Your tool definitions are not getting worse — they were always wrong, the previous model was just generous about it. The new model is correct and your code is broken.

I want to lay out, in some detail, what schema drift actually looks like in practice, why it happens at the boundaries it happens at, and what disciplines around tool definition will make you robust to it without forcing you to pin model versions forever.

## What "the schema" actually is

The first piece of useful framing is that a tool definition is not really one schema — it is a stack of schemas, each of which can change independently across model versions, and each of which has its own enforcement story.

There is the schema you wrote, which is a JSON Schema document attached to the tool's parameters field. There is the schema the provider's serving infrastructure parses out of your request, which may not be exactly what you wrote because the provider may strip unsupported keywords, normalize types, or rewrite enums into something the model can consume. There is the schema the model was trained to honor, which is approximately but not exactly the schema the provider parsed — the training set probably used a slightly different representation than the production serving path. There is the schema the constrained-decoding layer enforces, if there is one — some providers run a token-level constrained-decoding pass that rejects any output that does not match the schema, and the rules of that constrainer may differ from the rules of JSON Schema itself.

When you upgrade the model, several of these can shift simultaneously. The provider may have changed the parser to be stricter about type coercion. The model may have been trained with a different schema flavor — maybe the new generation uses a different set of supported keywords. The constrained-decoding layer may have been updated to reject patterns the previous one accepted. Any of these can break tool calls that were working yesterday, and the error messages you get back almost never identify which layer is responsible.

## The categories of drift I have actually hit

After tracking these for a while, the drifts I have personally observed cluster into about six categories. They are worth knowing because the mitigation differs per category.

**Type coercion strictness.** The old model, given a parameter typed as `integer`, would happily emit `"42"` as a string and the provider would silently coerce it. The new model emits a JSON validation error and the tool call fails with no usable arguments. The fix is either to relax your schema to accept both string and integer, or to make sure your prompt is unambiguous about the type, or to add a coercion pass on your side before validating.

**Enum case sensitivity.** Tool has an enum parameter `status` with values `["open", "closed"]`. The old model sometimes emits `"Open"` and you got it through. The new model emits `"Open"` and the constrained decoder rejects it, retrying until it gets `"open"`, which adds latency and sometimes never converges. The fix is to either normalize enum values on input or to actually use exact-case enums in the schema and rely on the constrainer.

**Required field interpretation.** This one is particularly nasty. JSON Schema's `required` field lists which properties must be present. The old model treated this as a strong hint. The new model treats it as a strict requirement. If your schema marks a field as required but your prompt's example doesn't really need it, you will start getting empty or hallucinated values for that field where you previously got nothing. The fix is to be more careful about what you mark as required — if the parameter is genuinely optional, do not list it.

**Nested object support.** Some provider schemas support arbitrary nesting; some flatten beyond a certain depth. New model versions sometimes change the maximum supported depth, or change how they handle `$ref` (most production providers do not support $ref at all but are inconsistent about whether they reject schemas containing it or silently inline). I have had tools that worked fine for months stop working because the provider's schema preprocessor was updated and started rejecting a `$ref` we had been using to share a sub-schema across three tool definitions.

**Additional properties handling.** The default behavior for `additionalProperties` in JSON Schema is `true` — extra properties are allowed. Some providers default this to `false` for tool schemas, some don't, and the default has changed between model versions on at least one provider I know of. If you are passing through a schema that expected the permissive default, the new strict default may cause the model to refuse to add properties it would previously have happily emitted, leading to tool calls that lack fields you needed.

**Array constraints.** `minItems`, `maxItems`, `uniqueItems` — older models often ignored these; newer models often enforce them via constrained decoding, which means a request that should produce three items might hang trying to produce a fourth because your schema has an off-by-one in `maxItems`. I have personally shipped a `maxItems: 1` constraint into a tool that wanted "one or more items" and watched the model spend twenty seconds trying to satisfy a schema that did not match the prompt's intent.

## Why the new model is "stricter"

It is worth taking a moment to understand why each new generation of models tends to be stricter about tool schemas, because once you understand it, you stop being surprised when it happens.

The driver is constrained decoding. Several years ago tool calling worked entirely through prompting — the model was shown the schema, asked to produce JSON matching it, and you crossed your fingers. Validation was best-effort and the model frequently produced near-misses that were technically wrong but parseable.

Modern serving stacks layer constrained decoding on top: at each token, the decoder consults the schema to figure out which tokens would still allow a valid completion, and masks the logits to only allow those. This makes the output guaranteed-valid against the schema — but only against whatever schema flavor the constrainer implements, which may have edge cases that diverge from full JSON Schema, and which may change between model versions because the constrainer is part of the serving stack and gets updated.

The trend is clear: more validation moves from "model best effort" to "decoder hard enforcement," and the surface area for schema-related failures grows with each generation. This is fundamentally good — it means tool calls fail loudly instead of producing garbage — but it also means schema bugs that were latent for a year suddenly become production incidents on upgrade day.

## Disciplines that survive upgrades

The straightforward and somewhat boring answer is: write your schemas as if the constrainer is real and strict, even when you are testing against models where it is permissive. Specifically:

Use the most specific type you genuinely accept, and no more. If a parameter is always going to be parsed as an integer, type it as an integer, do not type it as `string` "for safety." The "safety" is illusory — strings have a much larger valid space and the model will surprise you with the contents.

Mark fields required only when you actually require them. The old habit of marking everything required because "it makes the model fill in the field" backfires under strict enforcement: the model will hallucinate values to satisfy the schema rather than acknowledging it doesn't have the information.

Avoid `oneOf`, `anyOf`, and `$ref` unless you have specifically tested that your provider supports them correctly. These are the most common causes of provider-specific preprocessing failures, and the failures are the kind where the schema appears to be accepted but the model behavior is subtly wrong.

Pin enum values to lowercase or to a single canonical case, and validate this in your CI. Mixed-case enums are a permanent source of pain that the constrainer will eventually start enforcing.

Set `additionalProperties: false` explicitly on every object schema, even if you think the provider defaults to false. Make the default explicit so the schema's behavior does not depend on the provider's default.

Be paranoid about array constraints. `minItems` and `maxItems` should reflect actual product requirements, not "I think one is reasonable" — the constrainer will enforce them and the model will spin trying to satisfy them.

## Detect drift before users do

The single most useful piece of infrastructure for surviving model upgrades is a tool-call replay test that exercises every tool definition against the candidate model before you flip the version flag in production. This sounds obvious but I have rarely seen it done well.

The shape that works for me is: maintain a fixture of "synthetic conversations" — short two-or-three-turn dialogs that are designed to deterministically trigger each tool call. For each tool, you have at least three fixtures: one that should trigger the tool with all required fields, one that should trigger it with only required fields and no optional ones, and one that should trigger an edge-case combination. The fixtures live in your repo and are updated whenever you add a tool.

When a new model version drops, you run every fixture against the new model and assert two things. First, that the tool was called at all. Second, that the arguments parse against your tool's expected types — not just against the JSON Schema you sent the model, but against your actual handler's expectations.

The second check is the important one. JSON Schema validation tells you the model produced something the schema accepts. It does not tell you whether the model's interpretation of an ambiguous parameter changed across versions. For instance, a parameter named `limit` with type integer is schema-valid whether the model passes 10 or 10000. If the new model has a tendency to be more aggressive with limits than the old one, your downstream system might suddenly start hammering things harder than before. Catching this requires asserting on the actual parsed values, not just on schema compliance.

The fixtures should be deterministic to the extent possible. Use temperature zero, fixed seeds where supported, and avoid open-ended phrasing in the user turn. A fixture like "find me three recent items" is bad — "three" is a hint, not a constraint, and the model may emit four or two on a different version. A fixture like "find me exactly three recent items" is better. A fixture that explicitly says "use the find_items tool with limit=3" is best, because then the only thing you are testing is whether the model can correctly fill in the arguments, not whether it can correctly choose the tool.

## The version-pin escape hatch

A reasonable response to all of this is "fine, just pin the model version forever." This works for a while but it is not a sustainable strategy because providers deprecate old versions, sometimes with as little as ninety days' notice. You will need to upgrade eventually, and "eventually" tends to coincide with the worst possible week.

The compromise I recommend is: pin the production model version, but always run an off-production replay suite against the latest available version, weekly. The suite consumes a small amount of money and produces a continuously-updated picture of what would break on the next upgrade. When the inevitable deprecation announcement arrives, you do not have to discover the schema drift under deadline pressure — you already know what is going to break, you have already filed the tickets, and the upgrade is mostly mechanical.

The replay suite should cover not just successful tool calls but also intentionally-malformed ones — schemas with deliberate violations, prompts that ask for impossible parameter combinations, edge cases involving very long strings or deeply-nested structures. These are the cases where the constrainer's edge-case handling matters most, and they are exactly the cases where new model versions tend to diverge.

## A note on documentation lag

One reliable failure mode is that the provider's documentation is several weeks behind the actual serving behavior. A new model ships and the docs say "supports JSON Schema draft-07 with these limitations" — and the limitations are accurate as of the previous model, but the new model has a slightly different list. You will not find the new limitations documented anywhere; you will find them by hitting them.

The practical consequence is that when you do hit a tool call failure on a fresh model, your first hypothesis should be "the schema-handling rules quietly changed," not "we have a bug in our code." Reproduce against the previous model first; if it works there and fails on the new one, you know it is drift, and your next move is to simplify the schema to figure out which keyword or pattern is the trigger. Almost every time I have done this exercise, the answer has been one of the categories above — type strictness, enum casing, required fields, nesting, additional properties, array constraints. There is no seventh category in my experience, though I am sure one is coming.

## Closing thought

The thing I want to leave you with is this: tool schemas are not "just configuration." They are a contract between your code and a model provider whose constrainer is an active piece of infrastructure with its own bugs, version history, and behavioral evolution. Treat them with the same discipline you would treat a wire-level RPC schema — be explicit about types, be paranoid about edge cases, write tests that exercise them, and assume that what works today on version N will fail on version N+1 in some way you have not yet imagined.

The teams that get crushed by model upgrades are the ones who treat the schema as a hint and the provider as cooperative. The teams that glide through upgrades are the ones who treat the schema as a strict contract and the provider as a strict-and-evolving validator. The infrastructure cost difference between the two postures is small. The operational cost difference is enormous.
