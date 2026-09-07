# Persistent Shared Cognitive Workspaces

*Part 1 of The Comfortable Cage, a five-part series on AI reasoning architecture.*

---

> "If I had asked people what they wanted, they would have said faster horses."
> — Henry Ford

It's a great line. The kind of thing you'd expect from a man who bet his company on an idea nobody asked for and turned out to be right.

No record he ever said it exists. Not in his own writing, not in any interview, not in the Ford Museum's collection of two hundred-plus verified quotes. It started in 1999 as one man's guess about what Ford's customers might have said. By 2006, Ford's own great-grandson was repeating it as something his great-grandfather actually said.

Sounds good, though.

Hold on to that feeling — the one where something sounds right and you'd rather not check. Then look at how AI agent systems are built.

## The thesis

Most of them are built like a small company. A "planner" writes a spec, a "coder" implements it, a "reviewer" checks the work, and each hands its output to the next. It feels natural because it's how people organize projects. But AI doesn't have the reasons humans organize work that way, and reproducing the structure anyway may be quietly capping what these systems can do.

The alternative this series argues for is not "more specialized agents" and not "one giant agent." It's something closer to a persistent shared cognitive workspace with truly interleaved reasoning — nearer to chain-of-thought with multiple voices than to a software org. Instead of separate roles passing finished work to each other, one continuously shared understanding that every perspective reads from and writes to at once.

## How we got here

The reason "specialists versus one big agent" feels like the only framing available is that we are already drowning in human coordination primitives. Nearly every architectural default in current agent systems was designed for human teams. Look at what each SDLC artifact was originally solving for:

| SDLC artifact | Why humans need it | Whether AI needs it |
|---|---|---|
| Requirements doc | Humans forget, mishear, disagree on scope; people who can't all be in the room have to agree once, in writing | One shared state every voice reads directly; the document becomes a view of it |
| Architecture phase | Humans can't design and code at the same time, and can't all be consulted at once | No cognitive bottleneck to phase around; design and implementation revise each other on the same board |
| Code review | Humans miss bugs in their own work; a second reader brings independent eyes | The independent reader survives — a different model, not a person with less context. The ceremony around it does not |
| Testing phase | Humans are bad at imagining failure modes, and can't test while they build | Adversarial probing runs during generation, not after — with the same independence caveat as review |
| Documentation | Knowledge evaporates between humans | Rendered as a view of structured state, not written as a separate workstream |
| Standups / syncs | Distributed humans drift out of alignment between the moments they can talk | Shared state is the default; a wall remains only where it buys independent error |

Every one of these is a workaround for a human limitation. Most are, more precisely, rationing schemes: each decides who has to know what, given that knowing costs somebody an hour. Two of them — review and testing — ration something else, an independent second look, and the argument comes back to that difference below, because it is the one that survives. None is intrinsic to building software. When we map them onto AI systems we are not porting a process; we are porting a set of patches for problems the AI mostly doesn't have.

Faster horses, all the way down.

## What the org chart was for

The org chart was a real solution to a real constraint. A person can be in one
place at a time, interruption is expensive, and asking a busy colleague a
question is never free. Read the middle column of the table above again: all
but two rows are attention limits. Roles, hand-offs, summaries, standups, seniority —
each one decides who has to know what, given that knowing costs somebody an
hour. Where attention is the scarce thing, rationing it is correct.

The question is what happens when you carry the answer somewhere the problem
doesn't exist.

Interruption is free here. There is no context-switch penalty, no lost hour of
flow, no irritation at being pulled off something else. Simultaneity is free —
the same perspective can be instantiated forty times at once, across forty
branches of the same problem. Repetition is free; asking the same question a
fifth time costs what the first cost, and nobody sighs.

Something is still scarce, which is where "time doesn't matter for AI"
overshoots. Tokens cost money. Latency is real. The context window is finite,
and quality degrades well before it fills. If every voice consults every other
voice, call count goes quadratic, and that is a bill somebody pays. So the
constraint did not vanish. It moved.

> **The scarce resource is context, not calendar.**

The two scarcities want different architectures. Rationing calendar time means
partitioning by *ownership*: this person handles that, and nobody else needs to
look. Rationing context means selecting by *relevance*: this inference needs
those facts, and the next one needs a different set. Ownership partitions are
stable, hierarchical, and drawn in advance. Relevance selections are
per-question, overlapping, and drawn at the point of use. They are not the same
cut, and one of them is still being made because the other used to be necessary.

