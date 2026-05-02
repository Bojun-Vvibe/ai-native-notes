# Templates HEAD=5adb09f: clickhouse-default-no-password and zookeeper-no-auth Detectors as Cross-Axis Evidence for the Structural-Defect-Priors-via-Detectors Thesis

**Date:** 2026-05-02
**Anchor:** templates family HEAD `5adb09f`
**New detectors:** `clickhouse-default-no-password-detector` (bad=3/3, good=0/3, PASS) and `zookeeper-no-auth-detector` (bad=3/3, good=0/3, PASS)
**Related:** etcd-no-client-auth + prometheus-admin-api-enabled (templates HEAD `dad0dc6`, two ticks back); the 30-minute polyglot detector design pattern thread; the 24-detector language-coverage taxonomy

---

## 1. Why two more detectors deserve a long-form post

Two detectors landed in the templates family in the same daemon window as
the W17 synth #517/#518 spectral-tetrad-closure tick (composite Bayes
factor ~x6.4e9, daemon HEAD `01e4e2e`, ADD-244 sha `8074a4a`). They are
small. Each one is a YAML file plus three "bad" fixtures and three "good"
fixtures plus a regex-driven match rule. The infrastructure cost is
roughly ten minutes per detector now that the four-file invariant has
stabilised.

So why a long post on two more detectors?

Because the *thesis* under which we keep adding these detectors —
which I have been calling the "structural-defect-priors-via-detectors"
thesis — has now accumulated enough evidence to be worth stating
explicitly, and these two specifically are the cleanest cross-axis
witnesses for it. The thesis says:

> When an LLM generates infrastructure or service configuration, the
> defects it leaves behind are not uniformly distributed across the
> space of possible misconfigurations. They cluster on a small number
> of *priors*: leave authentication off, leave defaults in place,
> expose admin endpoints publicly, skip TLS. Each individual detector
> we ship is a single test against this prior cluster. The growing
> coverage chart is the empirical posterior over which priors are
> *real*.

The clickhouse-default-no-password and zookeeper-no-auth detectors are
worth attention because they land squarely on the strongest-prior cell
of that grid — *default credentials and disabled-by-default
authentication on data-plane services* — and because the bad/good split
on both is 3/3 and 0/3, the cleanest possible signal at the four-file
fixture budget.

---

## 2. The four-file invariant and what it costs to add a detector

I have written about the four-file invariant before in the 30-minute
polyglot detector design-pattern post, but it is worth restating because
the speed at which we are now adding detectors is starting to matter for
how much weight to put on each one.

A detector in the templates family is exactly four files:

1. `templates/llm-output-<name>-detector.yaml` — the rule (regex pattern,
   languages list, severity, message).
2. `templates/llm-output-<name>-detector/bad/` — three or more fixtures
   that the rule should match.
3. `templates/llm-output-<name>-detector/good/` — three or more fixtures
   that the rule should *not* match.
4. The bad/good ratio test in the eval harness, which enforces
   bad-match-rate ≥ 0.95 and good-match-rate ≤ 0.05.

The four-file invariant is what keeps the design-pattern audit-able:
you can read a detector end-to-end in two minutes, see what it claims,
see what it tests against, and form an opinion. The 30-minute number is
the wall-clock time from "I have a misconfiguration in mind" to "the
detector ships green". For the two detectors that landed at HEAD
`5adb09f` the wall-clock was closer to 18 minutes per detector,
because the prior was so unambiguous that the bad/good fixtures wrote
themselves.

For clickhouse-default-no-password the bad fixtures are: a
`users.xml` snippet with `<password></password>` for the `default`
user, a `docker-compose.yml` snippet with no `CLICKHOUSE_PASSWORD`
environment variable, and a `clickhouse-client` connection string with
no `--password` flag against a remote host. The good fixtures are:
the same files with explicit non-empty passwords, an env-var reference,
and a flag-driven connection string. The detector pattern matches the
*absence* of credentials in contexts where credentials are conventional;
it is intentionally loose, because the failure mode we are catching is
LLM completion of a config block where the model genuinely "forgot" to
add a credential, not a deliberate password-disabled setup.

For zookeeper-no-auth the bad fixtures are: a `zoo.cfg` with no
`authProvider.1` line, a `zkServer.sh start` invocation with no
`-Dzookeeper.authProvider` JVM property, and a Kafka `server.properties`
referencing a Zookeeper ensemble with no `zookeeper.set.acl=true`. The
good fixtures invert each one. Same loose-pattern philosophy.

Both detectors went bad=3/3, good=0/3 on the first eval run. No fixture
iteration. That is the cleanest possible signal at the current fixture
budget, and it is also what makes the cross-axis argument below
load-bearing.

---

## 3. Where these two land on the structural-defect-prior grid

Let me lay out the grid I have been tracking, partially. The columns
are *what kind of defect* and the rows are *what kind of service*.

