# Framework Code Snapshot

This folder copies the implementation files that directly support the paper's
methodology description of MARLET. The original source repo is:

`/Users/zladwu/Development/mae-paper-20260406/MultiAgent`

Deployment endpoints and machine-local model paths in copied config snapshots
are redacted; framework parameters, budgets, and architecture switches are kept.

## Methodology Mapping

- Input objects, route decisions, agent outputs, retrieved chunks, and pipeline responses:
  `src/eduagentic/core/contracts.py`
- System wiring and architecture selection:
  `src/eduagentic/app.py`, `src/eduagentic/config.py`
- Supervisory routing, cue scoring, task regimes, retrieval gates, and visible task state:
  `src/eduagentic/ml/regime_router.py`, `src/eduagentic/ml/student_state.py`
- Parallel execution and controller variants:
  `src/eduagentic/orchestration/pipelines.py`, `src/eduagentic/orchestration/runtime.py`,
  `src/eduagentic/orchestration/swarm_bridge.py`
- LLM-backed specialist agents used to build the response brief:
  `src/eduagentic/agents/planner.py`, `diagnoser.py`, `rubric.py`, `retriever.py`,
  `tutor.py`, `critic.py`, and `base.py`
- Prompt assembly logic:
  `src/eduagentic/prompts/templates.py`, `src/eduagentic/agents/tutor.py`
- Retrieval, reranking, and context packing:
  `src/eduagentic/retrieval/corpus.py`, `index.py`, `reranker.py`, `packer.py`
- Dataset-entry normalization used by the experiments:
  `src/eduagentic/datasets/base.py`, `adapters.py`, `registry.py`
- Tokenization and text utilities used by routing/retrieval:
  `src/eduagentic/utils/text.py`
- Framework parameters and experiment budgets:
  `configs/system.example.yaml`, `configs/system.experiments.qwen4b.yaml`,
  `configs/system.experiments.qwen27b.yaml`

`MANIFEST.txt` lists every copied source/config file.
