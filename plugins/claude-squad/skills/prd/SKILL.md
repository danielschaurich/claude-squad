---
name: prd
description: Framing phase — PM brainstorms and defines the User Story, Planner validates feasibility. Output is the PRD that feeds into /design.
---

# /prd — Framing

Run the Framing phase: understand the problem, challenge assumptions, and produce a validated PRD. **All technical feasibility grounded in current stack docs via context7.**

The output of this command is a **PRD (Product Requirements Document)** — a self-contained artifact that feeds directly into `/design`.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role |
|-------|------|
| **PM** | Brainstorms, challenges assumptions, defines User Story with acceptance criteria |
| **Planner** | Validates feasibility against documented APIs, produces requirements with subtasks |

---

## Step 1: Check for UI Mockups

Before starting, search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, note their paths — agents will use pencil MCP tools (`batch_get`, `get_screenshot`) to review them. These mockups are the source of truth for UI/UX decisions. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

---

## Step 2: PM — Brainstorm & Define User Story

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition.

User request: {{input}}

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get) to review them — they contain design references for this feature. Never use Read or Grep on .pen files.
{{END IF}}

DO NOT produce a final User Story yet. Instead:
1. Restate the problem in your own words to check understanding.
2. Be skeptical — challenge assumptions, question scope, propose alternatives.
3. Brainstorm broadly: question the request, explore who's affected, consider alternatives, identify what could go wrong.
4. If mockups exist, analyze what they reveal about the user's vision.
5. Produce a DRAFT User Story with questions and concerns.
6. You MUST include at least 3 questions for the user in your "Questions for the User" section.

This is a first pass. The user will answer your questions before you finalize anything.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the PM's analysis to the user — show their understanding, concerns, alternatives, and questions.
2. **Ask the user to answer the PM's questions.** Do NOT proceed until the user responds.
3. If the user's answers raise new questions or change scope, run the PM agent again with the new context to produce the final User Story.
4. Repeat until the PM has no more blocking questions and the user confirms the User Story.

---

## Step 3: Planner — Technical Feasibility & Requirements

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the Planner agent. Follow the instructions in the planner agent definition.

Here is the PM's User Story (confirmed by the user):
[Insert PM output]

Here are the user's answers to PM questions:
[Insert user's answers from the conversation]

MANDATORY FIRST STEPS:
1. Read the project's dependency files (package.json, requirements.txt, etc.) to identify the exact stack and versions.
2. For each relevant library/framework, use resolve-library-id then query-docs via context7 to fetch current documentation.
3. Validate every PM acceptance criterion against the documented capabilities of the current stack.

DO NOT just validate and move on. Be skeptical:
- Challenge the PM's scope if it's too big.
- Propose simpler alternatives if they exist.
- Flag hidden complexity honestly.
- Surface tradeoffs the user needs to decide on.
- You MUST include at least 2 items in your "Questions & Tradeoffs for the User" section.

Then produce the requirements document with feasibility assessment, subtasks, and risks.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Planner's feasibility assessment to the user — show concerns, tradeoffs, and questions.
2. **Ask the user to answer the Planner's questions and confirm tradeoff decisions.** Do NOT proceed until the user responds.
3. If answers change the scope or requirements, update the PM's User Story accordingly and re-run the Planner if needed.

**Gate:** Requirements are only confirmed when:
- All PM questions are answered
- All Planner questions and tradeoffs are resolved
- The user explicitly confirms the requirements

---

## Output: PRD

After all steps complete, compile and present the PRD to the user. **This is the artifact that feeds into `/design`.**

```
## PRD: [Feature title]

### 1. User Story (PM)
[Full PM output — brainstorm, User Story, acceptance criteria, out of scope]

### 2. Requirements (Planner)
[Full Planner output — stack analysis, feasibility, subtasks, risks]

### UI Mockups Referenced
| .pen file | Screens/components reviewed | Used by |
|---|---|---|
| [path] | [What was found] | [Which agents referenced it] |
(or "No .pen files found in project")

### Stack Documentation Consulted
| Library/Framework | Version | Key docs consulted |
|---|---|---|
| [name] | [version] | [Topics verified] |

### Status
- Feasibility: [All clear / Issues flagged]
- Requirements: [X subtasks defined across Y acceptance criteria]
```

**Ask the user:** "PRD complete. Run `/design` to start designing, or adjust the plan first."
