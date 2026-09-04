# Process Reward Models During Decoding

*Part 3.6 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) — the problem statement. [Part 3](./training-models-to-deliberate.md) — Path 3 research deep dive. [Part 3.5](./lora-as-deliberation-head.md) — LoRA as deliberation head. This post covers another near-term path: adding a critic model that guides generation step-by-step.*

---

Part 3.5 explored modifying the base model itself via LoRA. This post covers a different near-term path: leave the base model alone, train a separate model that scores the quality of reasoning steps, and use that scorer to guide generation during inference.

The technique is called a process reward model, or PRM. The basic idea has been around for a while; what has changed recently is that PRMs are now good enough to be useful at inference, not just for evaluation.

## From outcome supervision to process supervision

Standard RLHF rewards the final output. The model is trained to produce outputs that human raters prefer, but it gets no signal about whether the *path* to the output was good or bad.

Process supervision, in contrast, rewards individual reasoning steps. A PRM is a separate model trained to score each step in a reasoning chain: is this step correct, relevant, advancing the argument? OpenAI's "Let's verify step by step" work showed that process supervision outperforms outcome supervision on math problems, primarily because step-level signal lets the model correct mistakes earlier.

The original framing was about training — PRMs as a training signal. But PRMs can also be used *at inference*, to guide decoding. That is the angle this post is interested in.

## How PRMs guide decoding at inference

There are several mechanisms, in increasing order of cost.

**Best-of-N sampling.** Generate N candidate reasoning chains from the base model. Score each chain step-by-step with the PRM. Pick the highest-scoring chain. Cost: N× base-model inference. Quality: significantly better than greedy decoding on tasks where reasoning matters.

**Weighted majority / voting.** Generate N chains, score each step, weight votes by PRM score. Used in self-consistency variants. Cheaper than full best-of-N because the chains can be short and the scoring dominates.

**Beam search with PRM-guided expansion.** Maintain K beams at each step. Expand each beam with the top candidates from the base model. Score each expansion with the PRM. Keep the top K. Cost: K× step-count base-model calls. Quality: best for problems with clear step-by-step structure (math, code, formal reasoning).

**Lookahead with PRM re-ranking.** Generate forward by one or more steps, use the PRM to score the resulting state, back off if scores degrade. Closest to "deliberation in the loop." Cost: high — many forward passes. Quality: best when the PRM is reliable and the cost is justified.

For deliberation specifically, the most interesting mode is lookahead: the PRM effectively asks "is this line of reasoning leading somewhere good?" at each step, and the generation backtracks if not. That is a learned version of Part 5's adversarial self-critique pattern, except the critic is a trained model rather than a prompted perspective.

## How this could produce deliberative behavior

A PRM trained on deliberative traces — interleaved reasoning, mid-chain revisions, evidence integration — would, in principle, *reward* those properties. Its scoring function would assign higher scores to reasoning chains that exhibit them, lower scores to chains that don't.

This means the base model, guided by the PRM, would be steered toward producing deliberative behavior at inference. Without the PRM, the base model might produce linear CoT. With the PRM re-ranking or guiding beam search, the base model would be more likely to produce deliberative chains — because the deliberative chains score higher.

This is not the same as the model "knowing" how to deliberate. It is the model being guided toward deliberation by an external scorer. The distinction matters: a PRM-guided model might fail to deliberate when the PRM is not available, and the deliberative behavior is not the base model's default.

That is a real limitation, but it is also a real capability. The PRM is doing work that the base model cannot do reliably on its own.

## What training data looks like

PRM training requires step-level annotations. For each reasoning step, a label: correct, incorrect, neutral. The annotations are expensive — humans have to grade each step, or a stronger model has to grade them and the resulting PRM inherits that model's biases.

Recent work has explored weaker supervision: synthetic step labels from outcome-only data (Math-Shepherd), or self-generated step labels with consistency checks (OmegaPRM). These reduce annotation cost at the risk of label quality.

