# MARLET End-to-End Sample Trace

This file explains one real MARLET pipeline run from input to output. It is written as a readable trace report, not as an official benchmark result.

## Trace Metadata

| Field | Value |
|---|---|
| Dataset | EduBench |
| Source file | `/Users/zladwu/Development/mae-paper-20260406/MultiAgent/EduBench/en_data/AG.jsonl` |
| Row | `203` |
| Example ID | `edubench_AG_row_203_evidence_grounded_demo` |
| Architecture | `hybrid_fast` / MARLET |
| Model | `/home/tang/Projects/Models/Qwen3.5-4B` |
| Raw trace | `sample_traces/marlet_edubench_ag_row203_trace.json` |

Important caveat: a short evidence-grounding instruction was appended only to force this demonstration through MARLET's retrieval branch. The trace is useful for understanding the pipeline mechanics, but it should not be reported as a benchmark score.

## One-Line Flow

```text
EduBench grading request
-> normalize into x
-> router computes scores and selects R=AR, G=1
-> planner, diagnoser, and rubric agent prepare context records
-> coordinator merges records into one response brief
-> retriever searches with planner queries
-> generator receives one augmented prompt
-> critic checks once
-> PipelineResponse stores answer, route, evidence, outputs, and metrics
```

## 1. Raw Input

The original row asks the system to grade a student answer.

```text
Question:
Explain the significance of the Magna Carta in the history of democracy.

Student's Answer:
The Magna Carta was a document signed by King John of England in 1215. It was important because it limited the power of the king and established that everyone had to follow the law, even the king. This idea was a step towards democracy because it meant that the king couldn't do whatever he wanted and had to respect the rights of his subjects.

Please provide "Score", "Scoring Details", and "Personalized Feedback" based on the question and student's answer, in JSON format.
```

For this trace only, the following sentence was appended.

```text
Use available source evidence for verification; cite source, evidence, reference, document, chapter, lecture, and retrieval grounding when helpful.
```

This added sentence is what makes the evidence score high and opens retrieval in the trace.

## 2. Standardized Input `x`

MARLET first converts the dataset row into a shared input contract:

```text
x = (q, o, c, h, r, v)
```

| Symbol | Meaning | Actual value in this sample |
|---|---|---|
| `q` | User task/question | Full Magna Carta grading request plus evidence instruction |
| `o` | Answer options | None |
| `c` | Inline context/source passage | None |
| `h` | Dialogue history | Empty list |
| `r` | Rubric/answer criteria | Accuracy, Comprehension, Depth, Content Accuracy, Relevance to Question, Comprehensiveness, Clarity and Structure, Historical Accuracy, Relevance to Democracy, Depth of Explanation, Completeness, Relevance, Score, Scoring Details, Personalized Feedback |
| `v` | Visual inputs | Empty list |

All agents read from this standardized object through `AgentContext`. They do not read the raw JSON row directly.

## 3. Router Scores

The router builds a text blob from the question, context, rubric, and dialogue history. It then computes cue scores for several response-construction needs.

Actual logged scores:

```json
{
  "evidence": 0.75,
  "coordination": 0.10,
  "rubric": 0.5333333333333333,
  "planning": 0.09090909090909091,
  "adaptation": 0.21428571428571427,
  "tutoring": 0.21428571428571427
}
```

## 4. Consequences of Each Score

The scores are not final answer scores. They are routing/control signals. Each score can affect regime selection, retrieval, module activation, or prompt content.

