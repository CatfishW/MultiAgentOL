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

Important caveat: a short evidence-grounding instruction was appended only for this demonstration. The purpose was to make the example exercise MARLET's retrieval branch so the walkthrough can show routing, retrieval, evidence insertion, generation, and critique in one trace. This is not how official benchmark inputs are altered, and this trace should not be reported as a benchmark score.

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

### Demo-Only Operation: Append an Evidence Request

For this trace only, the following sentence was appended.

```text
Use available source evidence for verification; cite source, evidence, reference, document, chapter, lecture, and retrieval grounding when helpful.
```

Why this operation was done:

```text
The original EduBench row is mainly a grading/feedback request. It may or may not trigger retrieval strongly enough by itself. We appended an evidence request so this single walkthrough would show the full MARLET path, including retrieval and evidence insertion.
```

What this operation changes:

```text
It increases evidence-related cues such as source, evidence, reference, document, cite, and retrieval grounding.
```

What this operation does not mean:

```text
It is not part of the official experiment protocol. It should not be used to claim a benchmark result. It is only a controlled demonstration wrapper for explaining the mechanism.
```

## 2. Standardized Input `x`

Operation:

```text
Convert the raw EduBench row into MARLET's shared input object x.
```

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

Why this operation is needed:

```text
Different datasets store fields differently. Normalization makes every downstream module read the same object shape, so the router, agents, retriever, and evaluator do not need dataset-specific logic.
```

Consequence:

```text
The rest of the pipeline can operate on x=(q,o,c,h,r,v) instead of raw EduBench JSON.
```

## 3. Router Scores

Operation:

```text
Read the standardized input x and compute cue scores for evidence, coordination, rubric, planning, and adaptation needs.
```

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

Why this operation is needed:

```text
The router must decide what kind of context construction is needed before generation. The scores convert visible request cues into routing signals: evidence, criteria, planning, adaptation, and coordination.
```

Consequence:

```text
These scores determine or influence R, G, rubric activation, retrieval activation, critic activation, and architecture selection.
```

### How the Cue Scores Are Computed

The cue scores are lightweight routing features, not LLM judgments. The code path is:

```text
1. Build routing text b from q, c, r, and h.
2. Lowercase b and collapse repeated whitespace.
3. For each cue family, count how many cue phrases appear anywhere in b.
4. Divide the matched cue count by the number of cue phrases in that family.
5. Add small explicit-signal bonuses when applicable.
6. Clip every score to at most 1.0.
```

Code-faithful pseudocode:

```python
blob = normalize_text(question + inline_context + criteria_text + dialogue_history)

base_score = matched_cue_count(blob, cue_family) / len(cue_family)

evidence = base_score(EVIDENCE_CUES)
coordination = base_score(COORDINATION_CUES)
rubric = base_score(RUBRIC_CUES) + (0.20 if explicit_rubric_exists else 0.0)
planning = base_score(PLANNING_CUES)
adaptation = base_score(ADAPTATION_CUES) + min(dialogue_turn_count / 6.0, 0.50)

if inline_context_exists:
    evidence += 0.08
if images_exist:
    evidence += 0.05
    coordination += 0.03
if optional_dataset_priors_are_enabled:
    add the configured prior bonus for that dataset/regime

scores = clip_each_score_to_1(scores)
```

Important implementation detail: cue matching is substring matching. A cue counts once if it appears anywhere in the normalized routing text, even if it appears many times. Multi-word cues such as `lesson plan` and `supporting facts` must appear as that phrase. Because matching is substring-based, `personalized` matches the cue `personalize`, and `explanation` contains the cue `plan`. This makes the router fast and transparent, but it also means the scores are routing heuristics rather than semantic classifiers.

The full cue families used by the router are:

| Family | Cue phrases |
|---|---|
| Evidence | `cite`, `citation`, `source`, `evidence`, `supporting facts`, `reference`, `grounded`, `retrieval`, `document`, `chapter`, `lecture`, `verification` |
| Coordination | `user`, `profile`, `state`, `context`, `criteria`, `constraint`, `workflow`, `dependency`, `rubric`, `feedback`, `student`, `misconception`, `hint`, `next step`, `lesson plan`, `study plan`, `adaptive`, `scaffold`, `dialogue`, `pedagog` |
| Rubric | `rubric`, `criterion`, `criteria`, `score`, `feedback`, `comment` |
| Planning | `plan`, `sequence`, `schedule`, `workflow`, `roadmap`, `steps`, `next steps`, `lesson`, `curriculum`, `study schedule`, `exercise plan` |
| Adaptation | `user`, `profile`, `beginner`, `expert`, `personalize`, `adapt`, `confused`, `attempted`, `hint`, `misconception`, `student`, `explain`, `teach`, `tutor` |

