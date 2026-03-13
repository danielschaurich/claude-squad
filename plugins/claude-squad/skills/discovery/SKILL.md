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

### Step 1.1: PM — Define User Story

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition.

User request: {{input}}

Translate this into a clear User Story with measurable acceptance criteria. Focus on the PROBLEM, not the solution. Define what "done" looks like.
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

### Step 2.1: Architect — Technical Design

Use the Agent tool with `subagent_type: "claude-squad:architect"`.

Prompt the agent:
```
You are the Architect agent. Follow the instructions in the architect agent definition.

Here is the PM's User Story:
[Insert PM output]

Here is the Planner's requirements document:
[Insert planner output]

MANDATORY: For every library/framework you reference in your design, use resolve-library-id then query-docs via context7 to verify the APIs exist in the installed version. Include doc references for every technical decision.

Design the architecture: file structure, types, patterns, implementation order. Every pattern must be backed by documentation.
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
[Full PM output — User Story, acceptance criteria, out of scope]

### 2. Requirements (Planner)
[Full Planner output — stack analysis, feasibility, subtasks, risks]

### 3. Architecture (Architect)
[Full Architect output — file structure, types, patterns, implementation order, doc references]

### 4. Test Plan (Test Planner)
[Full Test Planner output — test cases, edge cases, regression checks, doc references]

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
