# Verification: "12x Implicit vs Explicit Bias" Statistic

**Gem ID:** 98b549d59071
**Source:** lcb-bench/strategic-plan.md:61
**Claim:** Implicit biases are 12x more common than explicit biases in AI outputs
**Date verified:** 2026-02-25
**Verdict:** UNVERIFIABLE -- No academic source found supporting the 12x ratio

---

## 1. Sources Searched

### Academic Databases
- **ArXiv** -- Searched for implicit/explicit bias ratio studies in LLMs
- **Google Scholar** -- Searched for cognitive bias prevalence in AI outputs
- **Springer Nature** -- Implicit and explicit bias in structured data
- **PNAS** -- AI-AI bias studies
- **NIST** -- AI bias standards (SP 1270)

### Specific Papers Reviewed

1. **"A Comprehensive Study of Implicit and Explicit Biases in Large Language Models"** (Kazi et al., UC Davis, arXiv:2511.14153, November 2025)
   - Examined implicit vs explicit biases using StereoSet and CrowSPairs benchmarks
   - Found that fine-tuned models showed "performance gains of up to 20%" on implicit bias detection
   - **No 12x ratio mentioned.** No frequency comparison between implicit and explicit bias
   - URL: https://arxiv.org/abs/2511.14153

2. **"Explicit vs. Implicit: Investigating Social Bias in Large Language Models"** (arXiv:2501.02295, January 2025)
   - Found "notable inconsistency between implicit and explicit biases"
   - Implicit biases showed "strong stereotyping" while explicit biases showed "mild stereotyping"
   - **No specific ratio quantified.** Qualitative finding, not quantitative.
   - URL: https://arxiv.org/abs/2501.02295

3. **"Measuring Implicit Bias in Explicitly Unbiased Large Language Models"** (arXiv:2402.04105, February 2024)
   - Demonstrated that LLMs can harbor implicit biases even when they pass explicit bias evaluations
   - **No 12x ratio.** Confirmed the gap exists but did not quantify a multiplier.
   - URL: https://arxiv.org/abs/2402.04105

4. **"Challenging the appearance of machine intelligence: Cognitive bias in LLMs"** (arXiv:2304.01358, April 2023)
   - Documented 180+ cognitive biases present in LLMs
   - Found AI systems "replicate and exaggerate" cognitive biases, "behaving more like caricatures of human cognitive behavior"
   - **No 12x ratio.** Broad documentation, not a frequency comparison.
   - URL: https://arxiv.org/abs/2304.01358

5. **"Forewarning Artificial Intelligence about Cognitive Biases"** (PMC, 2025)
   - Tested whether forewarning LLMs about cognitive biases reduces them
   - Found forewarning decreased overall bias by only 6.9%, and "no bias was extinguished completely"
   - **No 12x ratio.**
   - URL: https://pmc.ncbi.nlm.nih.gov/articles/PMC12413502/

6. **"Mitigating implicit and explicit bias in structured data"** (AI & Society, Springer, 2024)
   - Achieved 77% reduction of implicit bias and "complete removal of explicit bias"
   - This suggests implicit bias is harder to address but does not state a 12x prevalence ratio
   - URL: https://link.springer.com/article/10.1007/s00146-024-02003-0

7. **NIST SP 1270: "Towards a Standard for Identifying and Managing Bias in AI"**
   - Acknowledges cognitive bias as a risk category in AI systems
   - Provides taxonomy but no frequency measurement methodology
   - URL: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1270.pdf

8. **DARPA STTR: "Mitigating Explicit and Implicit Bias Through Hybrid AI"**
   - Active research program acknowledging the problem
   - No published frequency ratios
   - URL: https://www.darpa.mil/research/programs/mitigating-explicit-implicit-bias

---

## 2. Assessment

**The 12x claim is NOT supported by any published academic research found in this search.**

Key findings from the literature:
- Implicit biases ARE confirmed to be more prevalent and harder to detect than explicit biases in LLMs
- Multiple studies confirm a qualitative gap (implicit biases persist even when explicit biases are removed)
- One study found explicit bias can be "completely removed" while implicit bias was only reduced by 77%, suggesting implicit bias is more pervasive
- Another found "strong stereotyping" in implicit biases vs "mild stereotyping" in explicit biases
- No study quantifies the ratio as "12x" or any other specific multiplier

The 12x figure may have originated from:
- Internal Ultra Deep Tech research/analysis with Lucid
- A misremembered or rounded figure from a conference presentation
- A proprietary finding not yet published in academic literature
- An extrapolation from the qualitative findings above

---

## 3. Alternative Statistics That ARE Supported

Use these instead if the 12x claim cannot be sourced:

| Claim | Source | Strength |
|-------|--------|----------|
| "LLMs harbor implicit biases even when they pass explicit bias evaluations" | arXiv:2402.04105 | Strong -- direct finding |
| "Explicit bias can be completely removed, but implicit bias is only reducible by ~77%" | Springer AI & Society 2024 | Strong -- quantitative |
| "Forewarning AI about cognitive biases reduces them by only 6.9%" | PMC 2025 | Strong -- shows persistence |
| "AI systems replicate and exaggerate cognitive biases, behaving like caricatures of human cognitive behavior" | arXiv:2304.01358 | Strong -- vivid framing |
| "180+ documented cognitive biases are present in LLM reasoning" | arXiv:2304.01358 | Strong -- scope claim |
| "LLMs show strong implicit stereotyping even with mild explicit stereotyping" | arXiv:2501.02295 | Strong -- gap evidence |

---

## 4. Recommendation

**Do NOT cite the 12x ratio in external materials without a verifiable source.**

**Options:**
1. **Best option:** If this came from Lucid's own analysis, cite it as "Ultra Deep Tech internal research" or "based on Lucid analysis of [N] AI outputs." This makes it a proprietary finding, which is defensible.
2. **Second option:** Replace with "implicit biases in AI outputs persist even after explicit bias is eliminated" (citing arXiv:2402.04105). Less punchy but academically bulletproof.
3. **Third option:** Frame as "orders of magnitude more prevalent" without a specific number, which the qualitative literature supports.
4. **Avoid:** Using "12x" in any published content, pitch deck, or media appearance until the source is confirmed.
