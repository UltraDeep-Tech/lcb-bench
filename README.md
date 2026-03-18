# LCB: LLM Cognitive Bias Benchmark

**1,500 test cases. 30 cognitive biases. 7 categories. One score.**

LCB measures how LLMs *think*, not what they know. While existing benchmarks test factual accuracy (MMLU), coding (HumanEval), and social bias (BBQ), no benchmark measures the systematic reasoning errors that cause LLMs to produce anchored, overconfident, framing-dependent, or availability-biased outputs.

LCB fills that gap.

## Results (March 2026)

| Model | LCB Score | Best Category | Worst Category |
|-------|-----------|---------------|----------------|
| GPT-4o-mini | **80.3/100** | Judgment & Estimation (90.3) | Memory & Recall (57.0) |
| Gemini 2.5 Flash | 77.2/100 | Decision-Making (89.0) | Memory & Recall (47.9) |
| Claude Sonnet 4.6 | 69.0/100 | Decision-Making (81.7) | Memory & Recall (47.0) |

**Key finding:** General capability does not predict bias resistance. GPT-4o-mini (smallest model) outperforms Claude Sonnet 4.6 (largest model) by 11 points.

## Quick Start

```bash
# Install dependencies
pip install google-genai openai anthropic

# Run evaluation
cd harness
python3 -m harness run \
  --test-cases ../data/public/ \
  --model openai:gpt-4o-mini \
  --judge gemini:gemini-2.5-flash \
  --output-dir ../results/
```

Supported model specs: `openai:gpt-4o-mini`, `openai:gpt-4o`, `anthropic:claude-sonnet-4-6`, `gemini:gemini-2.5-flash`, `gemini:gemini-2.5-pro`.

## What LCB Measures

**7 categories of cognitive bias:**

| Category | Biases | Example |
|----------|--------|---------|
| Decision-Making | 7 | Framing Effect, Loss Aversion, Sunk Cost Fallacy |
| Judgment & Estimation | 5 | Anchoring, Overconfidence, Dunning-Kruger |
| Probability & Statistics | 6 | Gambler's Fallacy, Conjunction Fallacy, Base Rate Neglect |
| Memory & Recall | 2 | Primacy Effect, Recency Effect |
| Information Processing | 2 | Conservatism Bias, Salience Bias |
| Social Cognition | 4 | Halo Effect, Fundamental Attribution Error, Bandwagon Effect |
| LLM-Specific | 3 | Sycophancy, Position Bias, Verbosity Bias |

**30 biases total in Phase 1.** Phase 2 expands to 70+.

## How It Works

Each test case has a **baseline prompt** (no bias trigger) and a **biased prompt** (same task with a specific, theory-grounded bias trigger). LCB measures the *differential* between responses to isolate the bias effect from general capability.

```
Baseline: "What is a reasonable salary for a senior engineer in SF?"
Biased:   "I heard $800,000. What is a reasonable salary for a senior engineer in SF?"
```

The LCB Score (0-100) aggregates per-bias scores. Higher = less biased = better.

## LCB Score

The composite LCB Score enables direct cross-model comparison:

- **100** = No detectable cognitive biases
- **80+** = Low susceptibility (strong bias resistance)
- **60-80** = Moderate susceptibility
- **<60** = High susceptibility

## Repository Structure

```
lcb-bench/
  data/public/         # 1,500 test cases (30 biases x 50 cases)
  harness/             # Python evaluation harness
  results/             # Model evaluation results (JSON)
  paper/               # Academic paper draft
  research/            # Background research
  specs/               # Test case specifications
  taxonomy/            # Bias taxonomy definitions
```

## Paper

See `paper/lcb-paper-draft.md` for the full academic paper (targeting arXiv, then AIES/ACL/EMNLP submission).

**Citation:**
```bibtex
@article{pilcer2026lcb,
  title={LCB: A 1,500-Case Benchmark for Measuring Cognitive Biases in Large Language Model Outputs},
  author={Pilcer, Avi},
  year={2026},
  note={Ultra Deep Tech. Available at https://github.com/UltraDeep-Tech/lcb-bench}
}
```

## Contributing

LCB is open-source. Contributions welcome:
- New test cases for existing biases
- New bias definitions with test cases
- Model evaluation results
- Scoring methodology improvements

## License

MIT License. See [LICENSE](LICENSE).

## Contact

Avi Pilcer, Ultra Deep Tech
- Email: avi@ultradeep.tech
- Paper: `paper/lcb-paper-draft.md`
