---
name: implement
description: Build & Validation phase — Dev implements code, Reviewer ensures quality, QA validates tests. Expects PRD + Design Document as input.
---

# /implement — Build & Validation

Run the Build and Validation phases: implement the code, review it, test it, and validate quality. **All technical decisions grounded in current stack docs via context7.**

This command expects the **PRD** (from `/prd`) and **Design Document** (from `/design`) as context. If not provided, it will ask the user for: the architecture plan, test plan, and acceptance criteria.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions or writing code. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role |
|-------|------|
| **Dev** | Implements code following the Architect's plan, writes unit tests |
| **Reviewer** | Reviews correctness, simplicity, security, doc compliance, performance |
| **QA** | Implements and runs all tests from the Test Plan, validates acceptance criteria |

---

## Input

This skill requires the following context (produced by `/prd` + `/design`):
- **PM output** — User Story and acceptance criteria
- **Planner output** — Requirements and subtasks
- **Architect output** — File structure, types, patterns, implementation order
- **Test Planner output** — Test plan with all test cases
- **UI mockups** — Any `.pen` file paths identified during Design (if applicable)

If this context is available from previous `/prd` and `/design` runs in the same conversation, use it directly. If not, ask the user to provide or paste the PRD and Design Document.

If the Design Document references `.pen` files, pass their paths to Dev and Reviewer agents so they can reference the mockups using pencil MCP tools. **Never use Read or Grep on .pen files — only pencil MCP tools can read their contents.**

---

## Phase 1: Build

**Goal:** Implement and review code with continuous testing.

### Step 1.1: Dev — Implementation

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

### Step 1.2: Reviewer — Code Review

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

### Step 1.3: QA — Continuous Testing

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

## Phase 2: Validation

**Goal:** Full validation — e2e, regression, UAT, performance, and security.

### Step 2.1: QA — E2E & Regression

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
1. All tests from the Test Plan (should already pass from Build phase)
2. E2E tests for the complete user journey
3. Full regression suite — run ALL existing tests
4. Map every PM acceptance criterion to passing tests

Report the complete QA Report with acceptance criteria mapping.
```

### Step 2.2: Reviewer — Performance & Security Validation

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

### Step 2.3: PM — UAT Validation

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition (UAT mode — Phase 4).

Here is the original User Story and acceptance criteria:
[Insert PM output from PRD]

Here is the QA Report:
[Insert QA report]

Here is the Performance & Security Review:
[Insert reviewer output]

Validate that what was built solves the original problem. Check every acceptance criterion against the test results. Produce the UAT Report.
```

**Gate:** If PM returns REJECTED or Reviewer found critical issues, loop back to Dev to fix, then re-validate. Max 2 cycles, then ask the user.

---

## Output: Implementation Report

After all steps complete, present the final report:

```
## Implementation Complete

### Build
- Files changed: [list]
- Unit tests written: [count]
- Code review: APPROVED (cycles: [N])
- Test suite: PASS

### Validation
- E2E tests: PASS/FAIL
- Regression: PASS/FAIL
- PM acceptance criteria: [X/Y] validated
- Performance review: [status]
- Security review: [status]
- UAT: APPROVED/REJECTED

### Documentation Compliance
All implementations verified against current stack documentation via context7:
[List of libraries/frameworks consulted with versions]
```