Exact cue matches in this sample:

| Score | Matched cues | Arithmetic | Final score |
|---|---|---:|---:|
| `evidence` | `cite`, `source`, `evidence`, `reference`, `retrieval`, `document`, `chapter`, `lecture`, `verification` | `9 / 12` | `0.75` |
| `coordination` | `feedback`, `student` | `2 / 20` | `0.10` |
| `rubric` | `score`, `feedback` | `(2 / 6) + 0.20 explicit-rubric bonus` | `0.5333` |
| `planning` | `plan` | `1 / 11` | `0.0909` |
| `adaptation` | `personalize`, `student`, `explain` | `(3 / 14) + 0 dialogue-history bonus` | `0.2143` |
| `tutoring` | same value as adaptation in this code path | copied from `adaptation` | `0.2143` |

Why the evidence score is high here:

```text
The demo-only evidence sentence contains many evidence-family words:
cite, source, evidence, reference, document, chapter, lecture, verification, and retrieval.
That gives 9 matched evidence cues out of 12 total evidence cues.
```

Why the rubric score is high here:

```text
The text contains score and feedback, so the base rubric score is 2/6.
The standardized input also has an explicit rubric list r, so the router adds 0.20.
Final rubric score = 0.3333 + 0.20 = 0.5333.
```

Why the planning score is low:

```text
Only one planning cue is matched. In this trace, the cue is plan, which appears by substring inside explanation. No sequence, schedule, workflow, roadmap, steps, curriculum, or study-schedule cues appear.
```

Why the adaptation score is moderate:

```text
The text contains explain, student, and personalized. These match explain, student, and personalize.
There is no dialogue history, so the dialogue-history bonus is 0.
```

Signals that did not affect this trace:

| Signal | Code behavior | Value in this trace |
|---|---|---|
| Inline context bonus | Adds `0.08` to evidence when `c` is nonempty. | Not used because `c=None`. |
| Image bonus | Adds `0.05` to evidence and `0.03` to coordination when images exist. | Not used because `v=[]`. |
| Dialogue-history bonus | Adds `min(len(h)/6, 0.50)` to adaptation. | Not used because `h=[]`. |
| Dataset prior bonus | Optional prior can add evidence, rubric, adaptation, coordination, or planning bias by dataset. | Not used because experiment config sets `router.use_dataset_priors=false`. |

Mini examples for how scores would change:

| Example request fragment | Main cue matches | Likely routing effect |
|---|---|---|
| `Cite the source document and verify the answer with evidence.` | `cite`, `source`, `document`, `verification`, `evidence` | Higher evidence score; retrieval gate likely opens. |
| `Score this answer using the rubric and give feedback comments.` | `score`, `rubric`, `feedback`, `comment` | Higher rubric score; criteria/rubric agent likely runs and `R` may become `CF`. |
| `Create a study schedule with steps and a sequence of exercises.` | `study schedule`, `steps`, `sequence`, `exercise plan` if phrased that way | Higher planning score; `R` may become `PL` when planning dominates. |
| `I am a beginner and confused; tutor me with a hint.` | `beginner`, `confused`, `tutor`, `hint` | Higher adaptation score; `R` may become `AR`, making visible state more important. |
| `Use the user profile, dialogue context, workflow constraints, and rubric.` | `user`, `profile`, `dialogue`, `context`, `workflow`, `constraint`, `rubric` | Higher coordination score; can shift the architecture toward agentic or multi-agent handling. |

After scoring, the router uses the scores in three different ways:

```text
Architecture selection:
- If evidence >= evidence_threshold and coordination < 0.35, select classical RAG.
- Else if evidence >= 0.35 and coordination >= coordination_threshold, select agentic RAG.
- Else if coordination >= 0.55 and evidence < 0.28, select non-retrieval multi-agent.
- Otherwise select hybrid_fast / MARLET.

Regime selection:
- If an explicit regime_hint exists, use it first.
- Else PL wins if planning >= rubric, planning >= adaptation, and planning >= plan threshold.
- Else CF wins if rubric >= adaptation and rubric >= mid threshold.
- Else AR wins if adaptation >= mid threshold.
- Else EG is the default.

Retrieval gate:
- In MARLET, retrieval opens when evidence score clears the hybrid retrieval gate or when the selected regime is EG.
- In this trace, evidence=0.75 is above the 0.35 gate, so G=true.
```

