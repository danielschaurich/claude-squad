---
name: squad
description: Run a full 5-phase squad workflow (Discovery → Blueprint → Build → Quality → Release) with documentation-driven development via context7.
---

# /squad — Development Squad Orchestrator

Run a full development squad workflow across 5 phases. **Every technical decision is grounded in the current documentation of the project's stack via context7 MCP.**

## Core Principle: Documentation-Driven Development

Every agent in this squad MUST consult the current documentation for the libraries and frameworks in use before making decisions or writing code. No assumptions based on prior knowledge — only what the docs confirm for the installed versions.

This is enforced at every phase:
- **Discovery:** Planner verifies feasibility against documented APIs
- **Blueprint:** Architect validates design patterns against docs; Test Planner uses documented test APIs
- **Build:** Dev verifies every API call before writing code; Reviewer checks doc compliance
- **Quality:** QA uses documented test APIs; Reviewer validates perf/security patterns from docs
- **Release:** Release agent verifies CI/CD and deploy tool docs

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

**Gate:** Present the Blueprint (Architecture + Test Plan) to the user. Ask for confirmation before entering Build phase. This is the "execution plan the team agrees on."

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

## Phase Summary

After all phases complete, output the final summary to the user:

```
## Squad Complete ✓

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
