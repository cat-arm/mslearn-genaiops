# Threshold Analysis: Trail Guide Agent

## Investigation Goal

Explore how different pass/fail thresholds affect scoring outcomes for the
same 89-item evaluation run, and recommend a production threshold.

This analysis reuses the raw per-item scores from the existing run
(`evalrun_7a41004cefb3426aba47e28e270c87a6`) rather than re-running the cloud
evaluation — the scores are already known per item, so pass rate at any
threshold can be recomputed directly.

## Score Distribution

The judge's scores are strongly bimodal: almost every item scores a perfect
5.0, with a small number of exceptions and no scores in between.

| Metric | Score distribution (n=89) |
|--------|---------------------------|
| Intent Resolution | 87 × 5.0, 2 × 4.0 |
| Relevance | 88 × 5.0, 1 × 4.0 |
| Groundedness | 88 × 5.0, 1 × 2.0 |

## Pass Rate Comparison by Threshold

| Threshold | Intent Resolution | Relevance | Groundedness | Overall Avg |
|-----------|-------------------|-----------|---------------|-------------|
| >= 3.0 (default) | 100.0% | 100.0% | 98.9% | 99.6% |
| >= 4.0 | 100.0% | 100.0% | 98.9% | 99.6% |
| >= 4.5 | 97.8% | 98.9% | 98.9% | 98.5% |
| >= 5.0 | 97.8% | 98.9% | 98.9% | 98.5% |

## Key Observation

Raising the threshold from **3.0 to 4.0 costs nothing** — every item that
passes at 3.0 also scores at least 4.0 in this dataset, except the single
groundedness failure (score 2.0) which fails at every threshold tested.
That means 4.0 acts as a free safety margin: it would catch any future
borderline response scored exactly 4.0 as still passing, while giving no
room for a weak 3.0-or-3.5 response to slip through unnoticed.

Raising the threshold further to 4.5 or 5.0 starts rejecting responses that
are otherwise good (the three 4.0 scores) — this introduces false
positives (flagging acceptable responses as failures) without gaining any
additional detection of real problems, since the one genuine failure
(groundedness score 2.0) is already caught at every threshold from 3.0
upward.

## Recommendation

**Use threshold = 4.0 for production.**

- Justification: in this dataset it changes zero pass/fail outcomes
  compared to the default of 3.0, so it carries no false-positive risk
  today, but it removes the wide 3.0–3.9 range where a genuinely mediocre
  response could still register as "pass." It shifts the bar from "not
  obviously bad" to "clearly good," which better matches an LLM judge that
  otherwise appears to award near-perfect scores by default.
- Do **not** raise the threshold to 4.5 or 5.0: the judge's scoring
  granularity in this run shows only 5.0, 4.0, or a hard failure (2.0) — no
  scores land in the 4.0–4.9 band. A 4.5+ threshold would therefore
  reject legitimately good responses (the 4.0 cases) as failures, which is
  a risk-tolerance trade the data does not support.