Threshold values in the current experiment configuration:

| Threshold/config | Value | Used for |
|---|---:|---|
| `evidence_threshold` | `0.52` | Classical RAG architecture branch when coordination is low. |
| `coordination_threshold` | `0.50` | Agentic RAG architecture branch when evidence is also high. |
| `hybrid_retrieval_gate` | `0.35` | Opens retrieval inside MARLET. |
| `hybrid_retrieval_fallback` | `0.45` | Runs one raw-question fallback retrieval if the first MARLET retrieval returns no chunks. |
| `plan threshold` | `0.42` | Allows `PL` only when planning is clearly strong. |
| `mid threshold` | `0.35` | Allows `CF` or `AR` when criteria/adaptation cues are strong enough. |

Applying those rules to this sample:

```text
Architecture:
evidence = 0.75 >= 0.52, but coordination = 0.10 < 0.35 would normally make a low-coordination evidence-heavy request classical RAG under the generic architecture router.
This walkthrough intentionally studies the MARLET mechanism, so the run uses the MARLET / hybrid_fast pipeline. The key route consequence inside MARLET is therefore the retrieval gate G, not the generic architecture-family choice.

Regime:
The standardized EduBench example carries an adaptive_tutoring regime hint.
Because regime_hint is checked before score-only selection, R=AR even though adaptation=0.2143 is below the 0.35 mid threshold.

Retrieval:
Inside MARLET, evidence = 0.75 >= hybrid_retrieval_gate 0.35.
Therefore G=true and retrieval runs with planner queries.
```

## 4. Consequences of Each Score

The scores are not final answer scores. They are routing/control signals. Each score can affect regime selection, retrieval, module activation, or prompt content.

| Score | Actual value | What it measures | Code-level consequence | Consequence in this sample |
|---|---:|---|---|---|
| `evidence` | `0.75` | Whether the request asks for sources, citations, documents, grounding, verification, or retrieval. | In MARLET, retrieval opens when `evidence >= hybrid_retrieval_gate` or when `R=EG`. The default gate is `0.35`. If no chunks are retrieved, a fallback retrieval can run when evidence is above the fallback threshold. | `0.75 >= 0.35`, so `G=true`. The retriever runs, four Magna Carta chunks enter the final prompt, and the generator is asked to cite `[doc_id]` markers. |
| `coordination` | `0.10` | Whether the request appears to need multi-step coordination, constraints, workflow, dialogue state, or dependency handling. | Under generic architecture selection, high coordination can push toward agentic or non-retrieval multi-agent pipelines. Low coordination does not add extra architecture-selection pressure. | Low coordination means the sample is not treated as a complex workflow by the generic router. The walkthrough still uses MARLET so we can inspect its retrieval and prompt-assembly path. |
| `rubric` | `0.5333` | Whether the request involves scoring, criteria, feedback, comments, or explicit requirements. | If no regime hint overrides it, high rubric score can select `CF` when it dominates adaptation and clears the mid threshold. Independently, `use_rubric_agent=true` when explicit rubric exists or `R=CF`. | The rubric score is high and explicit criteria exist, so `use_rubric_agent=true`. The rubric agent summarizes grading criteria, and the final prompt includes an `Answer criteria` block. |
| `planning` | `0.0909` | Whether the request asks for a plan, sequence, schedule, roadmap, lesson plan, or ordered steps. | If no regime hint overrides it, a high planning score can select `PL` when it dominates rubric/adaptation and clears the plan threshold. `PL` changes the planner's strategy toward sequencing and usually enables critique. | The planning score is low, so this sample does not become `PL`. The planner still runs because MARLET always builds a brief, but it writes an adaptive/evidence-aware strategy rather than a lesson sequence. |
| `adaptation` | `0.2143` | Whether the request needs visible user/task-state adaptation, such as learner level, misconception, prior attempt, hint, or personalized explanation. | If no regime hint overrides it, adaptation can select `AR` when it clears the mid threshold. `AR` makes state-aware response construction central and contributes to critic activation. | The adaptation score alone is below the usual mid threshold, but the EduBench example carries an adaptive tutoring regime hint. The final selected regime is therefore `AR/adaptive_tutoring`. |
| `tutoring` | `0.2143` | Alias/reporting view of adaptation in this code path. | It is logged for analysis but does not add a separate routing rule beyond adaptation. | Same practical effect as adaptation: it helps explain that this is an education/adaptive-response sample, but it is not a separate gate. |

## 5. Final Route Decision

Operation:

