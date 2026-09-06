# The planted-defect corpus — what `panel-recall` measures

`panel-recall` runs the panel over `fixtures/` — defects planted in real code, each one
proven to misbehave by execution — and reports which known targets any pass matched. The
README's [What it actually catches](../README.md#what-it-actually-catches) quotes the
headline (25 of 27 targets, four passes); this page keeps the three results behind it.
It is a development instrument for controlled A/Bs where ground truth must be known and
iteration cheap, not evidence of absolute capability. The real-world numbers are in
[`benchmarks/README.md`](benchmarks/README.md).

Three results worth knowing before you trust any of the output:

- **Recall was limited by the roster, not by the models.** The two defects that panel
  never found — a `.get(k, default)` that doesn't apply to an explicit `null`, and a
  corrupt cache file silently becoming empty — are both found by a **six-vendor** panel
  (OpenAI / NVIDIA / Zhipu / Moonshot / DeepSeek / xAI): 4/6 → **6/6** on those two
  fixtures. The best two judges there, at 4/6 each, beat codex at 2/6 — and both were
  broken or out of credit until the roster was repaired. If your panel is missing things,
  check who is actually answering before concluding the models can't see it.
- **Running the same model twice recovered nothing.** First passes 25/27, with repeats
  25/27. The repeat-passes idea is well supported in the literature and did not reproduce
  here. An earlier grader bug reported +1 and it was an artifact. Adding a _different
  vendor_ did what adding a second pass of the same one could not.
- **Letting judges say "nothing is wrong here" is a precision/recall trade, not a free
  win either way.** One sentence of abstention licence is the whole difference.

  |             | findings/fixture | false positives       |
  | ----------- | ---------------- | --------------------- |
  | licence on  | 0.42             | 0 / 6 judges          |
  | licence off | 2.17             | 2, from 1 of 5 judges |

  Findings-per-fixture is measured on fixtures that _do_ contain defects, where the extra
  findings were verified **true** — so the licence suppresses real findings (one judge went
  3.00 → 0.00 on files with genuine defects). False positives are measured on
  `p01-exhaustive-codec`, the one fixture with **proven** absence rather than verified
  scope — which is what makes a false-positive rate computable at all. There, the same
  judge on the same code abstained with the licence and produced two demonstrably false
  findings without it (it claimed int and str subclasses were rejected; `encode(MyInt(1))`
  returns `'A'`).

  So: the licence costs true findings and prevents false ones. Which you want depends on
  whether chasing a false lead costs you more than missing a real defect. Caveat worth
  stating: one proven fixture, eleven reviews.
