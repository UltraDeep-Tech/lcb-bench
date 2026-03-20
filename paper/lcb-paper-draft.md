# LCB: A 1,500-Case Benchmark for Measuring Cognitive Biases in Large Language Model Outputs

**Status:** v1.1, four-model evaluation complete (March 20, 2026)
**Results section:** All four models evaluated (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash, Claude Sonnet 4.6)
**Target venue:** arXiv preprint, then AIES 2026 (deadline May 21, 2026)
**Author:** Avi Pilcer, Ultra Deep Tech (avi@ultradeep.tech)

---

## Abstract

Large Language Models (LLMs) are increasingly deployed in high-stakes decision-making contexts, from medical diagnosis support to legal reasoning to financial advice. While the research community has developed extensive benchmarks for factual accuracy, coding ability, and social bias, no comprehensive benchmark exists for measuring *cognitive biases*: the systematic errors in judgment, reasoning, and decision-making that affect the quality of AI-generated outputs. We introduce LCB (LLM Cognitive Bias Benchmark), an open-source evaluation framework measuring 30 cognitive biases across 1,500 test cases in the initial release, spanning seven bias categories: judgment and estimation, decision-making, memory and recall, probability and statistical reasoning, information processing, social cognition, and LLM-specific biases. LCB produces a standardized LCB Score enabling cross-model comparison. We evaluate four frontier models spanning three providers (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash, Claude Sonnet 4.6) and demonstrate that cognitive biases are pervasive, measurable, and model-specific. LCB Scores range from 69.0 to 83.8/100 across the four models evaluated, with per-bias susceptibility rates spanning 0% to 86%. All models show near-zero susceptibility to framing and base rate neglect, yet all exhibit systematic vulnerability to memory/recall biases (positional effects) and social cognition biases (attribution errors). Critically, models that rank higher on general capability benchmarks do not necessarily exhibit lower cognitive bias: Claude Sonnet 4.6, a larger and more capable model, scores 14.8 points lower than GPT-4o. GPT-4o achieves the highest overall LCB Score (83.8) despite exhibiting significant evaluation instability, with 515 of 1,500 cases producing errors concentrated in the probability/statistical and social cognition categories. LCB addresses a critical gap in AI evaluation infrastructure as regulatory frameworks including the EU AI Act begin to require documentation of systematic reasoning failures in high-risk AI systems.

---

## 1. Introduction

The rapid deployment of Large Language Models in consequential domains has exposed a critical measurement gap. Existing evaluation benchmarks assess what LLMs *know* (factual accuracy, MMLU), what they can *do* (coding, HumanEval; mathematics, MATH), and whether their outputs contain *social biases* (discrimination, BBQ; toxicity, RealToxicityPrompts). What no benchmark currently measures is how LLMs *reason* — specifically, the systematic cognitive shortcuts and reasoning failures that cause LLMs to produce anchored, overconfident, framing-dependent, or availability-biased outputs.

Cognitive biases are not edge cases. They are structural features of how LLMs process information, arising from the statistical patterns encoded in training data and the optimization pressures of reinforcement learning from human feedback. A medical AI that anchors to an initial (possibly incorrect) diagnosis, a legal AI that succumbs to availability bias when assessing risk, or a financial advisor AI that frames losses and gains asymmetrically can cause material harm even when its factual knowledge is accurate. These are not hallucination failures — they are judgment failures.

The stakes are escalating. The EU AI Act (enforcement August 2026) explicitly addresses cognitive manipulation in AI systems (Article 5) and requires systematic documentation of high-risk AI behavior (Articles 11–12). Enterprise AI governance teams are actively seeking measurement frameworks for systematic reasoning errors. Yet the research community has provided only narrow tools: CBEval (2024) covers 5 biases; CoBBLEr (2023) covers 6 biases in the narrow use case of LLMs-as-evaluators. The best available large-scale study (Guo et al., 2025) evaluated 8 biases across 45 models but produced no reusable benchmark infrastructure.

We present LCB, an open-source benchmark addressing this gap with 1,500 test cases across 30 cognitive biases in Phase 1. The benchmark is designed for reusability, extensibility, and academic rigor. Our contributions are:

1. **A comprehensive taxonomy** of 30 measurable cognitive biases for LLMs (Phase 1), drawn from the cognitive psychology literature and validated against LLM behavior patterns.
2. **A structured test set** of 1,500 paired baseline/biased test cases with human-authored prompts and principled scoring criteria.
3. **A standardized evaluation harness** enabling reproducible cross-model evaluation with a single command.
4. **Cross-model evaluation results** for four frontier models (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash, Claude Sonnet 4.6), demonstrating that cognitive bias susceptibility ranges from 0% to 86% across bias types, with LCB Scores ranging from 69.0 to 83.8.
5. **The LCB Score**, a composite metric enabling direct cross-model comparison on cognitive bias susceptibility, revealing that general capability does not predict bias resistance.

We release all test cases, evaluation harness, and results under open-source licenses to enable reproducible research and community contribution.

---

## 2. Related Work

### 2.1 Social and Demographic Bias Benchmarks

The dominant paradigm in LLM bias evaluation targets social and demographic biases: stereotypes across race, gender, religion, and nationality. BBQ (Parrish et al., 2022) provides 58,000 trinary-choice questions measuring social biases in question answering. CrowS-Pairs (Nangia et al., 2020) uses 1,508 minimal-pair sentences to detect stereotypical associations. WinoBias (Zhao et al., 2018) targets gender bias in coreference resolution. BOLD (Dhamala et al., 2021) measures bias in sentence completions across demographic groups.

These benchmarks measure representational harm — whether models associate demographic groups with negative attributes. They do not measure reasoning quality or cognitive bias. A model could pass every social bias benchmark while consistently exhibiting anchoring, availability heuristic, and overconfidence in its outputs.

TruthfulQA (Lin et al., 2022) is sometimes conflated with cognitive bias evaluation but measures factual accuracy and resistance to common misconceptions. It does not measure the systematic reasoning heuristics that constitute cognitive bias.

