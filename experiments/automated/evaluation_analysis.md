# Cloud Evaluation Analysis: Trail Guide Agent

## Evaluation Summary

Evaluated: 89 test cases
Scoring: GPT-5.1 as an LLM judge (1-5 scale)
Eval ID: `eval_c22c086579ea45fe93f31aa1d3a0fb0f`
Run ID: `evalrun_7a41004cefb3426aba47e28e270c87a6`

| Evaluator | Average Score | Pass Rate | Assessment |
|-----------|---------------|-----------|------------|
| Intent Resolution | 4.98 | 100.0% | Excellent intent understanding |
| Relevance | 4.99 | 100.0% | Excellent query-response alignment |
| Groundedness | 4.97 | 98.9% | Excellent factual accuracy |
| **Average** | **4.98** | **99.6%** | **Very High Quality Overall** |

## Key Findings

### Strengths

- Near-perfect scores across all three quality dimensions (>4.9 average)
- 100% pass rate on both Intent Resolution and Relevance — every response addressed what the user actually asked and stayed on-topic
- Groundedness pass rate of 98.9% (88/89) shows responses are almost always fully supported by the reference context

### Areas for Improvement

- One item failed Groundedness (score 2/5) — the only failure across the full dataset
- The failure pattern is specific: the agent added unsupported specifics (exact percentages) not present in the source material, and inverted a directional detail

### Failed Evaluation Analysis

**Query**: "How can I estimate how long a hike will take?"

**Ground truth**: Use Naismith's Rule (1hr/3mi + 1hr/2000ft gain), adjust for fitness and terrain, add breaks. Descending 1.5-2x faster. Always add buffer time.

**Agent response (excerpt)**: Explained Naismith's Rule correctly, but stated "fit hikers add 30%, less fit subtract 30%" to hike time — the ground truth only says "adjust for fitness" with no specific direction or percentage.

**Judge reasoning**: The core method (Naismith's Rule, terrain, breaks, descent speed, buffer time) was correctly grounded. However, the agent fabricated a specific fitness adjustment that is both unsupported by the source and logically backwards (a fitter hiker should need *less* extra time, not more).

- **Common failure pattern**: Agent fills in plausible-sounding specifics (exact percentages) when the source material is vague, rather than stating the adjustment qualitatively
- **Query type affected**: Estimation/calculation questions where the ground truth gives a rule but not exact tuning parameters
- **Recommended improvement**: Add an instruction to the agent prompt discouraging invented numeric specifics when the source guidance is qualitative — prefer phrasing like "adjust pace for your fitness level" over invented percentages

## Automated Evaluation Benefits

- **Scales** to hundreds/thousands of items efficiently
- **Consistent** scoring criteria across all evaluations
- **Repeatable** and trackable over time
- **CI/CD ready** — this evaluation now runs automatically via GitHub Actions on PRs touching `src/agents/trail_guide_agent/**`
- **Detailed reasoning** provided for each score, enabling root-cause analysis like the one above

## Recommended Use Cases

| Scenario | Recommended Approach | Rationale |
|----------|---------------------|-----------|
| Testing new prompts (50+ queries) | **Automated** | Scale, speed, consistency |
| Continuous integration testing | **Automated** | Fast feedback in pipelines |
| Baseline establishment | **Automated** | Quantifiable metrics at scale |
| Production monitoring (ongoing) | **Automated** | Continuous quality tracking |
| Investigating edge cases | **Manual review** | Deep dive into specific failures |

## Next Steps

1. Use automated evaluation as primary quality gate for agent changes
2. Automated evaluation is now wired into CI/CD (GitHub Actions, PR-triggered)
3. Establish alerting thresholds (e.g., groundedness < 4.5 fails deployment)
4. Schedule regular evaluations to track quality over time
5. Add a prompt instruction against inventing unsupported numeric specifics, then re-run this dataset to confirm the fix
