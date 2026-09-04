# A Workspace in Code

*Part 5 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) — the problem statement. [Part 2](./beyond-the-org-chart.md) — the strategic paths. [Part 3](./training-models-to-deliberate.md) — Path 3 research deep dive. [Part 4](./the-hybrid-failure-mode.md) — Path 2 in practice. Part 5 turns the Part 1 architecture into concrete prompt-engineering patterns.*

---

Part 1 described four architectural patterns for shared cognitive workspaces:

1. The blackboard pattern
2. Multi-voice chain-of-thought
3. Perspective-stitched reasoning
4. Adversarial self-critique loops

These were abstractions. This post turns them into code.

The implementations below are deliberately minimal. They show the structural shape, not a production-ready system. Real systems would add error handling, tracing, evaluation hooks, and human-review integration. The point here is to make the patterns concrete enough that you can adapt them to your own context.

## Pattern 1 — Multi-voice chain-of-thought

The simplest pattern. A single model, prompted to hold multiple perspectives in conversation with each other within one context window.

```python
def multi_voice_deliberation(task, perspectives, max_rounds=5):
    scratchpad = ""
    for round in range(max_rounds):
        for perspective in perspectives:
            response = llm(
                system=PERSPECTIVE_PROMPTS[perspective],
                user=f"""Task: {task}

Current deliberation:
{scratchpad}

Respond as {perspective}. Build on what's been said.
If you have nothing new to add, say 'no contribution'."""
            )
            if "no contribution" not in response.lower():
                scratchpad += f"\n\n[{perspective}]\n{response}"
    return llm(
        system="You are a synthesizer.",
        user=f"""Task: {task}

Deliberation:
{scratchpad}

Produce the best answer you can, integrating the perspectives above."""
    )
```

Key properties:
- One model, one context window. No hand-offs, no information loss.
- Each perspective can see all previous contributions.
- Perspectives can revise earlier positions — they're all in the scratchpad.
- Termination is by round count, which is a weak form of convergence detection.

The weakness: the round count is a heuristic. A better implementation has each perspective declare convergence or signal no-contribution, and stops when all perspectives converge.

### Prompt template for a perspective

```
You are {perspective_name}.

Your job is to {role description}. You are not the only
perspective on this task. Other perspectives will speak
before and after you. Your contribution should:

- Build on what others have said, especially when they
  raised concerns you had not considered.
- Explicitly revise your position when you encounter
  evidence that warrants revision.
- Say "I have no contribution" when the current state
  of the deliberation does not need your input.

Do not try to wrap things up or produce a final answer.
Other parts of the system handle that.
```

The "I have no contribution" mechanic matters. Without it, every perspective feels obligated to say something, and the deliberation degrades into noise — perspectives restating each other or padding their output.

### When to use this

Tasks with clear perspective diversity: design decisions with multiple stakeholders, ethical questions, strategic analysis, anything where the answer genuinely depends on what viewpoint you take. Cheapest pattern to run; lowest context overhead.

## Pattern 2 — The blackboard

Multiple perspectives write to a shared structure. The structure has typed slots — *current hypothesis*, *evidence for*, *evidence against*, *open questions*. Each perspective reads the structure, updates the slots it has authority over, and signals when its updates are complete.

```python
class Blackboard:
    def __init__(self):
        self.hypothesis = None
        self.evidence_for = []
        self.evidence_against = []
        self.open_questions = []
        self.history = []

    def update(self, perspective, updates):
        for key, value in updates.items():
            if hasattr(self, key):
                setattr(self, key, value)
        self.history.append({
            "perspective": perspective,
            "updates": updates
        })

    def render(self):
        return f"""
HYPOTHESIS: {self.hypothesis}

EVIDENCE FOR:
{chr(10).join('- ' + e for e in self.evidence_for)}

EVIDENCE AGAINST:
{chr(10).join('- ' + e for e in self.evidence_against)}

OPEN QUESTIONS:
{chr(10).join('- ' + q for q in self.open_questions)}
"""

def blackboard_deliberation(task, perspectives):
    bb = Blackboard()
    bb.hypothesis = llm(
        "You are an initial hypothesis generator.",
        f"State a starting hypothesis for: {task}"
    )

    for _ in range(MAX_ROUNDS):
        for perspective in perspectives:
            updates = json.loads(llm(
                system=PERSPECTIVE_PROMPTS[perspective],
                user=f"""Current blackboard state:
{bb.render()}

Update the blackboard. Return JSON with keys
'hypothesis', 'evidence_for', 'evidence_against',
'open_questions'. Only include keys you want to
change."""
            ))
            bb.update(perspective, updates)

        if convergence_detected(bb):
            break

    return bb
```

The blackboard structure does something multi-voice doesn't: it forces explicit treatment of evidence on both sides. The *evidence against* slot is hard to ignore, which is the point.

The weakness: JSON-updating the structure is brittle. Models sometimes produce malformed JSON or update slots they weren't asked to. Robust implementations need schema validation and partial-update handling. A fallback to text-only updates, parsed leniently, is more resilient than strict JSON.

### When to use this

Tasks with explicit evidence/proof structure: debugging hypotheses, scientific reasoning, debugging why a system behaves a certain way, evaluating the strength of an argument. The typed slots force a discipline that pure free-form deliberation doesn't.

## Pattern 3 — Perspective-stitched reasoning

Multiple parallel reasoning threads, each with its own trajectory. Periodic synchronization points where threads exchange. Closer to ensemble methods than to role-play.

