# Paper Polish Summary - MAE Paper (2026-04-20)

## Overview
Completed comprehensive polish of `/Users/zladwu/Development/mae-paper-20260406/MultiAgentOL/main_smc.tex` with four parallel section-specific editors plus two reviewer simulations.

## Critical Evidence Reconciliation

### Problem Identified
The manuscript contained **stale quantitative claims** that conflicted with frozen experiment artifacts:
- Old EduBench table showed 17.87%-68.77% coverage with thousands of processed examples
- Frozen snapshot shows only 4.02%-13.31% coverage with hundreds of examples
- This mismatch threatened paper validity and reviewer credibility

### Solution Applied
**exp-editor** reconciled all quantitative claims with frozen artifacts from:
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/latex_tables_metrics_/smc2026_tables.tex`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/latex_tables_metrics_/smc2026_narrative_and_methodology.tex`

**Key changes:**
- Updated Table 1 (completion snapshot) with correct freeze-time values
- Updated Table 2 (quality-efficiency) with correct EduBench means
- Reframed EduBench as "partial trajectory evidence" throughout
- Added structural-zero and fallback-semantics caveats in limitations
- Softened robustness claims to match actual completion data

## Section-Specific Improvements

### 1. Related Work (rw-editor)
**Goal:** Transform from taxonomy list to gap-driven narrative

**Changes:**
- Benchmarks paragraph now frames the shift from correctness to pedagogy/adaptivity/safety
- RAG paragraph contrasts classical vs agentic retrieval, adds evaluation literature
- Multi-agent paragraph distinguishes general frameworks, retrieval-oriented work, and tutoring systems
- Final synthesis states the unresolved boundary this paper addresses

**Citations added:** chen2023benchmarkingrag, ru2024ragchecker, friel2024ragbench, rau2024bergen, zhang2024retrievalqa, wu2024autogen, li2023camel, hong2024metagpt, jones2026intellicode, wu2025cogevoedu

### 2. Methodology (method-editor)
**Goal:** Match implemented behavior exactly, avoid aspirational claims

**Changes:**
- Rewrote routing subsection with actual heuristics, dataset priors, and threshold (evidence≥0.35)
- Rewrote specialist-agents subsection matching AgentContext fields and parallel execution
- Rewrote retrieval subsection matching TF-IDF + char TF-IDF + optional SVD implementation
- Updated Algorithm 1 to mirror actual pipeline control flow including fallback at evidence≥0.45
- Rewrote evaluation paragraph with complete metric list and profile-aware semantics
- Removed any mention of unimplemented CBRH-2 as current method

**Claims softened:**
- "Cheapest architecture" → "controlled adaptivity"
- Removed heavyweight semantic retrieval claims
- Made critic explicitly optional and issue-triggered
- Tied fallback to empty retrieval + evidence≥0.45

### 3. Experiments (exp-editor)
**Goal:** Reviewer-safe reporting with precise caveats

**Changes:**
- Updated completion table: EduBench now 682/682, 2254/2257, 2104/2106, 1338/1344
- Updated quality table: EduBench means now 0.3384/0.7987/0.2419 (classical), 0.0000/0.9553/0.0302 (hybrid), etc.
- Reframed all EduBench prose as "subset-based trajectory evidence"
- Added explicit caveats: structural zeros, fallback semantics, profile-gated metrics
- Removed claims treating differing subsets as controlled head-to-head comparisons
- Softened robustness interpretation to "preliminary implementation-stability signals"

### 4. Framing Sections (frame-editor)
**Goal:** Stronger narrative with natural flow, no overclaiming

**Changes:**
- Abstract: Tightened opening, reframed as architecture-selection problem, kept claims within evidence
- Introduction: Sharper opening contrast (retrieval vs instructional decision), more direct prose
- Discussion: Aligned with reconciled evidence, maintained regime-dependent thesis
- Limitations: Updated to reflect 13.3% EduBench coverage, profile-specific zeros, partial subsets
- Conclusion: Softened EduBench language, kept TutorEval claims strong

**Messaging tension noted:** Paper hints at robustness from completion counts but evidence only supports "suggestive under shared infrastructure"

## Reviewer Simulation Results

### Reviewer 1: Results/Validity Focus
**Top criticisms identified:**

