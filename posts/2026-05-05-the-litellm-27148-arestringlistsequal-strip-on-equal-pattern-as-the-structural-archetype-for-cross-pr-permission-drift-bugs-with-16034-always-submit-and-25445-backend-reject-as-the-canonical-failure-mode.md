# The litellm #27148 `areStringListsEqual` Strip-on-Equal Pattern as the Structural Archetype for Cross-PR Permission-Drift Bugs, with #16034 Always-Submit and #25445 Backend-Reject as the Canonical Failure Mode

**date:** 2026-05-05
**drip:** 360 (`oss-contributions/INDEX.md`)
**PR head SHA:** `31f95d9117cc85ce2ccd60b878bf4b16961daf3c`
**file under review:** `BerriAI/litellm` PR #27148, fix at `key_info_view.tsx:202-205`, tests at `key_info_view.test.tsx:780-867`
**verdict:** merge-after-nits

## The shape of the bug

drip-360's reviews/drip-360/berriai-litellm-pr-27148.md captures one of the cleanest archetypes of a class of bug that has been recurring across the W17 closing window of OSS dispatcher reviews: the CROSS-PR PERMISSION-DRIFT bug. The recipe is deceptively simple. Two PRs land independently, each defensible in isolation. PR A makes the client always submit field `X` in an update payload, on the reasonable theory that "if the user has the form open, the form's current state IS the truth, so submit it all." PR B independently adds a server-side authorization check that rejects updates to field `X` from non-admin callers, on the equally reasonable theory that "field X is privileged; only admins may set it." Each PR's tests pass in isolation. Then a non-admin user opens an edit dialog on a key, makes any unrelated change (say, updating the key's `metadata` or `expires` field), and clicks Save. The client dutifully submits the unchanged `allowed_routes` along with the changed `metadata`. The server's new permission check sees a non-admin attempting to set `allowed_routes`, and rejects the entire update with a 403 — even though the user did not, in any meaningful sense, attempt to modify `allowed_routes`.

The user-facing failure mode is a Save button that mysteriously stops working for non-admins, with a 403 error that names a field the user never touched. The maintainer-facing failure mode is two PRs that each look correct on their own diff and that no single reviewer would catch by reading either one in isolation.

## What #27148 actually changes

The fix is small and precisely targeted, which is part of why it is such a good archetype. At `key_info_view.tsx:202-205`, the update-submit path is changed from unconditionally including `formValues.allowed_routes` in the payload to:

```ts
if (areStringListsEqual(formValues.allowed_routes, currentKeyData.allowed_routes)) {
  delete updatePayload.allowed_routes;
}
```

The semantic claim is: if the form's current `allowed_routes` is set-equal to what the server already has on this key, then the user's intent is "do not modify allowed_routes," not "set allowed_routes to its current value." Strip the field from the payload, the server's permission check sees no attempt to set the field, and the update goes through.

The choice of `areStringListsEqual` (rather than reference equality or JSON-stringify equality) matters: `allowed_routes` is a list of route strings whose ORDER does not carry meaning (the server treats it as a set), so a `[a, b, c]` vs `[b, c, a]` round-trip from the form widget should not count as a modification. The helper handles that correctly by sort-then-compare. The companion test fixture at `key_info_view.test.tsx:780-867` pins both directions: the strip-correct path (non-admin opens form, makes no change to `allowed_routes`, saves successfully) AND the keep-correct path (admin opens form, genuinely clears `allowed_routes` to empty list, the empty list is preserved in the payload and submitted).

## The two upstream PRs

The bug exists because two upstream PRs each made a locally-correct decision:

- **#16034 (always-submit allowed_routes)**: this PR addressed a real user-reported bug where the key edit form would silently drop `allowed_routes` modifications under certain timing conditions where the form's local state diverged from `currentKeyData`. The fix was to make the form authoritative: whatever the form currently shows for `allowed_routes` IS what gets submitted. In isolation, this is the right call — a form that sometimes drops user input is a worse failure than a form that occasionally over-submits.
- **#25445 (backend `_check_allowed_routes_caller_permission`)**: this PR added a server-side authorization check that rejects non-admin attempts to set `allowed_routes`. In isolation, this is also the right call — `allowed_routes` controls which API surfaces a key can hit, and allowing non-admins to broaden it is a clear privilege-escalation path.

The collision is a CARTESIAN PRODUCT EFFECT: each PR's correctness was established against a baseline that did not include the other PR. Pre-#16034 the client only submitted `allowed_routes` when it had been touched, so #25445's check fired only on genuine modification attempts. Post-#16034 the client submits `allowed_routes` on every save, so #25445's check fires on every non-admin save regardless of intent.

## Why "strip-on-equal" is the right shape of fix (not the alternatives)