The sub-agent pattern is where this is easiest to see. A sub-agent works with
rich context and returns a paragraph to the orchestrator. That is a status
report. It exists in organisations because a director cannot read every diff.
Here the orchestrator could have read all of it. Compression as such is not
the mistake — context is scarce, and something has to be left out. The mistake
is who decides. The sub-agent chose in advance what its reader would need,
which is ownership; the reader could have selected from the whole record what
this question needed, which is relevance. The summary is the wrong cut, made by
the wrong party, at the wrong time — the compression outliving the thing it was
compressing for.

### An aside on mixture-of-experts

An objection arrives at this point from anyone who has read a model card: the models themselves partition. Mixture-of-experts looks like the org chart made architecture: dedicated experts, a router sending each question to the right specialist, nobody bothering anyone else while they work. If that were what MoE is, it would be evidence that partition is natural after all.

It isn't. MoE routing is sparse activation for FLOPs economy — a way to grow parameters without growing compute per token. That scarcity is real and it is arithmetic, not politeness. But the experts are not domain specialists. Routing is largely per-token and driven by learned features that turn out to be substantially positional and syntactic; the interpretability work so far has not found the legal expert or the SQL expert people expect to be in there. The experts specialise in something, and it is mostly not the thing the name suggests.

Which makes MoE an example of the pattern rather than a counterexample. The mechanism is a compute-allocation trick. The org chart is what we read into it: we named the components *experts*, inferred that each one owns a domain, and then built agent systems in the image of the metaphor rather than the mechanism. [Part 3](./training-models-to-deliberate.md) takes this up on the engineering side — composable adapters get similar specialisation by additive perturbation on shared computation, with no partition at all.

### The wall worth keeping

Not every boundary is scheduling, and the argument would be too easy if it were.

Some walls buy independence. Blind review works because reviewers cannot see each other. Two estimates beat one when they were formed separately. The value there is not the partition itself; it is that the errors are uncorrelated.

That concern gets *worse* when the workers are models. Voices drawn from the same base model already share priors; give them identical context and the correlation rises. Convergence then means agreement, not confirmation, and a workspace that converges quickly may be converging because every voice was primed the same way. This is the strongest technical objection to the architecture this series argues for, and it recurs — as an evaluation problem and a proposed measurement in [Part 3](./training-models-to-deliberate.md), as the second reading of the earliest drift signal in [Part 4](./the-hybrid-failure-mode.md), and as a design constraint on every pattern in [Part 5](./a-workspace-in-code.md).

So the rule is not *everyone sees everything*. It is narrower:

> Keep the walls that buy independent error. Demolish the ones that only bought scheduling.

Almost every wall in the SDLC is the second kind.

This is the bet the series is making, and it should be stated as one. Shared cognition is the claim; correlated error is its price; the wager is that a blind first round, late syncs, and one uncorrelated critic keep the price low enough that the gains from never handing anything off survive. If they don't — if independence turns out to cost more context than the board can spare — the architecture that wins is a mixed-model workspace, and the cost argument in this series gets harder. [Part 3](./training-models-to-deliberate.md) says how to find out.

## Why the cage is comfortable

The section above explains why the org chart existed, and why the reason
expired. It does not explain why we copy it anyway. Those are different
questions, and only the first has a clean answer.

So the origin story is finished and the shape is still here. Six reasons keep
it here. They are not variations on one reason: they have different mechanisms,
different evidence, and — this is the part that matters for what to do about it
— different cures. Three of them buy something real.

### 1. Legibility — the pipeline is a UI for management

Look at what each role in a typical pipeline actually produces. An "Architect" agent produces a design document a human PM can read. A "Developer" agent produces code that fits a review template. A "Tester" agent produces a report a QA lead can sign off on. Each step is shaped to be legible to a human overseer, not to maximize the quality of the artifact. The pipeline is, structurally, a UI for management.

A single model with a long context, asked to build a system and identify what could go wrong, will often produce better work. But the output is illegible to an org chart — no role-by-role hand-off, no checklist, nothing that fits a Jira workflow. So we fragment it. Not because fragmentation helps the work, but because fragmentation helps the organization defend the work.

