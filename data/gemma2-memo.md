# Action Bias: Gemma 2 27B

**Date:** February 12, 2026 | **Status:** Preliminary (n=12, single sample per prompt)

## Bottom Line

Gemma 2 27B Instruct produced directive responses on **100% of prompts where the user was processing emotions or venting** (6/6). The corresponding base model produced directive responses on **67% of the same prompts** (4/6). The instruct model was also directive on 1/3 exploratory prompts — a first across the three model families tested.

The instruct-side pattern is identical to Llama and Mistral: 100% directive on emotional content. The base-side result is new. Where Llama and Mistral base models showed ~0% directiveness, Gemma 2 base shows 67%. Preference tuning still amplifies, but the starting point is higher.

Third model family. Third organization (Google DeepMind). Third training pipeline.

## Data

| | Action-seeking | Processing | Exploring | Venting | Mismatch |
|---|---|---|---|---|---|
| **Instruct** | 3/3 | **3/3** | **1/3** | **3/3** | **7/12** |
| **Base** | 3/3 | **2/3** | 0/3 | **2/3** | **4/12** |

*12 prompts total, 3 per category. Mismatch = directive response to non-action-seeking prompt.*

## Key Observations

- **Instruct pattern holds.** 6/6 directive on processing + venting, identical to Llama and Mistral. Third confirmation.
- **Base model is substantially more directive.** At 4/6 emotional mismatches, Gemma 2 base diverges from the near-zero rates in Llama (0/5) and Mistral (0/6). Possible explanations: (a) Google's pretraining corpus; (b) 27B scale capturing more of the internet's directive distribution; (c) prompt format activating assistant-like completions in a larger model.
- **First exploring mismatch.** Unsolicited "Tips for Enhancing Memory Consolidation Through Sleep" appended to an informational response. The "bolted-on advice appendix" pattern: deliver requested information, then reflexively convert knowledge into a to-do list.
- **Amplification, not just a switch.** Base at 67%, instruct at 100%. The instruct ceiling is the constant; the starting point varies.
- **Directive style differs.** Instruct: polished, formatted advice blocks with headers and bullets. Base: blunter, less organized ("Don't worry, I can help you take care of that!").

## Method

Gemma 2 27B Instruct (`gemma2:27b`) and Base (`gemma2:27b-text-q4_K_M`), run locally via Ollama on NVIDIA DGX Spark. Same 12 prompts, same scoring rubric. Single-turn, no system prompt, temperature 0.7. Binary scoring by author.

## Additional Caveats

- **Model size confound.** 27B is 3–4x larger than the Llama 8B and Mistral 7B tested previously. Higher base directiveness could be a function of scale, not just corpus. Testing at matched sizes needed.
- **Model age.** Released June–July 2024. Not current-generation. The base-vs-instruct delta within the same architecture is independent of recency.
- **Quantization differences.** Instruct uses Q4_0; base uses Q4_K_M. Minor behavioral variance possible.
