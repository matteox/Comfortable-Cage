# LoRA as Deliberation Head

*Part 3.5 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) — the problem statement. [Part 2](./beyond-the-org-chart.md) — the strategic paths. [Part 3](./training-models-to-deliberate.md) — the Path 3 research deep dive. This post is a near-term, tractable experiment scoped out of Part 3.*

---

Part 3 ended on the honest assessment that Path 3 — training models natively for multi-perspective deliberation — is a 5–15 year horizon with several open research problems. Process-level loss functions. Deliberative training data. Process evaluation. Faithfulness verification. None of these is solved.

This post asks: what can we do *now*, with existing base models and parameter-efficient fine-tuning, that buys some of the same outcome? Not as a substitute for Path 3, but as a near-term experiment that either fails informatively or produces a usable intermediate capability.

The answer this post explores: **train a LoRA adapter on synthetic deliberative reasoning traces, apply it to a base model, and use the LoRA-active model as a behavioral substitute for native deliberation training.**

## What LoRA actually buys you

LoRA (Low-Rank Adaptation) freezes the base model's weights and learns a small additive perturbation: for a weight matrix W, learn ΔW = A·B where A and B are low-rank. The forward pass becomes y = (W + AB)x. The ΔW is small — rank 4 to 32 is typical — and applied alongside the frozen W.

The practical consequence: behavior becomes a switchable property of the model. A 7B base model with a 50MB LoRA behaves differently than the same model with no LoRA, and you can stack multiple LoRAs at inference.

This is the lever. It means you can ship behavioral "modes" — terse, formal, deliberative, code-first — as small adapters that compose and swap, without retraining the base.

## The architecture: LoRA trained on deliberative traces

The plausible near-term system has four stages.

**Stage 1: Generate deliberative training data.** Run the Part 5 patterns (multi-voice, blackboard, perspective-stitched) on a frontier model that *can* deliberate when prompted carefully. The outputs are interleaved reasoning traces: perspectives, revisions, evidence integration, mid-reasoning reversals, occasional abandonment of positions. Filter for quality using a separate judge model or a heuristic on trace properties (revision count, evidence citation density, position changes).

You now have a corpus of "what good deliberation looks like" for this class of problems.

**Stage 2: Train a LoRA on the traces.** Standard supervised fine-tuning, but on a base model whose natural mode is linear CoT. The LoRA's job is to shift the base model's default behavior toward the patterns in the training data. Rank-16 LoRA is a reasonable starting point — small enough to avoid severe forgetting, large enough to express the behavioral shift.

**Stage 3: At inference, apply the LoRA.** Same base model, with the LoRA active, prompted with workspace-style scaffolding (Part 5). Without the LoRA, the same prompts produce more linear, single-voice reasoning. With the LoRA, the model produces interleaved, revising reasoning natively, even with lighter scaffolding.

**Stage 4: Evaluate against three benchmarks.**
- **Deliberative quality** — does the LoRA-active model produce interleaved, revising reasoning when asked?
- **Decisiveness** — does it commit when warranted, not over-deliberate?
- **Capability preservation** — does it still do math, code, factual recall, and writing at the level of the base model?

The interesting result would be: base + LoRA matches base + heavy workspace-style scaffolding on deliberative tasks, with similar capability preservation, and lower prompt-engineering overhead.

## The training data problem

You can generate deliberative traces from frontier models, but this is partially circular. If the frontier model already deliberates well, you are distilling an existing capability into a LoRA. If it does not, you have nothing to distill.

The honest framing: this approach does not exceed the source model's deliberation quality. It transfers that quality into a smaller, more efficient target model, or makes it accessible with less prompting. That is a real and useful capability shift. It is not Path 3. Path 3 is about training deliberation into models that don't yet have it. This is about extracting deliberation from models that do, into models that don't, with no forgetting of base capabilities.

There is also the distillation-of-biases problem. Whatever blind spots the source model has — its preferred framings, its blind confidence in some domains, its tendency to over-revise in others — get baked into the LoRA. This is the same problem any distillation approach has. Mitigations: diverse source models (multiple frontier models contributing traces), held-out evaluation that includes adversarial tasks, and careful attention to the source model's known failure modes.

## Catastrophic forgetting, specifically for deliberation

Standard LoRA reduces but does not eliminate the forgetting risk. For deliberation, the failure mode is specific and worth naming.

A model trained heavily on deliberative traces can become *excessively* deliberative — hesitant, revision-prone, unable to commit to a position when one is actually warranted. This is a known pathology of over-applied CoT and self-critique patterns. A model that "always revises" can be worse than one that revises when appropriate, because deliberation has costs: context window burn, latency, sometimes reasoning itself going in circles.

The LoRA has to learn not just "how to deliberate" but "when to deliberate." The latter is the harder signal, and it is not always present in the training data. Pure deliberative traces teach "always deliberate." A training mix that includes decisive, single-pass reasoning teaches "deliberate when warranted, commit when warranted." The latter is what you want.

