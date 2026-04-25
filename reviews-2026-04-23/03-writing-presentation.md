# Review 3: Writing Quality and Presentation

**Reviewer**: Oracle Agent — Writing & Presentation Focus
**Paper**: *Adaptive Tutoring Beyond Classical RAG: Conditional Retrieval in a Repository-Grounded Multi-Agent Study*
**Target**: IEEE SMC 2026
**Date**: 2026-04-23

---

## Overall Rating: **Weak Accept** (borderline Accept)

The paper has been significantly improved from the pre-polish state. EduBench reframing is defensible, methodology is grounded in implementation, and related work tells a gap-driven narrative. However, experimental thinness remains the structural weakness, and several presentation-level issues undermine the impression of a finished product.

### Scorecard

| Criterion | Score (–10) |
|---|---|
| Clarity | 7.5 |
| Organization | 8.0 |
| Writing Quality | 7.0 |
| Visual Presentation | 7.5 |
| Formatting (IEEE SMC) | 8.0 |

---

## Section-by-Section Assessment

### 1. Abstract Quality: 7/10

**Strengths:** Leads with problem framing, numbers are scannable, concludes with implication. **"Issues:**

- **Sentence 1 is overloaded** (L42): 28-word sentence doing both background and gap simultaneously.
- **TutorEval → EduBench transition compressed** into one paragraph-spanning sentence. Consider splitting after "tokens." into a new sentence.
- **"thinking budget 512"** (L42) — jargon without context. Suggest: "(CoT budget 512 tokens)"

### 2. Introduction Narrative: 8/10

**Strengths:** Opening two queries are effective and concrete. Paragraph 3 correctly identifies "not whether, but when."

**Issues:**
- **L47:** "A student asks a tutoring system" — slightly formal for a narrative hook.
- **L53:** "Recent education-centered benchmarks make that distinction harder to ignore." — vague transition. Suggest: "Recent benchmarks shift the evaluation target from answer correctness to pedagogical quality, making this distinction material."
- **L55:** 41-word sentence combining definition, systems framing, and thesis. **"dominate"** used twice in succession. **L57:** Contribution (i) is a 53-word sentence. Split into at least two clauses.

### 3. Clarity of Methodology: 7.5/10

**Strengths:** Routing equations are precise, specialist agents described concretely, retrieval stack names every component with parameter.

**Issues:**
- **L97:** "five execution families" immediately contradicted by exclusion of agentic_rag.
- **L97:** "controlled adaptivity" undefined and never reused.
- **L108:** Five scores computed but only two ($s_e$, $s_c$) appear in equations.
- **Eq. 1 (L111–116):** `\texttt{otherwise}` catch-all for hybrid means broad traffic. Expected distribution not discussed.
- **L123:** "we report sensitivity as a planned ablation in \S\ref{sec:results}" — **broken cross-reference**.
- **L142:** "profile-aware" undefined; `corpus_factuality` described but never reported.

### 4. Results Presentation: 7/10

**Strengths:** Table 2 well-structured, prose correctly notes hybrid/non-RAG tie on EduBench.

**Issues:**
- **Table 2 mixes EB12D with TutorEval rows** where EB12D is "--". Column exists only for EduBench.
- **L159:** "lose clearly" too strong for 0.008 margin without statistical testing. Soften to "trail on composite."
- **§3.3 Planned controlled experiments** reads like a TODO list. Move to limitations.
- **L155:** "omit success-rate tables" draws attention to omission.

### 5. Figure Quality: 7/10

- **Figure 1 (family.png):** Concern about conference-resolution text readability at `\columnwidth`.
- **Figure 2 (MultiAgentMain.png):** Caption introduces terms ("shared state bus," "monitoring loop") not in methodology text. Cross-check.
- **Positioning Table (Table 1):** Excellent. Best-designed element in the paper.

### 6. Prose Quality: 7/10

**Terminology Inconsistency (medium severity):**
The hybrid is called "conditional-retrieval hybrid," "the proposed hybrid," `\texttt{hybrid_fast}`, and "hybrid" — interchangeably. **Pick one primary name.**

