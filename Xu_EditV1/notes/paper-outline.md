# Working Paper Outline

## Tentative title

When Should Multi-Agent Systems Replace or Extend Retrieval-Augmented Generation in Education?

## Core contribution candidates

1. A framework for deciding when educational tasks are retrieval-dominant versus coordination-dominant.
2. A taxonomy for distinguishing classical RAG, agentic RAG, and non-RAG multi-agent educational systems.
3. An evaluation protocol and benchmark mapping comparing classical RAG, agentic RAG, and non-RAG multi-agent baselines.

## Section skeleton

1. Introduction
   - Problem framing
   - Limits of classical RAG in education
   - Why multi-agent coordination may help
   - Contributions

2. Background and Related Work
   - RAG for educational assistants
   - Agentic RAG and workflow systems
   - Multi-agent LLM systems
   - Intelligent tutoring and pedagogical dialogue systems

3. Framework
   - Task taxonomy
   - Agent roles and communication
   - Memory, retrieval, and tool access
   - Safety, calibration, and human oversight

4. Evaluation Design
   - Comparison families
   - Datasets and benchmarks
   - Metrics
   - Human evaluation in education

5. Evaluation Blueprint and Benchmark Mapping
   - Where classical RAG should remain competitive
   - Where agentic or multi-agent systems may outperform
   - Failure modes, benchmark limits, and cost tradeoffs

6. Discussion
   - Implications for educational deployment
   - Limits of current benchmarks
   - Reproducibility and risk

7. Conclusion

## Baseline families to include

- Classical RAG baseline
- Agentic RAG baseline
- Non-RAG multi-agent baseline
- Optional single-agent planning baseline

## Reviewer-sensitive areas

- Overclaiming replacement rather than scoped superiority
- Weak education-specific evaluation
- Missing cost/latency tradeoffs
- Failure to separate retrieval quality from orchestration quality
