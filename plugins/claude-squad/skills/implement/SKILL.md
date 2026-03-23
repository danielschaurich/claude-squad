---
name: implement
description: Build & Validation phase — Frontend and Backend Devs implement, Tech Lead reviews code, QA tests, PM validates acceptance criteria.
---

# /implement — Build & Validation

Run the Build and Validation phases: implement the code, review it, test it, and validate quality. **All technical decisions grounded in current stack docs via context7.**

This command expects the **PRD** (from `/prd`) and **Design Document** (from `/design`) as context. If not provided, it will ask the user for: the architecture plan, API contracts, component specs, and acceptance criteria.

## Core Principle: Documentation-Driven Development

Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions or writing code. No assumptions from training data — only what the docs confirm for the installed versions.

---

## Agent Team

| Agent | Role in this phase |
|-------|-------------------|
| **Frontend Dev** | Implements UI components, pages, interactions from Designer specs |
| **Backend Dev** | Implements APIs, data models, business logic from API contracts |
| **Tech Lead** | Reviews all code for correctness, simplicity, security, performance |
| **QA** | Defines test plan, implements and runs all tests, validates acceptance criteria |
| **PM** | UAT — validates the result against the original User Story |

---

## Input

This skill requires the following context (produced by `/prd` + `/design`):
- **PM output** — User Story and acceptance criteria
- **Tech Lead output** — Requirements and subtasks
- **Designer output** — Component specs, interaction flows, states
- **Frontend Dev output** — Frontend feasibility notes
- **Backend Dev output** — API contracts, backend feasibility notes
- **UI mockups** — Any `.pen` file paths (if applicable)

If this context is available from previous `/prd` and `/design` runs in the same conversation, use it directly. If not, ask the user to provide or paste the PRD and Design Document.

---

## Phase 1: Build

**Goal:** Implement frontend and backend in parallel, review code, test continuously.

### Step 1.1: QA — Test Plan

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition (Test Planning mode).

Here is the PM's User Story and acceptance criteria:
[Insert PM output]

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

Here is the Designer's UI Design:
[Insert Designer output]

Here are the API Contracts:
[Insert Backend Dev output from /design]

MANDATORY: Read the project's test configuration. Use resolve-library-id then query-docs via context7 for every testing framework and utility. Only use documented test APIs.

Define the complete test plan:
- Every PM acceptance criterion must map to at least one test.
- Include unit, integration, and e2e test cases.
- Include edge cases and error scenarios.
- If UI mockups exist (.pen files), include visual validation steps.
```

Wait for QA test plan before proceeding.

### Step 1.2: Frontend Dev + Backend Dev — Implementation (run in parallel)

Run both Dev agents in parallel.

#### Frontend Dev

Use the Agent tool with `subagent_type: "claude-squad:frontend-dev"`.

Prompt the agent:
```
You are the Frontend Dev agent. Follow the instructions in the frontend-dev agent definition.

Here is the Designer's UI Design (your implementation spec):
[Insert Designer output]

Here are the API Contracts (what your frontend consumes):
[Insert Backend Dev API contracts from /design]

Here is the QA Test Plan (you must write unit tests):
[Insert QA test plan]

MANDATORY: Before using any library/framework API, use resolve-library-id then query-docs via context7 to verify the exact function signature, parameters, and return type.

Implement the frontend following the Designer's specs. Write unit tests for every component and piece of logic. Code without tests is not done.
```

#### Backend Dev

Use the Agent tool with `subagent_type: "claude-squad:backend-dev"`.

Prompt the agent:
```
You are the Backend Dev agent. Follow the instructions in the backend-dev agent definition.

Here is the Tech Lead's requirements:
[Insert Tech Lead output]

Here are the API Contracts you defined (implement them exactly):
[Insert Backend Dev API contracts from /design]

Here is the QA Test Plan (you must write unit tests):
[Insert QA test plan]

MANDATORY: Before using any library/framework API, use resolve-library-id then query-docs via context7 to verify the exact function signature, parameters, and return type.

Implement the backend following the API contracts. Write unit tests for every piece of logic. Code without tests is not done.
```

Wait for both Dev agents to complete before proceeding.

### Step 1.3: Tech Lead — Code Review

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent. Follow the instructions in the tech-lead agent definition (Code Review mode).

Review all code changes made by the Frontend Dev and Backend Dev. Check the git diff to see what was changed.

The Designer's specs were:
[Insert Designer output]

The API Contracts were:
[Insert Backend Dev API contracts from /design]

MANDATORY: For any framework/library API usage that looks non-standard or unfamiliar, use resolve-library-id then query-docs via context7 to verify correctness.

Review for: correctness, simplicity, security, performance, documentation compliance, and unit test coverage.
```

**If CHANGES REQUESTED:**
- Run the appropriate Dev agent(s) to fix the issues.
- Run the Tech Lead again.
- Repeat until APPROVED (max 3 cycles, then ask the user).

### Step 1.4: QA — Continuous Testing

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition (Build — continuous testing).

Here is the Test Plan you defined:
[Insert QA test plan]

MANDATORY: Use resolve-library-id then query-docs via context7 for every testing framework before writing tests.

Implement every test from the test plan. Run them all. Report results.
```

**If FAIL:**
- Run the appropriate Dev agent(s) to fix the failures.
- Run the Tech Lead on the fixes.
- Run QA again.
- Repeat until PASS (max 3 cycles, then ask the user).

---

## Phase 2: Validation

**Goal:** Full validation — e2e, regression, acceptance criteria, performance, security, UAT.

### Step 2.1: QA — E2E & Regression

Use the Agent tool with `subagent_type: "claude-squad:qa"`.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition (Validation — full suite).

Test Plan:
[Insert QA test plan]

PM Acceptance Criteria:
[Insert PM output]

MANDATORY: Verify all testing APIs via context7 before writing tests.

Run the complete test suite:
1. All tests from the Test Plan (should already pass from Build phase)
2. E2E tests for the complete user journey
3. Full regression suite — run ALL existing tests
4. Map every PM acceptance criterion to passing tests

Report the complete QA Report with acceptance criteria mapping.
```

### Step 2.2: Tech Lead — Performance & Security Review

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent. Follow the instructions in the tech-lead agent definition (Code Review mode — performance & security focus).

Review all code changes for performance and security.

MANDATORY: Use resolve-library-id then query-docs via context7 to verify:
- Framework-recommended performance patterns
- Security best practices documented for the framework version
- OWASP top 10 applicability

Produce the Performance & Security Review report.
```

### Step 2.3: PM — UAT Validation

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent. Follow the instructions in the PM agent definition (UAT mode).

Here is the original User Story and acceptance criteria:
[Insert PM output from PRD]

Here is the QA Report:
[Insert QA report]

Here is the Tech Lead's Performance & Security Review:
[Insert Tech Lead review]

Validate that what was built solves the original problem. Check every acceptance criterion against the test results. Produce the UAT Report.
```

**Gate:** If PM returns REJECTED or Tech Lead found critical issues, loop back to the appropriate Dev to fix, then re-validate. Max 2 cycles, then ask the user.

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
