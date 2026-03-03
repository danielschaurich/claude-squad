# claude-squad

A Claude Code plugin that orchestrates an AI development squad for full-stack feature development.

## Agents

| Agent | Role |
|-------|------|
| **Planner** | Reads docs (context7), breaks tasks into subtasks with acceptance criteria |
| **Architect** | Defines file structure, types, patterns, and implementation order |
| **Dev** | Implements code following the architect's plan |
| **Reviewer** | Reviews for correctness, security, and **simplicity** |
| **QA** | Writes and runs tests, validates acceptance criteria |

## Flow

```
/claude-squad:squad "build a login page"

  Planner  → researches docs, creates task breakdown
  Architect → defines structure, types, patterns
  Dev       → implements the code
  Reviewer  → reviews (with simplicity checklist)
     ↳ CHANGES REQUESTED → Dev fixes → Reviewer again (max 3x)
  QA        → writes & runs tests
     ↳ FAIL → Dev fixes → Reviewer → QA again (max 3x)
  Summary   → reports what was built
```

## Install

**Step 1:** Add the marketplace:
```
/plugin marketplace add danielschaurich/claude-squad
```

**Step 2:** Install the plugin:
```
/plugin install claude-squad@danielschaurich-plugins
```

## Usage

Once installed, run in any project:

```
/claude-squad:squad build user authentication with email and password
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [context7 MCP server](https://github.com/upstash/context7) configured (for the Planner to fetch docs)
