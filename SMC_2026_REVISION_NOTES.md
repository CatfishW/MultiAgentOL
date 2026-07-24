# SMC 2026 Revision Notes

This pass addresses reviewer 7 (review 21384) and reviewer 9 (review
21386) against `main_smc.tex`.

## Changes made

1. **Core novelty clarified.** The abstract, introduction, related work,
   contributions, and conclusion now identify MARTC's contribution as a
   bounded, training-free context controller. Cue-based task-state
   routing, specialist prompt construction, and conditional retrieval
   are the mechanism; the fixed generator and one-pass critic are the
   execution contract.
2. **Key terms defined.** "Task state," "task-state control," and
   "structured prompt construction" are defined on first use.
3. **Closest methods distinguished.** The related-work section now
   contrasts MARTC directly with Self-RAG, CRAG, Adaptive-RAG, TARG,
   Agentic RAG, MA-RAG, and multi-agent tutoring systems.
4. **Router made auditable.** The exact four cue lexicons, score
   normalization, explicit-input bonuses, thresholds, cross-backbone
   reuse, and a known lexical false-positive example are reported. The
   implementation remains the authoritative source:
   `framework_codes/src/eduagentic/ml/regime_router.py`.
5. **Threshold sensitivity reported cautiously.** The manuscript reports
   the existing gate sweep over `[0.30, 0.55]` and the flat aggregate
   quality interval `[0.30, 0.45]`. The limitations section states that
   this does not replace labeled routing-accuracy analysis.
6. **Experimental setup expanded.** Both backbones, hardware, generation
   settings, retrieval limits, cache-aware latency interpretation, and
   paired sample sizes are now stated in the main text.
7. **Metrics and statistics specified.** Token-F1, rubric coverage,
   tutor-hit, EB12D, EduAlign, latency, and token accounting are defined.
   Significance markers are tied to 10,000 paired permutations, with
   2,000 paired bootstrap resamples for 95% intervals.
8. **Claims narrowed.** Automatic metrics are consistently described as
   proxies. The abstract, discussion, limitations, and conclusion no
   longer imply that these results establish human-rated pedagogical
   quality.
9. **Table inconsistency corrected.** In the 4B TutorEval latency row,
   the displayed values `31.90` (RAG), `31.05` (Agentic), and `39.11`
   (MARTC) imply a MARTC-minus-RAG delta of `+7.21`, not `-2.79`.
   Agentic is therefore bolded as the lowest displayed latency. In the
   27B TutorEval token row, RAG is bolded because `6681 < 6730 < 7200`.
10. **Resource trade-offs stated explicitly.** The discussion now notes
    the 4B TutorEval latency increase and the small 27B TutorEval token
    increase instead of presenting MARTC as uniformly cheaper.

## Reviewer coverage

| Reviewer request | Status |
| --- | --- |
| Clarify novelty and identify the core contribution | Addressed in the manuscript |
| Define task-state control and structured prompt construction | Addressed in the introduction |
| Provide exact cue sets and thresholds | Addressed in Supervisor Routing |
| Explain threshold selection and sensitivity | Partially addressed with the fixed-value protocol and existing sweep |
| Give correct and incorrect routing examples | Addressed with auditable examples and a failure mode |
| Add sample sizes and metric details | Addressed in Implementation Details |
| Add human evaluation or metric-to-human validation | Not available; explicitly acknowledged as the principal limitation |

## Remaining author actions before submission

1. Add a human expert study or an independently supported
   metric-to-human correlation analysis if results can be obtained before
   the deadline. The current revision does not fabricate either.
2. Archive the full threshold-sweep outputs and routing examples used for
   the sensitivity claim, ideally in a small CSV/JSON artifact.
3. Confirm that the statistical procedure stated in the manuscript is
   the one used for every displayed significance marker, especially the
   27B tables.
4. Verify the latest 4B TutorEval latency source. This revision preserves
   the newest displayed values and corrects their arithmetic; an older
   repository revision contained `51.90`, `51.05`, and `49.11`.
5. Perform the final IEEE PDF compliance check after author metadata,
   copyright text, and submission ID requirements are known.