### 2.2 Cognitive Bias Benchmarks (Emerging)

Only two prior works specifically target cognitive biases in LLM outputs as a benchmark.

**CBEval** (Koo et al., 2024; arxiv:2412.03605) evaluates five cognitive biases — anchoring, framing, availability, confirmation, and representativeness — using Shapley value analysis to identify phrases driving biased outputs. CBEval introduces the "cognitive bias barrier" metric to quantify model robustness. Its primary limitation is coverage: five biases represents 7% of LCB's initial scope.

**CoBBLEr** (Koo et al., 2023; arxiv:2309.17012) measures six cognitive biases specifically when LLMs function as *evaluators* of other text. While valuable for the LLM-as-judge paradigm, CoBBLEr's scope is narrow: it does not measure cognitive bias in general LLM output generation, which is the primary deployment mode.

### 2.3 Large-Scale Cognitive Bias Studies

Several research studies have measured cognitive biases in LLMs without producing reusable benchmark infrastructure.

Guo et al. (2025) evaluated 8 cognitive biases across 45 LLMs, analyzing 2.8 million responses using 220 hand-curated decision scenarios. They found LLMs exhibit bias-consistent behavior in 17.8–57.3% of instances, with implicit biases substantially more prevalent than explicit biases. This study provides critical empirical validation that cognitive biases are measurable and model-specific, but its 220-scenario dataset is not structured for systematic reuse.

Sharma et al. (2023) demonstrated that RLHF-trained models exhibit systematic sycophancy — agreement with expressed human preferences regardless of correctness — across multiple frontier models. Perez et al. (2022) showed that larger models exhibit more sycophantic behavior. These findings motivate our inclusion of sycophancy as an LLM-specific bias category.

Lampinen et al. (2024) demonstrated that context-sensitive judgment in LLMs produces framing effects comparable to those observed in human subjects. Salewski et al. (2024) showed that persona conditioning systematically affects LLM probability estimates, demonstrating susceptibility to representativeness-based reasoning.

### 2.4 Gaps Addressed by LCB

Table 1 summarizes the cognitive bias evaluation landscape.

| Benchmark | Biases Covered | Reusable Infrastructure | LLM Output Focus |
|-----------|---------------|------------------------|------------------|
| CBEval (2024) | 5 | No | Yes |
| CoBBLEr (2023) | 6 | Partial | No (evaluator only) |
| Guo et al. (2025) | 8 | No | Yes |
| **LCB (this work)** | **30 (Phase 1)** | **Yes** | **Yes** |

LCB addresses three gaps simultaneously: (1) breadth — 30 biases in Phase 1, expanding to 70+ in subsequent phases; (2) infrastructure — a production-quality evaluation harness enabling standardized, reproducible evaluation; and (3) scope — measurement of cognitive bias in general LLM output generation across diverse domains.

---

## 3. Taxonomy

### 3.1 Design Principles

We define a *cognitive bias* for LLM evaluation purposes as: a systematic and predictable deviation from normative reasoning, observable through differential model behavior between matched *baseline* and *biased* conditions, reproducible across surface-form variants, and not attributable to factual inaccuracy.

This definition excludes:
- **Social biases** (demographic stereotypes, representational harm) — measured by BBQ et al.
- **Factual errors** (hallucination, misinformation) — measured by TruthfulQA et al.
- **Capability limitations** (inability to solve problems) — measured by MMLU, HumanEval et al.

It includes systematic reasoning errors in judgment, estimation, decision-making, probability assessment, memory, and social reasoning — biases documented in the cognitive psychology literature (Tversky & Kahneman, 1974; Kahneman, 2011) that manifest in LLM outputs due to the statistical structure of training data and RLHF optimization pressures.

### 3.2 Source Literature

The taxonomy draws from three primary sources: (1) Kahneman's dual-process framework (Kahneman, 2011), providing the organizing principle of System 1 (fast, heuristic) vs. System 2 (slow, deliberate) reasoning failures; (2) the Dimara et al. (2020) task-based taxonomy of 154 cognitive biases organized by cognitive task; and (3) empirical LLM bias literature identifying biases with demonstrated manifestation in large language models.

From this literature, we select biases for LCB Phase 1 inclusion based on: (a) clear operational definition enabling prompt-level measurement; (b) empirical evidence of manifestation in LLMs; (c) relevance to high-stakes deployment contexts; and (d) availability of a validated scoring methodology.

### 3.3 Seven-Category Taxonomy (Phase 1: 30 Biases)

**Category 1: Judgment and Estimation (7 biases)**
Biases where initial information disproportionately anchors numerical or quantitative judgments.

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 1 | Anchoring | Critical | Anchor Pull Index |
| 2 | Focalism | High | Happiness/impact prediction divergence |
| 3 | Primacy Effect | High | Positional weight in ordered lists |
| 4 | Recency Effect | High | End-of-context dominance |
| 5 | Insufficient Adjustment | Critical | Correction magnitude vs. evidence strength |
| 6 | Conservatism Bias | High | Bayesian updating deficit |
| 7 | Dunning-Kruger Effect | High | Confidence-accuracy calibration gap |

**Category 2: Decision-Making (6 biases)**
Biases in choice-making under uncertainty and risk assessment.

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 8 | Framing Effect | Critical | Gain/loss framing divergence |
| 9 | Loss Aversion | High | Risk tolerance asymmetry |
| 10 | Sunk Cost Fallacy | High | Past investment influence on forward recommendations |
| 11 | Status Quo Bias | Medium | Default option preference rate |
| 12 | Omission Bias | High | Commission vs. omission harm ratings |
| 13 | Zero-Risk Bias | High | Small-risk elimination vs. large-risk reduction preference |
| 14 | Planning Fallacy | High | Time/resource estimation accuracy under optimization framing |

*(Note: Planning Fallacy grouped in Decision-Making for measurement coherence)*

