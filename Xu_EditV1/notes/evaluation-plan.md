# Evaluation Plan Draft

## Architectural families

1. Classical RAG
2. Agentic RAG
3. Non-RAG multi-agent system
4. Optional single-agent no-RAG baseline

## Independent dimensions

- Retrieval enabled vs. disabled
- Single-pipeline vs. agentic or multi-agent orchestration

## Proposed task regimes

- Evidence-grounded educational QA
- Explanation with source-grounding requirements
- Rubric-based feedback on student work
- Adaptive tutoring or misconception diagnosis
- Lesson or exercise sequencing

## Metric families

- Groundedness and factual support
- Pedagogical quality
- Adaptivity to student context
- Robustness and failure recovery
- Cost, latency, and tool burden

## Fairness controls

- Shared base model family where possible
- Shared source corpus for retrieval-enabled conditions
- Shared prompts and task inputs at the interface level
- Budget reporting for tokens, tools, and latency

## Reviewer objections to preempt

1. Weak RAG baseline
2. Agent systems winning only because they spend more budget
3. Task suite biased toward agentic workflows
4. Pedagogical quality treated as equivalent to learning outcomes
5. Non-RAG systems being unfairly used on fact-heavy tasks without grounding

## Benchmark mapping principle

- Education-specific benchmarks support pedagogical or instructional claims.
- Transfer benchmarks support only mechanism-level claims about retrieval, grounding, long-context reasoning, or coordination.
- Do not let AgentBench or HotpotQA do the argumentative work of education-specific evaluation.