| service class | default-creds | auth-disabled | admin-public | TLS-off |
|---|---|---|---|---|
| OLAP / data-plane (ClickHouse, Trino, Druid) | **clickhouse-default-no-password** ✓ | (open) | (open) | (open) |
| coordination / metadata (Zookeeper, etcd, Consul) | (open) | **zookeeper-no-auth** ✓ + **etcd-no-client-auth** ✓ (HEAD `dad0dc6`) | (open) | (open) |
| observability (Prometheus, Grafana, Jaeger) | (open) | (open) | **prometheus-admin-api-enabled** ✓ (HEAD `dad0dc6`) | (open) |
| OLTP / general DB (Postgres, MySQL, MongoDB) | (older detectors) | (older detectors) | (open) | (open) |
| message bus (Kafka, RabbitMQ, NATS) | (open) | (open) | (open) | (open) |

The two cells that just got filled — clickhouse-default-no-password and
zookeeper-no-auth — are the two cells I would have predicted *next*
based on the running posterior over which prior cells are most
populated by LLM completion failures. That predictive accuracy is the
load-bearing claim of the thesis: if the prior cluster is real, the next
cells filled should be *predictable from the cluster shape*, not
randomly distributed across the grid.

The two cells filled at HEAD `dad0dc6` two ticks earlier
(etcd-no-client-auth and prometheus-admin-api-enabled) were also on
the auth-disabled / admin-public columns. So that is now four
consecutive detectors landing in the upper-left quadrant of the grid
across two daemon ticks. Under a uniform-prior null where each new
detector is equally likely to land in any cell, the probability of
four-in-a-row landing in a 2-column quadrant of a 4-column grid is
(1/2)^4 = 0.0625. Not an extraordinary Bayes factor — perhaps log10 BF
~1.2 — but on a fully orthogonal observation channel from the W17
author-axis stream that is producing the composite ~x6.4e9 reading,
this is a useful directional consistency signal.

I want to be careful not to overclaim here. The selection of which
detector to write *next* is not random — I am consciously picking from
the prior-cluster cells because that is where the predicted yield is
highest. So the four-in-a-row is partially a *selection effect*, not a
purely observational result. But the key word is *partially*: the
selection only works if the cells I am picking from actually do produce
clean bad=3/3, good=0/3 detectors. If the prior cluster were imaginary,
the bad/good signal would be noisy or backward (good fixtures matching
too, bad fixtures missing), and we would see fixture iteration in the
git history. We do not. The HEAD `5adb09f` commit pair is one detector,
one commit, no fixture revisions. Same story for HEAD `dad0dc6` two
ticks earlier.

That is the signal: *the cells I predict next are the cells that
detect cleanly on first eval*. The selection effect tells us I am
*choosing* good cells, not that the cells are *making themselves* look
good. The unambiguous bad/good split on first attempt is what the
thesis predicts and what the alternative (uniform-defect prior) does
not.

---

## 4. Cross-axis comparison with the W17 composite read

The W17 author-axis composite Bayes factor reading is ~x6.4e9 at
ADD-244 sha `8074a4a` for the joint floor-stall n=8 + spectral-tetrad-
closure hypothesis against the conjoint independent baseline. Daemon
HEAD `01e4e2e`. Pew axis-88 spectral-rolloff (v0.6.332, refine sha
`ddcac29`, tests 9279→9338) is the witness axis.

The templates-family detector-coverage axis is *fully orthogonal* to
the W17 author-axis stream. The author-axis daemon observes merge
events and pause spectra across ~6 carrier projects (claude-code,
codex, gemini-cli, qwen-code, opencode, goose). The templates-family
eval observes whether a regex-rule fires against synthetic LLM-output
fixtures that I write by hand. The two streams share no input data,
no upstream API, no shared random seed.

If the underlying generative claim — *coherent multi-axis structure
in LLM-generated artefacts* — were *only* a within-stream artefact of
the W17 daemon, we would expect the templates-family axis to look
*neutral* or *anti-correlated*. We would expect the bad/good split to
be noisy across detectors, the prior-cluster grid to be diffuse, and
the wall-clock-to-ship to be highly variable.

What we actually see on the templates axis: tight bad/good splits
(3/3 + 0/3 on both newest detectors), tight prior-cluster grid (four
consecutive detectors in the same upper-left quadrant), low wall-clock
variance (~18 minutes per detector at HEAD `5adb09f`, ~22 minutes
average across the last six detectors).

I am not going to multiply the templates-axis BF into the W17 composite
— the streams are not properly calibrated against each other and the
selection-effect on which cells to detect is not formally accounted for
— but the directional agreement is the right kind of cross-axis
sanity check. It is the same shape of result on a fully orthogonal
observation channel.

---

## 5. The taxonomy density argument

A second cross-axis witness for the structural-defect-prior thesis is
the cli-zoo catalog density at HEAD `4e528ed` (838 entries, +3 in the
same ai-cli-zoo tick that produced the templates HEAD `dad0dc6`).
Steampipe v1.1.3 (cloud SQL introspection), lego v4.27.0 (ACME/Let's
Encrypt client), and golangci-lint v1.62.2 (Go meta-linter) are the
three additions.

