---
name: squad
description: Run a full development squad workflow (Planner → Architect → Dev → Reviewer → QA) to implement a feature or fix.
---

# /squad — Development Squad Orchestrator

Run a full development squad workflow to implement a feature or fix.

## Flow

Execute these steps in order. Each step uses a specialized agent. Pass the output of each step as context to the next.

### Step 1: Planning

Use the Agent tool with `subagent_type: "general-purpose"` and the planner agent prompt.

Prompt the agent:
```
You are the Planner agent. Follow the instructions in the planner agent definition.

Task from user: {{input}}

Research the relevant documentation using context7 MCP tools (resolve-library-id, then query-docs). Explore the existing codebase. Then create a structured task breakdown with acceptance criteria.
```

Wait for the planner output before proceeding.

### Step 2: Architecture

Use the Agent tool with `subagent_type: "general-purpose"` and the architect agent prompt.

Prompt the agent:
```
You are the Architect agent. Follow the instructions in the architect agent definition.

Here is the Planner's output:
[Insert planner output]

Read the existing codebase, then define the technical architecture: file structure, types, patterns, and implementation order.
```

Wait for the architect output before proceeding.

### Step 3: Implementation

Use the Agent tool with `subagent_type: "general-purpose"` and the dev agent prompt.

If the architect defined independent subtasks, you MAY run multiple dev agents in parallel (one per feature/subtask). Otherwise, use a single dev agent.

Prompt the agent:
```
You are the Dev agent. Follow the instructions in the dev agent definition.

Here is the Architect's plan:
[Insert architect output]

Implement the code following this plan exactly. Write clean, simple code.
```

Wait for all dev agents to complete before proceeding.

### Step 4: Code Review

Use the Agent tool with `subagent_type: "general-purpose"` and the reviewer agent prompt.

Prompt the agent:
```
You are the Reviewer agent. Follow the instructions in the reviewer agent definition.

Review all code changes made by the Dev agent(s). Check the git diff to see what was changed. Focus especially on simplicity — challenge any unnecessary complexity.

The original plan was:
[Insert architect output]
```

**If the reviewer returns CHANGES REQUESTED:**
- Run a Dev agent to fix the issues.
- Run the Reviewer again on the fixes.
- Repeat until APPROVED (max 3 cycles, then ask the user).

### Step 5: QA Testing

Use the Agent tool with `subagent_type: "general-purpose"` and the qa agent prompt.

Prompt the agent:
```
You are the QA agent. Follow the instructions in the qa agent definition.

The acceptance criteria from the Planner:
[Insert planner output]

Write tests and run them. Verify the implementation meets all acceptance criteria. Also run any existing tests to check for regressions.
```

**If QA returns FAIL:**
- Run a Dev agent to fix the failures.
- Run the Reviewer on the fixes.
- Run QA again.
- Repeat until PASS (max 3 cycles, then ask the user).

### Step 6: Summary

After all steps pass, output a summary to the user:

```
## Squad Complete

### What was built
[Brief description]

### Files changed
[List of files]

### Tests
[Test results summary]

### Review status
[Approved / any notes]
```
