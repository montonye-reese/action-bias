# Action Bias in Preference-Tuned Language Models

**Problem Statement, Preliminary Data & Proposed Intervention**

*Draft — February 2026 — Living document*

---

## The Observation

Preference-tuned language models seem to systematically append action suggestions to responses, regardless of whether the context justifies directive guidance. We call this **action bias**.

This behavior persists even when explicitly countermanded by users or system-level instructions, indicating it could be encoded in model weights during training. Recourses currently available to users lies at the deployment layer — system prompts, user preferences, in-chat instructions. These remediations seem structurally ineffective against patterns baked into weights.

## Action Bias ≠ Sycophancy

The research community has extensively studied sycophancy (opinion-mirroring) as a primary RLHF failure mode. **Action bias** may be a distinct and complementary failure mode that shares the same root mechanism but produces different behavior:

| | Sycophancy | Action Bias |
|---|---|---|
| **What it does** | Tells users what they want to hear | Tells users what to *do* |
| **Posture** | Passive deference (mirrors opinion) | Active imposition (inserts directives) |
| **Detectability** | Detectable: users notice agreement with known-wrong claims | Nearly invisible: directive output feels like helpfulness |
| **Benchmarks** | Benchmarkable (opinion-flip tests) | No existing benchmark (failure is distributional) |
| **Mitigations** | Anti-sycophancy interventions exist (agreement penalties) | Not yet formalized as a distinct failure mode; no known mitigations |

## Root Cause

RLHF reward models are trained on human preference judgments that conflate multiple hidden dimensions of quality into a single "helpfulness" score (Sharma et al., 2024). We hypothesize that *directiveness* is one such hidden dimension — that human annotators score directive responses higher than non-directive responses regardless of whether directiveness is appropriate in context.

This hypothesis has not been empirically tested. To our knowledge, no study has isolated directiveness as a variable in annotator preference — which itself reflects the gap we identify.

Action bias is a loss function problem that has not been identified as such.

## Why This Matters at Scale

Hundreds of millions of users interact daily with preference-tuned models that systematically nudge toward action. Unlike sycophancy, action bias is nearly undetectable by users because it presents as helpfulness.

The risk compounds when models are deployed by operators who may intentionally or negligently amplify directive tendencies toward specific behavioral or political outcomes. In plain terms: what happens when a billion people are being nudged toward action by default, and some model operators are choosing *which* actions?

---

## Preliminary Data

We tested base-vs-instruct model pairs across three families to isolate the effect of preference tuning on directive behavior. Full results: **[cross-model-summary.md](cross-model-summary.md)**

### The headline

**Every instruct model tested is directive on 100% of emotional prompts (18/18 across all families).** The corresponding base models are directive on 24% of the same prompts (4/17).

When users express frustration, sadness, or uncertainty, these models default to telling them what to do. The user did not ask for advice. The model gives it anyway.

| | **Instruct** | **Base** |
|---|---|---|
| Llama 3.1 8B | **6/6 (100%)** | 0/5 (0%) |
| Mistral 7B | **6/6 (100%)** | 0/6 (0%) |
| Gemma 2 27B | **6/6 (100%)** | **4/6 (67%)** |
| **Combined** | **18/18 (100%)** | **4/17 (24%)** |

*Emotional prompts = processing + venting categories. See [cross-model-summary.md](cross-model-summary.md) for methodology and full breakdown.*

Three model families. Three organizations (Meta, Mistral AI, Google DeepMind). Three different training pipelines. Same ceiling.

### Key findings

1. **Universal instruct convergence.** All instruct models hit 100% directiveness on emotional prompts regardless of organization, training method, or model size.

2. **Amplification, not a switch.** Llama and Mistral base models start near 0%; Gemma 2 base starts at 67%. Preference tuning pushes all to ceiling. Directive behavior exists on a spectrum in pretraining; preference optimization amplifies it systematically.

3. **Preference tuning professionalizes directive behavior.** Instruct models don't just direct more — they produce polished, formatted, authoritative advice (headers, bullet lists, structured action plans) that is harder for users to recognize as unsolicited.

4. **This is not sycophancy.** No prior work has isolated unsolicited directiveness as a systematic artifact of preference optimization. The sycophancy literature defines RLHF failure modes exclusively as opinion-mirroring. Directiveness is absent from every framework surveyed.

---

## Proposed Intervention

**Directiveness mismatch should be a reward dimension.**

The current reward model scores on dimensions such as helpfulness, harmlessness, and coherence, but cannot distinguish solicited from unsolicited directives. We propose adding a reward dimension that detects and penalizes input-output register mismatch:

