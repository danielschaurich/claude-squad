---
name: squad
description: Run the full 5-phase squad workflow (Discovery → Blueprint → Build → Quality → Release) end-to-end with documentation-driven development via context7.
---

# /squad — Full Development Squad

Run all 5 phases end-to-end in a single command. This is equivalent to running `/discovery` → `/implement` → Release, but without stopping for user confirmation between phases.

**Every technical decision is grounded in the current documentation of the project's stack via context7 MCP.**

> **Prefer the modular commands when you want control:**
> - `/discovery` — Run phases 1-2 only (Discovery + Blueprint). Review the plan before building.
> - `/implement` — Run phases 3-4 only (Build + Quality). Requires `/discovery` output as context.
> - `/squad` — Run everything (phases 1-5) including Release preparation.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions or writing code. No assumptions from training data — only what the docs confirm for the installed versions.

This is enforced at every phase:
- **Discovery:** Planner verifies feasibility against documented APIs
- **Blueprint:** Architect validates design patterns against docs; Test Planner uses documented test APIs
- **Build:** Dev verifies every API call before writing code; Reviewer checks doc compliance
- **Quality:** QA uses documented test APIs; Reviewer validates perf/security patterns from docs
- **Release:** Release agent verifies CI/CD and deploy tool docs

---

## Phase 1: Discovery

**Goal:** Understand the problem through conversation with the user. Challenge assumptions, explore alternatives, and only proceed when everyone agrees on what to build.

**CRITICAL: This phase is INTERACTIVE. Agents produce questions and concerns. You (the orchestrator) present them to the user and wait for answers. Do NOT skip the conversation.**

### Step 1.0: Check for UI Mockups

Before starting Discovery, search the project for `.pen` files (Pencil.dev designs):
```
Glob pattern: **/*.pen
```

If `.pen` files are found, note their paths — agents will use pencil MCP tools (`batch_get`, `get_screenshot`) to review them during their respective phases. These mockups are the source of truth for UI/UX decisions. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

### Step 1.1: PM — First Pass Analysis (NOT the final story)

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

### Step 1.2: Planner — Technical Discovery & Feasibility

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

## Phase 2: Blueprint

**Goal:** Design the technical solution and test strategy — validated against docs, discussed with the user, agreed by all.

**CRITICAL: This phase is INTERACTIVE. The Architect must present design options and tradeoffs, not just a final answer. The user decides on key design questions.**

### Step 2.1: Architect — Brainstorm & Technical Design

Use the Agent tool with `subagent_type: "claude-squad:architect"`.

Prompt the agent:
```
You are the Architect agent. Follow the instructions in the architect agent definition.

Here is the PM's User Story (confirmed by the user):
[Insert PM output]

Here is the Planner's requirements document (confirmed by the user):
[Insert planner output]

Here are the user's answers and decisions from Discovery:
[Insert relevant user answers/decisions]

MANDATORY: For every library/framework you reference, use resolve-library-id then query-docs via context7 to verify the APIs exist in the installed version.

IMPORTANT: Do NOT jump straight to a final design. Follow your brainstorming process first:
1. Research the current APIs — discover what's possible, not just what you already know
2. Study the existing codebase patterns
3. Generate at least 2-3 different approaches with trade-offs for each
4. Challenge your assumptions — what are you taking for granted?
5. Consider the "do less" option — could a simpler solution work?
6. If mockups exist, analyze how they influence component structure

DO NOT just present a final design. You must:
- Present multiple approaches for key decisions with pros/cons.
- Challenge complexity — propose simpler alternatives where possible.
- Flag risks and assumptions the user should confirm.
- You MUST include at least 2 items in your "Design Questions for the User" section.

Output BOTH the brainstorm exploration AND the architecture design with questions.
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Architect's design to the user — highlight key design decisions, alternatives considered, and questions.
2. **Ask the user to answer the Architect's design questions and confirm key decisions.** Do NOT proceed until the user responds.
3. If answers change the design significantly, re-run the Architect with updated context.

### Step 2.2: Test Planner — Test Strategy

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
```

Wait for Test Planner output before proceeding.

**Gate:** Present the Blueprint (Architecture + Test Plan) to the user. Specifically:
1. Summarize the key design decisions the Architect made (and which the user confirmed).
2. Show the test strategy overview.
3. Ask: "Are you happy with this plan? Anything you'd change before we start building?"
4. Do NOT enter Build phase until the user explicitly confirms.

---

## Phase 3: Build

**Goal:** Implement and review code with continuous testing.

### Step 3.1: Dev — Implementation

Use the Agent tool with `subagent_type: "claude-squad:dev"`.

If the Architect defined independent subtasks, you MAY run multiple Dev agents in parallel (one per subtask). Otherwise, use a single Dev agent.

Prompt the agent:
```
You are the Dev agent. Follow the instructions in the dev agent definition.

Here is the Architect's plan:
[Insert architect output]

Here is the Test Plan (you must write unit tests for your code):
[Insert test planner output]

MANDATORY: Before using any library/framework API, use resolve-library-id then query-docs via context7 to verify the exact function signature, parameters, and return type. Do NOT assume APIs exist — verify them.

Implement the code following the Architect's plan. Write unit tests for every piece of logic. Code without tests is not done.
```

Wait for all Dev agents to complete before proceeding.

### Step 3.2: Reviewer — Code Review

Use the Agent tool with `subagent_type: "claude-squad:reviewer"`.

Prompt the agent:
```
You are the Reviewer agent. Follow the instructions in the reviewer agent definition (Code Review mode — Phase 3).

Review all code changes made by the Dev agent(s). Check the git diff to see what was changed.

The Architect's plan was:
[Insert architect output]

MANDATORY: For any framework/library API usage that looks non-standard or unfamiliar, use resolve-library-id then query-docs via context7 to verify correctness. Flag any deprecated or non-existent API usage.

Focus on: correctness, simplicity, documentation compliance, and unit test coverage.
```

**If CHANGES REQUESTED:**
- Run a Dev agent to fix the issues (with the same context7 mandate).
- Run the Reviewer again.
- Repeat until APPROVED (max 3 cycles, then ask the user).

### Step 3.3: QA — Continuous Testing

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition (Build phase — continuous testing).

Here is the Test Plan to implement:
[Insert test planner output]

MANDATORY: Use resolve-library-id then query-docs via context7 for every testing framework before writing tests. Verify all matchers, utilities, and patterns against current docs.

Implement every test defined in the Test Plan. Run them all. Report results.
```

**If FAIL:**
- Run a Dev agent to fix the failures.
- Run the Reviewer on the fixes.
- Run QA again.
- Repeat until PASS (max 3 cycles, then ask the user).

---

## Phase 4: Quality

**Goal:** Full validation — e2e, regression, UAT, performance, and security.

### Step 4.1: QA — E2E & Regression

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition (Quality phase — full validation).

Test Plan:
[Insert test planner output]

PM Acceptance Criteria:
[Insert PM output]

MANDATORY: Verify all testing APIs via context7 before writing tests.

Run the complete test suite including:
1. All tests from the Test Plan (should already pass from Phase 3)
2. E2E tests for the complete user journey
3. Full regression suite — run ALL existing tests
4. Map every PM acceptance criterion to passing tests

Report the complete QA Report with acceptance criteria mapping.
```

### Step 4.2: Reviewer — Performance & Security Validation

Use the Agent tool with `subagent_type: "claude-squad:reviewer"`.

Prompt the agent:
```
You are the Reviewer agent. Follow the instructions in the reviewer agent definition (Performance & Security mode — Phase 4).

Review all code changes for performance and security.

MANDATORY: Use resolve-library-id then query-docs via context7 to verify:
- Framework-recommended performance patterns (memoization, lazy loading, query optimization)
- Security best practices documented for the framework version in use
- OWASP top 10 applicability

The Architect's design was:
[Insert architect output]

Produce the Performance & Security Review report.
```

### Step 4.3: PM — UAT Validation

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition (UAT mode — Phase 4).

Here is the original User Story and acceptance criteria:
[Insert PM output from Phase 1]

Here is the QA Report:
[Insert QA report]

Here is the Performance & Security Review:
[Insert reviewer Phase 4 output]

Validate that what was built solves the original problem. Check every acceptance criterion against the test results. Produce the UAT Report.
```

**Gate:** If PM returns REJECTED or Reviewer found critical issues, loop back to Dev to fix, then re-validate. Max 2 cycles, then ask the user.

---

## Phase 5: Release

**Goal:** Prepare for a safe deploy.

### Step 5.1: Release — Deploy Preparation

Use the Agent tool with `subagent_type: "claude-squad:planner"`.

Prompt the agent:
```
You are the Release agent. Follow the instructions in the release agent definition.

MANDATORY: Use resolve-library-id then query-docs via context7 for:
- The project's CI/CD tools (GitHub Actions, etc.)
- The hosting/deploy platform (Vercel, AWS, Docker, etc.)
- Any build tools in the pipeline

Verify all configuration syntax and commands against current docs.

Produce the complete Release Plan: changes inventory, CI/CD validation, deploy checklist, rollback plan, and post-deploy monitoring guide.
```

---

## Final Summary

After all phases complete, output:

```
## Squad Complete

### Phase 1: Discovery
- User Story: [title]
- Acceptance Criteria: [count] defined

### Phase 2: Blueprint
- Architecture: [key decisions]
- Test Plan: [count] tests defined
- Stack validated against docs: [libraries checked]

### Phase 3: Build
- Files changed: [list]
- Unit tests written: [count]
- Code review: APPROVED
- Test suite: PASS

### Phase 4: Quality
- E2E tests: PASS/FAIL
- Regression: PASS/FAIL
- PM acceptance criteria: [X/Y] validated
- Performance review: [status]
- Security review: [status]

### Phase 5: Release
- Deploy checklist: [count] steps
- Rollback plan: defined
- Breaking changes: [yes/no — details]

### Documentation Compliance
All technical decisions verified against current stack documentation via context7:
[List of libraries/frameworks consulted with versions]
```
