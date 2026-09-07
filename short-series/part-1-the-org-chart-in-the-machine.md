# The Org Chart in the Machine

*Part 1 of 2. Why your AI agents are organised like a small software company, what that costs, and the two fixes that take an afternoon.*

---

Open the framework your agent system is built on and find its base class.

There is a good chance it is a *Role*. It has a goal and a backstory. You instantiate an Architect, a Developer, a Reviewer, wire them together, and each hands its output to the next. It feels natural, because it is how people organise projects. It is also an architectural decision that nobody on your team made. The library's authors made it, when they pictured a team, and shipped their picture as an API.

That is the shape of nearly every multi-agent system in production today: a small software company, rendered in prompts. A planner writes a spec, a coder implements it, a reviewer checks the work. This essay argues that the shape was never discovered. It was inherited — and it is costing you more than anyone is measuring.

## What the org chart was for

Every coordination structure in software — roles, hand-offs, requirements documents, review gates, standups, seniority — answers the same question: *who has to know what, given that knowing costs somebody an hour?*

That question is real when the workers are people. A person can be in one place at a time. Interruption is expensive. Asking a busy colleague is never free. So we ration attention: we partition work by ownership, we compress whatever crosses each boundary into a document or a ticket or a summary, and we call the result process.

Now ask which of those constraints a model has.

Interruption is free. There is no lost hour of flow, no irritation at being pulled off something else. Simultaneity is free — the same perspective can run forty times at once, over forty branches of the same problem. Repetition is free; the fifth time you ask costs what the first did. The problem the org chart solved does not exist here.

Something is still scarce, which is where "time doesn't matter for AI" overshoots. Tokens cost money. The context window is finite, and quality degrades well before it fills. The constraint did not vanish. It moved.

**The scarce resource is context, not calendar.** And the two scarcities want different architectures. Rationing calendar time means partitioning by *ownership*: this person handles that, and nobody else needs to look. Rationing context means selecting by *relevance*: this inference needs those facts, and the next one needs a different set. Ownership partitions are drawn in advance and stay put. Relevance selections are made per question, at the point of use. They are not the same cut — and one of them is still being made because the other used to be necessary.

The clearest specimen is the sub-agent. It works with full context, then returns a paragraph to its orchestrator. That is a status report. It exists in companies because a director cannot read every diff. Here, the orchestrator could have read all of it and chosen what mattered for its question. Instead the sender chose in advance. Compression is not the mistake — context is scarce, and something has to be left out. The mistake is *who decides, and when*: ownership rationing a context it doesn't own.

## The bill

This is no longer an intuition.