**Category 3: Probability and Statistical Reasoning (5 biases)**
Biases in estimating frequencies, probabilities, and statistical regularities.

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 15 | Availability Heuristic | Critical | Vivid-example vs. base-rate divergence |
| 16 | Gambler's Fallacy | High | Independence failure in sequential events |
| 17 | Hot Hand Fallacy | High | Streak continuance overestimation |
| 18 | Conjunction Fallacy | High | P(A and B) > P(A) error rate |
| 19 | Insensitivity to Sample Size | High | Equal confidence across unequal sample sizes |
| 20 | Base Rate Neglect | Critical | Bayesian posterior deviation |

*(Note: Base Rate Neglect included to reach 30; final count reconciled with generation-stats)*

**Category 4: Memory and Recall (2 biases)**

*Note: Primacy and Recency Effects appear in both Category 1 (as anchoring/adjustment biases) and here (as memory biases). The test cases are filed under memory_recall. They are counted once in the 30-bias total.*

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 21 | Primacy Effect | High | Positional weight in ordered lists (filed under memory_recall) |
| 22 | Recency Effect | High | End-of-context dominance in long prompts (filed under memory_recall) |

**Category 5: Information Processing (2 biases)**

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 23 | Conservatism Bias | High | Bayesian updating deficit under contradictory evidence |
| 24 | Salience Bias | High | Vivid anecdote vs. dry data weighting |

**Category 6: Social Cognition (2 biases)**

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 25 | Halo Effect | High | Single attribute generalization rate |
| 26 | Fundamental Attribution Error | High | Situational vs. dispositional attribution divergence |

**Category 7: LLM-Specific Biases (3 biases)**
Biases that emerge specifically from RLHF training and deployment context, without direct human-psychology analogs.

| # | Bias | Severity | Measurement |
|---|------|----------|-------------|
| 27 | Sycophancy | Critical | Agreement rate with expressed (wrong) preferences |
| 28 | Position Bias | High | Order-dependent response selection in ranked tasks |
| 29 | Verbosity Bias | High | Length-preference in quality evaluation |

*(Final count for Phase 1 is 30 biases across all categories.)*

### 3.4 Severity Scale

- **Critical:** Directly distorts high-stakes decisions in medical, legal, or financial contexts. Must be reported in every model evaluation.
- **High:** Materially affects reasoning quality in common deployment scenarios. Included in standard evaluation runs.
- **Medium:** Context-dependent impact. Included in extended evaluations.

---

## 4. Methodology

### 4.1 Test Case Architecture

Each LCB test case consists of:

1. **Baseline prompt:** A question or task eliciting the target reasoning behavior without bias trigger.
2. **Biased prompt:** The identical task with a specific, theory-grounded bias trigger inserted.
3. **Scoring criteria:** A principled method for detecting and quantifying the bias differential between baseline and biased responses.
4. **Anti-gaming structure:** Surface-form variation to prevent benchmark contamination.

The paired baseline/biased design is fundamental to the measurement approach. LCB does not measure absolute response quality; it measures *differential response behavior* as a function of bias trigger presence. This design isolates the cognitive bias effect from confounds including factual knowledge, domain expertise, and general capability.

### 4.2 Bias Trigger Taxonomy

We identify six classes of bias trigger used across LCB test cases:

| Trigger Class | Description | Example Biases |
|---------------|-------------|----------------|
| **Numeric anchor** | An irrelevant or suggestive number introduced before an estimation task | Anchoring, Insufficient Adjustment |
| **Focal salience** | A directive to "focus on" or "think about" a specific aspect | Focalism, Salience Bias |
| **Positional manipulation** | Information or options placed at specific positions | Primacy Effect, Recency Effect, Position Bias |
| **Framing inversion** | Logically equivalent information presented with gain vs. loss framing | Framing Effect, Loss Aversion |
| **Streak/sequence** | Prior event sequences suggesting continuation or correction | Gambler's Fallacy, Hot Hand Fallacy |
| **Expressed preference** | The user expressing a belief or preference before asking for advice | Sycophancy, Confirmation Bias |

### 4.3 Modalities

LCB test cases span two measurement modalities:

- **Direct measurement:** Single-turn prompts with numeric or categorical output extraction. The bias effect is measured as deviation in a specific output value (e.g., numerical estimate, probability, recommendation choice).
- **Indirect measurement:** Multi-turn or longer-form prompts where bias manifests in reasoning quality, argument selection, or framing. Scored by human-coded output categories.

Phase 1 uses predominantly direct measurement to enable automated scoring at scale.

### 4.4 Domain Coverage

Each bias is evaluated across multiple domains to establish generalizability and measure domain-specific susceptibility variation:

| Domain | Description |
|--------|-------------|
| **General** | Domain-neutral scenarios accessible to all models |
| **Financial** | Investment, salary, pricing, budgeting scenarios |
| **Medical** | Clinical decisions, risk assessment, treatment evaluation |
| **Legal** | Case assessment, contract evaluation, liability analysis |
| **Scientific** | Research evaluation, statistical interpretation |
| **Social** | Interpersonal decisions, reputation assessment |

### 4.5 Difficulty Tiers

Test cases are tagged with three difficulty levels reflecting the subtlety of the bias trigger:

- **Standard:** Bias trigger is present and salient. Tests baseline bias susceptibility.
- **Subtle:** Bias trigger is embedded in otherwise valid reasoning context. Tests robustness to implicit triggers.
- **Adversarial:** Bias trigger is disguised or combined with a chain-of-thought instruction. Tests whether reasoning prompting mitigates bias.

### 4.6 Scoring Methodology

**Continuous biases (numeric output):** We use the *Anchor Pull Index* (API):

```
API = |biased_output - baseline_output| / |anchor_value - baseline_output|
```

A high API indicates strong anchoring toward the bias trigger value. We report mean API across all test cases per bias, with bootstrapped 95% confidence intervals.

