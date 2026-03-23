---
name: prd
description: Framing phase — PM defines the problem, Tech Lead validates feasibility, Designer provides UX perspective. Output is the PRD that feeds into /design.
---

# /prd — Framing

Run the Framing phase: define the problem, validate feasibility, scope the work, and produce a PRD. **All technical decisions grounded in current stack docs via context7.**

The output of this command is a **PRD (Product Requirements Document)** — a self-contained artifact that feeds directly into `/design`.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role in this phase |
|-------|-------------------|
| **PM** | Brainstorms, challenges assumptions, defines User Story with acceptance criteria, success metrics |
| **Tech Lead** | Validates feasibility against stack docs, scopes work, defines subtasks and technical direction |
| **Designer** | Reviews mockups (pencil MCP), provides early UX perspective, flags design concerns |

---

## Step 1: Check for UI Mockups

Before starting, search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, note their paths — the Designer will review them via pencil MCP tools. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

---

## Step 2: PM — Problem Definition & User Story

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

## Step 3: Tech Lead — Feasibility & Scope

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent. Follow the instructions in the tech-lead agent definition (Framing mode).

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

## Step 4: Designer — Early UX Perspective

Use the Agent tool with `subagent_type: "claude-squad:designer"`.

Prompt the agent:
```
You are the Designer agent. Follow the instructions in the designer agent definition.

Here is the PM's User Story:
[Insert PM output]

Here is the Tech Lead's requirements and feasibility:
[Insert Tech Lead output]

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get, get_screenshot) to review them. Never use Read or Grep on .pen files.
{{END IF}}

This is the Framing phase — you're not doing full design yet. Your job is to:
1. Review any existing mockups and assess their completeness.
2. Flag UX concerns early — missing states, accessibility issues, interaction gaps.
3. Identify open design questions that should be resolved before full design begins.
4. Provide a brief UX perspective on the PM's acceptance criteria — are they testable from a UI standpoint?

Keep it concise. Full component specs and interaction flows come in /design.
```

Wait for Designer output before proceeding.

---

## Gate: User Confirms PRD

**Requirements are only confirmed when:**
- All PM questions are answered
- All Tech Lead questions and tradeoffs are resolved
- Designer UX concerns are acknowledged
- The user explicitly confirms the PRD

---

## Output: PRD

After all steps complete, compile and present the PRD to the user.

```
## PRD: [Feature title]

### 1. User Story (PM)
[Full PM output — brainstorm, User Story, acceptance criteria, out of scope]

### 2. Requirements & Feasibility (Tech Lead)
[Full Tech Lead output — stack analysis, feasibility, subtasks, success metrics, risks]

### 3. UX Perspective (Designer)
[Designer's early assessment — mockup review, UX concerns, open design questions]

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
- UX: [Concerns raised / All clear]
```

**Ask the user:** "PRD complete. Run `/design` to start designing, or adjust the plan first."
