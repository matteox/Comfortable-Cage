# Training Models to Deliberate

*Part 3 of The Comfortable Cage. This is Path 3 from [Part 2](./beyond-the-org-chart.md) — the one with the highest ceiling and the longest timeline.*

---

Of the six paths, training models natively for multi-perspective deliberation has the most upside, the most interesting research questions, and the highest chance of being ignored until the easier paths stop working. This post lays out the problem once, then four near-term experiments that each buy some of the outcome without waiting for the full solution.

## What deliberation means here

Not chain-of-thought — that is linear, single-voice reasoning with no perspective diversity. Not self-consistency — many samples and a vote, with the intermediate reasoning never shared. Deliberation, in the sense that matters, is multiple perspectives operating on shared state, each able to revise its own position in light of the others, none finishing before the rest have weighed in, and termination by convergence rather than completion.

Multi-agent debate approximates it. Tree of Thoughts approaches it in narrow settings. Most LLM reasoning today does not do it natively. The claim of Path 3 is that models should do it by default, so the prompt-time architecture stops being the load-bearing element.

## Why current methods don't get there

RLHF rewards outputs human raters prefer. Constitutional AI rewards outputs that match principles. DPO and its variants reward outputs that match preference pairs. All of them have produced remarkable results, and none has produced a deliberative model, for a structural reason: they grade the answer, not the path to it. A model can reach a good answer through shallow heuristics, lucky sampling, or memorized patterns, and the training signal cannot tell that apart from careful reasoning.

Training for deliberation means a loss function sensitive to the quality of the reasoning process itself. That opens three problems, none solved.

**Loss functions for process quality.** The naive approach — humans grade reasoning traces — doesn't scale; traces are long, dense, and need expert judgment. More promising: train against the consistency of the final answer under different deliberation conditions (does the model give the same answer alone and under adversarial pressure? does it change its answer *only* when given legitimate counter-evidence?). Most promising and least developed: models trained to detect when their own chain has committed prematurely or missed an obvious objection, used as critics during training. The field knows what the loss should be sensitive to and does not yet know how to compute it.

**Training data.** Nearly every corpus is monologic — text, books, code, all written by individuals holding single positions. Deliberative sources exist (structured debates, judicial opinions, peer review, some meeting transcripts) but are small and domain-bound. Generated data from models debating themselves is circular: it works only if the model can already deliberate well.

**Evaluation.** Even a model trained to deliberate cannot easily be verified to deliberate *well*. Output metrics — accuracy, factuality, helpfulness — are imperfect but usable. There is no process equivalent for "considered the obvious counter-argument," "revised in light of legitimate evidence," "avoided premature convergence." Underneath all of it sits faithfulness: whether a reasoning trace reflects the model's actual computation or a post-hoc rationalization. Until that's solved, traces can't be trusted as either training signal or evaluation target.

What's being attempted is components, not yet a paradigm. Process supervision — OpenAI's "Let's Verify Step by Step" — graded reasoning steps rather than answers and outperformed outcome supervision on math, though it grades steps, not deliberation. Constitutional AI uses principles to guide self-critique, a weak form of the adversarial pattern. Debate training rewards arguments that survive adversarial probing. Recursive reward modeling and weak-to-strong generalization explore adjacent ground. The risk is that these assemble into something that *looks* deliberative on benchmarks without being so, the way some RLHF'd models look aligned while being sycophantic.

Full deliberation-as-default is realistically a 5–15 year horizon. The four experiments below are what can be run now.

## Four near-term experiments

Each adds one architectural element to the last: an adapter on the base model, then a separate critic, then the base model as its own critic, then an orchestrator that decides among them. None is a substitute for Path 3. Each either fails informatively or produces a usable intermediate capability.

### 1. A LoRA deliberation head

LoRA freezes a base model's weights and learns a small low-rank perturbation — for a weight matrix *W*, a ΔW = *A·B* of rank 4 to 32, applied alongside the frozen *W*. Behavior becomes a switchable property: a 7B base with a 50MB adapter behaves differently from the same base without it, and adapters stack at inference.

The experiment: run the [Part 5](./a-workspace-in-code.md) patterns on a frontier model that *can* deliberate when carefully prompted, collect the interleaved traces (perspectives, revisions, evidence integration, mid-reasoning reversals), filter them with a judge model, and train a rank-16 adapter on the result. With the adapter active, the base model should produce interleaved, revising reasoning natively, with much lighter scaffolding. Evaluate on three axes: deliberative quality, decisiveness (does it still commit when warranted?), and capability preservation on standard benchmarks. Compare against the base with heavy prompting, the base with none, and the frontier source.

Two caveats. First, this transfers deliberation from a model that has it into a model that doesn't; it cannot exceed the source model's quality, and it inherits the source's biases and blind spots. That's a real capability shift — smaller, cheaper, less prompting — but it is distillation, not Path 3. Second, a model trained only on deliberative traces learns *always* deliberate, and a model that always revises is worse than one that revises when appropriate. The training mix needs 30–50% decisive, single-pass traces, and the evaluation needs cases where the right answer is to commit without deliberating.

There is an architectural side-effect worth noticing. Mixture-of-experts specializes by partitioning weights and routing between them. Composable adapters specialize by additive perturbation on shared base computation — soft, stackable, switchable, with no architectural partition. Whether that preserves MoE's efficiency is empirical, but it's a different answer to the question this series opened with.

### 2. A process reward model at decoding time

Leave the base model alone. Train a separate scorer for the quality of individual reasoning steps and use it to guide generation. Process reward models were first used as a training signal; what's changed is that they're now good enough to use at inference.