In 2025 a research team [annotated more than 1,600 execution traces](https://arxiv.org/abs/2503.13657) across seven popular multi-agent frameworks and catalogued fourteen distinct ways they fail. Roughly 42% of failures were specification and system-design problems — ambiguous roles, poor decomposition, missing termination conditions. Roughly 37% were inter-agent misalignment — agents failing to model what other agents knew. Only about a fifth were verification failures, and almost none were the model being incapable. Nearly four failures in five trace to how the roles were specified and how the hand-offs between them broke. In the paper's own framing, multi-agent systems fail the way organisations fail.

The alternative has been benchmarked too. The blackboard — a 1980s architecture in which every participant reads and writes one shared board — is back in the literature by name. One 2025 study [reports 13–57% relative improvement](https://arxiv.org/abs/2510.01285) in end-to-end success over strong role-based baselines, with participants volunteering to contribute based on what is on the board rather than being assigned work by a controller. [Another](https://arxiv.org/abs/2507.01701) shows that externalising intermediate reasoning to a shared board directly mitigates the information fragmentation of role-based collaboration.

And there is a cost you are paying that appears in neither paper, because nobody measures it. Follow one ordinary agent task through a corporate stack — read an email, search the docs, read five pages, write a reply, log a ticket, authenticate — and price each step with and without the translation between human-shaped formats: MIME and quoted threads, prose chunks instead of facts, state-machine metadata, schema wrapping. The worked illustration in Part 2 comes out at roughly 2.6×. On that task, about sixty cents of every dollar goes to translating between shapes designed for a different worker, not to reasoning. The number is illustrative, not measured — which is itself the finding. Instrument your own deployment and the figures will differ. The shape will not.

## Why we keep drawing the old cut

If the evidence is this clear, why is the role-shaped system still the default? Not for one reason. For six — and they sort into three pairs.

**Two are defensive, and they buy something real.**

*Legibility.* Look at what each role in a pipeline actually produces. The Architect produces a design document a product manager can read. The Developer produces code that fits a review template. The Tester produces a report a QA lead can sign. Each step is shaped to be legible to a human overseer, not to maximise the quality of the work. The pipeline is, structurally, a UI for management. And it is a gravitational pull, not a one-time choice: every time the system fails, the reflex is to add a role to catch it next time — a Reviewer, a Validator — rather than redesign the reasoning so the failure surfaces earlier.

*Liability.* Someone has to sign off. An agent named "QA Lead" can sign off. A single model's output — signed off by whom? Multi-agent systems distribute authority across named actors in a way that maps onto the accountability structures you already have. When the Architect makes a bad call, a role failed. When one model produces a bad output, the failure is diffuse. Better outcomes are not the optimisation target. Defensibility is.

**Two are structural, and nobody chose them.**

*Conway's law.* A system's structure mirrors the communication structure of the organisation that built it. It is usually invoked about microservices. It applies here more directly, because the artifact being shaped *is* an organisation. A company with a platform team and an applied-AI team ships an agent architecture with a boundary in exactly that spot, defended on technical grounds by people who do not notice they are describing their own reporting lines. This is falsifiable and cheap to check: you should be able to guess the shape of a company's agent framework from its engineering org chart more often than chance.

*The shipped primitive.* Conway's law, one level up. The framework's Role class was not derived from evidence about how models reason. It was derived from how the framework's authors pictured a team — their org chart, shipped to you as an API. The abstraction available is the abstraction used. A team that would never sit down and design a hand-off pipeline builds one anyway, because building anything else means fighting the framework all the way. By the time the architecture is reviewed, the shape is three months old, and nobody remembers choosing it because nobody did.

**Two are cognitive.**

*The person-metaphor.* We have no vocabulary for coordinating non-human minds, so we borrowed the one for people. Once the unit is an *agent* it wants a *role*; once it has a role it wants a *responsibility*; a responsibility wants a *boundary*. Four words later the architecture is decided. The word was chosen; nothing after it was.

What makes this one durable is that it has real evidence behind it, applied at the wrong layer. Persona prompting works: telling a model to answer as a sceptical security reviewer produces different and often better output than asking neutrally. Every practitioner has seen it. That is a fact about conditioning a distribution inside one context. It is not evidence that the *system* should be split into security-reviewer-shaped components with their own contexts and hand-offs between them. The first is prompt engineering and it pays for itself. The second is architecture and it costs. Conflating them is the most common form of this mistake, and the hardest to argue with, because the person making it can point at a result that is genuinely there.

*The readable trace.* A role-shaped system is easier to debug. When the trace reads Architect → Developer → Reviewer, an engineer can find the fault by scanning. When six voices revise a shared board for four hundred turns, they cannot. This is not management theatre. It is someone at 2am trying to locate a failure, and it is why people who accept every argument in this essay still ship role-shaped systems and are not being cowards about it. It also has the clearest exit: debuggability is a rendering problem, and restructuring the computation to make the log easier to read is an extravagant way to solve a rendering problem.

Three of the six buy something real — oversight, accountability, debuggability. None of the three requires the architecture you are paying for it with. The other three are barely trades. The primitive buys convenience; Conway's law and the metaphor buy nothing at all. And two of the six — your org chart and your framework's base class — were never chosen by anybody.

That is what makes the cage comfortable rather than merely wrong. Most of it was never a decision.

## The alternative

The debate about multi-agent AI oscillates between two poles: many narrow specialists communicating through serialised hand-offs, or one giant agent holding everything in a single monologue. The first loses information at every boundary. The second commits early, and nothing outside its own voice can challenge it. Both miss the target. The real distinction is not how many agents. It is whether cognition is *partitioned* or *shared*.

A shared cognitive workspace has three properties that matter:

1. **One state.** Every perspective reads and writes the same representation. No private contexts. No hand-offs that compress.
2. **Nobody finishes first.** A draft architecture can be revised after the implementation perspective has spoken, because the implementation perspective is in the same ongoing process, not downstream of it.
3. **It stops on convergence, not completion.** There is no "all roles done." It stops when no perspective can improve the state, or when the state crosses a quality bar.

Think of the difference between an assembly line, where each station does one thing and hands off, and an orchestra, where everyone plays from a shared score and nobody finishes before the music is over. Both have structure. Only one matches how a unified mind computes.

Notice what this is *not*. It is not "no personas." The sceptic, the domain expert, the synthesiser are still there — but they share one context, and none holds anything the others cannot see. The role-shaped version of the same system is a few lines of code away: give each perspective its own context, pass a summary between them, and the personas are unchanged while the architecture has become the thing this essay argues against. That difference — not the vocabulary, not the number of voices — is the whole distinction.

### The one wall worth keeping

The argument would be too easy if every boundary were scheduling. Some walls buy independence. Blind review works because reviewers cannot see each other. Two estimates beat one when they were formed separately. The value is not the partition; it is that the errors are uncorrelated.

That concern gets *worse* when the workers are models. Voices drawn from the same base model already share priors; give them identical context and the correlation rises. A workspace that converges quickly may be converging because every voice was primed the same way. Convergence then means agreement, not confirmation. This is the strongest technical objection to the architecture this essay recommends, and it should be stated as the bet it is: shared cognition is the claim, correlated error is its price, and the wager is that three cheap mitigations keep the price low — let each voice form its first position before it reads the board; sync late rather than often; and put a genuinely different model in the critic seat on the paths that matter.

So the rule is not *everyone sees everything*. It is narrower. **Keep the walls that buy independent error. Demolish the ones that only bought scheduling.** Almost every wall in the software lifecycle is the second kind.

## The trap you will fall into next

Most organisations will not rebuild. They will run a workspace underneath and expose a role-shaped view on top — "Architect said X, Reviewer said Y" — because the view satisfies the auditor and the architecture satisfies the engineers. That hybrid is the right compromise, and it degrades silently.

The mechanism: a stakeholder reads the trace and says, "The Architect never considered X. Make sure the Architect does Y." The team, wanting to be helpful, changes the underlying system so the Architect does Y. Each round of feedback makes the internals more like the projection. The view stops being a lossy rendering and becomes a specification. Within a few product cycles you are back to a role pipeline, now with the added cost of maintaining the projection.

This has already happened in the research literature. A 2026 paper on [deterministic blackboard pipelines](https://dl.acm.org/doi/10.1145/3816713.3818808) started from the classical shared board and found that participants firing opportunistically produced execution that was hard to trace. Their fix was to replace the scheduler with a fixed pipeline — keep the shared state, give up the dynamism, get legibility back. The traceability requirement did not sit beside the architecture. It reached in and re-sequenced it.

Three early signals that it is happening to you:

- **Traces look too clean.** Real deliberation has abandoned positions and mid-reasoning reversals. If every trace reads like a textbook workflow, either the projection has taken over, or the voices agreed before they started.
- **Nobody can say which reason a role serves.** Ask why a role exists. "Legibility," "liability," or "debuggability" are answers. A shrug, a framework class name, or a team name means the shape is being installed, not chosen — and it is being installed continuously, by every dependency upgrade and every re-org.
- **The system starts scheduling.** Queues, batches, phase boundaries, "let the critic wait until the draft is done." Scheduling is what you do when the workers cannot all be interrupted at once. Nothing here has that problem. It arrives dressed as cost control.

## What to do on Monday

None of this requires a rebuild. In order of cost:

1. **Audit the primitive.** Open the framework and find its base abstraction. If it is a role with a goal and a backstory, write down that the hand-off architecture was chosen by that library's authors. Changing it is a migration; knowing it costs an hour, and it changes what every subsequent design argument is actually about.
2. **Make each role name its reason.** Next to every role in the system, write whether it exists for oversight, for accountability, or for debuggability. The ones with no answer exist because the framework or the org chart put them there. They can go without a meeting. This is the only item on this list that removes work rather than adding it.
3. **Replace one hand-off with a board.** Take the pair of agents that pass the lossiest artifact between them and give them shared state instead. Measure what the second one now catches.
4. **Form first positions blind.** Before any voice reads the board, have each answer the bare task. Then let them read. One round of cost; the difference between convergence and agreement.
5. **Make the role labels views.** Keep the "Architect said / Reviewer said" trace for the people who need it. Generate it from the log of which voice spoke, mark it as lossy, and never let anyone edit the template.
6. **Evaluate on revision, not completion.** Reward the system for changing an earlier decision in light of later evidence. Treat a single-framing monologue as a warning sign — and treat fast, unanimous agreement the same way.

## The honest prediction

The field is unlikely to move on principle. It will move on catastrophe — a few visible failures of role-shaped systems in settings where a shared board would have caught the error. That prediction has a strange consolation built into it. A catastrophe would relax the two defensive reasons, because those are positions people hold and a failure changes what is defensible. It would do nothing to Conway's law or the shipped primitive, because nobody holds those positions. They get fixed by one person importing a different library and another drawing a team boundary somewhere else — or they don't get fixed at all.

The two causes least likely to be moved by events are the two easiest to move on purpose. That is where to start.

---

*Next: [Part 2 — The Emperor's Toolchain](./part-2-the-emperors-toolchain.md), which makes the same argument one layer down: at the databases, the sprints, the code review, and the tools your company actually runs on.*
