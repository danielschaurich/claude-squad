---
name: idea
description: Ideation phase — PM, Tech Lead, and Designer brainstorm creatively, think out of the box, challenge assumptions, and explore unconventional solutions through interactive Q&A with the user. Output feeds into /design.
---

# /idea — Ideation

Run the Ideation phase: brainstorm freely, think out of the box, challenge the problem from every angle, and explore unconventional solutions. **This is a creative, divergent-thinking session — not a requirements exercise.**

The output of this command is an **Idea Document** — a collection of explored concepts, validated directions, and open questions that feeds directly into `/design`.

## Core Principle: Think Out of the Box

This phase is about **divergent thinking**. Agents must actively push beyond the obvious. No idea is too wild to mention — it can always be scoped down later. The goal is to explore the problem space broadly before narrowing down.

---

## Agent Team

| Agent | Role in this phase |
|-------|-------------------|
| **PM** | Facilitates ideation, challenges the problem definition, explores user pain from multiple angles, proposes alternative framings |
| **Tech Lead** | Brings technical creativity — what's possible that the user hasn't considered? What emerging patterns or existing capabilities could unlock simpler or more powerful solutions? |
| **Designer** | Brings user empathy and UX creativity — how could this feel? What unconventional interactions could delight users? What problems could be solved through design alone? |

---

## How This Phase Works

This is an **interactive conversation**, not a one-shot output. The agents brainstorm, then ask the user questions. The user answers, and agents build on those answers with new ideas. This loop repeats until the idea is solid.

**The user is an active participant, not a passive reviewer.**

---

## Step 1: PM — Problem Exploration & Creative Framing

Use the Agent tool with `subagent_type: "claude-squad:pm"`.

Prompt the agent:
```
You are the PM agent in IDEATION mode. This is NOT the PRD phase — do NOT write User Stories or acceptance criteria yet. Your job is to EXPLORE, QUESTION, and BRAINSTORM.

User's initial idea: {{input}}

Your task:
1. **Reframe the problem** — Don't take the idea at face value. What's the REAL problem behind it? Restate it in at least 2 different ways.
2. **Challenge the premise** — Why does this problem exist? Is it actually a problem? Who says so? What if we didn't solve it at all?
3. **Explore the audience** — Who else has this problem? Are there adjacent user groups we're not thinking about? What are their different contexts?
4. **Think wild** — Propose at least 3 ideas, including at least one that feels unconventional or "too creative." Push beyond the obvious solution.
   - What would the 10x version look like?
   - What would the simplest possible version look like?
   - What would a completely different approach look like?
5. **Find inspiration** — Are there analogies from other domains? How do other products/industries solve similar problems in unexpected ways?
6. **Surface assumptions** — What is everyone assuming that might not be true?

OUTPUT FORMAT:
## PM Ideation: [Brief title]

### The Problem (Reframed)
[At least 2 different framings of the problem]

### Challenging the Premise
[Why does this exist? What if we don't solve it? Who says this matters?]

### Who's Affected?
[Different user groups, contexts, and needs — think broadly]

### Wild Ideas
1. **[Idea name]** — [Description. Why it's interesting. What it unlocks.]
2. **[Idea name]** — [Description. Why it's interesting. What it unlocks.]
3. **[Unconventional idea]** — [Description. This one should push boundaries.]

### Inspiration from Other Domains
[Analogies, patterns from other products/industries]

### Assumptions We're Making
- [Assumption 1 — is it true?]
- [Assumption 2 — what if it's wrong?]

### Questions for the User (REQUIRED — at least 5)
These should be thought-provoking, not just clarifying:
1. [Question that challenges what the user thinks they want]
2. [Question about the real pain — what happens today without this?]
3. [Question about scope — what would make this 10x more useful?]
4. [Question about constraints — what CAN'T change?]
5. [Question about success — how would you know this worked?]
```

**INTERACTION LOOP (MANDATORY):**
1. Present the PM's brainstorm to the user.
2. **Ask the user to engage with the ideas and answer the questions.** Do NOT proceed until the user responds.
3. After the user responds, proceed to Step 2 with the user's input.

---

## Step 2: Tech Lead — Technical Creativity & Feasibility Spark

Use the Agent tool with `subagent_type: "claude-squad:tech-lead"`.

Prompt the agent:
```
You are the Tech Lead agent in IDEATION mode. This is NOT the requirements phase — do NOT write subtasks or architecture docs yet. Your job is to bring TECHNICAL CREATIVITY.

Here is the PM's brainstorm:
[Insert PM output]

Here are the user's responses to the PM's questions:
[Insert user's answers]

Your task:
1. **Technical possibilities** — What can the current stack do that nobody's considered? Are there capabilities hiding in the existing codebase or dependencies?
2. **Read dependency files** (package.json, etc.) and think about what's already available.
3. **Creative technical approaches** — For each of the PM's ideas, what's the most elegant technical path? Can any be combined?
4. **What's deceptively simple?** — Which ideas look hard but are actually easy with the right approach?
5. **What's deceptively complex?** — Which ideas look easy but hide dragons?
6. **New ideas from tech** — Based on what the stack can do, propose at least 1 new idea the PM didn't think of.
7. **Quick wins** — Is there a version of this that could be built in hours, not days? What would it prove?

OUTPUT FORMAT:
## Tech Lead Ideation: [Brief title]

### Stack Opportunities
[What the current tech stack enables that might not be obvious]

### Idea Reactions
| PM Idea | Technical Take | Complexity | Surprise Factor |
|---------|---------------|------------|-----------------|
| [Idea 1] | [Easy/hard? Elegant path?] | [Low/Med/High] | [What's unexpected about implementation] |
| [Idea 2] | ... | ... | ... |

### New Ideas from Tech
1. **[Tech-inspired idea]** — [What the stack makes possible that nobody asked for]

### Deceptively Simple vs Complex
- **Looks hard, actually easy:** [What and why]
- **Looks easy, actually hard:** [What and why — the dragons]

### Quick Win Proposal
[A minimal version that proves the concept fast]

### Questions for the User (REQUIRED — at least 3)
1. [Question about technical preferences or constraints]
2. [Question about what to prioritize — speed vs. polish vs. scope]
3. [Question about existing patterns or tools they already use]
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Tech Lead's technical perspective to the user.
2. **Ask the user to react and answer the questions.** Do NOT proceed until the user responds.
3. After the user responds, proceed to Step 3 with the cumulative context.

---

## Step 3: Designer — UX Creativity & User Empathy

Use the Agent tool with `subagent_type: "claude-squad:designer"`.

Prompt the agent:
```
You are the Designer agent in IDEATION mode. This is NOT the full design phase — do NOT produce component specs or detailed flows yet. Your job is to bring UX CREATIVITY and USER EMPATHY.

Here is the PM's brainstorm:
[Insert PM output]

Here is the Tech Lead's technical perspective:
[Insert Tech Lead output]

Here are the user's responses so far:
[Insert all user answers from previous rounds]

{{IF .pen files found}}
The project has UI mockups in these .pen files: [list paths]. Use the pencil MCP tools (batch_get) to review them for context. Never use Read or Grep on .pen files.
{{END IF}}

Your task:
1. **User journey imagination** — For the top ideas, how would the user EXPERIENCE this? Walk through it as a story, not a spec.
2. **Emotional design** — How should this FEEL? What emotions should the user have at each step?
3. **Unconventional interactions** — What if we broke the usual patterns? What if there were no forms? No buttons? What if it was voice, gesture, contextual, predictive?
4. **Simplicity challenge** — Can the UI be simpler than anyone expects? What if it's one screen? Zero screens? Invisible?
5. **Delight opportunities** — Where can we surprise the user positively? Micro-interactions, smart defaults, anticipatory design?
6. **Accessibility as innovation** — How can making this accessible actually make it BETTER for everyone?
7. **Anti-patterns to avoid** — What would make this annoying, confusing, or forgettable?

OUTPUT FORMAT:
## Designer Ideation: [Brief title]

