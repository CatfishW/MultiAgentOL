# Research Log

## Topic

Replacing or extending retrieval-augmented generation with multi-agent systems for educational applications.

## Research streams

### 1. Core framing

- What kinds of tasks actually need more than retrieval?
- When does orchestration outperform single-agent RAG?
- Where does multi-agent complexity fail to justify itself?

### 2. Educational applications

- Intelligent tutoring and pedagogical dialogue
- Feedback generation and assessment support
- Study planning, coaching, and classroom support
- Tool-using educational agents and workflow systems

### 3. Evaluation design

- Education-specific datasets and benchmarks
- Transfer benchmarks for grounded reasoning and tool use
- Fair comparisons among classical RAG, agentic RAG, and non-RAG multi-agent systems

## Evidence capture template

| Item | Type | Claim | Source URL | Metrics / Benchmark | Notes |
|---|---|---|---|---|---|
| TBD | paper/system | TBD | TBD | TBD | TBD |

## Open concerns

- Avoid overclaiming that multi-agent universally replaces RAG.
- Separate retrieval-heavy tasks from planning-heavy or interaction-heavy tasks.
- Ensure any education evaluation goes beyond generic QA accuracy.

## Confirmed benchmark leads

- EduBench: education-specific, multi-scenario pedagogical evaluation with 12 dimensions.
- TutorEval / LM-Science-Tutor: open-book versus closed-book tutor QA, useful for retrieval-sensitive educational comparison.
- ScienceQA: multimodal science QA with explanations and lectures; useful but not a pure tutoring benchmark.
- HotpotQA: strong transferable benchmark for multi-hop retrieval and evidence justification.
- AgentBench: strong transferable benchmark for tool use, workflow coordination, and agent environments.

## Benchmark-design implications

- Education-specific evaluation should not rely on factual QA alone.
- Transfer benchmarks are still useful when they isolate retrieval, justification, or tool-coordination behavior.
- The benchmark suite should combine at least one education-first benchmark with at least one retrieval-stress benchmark and one coordination-stress benchmark.