The mechanisms, in rising cost: best-of-N sampling (generate N chains, keep the highest-scoring); PRM-weighted voting; beam search with PRM-guided expansion; and lookahead — generate forward a step or more, score the resulting state, back off if the score degrades. Lookahead is the interesting mode for deliberation: the PRM asks "is this line of reasoning going somewhere?" at each step, and generation backtracks when it isn't. That's a learned version of the adversarial self-critique pattern, with a trained critic instead of a prompted one.

A PRM trained on deliberative traces would reward interleaving, revision, and evidence integration, steering the base toward deliberative chains because they score higher. The model isn't *knowing* how to deliberate; it's being steered by an external scorer, and the behavior disappears when the scorer does. But the scorer is doing work the base model can't do reliably on its own.

The data need is the hard part. Standard PRMs are trained on step-correctness labels; Math-Shepherd and OmegaPRM reduce annotation cost with synthetic or self-generated labels. A *deliberative* PRM needs labels at a higher level — when is revising a position responsive versus vacillating, when is acknowledging uncertainty a virtue versus a failure to commit — which nobody is collecting yet.

Limits: cost (best-of-10 is 10× inference; beam search of 5 over 20 steps is 100×), poor transfer across domains, evaluator bias inherited from whatever labeled the training data, and the base-model ceiling — a PRM improves *selection* among candidates, never generation itself. This will be deployed first where decisions are expensive enough to justify it: medical, legal, financial.

### 3. Inference-time constitutional critique

What if the critic is the base model, prompted to evaluate its own draft against explicit principles? No training, no separate model — the inference-time variant of Constitutional AI, and the cheapest of the four.

The principles encode the deliberation requirements: *before committing to a position, consider the strongest objection to it; if evidence emerges that contradicts an earlier claim, revise the claim; acknowledge uncertainty when the evidence doesn't support confidence; cite the basis for each load-bearing claim.* Generate, critique against the principles, revise, repeat until the critique reports no violations or the iteration budget runs out. Cost is 2N+1 calls per output — seven for three revisions. Roughly 7× a single-shot response, far cheaper than a PRM setup.

The limits are the obvious ones for a model grading itself. Same-model bias: it won't reliably find flaws it wouldn't have produced. Gaming: vague principles are satisfied superficially — "consider alternatives" by mentioning that alternatives exist, "acknowledge uncertainty" by adding hedges without changing the claim — so the principles must be specific enough that shallow compliance is detectable, which costs context. Latency: the iterations are sequential.

### 4. A learned router

The first three share a shape: a generator produces reasoning, and some evaluator shapes what gets produced. None addresses *structure* — which perspective to invoke when, when to switch from generation to critique, when to stop. Part 1 argued deliberation needs structure that emerges from the problem. Part 5 imposes structure via prompts. This experiment asks whether the structure can be learned.

A router is a small model that takes the current deliberation state and decides the next action: which perspective (skeptic, generator, synthesizer, domain expert), which adapter to activate, whether to invoke a critic, whether to terminate. It's trained on traces of good deliberation with the signal "at this decision point, what action led to better deliberation?" — process supervision at the structural level. The gains over hard-coded patterns: structure that adapts to the task (a simple question gets a simple deliberation), conditional steps ("if uncertain, invoke the skeptic"), and non-obvious orderings a designer would miss.

Two routing granularities matter. Token-level routing is classical MoE — routers inside the model choosing weight partitions per token. Adapter-level routing chooses which LoRA is active for a whole step. Deliberation steps are coherent units, so the coarser granularity fits better; a hybrid layers the two.

What's lost: interpretability (you can log which adapter fired, not why), robustness (learned routers fail unpredictably where hard-coded patterns fail legibly), and a new single point of failure — a bad router makes the deliberation bad even when every component is good. And the router needs the same deliberative traces the rest of this post can't find enough of.

## Horizons

| Horizon | What's realistic |
|---|---|
| Tractable now | Adapter-based deliberation heads distilled from frontier traces; PRMs for narrow domains with well-defined step quality (math, code, formal verification); best-of-N with PRM re-ranking for high-stakes decisions; inference-time constitutional loops where 5–10× cost is justified; hard-coded routers as a baseline |
| Research frontier | PRMs and critics trained on deliberative-quality labels rather than correctness; general-purpose PRMs that transfer across domains; routers trained on deliberative traces; faithfulness measurement at inference |
| Moonshot | Critics that improve generation rather than selection; self-improving loops where the model's own critiques and routing decisions become training data; critics that genuinely diverge from the generator's biases; routers that discover deliberation structures absent from their training data |

## What to watch for

- A credible demonstration that a model revises its position on legitimate counter-evidence in a way its base does not
- Process supervision that scales beyond narrow math domains
- Training paradigms that don't require human-labeled deliberation data
- Faithfulness work that lets us verify a trace matches the computation

If any of these land, Path 3 becomes something practitioners can build on. Until then, the four experiments above are what's available, and they are cheap enough to produce results in months.

The risk worth naming: that Path 3 never arrives, and the field optimizes the hybrid into a role-shaped system that performs like Path 1 with extra complexity. The defense is keeping deliberative research funded while the shipping pressure points elsewhere. The next post is about the other half of that defense — running the hybrid without letting it drift.

---

*Previous: [Part 2 — Beyond the Org Chart](./beyond-the-org-chart.md) · Next: [Part 4 — The Hybrid Failure Mode](./the-hybrid-failure-mode.md)*