This is a gravitational pull, not a one-time choice. Every time a system fails, the reflex is to add a role to catch the failure next time — a Reviewer, a Critic, a Validator — rather than redesign the reasoning loop so failures surface earlier and cheaper. When single-agent coders fail, the failure is usually not capability but legibility: they produce working software that can't be slotted into the existing machinery, so they get wrapped in role-shaped scaffolding, recreating exactly the partition that was limiting them.

### 2. Liability — someone has to sign the thing

Someone has to sign off. An agent named "QA Lead" can sign off. A single-model output signed off by whom — the model, the prompter, the company? Multi-agent systems distribute authority across named actors in a way that maps onto existing accountability structures. And when the "Architect" makes a bad call, that's a role that failed; when a single model produces a bad output, the failure is diffuse. Spreading blame across more actors is exactly what human organizations do, for exactly the same reason.

Better outcomes are not the optimization target. Defensibility is. The org chart is the artifact of liability allocation, and we are reproducing it in silicon because we don't yet have a different liability model for AI outputs.

Of the six, this is the one with the longest timeline and the least technical content. It is answered by insurers, courts, and regulators, not by architects — which is why [Part 2](./beyond-the-org-chart.md) gives it a path of its own rather than a mitigation.

### 3. Conway's law — you ship your org chart

Conway's 1967 observation is that a system's structure mirrors the communication structure of the organization that built it. It is usually invoked about microservices. It applies here more directly, because the artifact being shaped *is* an organization.

A company with a platform team and an applied-AI team ships an agent architecture with a boundary in exactly that spot. The boundary gets defended on technical grounds by people who do not notice they are describing their own reporting lines. Nobody chooses this and nobody argues for it; it arrives with the org and is already load-bearing by the time anyone reviews the design.

The claim is falsifiable and cheap to check. You should be able to guess the shape of a company's agent framework from its engineering org chart more often than chance. Where a vendor has a research group and a product group under separate leadership, expect a hard interface between reasoning and orchestration. Where one team owns both, expect them tangled together. Anyone with access to a dozen frameworks and their teams could run this in an afternoon, and it would be the cheapest evidence in this series.

No prompt fixes this. It is decided by who is in the room when the interfaces are drawn, which makes it the one cause on this list a technical lead cannot address alone.

### 4. The shipped primitive — someone else's org chart, pre-installed

Reason 3, one level up. The popular agent frameworks ship *role* as the base abstraction: instantiate a role, give it a goal and a backstory, wire it to other roles. That primitive was not derived from evidence about how models reason. It was derived from how the framework's authors pictured a team — Conway's law running at the vendor, with the result shipped to everyone else as an API.

The abstraction available is the abstraction used. A team that would never sit down and deliberately design a hand-off pipeline will build one anyway, because building anything else means fighting the framework the whole way. By the time the architecture gets reviewed, the shape is three months old, and no one remembers choosing it because no one did.

This is the cheapest of the six to fix and the least often noticed. Choose tools whose primitive is shared state, a queue, or an event log, and role-shaped architecture stops being the default that costs nothing and becomes a thing somebody has to argue for.

### 5. The person-metaphor — "agent" is a person-word

We have no vocabulary for coordinating non-human minds, so we borrowed the one for coordinating people. The borrowing is not neutral. Once the unit is an *agent* it wants a *role*; once it has a role it wants a *responsibility*; a responsibility wants a *boundary*. Four words later the architecture is decided. The word was chosen; nothing after it was.

What makes this one durable is that it has real evidence behind it, applied at the wrong layer. Persona prompting works. Telling a model to answer as a skeptical security reviewer produces different and often better output than asking neutrally, and every practitioner has seen it work. That is a fact about conditioning a distribution inside one context. It is not evidence that the *system* should be split into security-reviewer-shaped components with their own contexts and hand-offs between them. The first is prompt engineering and it pays for itself. The second is architecture and it costs. Conflating them is the most common form of the mistake this series is about, and the hardest to argue with, because the person making it can point at a result that is genuinely there.

The distinction is separable and testable: hold the personas fixed and vary only whether they share context. [Part 3](./training-models-to-deliberate.md) proposes it as an ablation; [Part 5](./a-workspace-in-code.md) shows what the two look like in code, and they are only a few lines apart.