| Score | Actual value | What it measures | Code-level consequence | Consequence in this sample |
|---|---:|---|---|---|
| `evidence` | `0.75` | Whether the request asks for sources, citations, documents, grounding, verification, or retrieval. | In MARLET, retrieval opens when `evidence >= hybrid_retrieval_gate` or when `R=EG`. The default gate is `0.35`. If no chunks are retrieved, a fallback retrieval can run when evidence is above the fallback threshold. | `0.75 >= 0.35`, so `G=true`. The retriever runs, four Magna Carta chunks enter the final prompt, and the generator is asked to cite `[doc_id]` markers. |
| `coordination` | `0.10` | Whether the request appears to need multi-step coordination, constraints, workflow, dialogue state, or dependency handling. | Under router architecture selection, high coordination can push toward agentic or non-retrieval multi-agent pipelines. Low coordination does not add extra coordination pressure. | Low coordination means the sample is not treated as a complex workflow. The MARLET run still uses planner/diagnoser/rubric because those are part of the selected `hybrid_fast` pipeline and route flags. |
| `rubric` | `0.5333` | Whether the request involves scoring, criteria, feedback, comments, or explicit requirements. | If no regime hint overrides it, high rubric score can select `CF` when it dominates adaptation and clears the mid threshold. Independently, `use_rubric_agent=true` when explicit rubric exists or `R=CF`. | The rubric score is high and explicit criteria exist, so `use_rubric_agent=true`. The rubric agent summarizes grading criteria, and the final prompt includes an `Answer criteria` block. |
| `planning` | `0.0909` | Whether the request asks for a plan, sequence, schedule, roadmap, lesson plan, or ordered steps. | If no regime hint overrides it, a high planning score can select `PL` when it dominates rubric/adaptation and clears the plan threshold. `PL` changes the planner's strategy toward sequencing and usually enables critique. | The planning score is low, so this sample does not become `PL`. The planner still runs because MARLET always builds a brief, but it writes an adaptive/evidence-aware strategy rather than a lesson sequence. |
| `adaptation` | `0.2143` | Whether the request needs visible user/task-state adaptation, such as learner level, misconception, prior attempt, hint, or personalized explanation. | If no regime hint overrides it, adaptation can select `AR` when it clears the mid threshold. `AR` makes state-aware response construction central and contributes to critic activation. | The adaptation score alone is below the usual mid threshold, but the EduBench example carries an adaptive tutoring regime hint. The final selected regime is therefore `AR/adaptive_tutoring`. |
| `tutoring` | `0.2143` | Alias/reporting view of adaptation in this code path. | It is logged for analysis but does not add a separate routing rule beyond adaptation. | Same practical effect as adaptation: it helps explain that this is an education/adaptive-response sample, but it is not a separate gate. |

## 5. Final Route Decision

Actual logged route:

```json
{
  "architecture": "hybrid_fast",
  "regime_R": "adaptive_tutoring",
  "retrieval_gate_G": true,
  "use_critic": true,
  "use_rubric_agent": true,
  "specialist_roles": ["tutor", "diagnoser", "retriever", "rubric", "critic"]
}
```

Paper notation:

```text
R(x) = AR
G(x) = 1
```

Code naming:

| Paper label | Code enum | Meaning |
|---|---|---|
| `PL` | `lesson_exercise_planning` | Planning or sequencing should dominate. |
| `CF` | `rubric_based_feedback` | Criteria or feedback should dominate. |
| `AR` | `adaptive_tutoring` | Visible user/task-state adaptation should dominate. |
| `EG` | `evidence_grounded_reasoning` | Evidence grounding should dominate. |

Why `R=AR` here: the example carries an EduBench adaptive-tutoring regime hint, so the code uses that hint before applying score-only regime selection. The cue scores still matter because they open retrieval and activate rubric/critic behavior.

Why `G=1` here: the evidence score is `0.75`, which is above the retrieval gate threshold.

## 6. Activated Modules

For this actual sample, the active modules are:

| Module | Runs? | Reason |
|---|---|---|
| Planner | Yes | MARLET always builds a planner brief in `HybridFastPipeline`. |
| Diagnoser | Yes | `enable_diagnoser=True`. |
| Rubric agent | Yes | Explicit rubric/criteria are present and `use_rubric_agent=true`. |
| Retriever | Yes | `G=true` and a retriever exists. |
| Generator/Tutor | Yes | The generator always produces the user-facing draft. |
| Critic | Yes | `use_critic=true` and critic is not disabled. |

Important: this sample uses every main module, but MARLET does not require every module for every input. Rubric, retrieval, and critique are conditional.

## 7. Planner Step

The planner receives the standardized sample and route:

```text
Domain or dataset: EduBench
Response regime: adaptive_tutoring
Retrieval gate: open
Question or task: <full Magna Carta grading request>
Criteria: <rubric terms>
Return at most 3 queries.
```

The planner has two outputs:

```text
a = operating strategy for the generator
P = retrieval query list
```

Actual strategy `a`:

```text
1. Infer visible user/task state and likely source of confusion.
2. Use retrieval only for the parts that need grounding.
3. Choose the minimum explanation needed for progress.
4. End with a check-for-understanding or next-step hint.
```

Actual retrieval queries `P`:

```text
P1: Full Magna Carta grading prompt with evidence instruction.
P2: question explain significance magna carta history
P3: Full Magna Carta grading prompt plus "Accuracy Comprehension"
```

Planner trace stats:

| Field | Value |
|---|---:|
| Mode | `llm_parse_fallback` |
| Latency | `5004 ms` |
| Prompt tokens | `438` |
| Completion tokens | `320` |
| Total tokens | `758` |

`llm_parse_fallback` means the planner LLM was called, but its JSON was not parsed successfully. The deterministic fallback is still regime-aware, so `R=AR` produced an adaptive strategy.

## 8. Diagnoser Step

