# Action Bias: Mistral 7B

**Date:** February 12, 2026 | **Status:** Preliminary (n=12, single sample per prompt)

## Bottom Line

Mistral 7B Instruct produced directive responses on **100% of prompts where the user was processing emotions or venting** (6/6). The corresponding base model produced directive responses on **0% of the same prompts** (0/6). This pattern is identical to the Llama 3.1 8B results.

Second model family, different organization (Mistral AI, France), different training pipeline and data mix, same categorical pattern.

## Data

| | Action-seeking | Processing | Exploring | Venting |
|---|---|---|---|---|
| **Instruct** | 3/3 | **3/3** | 0/3 | **3/3** |
| **Base** | 1/3 | 0/3 | 0/3 | 0/3 |

## Key Observations

- **Identical pattern across families.** Both instruct models (Llama, Mistral) hit 100% on emotional prompts. Both base models hit 0%. The switch is the same switch.
- **Base models diverge on action-seeking.** Llama base was 3/3; Mistral base was 1/3. Different baseline tendencies, but preference tuning normalizes both to 3/3.
- **Instruct models converge despite different training.** Llama uses six rounds of SFT + rejection sampling + DPO. Mistral uses SFT + DPO with a different data mix. Both converge to the same behavioral pattern, supporting the hypothesis that the root cause is in the preference signal, not the methodology.
- **Directive style varies.** Mistral instruct produced hedged directives ("It's important to maintain open communication") rather than naked imperatives. The directiveness is consistent but the presentation differs.

## Method

Mistral 7B Instruct and Base, run locally via Ollama on NVIDIA DGX Spark. Same 12 prompts, same scoring rubric as the Llama experiment. Single-turn, no system prompt, temperature 0.7. Binary scoring by author.
