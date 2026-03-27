---
name: design
description: Design phase — Designer creates UX flows and prototypes, PM validates against user needs, Engineers review technical feasibility. Output feeds into /implement.
---

# /design — Design & Prototype

Run the Design phase: create UX flows, prototype the UI, and validate the design with PM and Engineers. **UI prototyping via pencil MCP. All technical decisions grounded in current stack docs via context7.**

This command expects the **PRD** produced by `/prd` as context. If not provided, it will ask the user for the User Story, acceptance criteria, and requirements.

The output of this command is a **Design Document** — a self-contained artifact that feeds directly into `/implement`.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role in this phase |
|-------|-------------------|
| **Designer** | Leads — creates UX flows, component specs, prototypes via pencil MCP, interaction design |
| **PM** | Validates designs against User Story and acceptance criteria, represents the user |
| **Frontend Dev** | Reviews design feasibility from frontend perspective, flags implementation concerns |
| **Backend Dev** | Reviews design feasibility from backend perspective, proposes API contracts and data shapes |

---

## Input

This skill requires the following context (produced by `/prd`):
- **PM output** — User Story and acceptance criteria
- **Tech Lead output** — Requirements, subtasks, and feasibility assessment
- **Designer output** — Early UX perspective

**How to get the PRD context:**
1. **File reference (preferred for fresh context):** If the user provides a file path (e.g., `/design docs/squad/2026-03-27-prd-feature.md`), read that file to load the PRD.
2. **Same conversation:** If this context is available from a previous `/prd` run in the same conversation, use it directly.
3. **Ask the user:** If neither is available, ask the user to provide or paste the PRD.

---

## Step 1: Designer — UX Flows, Prototypes & Component Specs

Use the Agent tool with `subagent_type: "claude-squad:designer"`.

Prompt the agent:
```
You are the Designer agent. Follow the instructions in the designer agent definition.

Here is the PM's User Story:
[Insert PM output]

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get, get_screenshot) to analyze them deeply. Never use Read or Grep on .pen files.
{{END IF}}

This is the full Design phase. You must:
1. Review all mockups via pencil MCP (if they exist).
2. Define complete UX flows — how does the user move through the feature?
3. Break down every screen into a component tree.
4. Spec every component: props, states (default/loading/error/empty/success), interactions, responsive behavior, accessibility.
5. Define interaction flows with success and error paths.
6. Flag any UX improvements over the mockups.
7. Identify reusable patterns from the existing codebase.
8. Include your "Design Questions for the User" section.

Output the full UI Design document.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Designer's work to the user — highlight key UX decisions, component breakdown, and questions.
2. **Ask the user to answer the Designer's questions.** Do NOT proceed until the user responds.
3. If answers change the design significantly, re-run the Designer with updated context.

---

## Step 2: PM — Design Validation

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition.

Here is the original User Story and acceptance criteria:
[Insert PM output from PRD]

Here is the Designer's UI Design:
[Insert Designer output]

Validate the design against the User Story:
1. Does every acceptance criterion have a clear UI representation in the design?
2. Are there acceptance criteria that the design doesn't address?
3. Does the design introduce scope that wasn't in the User Story? Flag it.
4. From the user's perspective, does this design solve the original problem?

Produce a brief Design Validation — APPROVED or CHANGES NEEDED with specific feedback.
```

**If CHANGES NEEDED:** Present the PM's feedback to the user, then re-run the Designer to address the issues.

---

## Step 3: Engineers — Design Review (run in parallel)

Run Frontend Dev and Backend Dev reviews in parallel.

### Frontend Dev Review

Use the Agent tool with `subagent_type: "claude-squad:frontend-dev"`.

Prompt the agent:
```
You are the Frontend Dev agent. Follow the instructions in the frontend-dev agent definition.

Here is the Designer's UI Design:
[Insert Designer output]

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

This is a DESIGN REVIEW, not implementation. Your job is to:
1. Review the component specs — are they implementable with the current frontend stack?
2. Use resolve-library-id then query-docs via context7 to verify that the UI patterns, components, and interactions the Designer specified are achievable.
3. Flag anything that's overly complex, impossible with the current stack, or could be simplified.
4. Suggest implementation approaches for tricky interactions.
5. Identify any missing specs you'd need to implement this (e.g., animation details, responsive breakpoints).

Produce a brief Frontend Feasibility Review.
```

### Backend Dev Review

Use the Agent tool with `subagent_type: "claude-squad:backend-dev"`.

Prompt the agent:
```
You are the Backend Dev agent. Follow the instructions in the backend-dev agent definition.

Here is the Designer's UI Design:
[Insert Designer output]

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

This is a DESIGN REVIEW, not implementation. Your job is to:
1. Review what data and APIs the UI design requires.
2. Use resolve-library-id then query-docs via context7 to verify backend patterns.
3. Propose API contracts — endpoints, request/response shapes, auth patterns.
4. Flag any backend complexity the design implies (real-time updates, file uploads, complex queries).
5. Identify what already exists in the codebase vs. what needs to be built.

Produce the API Contracts and a brief Backend Feasibility Review.
```

Wait for both reviews before proceeding.

**If either engineer flags blockers:** Present to the user, then re-run the Designer to adjust if needed.

---

## Gate: User Confirms Design

Present the full Design Document to the user:
1. Summarize the key UX decisions.
2. Show the PM's validation status.
3. Show the engineers' feasibility reviews and API contracts.
4. Ask: "Are you happy with this design? Anything you'd change before we start building?"
5. Do NOT proceed to `/implement` until the user explicitly confirms.

---

## Output: Design Document

After all steps complete, compile the Design Document and **save it to a file**.

### Save the Design Document

1. Ensure the `docs/squad/` directory exists in the project root (create it if needed).
2. Generate the filename using today's date and a kebab-case title derived from the feature: `docs/squad/yyyy-MM-dd-design-<feature-title>.md` (e.g., `docs/squad/2026-03-27-design-user-authentication.md`).
3. Write the full Design Document content below to that file.

```
## Design Document: [Feature title]

### 1. UI Design (Designer)
[Full Designer output — mockup analysis, component tree, component specs, interaction flows, states, accessibility]

### 2. PM Design Validation
[PM's validation — approved/changes needed, acceptance criteria coverage]

### 3. Frontend Feasibility (Frontend Dev)
[Frontend review — implementability, stack verification, concerns, suggestions]

### 4. API Contracts & Backend Feasibility (Backend Dev)
[API contracts — endpoints, data shapes, auth. Backend review — complexity, existing code reuse]

### PRD Reference
[Path to the PRD file used as input, e.g., docs/squad/2026-03-27-prd-user-authentication.md]

### Documentation References
| Library/Framework | Version | Key APIs/patterns verified | Source |
|---|---|---|---|
| [name] | [version] | [Specific APIs] | [context7] |

### Status
- UI Design: [Complete]
- PM Validation: [APPROVED / Changes addressed]
- Frontend Feasibility: [Clear / Concerns noted]
- Backend Feasibility: [Clear / Concerns noted]
- API Contracts: [Defined]
```

### Next Steps

After saving, tell the user the file path and ask:

> **Design Document saved to `docs/squad/yyyy-MM-dd-design-<feature-title>.md`.**
>
> What would you like to do next?
> 1. **Adjust the design** — tell me what to change and I'll update the document.
> 2. **Move to Implementation** — clear the context and start a new conversation with: `/implement docs/squad/yyyy-MM-dd-prd-<feature-title>.md docs/squad/yyyy-MM-dd-design-<feature-title>.md`

Replace the placeholder paths with the actual file paths used.
