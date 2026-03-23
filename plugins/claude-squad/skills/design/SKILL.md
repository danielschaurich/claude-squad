---
name: design
description: Design & Prototype phase — Architect designs the technical solution, Test Planner defines test strategy. Uses pencil MCP for UI prototyping. Output feeds into /implement.
---

# /design — Design & Prototype

Run the Design phase: explore approaches, design the architecture, prototype the UI, and define the test strategy. **All technical decisions grounded in current stack docs via context7. UI prototyping via pencil MCP.**

This command expects the **PRD** produced by `/prd` as context. If not provided, it will ask the user for the User Story, acceptance criteria, and requirements.

The output of this command is a **Design Document** — a self-contained artifact that feeds directly into `/implement`.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role |
|-------|------|
| **Architect** | Explores approaches, designs technical solution validated against docs, prototypes UI via pencil MCP |
| **Test Planner** | Defines complete test strategy mapped to acceptance criteria |

---

## Input

This skill requires the following context (produced by `/prd`):
- **PM output** — User Story and acceptance criteria
- **Planner output** — Requirements, subtasks, and feasibility assessment

If this context is available from a previous `/prd` run in the same conversation, use it directly. If not, ask the user to provide or paste the PRD.

---

## Step 1: Check for UI Mockups

Search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, the Architect will use pencil MCP tools extensively during design. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

---

## Step 2: Architect — Brainstorm, Design & Prototype

Use the Agent tool with `subagent_type: "claude-squad:architect"`.

Prompt the agent:
```
You are the Architect agent. Follow the instructions in the architect agent definition.

Here is the PM's User Story (confirmed by the user):
[Insert PM output]

Here is the Planner's requirements document (confirmed by the user):
[Insert planner output]

Here are the user's answers and decisions from Framing:
[Insert relevant user answers/decisions]

MANDATORY: For every library/framework you reference, use resolve-library-id then query-docs via context7 to verify the APIs exist in the installed version.

IMPORTANT: Do NOT jump straight to a final design. Follow your brainstorming process first:
1. Research the current APIs — discover what's possible, not just what you already know
2. Study the existing codebase patterns
3. Generate at least 2-3 different approaches with trade-offs for each
4. Challenge your assumptions — what are you taking for granted?
5. Consider the "do less" option — could a simpler solution work?
6. If mockups exist, use pencil MCP tools (batch_get, get_screenshot) to analyze them deeply — they define the UI prototype. Map mockup screens to components.

DO NOT just present a final design. You must:
- Present multiple approaches for key decisions with pros/cons.
- Challenge complexity — propose simpler alternatives where possible.
- Flag risks and assumptions the user should confirm.
- If .pen mockups exist, include a prototype analysis: component breakdown, interactions, and how the design maps to the mockups.
- You MUST include at least 2 items in your "Design Questions for the User" section.

Output BOTH the brainstorm exploration AND the architecture design with questions.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Architect's design to the user — highlight key design decisions, alternatives considered, prototype analysis, and questions.
2. **Ask the user to answer the Architect's design questions and confirm key decisions.** Do NOT proceed until the user responds.
3. If answers change the design significantly, re-run the Architect with updated context.

---

## Step 3: Test Planner — Test Strategy

Use the Agent tool with `subagent_type: "claude-squad:test-planner"`.

Prompt the agent:
```
You are the Test Planner agent. Follow the instructions in the test-planner agent definition.

Here is the PM's User Story (acceptance criteria to map):
[Insert PM output]

Here is the Planner's requirements:
[Insert planner output]

Here is the Architect's design (confirmed by the user):
[Insert architect output]

MANDATORY: Read the project's test configuration. Use resolve-library-id then query-docs via context7 for every testing framework and utility. Only use documented test APIs.

Define the complete test plan — every PM acceptance criterion must map to at least one test. Include the doc reference for test APIs you specify.

If UI mockups exist (.pen files), include visual validation steps — QA should compare implementation against mockups using pencil MCP tools.
```

Wait for Test Planner output before proceeding.

---

## Output: Design Document

After all steps complete, compile and present the Design Document to the user. **This is the artifact that feeds into `/implement`.**

```
## Design Document: [Feature title]

### 1. Architecture (Architect)
[Full Architect output — brainstorm with approaches explored, chosen design, file structure, types, patterns, implementation order, doc references]

### 2. Prototype Analysis
[If .pen files found: component breakdown from mockups, screen-to-component mapping, interactions identified]
(or "No .pen files found in project")

### 3. Test Plan (Test Planner)
[Full Test Planner output — test cases, edge cases, regression checks, doc references]

### Documentation References
| Library/Framework | Version | Key APIs/patterns used | Doc source |
|---|---|---|---|
| [name] | [version] | [Specific APIs] | [What context7 returned] |

### Status
- Architecture: [Designed and doc-validated]
- Prototype: [Mapped to mockups / No mockups]
- Test plan: [X tests defined across Y acceptance criteria]
```

**Ask the user:** "Design complete. Run `/implement` to start building, or adjust the design first."
