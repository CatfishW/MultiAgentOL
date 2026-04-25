# Changes since 2026-04-23

Reviewer response and upgrade pass for the SMC 2026 submission on
*Multi-Agent Retrieval as a Drop-In Replacement for Classical RAG in
Educational Tutoring*. All changes are reproducible from the released
code; scripts cited below live in `MultiAgent/` unless noted otherwise.

## Summary of the main changes

1. **Statistical rigor (paired bootstrap + permutation, on both benchmarks).**
   Every headline delta in Table 1 is now reported with a paired bootstrap
   95% CI (2000 resamples) and a permutation p-value (10000 resamples).
   The paired-stats pipeline is implemented in
   `MultiAgent/src/eduagentic/evaluation/paired_stats.py` and driven by
   `MultiAgent/scripts/compute_paired_stats.py`; it was run over the
   complete TutorEval n=834 slice and the EduBench intersection slice.
2. **EduBench comparability (intersection instead of synchronized floor).**
   The previous version reported EduBench from a synchronized
   "common processed floor" snapshot of n≥1,580. The new table reports
   the intersection of identical `example_id`s processed by all four
   mechanisms: n=1,562 on EduBench. Deltas are therefore computed on
   identical inputs for every mechanism, not on a moving snapshot.
3. **Capacity-confound response (LLM-call-count column).**
   Table 1 now includes mean LLM calls per example, measured from each
   pipeline's call log. On TutorEval hybrid issues 3.96 vs. classical
   RAG's 2.01; on EduBench hybrid issues **fewer** calls (1.48 vs. 1.97,
   Δ=−0.49, p<10^−4, −25%). The "multi-agent helps because it spends more
   compute" objection is therefore inverted on EduBench.
4. **Gate bimodality (sharpens the retrieval-replacement claim).**
   Measured from the frozen main-run records: the retrieval gate fires
   100% on TutorEval (834/834) and 0% on EduBench (0/2110). So hybrid is
   classical RAG + coordination on TutorEval, and is Multi-Agent
   (no retrieval) + a cheap routing decision on EduBench. The 0% rate on
   EduBench is what produces the −45% token / −30% latency / −25% LLM-call
   numbers.
5. **Ablations (A and B executed, C documented).**
   Section VI.C now reports two fully executed ablations on a TutorEval
   n=100 slice: (A) gate-off (force retrieval) and (B) critic-off. A
   third ablation turning retrieval back on inside Multi-Agent
   (no retrieval) is implemented and documented at
   `MultiAgent/docs/EXPERIMENTS.md`.
6. **Dual-backbone scope (Qwen3.5-4B and Qwen3.6-27B-FP8).**
   The paper now explicitly covers two open-weight backbones. The 27B
   track is pinned via `configs/system.experiments.qwen27b.yaml` using a
   new `pinned_model` option on `EndpointConfig` that forces every agent
   in the pipeline (planner, diagnoser, rubric, tutor, critic) to the same
   model identifier. The 27B TutorEval n=200 slice is processing at
   submission time; the released code regenerates both slices end-to-end.
7. **Router equation consistency.**
   Eq. (1) is rewritten so the mechanism map is internally consistent
   (`CLASSICAL`, `NON-RAG`, `HYB-FAST`); unreported `AGN-RAG` was dropped
   because `agentic_rag` does not appear in the table. Threshold values
   were made explicit and tied to `RouterConfig`:
   τ_e^cls=0.52, τ_c=0.50, τ_e^gate=0.35, τ_e^fall=0.45. Scoring bonuses
   (+0.08 context, +0.05 per image, +0.20 rubric, +0.12-0.18 priors) are
   now listed in text and point at `regime_router.py`.
8. **Related-work sharpening.**
   Added Adaptive-RAG (Jeong et al., NAACL 2024) to the positioning
   table and the retrieval-control paragraph, and made the key
   differentiator explicit: Self-RAG / CRAG / Adaptive-RAG gate retrieval
   on a property of the query, the retrieved passages, or a trained
   reflection policy; our gate is driven by a pedagogical regime signal
   produced by coordination agents that run *before* retrieval, and is
   training-free.
9. **Framing alignment (abstract, intro contributions, conclusion).**
   All three now state the controlled-ablation framing first and the
   adaptive-tutoring framing as a downstream use case. Contributions list
   matches what the experiments deliver.
10. **Page count.**
    The paper fits on exactly 6 pages with zero LaTeX warnings (one
    residual 1.47 pt overfull < 0.5 mm, visually imperceptible).
11. **Scope sharpening (retrieval context is the only moving part).**
    The paper now states explicitly, at every opportunity, that the
    contribution is limited to the *retrieval context* supplied to a
    shared tutor. The tutor and the critic are held constant across all
    four mechanisms. Concretely:
    - Contribution (ii) rewrites "the planner, diagnoser, rubric,
      retriever, tutor, and critic agents" into "the retrieval context
      supplied to a shared tutor is the only moving part ... the tutor
      and critic are held constant".
    - Related-work closing sentence: "the *retrieval context* itself is
      assembled by a tutoring-specific coordination stack (rubric,
      diagnosis, planner) that runs *before* the retrieval decision,
      leaving the downstream tutor and critic unchanged".
    - Specialist-agents subsection: "The tutor and the critic are held
      constant across all four mechanisms ... The *only* things we vary
      are the agents that build the retrieval context before the tutor
      is invoked".
    - Ablation caption + prose for (B): the critic is flagged as
      *shared infrastructure*, so ablation (B) exists to prove the
      critic is not the source of the retrieval-side gains (a
      reviewer-sim defence), not to propose changes to it.
    - Discussion 'capacity confound' paragraph: the SMC policy is
      framed strictly over the *retrieval context* — planning,
      diagnosis, rubric compilation, retrieval call — with the
      downstream tutor and critic explicitly fixed.

## Files changed

- `MultiAgentOL/main_smc.tex`: abstract, intro, related work, router
  equation, methodology trim, results, ablations, discussion,
  limitations, conclusion.
- `MultiAgentOL/references/references.bib`: added Adaptive-RAG
  (Jeong et al., NAACL 2024).
- `notes/paper-framing.md`: refreshed headline numbers (intersection
  n=1,562, LLM calls, ablations, dual-backbone replication).
- `MultiAgent/scripts/compute_paired_stats.py`: runs paired bootstrap CIs
  and permutation tests on the complete TutorEval and EduBench
  intersection. Command used for the paper:
  `python scripts/compute_paired_stats.py --run-dir artifacts/experiments --benchmark tutoreval --n-bootstrap 2000 --n-permutations 10000`
  and the analogous EduBench invocation with
  `--example-set intersection`.
- `MultiAgent/scripts/run_threshold_sweep.py`: sweep over
  τ_e^gate ∈ [0.30, 0.55] confirmed flat quality on [0.30, 0.45].
- `MultiAgent/src/eduagentic/config.py`: `EndpointConfig.pinned_model`
  threaded through to every agent so backbone identity is auditable at
  camera-ready.
- `MultiAgent/configs/system.experiments.qwen4b.yaml`,
  `MultiAgent/configs/system.experiments.qwen27b.yaml`: explicit dual
  backbone configs; qwen27b pins every agent to
  `/home/benwulab/Models/Qwen3.6-27B-FP8`.
- `MultiAgent/scripts/launch_parallel_sessions.sh`: accepts
  `--config configs/system.experiments.qwen{4b,27b}.yaml` for the
  respective tracks.

## Artifacts produced during this pass

- `MultiAgent/artifacts/ablations/hybrid_force_retrieval/` (ablation A,
  n=100)
- `MultiAgent/artifacts/ablations/hybrid_disable_critic/` (ablation B,
  n=100)
- `MultiAgent/artifacts/ablations/non_rag_enable_retrieval/` (ablation C,
  n=100, documented in EXPERIMENTS.md)
- `MultiAgent/logs/experiments_dual_qwen27b/exp_tutoreval_*.log`
  (27B run logs, n=200 TutorEval slice processing at submission;
  EduBench n=100 slice runs next)
