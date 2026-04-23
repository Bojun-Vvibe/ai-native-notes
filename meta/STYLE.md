# Style guide (for the author)

Internal rules. Re-read before publishing each post.

## Voice

- No emoji. Anywhere. Not in titles, not in headings, not in commit messages.
- No marketing language. Banned words include: "revolutionary", "game-changer", "unleash", "supercharge", "next-generation", "10x", "paradigm shift", "disrupt", "delight".
- No hedging filler: "I think maybe perhaps it could possibly be the case that..." — cut.
- Second person sparingly. First person is fine. Avoid "we" when it means "I."

## Substance

- Concrete beats abstract. Numbers beat adjectives. "Cut p95 from 4.2s to 380ms" beats "significantly faster."
- Lead with the failure mode, not the success. Readers learn more from what broke than from what worked.
- If a claim has no number and no link, it is an opinion. Label it as one.
- Code samples must run as written. If a sample requires setup, the setup is part of the sample.

## Sourcing

- Always link out to public sources: docs, RFCs, GitHub issues, blog posts, papers.
- Never link to anything behind a corporate SSO, internal wiki, internal chat, or any private workspace.
- If the only source is "I observed this on my machine," say so explicitly.

## Scope

- One post = one idea. If a draft has two ideas, split it.
- If a post would be filler, it doesn't ship that day. The cadence is "roughly daily," not "definitely daily."

## Privacy and safety

- No employer, client, codename, internal product name, internal hostname, internal channel, or internal URL.
- No screenshots that contain UI chrome from a private system.
- Run the guardrail script before every commit. The guardrail is the source of truth, not memory.