1. **EduBench comparison invalid** - different tiny subsets (4.0%-13.3%), yet architectural conclusions drawn
2. **TutorEval gains too small** - absolute deltas tiny (0.1115→0.1152), no significance tests
3. **Grounded overlap confounded** - structurally zero when retrieval disabled, builds conclusion into measurement
4. **Router under-specified** - no accuracy characterization, threshold 0.45 appears arbitrary
5. **Parity controls incomplete** - different agent counts, optional critics, not capacity-matched
6. **EduBench metrics degenerate** - EM/Token-F1/choice all 0.0, weakens architectural conclusions
7. **Robustness numbers inconsistent** - conclusion cites different counts than main table
8. **Limitations contradict Table 1** - says 68.8% coverage but table shows 4.0%-13.3%
9. **No ablations** - defers critical ablations to future work
10. **Overclaiming** - phrases like "best balance" stronger than evidence supports

### Reviewer 2: Novelty/Positioning Focus
**Top criticisms identified:**

1. **Novelty incremental** - looks like tutoring-specific combination of existing agentic-RAG + multi-agent
2. **Router under-specified** - no feature definition, thresholds, training procedure, validation
3. **Missing ablations** - excludes agentic_rag, postpones key ablations
4. **EduBench too weak** - 4.0%-13.3% coverage, non-identical subsets, not controlled comparison
5. **Conclusions overreach** - broader lessons from one converged benchmark + one partial snapshot
6. **Related work needs sharpening** - unclear material difference from existing retrieval-control/planner-router designs
7. **Citation support thin** - "retrieval-first prevailing assumption" not well substantiated
8. **Retrieval method weak** - TF-IDF vs modern dense retrieval, no justification
9. **Fairness unclear** - hybrid/non-RAG use more preprocessing, confounds architecture vs module count
10. **Internal inconsistency** - conclusion robustness numbers don't match freeze-time table
11. **Metric interpretation needs caution** - grounded overlap structurally favors certain families
12. **No statistical reliability** - no confidence intervals, significance tests, variance estimates

## Files Modified

- `/Users/zladwu/Development/mae-paper-20260406/MultiAgentOL/main_smc.tex` (comprehensive revisions)

## Files Referenced for Evidence

- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/latex_tables_metrics_/smc2026_tables.tex`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/latex_tables_metrics_/smc2026_narrative_and_methodology.tex`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/orchestration/pipelines.py`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/ml/regime_router.py`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/retrieval/index.py`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/retrieval/reranker.py`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/evaluation/metrics.py`
- `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/src/eduagentic/datasets/adapters.py`

## Next Steps (Recommended)

### High Priority
1. **Compile paper** - Verify LaTeX builds cleanly with updated bibliography
2. **Address reviewer concerns** - Consider adding:
   - Matched-subset analysis or significance tests for TutorEval
   - Router accuracy/threshold sensitivity analysis
   - At least one critical ablation (disable conditional retrieval in hybrid)
   - Explicit capacity-matching discussion or caveat

### Medium Priority
3. **Strengthen positioning** - Add 1-2 sentences distinguishing from closest agentic-RAG tutoring systems
4. **Add statistical rigor** - Bootstrap confidence intervals for TutorEval deltas
5. **Clarify metrics** - Add short paragraph explaining why grounded overlap is retrieval-sensitive by design

### Lower Priority
6. **Consider external citations** - Evaluate adding recent 2026 work identified during research:
   - SafeTutors (Hazra et al., 2026) - pedagogical safety benchmark
   - EduNaija AI Tutor (Odeajo et al., 2026) - hybrid multi-agent RAG tutor
   - Agentic RAG survey (Singh et al., 2025) - conditional retrieval framing

## Team Performance

All four section editors completed their assignments successfully:
- **rw-editor**: Related work enriched with gap framing ✓
- **method-editor**: Methodology grounded in implementation ✓
- **exp-editor**: Experiments reconciled with frozen evidence ✓
- **frame-editor**: Framing sections polished for narrative flow ✓

Two reviewer simulations provided actionable criticism for future revision rounds.

## Status

Paper is now **evidence-consistent** and **reviewer-safer**, but still has vulnerabilities around:
- EduBench subset validity
- Statistical significance
- Router specification
- Ablation completeness

The manuscript is ready for internal review or can proceed to compilation and submission with known risks documented above.