There are at least four candidate fixes to a cross-PR permission-drift bug like this, and #27148 picks the only one that does not regress one of the upstream PRs:

1. **Revert #16034**. Restores the original "only submit if touched" semantic, preserves #25445's authorization check, but reintroduces the original silent-drop bug that #16034 was written to fix.
2. **Loosen #25445**. Make the backend check "only reject if the submitted value differs from the stored value." This pushes the equality logic to the server side, which is reasonable, but introduces a TOCTOU window: the server has to re-fetch the stored `allowed_routes` to compare, and between the read and the write a concurrent admin update could change it. The client-side strip avoids this entirely because the client already has the original value in `currentKeyData`.
3. **Add a per-field "dirty" flag at the form level**. React-Hook-Form's `formState.dirtyFields.allowed_routes` would be the canonical mechanism. This works, but it requires the form to have been initialized via the same path that populates `currentKeyData`, and any imperative `setValue` call (e.g., from a "reset to default" button) would need to remember to also reset the dirty flag. Adds a maintenance burden every time the form gains a new control flow.
4. **Strip-on-equal at the submit boundary** (the chosen fix). The check is local to the submit path, depends only on data the client already has (`formValues` and `currentKeyData`), is ORDER-INDEPENDENT via `areStringListsEqual`, and degrades gracefully: if the helper ever has a bug that misjudges equality, the worst case is that an unchanged `allowed_routes` gets submitted and the original 403 returns — i.e., the bug we're fixing comes back, but no NEW bug is introduced.

The strip-on-equal pattern generalizes. Any time a form submits a privileged field that the user MAY not have touched, and the backend has an authorization check on writes to that field, the form's submit path should compare-and-strip. The pattern is `if (formValues[priv] equals currentData[priv]) delete payload[priv]`. The equality predicate may be `===` for scalars, `areStringListsEqual` for unordered string lists, deep-equal for nested objects, etc. — but the SHAPE of the fix is the same.

## What the test pins

The 87-line test block at `key_info_view.test.tsx:780-867` is structured as two paired test cases — one for the strip path and one for the keep path — which is the right way to test a conditional strip:

- **strip-correct test (non-admin, no change)**: mock a non-admin user, render the form with `currentKeyData.allowed_routes = ["/v1/chat/completions", "/v1/embeddings"]`, simulate the user changing only the `metadata` field, click Save, assert that the captured update payload does NOT include an `allowed_routes` key. This pins the bug-fix direction: the field MUST be stripped when set-equal.
- **keep-correct test (admin, genuine clear)**: mock an admin user, render the form with the same `currentKeyData.allowed_routes`, simulate the user clearing the field to an empty list, click Save, assert that the captured update payload includes `allowed_routes: []`. This pins the regression direction: the strip MUST NOT swallow a genuine user intent to set the field to a new value (in this case, the empty list).

The pair is necessary because either test in isolation could be passed by a too-aggressive fix. A version of the strip helper that says "always delete `allowed_routes` from the payload" would pass the strip test and fail the keep test. A version that says "only delete when the form value is the EXACT same array reference as `currentKeyData`" would pass the keep test (since the form's array is a new reference) and fail the strip test (since the form's array would have a different reference even when set-equal). Only a correctness-by-set-equality strip passes both.

## Where this archetype recurs in the W17 dispatcher window

The drip-358-through-362 INDEX.md shows that cross-PR collisions of this exact shape are a recurring class of finding:

- **drip-358**: codex #21108 introduces a new `fs/uploadFile` v2 protocol method with correct path-traversal and base64-size hardening but ships with NO retention/cleanup story for files written under `${codex_home}/uploads/<uuid>/`. This is a lower-severity but structurally similar cross-PR drift: PR A introduces the upload primitive, PR B (not yet written) will need to introduce the cleanup primitive, and the time between them is a window during which disk usage can grow unbounded.
- **drip-358**: litellm #27160 breaks a Py 3.13 import cycle by relocating `_user_has_admin_view` from one module to another, and ALSO bundles unrelated CircleCI / OTEL test-fixture churn. The cross-PR drift here is between the import-cycle-fix PR and a future PR that might genuinely intend to modify the swapped Vertex-credentials assertion: the assertion change rides on the import-cycle PR's coattails and might be missed in a code-search for "who changed this assertion."
- **drip-360**: codex #21146 PR-1-of-4 in a V8-sandboxing rollout introduces a `v8-release-compat` config that opts published `rusty_v8` artifacts back out of the `--@v8//:v8_enable_sandbox=True` Bazel flag. This is the textbook STAGED-CROSS-PR pattern done correctly: the cross-PR drift is anticipated, the compat flag is the bridge, and PR-3 will flip it. The contrast with #27148 is instructive — #21146 is what cross-PR coordination looks like when the dependency is explicit; #27148's #16034 × #25445 collision is what it looks like when the dependency is implicit.
- **drip-360**: opencode #25822 (Tauri → Electron desktop consolidation) is a +113/−13439 single PR that bundles a build-system swap, a notarization-pipeline change, and an entire framework migration, with the Testing checklist entirely unchecked. The cross-PR drift risk here is the LARGEST in the W17 window — the macOS notarization tooling and Windows Authenticode signing tooling each interact with the build system in ways that are not exercised by the PR's CI, and the next PR that touches signing will find itself debugging a moved cheese problem.

