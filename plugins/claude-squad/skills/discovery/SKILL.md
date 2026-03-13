---
name: discovery
description: Run Discovery + Blueprint phases — produces User Story, validated requirements, architecture, and test plan. Output is the input for /implement.
---

# /discovery — Discovery & Blueprint

Run phases 1-2 of the squad workflow: understand the problem, validate feasibility, design the solution, and define the test strategy. **All technical decisions grounded in current stack docs via context7.**

The output of this command is a **Discovery & Blueprint Document** — a self-contained artifact that can be passed directly to `/implement` to execute the Build and Quality phases.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Phase 1: Discovery

**Goal:** Understand the problem, validate feasibility, produce clear requirements.

### Step 1.0: Check for UI Mockups

Before starting Discovery, search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, note their paths — agents will use pencil MCP tools (`batch_get`, `get_screenshot`) to review them during their respective phases. These mockups are the source of truth for UI/UX decisions. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

### Step 1.1: PM — Brainstorm & Define User Story

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition.

User request: {{input}}

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get) to review them — they contain design references for this feature. Never use Read or Grep on .pen files.
{{END IF}}

IMPORTANT: Do NOT jump straight to writing the User Story. Follow your brainstorming process first:
1. Question the request — what's the real pain? What assumptions are embedded?
2. Explore who's affected and their different contexts
3. Consider 2-3 alternative ways to frame or solve this problem
4. Identify what could go wrong or what's missing
5. If mockups exist, analyze what they reveal about the user's vision

THEN write the User Story with acceptance criteria, informed by your brainstorm.

Output BOTH the brainstorm thinking AND the final User Story.
```

Wait for PM output before proceeding.

### Step 1.2: Planner — Technical Discovery & Feasibility

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the Planner agent. Follow the instructions in the planner agent definition.

Here is the PM's User Story:
[Insert PM output]

MANDATORY FIRST STEPS:
1. Read the project's dependency files (package.json, requirements.txt, etc.) to identify the exact stack and versions.
2. For each relevant library/framework, use resolve-library-id then query-docs via context7 to fetch current documentation.
3. Validate every PM acceptance criterion against the documented capabilities of the current stack.

Then produce the full requirements document with feasibility assessment, subtasks, and risks.
```

Wait for Planner output before proceeding.

**Gate:** If the Planner flags any acceptance criteria as infeasible, present the findings to the user and ask how to proceed before continuing.

---

## Phase 2: Blueprint

**Goal:** Design the technical solution and test strategy — validated against docs, agreed by the squad.

### Step 2.1: Architect — Brainstorm & Technical Design

Use the Agent tool with `subagent_type: "claude-squad:architect"`.

Prompt the agent:
```
You are the Architect agent. Follow the instructions in the architect agent definition.

Here is the PM's User Story (including their brainstorm):
[Insert PM output]

Here is the Planner's requirements document:
[Insert planner output]

MANDATORY: For every library/framework you reference, use resolve-library-id then query-docs via context7 to verify the APIs exist in the installed version.

IMPORTANT: Do NOT jump straight to a design. Follow your brainstorming process first:
1. Research the current APIs — discover what's possible, not just what you already know
2. Study the existing codebase patterns
3. Generate at least 2-3 different approaches with trade-offs for each
4. Challenge your assumptions — what are you taking for granted?
5. Consider the "do less" option — could a simpler solution work?
6. If mockups exist, analyze how they influence component structure

THEN pick the best approach (with reasoning) and produce the full architecture.

Output BOTH the brainstorm exploration AND the final architecture design.
```

Wait for Architect output before proceeding.

### Step 2.2: Test Planner — Test Strategy

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the Test Planner agent. Follow the instructions in the test-planner agent definition.

Here is the PM's User Story (acceptance criteria to map):
[Insert PM output]

Here is the Planner's requirements:
[Insert planner output]

Here is the Architect's design:
[Insert architect output]

MANDATORY: Read the project's test configuration. Use resolve-library-id then query-docs via context7 for every testing framework and utility. Only use documented test APIs.

Define the complete test plan — every PM acceptance criterion must map to at least one test. Include the doc reference for test APIs you specify.
```

Wait for Test Planner output before proceeding.

---

## Output: Discovery & Blueprint Document

After all steps complete, compile and present the full document to the user. **This is the artifact that feeds into `/implement`.**

```
## Discovery & Blueprint Document

### 1. User Story (PM)
[Full PM output — brainstorm, User Story, acceptance criteria, out of scope]

### 2. Requirements (Planner)
[Full Planner output — stack analysis, feasibility, subtasks, risks]

### 3. Architecture (Architect)
[Full Architect output — brainstorm with approaches explored, chosen design, file structure, types, patterns, implementation order, doc references]

### 4. Test Plan (Test Planner)
[Full Test Planner output — test cases, edge cases, regression checks, doc references]

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
- Architecture: [Designed and doc-validated]
- Test plan: [X tests defined across Y acceptance criteria]
```

**Ask the user:** "Discovery & Blueprint complete. Run `/implement` to start building, or adjust the plan first."
