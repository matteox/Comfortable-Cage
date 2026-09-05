# Consolidation Changelog — The Comfortable Cage

Restructured from 10 files (README plus nine posts) to 6 (README plus five
posts). Word count across the posts went from 15,503 to 8,750 — a 44% cut —
with no argument dropped and every verified citation carried over.

This is a lighter cut than the Prometheus consolidation (74%) because the
repetition was localized rather than pervasive: Parts 1, 2, 4, and 5 each had
a distinct structure and were trimmed, not rewritten. The bulk of the savings
came from one merge.

## Why

Measured across the nine original posts:

| Signal | Count |
|---|---|
| Intros re-narrating the series thesis ("Part 1 argued that...") | 9 of 9 |
| Closing "What this implies / What this means for whom" section | 5 of 9 |
| "What's tractable vs what's the moonshot" table | 3 of 9 |
| "How this could produce deliberative behavior" section | 3 of 9 |

The repetition was concentrated in Part 3 and the 3.5–3.8 sub-series: five
posts, 7,550 words, half the series. Each of the four "experiment" posts
re-derived the deliberation objective, the training-data problem, the
faithfulness limitation, and the evaluation gap that Part 3 had already set
up, then closed on the same three-tier horizon table. The sub-series format
forced each post to stand alone, so each re-explained the shared premise.

## Old → new mapping

| Old file | Now |
|---|---|
| `README.md` | `README.md` (rewritten for five parts; adds a companion-series pointer) |
| `shared-cognitive-workspaces.md` (Part 1) | Same file, rewritten — see "Part 1" below |
| `beyond-the-org-chart.md` (Part 2) | Same file, trimmed |
| `training-models-to-deliberate.md` (Part 3) | Same file, **merged** with the four below |
| `lora-as-deliberation-head.md` (3.5) | Section 1 of Part 3, "A LoRA deliberation head" |
| `process-reward-models.md` (3.6) | Section 2 of Part 3, "A process reward model at decoding time" |
| `inference-time-constitutional-ai.md` (3.7) | Section 3 of Part 3, "Inference-time constitutional critique" |
| `learned-routing.md` (3.8) | Section 4 of Part 3, "A learned router" |
| `the-hybrid-failure-mode.md` (Part 4) | Same file, intro and nav trimmed |
| `a-workspace-in-code.md` (Part 5) | Same file, intro and nav trimmed |

All five surviving filenames are unchanged, so external links to Parts 1–5
keep working. The four 3.x files are superseded and should be deleted, or
left with a one-line pointer to Part 3 if inbound links matter.

## Part 1 — Persistent Shared Cognitive Workspaces (2,792 → 2,091 words)

- **Opens with the Ford passage** (option A from the discussion): the quote
  stands with a clean attribution, the reveal arrives a beat later in prose,
  closes on "Sounds good, though.", then bridges — "Hold on to that feeling...
  Then look at how AI agent systems are built." This is the passage's
  intended home; Prometheus now carries a two-line callback instead (see
  "Cross-series change").
- The thesis pull-quote is folded into a "The thesis" section rather than
  standing as a jargon H2 ("Beyond Specialization vs. Monolithization")
  immediately under the title.
- The nine-item "Existing Research" bullet list is condensed to one
  paragraph carrying all the same citations (Wei, Wang, Yao, Besta, Du, Bai,
  Shinn, Park, Wang) — a survey, not a reading list.
- The four architectural patterns are tightened and now point forward to
  Part 5 where each is shown in code, instead of re-describing what Part 5
  will cover.
- The "Implications" section (for training / deployment / evaluation) is
  replaced by a two-sentence bridge that names what Parts 2–5 each take up.
  The material itself now lives in those posts rather than being previewed.
- One faster-horse line added after the SDLC-artifact table: "Faster horses,
  all the way down."

## Part 2 — Beyond the Org Chart (1,610 → 1,120 words)

- Intro no longer re-narrates Part 1's conclusion; one sentence.
- "Three families of response" folded into the intro as a single sentence.
- "The map" table cut — it re-summarized the six paths the reader had just
  finished.
- "What this means for whom" kept as the series' one audience-facing
  closing, but condensed from four paragraphs to four sentences each.
- Paths 2 and 3 now point forward to Parts 4 and 3 respectively, replacing
  the old footer's role.
- "The honest prediction" kept intact, including the series' one retained
  "honest" close.

## Part 3 — Training Models to Deliberate (7,550 → 2,042 words, five posts → one)

- The problem is stated once: what deliberation means, why answer-grading
  methods can't produce it, and the three open problems (loss functions,
  data, evaluation/faithfulness).
- "What is actually being attempted" reduced from five bolded paragraphs to
  one paragraph naming the same five research directions.
- The four sub-series posts become four sections of roughly 300–400 words
  each, ordered as they were (adapter → separate critic → self-critic →
  orchestrator) with the one-sentence framing "each adds one architectural
  element to the last."
- The LoRA section keeps the mechanism, the four-stage pipeline (compressed
  to one paragraph), both caveats (distillation ceiling; over-deliberation
  and the 30–50% decisive-trace mix), and the MoE-vs-adapters observation.
  The numbered pilot protocol is folded into the pipeline paragraph.
- The PRM section keeps the four inference mechanisms, the deliberative-PRM
  data gap, Math-Shepherd / OmegaPRM, and all five limitations, condensed.
- The constitutional section keeps the four example principles, the 2N+1
  cost arithmetic, and the same-model-bias / gaming / latency limits. The
  Python loop is dropped here because Part 5 already shows the
  adversarial-critique pattern in code.
- The routing section keeps the decision set, the token-vs-adapter
  granularity argument, and the gains/losses including the
  single-point-of-failure point.
- **One horizon table** (tractable / frontier / moonshot) covering all four
  experiments replaces the three separate tables in 3.6, 3.7, and 3.8 plus
  the "if it works / if it fails" section in 3.5.
- "What to watch for" and the bad-equilibrium closing are kept from the
  original Part 3.
- The 5–15 year horizon appears once.

## Part 4 — The Hybrid Failure Mode (1,797 → 1,768 words)

- Intro breadcrumbs replaced with one line; opening two paragraphs merged.
- Navigation fixed: "Previous" now points to Part 3 instead of the removed
  Part 3.8.
- Otherwise unchanged. This was the most distinct post in the series and the
  best concrete material (the five drift signals); it didn't need cutting.

## Part 5 — A Workspace in Code (1,751 → 1,729 words)

- Intro breadcrumbs replaced with one line.
- Otherwise unchanged; the code patterns had nothing to merge with.

## Cross-series change (option A)

Prometheus Post 0 (`the-emperor-has-no-clothes.md` in the Prometheus
consolidated set) no longer carries the full Ford passage. It now opens with a
two-line callback — "The first series opened with a quote Henry Ford never
said. It sounded right, nobody checked, and by the end of that series the
reader had one question to carry" — and picks up the lens from there. The
full passage lives once, at the start of Part 1 of this series, which is
where Prometheus's own framing ("carries the Ford quote forward") says it
should be.

## Not changed

- All citations and their descriptions.
- The four architectural patterns, the six strategic paths, the five drift
  signals and five disciplines, and the four code patterns — all present.
- The author's voice, including the two retained "honest" uses (Part 2's
  closing section and Part 4's closing line).
- The three ASCII diagrams added in the earlier editing pass (Part 1's
  three-way cognition diagram, Part 4's drift diagram, Part 5's composition
  pipeline).