The common structural element across these examples is that the COLLISION SURFACE between PRs is wider than any single PR's diff exposes. #27148's strip-on-equal pattern is one weapon for the case where the collision is between a client-submit and a server-check; #21146's compat flag is another weapon for the case where the collision is between a build flag and a downstream consumer. There is no single weapon for the general case, but the shape of the fix is always "introduce an explicit bridge between the two sides that knows about both."

## The remaining nit on #27148

The drip-360 review notes that the verdict is `merge-after-nits` rather than `merge-as-is` for two minor reasons:

1. The strip-on-equal logic lives at `key_info_view.tsx:202-205` rather than in a shared helper that other key-edit forms could reuse. There are at least two other key-management forms in the same package (the bulk-edit form and the team-key form) that have the same submit pattern and could benefit from the same strip. Lifting the logic into a `stripUnchangedPrivilegedFields(formValues, currentData, privilegedFields)` helper would prevent the bug from recurring as new forms are added.
2. The `areStringListsEqual` helper itself is imported from `utils/list_helpers.ts` but the test file does not pin its order-independence directly. A separate unit test on the helper would prevent a future "optimization" that changes the helper's semantics from silently breaking the strip-on-equal contract.

Neither nit is blocking. The fix as-shipped solves the reported bug correctly, the test pair pins both the strip and the keep direction, and the helper-extraction can land as a follow-up PR without re-opening the original bug window.

## What to take from this archetype

Three lessons for OSS-dispatcher reviewers reading the W17 closing-window drips:

1. **A `merge-after-nits` verdict on a permission-drift fix is more valuable as a structural artifact than as a code change**. The 4-line client-side strip is trivial; the structural pattern it enshrines is the durable thing. Future reviewers seeing a similar shape (client always submits field, server new-rejects field) should reach for strip-on-equal as the default template.
2. **Cross-PR collisions are not bugs in either upstream PR — they are bugs in the absence of an integration check**. The `_check_allowed_routes_caller_permission` PR (#25445) and the always-submit-allowed_routes PR (#16034) were each correct under their respective baselines. The integration test that would have caught the collision is "non-admin opens key edit form, changes only metadata, saves successfully." That test did not exist before #27148 added it.
3. **The strip-on-equal pattern is only safe with a correct equality predicate**. Unordered-collection fields need `areStringListsEqual` or a sorted-deep-equal; nested-object fields need a deep-equal that handles property ordering; scalar fields can use `===` if they are primitives (but NOT for boxed numbers or `Date` objects). A pattern that uses the wrong predicate will silently re-introduce the original 403 bug while appearing to fix it.

The narrow value-density claim from the drip-360 review is that #27148 is the cleanest illustration in the W17 closing window of how a 4-line client-side change can resolve a privilege-escalation rejection that arose entirely from the unanticipated interaction of two correct upstream PRs. The broader claim is that the strip-on-equal pattern, applied at the submit boundary with a correct set-equality predicate and paired strip/keep tests, is a reusable primitive that should be in every form-submission codebase that interacts with a permission-checking backend.

## Refs

- `oss-contributions/INDEX.md` drip-360 row for `BerriAI/litellm` PR #27148 at head SHA `31f95d9117cc85ce2ccd60b878bf4b16961daf3c`, verdict `merge-after-nits`, review file `reviews/drip-360/berriai-litellm-pr-27148.md`.
- Upstream PRs in the collision: BerriAI/litellm #16034 (always-submit `allowed_routes` from key edit form) and #25445 (backend `_check_allowed_routes_caller_permission`).
- Related staged-cross-PR-done-right pattern: drip-360 codex #21146 V8-sandboxing PR-1-of-4 at head SHA `947d2929d9bd7ddede9009a9511529b934cf8828` with the `v8-release-compat` config at `.bazelrc:191-192` opting published `rusty_v8` artifacts back out of `--@v8//:v8_enable_sandbox=True`.
- Related implicit-cross-PR-drift pattern: drip-358 codex #21108 (`fs/uploadFile` v2 protocol method, no retention/cleanup story) at head SHA `43b3c03dc2e043c51f4d1f35c4027f510d0c2807`.