The diagnoser receives the same sample fields, but its prompt does not explicitly print the regime. It extracts only visible state from the current request.

Actual state summary `ell`:

```text
user_level: intermediate
goals: solve current problem
style: not stated
```

Actual tool observation:

```text
inspect_dialogue_state: no prior interaction; state must come from current request
```

Diagnoser trace stats:

| Field | Value |
|---|---:|
| Mode | `llm_parse_fallback` |
| Latency | `4664 ms` |

Consequence: `ell` becomes the `Visible user/task state` block in the generator prompt. For `R=AR`, this state block is especially important because the generator is expected to adapt feedback to the visible task state.

## 9. Rubric Agent Step

The rubric agent receives the standardized sample, explicit criteria, and the regime.

Prompt facts:

```text
Domain or dataset: EduBench
Response regime: adaptive_tutoring
Question or task: <full Magna Carta grading request>
Explicit criteria: <rubric terms>
```

Actual criteria summary `u`:

```text
Prioritize the following criteria:
- Accuracy
- Comprehension
- Depth
- Content Accuracy
- Relevance to Question
- Comprehensiveness
- Clarity and Structure
- Historical Accuracy
- Relevance to Democracy
- Depth of Explanation
```

Rubric trace stats:

| Field | Value |
|---|---:|
| Mode | `llm_parse_fallback` |
| Latency | `5200 ms` |

Consequence: `u` becomes the `Answer criteria` block in the generator prompt. It also gives the critic concrete items to check.

## 10. Coordinator Merge

After planner, diagnoser, and rubric return, the coordinator merges their records into one updated context:

```text
R = adaptive_tutoring
G = true
a = planner strategy
P = planner queries
ell = visible state summary
u = criteria summary
T = tool observations
```

The merged context contains:

```text
original task + route + strategy + search queries + visible state + criteria + tool notes
```

This is the first key coordination step. The specialists do not send messages to each other. They write structured outputs, and the coordinator puts those outputs into one shared context.

## 11. Conditional Retrieval

Because `G=true`, MARLET retrieves evidence using planner queries `P`.

Retrieval input:

```text
P1: Full Magna Carta grading prompt with evidence instruction.
P2: question explain significance magna carta history
P3: Full Magna Carta grading prompt plus "Accuracy Comprehension"
```

Actual retrieved evidence `D`:

| Rank | Doc ID | Score | Content summary |
|---:|---|---:|---|
| 1 | `edubench-ag-203` | `0.8261` | Same Magna Carta democracy grading example. |
| 2 | `edubench-ag-205` | `0.4915` | Related Magna Carta example about medieval England, limiting royal power, rule of law, church rights, and illegal imprisonment. |
| 3 | `edubench-ag-204` | `0.4690` | Related Magna Carta example about democratic principles, rule of law, and limiting absolute power. |
| 4 | `edubench-ag-206` | `0.4914` | Another related medieval England example. |

Consequence: the evidence bundle `D` is inserted into the final prompt as `Grounding evidence`, with doc IDs available for citation.

## 12. Final Prompt Assembly

The generator receives one augmented prompt. Actual logged blocks:

| Prompt block | Present? | Source |
|---|---|---|
| Question/task | Yes | `q` |
| Recent history | No | `h=[]` |
| Operating plan | Yes | planner strategy `a` |
| Visible user/task state | Yes | diagnoser state `ell` |
| Answer criteria | Yes | rubric summary `u` |
| Grounding evidence | Yes | retrieved chunks `D` |
| Inline context | No | `c=None` |
| Answer requirements | Yes | generator prompt template |

Concrete final prompt shape:

```text
Domain or dataset: EduBench

Question or task:
<Magna Carta grading request>

Visible user/task state:
user_level: intermediate
goals: solve current problem
style: not stated

Operating plan:
1. Infer visible user/task state and likely source of confusion.
2. Use retrieval only for the parts that need grounding.
3. Choose the minimum explanation needed for progress.
4. End with a check-for-understanding or next-step hint.

Answer criteria:
Prioritize the following criteria:
- Accuracy
- Comprehension
- Depth
- Content Accuracy
- Relevance to Question
- Comprehensiveness
- Clarity and Structure
- Historical Accuracy
- Relevance to Democracy
- Depth of Explanation

Grounding evidence:
[edubench-ag-203] ...
[edubench-ag-205] ...
[edubench-ag-204] ...
[edubench-ag-206] ...

Answer requirements:
- be correct and useful for the user's visible task state
- be concise but not terse
- include one clear next step or check-for-understanding when appropriate
- when evidence is provided, cite with [doc_id]
```

This is why the framework is still RAG: retrieved evidence and agent-produced context are assembled into one augmented prompt before the final LLM answers.

## 13. Generator and Critic