```python
def perspective_stitched(task, perspectives, sync_points=3):
    threads = {p: [] for p in perspectives}

    # Phase 1: independent exploration
    for p in perspectives:
        threads[p].append(llm(
            system=PERSPECTIVE_PROMPTS[p],
            user=f"Approach this task from your perspective: {task}"
        ))

    # Phase 2: synchronized exchange
    for _ in range(sync_points):
        sync_summary = "\n\n".join(
            f"[{p}]: {threads[p][-1]}" for p in perspectives
        )
        for p in perspectives:
            threads[p].append(llm(
                system=PERSPECTIVE_PROMPTS[p],
                user=f"""Other perspectives have produced:
{sync_summary}

Continue from your perspective, incorporating
their findings where relevant."""
            ))

    # Phase 3: synthesis
    return llm(
        system="You are a synthesizer.",
        user=f"""Task: {task}

Final state of each perspective:
{chr(10).join(f'[{p}]: {threads[p][-1]}' for p in perspectives)}

Produce an integrated answer."""
    )
```

The structural difference from multi-voice: each perspective has its own thread of thought that develops over time. They share information at sync points, but each retains its own trajectory.

### When to use this

Multi-method problems where genuinely different analytical approaches should be developed independently before integration. Math problems where one perspective tries algebra while another tries geometry. Strategic analysis where multiple stakeholders view the same situation from different frames. The cost is high — N parallel model calls per sync round — but the value is in letting distinct approaches mature before they influence each other.

## Pattern 4 — Adversarial self-critique

Generate, critique, revise. The critique is in dialogue with the generation, not a separate stage.

```python
def adversarial_critique(task, max_iterations=5):
    draft = llm(
        "You are a generator. Produce a first attempt.",
        task
    )

    for _ in range(max_iterations):
        critique = llm(
            "You are a critic. Find the strongest objection.",
            f"""Draft:
{draft}

Identify the single strongest objection to this
draft. Be specific. If the draft is correct, say
'NO OBJECTION'."""
        )

        if "NO OBJECTION" in critique:
            break

        revision = llm(
            "You are the generator. Address the critique.",
            f"""Draft:
{draft}

Critique:
{critique}

Revise the draft to address the critique. You may
revise fully or partially. Explain your revision."""
        )
        draft = revision

    return draft
```

The weakest of the patterns, but the easiest to implement. The critique-revision loop catches some classes of error — overconfidence, missing considerations, unsupported claims.

The structural weakness: the critique is generated by the same model that produced the draft, in response to a different system prompt. There is no guarantee the model will find genuine flaws in its own work. Constitutional AI addresses this by training the critique to be more incisive, but it's still imperfect. Same-model blind spots are real.

### When to use this

Output quality improvement on already-drafted material: writing, reasoning chains, plans, code. Best used as a finishing layer applied to the output of one of the other patterns, not as the primary deliberation mechanism.

## Choosing a pattern

The patterns aren't interchangeable. Quick guidance:

| Pattern | Best for | Cost | Risk |
|---|---|---|---|
| Multi-voice | Tasks with clear perspective diversity (design, strategy, ethics) | Low (one model, one context) | Context overflow on long tasks |
| Blackboard | Tasks with explicit evidence/proof structure | Medium (JSON schema, validation) | Schema brittleness |
| Perspective-stitched | Multi-method problems (math, science, optimization) | High (multiple parallel calls) | Integration losses at sync |
| Adversarial critique | Output quality improvement (writing, reasoning) | Low-medium | Same-model blind spots |

For most tasks, start with multi-voice. It is the simplest, cheapest, and most robust. Move to blackboard when the task has explicit evidence structures that free-form deliberation won't surface. Move to perspective-stitched when the task benefits from parallel method exploration. Use adversarial critique as a finishing layer for any of the above.

## Composing the patterns

These patterns compose. A realistic production system might:

1. Use **perspective-stitched** for the initial exploration phase, with three perspectives approaching the problem differently.
2. Pour the results into a **blackboard** to consolidate into a structured hypothesis with explicit evidence on both sides.
3. Run **multi-voice** over the blackboard to refine the hypothesis, with perspectives challenging each other.
4. Apply **adversarial critique** to the final synthesis as a sanity check.

Each layer is doing something the others can't. The cost is high — but the result is reasoning that has been triangulated through genuinely different paths, with explicit evidence handling and an adversarial pass at the end.

This is more work than a single linear CoT. It is also qualitatively different. Whether the difference is worth it depends on the stakes of the decision being made.

## What this gets you, what it doesn't

These patterns produce better reasoning than a single linear CoT for many tasks. They don't solve the fundamental limits — context window size, same-model critique blindness, JSON brittleness, convergence detection. They are engineering patterns, not breakthroughs.

What they do demonstrate: workspace-style reasoning is implementable today, with current models, in a few hundred lines of code. The barrier is not technical. It is the gravitational pull back toward role-shaped systems that Part 1 described.

Even when you implement these patterns, you will feel the urge to name them "Architect" and "Developer" — to make them legible by making them familiar. Resist that. The patterns work because they are not role-shaped. The moment they become role-shaped, the hand-off losses return.

The code in this post is a starting point. The discipline of keeping it from drifting back into roles is the actual work — and that work is what Part 4 was about.

---

## Series so far

- **Part 1** — [Persistent Shared Cognitive Workspaces](./shared-cognitive-workspaces.md) — the problem statement.
- **Part 2** — [Beyond the Org Chart](./beyond-the-org-chart.md) — the viable paths forward.
- **Part 3** — [Training Models to Deliberate](./training-models-to-deliberate.md) — Path 3 research deep dive.
- **Part 4** — [The Hybrid Failure Mode](./the-hybrid-failure-mode.md) — running Path 2 in practice.
- **Part 5** — A Workspace in Code (this post) — concrete prompt-engineering patterns.

*End of series. The five posts together form a complete argument: from diagnosis to architectural alternative to strategic landscape to research frontier to practical implementation. Whether the field actually moves along these lines is the open question the series ends on.*