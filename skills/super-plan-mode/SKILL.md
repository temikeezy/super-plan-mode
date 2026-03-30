---
name: super-plan-mode
description: This skill should be used when the user asks to "plan this before coding", "don't code yet just plan", "show me the plan first", "think through the approach", "design this first", "before you do anything show me a plan", "enter plan mode", "super plan mode", "plan mode on", "let's plan", "what's your plan", "run me through your plan", "hold on, plan first", "I want to see the plan before you start", or expresses any intent to review and approve an implementation approach before execution begins.
license: MIT
metadata:
  author: temikeezy
  version: "1.0.0"
  organization: openclaw
  date: March 2026
  abstract: A universal agent skill for structured plan mode. Built natively for OpenClaw and Claude Code. Generates a rich implementation plan — with affected files, confidence-scored steps, risk matrix, effort estimate, and alternative approaches — then halts execution at a numbered acceptance gate until the user explicitly approves. Also compatible with Cursor Agent, Windsurf, Codex CLI, Aider, and any AI coding agent.
---

# Super Plan Mode Skill

This skill activates Super Plan Mode: a structured planning workflow that generates a rich implementation plan and halts for explicit user approval before any code changes are made.

## Behavior When Active

When this skill is active:

1. DO NOT create, edit, or delete any files until the user has explicitly approved the plan.
2. Announce plan mode immediately: "Entering super plan mode. I will generate a plan and wait for your approval before making any changes."
3. Run the full planning workflow described below.
4. Present the numbered acceptance gate and wait.

## Compatibility

This skill works in any agent environment — Claude Code, Codex CLI, Cursor Agent, Windsurf, Aider, or any text-based AI coding assistant. See `references/agent-compat.md` for environment-specific notes.

## Planning Workflow

### Step 1: Clarify (if needed)

If the request is ambiguous, ask 1–3 focused questions before exploring. If the request is clear, proceed directly to exploration.

Do not ask more than 3 questions in a single message. Start with the most important.

### Step 2: Explore the Codebase (Read-Only)

**If subagents are supported (OpenClaw, Claude Code, Cursor 2.4+, Windsurf Wave 13+, Codex CLI):**
Launch 2–3 `plan-researcher` agents in parallel, each with a distinct focus:
- Agent A: architecture mapping and existing patterns
- Agent B: affected files and dependency tracing
- Agent C: risks, uncommitted changes, and fragile areas

After agents return, read all files they identify as critical.

**If no subagent support (Aider or unknown environments):**
Perform exploration inline sequentially following the same methodology:
1. Map architecture and patterns (Glob, Grep, Read)
2. Trace affected files and dependencies
3. Assess risks — run `git status` and `git log --oneline -10`

In both cases: read critical files, understand existing patterns, and identify risks before generating the plan.

### Step 3: Generate the Plan

Produce the plan document following the format in `references/plan-template.md` exactly.

**Before generating, check for `.super-plan-mode.json`** in the project root. If present, read it and apply its defaults (planSaveDir, autoPhaseCheckpoints, etc.).

Save the plan to: `[planSaveDir]/super-plan-mode-[unix-timestamp].md`
Default save location: `.claude/plans/super-plan-mode-[timestamp].md`

Inform the user where the plan was saved.

### Step 4: Present the Acceptance Gate

After presenting the plan, output the numbered menu exactly as follows:

```
────────────────────────────────────────────
  Plan ready. What would you like to do?

  1  Accept & Build
  2  Reject / Cancel
  3  Modify plan
  4  Accept Phase 1 only  ← (shown only for L/XL estimates)
────────────────────────────────────────────
  Enter number (or type the action):
```

**Wait for the user's response. Take no implementation action until they respond.**

Response handling:
- **1 / "accept" / "build" / "yes"** → proceed to implementation
- **2 / "reject" / "cancel" / "no"** → acknowledge, summarize useful research findings, stop
- **3 / "modify"** → ask what to change, revise the plan, show a diff of what changed, re-present gate
- **4 / "phase 1"** → implement only Phase 1 steps, then pause and re-present gate for remaining phases

### Step 5: Implement (Only After Acceptance)

Execute plan steps in order. For L/XL estimates with phases: after each phase completes, pause and report:
- What was completed
- What comes next
- Any unexpected findings that may affect the remaining plan

Ask the user to confirm before starting the next phase.

On full completion, summarize: files created/modified, steps completed, deviations from plan (if any), suggested next steps.

## Additional Resources

- **`references/plan-template.md`** — canonical plan output format with all required sections
- **`references/agent-compat.md`** — per-agent environment notes for exploration, gate behavior, model flags, and plan file handling