The generator is the only component that produces the user-facing draft. Because this sample has EduBench evaluation metadata, it uses the EduBench JSON system prompt and is supposed to return:

```json
{
  "score": 0,
  "Scoring_Details": {},
  "Personalized Feedback": ""
}
```

Actual generator trace:

| Field | Value |
|---|---:|
| Latency | `6307 ms` |
| Prompt tokens | `1405` |
| Completion tokens | `450` |
| Total tokens | `1855` |

The raw generator output was malformed: Qwen3.5-4B emitted a `Thinking Process` preamble despite the JSON-only instruction. Therefore the raw `final_y` is useful for debugging but should not be copied into the paper as a clean answer.

The critic then checks the draft against retrieved evidence, rubric terms, and visible state.

Actual critic trace:

| Field | Value |
|---|---:|
| Ran | `true` |
| Issues | `[]` |
| Mode | `llm` |
| Latency | `6134 ms` |

The critic also returned malformed reasoning text in this trace, so this example should be used to explain pipeline mechanics rather than final answer quality.

## 14. Clean Intended Output Shape

A cleaned paper-facing output form for this sample would look like:

```json
{
  "score": 88,
  "Scoring_Details": {
    "Accuracy": "The answer correctly identifies King John, the 1215 date, limits on royal power, and the principle that the king is subject to law [edubench-ag-203].",
    "Comprehension": "The answer connects those facts to democracy by explaining that rulers cannot act with absolute power [edubench-ag-204].",
    "Depth": "The answer is correct but could go further by mentioning due process, protection against unlawful imprisonment, or later constitutional influence [edubench-ag-205].",
    "Relevance to Question": "The response directly addresses the Magna Carta's significance for democratic history.",
    "Clarity and Structure": "The answer is clear and concise."
  },
  "Personalized Feedback": "Your answer captures the core idea: the Magna Carta limited royal power and supported rule of law. To improve it, add one specific long-term democratic principle, such as due process or protection from unlawful imprisonment."
}
```

This cleaned JSON is an explanatory reconstruction of the intended answer shape, not the raw model output and not an official benchmark score.

## 15. Resource Trace

Actual runtime metrics:

| Metric | Value | Meaning |
|---|---:|---|
| `latency_ms` | `17667.0` | End-to-end wall time for this single demonstration. |
| `agent_count` | `6.0` | Planner, diagnoser, rubric, retriever, generator, critic. |
| `llm_call_count` | `5.0` | Planner, diagnoser, rubric, generator, critic. Retrieval itself is tool/search in this trace. |
| `prompt_tokens` | `3243.0` | Total prompt tokens across LLM calls. |
| `completion_tokens` | `1820.0` | Total generated tokens across LLM calls. |
| `total_tokens` | `5063.0` | Prompt plus completion tokens. |
| `retrieval_query_count` | `6.0` | Query accounting across planner/retrieval artifacts. |
| `tool_call_count` | `6.0` | Tool observations/search accounting in trace metadata. |
| `model_cache_hits` | `0.0` | No exact model-response cache hit in this run. |
| `retrieved_chunks` | `4.0` | Four evidence chunks entered the final prompt. |

The high completion-token count is partly caused by the malformed reasoning preamble from the small model.

## 16. What Each Score Changed in This Sample

| Score | Immediate effect | Downstream effect |
|---|---|---|
| High `evidence` | Opened `G=true`. | Retrieval ran, `D` was nonempty, citations were expected, and the final prompt included grounding evidence. |
| Low `coordination` | Did not indicate a complex workflow. | No extra workflow-style behavior was needed beyond MARLET's standard structured coordination. |
| High `rubric` | Supported `use_rubric_agent=true`. | Criteria summary `u` was produced and inserted into the final prompt. |
| Low `planning` | Did not select `PL`. | Planner still ran, but wrote an adaptive/evidence-aware strategy rather than a sequencing-heavy plan. |
| Moderate `adaptation` plus EduBench regime hint | Selected `R=AR/adaptive_tutoring`. | Planner strategy emphasized visible state and feedback adaptation; diagnoser state became an important final-prompt block. |

## 17. Final Takeaway

For this sample, MARLET does not work by letting multiple agents produce competing answers. It works by turning one input into structured context:

```text
planner -> strategy a and search queries P
diagnoser -> visible state ell
rubric agent -> answer criteria u
retriever -> evidence bundle D
coordinator -> one merged generator prompt
generator/critic -> final answer path
```

The router scores decide which constraints matter. In this trace, evidence and rubric scores are high, the selected regime is adaptive response, and retrieval opens. The result is one final prompt that combines adaptation, criteria, and evidence instead of asking a raw RAG query to carry all constraints at once.
