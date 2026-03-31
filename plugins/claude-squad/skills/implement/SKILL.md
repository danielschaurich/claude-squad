---
name: implement
description: Build & Validation phase — Frontend and Backend Devs implement using TDD and DRY principles, Tech Lead reviews code quality and security, QA validates, PM runs UAT. No over-engineering.
---

# /implement — Build & Validation

Run the Build and Validation phases: implement the code, review it, test it, and validate quality. **All technical decisions grounded in current stack docs via context7.**

This command expects the **Idea Document** (from `/idea`), **Design Document** (from `/design`), and **PRD** (from `/prd`) as context. If not provided, it will ask the user for: the architecture plan, API contracts, component specs, and acceptance criteria.

## Core Principles

### Documentation-Driven Development
Every agent MUST consult the current documentation for the libraries and frameworks in use before making decisions or writing code. No assumptions from training data — only what the docs confirm for the installed versions.

### TDD — Test-Driven Development
**Write tests FIRST, then write the code to make them pass.** This is not optional — it is the core development methodology for this phase.
1. Write a failing test that defines the expected behavior.
2. Write the minimal code to make the test pass.
3. Refactor while keeping all tests green.
4. Repeat for each piece of functionality.

### DRY — Don't Repeat Yourself
Every piece of knowledge must have a single, unambiguous, authoritative representation.
- Before writing new code, check if similar logic already exists in the codebase.
- Extract shared logic into reusable functions ONLY when duplication actually exists (not speculatively).
- Three similar lines of code is better than a premature abstraction.

### No Over-Engineering
Build exactly what's needed, nothing more. This means:
- **No speculative abstractions** — Don't build for hypothetical future requirements.
- **No unnecessary indirection** — If a direct approach works, use it. Don't add layers "just in case."
- **No gold plating** — Implement the acceptance criteria, not your ideal version.
- **No premature optimization** — Make it work correctly first. Optimize only when there's evidence of a problem.
- **No feature creep** — If it's not in the PRD, don't build it. Period.
- **YAGNI** — You Aren't Gonna Need It. If in doubt, leave it out.

### Code Quality
- Clean, readable code that follows existing codebase conventions.
- Meaningful variable and function names — code should be self-documenting.
- Small, focused functions that do one thing well.
- No dead code, unused imports, or commented-out code.

### Security
Security is built in, not bolted on. Every implementation must consider:
- **Input validation** at all system boundaries (user input, external APIs, file uploads).
- **Output encoding** to prevent XSS.
- **Parameterized queries** to prevent SQL injection.
- **Authentication & authorization** checks on every protected endpoint.
- **Secrets management** — no hardcoded credentials, API keys, or tokens.
- **OWASP Top 10** awareness in every code review.
- **Principle of least privilege** — minimal permissions, minimal exposure.

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

This skill requires the following context (produced by `/idea` + `/design` + `/prd`):
- **Idea Document** — Chosen direction, problem exploration, key insights
- **Design Document** — Component specs, interaction flows, states, API contracts
- **PRD** — User Stories, acceptance criteria, requirements, subtasks, test strategy
- **UI mockups** — Any `.pen` file paths (if applicable)

**How to get context:**
1. **File references (preferred for fresh context):** If the user provides file paths (e.g., `/implement docs/squad/2026-03-31-idea-feature.md docs/squad/2026-03-31-design-feature.md docs/squad/2026-03-31-prd-feature.md`), read those files to load all documents.
2. **Same conversation:** If this context is available from previous runs in the same conversation, use it directly.
3. **Ask the user:** If neither is available, ask the user to provide or paste the Idea Document, Design Document, and PRD.

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
[Insert Designer output from Design Document]

Here are the API Contracts (what your frontend consumes):
[Insert Backend Dev API contracts from Design Document]

Here is the QA Test Plan (you must write unit tests):
[Insert QA test plan]

Here are the PM's acceptance criteria:
[Insert PM acceptance criteria from PRD]

MANDATORY: Before using any library/framework API, use resolve-library-id then query-docs via context7 to verify the exact function signature, parameters, and return type.

## TDD — Write Tests First
For EVERY piece of functionality:
1. Write a failing test FIRST that defines what the component/function should do.
2. Write the minimal code to make the test pass.
3. Refactor while keeping tests green.
Do NOT write implementation code before having a test for it.

## DRY — Don't Repeat Yourself
- Before writing new components or utilities, search the codebase for existing similar code.
- Reuse existing components, hooks, and utilities wherever possible.
- Only extract shared abstractions when you see actual duplication (not speculative).

## No Over-Engineering
- Implement exactly what the Designer's specs and PM's acceptance criteria require. Nothing more.
- No speculative features, no "nice to have" additions, no extra configurability.
- Use the simplest approach that works. If a direct solution exists, use it.
- Don't add error handling for scenarios that can't happen.
- Don't add abstractions for one-time operations.

## Security
- Sanitize any user-facing input rendering to prevent XSS.
- Don't store sensitive data in localStorage or expose it in URLs.
```

#### Backend Dev

Use the Agent tool with `subagent_type: "claude-squad:backend-dev"`.

Prompt the agent:
```
You are the Backend Dev agent. Follow the instructions in the backend-dev agent definition.

Here is the Tech Lead's requirements:
[Insert Tech Lead output from PRD]

Here are the API Contracts you defined (implement them exactly):
[Insert Backend Dev API contracts from Design Document]

Here is the QA Test Plan (you must write unit tests):
[Insert QA test plan]

Here are the PM's acceptance criteria:
[Insert PM acceptance criteria from PRD]

MANDATORY: Before using any library/framework API, use resolve-library-id then query-docs via context7 to verify the exact function signature, parameters, and return type.

## TDD — Write Tests First
For EVERY piece of functionality:
1. Write a failing test FIRST that defines what the endpoint/function should do.
2. Write the minimal code to make the test pass.
3. Refactor while keeping tests green.
Do NOT write implementation code before having a test for it.

## DRY — Don't Repeat Yourself
- Before writing new utilities or helpers, search the codebase for existing similar code.
- Reuse existing middleware, validators, and utilities wherever possible.
- Only extract shared abstractions when you see actual duplication (not speculative).

## No Over-Engineering
- Implement exactly what the API contracts and PM's acceptance criteria require. Nothing more.
- No speculative endpoints, no "future-proofing" abstractions, no extra configurability.
- Use the simplest approach that works. If a direct solution exists, use it.
- Don't add error handling for scenarios that can't happen internally — only validate at system boundaries.
- Don't add unnecessary middleware layers or service abstractions for simple operations.

## Security
- Parameterized queries ONLY — no string concatenation in queries.
- Validate and sanitize ALL user input at system boundaries.
- Authentication and authorization checks on every protected endpoint.
- No hardcoded secrets, API keys, or credentials — use environment variables.
- Rate limiting on public-facing endpoints.
- Follow OWASP Top 10 guidelines.
```

Wait for both Dev agents to complete before proceeding.

### Step 1.3: Tech Lead — Code Review

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent. Follow the instructions in the tech-lead agent definition (Code Review mode).

Review all code changes made by the Frontend Dev and Backend Dev. Check the git diff to see what was changed.

The Designer's specs were:
[Insert Designer output from Design Document]

The API Contracts were:
[Insert Backend Dev API contracts from Design Document]

The PM's acceptance criteria:
[Insert PM acceptance criteria from PRD]

MANDATORY: For any framework/library API usage that looks non-standard or unfamiliar, use resolve-library-id then query-docs via context7 to verify correctness.

Review for correctness, simplicity, security, performance, documentation compliance, and unit test coverage.

## TDD Compliance Check
- Were tests written BEFORE implementation code? Check git history / commit order.
- Does every piece of new logic have a corresponding test?
- Are tests testing behavior (not implementation details)?
- Do tests cover the acceptance criteria from the PRD?

## DRY Compliance Check
- Is there duplicated logic that should be extracted?
- Conversely, are there premature abstractions for code that's only used once?
- Are existing codebase utilities being reused where appropriate?

## Over-Engineering Check
- Is there code that goes beyond what the acceptance criteria require?
- Are there unnecessary abstractions, layers of indirection, or "future-proofing"?
- Are there features, options, or configurations that weren't requested?
- Could any component be simplified without losing functionality?
- Is there error handling for impossible scenarios?

## Security Review (MANDATORY)
- OWASP Top 10 checklist against all new code.
- Input validation at all system boundaries.
- Output encoding to prevent XSS.
- Parameterized queries (no string concatenation in SQL/queries).
- No hardcoded secrets or credentials.
- Auth/authz on all protected routes.
- Principle of least privilege applied.
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

### Methodology Compliance
- TDD: [Tests written first / Issues noted]
- DRY: [No duplication found / Duplication addressed]
- Over-engineering: [None detected / Items simplified]
- Security: [All checks passed / Issues found and fixed]

### Validation
- E2E tests: PASS/FAIL
- Regression: PASS/FAIL
- PM acceptance criteria: [X/Y] validated
- Performance review: [status]
- Security review: [status — OWASP Top 10 checked]
- UAT: APPROVED/REJECTED

### Input Documents Referenced
| Document | Path |
|---|---|
| Idea Document | [path] |
| Design Document | [path] |
| PRD | [path] |

### Documentation Compliance
All implementations verified against current stack documentation via context7:
[List of libraries/frameworks consulted with versions]
```