**Dense Sentences:**
- L108: 42 words, two parentheticals, one arrow notation
- L138: 31 words, three λ parameters, one semicolon

**Jargon Without Definition:**
- "thinking budget" (abstract L42)
- "profile-aware" (L142)

**Repetitive Phrasing:**
"Shared repository-grounded controls/infrastructure" appears in abstract, introduction, methodology, and setup. Vary the language.

### 7. Section Flow: 7.5/10

- **Intro → Related Work:** Smooth. Good signposting.
- **Related Work → Methodology:** Adequate.
- **Methodology → Results:** Abrupt. "updated paper numbers" is editorial.
- **Results → Discussion:** Planned experiments subsection interrupts flow.
- **Discussion → Limitations → Conclusion:** Clean.

### 8. IEEE SMC Format Compliance: 8/10

| Item | Status |
|---|---|
| Document class `ieeeconf` | ✅ |
| Font size 10pt | ✅ |
| `\pagestyle{empty}` (L39) | ⚠️ Suppresses page numbers |
| `\resizebox{\textwidth}{!}` (Table 2) | ⚠️ Verify font size in PDF |
| Page count | ⚠️ ~5 pages, under 6-page target |

### 9. Limitations Honesty: 8/10

**Strengths:** Three specific constraints, each with remediation. **Issues:** Named script references (`\texttt{run_eval_session.py}` etc.) read as implementation noise. Belong in code release, not paper.

---

## Line-by-Line Issue Summary

| Line | Severity | Issue |
|---|---|---|
| L42 | Low | "thinking budget 512" undefined |
| L42 | Low | Two benchmark results crammed into one flow |
| L53 | Low | "harder to ignore" is vague |
| L55 | Medium | "dominate" repeated twice, 41-word sentence |
| L57 | Medium | 53-word contribution sentence |
| L97 | Medium | "five execution families" contradicted |
| L97 | Low | "controlled adaptivity" undefined |
| L108 | Low | Five scores listed, only two used |
| L123 | **Medium** | Broken cross-reference to sensitivity analysis |
| L142 | Medium | "profile-aware" undefined, `corpus_factuality` unreported |
| L147 | Low | "updated paper numbers" is editorial |
| L155 | Low | "omit success-rate tables" draws attention |
| L159 | Medium | "lose clearly" too strong |
| L192–193 | **Medium** | Planned experiments in §Results |
| L203 | Medium | Named script references in limitations |
| L39 | Medium | `\pagestyle{empty}` suppresses page numbers |
| L168 | Low | `\resizebox` font size check |

---

## Key Prose Improvements (rewritten)

**Contribution (i) — split:**
> *Before:* "(i) We formalize adaptive tutoring as architecture selection conditioned on a lightweight regime router and operationalize it in a repository-grounded framework that instantiates classical RAG, non-RAG multi-agent, single-agent, and hybrid conditional-retrieval pipelines on a shared substrate..."
> *After:* "(i) We formalize adaptive tutoring as architecture selection conditioned on a lightweight regime router. Our framework instantiates four configurations — classical RAG, non-RAG multi-agent, single-agent, and a conditional-retrieval hybrid — on a shared substrate."

**Systems framing — split:**
> *Before:* "...deciding when evidence grounding should dominate and when instructional coordination should dominate."
> *After:* "...deciding when evidence grounding takes priority and when instructional coordination should lead."

**Results transition:**
> *Before:* "This section reports the updated paper numbers..."
> *After:* "We now present results under the protocol described above..."

**Limitations:**
> *Before:* "each tied to a concrete remediation that is already implemented as a named script..."
> *After:* "We emphasize three constraints and describe available remediations for each."

---

## Formatting Corrections Needed

1. **Remove `\pagestyle{empty}` (L39)** — restore page numbering
2. **Fix broken cross-reference (L123)**
3. **Move planned experiments (§3.3) out of results**
4. **Unified terminology** — "conditional-retrieval hybrid" → "hybrid" in prose

**Effort estimate: 1–4 hours.** Remaining prose fixes are point edits. Terminology unification is global find-and-replace + 6-8 manual adjustments.
