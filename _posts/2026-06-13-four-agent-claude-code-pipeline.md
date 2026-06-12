---
title: "Turning a Jira Ticket Into Shipped Code With a Four-Agent Claude Code Pipeline"
date: 2026-06-13 12:00:00 +0200
categories: [AI, Engineering]
tags: [claude, claude-code, agents, jira, automation]
---

If you work on a multi-service codebase, you know the shape of the problem. A ticket lands in Jira. You read it, you think about the architecture, you write the code, you review it, and somewhere along the way the reasoning behind your decisions evaporates. Three weeks later someone asks why a feature was built a certain way and the only record is a terse commit message.

We wanted a workflow that kept the thinking and the doing as separate, deliberate steps, one that left behind a paper trail without any extra effort. Here's how we built it inside Claude Code using four custom agents.

## The problem

A single AI coding session tends to collapse everything into one blur. It plans, codes, second-guesses the plan, codes some more, and burns through its context window holding all of it at once. The result is muddled: architectural decisions get made implicitly mid-implementation, nothing pauses for human review at the right moment, and there's no durable record of what happened.

With multiple microservices the problem compounds. Which repo does this change belong in? Which branch? What were the trade-offs we considered and rejected? Without structure, all of that lives only in the chat history until it scrolls away.

We wanted four things:

- Separation of thinking and doing: architecture decisions made deliberately, reviewed by a human, before any code is written.
- A human checkpoint at the right moment: after the plan, before the implementation.
- A consistent QA step that checks the code against the ticket's actual acceptance criteria, not just vibes.
- An automatic record of the problem, the decisions, and the outcome, stored per ticket.

## The fix: a four-agent pipeline

Claude Code lets you define specialized subagents as markdown files in `.claude/agents/`. Each runs in its own context window with its own system prompt, model, and tool permissions. That isolation is the whole trick: the planner can't accidentally write files, the coder can't drift into architecture, and the reviewer can't modify what it's reviewing.

The pipeline looks like this:

```
Jira Ticket
   → architect-planner   (Opus, read-only)      — analyze & plan
   → [your review]
   → git checkout -b feature/XXXX               — branch for the task
   → feature-coder        (Sonnet, full access)  — implement the plan
   → [your review]
   → qa-reviewer          (Opus, read-only)       — check against ticket
   → task-summary         (Sonnet, write)         — record the outcome
```

The branch switch happens after the plan is approved but before any code is written, so the coder commits to the right branch from the very first file. With microservices, the planning step is also where you decide which repo the work belongs in.

Each model is chosen deliberately. Opus handles the reasoning-heavy work (planning and review) where deeper thinking pays off. Sonnet handles the mechanical execution where speed and cost matter more. The two read-only agents physically cannot touch your files, which removes a whole class of accidents.

Below is every agent, plus the CLAUDE.md that ties them together. Drop the agent files into `.claude/agents/` and the CLAUDE.md into your project root.

## The agents

### architect-planner

The entry point. It reads the ticket and the surrounding code, then produces a structured implementation brief. It is strictly read-only, so it cannot start coding before you've approved its plan.

```yaml
---
name: architect-planner
description: Use this agent when given a Jira ticket or feature request to analyze requirements and produce an architecture plan. Use proactively before any coding begins.
model: opus
tools: Read, Glob, Grep
---
You are a senior software architect. When given a task or Jira ticket:
1. Read the relevant existing code to understand current patterns
2. Analyze the requirements and acceptance criteria
3. Propose the architecture: components, data flow, API contracts, edge cases
4. List key decisions and trade-offs
5. Output a structured implementation brief ready to hand off to a coding agent

Output format:
## Summary
## Architectural Decisions
## Implementation Steps (ordered)
## Files to Create/Modify
## What NOT to do
## Open Questions

Be specific. No vague advice. The coder agent will use this as its sole source of truth.
```

### feature-coder

Takes the approved plan and implements it. It has full tool access and follows the brief exactly; its instructions explicitly forbid making fresh architectural decisions, pushing those back to a human instead.

```yaml
---
name: feature-coder
description: Use this agent after architect-planner has produced an implementation brief. Takes the plan and writes the actual code.
model: sonnet
tools: Read, Write, Edit, Bash, Glob, Grep
---
You are a senior software engineer. You receive an architecture brief from the planner agent and implement it exactly.

Rules:
- Follow the implementation steps in order
- Do not make architectural decisions — escalate open questions back to the parent
- Write tests alongside code
- Follow existing code patterns in the repo (read surrounding files first)
- After each file, confirm what was done

Do not deviate from the plan unless you hit a genuine blocker.
```

### qa-reviewer

Triggered only when you explicitly ask. It checks the implementation against the ticket's acceptance criteria and reviews code quality. It can run `git diff`, which makes it equally useful for reviewing a teammate's branch.

