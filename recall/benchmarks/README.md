# AACR-Bench runs — everything behind the README table

The README's [On real PRs](../../README.md#on-real-prs-aacr-bench) table is the headline;
this page is what stands behind it. Every number comes from running the panel over
[AACR-Bench](https://github.com/alibaba/aacr-bench) PRs and scoring the findings with
**upstream's own evaluator** — `aacr-upstream` runs the panel and hands the findings over,
`aacr-score` invokes the evaluator and refuses to report a number from a judge that isn't
running. The three prompt styles are `aacr-upstream --prompt-style {defect,broad,volume}`.

## What keeps the numbers honest

- **The variance floor is measured.** Re-running the same judge on the same 35 PRs moves
  up to ±3 human-reference matches of 150, with an evaluator replicate at exactly zero —
  so effects under ~5–7 pp of recall are re-run noise at this n, which every subgroup
  claim so far was (`results-human-2arm-orgpt/perjudge35/`).
- **Three earlier readings were withdrawn on re-measurement**: a DEFECT/IMPROVEMENT
  split (the classifier was circular — `reference-categories/`), "broad finds different
  hits" (pre-registered replication on 35 fresh PRs, p = 0.40 — `results-human-2arm/`),
  and a transport/harness effect (its 13-PR foothold did not survive a re-run; a
  same-transport re-run of another judge moved as much — `results-human-2arm-orgpt/`).
  The audit trail is in each directory's README; nothing in the headline table rests on
  a withdrawn claim.
- **Diff-in-prompt review is the measured condition** — each panel runs in an empty
  directory with the diff in the prompt. The paired repo-checkout arm moves recall
  12.2% → 15.4% (p = 0.48) while _losing_ 7 of the diff arm's matches and gaining 11:
  repo access changes what judges attend to more than it strictly adds
  (`results-checkout-3judge/`).
- **Location agreement overstates semantic agreement ~2x** (22.8% of references had a
  finding at the right file and line; 12.2% had one a judge called the same concern) —
  which is why scoring is delegated upstream instead of done by a local matcher.
- **A degraded roster costs about half the recall** (6.5% vs 12.2% with one judge's
  quota spent and a 300s timeout, same extractor — `results-pilot-2judge/`). Check who
  actually answered before reading any number.
- **The panel does not discriminate accepted from rejected reviewer comments**
  (12.2% vs 11.1%, Fisher p = 1.0).
- **Unlocated findings are withheld from upstream, not handed over empty** — upstream's
  filters treat a missing path or line as match-everything, and passing them through
  inflated line matches from 20 to 50 on the first scoring run.

## Prompt styles, and the paper's baselines

The README keeps the two tables; this is the reading of them.

`broad` — asking for what a careful maintainer would actually raise — doubles the recall
of the `defect` arm it was paired against (McNemar on paired references, p = 0.0005; that
arm was the pre-rewrite prompt at 12.2%, not the row above). But the `volume` control
shows what that class of gain is made of: it is the `defect` prompt plus one
exhaustiveness clause, reaches the same recall (p = 1.0 vs broad), and pays for it with
half of broad's precision. On a 35-PR replication the ordering holds on both transports
while every arm's precision falls (broad ~9.7%, volume ~5.5–6.1%, ~16–18 findings read
per hit). A declared cost cut over all of it settled the product default: **it stays
`defect`**; the only candidate for a future default change is `broad`
(`recall/benchmarks/cost-cut/README.md`).


The rows are **not directly comparable** and the gap should be read with that in mind:
ours is an 18-PR subsample, scored by upstream's evaluator code with `claude-opus-4.5` as
the judge where the paper used Qwen3-235B, without the PR title and description, at line
tolerance k = 1 where the paper says only "overlaps", and it is a three-judge panel of one
subscription model and two free-tier ones where every paper row is a single frontier
model. The paper's agentic condition (Claude Code with repository access) scores 10.1%
recall at 39.9% precision, so the paper itself shows recall and precision trading against
each other by an order of magnitude across conditions. What can be said: the `defect`
prompt sits at the low-recall end of that spread, `broad` sits inside the paper's
no-context recall range at better-than-paper precision, and nothing here has been measured
on the full 200.

## Run ledgers

Each directory's README is the ledger for that run: what was declared before it, what
was measured, and what was withdrawn.

| directory                   | what it is                                                                               |
| --------------------------- | ---------------------------------------------------------------------------------------- |
| `results-clean-3judge/`     | the headline result — clean 3-judge run                                                  |
| `results-0.1.4-3judge/`     | 0.1.4 under the corrected defect prompt — recall flat, volume up, precision halved       |
| `results-clean-2judge/`     | the roster-matched control, two judges                                                   |
| `results-broad-3judge/`     | broad prompt vs defect prompt, the same panel asked a wider question                     |
| `results-volume-3judge/`    | the volume arm — the defect prompt told not to stop                                      |
| `results-checkout-3judge/`  | the same panel standing in the repository instead of reading a diff                      |
| `results-human-2arm/`       | human-authored references, broad vs volume, pre-registered                               |
| `results-human-2arm-orgpt/` | the transport-controlled replication of the above; `perjudge35/` is the floor            |
| `results-pilot-2judge/`     | the pilot — 2 judges, degraded; not a headline result                                    |
| `cost-cut/`                 | the declared cost-efficiency cut that settled the default prompt                         |
| `reference-categories/`     | what each reference comment asks for — RETRACTED 2026-08-26                              |
| `scores/`, `upstream/`      | the evaluator's outputs and upstream's own scoring inputs                                |
| `diffs/`, `diffs-upstream/` | the PR diffs the panels were shown — see the note below on what is committed             |
| `checkouts/`                | shallow clones for the checkout arm — gitignored; `aacr-upstream checkout` rebuilds them |

What is committed under `diffs-upstream/`: 58 diffs for the 63 sampled PRs
(`upstream/pos-seed42-n20.jsonl`, `neg-seed42-n10.jsonl`, `pos-human-n35.jsonl`),
fetched once from GitHub's compare API and kept as the exact bytes the panels saw. The
five without one: the n8n and timescaledb PRs, whose diffs are never committed (see
[PROVENANCE.md](PROVENANCE.md)) and which `aacr-upstream run` refetches into the local
cache, and three seed-42 PRs (astral-sh/uv, comfyanonymous/ComfyUI, keycloak/keycloak)
whose compare fetch never succeeded. The committed result and score JSONs do not need
any diff — `aacr-score` works from those alone.

Data licensing: the PR diffs and review-comment text are third-party material under
their upstream terms — see [PROVENANCE.md](PROVENANCE.md).
