# Inference-Time Constitutional AI

*Part 3.7 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) — the problem statement. [Part 3](./training-models-to-deliberate.md) — Path 3 deep dive. [Part 3.5](./lora-as-deliberation-head.md) — modifying the base model. [Part 3.6](./process-reward-models.md) — adding a critic model. This post covers using the base model as its own critic, guided by principles.*

---

Part 3.6 added a separate trained critic (a process reward model) that guides the base model during decoding. This post asks what happens when the critic is the base model itself, prompted to evaluate against a set of principles. No additional training. No separate model. Just structured self-critique at inference time.

This is the inference-time variant of Constitutional AI, and it is one of the cheapest near-term paths to more deliberative behavior.

## Constitutional AI in training vs at inference

Constitutional AI, in its original Anthropic formulation, trains the model to critique and revise its own outputs against a set of principles. The training signal comes from the model's own critiques, filtered through the principles. The result is a model that has internalized the principles and applies them by default.

The inference-time variant skips the training. The model is given the principles at inference. When producing an output, the model is asked to first generate, then critique against the principles, then revise. Multiple iterations. The principle-following behavior emerges from the prompting rather than from the weights.

The difference is cost. Training-time constitutional AI requires the full training pipeline — labeled critiques, RL fine-tuning, evaluation. Inference-time constitutional AI requires only a longer prompt and multiple inference calls per output. The capability ceiling is lower, but the deployment cost is dramatically lower.

## How this could produce deliberative behavior

The principles, if chosen well, can encode deliberation requirements. Examples:

- *"Before committing to a position, consider the strongest objection to it."*
- *"If evidence emerges during your reasoning that contradicts an earlier claim, revise the earlier claim."*
- *"Acknowledge uncertainty explicitly when the evidence does not support a confident answer."*
- *"Cite the basis for each load-bearing claim."*

When the model is asked to critique its draft against these principles and then revise, the resulting output tends to be more deliberative — more revision, more acknowledgment of counter-arguments, more explicit uncertainty handling. The deliberation is forced by the principles.

This is structurally similar to Part 5's adversarial self-critique pattern, but with the critique framed as compliance-with-principles rather than as a free-form "find the strongest objection." The framing matters: principle-based critiques are easier for the model to apply consistently, because the criteria are explicit.

A more sophisticated version uses multiple principles at different stages: a "consider alternatives" principle before the draft, an "evaluate evidence" principle during revision, an "acknowledge limits" principle in the final synthesis. This approximates the structure of the Part 5 blackboard pattern, with principle-invocation standing in for typed slots.

## What it looks like in practice

A typical inference-time constitutional AI loop:

```python
def constitutional_generate(task, principles, max_iterations=3):
    draft = llm(f"""Task: {task}

Produce a first draft response.""")
    for _ in range(max_iterations):
        critique = llm(f"""Task: {task}

Draft:
{draft}

Principles:
{chr(10).join(f'- {p}' for p in principles)}

Critique the draft against the principles.
For each principle violated, identify the violation
and propose a revision. If no principles are
violated, say 'NO VIOLATIONS'.""")
        if "NO VIOLATIONS" in critique:
            break
        draft = llm(f"""Task: {task}

Previous draft:
{draft}

Critique:
{critique}

Revise the draft to address the critique.""")
    return draft
```

Cost: 2N+1 inference calls per output, where N is the number of revision iterations. For N=3, that's 7 calls. At current API pricing, this is roughly 7× the cost of a single-shot response. Not cheap, but much cheaper than a process reward model setup.

## Limitations

**Same-model bias.** The model is critiquing its own work. There is no guarantee it will find flaws it would not have produced in the first place. Constitutional AI at training time partially addresses this by training the critique to be more incisive; at inference time, you get whatever critique fidelity the base model provides. For most models, this means the critique tends to be surface-level — flagging obvious issues, missing subtle ones.

**Principles can be gamed.** If the principles are vague enough, the model can satisfy them superficially without genuine engagement. "Consider alternatives" can be satisfied by mentioning that alternatives exist, without actually weighing them. "Acknowledge uncertainty" can be satisfied by adding hedging language without changing the underlying claim. The principles have to be specific enough that shallow compliance is detectable, which usually means longer principles that cost more context.

