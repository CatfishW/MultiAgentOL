# Comprehensive Reviewer Feedback Summary (2026-04-20)

## Overall Verdicts

- **Novelty Reviewer**: Weak Accept (borderline)
- **Experimental Validity Reviewer**: Accept with major revisions (4/10 validity)
- **Writing Quality Reviewer**: Solid with revisions needed

## Critical Issues (Must Fix for Acceptance)

### 1. EduBench Subset Comparability Problem ⚠️ FATAL FLAW
**All three reviewers flagged this as the most serious issue**

**Problem**: Architectures evaluated on different, non-overlapping subsets:
- classical_rag: 4.02% coverage (682 examples)
- hybrid_fast: 13.31% coverage (2,254 examples)
- non_rag_multi_agent: 12.42% coverage (2,104 examples)
- single_agent_no_rag: 7.93% coverage (1,338 examples)

**Why it's fatal**: Cannot make comparative claims when test sets differ. Selection bias is uncontrolled.

**Current problematic claims**:
- Abstract (L34): "coordination-heavy paths show higher rubric coverage"
- Results (L147-148): "coordination-heavy routes show markedly higher rubric coverage"
- Discussion (L189-190): "complementary signal"
- Conclusion (L199-200): "superior rubric coverage (0.955-0.977 vs. 0.735-0.799)"

**Required fixes**:
1. **Option A (Preferred)**: Recompute metrics on intersection of successfully completed examples
2. **Option B**: Remove all comparative EduBench claims; relegate to appendix as "preliminary trajectory evidence"
3. **Option C**: Wait for 100% EduBench convergence before submission

### 2. Missing Statistical Significance Tests ⚠️ CRITICAL
**Problem**: No confidence intervals, standard deviations, or significance tests anywhere

**Affected claims**:
- TutorEval improvements: +3.3% Token-F1, +5.0% grounded overlap, -5.4% latency
- These are modest gains that may not be statistically significant

**Required fix**: Add bootstrap confidence intervals or permutation tests for all TutorEval comparisons

### 3. Missing Critical Ablations ⚠️ CRITICAL
**Problem**: Cannot attribute performance to specific design choices without ablations

**Minimum required ablations**:
1. **Hybrid without conditional retrieval** (isolate routing contribution)
2. **Non-RAG with retrieval enabled** (separate grounding from coordination)
3. **Hybrid without critic** (isolate critic contribution)

**Current status**: Paper acknowledges these as "planned for future analysis" (L184-186) but this is insufficient

### 4. Grounded Overlap Metric Confound ⚠️ CRITICAL
**Problem**: Metric structurally favors retrieval systems (returns 0.0 when no chunks retrieved)

**Required fix**: Replace or supplement with retrieval-agnostic grounding metric (e.g., factual consistency with corpus)

### 5. Novelty vs SelfRAG/CRAG Unclear
**Problem**: Paper doesn't clearly differentiate conditional retrieval from prior work

**Required fix**: Add explicit comparison showing how tutoring-specific routing differs from SelfRAG/CRAG mechanisms

## High-Priority Issues (Strongly Recommended)

### 6. Missing Model Specification
**Problem**: Never specifies which LLM is used (GPT-4, Claude, Llama, etc.)

**Required fix**: Add model details in setup section

### 7. Router/Retrieval Hyperparameters Under-Specified
**Problem**: 
- Threshold 0.35 for conditional retrieval - how chosen?
- Threshold 0.45 for fallback - why different?
- Retrieval weights (λ_w, λ_c, λ_ℓ) - what values?

**Required fix**: Add appendix with full hyperparameter specification and tuning procedure

### 8. Incorrect Rubric Coverage Numbers in Conclusion
**Problem**: Line 199-200 claims "0.955-0.977" but Table 2 shows 0.9553, 0.9541

**Required fix**: Correct to match actual data

### 9. Excessive Hedging Language
**Problem**: Undermines confidence with phrases like "should not be read as", "may partly reflect", "should be read as directional"

**Required fix**: State what results DO show, not just what they don't show

## Medium-Priority Issues

### 10. Transfer Benchmarks Mentioned But Not Used
**Problem**: Line 119 lists HotpotQA, SCROLLS, etc. but never reports results

**Required fix**: Either use them or remove mention

### 11. Agentic_RAG Excluded Without Explanation
**Problem**: Implemented but excluded because "comparable outputs not available" - why?

**Required fix**: Report results or explain failure mode

### 12. No Human Evaluation
**Problem**: For tutoring systems, human evaluation is essential

**Required fix**: Add small-scale human eval (50-100 examples) or acknowledge as major limitation

### 13. Define "Orchestration" on First Use
**Problem**: Term used throughout but never formally defined

**Required fix**: Add definition at line 47

## Minor Issues (Nice to Have)

### 14. Figure/Table Improvements
- Figure 1: Make hybrid_fast more visually distinct
- Table 1: Add warning about non-comparable subsets in caption
- Figure 3: Use different visual encoding for partial EduBench data

### 15. Writing Clarity
- Tighten verbose constructions (L39, L83, L189)
- Add roadmap sentence to Related Work (L52)
- Improve section transitions

## Recommended Revision Strategy

### Phase 1: Fix Fatal Flaws (Required for acceptance)
1. Address EduBench subset problem (use intersection or remove comparative claims)
2. Add significance tests for TutorEval
3. Implement minimum 2-3 ablations
4. Fix grounded overlap metric

### Phase 2: Strengthen Experimental Validity
5. Specify model and hyperparameters
6. Clarify novelty vs SelfRAG/CRAG
7. Add human evaluation or acknowledge limitation

### Phase 3: Polish Writing
8. Fix data errors (rubric coverage numbers)
9. Reduce hedging language
10. Improve clarity and flow

## Current Paper Status

- **Page count**: 5 pages (under 6-page target) ✅
- **Citations**: All fixed ✅
- **Technical accuracy**: Strong (verified against implementation) ✅
- **Experimental validity**: 4/10 (needs major work) ⚠️
- **Novelty**: Borderline (incremental contribution) ⚠️

## Bottom Line

The paper has solid technical execution and careful experimental controls, but **cannot be submitted in current form** due to:
1. Invalid EduBench comparisons (different test sets)
2. Missing statistical rigor
3. Missing critical ablations
4. Unclear novelty positioning

**Estimated revision effort**: 2-4 weeks for full revision cycle including new experiments
