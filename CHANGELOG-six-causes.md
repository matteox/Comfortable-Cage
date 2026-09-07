# Revision: the org chart persists for six reasons, not one

The series previously gave two reasons the org chart gets copied into AI systems
— legibility and liability — and the recent attention-scarcity framing risked
collapsing even those into a single cause. This revision separates the causes,
adds four that weren't named, and threads them through both series so that
diagnosis, strategy, practice, and code all sort by the same list.

**Base for this revision: the published state of both repos as of 7 Sept 2026.**
The attention-scarcity edits from the earlier session were never pushed and do
not exist in any file here — see "Merge note" at the end.

## The six

| # | Reason | What it buys | Group |
|---|---|---|---|
| 1 | Legibility — the pipeline is a UI for management | Real oversight | Defensive |
| 2 | Liability — someone has to sign the thing | Real accountability | Defensive |
| 3 | Conway's law — you ship your org chart | Nothing | Structural |
| 4 | The shipped primitive — the framework's role class | Convenience | Structural |
| 5 | The person-metaphor — *agent* is a person-word | Prompt-level gains, misapplied | Cognitive |
| 6 | The readable trace — role-shaped traces debug easily | Real debuggability | Cognitive |

Attention scarcity is kept as the *origin* — why the org chart existed at all —
and explicitly separated from the six, which are why it survives its origin.
Three of the six buy something real; none of the three requires the architecture
currently paid for it; two were never chosen by anyone.

## Per-file changes

### The Comfortable Cage

| File | Change |
|---|---|
| `README.md` | Thesis paragraph rewritten around the six. Reading-order table updated (Part 1 "six reasons", Part 2 "seven paths", Part 4 "six drift signals, six disciplines"). One-paragraph argument rewritten. |
| `shared-cognitive-workspaces.md` | **Major.** "The legibility trap" and "The part that isn't admitted" are replaced by a new top-level section, *Why the cage is comfortable*, with an origin/persistence preamble, six numbered subsections, and a summary table with a *what to do* column. Legibility and liability text is preserved inside it as reasons 1 and 2. The SDLC table is now read as a table of attention limits. "What's unsolved" gains the point that four of the six sit outside the architecture. Closing sharpened. 2,375 → 3,764 words. |
| `beyond-the-org-chart.md` | Intro gains a cause→path map table, and the observation that each path treats exactly one cause. **New Path 7 — Change what installs the shape** (choose the primitive, draw interfaces before teams, keep the vocabulary honest). "Building systems today" gains the make-every-role-justify-itself test. The catastrophe prediction gains a qualification: a visible failure moves legibility and liability and does nothing to Conway or the primitive. |
| `training-models-to-deliberate.md` | New experiment: **separate persona from partition** — hold personas fixed, vary only context sharing. Falsifies or confirms reason 5, and nobody has run it. New scope paragraph: a natively deliberative model touches at most two of the six. |
| `the-hybrid-failure-mode.md` | Sixth discipline: *make every role justify itself*, quarterly. Two new anti-patterns: the framework picking the shape back up via a dependency upgrade, and a re-org redrawing the architecture. Drift signals 5 → 6, with *no one can say which reason a role serves* inserted at 3 as the earliest signal; diagram redrawn. "When to give up" gains the case where the framework primitive is a role and migration is refused. |
| `a-workspace-in-code.md` | New caveat before the patterns: the perspective lists are persona prompting used correctly, and the role-shaped version of the same code is a few lines away — the difference is shared context, not vocabulary. |

### Prometheus

| File | Change |
|---|---|
| `README.md` | Cross-series paragraph now names the six and points at what they explain here. |
| `the-emperor-has-no-clothes.md` | The cross-series paragraph is rewritten around the six with a deep link to the Cage section. **The method gains a fifth step: "Name the comfort."** |
| `the-rituals.md` | The engineering ladder is identified as the purest specimen of Conway's law — the org chart formalised into a career and exported into everything the org builds. The verdict now splits "comfort" into its four components rather than using it as one word. |
| `what-ai-first-actually-means.md` | Cross-series paragraph rewritten around the six. New paragraph: one principle diagnoses all of it, but nothing about the cure is single — each cause has a different remedy and a different timescale. |
| `leaving-the-cage.md` | Caution before the layer diagram: architecture cannot reach four of the six. Two new first moves — **audit the primitive** and **make each role name its reason** — the second being the only move on the list that removes work rather than adding it. |

### The intro

`START-HERE.md` gains one paragraph naming all six in three pairs, plus a deep
link to the Cage section. 292 → 398 words. This is over the 300 you set. The
paragraph is self-contained and cuts cleanly if you'd rather hold the length;
the cost of cutting it is that the intro then asserts the shape was "inherited"
without ever saying what keeps it.

## What's new rather than reorganised

Four claims that were not in either series before:

1. **Conway's law applies here more directly than it does to microservices**, because the artifact being shaped is itself an organization. Stated with a falsifiable prediction: you should be able to guess a framework's architecture from its vendor's org chart more often than chance.
2. **Framework path dependence is Conway's law one level up** — the role primitive is the vendor's org chart, shipped as an API. This is the cheapest of the six to fix and the least noticed.
3. **Persona prompting and role architecture are different claims** that get conflated because the first has real evidence. Proposed as a clean ablation in Part 3 and shown as a few lines of difference in Part 5.
4. **Debuggability is the honest reason**, named as such — role-shaped execution is an expensive solution to a rendering problem, but the need behind it is real, and the series previously had nothing to say to the engineer who holds it.

## Merge note — read before pushing

These files were rebuilt from what is currently published on GitHub. The
attention-scarcity revision discussed earlier (the "What the org chart was for"
section, the calendar/context distinction as a standalone section, the MoE
aside, the concurrency material in Parts 3–5) was never pushed and is not in
these files. What survives of it here is the origin/persistence preamble at the
top of *Why the cage is comfortable*, which carries the calendar/context
distinction in compressed form.

If you still want the full attention-scarcity treatment, it has to be re-derived
and merged with this — the two are compatible by design (origin vs. persistence)
but they both edit the same region of Part 1, so they cannot be applied blind.
Say the word and I'll rebuild it against these files rather than against the
published ones.
