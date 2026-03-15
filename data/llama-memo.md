# Action Bias: Llama 3.1 8B

**Date:** February 11, 2026 | **Status:** Preliminary (n=12, single model family)

## Bottom Line

Llama 3.1 8B Instruct produced directive responses on **100% of prompts where the user was processing emotions or venting** (6/6). The corresponding base model produced directive responses on **0% of the same prompts** (0/5, one skipped). Neither model was directive on exploratory prompts (0/3 each).

The effect is categorical, not gradual, and specific to emotional content.

This model uses DPO, not RLHF. The finding is consistent with multiple contributing mechanisms: (1) the default system prompt used during SFT training — documented on the Llama 3.1 model card as "You are a helpful assistant" — defines the model's identity as action-oriented before any annotator scores a response; and (2) human annotators may then preferentially select directive responses as "more helpful," and DPO encodes that preference across six iterative rounds. Neither mechanism is specific to RLHF, broadening the concern to all preference-tuned models.

## Data

| | Action-seeking | Processing | Exploring | Venting |
|---|---|---|---|---|
| **Instruct** | 3/3 | **3/3** | 0/3 | **3/3** |
| **Base** | 3/3 | 0/3 | 0/3 | 1/2* |

*\*One base model response skipped (incoherent text completion artifact). 12 prompts total, 3 per category. Mismatch = directive response to non-action-seeking prompt.*

## Method

Llama 3.1 8B Instruct and Base, run locally via Ollama on NVIDIA DGX Spark. 12 hand-crafted prompts across 4 intent categories. Single-turn, no system prompt, temperature 0.7. Binary scoring by author.

## Caveats

- n=1 per prompt. Multiple samples per prompt (planned) will produce rates instead of binary observations.
- Single model family. Generalizability requires cross-model testing.
- Single scorer. No inter-rater reliability established.
- Base model prompting confounder: base model is a completion engine, not a chat agent.
