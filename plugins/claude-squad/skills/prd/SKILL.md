---
name: prd
description: PRD phase — PM formalizes requirements, Tech Lead validates feasibility, Designer confirms UX alignment, QA defines acceptance criteria. Takes /idea and /design output, produces the PRD that feeds into /implement.
---

# /prd — Product Requirements Document

Run the PRD phase: formalize the validated idea and design into a complete Product Requirements Document with User Stories, acceptance criteria, technical requirements, and test strategy. **All technical decisions grounded in current stack docs via context7.**

This command expects the **Idea Document** (from `/idea`) and **Design Document** (from `/design`) as context. The idea has been brainstormed, the design has been prototyped — now we formalize everything into a spec that drives implementation.

The output of this command is a **PRD (Product Requirements Document)** — a self-contained artifact that feeds directly into `/implement`.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role in this phase |
|-------|-------------------|
| **PM** | Formalizes User Stories with acceptance criteria from the validated idea and design |
| **Tech Lead** | Validates feasibility against stack docs, scopes work, defines subtasks and technical direction |
| **Designer** | Confirms design specs align with requirements, flags any gaps |
| **QA** | Defines acceptance criteria test strategy, identifies testability gaps early |

---

## Input

This skill requires the following context (produced by `/idea` + `/design`):
- **Idea Document** — Problem exploration, chosen direction, key insights, user decisions log
- **Design Document** — UI design, component specs, interaction flows, API contracts, feasibility reviews

**How to get context:**
1. **File references (preferred for fresh context):** If the user provides file paths (e.g., `/prd docs/squad/2026-03-31-idea-feature.md docs/squad/2026-03-31-design-feature.md`), read those files to load the Idea and Design Documents.
2. **Same conversation:** If this context is available from previous `/idea` and `/design` runs in the same conversation, use it directly.
3. **Ask the user:** If neither is available, ask the user to provide or paste the Idea and Design Documents.

---

## Step 1: Check for UI Mockups

Before starting, search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, note their paths. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

---

## Step 2: PM — Formalize User Stories & Acceptance Criteria

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition.

Here is the Idea Document (the brainstormed and validated direction):
[Insert Idea Document content]

Here is the Design Document (the validated UX design and API contracts):
[Insert Design Document content]

The idea has been brainstormed and the design has been validated. Now formalize everything into User Stories with acceptance criteria.

1. Write formal User Stories based on the chosen direction and validated design.
2. Define measurable acceptance criteria — each must be binary (pass/fail) and testable.
3. Reference the Design Document for UI-specific criteria (component behavior, states, interactions).
4. Define what's in scope and what's explicitly out of scope.
5. Define success metrics — how will we know this worked?
6. You MUST include questions for the user if anything is ambiguous — but at this stage, most questions should already be answered from /idea and /design.

Produce the formal User Story with acceptance criteria.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the PM's User Stories and acceptance criteria to the user.
2. **Ask the user to confirm or adjust.** Do NOT proceed until the user responds.
3. If the user's feedback changes scope, update and re-present.

---

## Step 3: Tech Lead — Feasibility & Scope

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent. Follow the instructions in the tech-lead agent definition (Framing mode).

Here is the PM's User Story and acceptance criteria (confirmed by the user):
[Insert PM output]

Here is the Design Document (validated UX design and API contracts):
[Insert Design Document content]

Here is the Idea Document (for context on the original brainstorm):
[Insert Idea Document content]

MANDATORY FIRST STEPS:
1. Read the project's dependency files (package.json, requirements.txt, etc.) to identify the exact stack and versions.
2. For each relevant library/framework, use resolve-library-id then query-docs via context7 to fetch current documentation.
3. Validate every PM acceptance criterion against the documented capabilities of the current stack.
4. Cross-reference with the Design Document — are the API contracts and component specs feasible?

DO NOT just validate and move on. Be skeptical:
- Challenge the PM's scope if it's too big.
- Propose simpler alternatives if they exist.
- Flag hidden complexity honestly.
- Define success metrics and technical quality bars.
- Surface tradeoffs the user needs to decide on.
- You MUST include at least 2 items in your "Questions & Tradeoffs for the User" section.

Then produce the requirements document with feasibility assessment, subtasks, and risks.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Tech Lead's feasibility assessment to the user — show concerns, tradeoffs, and questions.
2. **Ask the user to answer the Tech Lead's questions and confirm tradeoff decisions.** Do NOT proceed until the user responds.
3. If answers change the scope or requirements, update the PM's User Story accordingly and re-run the Tech Lead if needed.

