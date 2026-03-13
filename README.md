# claude-squad

A Claude Code plugin that orchestrates an AI development squad across 5 phases with documentation-driven development.

## Core Principle

**Every technical decision is verified against the current documentation of the project's stack via context7 MCP.** No assumptions from training data — only what the official docs confirm for the installed versions.

## Agents

| Agent | Role | Uses context7 |
|-------|------|:---:|
| **PM** | Translates user pain into User Stories + acceptance criteria, validates via UAT | — |
| **Planner** | Validates feasibility against documented APIs, produces requirements | Yes |
| **Architect** | Designs technical solution backed by doc references | Yes |
| **Test Planner** | Defines test strategy using documented test framework APIs | Yes |
| **Dev** | Implements code with verified API calls + unit tests | Yes |
| **Reviewer** | Reviews correctness, simplicity, security, and doc compliance | Yes |
| **QA** | Implements and runs all tests from the Test Plan | Yes |
| **Release** | Prepares deploy checklist, rollback plan, CI/CD validation | Yes |

## 5-Phase Flow

```
/claude-squad:squad "build a login page"

Phase 1: Discovery
  PM        → User Story + acceptance criteria
  Planner   → Feasibility check against docs, requirements doc
  ↳ Gate: User confirms requirements

Phase 2: Blueprint
  Architect    → Technical design (doc-validated)
  Test Planner → Test strategy (doc-validated)
  ↳ Gate: User confirms architecture + test plan

Phase 3: Build
  Dev       → Implements code + unit tests (APIs verified via docs)
  Reviewer  → Code review + simplicity checklist + doc compliance
     ↳ CHANGES REQUESTED → Dev fixes → Reviewer (max 3x)
  QA        → Implements + runs test plan
     ↳ FAIL → Dev fixes → Reviewer → QA (max 3x)

Phase 4: Quality
  QA        → E2E + regression tests
  Reviewer  → Performance + security validation (doc-based)
  PM        → UAT against original acceptance criteria
     ↳ REJECTED → Dev fixes → re-validate (max 2x)

Phase 5: Release
  Release   → Deploy checklist, rollback plan, CI/CD validation
  Summary   → Full report across all phases
```

## Install

```
/plugin marketplace add danielschaurich/claude-squad
/plugin install claude-squad@danielschaurich-plugins
```

## Usage

```
/claude-squad:squad build user authentication with email and password
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured
