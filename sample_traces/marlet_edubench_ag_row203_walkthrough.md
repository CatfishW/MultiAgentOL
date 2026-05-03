# MARLET Single-Sample Walkthrough

This walkthrough uses one real EduBench assessment/grading example from `EduBench/en_data/AG.jsonl`, row 203. A short evidence-grounding sentence was appended only to force the demonstration through the retrieval branch, so this file should be treated as a pipeline trace rather than a benchmark result.

Raw trace: `sample_traces/marlet_edubench_ag_row203_trace.json`

## 1. Raw Input

The original row asks the system to grade a student's answer:

```text
Question: Explain the significance of the Magna Carta in the history of democracy.

Student's Answer:
The Magna Carta was a document signed by King John of England in 1215. It was important because it limited the power of the king and established that everyone had to follow the law, even the king. This idea was a step towards democracy because it meant that the king couldn't do whatever he wanted and had to respect the rights of his subjects.

Please provide "Score", "Scoring Details", and "Personalized Feedback" based on the question and student's answer, in JSON format.
```

For this trace only, the following sentence was appended:

```text
Use available source evidence for verification; cite source, evidence, reference, document, chapter, lecture, and retrieval grounding when helpful.
```

## 2. Standardized Input Contract

MARLET first converts the row into a common input object. In plain English, the framework keeps the task, any available context, conversation history, grading criteria, and modalities in one place.

```text
q: the full grading request above
o: no answer choices
c: no inline source passage
h: no previous dialogue turns
r: Accuracy, Comprehension, Depth, Content Accuracy, Relevance to Question, Comprehensiveness, Clarity and Structure, Historical Accuracy, Relevance to Democracy, Depth of Explanation, Score, Scoring Details, Personalized Feedback
v: no images
```

This is the object every component reads from. The agents do not independently answer the user at this stage.

## 3. Routing Decision

The router scans the standardized input for evidence needs, coordination needs, rubric needs, planning needs, and adaptation needs. In this example, the evidence score is high because the appended instruction asks for source evidence and citations, and the rubric score is high because the task explicitly asks for scoring and feedback.

Logged route:

```json
{
  "architecture": "hybrid_fast",
  "regime_R": "adaptive_tutoring",
  "retrieval_gate_G": true,
  "use_critic": true,
  "use_rubric_agent": true,
  "specialist_roles": ["tutor", "diagnoser", "retriever", "rubric", "critic"],
  "scores": {
    "evidence": 0.75,
    "coordination": 0.10,
    "rubric": 0.5333,
    "planning": 0.0909,
    "adaptation": 0.2143
  }
}
```

Interpretation:

- `R=adaptive_tutoring` means the response should be adapted to the visible task/user state, not just factually correct.
- `G=true` means retrieval is allowed for this sample.
- `use_rubric_agent=true` means explicit criteria should be summarized before generation.
- `use_critic=true` means the draft answer will be checked once before returning.

## 4. Parallel Specialist Preparation

After routing, MARLET runs preparatory agents in parallel where possible. They do not chat with each other. They write their outputs into a shared context object that is later assembled into one final prompt.

### Planner Output

The planner does two jobs: it writes an operating strategy for the final generator and produces scoped retrieval queries. It is not the retriever and it does not answer the user.

Logged strategy:

```text
1. Infer visible user/task state and likely source of confusion.
2. Use retrieval only for the parts that need grounding.
3. Choose the minimum explanation needed for progress.
4. End with a check-for-understanding or next-step hint.
```

Logged retrieval queries:

```text
1. Full task text with the student answer and evidence instruction.
2. question explain significance magna carta history
3. Full task text plus "Accuracy Comprehension"
```

### Diagnoser Output

The diagnoser extracts only visible state from the current request. It does not invent a persistent student profile.

Logged state:

```text
user_level: intermediate
goals: solve current problem
style: not stated
```

### Rubric Agent Output

The rubric agent turns the explicit criteria into a compact checklist that the generator must enforce.

Logged criteria summary:

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

## 5. Conditional Retrieval

Because `G=true`, MARLET retrieves evidence using the planner's scoped queries instead of using only the raw prompt. Retrieval is therefore part of prompt construction, not a separate answer path.

Logged retrieved evidence:

```text
1. edubench-ag-203, score=0.8261
   The same Magna Carta democracy grading example.

2. edubench-ag-205, score=0.4915
   A related Magna Carta example about medieval England, including limits on royal power and rule of law.

3. edubench-ag-204, score=0.4690
   A related Magna Carta example about democratic principles and limits on absolute power.

4. edubench-ag-206, score=0.4914
   Another related medieval England example, including limits on royal power and legal accountability.
```

The selected chunks are later inserted into the generator prompt with citation IDs such as `[edubench-ag-203]`.

## 6. Prompt Assembly

This is the key RAG step. MARLET converges every useful intermediate output into one augmented prompt before calling the final LLM generator.

Logged prompt blocks present:

```text
question/task: present
recent history: absent
planner strategy: present
visible state: present
criteria summary: present
retrieved evidence: present
inline context: absent
answer requirements: present
```

The final generator sees a prompt with this structure:

```text
Domain or dataset: EduBench

Question or task:
<grading request>

Visible user/task state:
user_level: intermediate
goals: solve current problem
style: not stated

Operating plan:
<planner strategy>

Answer criteria:
<rubric checklist>

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

This is why MARLET still performs RAG: retrieved evidence and agent-produced context are assembled into one prompt for the final LLM.

## 7. Final Answer Path

The generator is the only component that produces the user-facing draft. The critic then receives the draft and checks groundedness, criteria coverage, and usefulness. In the raw logged run, the small Qwen model emitted an unwanted reasoning preamble despite the JSON-only instruction, so the raw `final_y` is useful for debugging but should not be copied into the paper as the answer.

A cleaned paper-facing output form for this sample is:

```json
{
  "score": 88,
  "Scoring_Details": {
    "Accuracy": "The answer correctly identifies King John, the 1215 date, limits on royal power, and the principle that the king is subject to law [edubench-ag-203].",
    "Comprehension": "The answer connects those facts to democracy by explaining that rulers cannot act with absolute power [edubench-ag-204].",
    "Depth": "The answer is correct but could go further by mentioning longer-term constitutional influence, due process, or protections against unlawful imprisonment [edubench-ag-205].",
    "Relevance to Question": "The response directly addresses the significance of the Magna Carta in democratic history.",
    "Clarity and Structure": "The explanation is clear and concise."
  },
  "Personalized Feedback": "Your answer captures the core idea well: the Magna Carta limited the king and supported the rule of law. To make it stronger, add one concrete long-term democratic principle, such as due process or protection from unlawful imprisonment."
}
```

The score above is a cleaned explanatory presentation of the intended answer shape, not an official benchmark metric.

## 8. Resource Trace

Logged runtime for this single demonstration:

```json
{
  "latency_ms": 17667.0,
  "agent_count": 6.0,
  "llm_call_count": 5.0,
  "prompt_tokens": 3243.0,
  "completion_tokens": 1820.0,
  "total_tokens": 5063.0,
  "retrieved_chunks": 4.0,
  "tool_call_count": 6.0
}
```

The high token count is partly caused by the malformed reasoning output from this small-model trace. The important methodological point is the flow: input normalization, route selection, parallel specialist preparation, gated retrieval, prompt assembly, generation, and optional critique.
