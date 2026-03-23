# Grove — A Multi-Model Ensemble IDE Framework

> What if your IDE ran every coding task through multiple AI models simultaneously, then automatically selected or synthesized the best result?

This is an open idea / architecture proposal. I don't have time to build it. If you do, go for it — just credit the origin.

---

## The Problem

Every AI coding tool today uses a single model per request. You pick GPT-4o, Claude, or Gemini, and you get one answer. If that answer is wrong or suboptimal, you either prompt again or switch tools manually.

But models have different strengths. Claude is strong on reasoning and architecture. GPT-4o is fast and good at boilerplate. Gemini handles long context well. Running them in competition — and automatically picking or merging the winner — should produce better code than any single model alone.

The research backs this up. A 2025 paper ("Wisdom and Delusion of LLM Ensembles for Code Generation") showed the theoretical ceiling of a well-run ensemble is **83% higher** than the best single model. We're leaving enormous quality on the table.

---

## The Idea

A Cursor-like IDE that, for every coding task:

1. **Fans out** the request to N models in parallel (e.g. GPT-4o, Claude Sonnet, Gemini Pro)
2. **Evaluates** each output — via test execution, static analysis, and/or an LLM judge
3. **Selects or synthesizes** the best result automatically, without requiring the developer to vote
4. **Shows its work** — a diff view showing which model contributed what, and why

The developer writes code at normal speed. The ensemble runs in the background. The best result surfaces.

---

## Why This Doesn't Exist Yet

The closest product is **Windsurf Arena Mode** (Feb 2026), which runs two models in parallel and asks the *human* to vote. That's a step forward, but:

- It requires human judgment on every request — which doesn't scale
- It picks a winner rather than synthesizing the best parts of each output
- It doesn't use test execution to evaluate correctness — it relies on vibes

**Together AI's Mixture of Agents (MoA)** runs parallel proposers and uses an LLM to aggregate, but it's designed for prose, not code. Merging prose is easy. Merging code that must compile and pass tests is a harder, unsolved problem.

Academic work has gone further — Stanford's **CodeMonkeys** (Jan 2025) generates parallel trajectories, uses model-generated tests to vote, and achieves 66.2% on SWE-bench Verified, beating every individual model in the ensemble. But it's a research CLI tool, not a developer product.

The gap: **nobody has shipped this as a first-class IDE experience with automatic synthesis.**

---

## Proposed Architecture

```
Developer writes prompt
        |
        v
  Task Dispatcher
  /      |      \
GPT-4o  Claude  Gemini   <-- parallel, same prompt
  \      |      /
   Output Collector
        |
        v
   Evaluation Layer
   - Execute against tests (if available)
   - Static analysis (linting, type errors)
   - LLM judge scoring (correctness, style, security)
        |
        v
   Selection / Synthesis Engine
   - If one output clearly wins: apply it
   - If outputs are complementary: merge at the diff level
   - If ambiguous: surface top 2 to developer with scores
        |
        v
   IDE applies result + shows provenance diff
```

### The Hard Part: Code Synthesis

Selecting the best single output is straightforward. Synthesizing the best parts of multiple outputs is not — code has strict syntactic and semantic constraints that prose doesn't.

Possible approaches:
- **Diff-based merging** — treat model outputs as branches, use a structured merge algorithm
- **LLM-as-merger** — feed all outputs to a strong model with the instruction "combine the best elements of these implementations"
- **Function-level selection** — select the best implementation of each function independently, then compose

The right answer is probably a hybrid of all three depending on output similarity.

---

## What the Research Says

| Finding | Source |
|---|---|
| Diversity-based selection beats majority voting ("popularity trap") | arxiv 2510.21513 |
| Debate between models adds little; voting drives the gains | NeurIPS 2025, "Debate or Vote" |
| Test-as-judge is the right evaluation mechanism for code | CodeMonkeys, arxiv 2501.14723 |
| Self-MoA (one great model × N) can beat heterogeneous MoA | "Rethinking MoA", Feb 2025 |
| Ensemble ceiling is 83% above best single model | arxiv 2510.21513 |

Key implication: **don't use naive majority voting.** Select based on diversity and correctness signals, not consensus. The most popular output is not the best output.

---

## Open Questions

- What's the latency budget? Parallel fan-out adds wall-clock time if models respond at different speeds. Does the slowest model block the result?
- How do you handle stateful context (open files, project structure) across N simultaneous model calls?
- At what task granularity does ensembling make sense — per keystroke, per function, per PR?
- Is heterogeneous multi-model always better than self-consistency (same model × N)? Research suggests not always.
- How do you handle cost? Running 3 models per request multiplies API spend. Smart routing (only fan out on hard/ambiguous tasks) may be necessary.
- What's the right UI for provenance? Developers may not trust code they can't trace.

---

## Prior Art

| Project | What it does | Gap |
|---|---|---|
| Windsurf Arena Mode | Side-by-side model comparison, human votes | Human-in-the-loop, no synthesis |
| Together AI MoA | Parallel proposers + LLM aggregator | Prose-focused, not code-specific |
| CodeMonkeys | Multi-trajectory ensemble for SWE-bench | Research CLI, not a developer product |
| Cursor parallel agents | Task decomposition across agents | Not same-task ensemble |
| Agentless | Multi-sample generation + selection | Single model, no heterogeneous ensemble |
| RouteLLM | Routes queries to best single model | Selection, not ensemble |

---

## If You Want to Build This

Start here:
1. Use **Together AI's MoA** as the backend scaffolding — it handles parallel fan-out and aggregation
2. Build on top of **Cline** (open-source VS Code extension) for the IDE layer
3. Use **CodeMonkeys' test-as-judge selection** logic for evaluation
4. The synthesis/merge engine is the research contribution — that's the novel hard part

Key papers:
- [Wisdom and Delusion of LLM Ensembles for Code (2510.21513)](https://arxiv.org/abs/2510.21513)
- [CodeMonkeys (2501.14723)](https://arxiv.org/abs/2501.14723)
- [Mixture-of-Agents (2406.04692)](https://arxiv.org/abs/2406.04692)
- [Debate or Vote, NeurIPS 2025](https://openreview.net/forum?id=iUjGNJzrF1)
- [Enhancing LLM Code Generation with Ensembles (2503.15838)](https://arxiv.org/pdf/2503.15838)

---

## Status

This is an open proposal, not an active project. The competitive research was done in March 2026.

If you build something based on this, open an issue or reach out — happy to discuss.
