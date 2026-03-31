# claude-squad

A Claude Code plugin that orchestrates an AI development squad with specialized agent teams across 4 phases.

## Core Principles

- **Documentation-Driven Development** — Every technical decision is verified against the current documentation of the project's stack via context7 MCP. No assumptions from training data.
- **TDD** — Tests are written first, then code to make them pass.
- **DRY** — No duplication. Reuse existing code before writing new.
- **No Over-Engineering** — Build exactly what's needed, nothing more.
- **Security Built In** — OWASP Top 10 awareness, input validation, parameterized queries, no hardcoded secrets.

## Commands

| Command | Phase | Agent Team |
|---------|-------|------------|
| `/idea` | Ideation | PM, Tech Lead, Designer |
| `/design` | Design & Prototype | Designer, PM, Frontend Dev, Backend Dev |
| `/prd` | Product Requirements | PM, Tech Lead, Designer, QA |
| `/implement` | Build & Validation | Frontend Dev, Backend Dev, Tech Lead, QA, PM |

### Workflow
```
/idea "smart notifications"       → Idea Document (brainstorm, explore, converge)
  (interactive Q&A with the team)
/design                           → Design Document (UX Flows + API Contracts + Feasibility)
  (review and adjust)
/prd                              → PRD (User Stories + Requirements + Test Strategy)
  (review and adjust)
/implement                        → Build, Review, Test, Validate (TDD + DRY + Security)
```

## Agents

| Agent | Role | Uses context7 |
|-------|------|:---:|
| **PM** | Defines the problem, brainstorms ideas, User Story, acceptance criteria, UAT validation | — |
| **Tech Lead** | Feasibility, scope, technical direction, code review, perf/security | Yes |
| **Designer** | UX flows, prototypes (pencil MCP), component specs, interaction design | — |
| **Frontend Dev** | Implements UI components, pages, interactions | Yes |
| **Backend Dev** | Implements APIs, data models, business logic | Yes |
| **QA** | Test strategy, test implementation, e2e, regression, acceptance validation | Yes |

## Flow

```
/idea ─────────────────────────────────────────────
  PM          → Reframe problem, challenge premise, wild ideas
  Tech Lead   → Technical creativity, stack opportunities, quick wins
  Designer    → User empathy, unconventional interactions, simplicity
  ↳ Interactive Q&A with user at every step
  ↳ Gate: User confirms direction

/design ───────────────────────────────────────────
  Designer      → UX flows, prototypes (pencil MCP), component specs
  PM            → Validates design against chosen direction
  Frontend Dev  → Reviews frontend feasibility
  Backend Dev   → Proposes API contracts, reviews backend feasibility
  ↳ Gate: User confirms Design

/prd ──────────────────────────────────────────────
  PM          → Formalizes User Stories, acceptance criteria
  Tech Lead   → Validates feasibility, scopes subtasks
  Designer    → Confirms design-requirements alignment
  QA          → Defines test strategy, validates testability
  ↳ Gate: User confirms PRD

/implement ────────────────────────────────────────
  Build (TDD + DRY + No Over-Engineering):
    QA            → Defines test plan
    Frontend Dev  → TDD: tests first, then UI implementation
    Backend Dev   → TDD: tests first, then backend implementation
    Tech Lead     → Code review (TDD/DRY/security/quality)
       ↳ CHANGES REQUESTED → Devs → Tech Lead (3x)
    QA            → Runs all tests
       ↳ FAIL → Devs → Tech Lead → QA (3x)

  Validation:
    QA            → E2E + regression
    Tech Lead     → Performance + security review (OWASP Top 10)
    PM            → UAT against acceptance criteria
    ↳ Gate: REJECTED → Devs fix → re-validate (2x)
```

## Install

```
/plugin marketplace add danielschaurich/claude-squad
/plugin install claude-squad@danielschaurich-plugins
```

## Usage

```bash
# Step by step
/claude-squad:idea build smart notifications for our app
/claude-squad:design
/claude-squad:prd
/claude-squad:implement
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured
- [pencil MCP server](https://pencil.dev) configured (for UI prototyping with .pen files)