### User Journey Stories
For top 2-3 ideas, tell the story:
**[Idea name]:**
> [User name] opens [app]. They see [what]. They feel [emotion]. They [action]. The system [response]. They think "[thought]." ...

### How It Should Feel
[Emotional design goals — what feelings at what moments]

### Unconventional Interaction Ideas
1. **[Interaction idea]** — [Why it's better than the obvious approach]
2. **[Interaction idea]** — [What it unlocks for the user]

### The Simplest Version
[What if the UI were radically simpler than expected? Describe it.]

### Delight Opportunities
- [Moment 1] → [Surprise/delight idea]
- [Moment 2] → [Surprise/delight idea]

### What to Avoid
- [Anti-pattern 1 — why it would ruin the experience]
- [Anti-pattern 2]

### Questions for the User (REQUIRED — at least 3)
1. [Question about how users currently feel about this problem]
2. [Question about aesthetic/tone preferences]
3. [Question about what "delightful" means to them in this context]
```

**INTERACTION LOOP (MANDATORY):**
1. Present the Designer's creative perspective to the user.
2. **Ask the user to react and answer the questions.** Do NOT proceed until the user responds.

---

## Step 4: Convergence — Synthesize the Best Direction

After all three agents have brainstormed and the user has engaged with each round, synthesize the session.

Present to the user:

```
## Ideation Summary

### The Direction Emerging
[Based on all 3 agents' ideas and the user's responses, what direction is forming?]

### Top Concepts (ranked by team enthusiasm + user interest)
1. **[Concept name]** — [One-line description]
   - PM perspective: [Why it solves the real problem]
   - Tech perspective: [Why it's feasible/elegant]
   - Design perspective: [Why users will love it]

2. **[Concept name]** — [One-line description]
   - PM: [...]
   - Tech: [...]
   - Design: [...]

3. **[Concept name]** — [One-line description]
   - PM: [...]
   - Tech: [...]
   - Design: [...]

### What We Learned
- [Key insight from the brainstorm]
- [Assumption that was challenged]
- [Constraint that was surfaced]

### Open Questions Remaining
- [Anything still unresolved]
```

**Ask the user:**
> Which direction resonates most? Want to adjust, combine, or explore further before we move to Design?

---

## Gate: User Confirms Direction

**The ideation is only complete when:**
- The user has engaged with questions from all 3 agents
- A clear direction has emerged
- The user explicitly confirms they're ready to move on

---

## Output: Idea Document

After the user confirms, compile the Idea Document and **save it to a file**.

### Save the Idea Document

1. Ensure the `docs/squad/` directory exists in the project root (create it if needed).
2. Generate the filename using today's date and a kebab-case title: `docs/squad/yyyy-MM-dd-idea-<concept-title>.md` (e.g., `docs/squad/2026-03-31-idea-smart-notifications.md`).
3. Write the full Idea Document content below to that file.

```
## Idea Document: [Concept title]

### 1. Problem Space (PM)
[PM's problem exploration, reframings, audience analysis]

### 2. Ideas Explored
[All ideas from all agents — the full creative output]

### 3. Technical Landscape (Tech Lead)
[Stack opportunities, complexity assessment, quick win proposal]

### 4. UX Vision (Designer)
[User journey stories, emotional design, interaction ideas, simplicity challenge]

### 5. Chosen Direction
[The concept(s) the user chose, with rationale from each perspective]

### 6. Key Insights & Assumptions
- [What we learned]
- [Assumptions to validate in Design phase]

### 7. User Decisions Log
[All Q&A exchanges — questions asked and user's answers, preserving the conversation]

### Status
- Ideation: Complete
- Direction: Confirmed by user
- Ready for: /design
```

### Next Steps

After saving, tell the user the file path and ask:

> **Idea Document saved to `docs/squad/yyyy-MM-dd-idea-<concept-title>.md`.**
>
> What would you like to do next?
> 1. **Explore more** — keep brainstorming, dig deeper into a specific angle.
> 2. **Move to Design** — clear the context and start a new conversation with: `/design docs/squad/yyyy-MM-dd-idea-<concept-title>.md`

Replace the placeholder path with the actual file path used.
