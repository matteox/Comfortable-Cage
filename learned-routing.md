# Learned Routing

*Part 3.8 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) — the problem statement. [Part 3](./training-models-to-deliberate.md) — Path 3 deep dive. [Part 3.5](./lora-as-deliberation-head.md) — modifying the base model. [Part 3.6](./process-reward-models.md) — adding a critic model. [Part 3.7](./inference-time-constitutional-ai.md) — using the base model as its own critic. This post covers adding a small orchestrator model that decides what to do and when.*

---

The previous three posts in the 3.x sub-series explored near-term paths to deliberative behavior:
- 3.5 — modify the base model via LoRA
- 3.6 — add a separate critic (PRM)
- 3.7 — use the base model as its own critic, prompted by principles

All three have the same shape: a *generator* produces reasoning, with some form of *evaluation* shaping what gets produced. What none of them address is *structure* — who decides what the deliberation looks like in the first place. Which perspective gets invoked when. When to switch from generation to critique. When to stop.

Part 1's workspace argument was that deliberation needs structure that emerges from the problem, not structure imposed up front. Part 5 sketched patterns that impose structure via prompts. This post asks whether the structure itself can be learned.

## The idea: a learned orchestrator

A learned router — sometimes called an agent router, a planner, or a controller — is a small model that takes the current state of a deliberation and decides what to do next. The decisions might include:

- Which perspective to invoke next (skeptic, generator, synthesizer, domain expert)
- Which LoRA to activate (Part 3.5's deliberation head vs. a code-specialist LoRA vs. a writing-style LoRA)
- Whether to invoke a critic (Part 3.6's PRM, Part 3.7's constitutional critique)
- Whether to terminate (the convergence-detection problem from Part 1)

The router is trained on traces of good deliberation. The training signal is: at each decision point, what action would have led to better deliberation? This is similar to process supervision but at the structural level rather than the content level.

The architecture looks like a learned version of the Part 5 patterns, with the router playing the role of the human orchestrator who would normally hand-code the workflow.

## How this could produce deliberative behavior

The pitch is straightforward: instead of hard-coding a deliberation structure (multi-voice, then synthesis; or generate, then critique, then revise), learn the structure from data. The router learns which sequence of perspectives, critiques, and revisions produces the best outputs.

The advantage over hard-coded structure:
- The structure adapts to the task. A simple question gets a simple deliberation. A complex ethical question gets an extended multi-perspective deliberation. The router decides.
- The structure can include non-obvious steps. A human-designed workflow might miss that this particular task benefits from invoking the domain expert *after* the skeptic, not before. The router can learn that.
- The structure improves with data. As more deliberative traces are collected, the router's decisions improve. Hard-coded structure doesn't.

The advantage over single-prompt patterns:
- The deliberation can be conditional. "If the model is uncertain, invoke the skeptic; if the model is confident, proceed to synthesis." Hard-coded patterns can't make these distinctions without becoming unwieldy.
- The router can choose between radically different deliberation styles for different tasks. Math gets a step-by-step PRM-guided approach. Ethics gets a multi-voice deliberation. Code gets a critique-revise loop. The router picks.

This is closer to Part 1's vision of structure emerging from the problem, not imposed by the framework.

## Routing tokens vs routing LoRAs

There are two distinct modes for learned routing, with different implications.

**Routing tokens (the MoE pattern).** At each layer of the model, the router decides which subset of parameters to use for the next token. This is classical Mixture of Experts. It is architectural: the routers live inside the base model, and the experts are pre-defined partitions of the weights.

**Routing LoRAs.** At each step of generation, the router decides which LoRAs to activate. The base model is shared; only the LoRA perturbation changes. This is closer to Part 3.5's frame.

These are not equivalent. Token-level MoE routing operates at a much finer granularity — different tokens can route to different experts within the same generation. LoRA routing operates at a coarser granularity — once you activate a LoRA, it applies to the whole subsequent generation until you switch.

For deliberation specifically, LoRA routing may be more natural. A deliberation step tends to be a coherent unit: this perspective, this critique, this revision. Switching mid-step is usually wrong. Coarser routing aligns better with the natural structure of deliberation.

But there is a hybrid worth considering: token-level routing within a base model, plus LoRA-level routing on top. The token-level routing handles "which part of the base computation is most relevant for this token"; the LoRA-level routing handles "which behavioral mode should dominate this step of the deliberation." This is essentially Part 3.5's LoRA composition + classical MoE, layered.

## What's gained, what's lost

Gained:
- Structure that adapts to the task
- Learned composition of perspectives and critics
- Conditional deliberation (only deliberate when warranted)
- The capability to learn from traces of good deliberation, improving over time

Lost (relative to the simpler paths in 3.5–3.7):
- Interpretability. A learned router's decisions are not directly inspectable. You can log which LoRA it activated, but why it made that choice is a black box.
- Robustness. A learned router can fail in ways hard to predict. A hard-coded pattern fails predictably (the pattern didn't account for this case) and is easier to fix.
- Training cost. The router itself needs training data, which has the same deliberative-corpus problem from Part 3.

The router also creates a new kind of single point of failure. If the router is bad, the deliberation is bad, even if the base model and critics are good. The router becomes load-bearing in a way that hand-coded patterns are not.

## Connection to the MoE reframing

This is where the post connects back to the original question about whether humans limit AI by imposing MoE-style structures.

Classical MoE is a specific architectural commitment: experts are pre-defined partitions of weights, routing is token-level and architectural, experts share no computation. The motivation is efficiency — only the relevant experts activate per token.

LoRA routing is structurally different: experts (LoRAs) share all base-model computation, routing is at a coarser granularity (per deliberation step), and the router is learned rather than architectural. The efficiency argument is weaker (the base model is always computed), but the flexibility argument is stronger (experts compose, decompose, and turn off).

Whether LoRA routing preserves MoE's efficiency advantages is an empirical question. In practice, base-model inference dominates the cost of LoRA activation, so the efficiency loss is modest. The flexibility gain is substantial.

This is the version of MoE that doesn't limit AI by imposing architectural partition. It is a *behavioral* MoE, where "experts" are modes of operation rather than partitions of computation. Whether this turns out to be a meaningful distinction or a marketing rebranding is something experiments will tell us.

## What's tractable vs what's the moonshot

**Tractable (now):**
- Hard-coded routers that mimic learned routing (state machines over deliberation patterns). Not learned, but useful as a baseline.
- Routing LoRAs for known task types (math → PRM-guided chain, ethics → multi-voice, code → critique-revise). Cheap to implement, easy to debug.

**Near-term (1–2 years):**
- Learned routers trained on deliberative traces. Cheap to train once traces exist; main cost is trace collection.
- Router evaluation: did the router's decisions lead to better deliberation than a baseline hand-coded structure? Requires the evaluation protocols from Part 3.

**Research-frontier (3–5 years):**
- Routers that generalize across task types (no retraining for new domains)
- Routers that learn from weaker signals (outcome-only, not process-level)
- Hybrid token + LoRA routing that combines the efficiency of MoE with the flexibility of behavioral routing

**Moonshot:**
- Routers that discover new deliberation structures not present in training data
- Self-improving routers where router decisions become training data for the next router

## What this implies for the series

Learned routing closes a loop that the previous posts left open. Part 3.5 modifies the base. Part 3.6 adds a critic. Part 3.7 prompts the base to be its own critic. None of them address *structure* — who decides what the deliberation looks like.

Learned routing says: structure can be learned too. The router is a small, fast model that does what a human orchestrator would otherwise do, but better and faster.

If this works, the architectural shape that emerges looks like:
- A shared base model (anyone's frontier model)
- A library of LoRAs (one per behavioral mode — deliberation, code, terseness, etc.)
- A small router that picks which LoRA(s) to activate per task
- Optional critics (PRMs, constitutional prompts) that the router can invoke

This is a *behavioral* MoE, with composition, decomposition, and learned structure. It is not the architectural MoE that Part 1 critiqued. Whether it is meaningfully different — or just MoE with extra steps — depends on whether the flexibility gain justifies the efficiency cost.

The honest answer: we don't know yet. The experiments haven't been run at scale. But the path is now clear, and the experiments are cheap enough to run that we should see results.

---

## Series so far

- **Part 1** — [Persistent Shared Cognitive Workspaces](./shared-cognitive-workspaces.md) — the problem statement.
- **Part 3** — [Training Models to Deliberate](./training-models-to-deliberate.md) — Path 3 research deep dive.
- **Part 3.5** — [LoRA as Deliberation Head](./lora-as-deliberation-head.md) — modifying the base model.
- **Part 3.6** — [Process Reward Models](./process-reward-models.md) — adding a separate critic.
- **Part 3.7** — [Inference-Time Constitutional AI](./inference-time-constitutional-ai.md) — using the base model as its own critic.
- **Part 3.8** — Learned Routing (this post) — adding an orchestrator that decides what to do.

*The 3.x sub-series is now complete. The next post in the main series is [Part 4 — The Hybrid Failure Mode](./the-hybrid-failure-mode.md), the practitioner-focused deep dive on Path 2.*