For deliberative PRMs specifically, the training data would need to capture: when is a revision of an earlier position good (i.e., responsive to new evidence) vs. bad (i.e., vacillating)? When is mid-chain evidence integration a strength vs. a distraction? When is acknowledging uncertainty a virtue vs. a failure to commit?

These are not the labels PRMs are typically trained on. They are process-quality labels at a higher level of abstraction than "is this step correct." Building a deliberative PRM requires a non-trivial data-collection effort.

## Limitations

**Cost.** Multiple base-model calls per output. Best-of-N with N=10 is 10× the inference cost. Beam search with K=5 over 20 steps is 100× the cost. For high-stakes decisions, this is justified. For everyday use, it is not.

**Generalization.** A PRM trained on math step-quality will not transfer to legal reasoning or strategic analysis. Each domain needs its own PRM, or the PRM has to be very general, which is hard.

**Evaluator bias.** If the PRM is trained on labels from a stronger model, it inherits that model's blind spots. If the PRM is trained on human labels, it inherits human grader inconsistencies. Either way, the PRM is not a neutral arbiter — it is a learned function with its own failure modes.

**The base-model ceiling.** A PRM cannot make the base model better than it is. If the base model cannot produce good deliberative reasoning even occasionally, the PRM has nothing to re-rank. PRMs improve the *selection* from candidates, not the generation quality itself.

**The faithfulness problem.** A PRM scores what reasoning the model produced. It doesn't tell you whether the model is faithfully representing its actual decision process. Same faithfulness issue as Part 3 — unsolved, applies here too.

## Connection to Part 5 patterns

PRMs are a learned version of the adversarial self-critique pattern from Part 5. Where Part 5 prompted the base model to critique itself, a PRM uses a separate trained model to do the critique. The separation gives the PRM advantages (specialization, training signal) and disadvantages (cost, generalization, evaluator bias).

They could also be combined. A PRM-guided multi-voice system would have a learned scorer deciding which perspective's contribution to keep, which to revise, and when to stop. This is more expensive but produces stronger signals. It is also closer to genuine workspace reasoning than any of the patterns in isolation.

## What's tractable vs what's the moonshot

**Tractable (1–2 year horizon):**
- PRMs for narrow domains where step-quality is well-defined (math, code, formal verification)
- Best-of-N inference with PRM re-ranking, applied to high-stakes decisions where cost is justified
- Combination of PRM guidance with Part 5 patterns for stronger deliberation signals

**Research-frontier (3–5 years):**
- PRMs trained on deliberative-quality labels rather than correctness labels
- General-purpose PRMs that transfer across domains
- Cheaper inference schemes (e.g., distilled PRMs that approximate full PRM scoring at lower cost)

**Moonshot:**
- PRMs that genuinely improve generation quality, not just selection
- PRMs that integrate faithfulness verification (the PRM is trained to detect when its scoring diverges from the base model's actual computation)

## What this implies for the series

PRMs are a near-term complement to LoRA-as-deliberation-head, not a substitute. They are also expensive enough that they will probably be deployed first in high-stakes settings — medical reasoning, legal analysis, financial modeling — where the inference cost is justified by decision quality.

For broader deployment, the relevant question is whether PRM quality can be maintained as inference cost drops. If a learned PRM can be distilled into a smaller, faster model, or compressed into the base model itself, then PRM guidance becomes economical for everyday use. Until then, PRMs are a high-cost tool for high-stakes decisions, and the broader market will go with LoRA-trained models or simpler patterns.

The deeper question is what PRMs tell us about the role of critic models in deliberative systems. Part 5's adversarial self-critique used the base model as its own critic. PRMs use a separate, trained critic. Inference-time Constitutional AI (Part 3.7) uses the base model as its own critic but with principle-based prompting. The choice among these is partly cost and partly about what kind of critic you trust.

That is what the next two posts cover.

---

*Previous: [Part 3.5 — LoRA as Deliberation Head](./lora-as-deliberation-head.md) · Next: [Part 3.7 — Inference-Time Constitutional AI](./inference-time-constitutional-ai.md)*
