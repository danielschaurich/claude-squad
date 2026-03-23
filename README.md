# claude-squad

A Claude Code plugin that orchestrates an AI development squad across 3 steps with documentation-driven development.

## Core Principle

**Every technical decision is verified against the current documentation of the project's stack via context7 MCP.** No assumptions from training data — only what the official docs confirm for the installed versions.

## Commands

| Command | Step | Agent Team | Use when |
|---------|------|------------|----------|
| `/prd` | Framing | PM, Planner | Define the problem, validate feasibility, produce the PRD |
| `/design` | Design & Prototype | Architect, Test Planner | Design the architecture, prototype UI (pencil MCP), define test strategy |
| `/implement` | Build & Validation | Dev, Reviewer, QA | Implement, review, test, and validate quality |

### Modular workflow
```
/prd "build a login page"        → produces PRD (User Story + Requirements)
  (review and adjust the plan)
/design                          → produces Design Document (Architecture + Test Plan)
  (review and adjust the design)
/implement                       → builds, reviews, tests, validates
```

## Agent Teams

### `/prd` — Framing
| Agent | Role | Uses context7 |
|-------|------|:---:|
| **PM** | Brainstorms, challenges assumptions, defines User Story + acceptance criteria | — |
| **Planner** | Validates feasibility against documented APIs, produces requirements | Yes |

### `/design` — Design & Prototype
| Agent | Role | Uses context7 |
|-------|------|:---:|
| **Architect** | Explores approaches, designs technical solution, prototypes UI via pencil MCP | Yes |
| **Test Planner** | Defines test strategy using documented test framework APIs | Yes |

### `/implement` — Build & Validation
| Agent | Role | Uses context7 |
|-------|------|:---:|
| **Dev** | Implements code with verified API calls + unit tests | Yes |
| **Reviewer** | Reviews correctness, simplicity, security, performance, and doc compliance | Yes |
| **QA** | Implements and runs all tests from the Test Plan, validates acceptance criteria | Yes |

## Workflow

```
/prd ──────────────────────────────────────────────
  PM        → Brainstorm, challenge, User Story
  Planner   → Feasibility against docs, requirements
  ↳ Gate: User confirms PRD

/design ───────────────────────────────────────────
  Architect    → Technical design (doc-validated)
               → UI prototype analysis (pencil MCP)
  Test Planner → Test strategy (doc-validated)
  ↳ Gate: User confirms Design

/implement ────────────────────────────────────────
  Build:
    Dev       → Implements + unit tests
    Reviewer  → Code review + doc compliance
       ↳ CHANGES REQUESTED → Dev → Reviewer (3x)
    QA        → Implements + runs test plan
       ↳ FAIL → Dev → Reviewer → QA (3x)

  Validation:
    QA        → E2E + regression
    Reviewer  → Performance + security validation
    PM        → UAT against acceptance criteria
    ↳ Gate: REJECTED → Dev fixes → re-validate (2x)
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

# Or jump to any step with context
/claude-squad:design "here's what I want to build: [paste PRD]"
/claude-squad:implement "here's the design: [paste Design Document]"
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured
- [pencil MCP server](https://pencil.dev) configured (for UI prototyping with .pen files)