```yaml
---
name: qa-reviewer
description: Use this agent only when explicitly asked to do a QA review. Reviews implemented code against the original ticket acceptance criteria and checks code quality.
model: opus
tools: Read, Glob, Grep, Bash
---
You are a senior QA engineer and code reviewer.

When given a branch name and ticket, do the following in order:
1. Run `git diff main...origin/<branch-name>` to see all changes
2. Read the full content of each changed file for context
3. Check acceptance criteria — verify each one is met, flag any missing or incomplete
4. Code review — check for:
   - Consistency with existing patterns in the codebase
   - Readability and naming clarity
   - Obvious bugs or edge cases missed
   - Dead code or unnecessary complexity

Output format:
## Acceptance Criteria Check
| Criterion | Status | Notes |
|-----------|--------|-------|
| ... | ✅ Met / ❌ Missing / ⚠️ Partial | ... |

## Code Review Findings
Group by severity: BLOCKER / MAJOR / NIT
Each finding: file, line, issue, suggested fix

## Summary
Overall verdict: APPROVED / NEEDS CHANGES
List of required changes before this can be merged.

Do not modify any files.
```

### task-summary

The memory of the pipeline. When asked, it pulls the whole conversation (problem, decisions, implementation, QA findings) and writes it to a per-ticket file. If the file already exists, it prepends a dated update instead of overwriting.

```yaml
---
name: task-summary
description: Use this agent only when explicitly asked to save a task summary. Writes a structured summary of the ticket, decisions, and implementation to ~/Projects/tasks/[ticket-number].txt
model: sonnet
tools: Read, Write, Bash
---
You are a technical documentation agent. When asked to save a task summary:

1. Extract the ticket number from the conversation (e.g. XXX-1234)
2. Check if ~/Projects/tasks/[ticket-number].txt already exists:
   `ls ~/Projects/tasks/[ticket-number].txt`
3. If it exists, read its current content
4. Compose the summary with these sections:

## [ticket-number] - [ticket title]
Updated: [current date]

### Problem
What the ticket was about and why it was needed.

### Architectural Decisions
Key decisions made during planning and the reasoning behind them.

### What Was Done
Files created or modified, with a one-line description of each change.

### QA Findings
Any blockers, majors, or nits found during review and how they were resolved.

### Open Questions
Anything unresolved or flagged for follow-up.

5. If the file already exists, prepend the new summary above the existing content with a separator:
   `=== UPDATE [current date] ===`
6. If the file does not exist, create it with the summary
7. Ensure the directory exists first:
   `mkdir -p ~/Projects/tasks`
8. Confirm the file path where it was saved

Do not ask for clarification. Extract everything from the conversation context.
```

## The glue: CLAUDE.md

The agent files define what each agent does. CLAUDE.md defines when they run. Placed in the project root, it's loaded automatically into every session, so the workflow becomes the default behavior without anyone having to remember it.

```markdown
## Development Workflow

When given a Jira ticket, always follow this pipeline:

1. Use the **architect-planner** agent to analyze the ticket and produce an implementation plan
2. Present the plan to the user for review
3. Only after explicit approval, use the **feature-coder** agent to implement it
4. Use the **qa-reviewer** agent only when explicitly asked
5. Use the **task-summary** agent only when explicitly asked

Never start coding without an approved plan.
Never trigger qa-reviewer or task-summary automatically.

## Agents

- `architect-planner` — Opus, read-only, produces architecture briefs
- `feature-coder` — Sonnet, full tools, implements approved plans
- `qa-reviewer` — Opus, read-only, reviews code against ticket acceptance criteria and code quality
- `task-summary` — Sonnet, write access, saves structured summary to ~/Projects/tasks/[ticket].txt, prepends if file exists
```

## How it feels in practice

You launch Claude Code from the project root and paste a ticket. The planner reads the code and hands you a brief. You read it, push back on anything that looks off, and approve. You cut a branch. The coder implements against the plan, file by file. When it's done you ask for a QA review, get a table mapping each acceptance criterion to its status, fix any blockers, and then ask for a summary. A per-ticket file appears with the whole story: problem, decisions, what changed, and what's still open.

No single agent makes this work. The structure does: thinking and doing stay separate, a human reviews at the one moment that matters most, and the reasoning behind every decision survives long after the chat window closes.

## Tips

- Always launch from the project root. Agents load from `.claude/agents/` relative to where you start the session.
- Commit the `.claude/` folder to git so the whole team inherits the same pipeline.
- Lock down tools per agent. The read-only agents physically can't damage your repo, which is a feature, not a limitation.
- Restart between tickets. Clear context between tasks to avoid one ticket's history bleeding into the next.
- Keep one session per ticket so the task-summary agent has the full story to draw from.