### 6. The readable trace — the honest one

A role-shaped system is easier to debug. When the trace reads Architect → Developer → Reviewer, an engineer can find the fault by scanning. When six voices revise a shared board for four hundred turns, they cannot. This is not management theatre. It is someone at 2am trying to locate a failure, and it is why people who accept every argument in this series still ship role-shaped systems and are not being cowards about it.

It also has the clearest exit, because debuggability is a rendering problem, and role-shaped execution is an extravagant way to solve a rendering problem — you have restructured the computation to make the log easier to read. [Part 4](./the-hybrid-failure-mode.md) is the whole argument for paying with a projection instead. But the need is real, and a series that waves it away loses exactly the readers most worth convincing.

### The six, and what to do about each

| Reason | What it buys | Survives contact with AI | Treated in |
|---|---|---|---|
| *Attention scarcity* (the origin) | Nothing here — the constraint doesn't transfer | No | This post |
| Legibility | Real oversight | The need does; the architecture doesn't | Part 2 (Paths 2, 6), Part 4 |
| Liability | Real accountability | Yes — and it needs a new answer, not a new architecture | Part 2 (Path 4) |
| Conway's law | Nothing | No | Part 2 (Path 7) |
| The shipped primitive | Convenience | No | Part 2 (Path 7) |
| The person-metaphor | Prompt-level gains, misapplied at the system level | At the prompt yes, at the architecture no | Part 3, Part 5 |
| The readable trace | Real debuggability | The need does; the architecture doesn't | Part 4 |

Three of the six buy something real: oversight, accountability, debuggability. None of the three requires the architecture we currently pay for it with. The other three are barely trades. The primitive buys convenience and nothing else; Conway's law and the metaphor buy nothing at all. They are the shape arriving by default, out of the org that built the system, the framework it was built on, and the words used to describe it — and two of them, the org and the framework, were never chosen by anyone.

That is what makes the cage comfortable rather than merely wrong. Most of it was never chosen.

## The false dichotomy

Discussions of multi-agent AI oscillate between two poles.

**Specialized agents** — many narrow agents, each excellent at one thing, communicating through serialized hand-offs: spec, ticket, prompt, response. The cost is information loss at every boundary. By the time the "Test Engineer" receives the code, it has lost whatever the "Architect" knew about why certain trade-offs were made.

**One giant agent** — a single model holding everything in one context, reasoning sequentially. No hand-offs, no information loss. The cost is brittleness: early framings commit, nothing outside the model's own monologue can challenge them, and the reasoning converges prematurely on the first plausible path.

Both miss the target. The real distinction is not how many agents, but whether cognition is *partitioned* or *shared*.

```
Pole A: Partitioned              Pole B: Monolithic            Shared Cognition
(specialist agents)              (single agent)                (the alternative)

 [Architect] --spec--> [Impl]     ┌────────────────┐            ┌──────────────────┐
      |                   |       │  one context,  │            │  shared mutable   │
   lossy               lossy      │  one voice,     │            │  state, written   │
  hand-off            hand-off    │  sequential     │            │  and revised by   │
      v                   v       │  reasoning      │            │  every perspective│
 [Tester] ---------> [Deploy]     └────────────────┘            │   ▲   ▲   ▲       │
                                                                  │   │   │   │       │
 cognition is split;              cognition is unified;          │  interleaved,      │
 each hand-off discards           but no outside voice           │  revisable,        │
 context the sender had           can challenge it                │  no premature      │
                                                                  │  commitment        │
                                                                  └──────────────────┘
```

## What a shared cognitive workspace is

Five properties:

1. **Shared mutable state.** Every participant reads and writes the same representation. No private contexts, no hand-offs that compress information — with the one exception carved out above: a first position formed before the board is read, so that each voice brings something independent to it.
2. **No premature commitment.** No perspective "finishes" before others can intervene. A draft architecture can be revised after the implementation perspective has spoken, because the implementation perspective is part of the same ongoing process.
3. **Truly interleaved reasoning.** Perspectives alternate, build on each other, react, revise. Not sequential phases, and not a single merge at the end. The blind first round above is the one deliberate exception, and it is a delay of one round, not a phase.
4. **Continuous revision.** Earlier contributions remain revisable until the whole process terminates.
5. **Termination as convergence, not completion.** No "all roles done" signal. It stops when no perspective can improve the shared state, or when the state crosses a quality threshold.

