# claude-squad

A Claude Code plugin that orchestrates an AI development squad with specialized agent teams for each phase.

## Core Principle

**Every technical decision is verified against the current documentation of the project's stack via context7 MCP.** No assumptions from training data — only what the official docs confirm for the installed versions.

## Commands

| Command | Phase | Agent Team |
|---------|-------|------------|
| `/prd` | Framing | PM, Tech Lead, Designer |
| `/design` | Design & Prototype | Designer, PM, Frontend Dev, Backend Dev |
| `/implement` | Build & Validation | Frontend Dev, Backend Dev, Tech Lead, QA, PM |

### Workflow
```
/prd "build a login page"        → PRD (User Story + Requirements + UX Perspective)
  (review and adjust)
/design                          → Design Document (UX Flows + API Contracts + Feasibility)
  (review and adjust)
/implement                       → Build, Review, Test, Validate
```

## Agents

| Agent | Role | Uses context7 |
|-------|------|:---:|
| **PM** | Defines the problem, User Story, acceptance criteria, UAT validation | — |
| **Tech Lead** | Feasibility, scope, technical direction, code review, perf/security | Yes |
| **Designer** | UX flows, prototypes (pencil MCP), component specs, interaction design | — |
| **Frontend Dev** | Implements UI components, pages, interactions | Yes |
| **Backend Dev** | Implements APIs, data models, business logic | Yes |
| **QA** | Test strategy, test implementation, e2e, regression, acceptance validation | Yes |

## Flow

```
/prd ──────────────────────────────────────────────
  PM          → Brainstorm, challenge, User Story
  Tech Lead   → Feasibility, scope, requirements
  Designer    → Early UX perspective, mockup review
  ↳ Gate: User confirms PRD

/design ───────────────────────────────────────────
  Designer      → UX flows, prototypes (pencil MCP), component specs
  PM            → Validates design against User Story
  Frontend Dev  → Reviews frontend feasibility
  Backend Dev   → Proposes API contracts, reviews backend feasibility
  ↳ Gate: User confirms Design

/implement ────────────────────────────────────────
  Build:
    QA            → Defines test plan
    Frontend Dev  → Implements UI (parallel)
    Backend Dev   → Implements backend (parallel)
    Tech Lead     → Code review
       ↳ CHANGES REQUESTED → Devs → Tech Lead (3x)
    QA            → Runs all tests
       ↳ FAIL → Devs → Tech Lead → QA (3x)

  Validation:
    QA            → E2E + regression
    Tech Lead     → Performance + security review
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
/claude-squad:prd build user authentication with email and password
/claude-squad:design
/claude-squad:implement
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured
- [pencil MCP server](https://pencil.dev) configured (for UI prototyping with .pen files)
