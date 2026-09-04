# The Comfortable Cage

A nine-part blog series on AI reasoning architecture and the human organizational shapes we project onto it.

## The thesis

We are limiting AI — not by what we build, but by how we organize what we build. Most multi-agent AI systems today are organized into roles the way human teams are organized into roles: hand-offs, sign-offs, separate specialists with partitioned contexts.

These shapes were designed for workers with payroll, ego, and eight-hour attention limits. AI systems have none of those constraints. We copy the structure anyway — not because it produces better AI, but because the org has to be able to defend the AI to itself.

This series argues that the alternative requires rethinking both architecture (workspace-style reasoning) and institutions (training, evaluation, accountability) at the same time.

## Reading order

The series is meant to be read in order. Each post builds on previous ones; by the end you'll be reading code that depends on concepts introduced in Part 1.

| # | Post | What it covers |
|---|---|---|
| 1 | [Persistent Shared Cognitive Workspaces](./shared-cognitive-workspaces.md) | The problem statement. Why org charts get projected onto AI, what it costs, and what the alternative looks like. |
| 2 | [Beyond the Org Chart](./beyond-the-org-chart.md) | Given the diagnosis, the six viable paths forward. |
| 3 | [Training Models to Deliberate](./training-models-to-deliberate.md) | Path 3 research deep dive — the moonshot. What "training models to deliberate" actually requires. |
| 3.5 | [LoRA as Deliberation Head](./lora-as-deliberation-head.md) | A near-term experiment scoped from Part 3. Cheap, tractable, might fail informatively. |
| 3.6 | [Process Reward Models](./process-reward-models.md) | Add a critic model that guides generation step by step. |
| 3.7 | [Inference-Time Constitutional AI](./inference-time-constitutional-ai.md) | Use the base model as its own critic, prompted by principles. |
| 3.8 | [Learned Routing](./learned-routing.md) | A small orchestrator that decides what to do and when. |
| 4 | [The Hybrid Failure Mode](./the-hybrid-failure-mode.md) | The most likely adoption path — workspace reasoning internally, role-shaped observability externally — and how it tends to degrade. |
| 5 | [A Workspace in Code](./a-workspace-in-code.md) | Concrete prompt-engineering implementations of the Part 1 patterns. The most hands-on post. |

Posts 3.5–3.8 form a sub-series on near-term experiments. Each one adds an architectural element to the previous (LoRA on the base, then a separate critic, then self-critique, then an orchestrator). Read individually or in sequence — they reinforce each other.

## The argument in one paragraph

The current trajectory of multi-agent AI is impressive and constrained. The systems are good enough to ship but not good enough to reveal what they could be. We know how to do better architecturally — shared reasoning, no hand-offs, interleaved perspectives — but we keep reaching for role-shaped systems because role shapes are what existing institutions can defend. The path forward is concrete: better training objectives, new evaluation criteria, near-term experiments in LoRA-based behavioral modulation, critic models, learned routing, and accountability frameworks that focus on outcomes rather than process. The series walks through the problem, the strategic landscape, the research frontier, and the implementations — ending on the question of whether the field will move on principle or on catastrophe.