**Cost.** Multiple inference calls per output. At scale, this adds up. For high-stakes decisions, the cost is justified. For everyday use, it isn't.

**Latency.** Sequential critique-revise iterations don't parallelize cleanly. End-to-end latency is roughly N× single-call latency. Real-time use cases may not tolerate this.

**The principle-design problem.** Choosing principles is itself an untracked task. Good principles are specific, falsifiable, and capture genuine deliberation requirements. Bad principles are vague, easy to game, or impose a particular view of what "good reasoning" looks like. There is no canonical set, and the choice of principles is doing real work that the system doesn't expose.

**Faithfulness, again.** The model's critique is generated text, not a faithful report of its actual evaluation process. A model can produce a critique that looks like genuine engagement without actually engaging. This is the same faithfulness problem as Part 3 — inference-time constitutional AI doesn't solve it and may even obscure it.

## Connection to Part 5 patterns

Inference-time constitutional AI is structurally similar to two of the Part 5 patterns:

- **Adversarial self-critique** — same shape, different framing. Self-critique asks for the strongest objection; constitutional AI asks for principle violations. The principles are essentially pre-articulated objections.
- **Multi-voice CoT** — the "principle invoker" can be treated as one of several perspectives in a multi-voice deliberation. The principles are the perspective's contribution.

A combined system would use Part 5's multi-voice structure for the deliberation itself, then use inference-time constitutional principles to evaluate the resulting synthesis. This is more expensive but produces stronger signals than either approach in isolation.

## Connection to PRMs

Inference-time constitutional AI is the cheaper, weaker sibling of Part 3.6's process reward models. PRMs are trained to score reasoning quality; inference-time constitutional AI uses prompted self-critique to do the same job, without training. The trade-off is straightforward:

- **PRMs**: better quality critique, more expensive (training + inference), generalize less well
- **Inference-time constitutional AI**: weaker critique, cheaper (inference only), more flexible (principles can be changed without retraining)

For deployment, the choice depends on volume and stakes. High volume, low stakes: inference-time constitutional AI. Low volume, high stakes: PRMs (or both).

## What's tractable vs what's the moonshot

| Horizon | Direction |
|---|---|
| Tractable (now) | Inference-time constitutional loops for high-stakes decisions where 5–10× inference cost is justified |
| Tractable (now) | Principle libraries curated for specific domains (legal reasoning, medical advice, code review) |
| Tractable (now) | Combined systems using Part 5 patterns + constitutional critique for stronger signals |
| Research-frontier (2–5 yrs) | Critique models *trained* to be more incisive at inference without modifying the base (a "critique LoRA") |
| Research-frontier (2–5 yrs) | Automated principle discovery — the model proposes principles rather than relying on human-curated ones |
| Research-frontier (2–5 yrs) | Faithfulness measurement at inference — detecting when critique-generated text doesn't match actual evaluation |
| Moonshot | Self-improving constitutional loops where the model's own critiques become training data for the next iteration |
| Moonshot | Critique models that genuinely diverge from the generator's biases (closer to the deliberation concept in Part 1) |

## What this implies

Inference-time constitutional AI is the cheapest near-term path to more deliberative behavior. It doesn't require training, can be deployed with existing models, and produces measurable improvements on tasks where the principles are well-chosen.

Its main limitation is the same-model bias: the model is critiquing itself, with the critique quality bounded by what the model can already do. For tasks well within the model's capability, this works well. For tasks at the edge of the model's capability, the critique is shallow and the revisions are cosmetic.

The deeper question is whether critique and generation should be the *same* model at all. Part 1's workspace argument was that genuine deliberation requires perspectives that can genuinely disagree. Same-model critique is not that — it is a single model playing two roles. PRMs are closer to genuine separation, at higher cost. Learned routing (Part 3.8) is a third option: a separate orchestrator model that decides which critique to invoke.

The next post explores that third option.

---

*Previous: [Part 3.6 — Process Reward Models](./process-reward-models.md) · Next: [Part 3.8 — Learned Routing](./learned-routing.md)*
