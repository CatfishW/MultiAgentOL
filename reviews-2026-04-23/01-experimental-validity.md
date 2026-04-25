# Review 1: Experimental Validity

**Reviewer**: Oracle Agent — Experimental Validity Focus
**Paper**: *Adaptive Tutoring Beyond Classical RAG: Conditional Retrieval in a Repository-Grounded Multi-Agent Study*
**Target**: IEEE SMC 2026
**Date**: 2026-04-23

---

## Overall Rating: **Weak Reject**

### Scores (1–10)

| Dimension | Score | Rationale |
|---|---|---|
| Clarity | 6 | Well-written, but router equations contain internal contradictions that create confusion |
| Soundness | 4 | EduBench subset problem remains unfixed; TutorEval deltas untested; ablations A–E committed but absent |
| Significance | 5 | Architecture-selection framing is useful; result is directionally interesting but not established |
| Originality | 5 | Incremental composition of known retrieval-gating and multi-agent patterns |

---

## Issues by Severity

### FATAL (1)

#### 1. EduBench subset comparability is unresolved — still invalid for architectural comparison (L180–183, Table 2)

The revision reframes EduBench as "synchronized snapshot at common processed floor n_min=1580" but still reports comparative claims:

> "hybrid and non-RAG multi-agent tie on composite (0.227) and EB12D mean (0.398), with hybrid slightly better rubric coverage and latency" (L42 abstract, L161 body)

This is architecturally meaningful language, not "directional trajectory" language, and it's in the abstract. The subsets are by definition non-overlapping. A common floor of 1,580 ensures each has *at least* that many runs, but says nothing about whether the examples were comparable in difficulty, topic distribution, or scenario mix.

**The "mitigation" is reframing, not fixing.** A referee cannot distinguish: (a) the hybrid genuinely matches multi-agent, from (b) the hybrid happened to get an easier slice of EduBench.

---

### CRITICAL (5)

#### 2. TutorEval deltas are too small without significance testing (L42, L159, Table 2)

Composite delta: 0.383 − 0.377 = **+0.006**. Token-F1 delta: 0.115 − 0.112 = **+0.003**. The paper claims "best composite" and "best Token-F1" (bolded in Table 2) without any confidence interval or p-value. Limitations admits this (L203 item ii) but abstract and results continue to present the deltas as clear wins.

#### 3. Ablations A–E are committed but entirely absent (L192)

Section 4.3 states: "we commit to a fixed set of controlled experiments." This is not an ablation study — it's a TODO list. For an empirical paper claiming that a conditional retrieval gate improves performance, ablations (A) hybrid without gate and (C) hybrid without critic are not optional; they are the minimal evidence that the claimed mechanism, not some confound, drives the result.

#### 4. Router equations contain internal contradictions (L112–117)

The piecewise function defines NON-RAG as activating when `s_c(x) ≥ 0.55`, but states `(τ_e^cls, τ_c) = (0.52, 0.50)`. The value 0.55 contradicts τ_c = 0.50. Similarly, `s_c(x) < 0.35` for classical RAG but 0.35 is also the retrieval gate threshold. The boundary between CLS-RAG and HYB-FAST is under-specified: if `s_e ≥ 0.52` and `s_c = 0.35`, where does it go? The "otherwise" catch-all swallows this but the boundary behavior is opaque.

#### 5. `corpus_factuality` is described but never reported (L142)

The evaluation protocol says this retrieval-agnostic grounding metric is "populated for every architecture using the same index." It does not appear in Table 2. If it was computed and is meaningful, report it. If it wasn't, this sentence reads as window dressing.

---

### MAJOR (5)

#### 6. Agentic RAG excluded without explanation (L97, L151)

"The implemented agentic_rag path is not included because parity-controlled outputs were unavailable." This is a red flag for cherry-picking. Why weren't parity-controlled outputs available?

#### 7. Identical tied scores suggest metric granularity loss (L182–183, Table 2)

EduBench composite: 0.227 for both hybrid and non-RAG. EB12D: 0.398 for both. These are suspiciously identical to 3 decimal places at n≥1,580. This either indicates insufficient metric resolution or aggressive rounding.

#### 8. No human evaluation (L203)

The paper commits to a 4×25 stratified human eval but hasn't run it. For tutoring systems, auto-metrics like Token-F1 have known weak correlation with pedagogical quality.

#### 9. Capacity matching asserted but not demonstrated (L134)

"All agents share the same small model backbone and shared budget policy, so differences across families reflect structural rather than capacity differences." But the families don't have the same number of agents, LLM calls, or preprocessing. The paper controls the *per-call* budget but not the *total* computation budget per example.

#### 10. TF-IDF retrieval with no justification (L138)

Using purely lexical retrieval for an LLM tutoring system in 2026 requires brief justification.

---

### MINOR (3)

#### 11. "evidence-grounded" (EG) regime in Eq. 2 (L121) is never defined

#### 12. Limitations section (L202–203) is good but could add scenario分布 details

#### 13. Named script references in limitations read as implementation noise

---

## Priority-Ranked Revision Recommendations

**Priority 1 — Before any further writing:**
1. Run `compute_paired_stats.py` on TutorEval deltas. Report 95% bootstrap CIs.
2. Fix the router equations. Resolve 0.55 vs 0.50 contradiction.
3. Execute at minimum ablations A and C.

**Priority 2 — Experimental:**
4. Remove EduBench from main results or provide scenario-level breakdowns.
5. Report `corpus_factuality` or remove the description.
6. Explain agentic RAG exclusion in limitations.

**Priority 3 — Writing:**
7. Soften abstract claims pending significance tests.
8. Add TF-IDF justification sentence.
9. Define EG regime.

---

## Bottom Line

The EduBench issue remains fatal in practice even if rhetorically mitigated. The TutorEval evidence is directionally consistent but too thin without significance tests and ablations. The paper has clean writing, honest limitations, and a useful systems framing — but needs executed ablations (at minimum A and C) and bootstrap CIs on TutorEval deltas before it becomes a competitive SMC submission.
