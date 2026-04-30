# Figure 1 Prompt: Traditional RAG vs MARLET

Use `gpt-image-2` through the `chatgpt2api-imagegen` skill.

```text
Create a publication-quality CVPR-style architecture figure for a research paper.
Canvas: landscape 16:9, clean white background, flat vector graphic, crisp lines, no gradients that reduce print clarity.
Overall composition: exactly two large side-by-side panels with a thin vertical divider and a shared visual rhythm.

Figure title at top center:
"Traditional RAG vs. MARLET"

Left panel title:
"Traditional RAG"

Left panel visual message:
Retrieval is the controller. The surface query directly drives vector search before the tutor writes.

Left panel layout:
1. Rounded input box: "Student Query"
2. Thick solid arrow to a large central box: "Vector Search"
3. Solid arrow to box: "Top-k Passages"
4. Solid arrow to box: "LLM Tutor"
5. Final small output box: "Tutor Response"
Add a small caption under the left panel: "query-only retrieval first"

Right panel title:
"MARLET: Multi-Agent Retrieval"

Right panel visual message:
Coordination is the controller. Learner state, rubric constraints, and scoped search are assembled before the tutor writes; retrieval is conditional, not mandatory.

Right panel layout:
1. Rounded input box: "Student Query"
2. Two smaller context chips beside it: "Learning State" and "Rubric"
3. Solid arrows from these inputs into a prominent central box: "Supervisor Router"
4. From "Supervisor Router", three compact specialist boxes in a horizontal row:
   "Planner", "Diagnoser", "Rubric Agent"
5. A small diamond below the specialists: "Retrieval Gate"
6. Dashed arrow from "Retrieval Gate" to a side box: "Conditional Retrieval"
7. Solid arrows from Planner, Diagnoser, Rubric Agent, and Conditional Retrieval into a merge box: "Tutoring Brief"
8. Solid arrow from "Tutoring Brief" to box: "LLM Tutor"
9. Dashed feedback/check arrow from "LLM Tutor" to a small box: "Critic"
10. Final output box: "Tutor Response"
Add a small caption under the right panel: "learner-state-aware control before tutoring"

Style requirements:
- CVPR / NeurIPS systems-figure aesthetic: precise grid alignment, generous whitespace, strong hierarchy, clean typography, no clutter.
- Use dark slate text and outlines. Traditional RAG panel uses neutral gray with one muted red accent on "Vector Search".
- MARLET panel uses cool blue and teal accents on "Supervisor Router", "Retrieval Gate", and "Tutoring Brief".
- Keep all text large and readable when inserted into a two-column academic PDF.
- Use rounded rectangles, simple arrows, and light shaded containers behind each panel.
- Make the MARLET router visually dominant, but keep the LLM Tutor box the same size in both panels to show the downstream tutor is fixed.
- Dashed arrows mean conditional or checking behavior; solid arrows mean always-active flow.

Strict constraints:
- Do not add any labels beyond the labels listed above.
- Do not include equations, numeric thresholds, icons, people, robots, books, decorative illustrations, logos, watermarks, or 3D effects.
- Do not create more than two panels.
- Do not use tiny text, paragraph text, or crowded legends.
- Spell exactly: "MARLET", "Traditional RAG", "Supervisor Router", "Retrieval Gate", "Tutoring Brief", "LLM Tutor".
```
