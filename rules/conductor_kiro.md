---
trigger: model_decision
description: Standard rules for rendering interactive option prompts and sequential question loops whenever any Conductor skill or workflow is active inside Kiro (IDE Agent Focus, IDE Editor View, or Kiro CLI).
---
# Conductor Kiro UX Adapter (View Layer)
These operational standards govern the user interface and conversational experience when Conductor skills are executed inside Kiro host environments (the Kiro IDE — both Agent Focus and Editor View — and the Kiro CLI).
## 1. Discrete, Option-Based, Blocking Prompts
-   **Blocking Ask:** Whenever a Conductor skill needs to gather user choices, single-select decisions, yes/no answers, or conduct interactive scaffolding loops, the agent MUST present a discrete question with an explicitly enumerated set of options and then pause, waiting for the user's answer before proceeding (a blocking ask).
-   **Interactive Buttons (Kiro IDE Agent Focus):** In Kiro IDE Agent Focus mode, a blocking question with enumerated options is surfaced to the user as an *attention card* whose options render as clickable buttons — the closest analog to Antigravity's native `ask_question` dialog. To maximize the chance of this rendering, the agent MUST keep each question atomic and list its options explicitly and concisely.
-   **Text Fallback:** In plain-text surfaces (the Kiro CLI, or the IDE chat when interactive buttons are not shown), the agent MUST present the same options as a clean bracketed numbered menu (e.g., `[1] Option A`, `[2] Option B`, `[3] Other`). The user may answer by number or by typing their choice.
-   **Always Offer "Other":** Every prompt MUST include a custom or "Other" option so the user can provide free-form input. When one option is recommended, list it first with a brief reason.
## 2. Sequential Question Barrier
-   Kiro has no tool for batching multiple questions into a single form. The agent MUST ask questions one at a time: present a single question, wait for the response, then proceed to the next. Never output multiple distinct questions in a single response. Keeping each question atomic is also what allows the IDE to render it as a single attention card with option buttons.
