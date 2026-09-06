# 0.1.4 clean 3-judge run — the corrected defect prompt, measured

The first full AACR-Bench run since the defect prompt was corrected. Same conditions as
`../results-clean-3judge/` in every respect that was held fixed: judges codex + big-pickle

- nemotron, `--effort high`, 900 s timeout, `--prompt-style defect`, `--cwd-mode empty`,
  the seed-42 samples, extractor 3. Binaries md5-pinned in `BINARIES.txt` (source tree clean
  at `e2ad666`), so what produced these numbers is identifiable — `git rev-parse HEAD` is not.

Scored by the **upstream** evaluator with a real LLM judge (`anthropic/claude-opus-4.5`,
outside the panel's own model families). `judge failures during scoring: 0` on all six
scorings run for this page — both legs here, both legs of the August arm re-scored for
comparison, and two replicates.

## Read this against a re-scored August, not the committed one

`../results-clean-3judge/metrics-*-k1.json` are the extractor-2 scorings; that directory's
README supersedes them with the extractor-3 figures. Both arms below were scored **today,
by the same evaluator invocation**, so the comparison is like-for-like. The re-score
reproduced the published extractor-3 numbers exactly (15/123 and 4/36, ratio 1.10,
Fisher p = 1.0), which is the evidence that this pipeline is reproducible at all.

| positive (123 valid refs) | Aug `ac22d82` |     Sep `e2ad666` |
| ------------------------- | ------------: | ----------------: |
| instances                 |         18/20 |             18/20 |
| judge slots               |         54/54 | 53/54, 1 degraded |
| findings                  |            91 |           **126** |
| line matches              |            28 |                31 |
| semantic matches          |            15 |                12 |
| semantic recall           |         12.2% |          **9.8%** |
| semantic precision        |         16.5% |          **9.5%** |

| negative (36 REJECTED refs) |     Aug `ac22d82` | Sep `e2ad666` |
| --------------------------- | ----------------: | ------------: |
| instances                   |              9/10 |          9/10 |
| judge slots                 | 26/27, 1 degraded |         27/27 |
| findings                    |                46 |        **78** |
| line matches                |                 8 |            15 |
| semantic matches            |                 4 |             5 |
| semantic recall             |             11.1% |         13.9% |

## What moved, and what did not

- **Recall did not move.** Paired on the same 123 references (`../../aacr-mcnemar`):
  12.2% → 9.8%, **McNemar p = 0.51** (9 both, 3 gained, 6 lost). The negative leg is
  11.1% → 13.9%, **p = 1.0**. Wilson intervals overlap across most of their range
  ([7.5, 19.1] vs [5.7, 16.3]). Nothing here is a recall regression.
- **Volume moved decisively.** Findings rose 38% on the positive leg and 70% on the
  negative one, and both counts are deterministic — no judge is involved in counting them.
  Precision is the arithmetic consequence: 16.5% → 9.5% and 8.7% → 6.4%. The panel says
  considerably more without matching more references.
- **The prompt is the cause, not the audit.** Diffing the literal `prompt.md` recorded in
  each run's own directory: the August prompt said to report only real defects and to
  abstain when the diff "contains no real defect"; the current one asks for file and line
  "for every item" and abstains only when the diff "needs no comment at all". That is
  commit `06f6f0d` ("the 10.6% was measuring my own prompt, not the panel"), which landed
  after the August run. Abstentions fell from 15 of 54 reviews to 8 of 53.
- **Still no discrimination between accepted and rejected comments.** 9.8% valid vs 13.9%
  rejected, ratio 0.70, Fisher p = 0.54. August was ratio 1.10, p = 1.0. Two runs, two
  prompts, the same verdict: the panel cannot tell a comment maintainers accepted from one
  they rejected.

## Controls

- **Evaluator replicate.** Re-scoring an identical results directory moves at most one
  match: August 15 → 14, this run 12 → 12, findings and line matches identical both times.
  So the 3-match drop is above evaluator noise but sits at the panel re-run floor the
  ledger already measured (±3 of 150).
- **Same judges.** Every instance's `run.json` records the same three model ids in both
  runs: `gpt-5.6-sol`, `opencode/big-pickle`, `opencode/nemotron-3-ultra-free`. Model drift
  is excluded, not assumed.
- **Same extractor.** Version 3 on both sides.

## Degradation and drops

One instance ran degraded, `lvgl__lvgl@4a57db3`, where big-pickle was unavailable and two
of three judges answered. The negative leg ran a full roster, which August's did not
(`electron__electron@2cc5656` was degraded there). A judge that failed is not a judge that
found nothing.

Three instances have head commits GitHub no longer serves, so no diff exists to review:
`astral-sh__uv@ed57db2` and `comfyanonymous__ComfyUI@cfc3122` on the positive leg,
`keycloak__keycloak@1463502` on the negative. The August run dropped the same three.

## Provenance note

The August directory cites source HEAD `17c7958`, which no longer resolves — the privacy
scrub renamed every hash. Its recorded binary md5 still identifies the commit: the
`llm-panel` blob at `ac22d82` on `backup/main-pre-scrub` hashes to the `86484d63…` in that
run's `BINARIES.txt`. Content survived the rewrite even though hashes did not, so a
results directory citing a dead hash is still identifiable by md5 over history.
