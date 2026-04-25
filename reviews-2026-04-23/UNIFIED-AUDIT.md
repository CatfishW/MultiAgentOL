# Unified Review Audit — MultiAgentOL Paper

**Date**: 2026-04-23
**Paper**: *Adaptive Tutoring Beyond Classical RAG: Conditional Retrieval in a Repository-Grounded Multi-Agent Study*
**Target**: IEEE SMC 2026
**Reviewers**: 4× Oracle Agents (Experimental Validity, Novelty/Positioning, Writing/Presentation, Technical/Reproducibility)

---

## Verdict Summary

| Reviewer | Rating | Soundness | Originality | Clarity |
|---|---|---|---|---|
| Experimental Validity | **Weak Reject** | 4/10 | 5/10 | 6/10 |
| Novelty & Positioning | **Weak Accept** | — | 4/10 | — |
| Writing & Presentation | **Weak Accept** | — | — | 7.5/10 |
| Technical Soundness | **Weak Reject** | 6/10 | — | — |

**Aggregate: Weak Accept → Borderline. Three fixable gaps.**

---

## 🔴 FATAL Issues (must fix before submission)

### 1. Capacity is NOT matched — results are confounded
All four reviewers converged. Families use different agent counts (classical RAG: 2–3, hybrid: 5–7), but `BudgetPolicy` caps are dead code in `pipelines.py`. More agents = more LLM calls = more capacity.

**Fix**: Run capacity-matched comparison OR add explicit caveat + position hybrid-vs-classical as primary result (least confounded pair).

### 2. EduBench subset comparability unresolved
Despite "common processed floor" reframing, 1,580 examples per architecture are from **potentially different subsets** (4–13% coverage). Comparative claims from non-identical test sets are unsound.

**Fix**: (A) Per-scenario breakdowns, (B) Intersection-based metrics, or (C) Remove comparative EduBench claims.

### 3. No statistical significance testing on TutorEval deltas
Hybrid vs classical: composite +0.006, Token-F1 +0.003. At n=834 these *might* be significant, but unestablished.

**Fix**: Run `compute_paired_stats.py`. Report 95% bootstrap CIs and p-values. (~30 min)

---

## 🟠 CRITICAL Issues

### 4. Critical ablations (A, C) not executed
The hybrid's advantage could come from the retrieval gate, critic, parallel specialists, or any combination. Without ablations, you cannot attribute results.

**Fix**: Run ablation A (hybrid without gate) and C (hybrid without critic) on ≥200 examples.

### 5. Missing training-free adaptive retrieval literature
FRAG, Paper-RAG, Adaptive-RAG, SIFT all implement training-free per-example retrieval gating. Without them, novelty objection persists.

**Fix**: Add 2–3 sentences naming methods + articulating difference (pedagogical regime vs retrieval quality as gating signal). Update Table 1.

### 6. Router scoring function incomplete in paper
Paper lists 5 scores + 2–3 thresholds. Actual `regime_router.py` has ~8 hard-coded bonuses (context +0.08, images +0.05, rubric +0.2, dialogue scaling, per-dataset priors +0.12 to +0.18). Reproducer cannot implement from paper alone.

**Fix**: Add scoring formula paragraph/table or explicit source code reference.

### 7. Contributions overclaim
Contribution (i) calls routing table a "formalization." Contribution (iii) claims "strongest overall balance" without significance.

**Fix**: Recast contributions to match delivered evidence.

---

## 🟡 MAJOR Issues

8. Router equations have internal contradictions (`s_c ≥ 0.55` vs `τ_c = 0.50`)
9. Critic "optional" but activates on >80% of examples (reframe as structural)
10. `corpus_factuality` described but never reported in results
11. Agentic RAG excluded without failure-mode explanation
12. Corpus details missing (source, document count, chunk count)
13. "Supervisory controller" framing under-developed
14. Planned experiments subsection in §Results is a TODO list (move to Limitations)
15. TF-IDF vs dense retrieval needs brief justification

---

## 🟢 MINOR Issues

16. Terminology: "conditional-retrieval hybrid" / "hybrid_fast" / "hybrid" — pick one primary
17. `\pagestyle{empty}` suppresses page numbers
18. Abstract: "thinking budget 512" undefined on first read
19. "five execution families" contradicted by exclusion
20. Contribution (i) is 53 words (split)
21. EB12D column in Table 2 is "--" for TutorEval rows
22. Methodology → Results transition reads editorially ("updated paper numbers")

---

## Recommended Repair Order (by ROI)

| Priority | Task | Effort | Impact |
|---|---|---|---|
| 1 | Run `compute_paired_stats.py` on TutorEval | 30 min | Eliminates #3, strengthens all claims |
| 2 | Run ablations A + C on 200 examples | 2 hrs | Eliminates #4, enables causal attribution |
| 3 | Capacity caveat + position hybrid-vs-classical as primary | 1 hr writing | Addresses #1, desk-reject trigger |
| 4 | Resolve EduBench: breakdown or remove comparative claims | 1–2 hrs | Addresses #2 |
| 5 | Add FRAG, Paper-RAG, SIFT to related work + Table 1 | 45 min | Addresses #5 |
| 6 | Fix router equations, document scoring bonuses | 30 min | Addresses #6, #8 |
| 7 | Move planned experiments to limitations | 15 min | Addresses #14 |
| 8 | Recast contributions to match delivered evidence | 30 min | Addresses #7 |
| 9 | Prose fixes (terminology, abstract, pagestyle) | 1 hr | Addresses #16–22 |

**Total estimated effort: ~7–8 hours.**

The paper is substantively sound (same model, same corpus, clean comparison methodology) but needs statistical rigor, ablation evidence, and honest capacity discussion to survive review.
