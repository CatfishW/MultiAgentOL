# Review 4: Technical Soundness and Reproducibility

**Reviewer**: Oracle Agent — Technical & Reproducibility Focus
**Paper**: *Adaptive Tutoring Beyond Classical RAG: Conditional Retrieval in a Repository-Grounded Multi-Agent Study*
**Target**: IEEE SMC 2026
**Date**: 2026-04-23

---

## Overall Rating: **Weak Reject** (6.5/10)

| Dimension | Score | Notes |
|---|---|---|
| Technical Soundness | 6/10 | Solid architecture, but experimental validity undermined by capacity mismatch |
| Reproducibility | 7.5/10 | Code-to-paper alignment good; key details (corpus, thresholds, scoring) missing |
| Implementation Clarity | 7.5/10 | Clean codebase, well-structured modules, clear contracts |
| Experimental Controls | 4/10 | Capacity not matched, no statistics, no ablations, snapshot ambiguity |
| Engineering Rigor | 5.5/10 | BudgetPolicy exists but unenforced; latency under-counts retrieval; critic unmeasured |

---

## Issue Register

### FATAL

#### **Capacity is not matched — architectural complexity confounds results**

**Paper**: L97 claims "four families with comparable outputs"; L134 claims "differences reflect *structural* rather than capacity differences."

**Reality from code** (`pipelines.py` L186-412):

| Family | Agents | LLM calls |
|---|---|---|
| Classical RAG | retriever + tutor (2–3) | 2–3 |
| Non-RAG multi-agent | planner + diagnoser + rubric + tutor (4–5) | 4–5 |
| Single agent | tutor (1–2) | 1–2 |
| Hybrid fast | planner + diagnoser + rubric + tutor + [retriever] + [critic] (5–7) | 5–7 |

The `BudgetPolicy` in `contracts.py` L29-36 defines `max_agents=4`, `max_tool_calls=4`, but **these budgets are never checked in `pipelines.py`**. The BudgetPolicy dataclass is **dead code**.

**Why it matters:** More agents = more tokens = more reasoning capacity. If hybrid does better, it may be because it gets more LLM calls, not because the architecture is superior.

**Recommendation:** Either (a) add capacity normalization caveat, or (b) run capacity-matched ablation (equal LLM calls per example).

---

### CRITICAL

#### **No statistical significance tests in final results**

TutorEval gains are modest: composite 0.377→0.383 (+0.006), Token-F1 0.112→0.115 (+0.003). `compute_paired_stats.py` exists at `/MultiAgent/scripts/compute_paired_stats.py` (L1-319) and is well-implemented. But the paper's Limitations (L203) explicitly states: "Neither is executed over the final n=834 TutorEval slice."

**Recommendation:** Run `compute_paired_stats.py` on TutorEval. Report 95% CI and p-values. (~30 min)

#### **Critical ablations are promised but not executed**

L189-192 lists five controlled experiments (A-E). The ablation knobs exist in `config.py` L67-82 (`hybrid_force_retrieval`, `hybrid_disable_critic`, etc.), which is good infrastructure. But no results are reported.

**Recommendation:** At minimum, run ablation A (`hybrid_force_retrieval=True`) and C (`hybrid_disable_critic=True`) on 200 TutorEval examples.

#### **Router scoring formula under-specified for reproduction**

**Paper**: L108-109 describes "five heuristic scores from normalized keyword cues."

**Reality** (`regime_router.py` L16-62, L111-143): The scoring function includes hard-coded bonuses not mentioned in paper:
- `rubric` adds `+0.2` if `example.rubric` present
- `tutoring` adds `min(len(dialogue_history) / 6.0, 0.5)`
- `context_text` present: `evidence += 0.08`
- `images` present: `evidence += 0.05`, `coordination += 0.03`
- Dataset priors add `+0.12` to `+0.18`
- All scores clamped to `[0, 1]`

**Recommendation:** Add paragraph listing complete scoring algorithm, or reference source code explicitly.

---

### MAJOR

#### **EduBench "common processed floor" is ambiguous**

L155 claims "common processed floor of 1,580 examples per architecture." Does this mean the same 1,580 examples, or that each processed ≥1,580 examples from potentially different subsets?

**Recommendation:** State explicitly whether metrics are on identical or non-identical example sets.

#### **Latency measurement under-counts retrieval overhead**

`total_latency_ms` measures from `started = time.perf_counter()` to `_finalize()`. The index search time (TF-IDF + SVD in `index.py`) is excluded from `api_time_ms` and only in wall-clock. The comparison is fair at wall-clock level, but the paper doesn't decompose whether speed advantages come from fewer LLM calls or conditional retrieval skipping.

