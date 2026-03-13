# claude-squad

A Claude Code plugin that orchestrates an AI development squad across 5 phases with documentation-driven development.

## Core Principle

**Every technical decision is verified against the current documentation of the project's stack via context7 MCP.** No assumptions from training data — only what the official docs confirm for the installed versions.

## Commands

| Command | Phases | Use when |
|---------|--------|----------|
| `/discovery` | 1-2 (Discovery + Blueprint) | You want to plan first, review, then build separately |
| `/implement` | 3-4 (Build + Quality) | You have a Discovery & Blueprint Document ready |
| `/squad` | 1-5 (All phases) | You want the full end-to-end workflow |

### Modular workflow
```
/discovery "build a login page"    → produces Discovery & Blueprint Document
  (review and adjust the plan)
/implement                         → builds, reviews, tests, validates
```

### Full workflow
```
/squad "build a login page"        → runs everything including Release preparation
```

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
Phase 1: Discovery ─────────────────────────────┐
  PM        → User Story + acceptance criteria   │ /discovery
  Planner   → Feasibility against docs           │
                                                  │
Phase 2: Blueprint                                │
  Architect    → Technical design (doc-validated) │
  Test Planner → Test strategy (doc-validated)    │
  ↳ Gate: User confirms plan                     ┘

Phase 3: Build ──────────────────────────────────┐
  Dev       → Implements + unit tests             │ /implement
  Reviewer  → Code review + doc compliance        │
     ↳ CHANGES REQUESTED → Dev → Reviewer (3x)   │
  QA        → Implements + runs test plan         │
     ↳ FAIL → Dev → Reviewer → QA (3x)           │
                                                  │
Phase 4: Quality                                  │
  QA        → E2E + regression                    │
  Reviewer  → Performance + security validation   │
  PM        → UAT against acceptance criteria     │
  ↳ Gate: REJECTED → Dev fixes → re-validate (2x)┘

Phase 5: Release ────────────────────────────────── /squad only
  Release   → Deploy checklist, rollback plan
  Summary   → Full report across all phases
```

## Install

```
/plugin marketplace add danielschaurich/claude-squad
/plugin install claude-squad@danielschaurich-plugins
```

## Usage

```bash
# Plan first, build later
/claude-squad:discovery build user authentication with email and password
/claude-squad:implement

# Or run everything at once
/claude-squad:squad build user authentication with email and password
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured
