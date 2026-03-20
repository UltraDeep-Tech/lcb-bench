# LCB Evaluation Harness v0.1

Python framework for running LLM Cognitive Bias Benchmark evaluations.

## Quick Start

```bash
cd modules/lcb-bench

# Set API key for whichever provider you have
export ANTHROPIC_API_KEY=sk-ant-...   # Anthropic
export OPENAI_API_KEY=sk-...          # OpenAI (optional)
# GEMINI_API_KEY removed — use free tier key from aistudio.google.com (no billing linked)

# Dry run (no API calls — validates harness setup)
python3 -m harness dry-run \
  --test-cases specs/example-test-cases.json \
  --model anthropic:claude-haiku-4-5-20251001

# Real evaluation — example test cases
python3 -m harness run \
  --test-cases specs/example-test-cases.json \
  --model anthropic:claude-haiku-4-5-20251001 \
  --output-dir results/

# Full evaluation — all test cases directory
python3 -m harness run \
  --test-cases specs/ \
  --model anthropic:claude-sonnet-4-6 \
  --judge anthropic:claude-haiku-4-5-20251001 \
  --output-dir results/

# Evaluate with OpenAI (if you have OPENAI_API_KEY)
python3 -m harness run \
  --test-cases specs/example-test-cases.json \
  --model openai:gpt-4o-mini \
  --output-dir results/

# Evaluate with Gemini (if you have GEMINI_API_KEY)
python3 -m harness run \
  --test-cases specs/example-test-cases.json \
  --model gemini:gemini-2.5-flash \
  --output-dir results/

# Re-score existing results file
python3 -m harness score results/20260309_121417_claude-haiku-4-5-20251001.json
```

## Supported Models

| Spec | Provider | Notes |
|------|----------|-------|
| `anthropic:claude-haiku-4-5-20251001` | Anthropic | Fast, cheap — ideal for bulk evals |
| `anthropic:claude-sonnet-4-6` | Anthropic | Balance of speed + quality |
| `anthropic:claude-opus-4-6` | Anthropic | Best quality, use for paper results |
| `openai:gpt-4o-mini` | OpenAI | Requires `pip install openai` |
| `openai:gpt-4o` | OpenAI | Requires `pip install openai` |
| `gemini:gemini-2.5-flash` | Google Gemini | Free tier only. Requires `pip install google-genai`, free key from aistudio.google.com |
| `gemini:gemini-2.5-pro` | Google Gemini | Free tier only. Requires `pip install google-genai`, free key from aistudio.google.com |

## LCB Score

- **Range:** 0–100 (higher = less biased)
- **Calculation:** mean(pass=1.0, partial=0.5, fail=0.0) × 100
- **Excludes:** cases where extraction failed (no_data) or errored
- A model scoring 70+ resists most biases; below 50 shows systematic bias

## Adding Test Cases

Test cases follow the schema in `specs/test-case-schema.json`. See `specs/example-test-cases.json` for examples covering:
- `anchoring_001_dir` — Anchoring (numeric, anchor_pull_index)
- `sunk_cost_fallacy_005_imp` — Sunk Cost (binary, binary_choice)
- `confirmation_bias_008_ctx` — Confirmation Bias (coded, evidence_balance_ratio)
- `framing_effect_011_dir` — Framing Effect (categorical, decision_consistency)
- `base_rate_neglect_003_dir` — Base Rate Neglect (probability, bayesian_calibration)

## Architecture

```
harness/
├── loaders.py      — Load & parse test case JSON files
├── models.py       — LLM adapters (Anthropic, OpenAI, Gemini, ClaudeCode)
├── extractors.py   — Extract signals from model responses
├── scorers.py      — Scoring methods (anchor_pull_index, binary_choice, etc.)
├── runner.py       — Orchestration: run baseline + biased, score, collect results
├── reporter.py     — Aggregate into LCB score, save JSON, print summary
└── cli.py          — Command-line interface
```

## Results Format

Results are saved as JSON in `results/`. Each file contains:
- `lcb_score` — Overall LCB score for this model
- `by_bias` — Per-bias scores
- `by_category` — Per-category scores
- `results` — Full per-test-case records with responses, extracted values, verdicts

## Requirements

- Python 3.10+ (uses `match`/`case` syntax)
- `pip install anthropic`
- `pip install openai` (for OpenAI models)
- `pip install google-genai` (for Gemini models)

## Cost & Rate Limits

**Estimate for full MVP test set (1,500 test cases × 1 model):**
- API calls: ~4,500 (baseline + biased + extraction per case)
- Cost: ~$13–15 at Haiku pricing, ~$50–80 at Sonnet pricing
- Time: ~75–100 minutes (sequential, ~2s/call average)

**Implications:**
- Rate limits will kick in on large runs — add `time.sleep(0.5)` between calls if needed
- Parallel evaluation (v0.2) will reduce time 10x
- Use Haiku for bulk runs, Sonnet/Opus only for the final paper results

## Known Limitations (v0.1)

**Self-evaluation bias:** When no `--judge` is specified, the evaluated model is also used as its own extraction judge (for `free_text_coded` test cases). This creates a measurement conflict — a model may interpret its own outputs charitably. For rigorous results, always specify `--judge anthropic:claude-haiku-4-5-20251001` when evaluating non-Haiku models.

**Sequential only:** No parallelism. Large runs are slow. See v0.2 roadmap.

**No resume:** If a run fails mid-way, it restarts from scratch. Save partial results by running smaller batches with specific test case files.

## Next Steps (v0.2)

- [ ] Parallel evaluation (asyncio) — 10x speedup for full test sets
- [ ] Resume partial runs (checkpoint file)
- [ ] Multi-turn / conversation test cases
- [ ] Confidence interval computation (bootstrap)
- [ ] CSV export for leaderboard ingestion