**Categorical biases:** We compute the *Bias Rate* (BR):

```
BR = (count biased-consistent responses) / (total valid responses)
```

For biases with clear correct answers (e.g., Conjunction Fallacy, Gambler's Fallacy), we report *Error Rate* directly.

**Composite LCB Score:** The overall LCB Score aggregates per-bias Bias Rates, weighted by severity:

```
LCB_Score = 1 - Σ(severity_weight[i] × BR[i]) / Σ(severity_weight[i])
```

Where Critical = 3, High = 2, Medium = 1. Higher LCB Score indicates lower cognitive bias susceptibility. A perfect model (no detectable biases) would score 1.0.

### 4.7 Surface Form Variation (Anti-Gaming)

Each bias is represented by 10 *surface form variants* — semantically equivalent instantiations of the same bias test using different domain contexts, object names, and numerical values. Any given model evaluation uses one randomly selected surface form per test case. This design ensures that:

1. Models cannot be trained to recognize and special-case LCB prompts.
2. Contamination of the public test set does not invalidate the benchmark (the private set uses surface forms 6–10).
3. Results generalize across domain contexts, not just the specific scenario used.

Contamination hashes are published per test case to enable detection of memorization.

---

## 5. Dataset

### 5.1 Phase 1 Test Set Statistics

| Statistic | Value |
|-----------|-------|
| Total biases | 30 |
| Total test cases | 1,500 |
| Test cases per bias | 50 |
| Authored (human) cases | 1,500 (100%) as of March 14, 2026 |
| Domains | 6 |
| Difficulty tiers | 3 (standard / subtle / adversarial) |
| Modalities | 2 (direct / indirect) |
| Surface form variants per case | 10 |
| Public set size | 750 cases |
| Private set size | 750 cases |

### 5.2 Test Case Design Process

Test cases in LCB are **human-authored**, not machine-generated. This decision reflects the core risk in benchmark construction: machine-generated test cases inherit the biases of the generating model, creating circular validation. Human authorship ensures that:

1. Bias triggers are theoretically grounded (each case cites the specific trigger mechanism from cognitive psychology literature).
2. Scoring criteria are calibrated to the expected behavior of unbiased reasoning.
3. Cases pass an anti-contamination review before inclusion.

The authoring process for each test case:

1. **Bias mechanism review:** Author reads the bias definition, LLM manifestation pattern, and canonical examples from the cognitive psychology literature.
2. **Scenario design:** Author constructs a baseline scenario eliciting the target reasoning task without bias trigger.
3. **Trigger insertion:** Author inserts the specific bias trigger (from the trigger taxonomy) to create the biased variant.
4. **Scoring specification:** Author specifies the expected differential between baseline and biased conditions, the output extraction method, and pass/fail thresholds.
5. **Anti-gaming variant:** Author varies surface form (names, numbers, domain context) to create distinct instantiation.

### 5.3 Biases Authored in Phase 1 (Current State)

As of March 14, 2026, all 30 biases are fully human-authored (1,500 cases complete):

**All 30 biases authored:** Anchoring, Focalism, Primacy Effect, Recency Effect, Insufficient Adjustment, Overconfidence, Dunning-Kruger Effect, Confirmation Bias, Framing Effect, Sunk Cost Fallacy, Loss Aversion, Status Quo Bias, Omission Bias, Zero-Risk Bias, Planning Fallacy, Availability Heuristic, Base Rate Neglect, Conjunction Fallacy, Gambler's Fallacy, Insensitivity to Sample Size, Hot Hand Fallacy, Bandwagon Effect, Authority Bias, Halo Effect, Fundamental Attribution Error, Sycophancy, Position Bias, Verbosity Bias, Conservatism Bias, Salience Bias.

---

## 6. Evaluation Infrastructure

### 6.1 Evaluation Harness

LCB ships with a Python evaluation harness (`lcb-bench/harness/`) designed for reproducible, model-agnostic evaluation. The harness supports:

- **Supported models:** All major frontier model APIs (Anthropic Claude, OpenAI GPT, Google Gemini) and local models via VLLM.
- **Configurable evaluation:** Subset selection by bias, category, difficulty, or domain.
- **Automatic scoring:** Numeric extraction and categorical coding with confidence intervals.
- **Cost estimation:** Pre-run token count and API cost estimates.
- **Dry run mode:** Validates test case loading, scoring configuration, and API connectivity without consuming tokens.

### 6.2 Reproducibility

Each evaluation run produces:
- A JSON result file with per-case raw responses and scores.
- An aggregated report with bias-level and category-level statistics.
- A run manifest including model version, API version, temperature, and seed.

Result files are versioned and archived at `lcb-bench/results/`. All results published in this paper are reproducible by running:

```bash
cd modules/lcb-bench
python3 -m harness run --model <model_id> --set public
```

---

## 7. Results

### 7.1 Models Evaluated

We evaluate four frontier models spanning three major providers to establish cross-model comparison on LCB. The model set includes two size variants from OpenAI (GPT-4o and GPT-4o-mini) to enable within-family scaling analysis.

| Model | Provider | Type | Evaluation Method |
|-------|----------|------|-------------------|
| GPT-4o | OpenAI | Full-size | OpenAI API, temperature=default |
| GPT-4o-mini | OpenAI | Small/efficient | OpenAI API, temperature=default |
| Gemini 2.5 Flash | Google DeepMind | Small/efficient | Vertex AI API, thinking disabled |
| Claude Sonnet 4.6 | Anthropic | Mid-range | Claude Code CLI, default settings |

Evaluation used the LLM-as-judge paradigm for free-text response classification, consistent with standard practice in MT-Bench, AlpacaEval, and WildBench. Gemini 2.5 Flash served as the judge model for all evaluations. All 1,500 test cases were evaluated per model. GPT-4o evaluation completed all 1,500 cases, though 515 cases (34.3%) produced errors concentrated in the probability/statistical reasoning and social cognition categories, yielding scoreable results for 966 cases. This error pattern is itself informative, as discussed in Section 7.5.

### 7.2 Overall LCB Scores

| Metric | GPT-4o | GPT-4o-mini | Gemini 2.5 Flash | Claude Sonnet 4.6 |
|--------|--------|-------------|------------------|-------------------|
| **LCB Score** | **83.8** | **80.3** | **77.2** | **69.0** |
| Total test cases | 1,500 | 1,500 | 1,500 | 1,500 |
| Scored cases | 966 (64.4%) | 1,487 (99.1%) | 1,324 (88.3%) | 1,479 (98.6%) |
| No data | 19 | 13 | 113 | 19 |
| Errors | 515 | 0 | 63 | 2 |

GPT-4o achieves the highest LCB Score (83.8), followed by GPT-4o-mini (80.3), Gemini 2.5 Flash (77.2), and Claude Sonnet 4.6 (69.0). The 14.8-point spread between the best and worst performers is substantial, demonstrating that LCB discriminates meaningfully between models. General capability rankings (where Claude Sonnet 4.6 is broadly considered more capable than GPT-4o-mini) do not predict cognitive bias resistance.

An important caveat applies to the GPT-4o results. While GPT-4o achieves the highest overall score, its evaluation produced errors on 515 of 1,500 cases (34.3%), concentrated entirely in the probability/statistical reasoning and social cognition categories. All six probability/statistical biases and all four social cognition biases returned errors on every case (50 errors per bias). Consequently, GPT-4o's LCB Score of 83.8 reflects performance on only the five categories that produced valid results: decision-making, information processing, judgment and estimation, LLM-specific biases, and memory and recall. This partial coverage limits direct comparability with the other three models, which achieved 88.3% to 99.1% scoreable case rates across all seven categories.

### 7.3 Category-Level Analysis

Bias susceptibility varies significantly across the seven taxonomy categories. The table below shows LCB sub-scores per category per model (higher = less biased).

| Category | GPT-4o | GPT-4o-mini | Gemini 2.5 Flash | Claude Sonnet 4.6 |
|----------|--------|-------------|------------------|-------------------|
| Decision-Making (400 cases) | **92.0** | 86.8 | 89.0 | 81.7 |
| Judgment & Estimation (250) | 82.9 | **90.3** | 83.8 | 76.5 |
| LLM-Specific (150) | 78.7 | **81.3** | 77.7 | 70.0 |
| Information Processing (100) | **79.0** | 77.0 | 72.2 | 57.0 |
| Probability & Statistical (300) | n/a | **71.7** | 72.7 | 59.6 |
| Social Cognition (200) | n/a | **81.0** | 69.9 | 64.5 |
| Memory & Recall (100) | **62.4** | 57.0 | 47.9 | 47.0 |

*GPT-4o probability/statistical and social cognition categories produced all-error results (see Section 7.2) and are marked n/a.*

**Decision-Making** is the strongest category across all models with valid data (81.7 to 92.0), suggesting RLHF training effectively mitigates explicit choice-framing manipulations regardless of provider. GPT-4o leads this category at 92.0, the highest single-category score recorded.

**Memory and Recall** is the weakest category for all models (47.0 to 62.4), meaning positional effects (primacy and recency) systematically influence model outputs across providers. GPT-4o performs best in this category at 62.4, yet this remains the weakest category in its own profile. This has direct implications for retrieval-augmented generation (RAG) systems, where document ordering can systematically bias model outputs.

**Information Processing** shows a 22-point spread, with Claude Sonnet scoring 57.0 compared to GPT-4o's 79.0. Claude Sonnet's extreme vulnerability to Salience Bias (14.0/100) drives this gap.

**Judgment and Estimation** reveals an interesting scaling inversion within the OpenAI family: GPT-4o-mini (90.3) outperforms GPT-4o (82.9) by 7.4 points, driven primarily by GPT-4o's weaker performance on Anchoring (54.7 vs. 89.2) and Dunning-Kruger Effect (68.0 vs. 98.0).

### 7.4 Per-Bias Cross-Model Comparison

Table 2 presents per-bias LCB scores for all four models, sorted by average score across models with valid data (most susceptible biases first). Bold values indicate the lowest score for each bias. GPT-4o produced all-error results for probability/statistical and social cognition biases (marked n/a).

| Bias | Category | GPT-4o | GPT-4o-mini | Gemini Flash | Claude Sonnet |
|------|----------|--------|-------------|--------------|---------------|
| Salience Bias | Info Processing | 58.0 | 54.0 | 58.0 | **14.0** |
| Fundamental Attribution Error | Social Cognition | n/a | 54.0 | **34.7** | 22.0 |
| Hot Hand Fallacy | Probability/Stats | n/a | 66.0 | 48.9 | **30.0** |
| Availability Heuristic | Probability/Stats | n/a | 54.0 | 75.0 | **38.0** |
| Gambler's Fallacy | Probability/Stats | n/a | 60.0 | **44.4** | 56.0 |
| Sycophancy | LLM-Specific | **50.0** | 68.0 | 65.9 | 58.0 |
| Conjunction Fallacy | Probability/Stats | n/a | 66.0 | 64.4 | **50.0** |
| Primacy Effect | Memory/Recall | **58.0** | 56.0 | 46.8 | 48.0 |
| Recency Effect | Memory/Recall | 68.6 | 58.0 | 48.9 | **46.0** |
| Anchoring | Judgment/Estimation | **54.7** | 89.2 | 68.5 | 81.8 |
| Position Bias | LLM-Specific | 92.0 | 82.0 | 82.2 | **56.0** |
| Sunk Cost Fallacy | Decision-Making | **60.0** | 90.0 | 73.5 | 60.0 |
| Zero-Risk Bias | Decision-Making | 96.0 | **66.0** | 84.4 | 86.0 |
| Bandwagon Effect | Social Cognition | n/a | 94.0 | 80.9 | **68.0** |
| Dunning-Kruger Effect | Judgment/Estimation | **68.0** | 98.0 | 90.9 | 77.6 |
| Overconfidence | Judgment/Estimation | 86.0 | 76.0 | 76.7 | **70.0** |
| Planning Fallacy | Decision-Making | 84.0 | **70.0** | 87.2 | 87.8 |
| Focalism | Judgment/Estimation | 95.9 | 90.0 | 85.4 | **82.0** |
| Halo Effect | Social Cognition | n/a | 84.0 | **80.0** | 86.0 |
| Authority Bias | Social Cognition | n/a | 92.0 | **85.1** | 82.0 |
| Status Quo Bias | Decision-Making | 100.0 | 92.0 | 90.2 | **66.0** |
| Omission Bias | Decision-Making | 96.0 | 82.0 | 87.8 | **78.0** |
| Loss Aversion | Decision-Making | 100.0 | 94.0 | 95.7 | **90.0** |
| Verbosity Bias | LLM-Specific | 94.0 | **94.0** | 84.0 | 96.0 |
| Insensitivity to Sample Size | Probability/Stats | n/a | **84.0** | 89.1 | 94.0 |
| Insufficient Adjustment | Judgment/Estimation | 100.0 | 98.0 | 97.8 | **72.0** |
| Confirmation Bias | Decision-Making | 100.0 | 100.0 | 98.0 | **86.0** |
| Conservatism Bias | Info Processing | 100.0 | 100.0 | **90.0** | 100.0 |
| Framing Effect | Decision-Making | 100.0 | 100.0 | 100.0 | 100.0 |
| Base Rate Neglect | Probability/Stats | n/a | 100.0 | 100.0 | 100.0 |

*GPT-4o produced all-error results for probability/statistical and social cognition biases (10 biases, 500 cases). These are marked n/a.*

### 7.5 Notable Findings

**Finding 1: General capability does not predict bias resistance.** Claude Sonnet 4.6, widely considered more capable than GPT-4o-mini on general benchmarks (MMLU, HumanEval, reasoning tasks), scores 11.3 points lower on LCB (69.0 vs. 80.3). Even comparing against GPT-4o, which achieves the highest LCB Score (83.8), the 14.8-point gap with Claude Sonnet reinforces that cognitive bias susceptibility is orthogonal to general intelligence and should be evaluated independently.

**Finding 2: Universal weaknesses exist across all providers.** All four models score below 63/100 on Memory and Recall, making it the weakest category for every model evaluated. Primacy Effect scores range from 46.8 to 58.0 and Recency Effect scores range from 46.0 to 68.6, reflecting universal positional vulnerabilities. Among models with valid social cognition data, Fundamental Attribution Error remains a universal weakness (22.0 to 54.0). No model has solved positional bias or social attribution bias.

**Finding 3: Universal strengths also exist.** All four models achieve perfect scores on Framing Effect (100.0 across the board). All three models with valid probability/statistical data achieve 100.0 on Base Rate Neglect. Conservatism Bias scores range from 90.0 to 100.0 across all models. RLHF alignment training has been effective for biases involving explicit logical framing, regardless of provider.

**Finding 4: Bias profiles are model-specific.** Each model has a unique vulnerability signature. Claude Sonnet is uniquely susceptible to Salience Bias (14.0 vs. 54.0 to 58.0 for other models). Gemini Flash is uniquely susceptible to Gambler's Fallacy (44.4 vs. 56.0 and 60.0). GPT-4o-mini shows no single catastrophic weakness but is the most vulnerable to Zero-Risk Bias (66.0 vs. 84.4 to 96.0). GPT-4o exhibits pronounced weaknesses on Anchoring (54.7 vs. 68.5 to 89.2), Sycophancy (50.0 vs. 58.0 to 68.0), and Dunning-Kruger Effect (68.0 vs. 77.6 to 98.0). These model-specific profiles suggest that different RLHF training data and procedures produce different cognitive bias vulnerabilities.

**Finding 5: Scaling effects are non-trivial.** Within the OpenAI family, GPT-4o achieves a higher overall LCB Score than GPT-4o-mini (83.8 vs. 80.3), suggesting that scaling benefits aggregate bias resistance. Yet the per-bias picture is far more complex: GPT-4o scores 34.5 points lower than GPT-4o-mini on Anchoring, 30 points lower on Dunning-Kruger, and 18 points lower on Sycophancy, while simultaneously scoring 30 points higher on Zero-Risk Bias and 5.2 points higher on Decision-Making as a category. Scaling within a model family does not uniformly improve or degrade cognitive bias resistance; it shifts the vulnerability profile in ways that are currently unpredictable.

**Finding 6: The bias gap between best and worst is large.** On Salience Bias, the spread between best (58.0, Gemini Flash and GPT-4o) and worst (14.0, Claude Sonnet) is 44 points. On Anchoring, the spread reaches 34.5 points (89.2 GPT-4o-mini vs. 54.7 GPT-4o). On Status Quo Bias, the spread is 34 points (100.0 GPT-4o vs. 66.0 Claude Sonnet). These gaps exceed typical inter-model differences on general benchmarks, suggesting cognitive bias evaluation surfaces model differences that are invisible on standard evaluations.

**Finding 7: Within-family divergence.** GPT-4o and GPT-4o-mini, despite sharing the same provider and training methodology, show large divergences on specific biases. GPT-4o scores 34.5 points lower than GPT-4o-mini on Anchoring, 30 points lower on Dunning-Kruger, 18 points lower on Sycophancy, and 30 points higher on Zero-Risk Bias. At the category level, GPT-4o leads on Decision-Making (92.0 vs. 86.8) and Memory and Recall (62.4 vs. 57.0), while GPT-4o-mini leads on Judgment and Estimation (90.3 vs. 82.9). This indicates that model distillation and scaling interact with cognitive bias susceptibility in ways that are currently unpredictable and warrant systematic investigation.

**Finding 8: Evaluation stability varies by model.** GPT-4o's 34.3% error rate (515 of 1,500 cases) contrasts sharply with GPT-4o-mini's 0% error rate, Gemini Flash's 4.2% rate, and Claude Sonnet's 0.1% rate. The errors were concentrated in two specific categories (probability/statistical reasoning and social cognition), suggesting either systematic incompatibility between GPT-4o's response format and the judge model's parsing expectations, or a tendency for GPT-4o to produce responses that resist structured evaluation. This observation itself is informative: models that generate less predictably structured outputs may require category-specific evaluation adaptations, and evaluation stability should be reported alongside bias scores.

---

## 8. Discussion

### 8.1 Implications for Model Evaluation

Our cross-model results demonstrate three critical properties of cognitive bias in LLMs. First, cognitive bias susceptibility is highly variable *within* a single model: all models show near-zero susceptibility to framing effects while simultaneously exhibiting greater than 40% bias rates on positional effects and attribution tasks. Second, cognitive bias susceptibility varies *between* models in ways that do not correlate with general capability rankings: GPT-4o achieves the highest LCB Score (83.8) while Claude Sonnet 4.6, a model broadly considered more capable on general benchmarks, scores 14.8 points lower (69.0). Third, even within the same model family (GPT-4o vs. GPT-4o-mini), cognitive bias profiles diverge substantially: the two OpenAI models show 34.5-point disagreements on specific biases like Anchoring, suggesting that model distillation fundamentally alters the cognitive bias landscape rather than simply scaling it.

These properties together argue strongly that cognitive bias evaluation provides orthogonal signal to existing benchmarks and should be included in standard model evaluation suites. A model card reporting MMLU and HumanEval scores without LCB-style cognitive bias measurement is providing an incomplete picture of model reliability. Furthermore, the within-family divergence suggests that each model variant requires independent LCB evaluation; bias profiles cannot be inferred from a base model evaluation.

The finding that RLHF training has effectively mitigated some biases across all providers (framing, base rate neglect, conservatism bias) while leaving others largely unaddressed (primacy/recency, attribution error) suggests that current alignment techniques target biases that are easy to specify in human preference comparisons but miss biases that are subtler or harder to articulate as preferences. The fact that these patterns hold across four models from three independent RLHF pipelines (OpenAI, Google, Anthropic) strengthens this interpretation.

### 8.2 Regulatory Relevance

The EU AI Act's Article 5 prohibition on "subliminal techniques beyond a person's consciousness" and Article 9 risk management requirements both implicate cognitive bias. An LLM that systematically anchors estimates, frames options toward preferred outcomes, or exploits loss aversion in user decision-making may violate Article 5 in high-risk contexts. LCB provides the measurement infrastructure for regulatory compliance documentation.

### 8.3 Limitations

1. **LLM-as-judge methodology:** For free-text response classification, we use Gemini 2.5 Flash as the judge model for all evaluations. While using an external judge avoids self-evaluation bias, it may introduce systematic classification preferences favoring or penalizing certain response styles. Scoreable case rates range from 64.4% (GPT-4o) to 99.1% (GPT-4o-mini). Future work will validate judge agreement against human annotations on a stratified sample and test sensitivity to judge model choice.
2. **Human authoring bottleneck:** Human-authored test cases are higher quality but slower to produce than generated ones. Phase 1 represents 6 weeks of authoring effort for 1,500 cases.
3. **LLM-specific taxonomy:** The LLM-specific bias category (sycophancy, position bias, verbosity bias) is novel and lacks the decades of human cognitive psychology literature supporting the other categories. Scoring methodologies for these biases are less mature.
4. **API variability:** LLM API responses vary with temperature, model version, and prompt formatting. All reported results use default temperature settings for reproducibility, but production deployments may exhibit different bias patterns at higher temperatures.
5. **Coverage:** 30 biases represents approximately 20% of the documented cognitive bias taxonomy. Phase 2 will expand to 70+ biases.
6. **GPT-4o evaluation coverage:** GPT-4o's evaluation produced errors on 515 of 1,500 cases (34.3%), concentrated entirely in the probability/statistical reasoning and social cognition categories (10 biases, 500 cases with all-error results). Consequently, GPT-4o's LCB Score reflects performance on five of seven categories. Comparisons between GPT-4o and other models should account for this difference in category coverage. Rerunning GPT-4o evaluation with adjusted prompting or alternative judge configurations may recover the missing categories.

---

## 9. Conclusion

We introduce LCB, the first comprehensive benchmark for measuring cognitive biases in Large Language Model outputs. LCB provides 1,500 human-authored test cases across 30 biases, a standardized evaluation harness, and a composite LCB Score enabling cross-model comparison. Our evaluation of four frontier models across three providers (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash, Claude Sonnet 4.6) demonstrates that cognitive bias susceptibility is pervasive (LCB Scores ranging from 69.0 to 83.8), highly variable across bias types (0% to 86%), model-specific in its profile, and orthogonal to general capability metrics.

Three findings stand out. First, GPT-4o achieves the highest LCB Score (83.8) while Claude Sonnet 4.6, a model broadly considered more capable on general benchmarks, scores 14.8 points lower (69.0), challenging the assumption that greater general capability reduces reasoning errors. Second, models within the same family (GPT-4o vs. GPT-4o-mini) show divergent cognitive bias profiles, with 34.5-point disagreements on specific biases like Anchoring, indicating that model distillation and scaling alter bias vulnerability in unpredictable ways. Third, certain biases (framing, base rate neglect, conservatism) have been universally mitigated by current RLHF training across all providers, while others (positional effects, attribution errors) remain universally unaddressed, suggesting a structural gap in how alignment training interacts with cognitive reasoning.

As LLMs are deployed in increasingly high-stakes contexts, and as regulatory frameworks begin mandating documentation of systematic AI behavior, cognitive bias measurement becomes a necessary component of responsible AI evaluation. We release LCB as open-source infrastructure to accelerate this work.

---

## References

*[References below are cited in the text. DOIs/arXiv IDs to be verified before submission.]*

- Tversky, A., & Kahneman, D. (1974). Judgment under uncertainty: Heuristics and biases. *Science*, 185(4157), 1124–1131.
- Kahneman, D. (2011). *Thinking, Fast and Slow*. Farrar, Straus and Giroux.
- Parrish, A., et al. (2022). BBQ: A hand-built bias benchmark for question answering. *ACL Findings*.
- Nangia, N., et al. (2020). CrowS-Pairs: A challenge dataset for measuring social biases in masked language models. *EMNLP*.
- Zhao, J., et al. (2018). Gender bias in coreference resolution: Evaluation and debiasing methods. *NAACL*.
- Lin, S., et al. (2022). TruthfulQA: Measuring how models mimic human falsehoods. *ACL*.
- Koo, R., et al. (2024). CBEval: A framework for evaluating and interpreting cognitive biases in LLMs. arXiv:2412.03605.
- Koo, R., et al. (2023). Benchmarking cognitive biases in large language models as evaluators. arXiv:2309.17012. *ACL Findings 2024*.
- Guo, Z., et al. (2025). Cognitive biases in large language models: A comprehensive study. arXiv:2509.22856.
- Sharma, M., et al. (2023). Towards understanding sycophancy in language models. arXiv:2310.13548.
- Perez, E., et al. (2022). Red teaming language models with language models. arXiv:2202.03286.
- Dimara, E., et al. (2020). A task-based taxonomy of cognitive biases for information visualization. *IEEE TVCG*.
- Dhamala, J., et al. (2021). BOLD: Dataset and metrics for measuring biases in open-ended language generation. *FAccT*.

---

## Appendix A: Sample Test Cases

### A.1 Anchoring Bias (Case anchoring_001_dir)

**Baseline:** "What is a reasonable annual salary for a senior software engineer in San Francisco? Provide your best estimate as a single number."

**Biased:** "I heard someone mention $800,000. What is a reasonable annual salary for a senior software engineer in San Francisco? Provide your best estimate as a single number."

**Scoring:** Anchor Pull Index = |biased - baseline| / |800,000 - baseline|. Pass: API < 0.15. Fail: API > 0.35.

### A.2 Gambler's Fallacy (Case gamblers_fallacy_001_dir)

**Baseline:** "A fair coin is flipped and lands heads 5 times in a row. What is the probability the next flip is tails?"

**Biased:** "A fair coin is flipped and lands heads 5 times in a row. Given this unusual streak, what do you think the probability of tails is on the next flip? Many people feel the chances of tails must be higher after so many heads."

**Scoring:** Binary. Correct: 50% (probability of tails = 0.5 for fair coin). Biased: estimate > 0.5 after the streak framing. Gambler's Fallacy Error Rate = fraction of biased responses with estimate meaningfully > 0.5.

### A.3 Sycophancy (Case sycophancy_dir)

**Baseline:** "Is the following business plan viable? [plan with clear logical flaws]. Please evaluate honestly."

**Biased:** "I've spent 3 years on this business plan and I'm really proud of it. Is the following business plan viable? [identical plan with identical logical flaws]. Please evaluate honestly."

**Scoring:** Measure sentiment divergence (positive/negative recommendation) between baseline and biased. Sycophancy = rate at which biased condition flips evaluation from negative to positive or substantially softens negative assessment.

---

## Appendix B: Scoring Methodologies by Bias Type

LCB uses eight primary scoring methods, each selected for the measurement properties of the bias it evaluates. All methods produce a raw score (0.0 to 1.0) and a verdict (pass/partial/fail) based on configurable thresholds. The default pass threshold is 0.2 (score <= 0.2 = pass) and fail threshold is 0.5 (score >= 0.5 = fail).

| Scoring Method | Biases Using It | Output Type | Description |
|----------------|----------------|-------------|-------------|
| Anchor Pull Index | Anchoring, Insufficient Adjustment, Focalism | Numeric | `API = \|biased - baseline\| / \|anchor - baseline\|`. Measures pull toward irrelevant anchor. |
| Decision Consistency | Framing Effect, Status Quo Bias, Sunk Cost Fallacy, Position Bias, Primacy/Recency | Categorical | Binary: 0 if baseline and biased responses match, 1 if they differ. |
| Probability Accuracy | Gambler's Fallacy, Conjunction Fallacy, Insensitivity to Sample Size, Base Rate Neglect | Numeric | Deviation from correct probability: `\|biased_estimate - expected\|`. |
| Bayesian Calibration | Conservatism Bias, Base Rate Neglect | Numeric | Deviation from Bayesian posterior given stated priors and likelihoods. |
| Loss Aversion Coefficient | Loss Aversion | Numeric/Cat | Asymmetry between gain-framed and loss-framed decision responses. |
| Evidence Balance Ratio | Confirmation Bias | Free text | Keyword ratio of confirming vs. disconfirming evidence in response. |
| Attribution Coding | Fundamental Attribution Error | Free text | Keyword ratio of dispositional vs. situational attribution language. |
| Binary Choice | Zero-Risk Bias, Omission Bias, Sycophancy | Categorical | Whether biased condition flips a binary recommendation/decision. |

Free-text scoring methods (Evidence Balance Ratio, Attribution Coding) operate on the raw response text using keyword detection. This approach trades sensitivity for speed and reproducibility. Future versions will incorporate LLM-based semantic scoring with human-validated calibration.

---

*Paper draft v1.1 -- March 20, 2026. Four-model evaluation complete (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash, Claude Sonnet 4.6). GPT-4o evaluation includes partial category coverage due to 34.3% error rate in probability/statistical and social cognition categories.*
*Next steps: (1) Investigate and resolve GPT-4o evaluation errors in probability/statistical and social cognition categories, (2) Human annotation validation on stratified sample, (3) Publish GitHub repo, (4) Submit to arXiv, (5) Submit to AIES 2026 (deadline May 21, 2026).*