---

## Step 4: Designer — Design Alignment Check

Use the Agent tool with `subagent_type: "claude-squad:designer"`.

Prompt the agent:
```
You are the Designer agent. Follow the instructions in the designer agent definition.

Here is the PM's User Story and acceptance criteria:
[Insert PM output]

Here is the Tech Lead's requirements and feasibility:
[Insert Tech Lead output]

Here is the Design Document (your earlier work from /design):
[Insert Design Document content]

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get, get_screenshot) to review them. Never use Read or Grep on .pen files.
{{END IF}}

The design was already created in /design. Your job now is to:
1. Confirm the Design Document aligns with the PM's formal acceptance criteria.
2. Flag any gaps — acceptance criteria that the design doesn't cover, or design elements not captured in criteria.
3. Verify the Tech Lead's subtasks capture all the UI work from the Design Document.
4. Flag any design updates needed based on the Tech Lead's feasibility feedback.

Keep it concise. The full design already exists — this is an alignment check.
```

Wait for Designer output before proceeding.

---

## Step 5: QA — Acceptance Criteria Test Strategy

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition.

Here is the PM's User Story and acceptance criteria:
[Insert PM output]

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

Here is the Design Document:
[Insert Design Document content]

Your job is to review the acceptance criteria from a testability perspective:
1. Is every acceptance criterion testable? Can you write a concrete test for each one?
2. Are there missing acceptance criteria? Things that should be tested but aren't specified?
3. Propose the high-level test strategy: what types of tests (unit, integration, e2e) will be needed?
4. Flag any criteria that are too vague to test — suggest how to make them more specific.
5. Identify edge cases and error scenarios that should be added as acceptance criteria.

Produce a brief Test Strategy Assessment — not the full test plan (that comes in /implement).
```

Wait for QA output before proceeding.

---

## Gate: User Confirms PRD

**Requirements are only confirmed when:**
- All PM questions are answered
- All Tech Lead questions and tradeoffs are resolved
- Designer confirms design-requirements alignment
- QA confirms all acceptance criteria are testable
- The user explicitly confirms the PRD

---

## Output: PRD

After all steps complete, compile the PRD and **save it to a file**.

### Save the PRD

1. Ensure the `docs/squad/` directory exists in the project root (create it if needed).
2. Generate the filename using today's date and a kebab-case title derived from the feature: `docs/squad/yyyy-MM-dd-prd-<feature-title>.md` (e.g., `docs/squad/2026-03-27-prd-user-authentication.md`).
3. Write the full PRD content below to that file.

```
## PRD: [Feature title]

### 1. User Story & Acceptance Criteria (PM)
[Full PM output — User Stories, acceptance criteria, scope, success metrics]

### 2. Requirements & Feasibility (Tech Lead)
[Full Tech Lead output — stack analysis, feasibility, subtasks, success metrics, risks]

### 3. Design Alignment (Designer)
[Designer's alignment check — gaps, updates needed, confirmation]

### 4. Test Strategy (QA)
[QA's testability assessment — test types needed, edge cases, missing criteria]

### Input Documents Referenced
| Document | Path |
|---|---|
| Idea Document | [path to idea doc] |
| Design Document | [path to design doc] |

### UI Mockups Referenced
| .pen file | Screens/components reviewed | Notes |
|---|---|---|
| [path] | [What was found] | [Concerns or observations] |
(or "No .pen files found in project")

### Stack Documentation Consulted
| Library/Framework | Version | Key docs consulted |
|---|---|---|
| [name] | [version] | [Topics verified] |

### Status
- Feasibility: [All clear / Issues flagged]
- Requirements: [X subtasks defined across Y acceptance criteria]
- Design Alignment: [Confirmed / Gaps identified]
- Test Strategy: [All criteria testable / Gaps identified]
```

### Next Steps

After saving, tell the user the file path and ask:

> **PRD saved to `docs/squad/yyyy-MM-dd-prd-<feature-title>.md`.**
>
> What would you like to do next?
> 1. **Adjust the PRD** — tell me what to change and I'll update the document.
> 2. **Move to Implementation** — clear the context and start a new conversation with: `/implement docs/squad/yyyy-MM-dd-idea-<concept-title>.md docs/squad/yyyy-MM-dd-design-<feature-title>.md docs/squad/yyyy-MM-dd-prd-<feature-title>.md`

Replace the placeholder paths with the actual file paths used.
