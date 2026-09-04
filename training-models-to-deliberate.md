# Training Models to Deliberate

*Part 3 of a series on AI reasoning architecture. [Part 1](./shared-cognitive-workspaces.md) laid out the problem statement. [Part 2](./beyond-the-org-chart.md) mapped the strategic paths. Part 3 zooms into Path 3 — the one with the highest ceiling and the longest timeline.*

---

Part 2 listed six paths forward. Path 3 — training models natively for multi-perspective deliberation — was the one with the highest potential upside and the longest timeline. It is also the path where the most interesting research questions live, and the path most likely to be ignored until the easier paths stop working.

This post is a closer look at what "training models to deliberate" actually means, what is missing from current approaches, and where the field needs to go.

## What we mean by deliberation

When we say a model should "deliberate," we mean something specific. Not just chain-of-thought — that is linear, single-voice reasoning with no genuine perspective diversity. Not just self-consistency — that is multiple samples with a final vote, but the intermediate reasoning is not shared.

Deliberation, in the sense that matters here, is:

- Multiple perspectives operating on shared state
- Each perspective able to revise its own position in light of others
- No perspective "finishes" before the others have weighed in
- Termination by quality threshold or convergence, not by completion

This is what multi-agent debate approximates. It is what Tree of Thoughts approaches in narrow settings. It is what most LLM reasoning today does *not* do natively.

The claim of Path 3 is that we should train models to do this by default — so that the prompt-time architecture is no longer the load-bearing element. The model arrives at workspace-style reasoning on its own, regardless of how it is prompted.

## The gap between current methods and what we need

Most current alignment work optimizes for output quality under a particular prompting regime. RLHF rewards outputs that human raters prefer. Constitutional AI rewards outputs that match a set of principles. DPO and its variants reward outputs that match preference pairs.

These methods have produced remarkable results. They have not produced deliberative models. The reason is structural: they grade the final answer, not the path to it. A model can produce a good answer through shallow heuristics, lucky sampling, or memorized patterns. The training signal does not distinguish between "this model reasoned carefully and arrived at the right answer" and "this model pattern-matched its way to a correct-looking answer."

To train for deliberation, the loss function needs to be sensitive to the quality of the reasoning process itself. We do not currently know how to do this.

## Loss functions for process quality

The naive approach: have humans grade reasoning traces. This has obvious scaling problems. Reasoning traces are long, dense, and require expert evaluation. Even with scalable oversight techniques — debate, recursive reward modeling, weak-to-strong generalization — the underlying signal is human judgment of process, which is expensive and inconsistent.

The more interesting approach: train against the consistency of the final answer under different deliberation conditions. If a model gives the same answer whether it deliberates alone or with adversarial perspectives, that is one signal. If a model changes its answer only when given legitimate counter-evidence, that is another. We can construct training signals from these properties without needing humans to grade the reasoning itself.

The still-more-interesting approach: train models to evaluate their own reasoning. A model trained to detect when its own chain-of-thought has committed prematurely, or failed to consider an obvious counter-argument, can be used as a critic during training. The critic is itself a model, but it provides denser process-level signal than human raters can.

None of these is solved. The field is at the stage of "we know what the loss function should be sensitive to, we do not yet have a way to compute it."

## Training data: where does it come from?

Most training corpora are monologic. Internet text, books, articles, code — all written by individuals expressing single positions. Deliberative corpora are rare.

We do have some sources:
- Transcripts of structured debates, judicial opinions, peer review
- Multi-party dialogue datasets (some meeting transcripts, legal records)
- Generated data from multi-agent systems (model debates with itself)

The first two are limited in scale and domain. The third is circular if we want to train the underlying model — we are asking the model to provide its own training signal, which works only if the model is already capable of generating high-quality deliberation.

Realistically, we need either much better sources of human deliberation data, or breakthroughs in self-supervised process learning. Neither is imminent.

## Evaluation: grading a reasoning process

This may be the hardest problem. Even if we could train a model to deliberate, we cannot easily verify that it is deliberating well.

Output evaluation has standard metrics — accuracy, factuality, helpfulness, harm avoidance. These are imperfect but usable. Process evaluation has no equivalent. We do not have good metrics for:

- Whether a model considered an obvious counter-argument
- Whether the model revised a position in light of legitimate evidence
- Whether the model avoided premature convergence
- Whether the model's reasoning trace is faithful to its actual decision process (a separate problem — models often generate post-hoc rationalizations that do not match their actual reasoning)

Some research groups are working on faithfulness — whether chain-of-thought reflects the model's actual computation. This is foundational but unsolved. Without it, we cannot trust reasoning traces as either training signal or evaluation target.

## What is actually being attempted

A non-exhaustive map of where research is pointing:

**Process supervision.** OpenAI's "Let's verify step by step" work graded individual reasoning steps rather than final answers. Found that process supervision outperformed outcome supervision on math problems. The closest large-scale attempt at process-level training, though it is still grading steps, not deliberation.

**Constitutional AI and variants.** Anthropic's constitutional approach uses principles to guide self-critique. The model critiques and revises its own outputs against a constitution. Adversarial self-critique, a weak form of deliberation.

**Multi-agent debate training.** Some work is training models specifically to perform well in debate settings. The model is rewarded for producing arguments that survive adversarial probing. Still early, but promising.

**Recursive reward modeling.** Models trained to evaluate other models, recursively. The hope is that the evaluator becomes a process-level critic that scales beyond human raters.

**Weak-to-strong generalization.** Recent OpenAI work training strong models to imitate weak models' processes, with the hope that the strong model improves on them. Adjacent to the deliberative question rather than directly addressing it, but exploring related ground.

None of these is yet a deliberative training paradigm. They are components that might assemble into one. The risk is that they assemble into something that *looks* deliberative in benchmarks but isn't, the way that some RLHF'd models produce sycophantic outputs that look aligned without being so.

## Timeline and what to watch for

Realistically, full deliberation-as-default is a 5–15 year horizon. The research infrastructure is being built, but the core problems — process-level loss functions, deliberative training data, process evaluation, faithfulness verification — are all open.

What to watch for:
- A credible demonstration that a model revises its position when given legitimate counter-evidence in a way a base model does not
- Process supervision results that scale beyond narrow math domains
- Training paradigms that don't require human-labeled deliberation data
- Faithfulness work that gives us a way to verify reasoning traces match actual computation

If any of these land, Path 3 starts becoming a path practitioners can build on. Until they do, Path 3 remains frontier research with no clear shipping date.

## What this implies

**For frontier labs:** this is the path to invest in, even if it doesn't ship for years. The other paths buy incremental improvement. This one changes what is possible. The cost is tolerating long timelines with no immediate product wins, which is structurally hard for labs under commercial pressure.

**For practitioners:** don't wait for Path 3. The other paths are what you can build today. But do follow the research — when Path 3 lands, it changes the capability ceiling for everyone.

**For the field as a whole:** the biggest risk is that Path 3 doesn't arrive, and we have optimized Path 2 into a role-shaped system that performs like Path 1 with extra complexity. That would be a bad equilibrium — extra engineering cost, same capability ceiling, harder to debug. The defense against it is keeping the deliberative research funded even when the immediate shipping pressure points elsewhere.

---

*Previous: [Part 2 — Beyond the Org Chart](./beyond-the-org-chart.md) · Next: [Part 3.5 — LoRA as Deliberation Head](./lora-as-deliberation-head.md) (a near-term experiment scoped from this post) · or skip ahead to [Part 4 — The Hybrid Failure Mode](./the-hybrid-failure-mode.md)*