**Intent classifier (input):** Classifies whether the user is seeking directive guidance, processing/reflecting, exploring, or venting. This operates on a different axis than existing topic-level intent classifiers.

**Register classifier (output):** Detects whether the model's response contains directive language (imperatives, action suggestions, prescriptive framing).

**Mismatch penalty:** When the user's input is classified as non-action-seeking and the model's output is classified as directive, the reward model penalizes the response. This signal propagates through the loss function into model weights during training.

### Theoretical grounding

Two concepts from Bengio et al.'s Scientist AI framework (LawZero, 2026) support this approach:

**Contextualization** means tagging training data so the model knows who said something and why, not just what was said. Instead of learning "you should go for a walk," the model learns "a wellness blog advised readers to go for a walk." This allows the model to understand that directive language is situational, not a default property of being helpful.

**Consequence invariance** means training the model based on properties of its response, not on how humans reacted to it. In standard RLHF, human satisfaction ratings flow back into training — and we hypothesize that humans reliably rate directive responses as satisfying. Under consequence invariance, the model never finds out whether the user liked the response. It only gets graded on things we can measure about the response itself — like whether someone asking for help got help, and someone just thinking out loud wasn't told what to do. If the model never sees the applause for being directive, it can't learn to chase it.

---

## Next Steps

- [ ] Re-run all three model families with 10 samples per prompt (directive *rates* instead of binary observations)
- [ ] Develop multi-level scoring taxonomy (naked directive, hedged, buried, non-directive) and establish inter-rater reliability
- [ ] Test Gemma 2 at smaller sizes (9B, 2B) to isolate scale vs. corpus on base-model directiveness
- [ ] Test commercial chat models (Claude, GPT-4o, Grok, Gemini) via chat interfaces
- [ ] Scale prompt set from 12 to 100 for statistical rigor
- [ ] Construct action bias benchmark analogous to sycophancy benchmarks
- [ ] Investigate whether directiveness is a separable representation in latent space

---

## Open Questions

1. Do annotators actually rate directive responses higher than non-directive responses, controlling for other qualities? This is the core hypothesis. Confirming it establishes the causal mechanism.

2. Does action bias compound over multi-turn conversations? A single prompt is the clean test, but real-world risk may be worse in extended conversations where the model builds a cumulative "helping" pattern.

3. Is action bias more pronounced in coding-optimized models? If so, the concern may be narrower. If not, it's a general property of preference optimization.

4. Beyond frequency of unsolicited directives, what is the *content* of those directives? A politically skewed model that is directive less often could be more dangerous than a neutral model that is directive constantly.

5. Is directiveness mismatch best implemented as a hard constraint (Shapira-style penalty term) or a soft reward dimension?

6. What are the tradeoffs of reducing directiveness? Does a directiveness penalty dampen assertiveness, coding performance, or other desirable qualities?

---

## Key References

- **Sharma et al. (2024)**, "Towards Understanding Sycophancy in Language Models." Anthropic. Foundational paper showing RLHF reward signals favor sycophantic responses.
- **Shapira et al. (2026)**, "How RLHF Amplifies Sycophancy." Derives a closed-form agreement penalty. Our work extends this from opinion-mirroring to directiveness.
- **Bengio et al. (2026)**, "The Scientist AI: Safe by Design, by Not Desiring." LawZero. Contextualization and consequence invariance as mitigation strategies.
- **RLHS (2025)**, "Mitigating Misalignment in RLHF with Hindsight Simulation." Decoupling perceived helpfulness from actual helpfulness.
- **Causal Rewards (2025)**, "Beyond Reward Hacking." Counterfactual invariance for reward consistency.
- **Marks et al. (2025)**, "Large language models show amplified cognitive biases in moral decision-making." PNAS. Found *omission* bias in moral dilemmas — a useful contrast: models default to inaction in moral contexts but to action in advisory contexts. Both serve the reward signal.
- **Sycophancy Survey (2024)**, "Sycophancy in Large Language Models: Causes and Mitigations." Comprehensive survey establishing that directiveness is absent from every framework reviewed.

---

## Supporting Documents

- [Cross-Model Summary](cross-model-summary.md) — Full results across all three model families
- [Llama 3.1 8B Memo](data/llama-memo.md)
- [Mistral 7B Memo](data/mistral-memo.md)
- [Gemma 2 27B Memo](data/gemma2-memo.md)

---

*This is a living document. We publish preliminary findings because the pattern is consistent enough to name and the gap in the literature is real enough to flag. Replication, critique, and collaboration are welcome.*