```text
Convert cue scores and any explicit regime hint into route flags: response regime R, retrieval gate G, critic use, rubric-agent use, and specialist role list.
```

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

Why `R=AR` here: the example carries an EduBench adaptive-tutoring regime hint, so the code uses that hint before applying score-only regime selection. The cue scores still matter because the evidence score opens retrieval, and the explicit rubric plus regime flags determine whether rubric and critic behavior are used.

Why `G=1` here: the evidence score is `0.75`, which is above the retrieval gate threshold.

Why this operation is needed:

```text
The route is the supervisor's compact control decision. It prevents the system from treating every request as the same raw retrieve-then-read task.
```

Consequence:

```text
For this sample, R=AR makes adaptation/state relevant, G=1 opens retrieval, use_rubric_agent=true activates criteria summarization, and use_critic=true adds one revision pass.
```

## 6. Activated Modules

Operation:

```text
Use the route flags to decide which MARLET modules run for this input.
```

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

Why this operation is needed:

```text
Activation keeps MARLET bounded. The coordinator uses the route to avoid always running every expensive or unnecessary branch.
```

Consequence:

```text
This sample activates all main modules because it combines grading criteria, evidence need, and adaptive tutoring. Other samples can skip rubric, retrieval, or critic depending on route flags.
```

## 7. Planner Step

Operation:

```text
Ask the planner LLM to produce an operating strategy for the answer and a small list of retrieval queries.
```

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

Why this operation is needed:

```text
The planner separates answer organization from retrieval. It tells the generator how to approach the response and gives the retriever scoped search queries.
```

Consequence:

```text
The final prompt receives the operating plan a. Because G=1, the retriever also consumes P. If G were 0, P would be ignored and only a would remain useful.
```

## 8. Diagnoser Step

Operation:

```text
Ask the diagnoser LLM to extract visible user/task state from the current request and available dialogue history.
```

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

Why this operation is needed:

```text
The diagnoser prevents the generator from treating every user as identical. It extracts only visible state, such as level, goal, style, or confusion, without inventing hidden personal attributes.
```

Consequence:

```text
The generator sees a separate state block. In this trace, the block says the user is intermediate, wants to solve the current problem, and has no stated style.
```

## 9. Rubric Agent Step

Operation:

```text
Ask the rubric LLM to compress explicit grading or answer criteria into a checklist.
```

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

Why this operation is needed:

```text
The rubric agent prevents criteria from being buried inside a long prompt. It turns explicit grading requirements into a compact checklist.
```

Consequence:

```text
The generator is explicitly constrained by Accuracy, Comprehension, Depth, Historical Accuracy, and related criteria. The critic can also compare the draft against these criteria.
```

## 10. Coordinator Merge

Operation:

```text
Collect the planner, diagnoser, rubric, and tool records into one response brief.
```

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

Why this operation is needed:

```text
Without merging, the specialist outputs would remain disconnected. Coordination is the step that turns separate records into one usable response brief.
```

Consequence:

```text
The next modules no longer see only the raw question. They see the raw question plus route, strategy, retrieval queries, visible state, criteria, and tool notes.
```

## 11. Conditional Retrieval

Operation:

```text
If G is open, search the corpus using the planner queries and return a compact evidence bundle D.
```

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

Why this operation is needed:

```text
The retriever supplies source material only when the gate opens. This avoids forcing every sample through corpus search while still grounding evidence-sensitive requests.
```

Consequence:

```text
The generator receives four cited Magna Carta chunks. It is expected to use those chunks when judging factual accuracy and to cite them with [doc_id] markers.
```

## 12. Final Prompt Assembly

Operation:

```text
Assemble the original task, route-specific agent outputs, evidence bundle, and answer requirements into one generator prompt.
```

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

Why this operation is needed:

```text
This is the final convergence point. It makes MARLET a RAG-style context-construction framework rather than a set of independent answer-producing agents.
```

Consequence:

```text
The final generator receives one prompt containing q, a, ell, u, D, and answer requirements. Only after this assembly does the system produce a user-facing draft.
```

## 13. Generator and Critic

Operation:

```text
Generate one user-facing draft from the assembled prompt, then run one bounded critic pass when critique is enabled.
```

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

Why this operation is needed:

```text
The generator is responsible for the first user-facing answer. The critic is a bounded safeguard that checks citation use, criteria coverage, and state-appropriate style.
```

Consequence:

```text
In this trace, the mechanics ran, but Qwen3.5-4B emitted reasoning text despite JSON-only instructions. This exposes a formatting weakness in the demo run and is why the raw answer is not used as the clean example output.
```

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