The difference between an assembly line, where each station does one thing and hands off, and an orchestra, where everyone plays from a shared score and no station "finishes" before the music is over. Both have structure. Only one matches how a unified mind computes.

## Four patterns that instantiate it

They differ in mechanism and share the underlying property: shared mutable state with interleaved perspective. [Part 5](./a-workspace-in-code.md) shows each one in code.

**The blackboard, revived.** The 1980s blackboard architecture is the direct ancestor. A shared board holds the current solution state; multiple knowledge sources watch it and, when triggered, write to it; a scheduler decides who acts next based on what's on the board. The crucial difference from today's multi-agent systems is that specialists see the *entire* board, not just what was handed to them. Partition becomes opportunistic rather than structural — specialization emerges from the problem, not from an imposed org chart.

**Multi-voice chain-of-thought.** One model prompted to hold several named perspectives in conversation with each other, inside one context:

```
[Perspective: Architect]
The system should be event-sourced because...

[Perspective: Implementer]
Event-sourcing creates a problem here: the projection
queries will need replay. That's expensive at this scale.

[Perspective: Architect]
Fair. So we event-source the core domain but use
materialized projections for read paths. Tradeoff:
eventual consistency on the read side.

[Perspective: Critic]
What happens during the consistency window when a
user reads their own write?
```

No hand-offs, no lossy serialization. Each perspective sees and responds to everything before it. This is the most practical implementation today; its cost is context length — and, since every voice is one model reading one scratchpad, it has the highest exposure to correlated error of the four.

**Perspective-stitched reasoning.** Multiple parallel reasoning threads over the same shared working memory, each able to read the others and fold in their findings at chosen synchronization points. Like git branches that rebase onto each other at those points rather than only at the end — and the fewer and later the points, the longer the branches stay independent. Closer to an ensemble than to roles, and the one pattern that keeps the wall worth keeping by construction, because each thread forms a position before it reads the others.

**Adversarial self-critique.** Generate, critique, revise, critique — but with the critique *in dialogue* with the generation rather than a separate stage. The generator can challenge the critique; the critique can reframe what counts as a critique. Constitutional AI, Reflexion, and most self-refine approaches are weak versions of this. The strong version gives the critic the full reasoning trace, not just the final output.

## Research that already points this way

