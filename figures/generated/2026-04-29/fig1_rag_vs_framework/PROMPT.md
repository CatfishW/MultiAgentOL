# Figure 1 Prompt: Traditional RAG vs Multi-Agent Retrieval

Use `gpt-image-2` through the `chatgpt2api-imagegen` skill.

```text
Create a clean IEEE SMC conference paper architecture diagram on a white background, landscape 16:9, crisp vector-infographic style. The figure must contain exactly two side-by-side comparison panels and no other panels.

Left panel title: Traditional RAG
Left panel content: a simple linear retrieve-then-read flow with four rounded boxes connected by arrows:
Query -> Vector Search -> Top-k Evidence -> LLM Tutor
Add one small note under the left flow: query-only context

Right panel title: Multi-Agent Retrieval (Ours)
Right panel content: a learning-state-aware tutoring framework with these boxes and arrows:
Student Query + Learning State + Rubric -> Supervisor Router -> Tutoring Brief -> LLM Tutor + Critic
Show four small branch boxes controlled by Supervisor Router and feeding into Tutoring Brief:
Planner
Diagnoser
Rubric Agent
Conditional Retrieval

Design requirements:
- Use only the labels listed above, spelled exactly.
- No equations, paragraphs, decorative labels, legends, logos, watermarks, or extra text.
- Make all text large, sharp, and readable after insertion into a two-column IEEE PDF.
- Use a muted gray/red accent for Traditional RAG and a blue/teal accent for Multi-Agent Retrieval.
- Use simple arrows and aligned rounded rectangles, with generous whitespace.
- The visual message should be: Traditional RAG retrieves from the surface query first, while Multi-Agent Retrieval builds a tutoring brief from learner state, rubric constraints, and conditional retrieval before the LLM tutor writes.
```