The `complexity_units` metric (L132-138) provides proper accounting but is not reported in the paper.

#### **Critic activation rate unknown**

**Paper**: L134 describes critic as "optional" and "issue-triggered."

**Reality** (`regime_router.py` L189-191): Critic activates when `enable_critic` AND (require_retrieval OR use_rubric_agent OR regime is ADAPTIVE_TUTORING/LESSON_PLANNING). Given TutorEval/EduBench map to ADAPTIVE_TUTORING regime, **the critic is almost always active for hybrid**. The paper claims "optional" but in practice it behaves as a permanent feature.

**Recommendation:** Report critic activation rate per benchmark per family.

#### **Corpus indexing details not reported**

L155: "All systems share the same indexed corpus." But never describes what the corpus actually is. `corpus.py` L47-96 loads from configurable directory of `.txt`, `.md`, `.jsonl`, `.json` files. Chunking: 220 tokens, 40 overlap.

**Missing from paper:**
- Source of indexed documents
- Total document/chunk count
- Whether corpus differs between TutorEval and EduBench

---

### MINOR

#### **Reranker uses default heuristic weights, not trained logistic regression**

**Paper**: L142 describes "six-feature logistic reranker."

**Reality** (`reranker.py` L66-89): Falls back to fixed weights `[0.55, 0.20, 0.10, 0.06, 0.05, 0.04]` when `self.model is None`. **Recommendation:** Clarify whether reranker was trained or used default weights.

#### **Threshold justification absent**

L123 states thresholds are "production values shipped with the repository; we report sensitivity as a planned ablation." `run_threshold_sweep.py` exists but no sensitivity analysis is reported.

#### **TF-IDF vs dense retrieval justification missing**

L138 describes retrieval stack as "intentionally lightweight" but doesn't justify why TF-IDF over Contriever, DPR, SBERT — the 2026 defaults.

#### **Latency accounting clarification**

The `complexity_units` metric (tokens + retrieval queries + chunks + agent calls) would be a better secondary efficiency metric than raw latency.

---

## Cross-Check: Prior Feedback Status

| Prior Issue | Status | Assessment |
|---|---|---|
| EduBench subset comparability | Partially fixed | Reframed as "snapshot" but still ambiguous |
| Missing statistical tests | Not fixed | Explicitly deferred |
| Missing critical ablations | Not fixed | Infrastructure exists, results absent |
| Grounded overlap confound | Fixed | `corpus_factuality` added, grounded_overlap removed from headline |
| Novelty vs SelfRAG/CRAG | Improved | Table I positions hybrid gate, but still thin |
| Model specification | Fixed | Qwen3.5-4B stated with thinking budget 512 |
| Router hyperparameters | Partially fixed | More parameters listed, but scoring bonuses not included |
| Rubric coverage numbers | Fixed | Numbers now match Table II |
| Excessive hedging | Improved | Prose is more direct |
| Agentic_RAG exclusion | Partially fixed | Mentioned but no explanation of why parity failed |
| Human evaluation | Not fixed | Deferred, acknowledged as limitation |

---

## Watch Out For

- **The paper's strongest claim** (hybrid > classical RAG) is the **least confounded comparison** (both use retrieval + tutor, hybrid adds routing). If you isolate this pair and add significance testing, the core contribution may survive review.
- **The weakest point** is the non-RAG vs hybrid comparison on EduBench, where both use similar agent counts but subset issues remain.
- **Capacity-fairness cannot be hand-waved away** — it's the issue most likely to trigger desk reject or negative committee discussion.

---

## Recommended Revision Path (priority order)

| # | Task | Effort |
|---|---|---|
| 1 | Run `compute_paired_stats.py` on TutorEval (hybrid vs classical, hybrid vs non-RAG) | ~30 min |
| 2 | Run ablation A (`hybrid_force_retrieval=True`) on 200 examples | ~2 hrs |
| 3 | Add capacity discussion + position hybrid-vs-classical as primary result | ~1 hr |
| 4 | Specify corpus: source, document count, chunk count | ~30 min |
| 5 | Document full scoring function with all bonuses | ~30 min |
| 6 | Clarify critic activation rate per family | ~1 hr |

**Bottom line:** Weak Reject on current form, but the path to Weak Accept is clear: run 2 experiments (~3 hrs), add 3 paragraphs (~2 hrs writing), and the paper becomes defensible.