The mitigation is data mix and evaluation. The evaluation is non-trivial — you need test cases where the right answer is to commit without deliberation, and the LoRA-active model has to do that. If it over-deliberates on those cases, the LoRA was over-trained on the wrong signal.

## The MoE reframing

Here is the angle that connects back to the original question about whether humans limit AI by imposing MoE-style structures.

LoRA composition gives you a *different* form of specialization than MoE. In MoE, experts are architecturally partitioned — separate weight matrices that route between at inference. In LoRA composition, "experts" are small behavioral adapters that combine additively. A base model with three LoRAs active simultaneously has, in some sense, three "experts" consultable together — and all the experts share the base's underlying computation.

This suggests something that neither pure MoE nor pure monolithic models offer cleanly:

- **Soft specialization that can be composed, decomposed, and turned off**
- **No architectural partition — all computation shares the base weights**
- **Specialization is a property of the inference-time perturbation, not the model**

Whether this preserves MoE's efficiency gains while avoiding its costs is an empirical question. But the architectural shape is interesting and it directly addresses the question of whether MoE-style partition is the right way to specialize.

## What an actual experiment looks like

If you wanted to test whether this is real, a concrete pilot:

1. Pick an open-weights base model (Llama-3-8B, Qwen-7B, or comparable).
2. Generate 10–50k deliberative traces using a frontier model with the Part 5 patterns. Filter for quality using a separate judge model. Maintain a mix that includes non-deliberative, single-pass traces at roughly 30–50% of the corpus, so the LoRA learns "deliberate when appropriate" rather than "always deliberate."
3. Train a rank-16 LoRA on the filtered mix.
4. Evaluate the LoRA-active model on three benchmark sets:
   - Deliberative quality on multi-perspective tasks
   - Decisiveness on tasks where the right answer is to commit
   - Capability preservation on standard benchmarks (MMLU, HumanEval, etc.)
5. Compare against three controls: base model with workspace prompts and no LoRA; base model with no prompts; frontier model that generated the training data.
6. Look for: LoRA-active model matches or approaches frontier model deliberative quality, with no significant capability loss, and with lighter prompting requirements.

The interesting negative result would be: LoRA-active model loses capability faster than expected, or only achieves deliberative behavior with prompts as heavy as the no-LoRA condition. That would suggest the LoRA cannot internalize the behavioral shift and the model is just doing prompt-following with extra overhead.

The interesting positive result would be: LoRA-active model produces deliberative behavior that *generalizes* — across tasks, prompt styles, and problem types — better than the base model with explicit scaffolding. That would be evidence that the behavioral shift has been internalized, not just pattern-matched.

## What this implies

**If the experiment works:** LoRA-as-deliberation-head is a near-term realization of Path 3, accessible without frontier-lab training runs. The capability gap between "models that can deliberate when prompted" and "models that deliberate by default" closes by an order of magnitude in inference cost and an order of magnitude in time-to-deploy.

**If the experiment fails:** the failure mode is informative. Either the training data wasn't good enough (suggests a distillation-data problem), the LoRA wasn't expressive enough (suggests we need higher rank or full fine-tuning), or deliberation can't be cleanly separated from other capabilities (suggests Path 3 is the only real path, and parameter-efficient methods are insufficient).

**Either way, the experiment is cheap enough to run that we should see results in months, not years.** That is the relevant property of near-term paths: they fail fast and informatively.

## What this does not solve

This approach does not exceed the source model's deliberation quality. It does not solve the process-evaluation problem. It does not solve faithfulness. It does not move us closer to models that genuinely improve their own deliberation through training. All of those remain Path 3 problems.

What it does is give us a tractable intermediate: smaller models with deliberative behavior, deployed cheaply, at the cost of accepting that the behavior is distilled from somewhere else. That is a real capability shift, even if it is not the final answer.

---

## Series so far

- **Part 1** — [Persistent Shared Cognitive Workspaces](./shared-cognitive-workspaces.md) — the problem statement.
- **Part 2** — [Beyond the Org Chart](./beyond-the-org-chart.md) — the viable paths forward.
- **Part 3** — [Training Models to Deliberate](./training-models-to-deliberate.md) — Path 3 research deep dive.
- **Part 3.5** — LoRA as Deliberation Head (this post) — a near-term experiment scoped from Part 3.

*Continue to [Part 3.6 — Process Reward Models During Decoding](./process-reward-models.md) — adding a critic model that guides generation step-by-step.*

*Continue to [Part 3.7 — Inference-Time Constitutional AI](./inference-time-constitutional-ai.md) — using the base model as its own critic.*

*Continue to [Part 3.8 — Learned Routing](./learned-routing.md) — a small orchestrator model that decides what to do and when.*
