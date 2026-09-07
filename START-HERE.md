# Start Here

Most AI agent systems are built like a small company: a planner hands off to a coder, who hands off to a reviewer. That shape was never discovered. It was inherited — and it is costing more than anyone is measuring.

The org chart is [a compression scheme for scarce attention](https://github.com/matteox/Comfortable-Cage/blob/main/shared-cognitive-workspaces.md#what-the-org-chart-was-for). A person can be in one place at a time, interruption is expensive, and asking a busy colleague a question is never free. Roles, hand-offs, summaries, review gates, standups, sprint boundaries, seniority — every one answers the same question: *who has to know what, given that knowing costs someone an hour?*

For a model, that hour costs nothing. Interruption is free. Simultaneity is free. Repetition is free. Something is still scarce — context, not calendar — but that scarcity wants a different architecture: relevance selected per question, not ownership partitioned in advance.

We keep drawing the old cut, and [not for one reason](https://github.com/matteox/Comfortable-Cage/blob/main/shared-cognitive-workspaces.md#why-the-cage-is-comfortable). Two are defensive: a role-shaped system is legible to whoever approves the work, and it gives blame a name to land on. Two are structural: your own org chart reproduces itself in what you build, and the framework you built it on already shipped someone else's. Two are cognitive: *agent* is a person-word that drags roles in behind it, and a role-shaped trace is one an engineer can actually read at 2am. Three of the six buy something real. None of the three needs the architecture we pay for it with, and two of the six were never chosen by anybody.

The bill is already visible. [Roughly four in five multi-agent failures](https://arxiv.org/abs/2503.13657) trace to how roles were specified and how hand-offs broke, not to model capability. Shared-board systems [beat role-based baselines by 13–57%](https://arxiv.org/abs/2510.01285) on published benchmarks. And [about sixty cents of every dollar](https://github.com/matteox/Prometheus/blob/main/what-ai-first-actually-means.md#sidebar-what-the-translation-costs) an agent spends goes to translating between human-shaped formats rather than to reasoning — a tax nobody optimizes because nobody measures it.

The clearest specimen: a sub-agent works with full context, then returns a paragraph to its orchestrator. That is a status report. It exists because a director cannot read every diff. Here, the orchestrator could have read everything.

Two series follow. [The first](https://github.com/matteox/Comfortable-Cage) diagnoses the reasoning layer, [the second](https://github.com/matteox/Prometheus) the tools, and [the last post](https://github.com/matteox/Prometheus/blob/main/leaving-the-cage.md) draws the alternative.

The cost of not changing is not a missed opportunity. It is a ceiling you pay for, in full, forever, while calling it process.