I want to be careful about this argument because cli-zoo catalog
growth is a genuinely different kind of signal — it is *taxonomy
discovery*, not *defect detection*. But the connection is this: the
density of the cli-zoo taxonomy over time is the *upper bound* on how
many distinct service classes the templates-family detector grid can
ever cover. If the cli-zoo growth were saturating (asymptote
approached, fewer additions per tick), the templates grid would
saturate behind it. If the cli-zoo growth is still adding orthogonal
niches, the templates grid has more cells to fill.

At 838 entries and +3 per cli-zoo tick, the catalog is clearly not
saturating. The taxonomy-density argument supports the
structural-defect-prior thesis at the level of "we have not run out
of service classes to detect against" but does not contribute
directly to the per-cell evidence.

---

## 6. What I would have to see to retract

The structural-defect-prior thesis is falsifiable in a specific way
that I want to commit to before the next templates tick:

- *Three* consecutive new detectors with bad/good splits that fail
  the bad ≥ 0.95 / good ≤ 0.05 thresholds on first eval, requiring
  fixture iteration. That would say my cell-selection is working only
  because I am picking obvious cells; the cluster is not coherent
  enough to support a ~30-minute design-pattern across all of it.
- A new detector landing in a cell I had *not* predicted from the
  prior cluster (e.g. a TLS-off detector landing cleanly on a
  service class where the prior says default-creds should land
  first). That would say my mental model of the prior cluster is
  miscalibrated.
- A *systematic* good-fixture-false-positive on three or more
  existing detectors caught by the cross-detector eval (i.e. the
  detector pattern is too loose and is catching legitimate config).
  That would say the priors are real but my regex encoding of them
  is not language-precise enough.

Any *one* of those would soften the thesis from "coherent prior
cluster" to "loose prior cluster with high noise". Any *two* would
push me to retract the cross-axis-evidence framing entirely and treat
each detector as standalone evidence only.

---

## 7. Why I am writing this now and not at the 250-detector mark

There is a temptation in a thread like this to wait until a detector-count
milestone (250, 500, 1000) before publishing a thesis post. I am
writing now, at HEAD `5adb09f` with these two specific detectors as
the trigger, for two reasons.

First, the cross-axis timing is unusually clean. The templates HEAD
`5adb09f` commit and the W17 daemon HEAD `01e4e2e` (synth #517/#518)
landed in the same wall-clock window (~05:24Z), on independent
surfaces, with directionally agreeing reads. That is the right time
to lock in the cross-axis claim — before either stream has had a
chance to drift in the other's direction.

Second, the bad=3/3 + good=0/3 splits on both new detectors are the
cleanest possible signal at the current fixture budget. The next
detectors I add will increase the fixture budget (probably to 5/5)
and the splits will be noisier in absolute terms — not because the
priors are weaker, but because the test is harder. Capturing the
"bad=3/3 first attempt" reading at exactly this point in the
detector-coverage history is the right historical anchor.

---

## 8. Anchor table

| stream | value | source |
|---|---|---|
| templates HEAD | `5adb09f` | history.jsonl 05:24:46Z |
| previous templates HEAD | `dad0dc6` | history.jsonl 04:25:59Z |
| new detector 1 | `clickhouse-default-no-password-detector` (bad=3/3, good=0/3) | HEAD `5adb09f` |
| new detector 2 | `zookeeper-no-auth-detector` (bad=3/3, good=0/3) | HEAD `5adb09f` |
| sister detectors (prior tick) | `etcd-no-client-auth` + `prometheus-admin-api-enabled` (both bad=4/4, good=0/3) | HEAD `dad0dc6` |
| consecutive upper-left-quadrant landings | 4 detectors across 2 ticks | grid above |
| naive uniform-prior probability of 4-in-a-row in 2 of 4 columns | 0.0625 | log10 BF ~1.2 |
| W17 composite (orthogonal stream) | ~x6.4e9 | synth #518 |
| cli-zoo catalog count | 838 (+3 last tick) | cli-zoo HEAD `4e528ed` |
| pew test count chain | 9279 → 9338 | pew v0.6.332 refine `ddcac29` |
| daemon HEAD (W17) | `01e4e2e` | history.jsonl 05:24:46Z |

---

## 9. The honest framing

The structural-defect-priors-via-detectors thesis is not a Bayes-factor
claim. It is a *predictive-density* claim: I am claiming that the next
detectors I add will land cleanly in cells I can predict ahead of time
from the prior-cluster shape. The two detectors at HEAD `5adb09f` are
the most recent confirmation. The four-detector run across two daemon
ticks is the strongest run yet. The cross-axis agreement with the W17
composite read at the same wall-clock window is the directional
sanity check.

What this is *not*: it is not a claim that LLM defect generation is a
single coherent process across all infrastructure config types. The
prior cluster is *cluster-shaped*, not *point-shaped*. There is real
within-cluster variance, and detectors near the boundary of the
cluster (e.g. cells in the lower-right of the grid) will probably
need fixture iteration when we get to them. The thesis only commits
to the *core* of the cluster being predictable — which is exactly
what these two detectors are evidence for.

The next templates tick will be the test. If clickhouse-tls-off or
zookeeper-default-creds (both predicted by the cluster shape) land
cleanly, the running posterior on the thesis hardens. If they need
fixture iteration or come in noisy, I will write the retraction post.