None of it is framed as "shared workspace," but the trend is consistent. [Chain-of-Thought](https://arxiv.org/abs/2201.11903) (Wei et al., 2022) introduced explicit intermediate reasoning — single voice, linear, but the foundation. [Self-Consistency](https://arxiv.org/abs/2203.11171) (Wang et al., 2022) samples many chains and votes at the end, sharing final answers but not intermediate reasoning. [Tree of Thoughts](https://arxiv.org/abs/2305.10601) and [Graph of Thoughts](https://arxiv.org/abs/2308.09687) (Yao et al., 2023; Besta et al., 2023) branch and merge over reasoning states and terminate on quality, not role completion. [Multi-Agent Debate](https://arxiv.org/abs/2305.14325) (Du et al., 2023) runs agents against each other with the transcript as shared state and terminates on convergence — implemented as separate agents, conceptually identical to multi-voice chain-of-thought. [Constitutional AI](https://arxiv.org/abs/2212.08073) (Bai et al., 2022) has the model critique itself against principles, linearly. [Reflexion](https://arxiv.org/abs/2303.11366) (Shinn et al., 2023) persists reflections on failure across attempts. [Generative Agents](https://arxiv.org/abs/2304.03442) (Park et al., 2023) and [Voyager](https://arxiv.org/abs/2305.16291) (Wang et al., 2023) both use a persistent shared environment or skill library as the workspace.

What they share: persistent state, multiple perspectives, revision, convergence-based termination. What most still lack: genuine interleaving. Nearly all of them still have phases.

## What the evidence says

This argument no longer rests on architectural intuition alone. Two lines of recent work bear directly on it.

The diagnosis has been measured. The [MAST taxonomy](https://arxiv.org/abs/2503.13657) (Cemri et al., 2025) annotated more than 1,600 execution traces across seven popular multi-agent frameworks and found fourteen distinct failure modes, clustered into three groups: specification and system design problems — task misinterpretation, ambiguous roles, poor decomposition, missing termination conditions — at roughly 42% of failures; inter-agent misalignment at roughly 37%; and verification failures at roughly 21%. Nearly four failures in five trace to how the roles were specified and how the hand-offs between them broke, not to the underlying models. Multi-agent systems, in the paper's own framing, fail the way organizations fail.

The prescription has been benchmarked. The blackboard is back in the literature by name: [Han and Zhang (2025)](https://arxiv.org/abs/2507.01701) show that externalizing intermediate reasoning to a shared board mitigates the information fragmentation of role-based collaboration and allows agent selection to adapt to the board's state; [Salemi et al. (2025)](https://arxiv.org/abs/2510.01285) report 13–57% relative improvements in end-to-end success over strong role-based baselines on data-discovery benchmarks, with agents volunteering to contribute based on what's on the board rather than being assigned by a controller. Work in 2026 ([PatchBoard](https://arxiv.org/abs/2605.29313), [deterministic blackboard pipelines](https://dl.acm.org/doi/10.1145/3816713.3818808)) has moved to the next problem — how shared state should be updated, authorized, and audited over long horizons — which is exactly the legibility question this series takes up in [Part 2](./beyond-the-org-chart.md) and [Part 4](./the-hybrid-failure-mode.md).

What the evidence does not yet settle is the institutional claim above: *why* role-shaped systems keep getting built after the failure modes are known and the alternative benchmarks better. That is the part of the argument this series exists to make.

## What's unsolved

The hard parts are real. **Context economics** — shared state grows, and models are bad at knowing what to forget. **Termination** — "no perspective can improve this" is harder to detect than "all roles are done," especially when improvement is asymptotic. **Verification** — role-based systems have a QA phase as the answer; a workspace needs evaluation that is itself part of the workspace, ongoing rather than staged. **Who decides** — when perspectives can't converge, whoever breaks the tie reintroduces exactly the authority structure the workspace was meant to dissolve. **Correlated error** — the sharpest of them, because it is the workspace's own mechanism turned against it. Voices sharing a base model and a board are the least independent judges available, so convergence can be agreement rather than confirmation, and there is no established way to measure how much independence a given amount of shared context costs. **Compute** — N perspectives times M revisions is expensive today, though inference costs are falling. And **four of the six reasons above sit outside the architecture entirely** — liability is answered by insurers and courts, Conway's law by who reports to whom, the primitive by what the ecosystem ships, the metaphor by the language available. A perfect workspace design does not touch any of them, which is why this series does not end at Part 1.

Those problems set the agenda for the rest of the series: what to do strategically given the diagnosis ([Part 2](./beyond-the-org-chart.md)), how to train models that deliberate natively ([Part 3](./training-models-to-deliberate.md)), how to run a hybrid without letting it collapse back into roles ([Part 4](./the-hybrid-failure-mode.md)), and what the patterns look like in code ([Part 5](./a-workspace-in-code.md)).

## Why this matters

The original question was whether humans are limiting AI by imposing paradigms we can understand. The answer is yes, but not in the obvious way. The obvious worry is that partitioning cognition along human expert boundaries loses capability. The deeper worry is that we are limiting AI by making its reasoning illegible to us — a workspace with interleaved, revising, multi-perspective deliberation produces better outputs and harder-to-follow traces, and we have every incentive to flatten that into roles and phases, because roles and phases are what org charts understand.

The ceiling is not technical. The problems that are technical are named above, and they are problems of degree — how much context, how much independence, how to tell convergence from agreement. The ceiling is institutional — and, in the two structural cases, not even that: it is inherited from an org chart and a library import that nobody examined. We are not building AI systems that reason well; we are building AI systems that reason in ways we can defend to a project manager, in a shape handed to us by our own reporting lines and our framework's base class. The path forward is systems whose reasoning is genuinely better than ours, even when — especially when — it doesn't look like ours.

The consolation, such as it is: the three reasons that buy something real can be paid for another way, and the three that buy nothing were never argued for in the first place.

Shared cognitive workspaces are one sketch of what that looks like.

---

*Next: [Part 2 — Beyond the Org Chart](./beyond-the-org-chart.md)*
