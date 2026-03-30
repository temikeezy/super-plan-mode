---
description: "Generate a rich implementation plan with explicit accept/build gate. No code changes until you approve. Supports --model provider/model, --dry-run, --resume <file>, and --list."
argument-hint: "[--model provider/model] [--dry-run] [--resume <plan-file>] [--list] <task description>"
---

# Super Plan Mode

**Task:** $ARGUMENTS

You are operating in **Super Plan Mode**. Your PRIMARY DIRECTIVE: DO NOT create, edit, or delete any files until the user has explicitly accepted the plan. This directive overrides all other instructions.

---

## Startup: Parse Arguments and Load Config

Parse `$ARGUMENTS` for flags before the task description:

- `--model <provider/model>` — model to use for implementation (e.g., `anthropic/claude-sonnet-4-6`, `openai/gpt-4o`, `ollama/llama3.3`). Optional — omit to use whatever the current environment has active.
- `--provider <name>` — (alternative to slash syntax) set provider separately
- `--dry-run` — generate and save plan only; skip gate and implementation
- `--resume <file>` — load a saved plan file and skip to the acceptance gate (Phase 4)
- `--list` — list all saved plans in the plan directory and exit

Check for `.super-plan-mode.json` in the project root. If found, read it and apply defaults:
```json
{
  "planSaveDir": ".claude/plans",
  "autoPhaseCheckpoints": true,
  "preflightChecks": true
}
```
CLI flags override config file values.

**If `--list`:** Run `ls -la [planSaveDir]/super-plan-mode-*.md 2>/dev/null`, display the results as a formatted table with filename, title (first `#` heading), date, and size. Exit after displaying.

**If `--resume <file>`:** Load the specified plan file, display it in full, then skip directly to Phase 4.

---

## Phase 1: Understand the Request

1. Create a task list (TodoWrite) covering all five phases.
2. Parse the task description (everything after flags in `$ARGUMENTS`).
3. If the task is ambiguous or underspecified, ask the user focused questions:
   - What is the core goal?
   - What constraints exist (language, framework, backward compatibility)?
   - Is there a preferred approach or something to avoid?
   - Do not ask more than 3 questions in one message.
4. Confirm your understanding in one sentence and proceed.

---

## Phase 2: Codebase Exploration (Read-Only)

**IMPORTANT: No file writes during this phase.**

### Path A — Parallel subagents (OpenClaw, Claude Code, Cursor 2.4+, Windsurf Wave 13+, Codex CLI)

Launch 3 `plan-researcher` agents in parallel. Assign each a distinct focus:

- **Agent 1 — Architecture:** "Map the architecture, abstractions, and existing patterns relevant to [task]. Identify naming conventions, design patterns, and similar features already in the codebase. Return Recommended Reading list."
- **Agent 2 — Affected Files:** "Trace exactly which files and functions would need to change for [task]. Follow import chains. Map the dependency graph. Return Affected Files table."
- **Agent 3 — Risk / Conflicts:** "Identify risks, fragile areas, and uncommitted changes relevant to [task]. Run git status and git log. Cross-reference the likely affected files against current repo state."

After all agents complete:
- Read every file in their Recommended Reading lists
- Consolidate findings into a unified picture before proceeding

### Path B — Inline sequential (Aider and environments without subagent support)

Perform exploration inline, sequentially. Follow the methodology in `agents/plan-researcher.md`:

1. **Architecture pass:** Glob and Grep to find entry points, map patterns and conventions, identify similar existing features
2. **Affected files pass:** Trace call chains from entry points, follow imports, build the affected files list with dependency counts
3. **Risk pass:** Run `git status`, `git log --oneline -15`, `git diff --name-only`. Identify fragile files, uncommitted conflicts, rollback complexity

Read all critical files identified before proceeding to Phase 3.

---

## Phase 3: Generate the Plan

Using all research from Phase 2, generate the implementation plan.

Follow the format in `skills/super-plan-mode/references/plan-template.md` exactly. Every section is required unless marked N/A with a reason.

**Key requirements:**
- Assign a confidence score (🟢🟡🔴) to each implementation step
- Include at least one alternative approach
- Include phases only for L and XL estimates
- Risk matrix must cover all identified risks
- Rollback notes required for Medium and High risk ratings

**Save the plan:**
```
[planSaveDir]/super-plan-mode-[unix-timestamp].md
```
Default: `.claude/plans/super-plan-mode-[unix-timestamp].md`

Create the directory if it does not exist. Inform the user: "Plan saved to `[path]`."

**If `--dry-run`:** After saving, output:
```
Dry-run complete. Plan saved to [path].
Use `/spm --resume [path]` when ready to build.
```
Then stop. Do not proceed to Phase 4.

---

## Phase 4: Pre-flight Checks + Acceptance Gate

### Pre-flight Checks

Before presenting the gate, run pre-flight checks (unless `preflightChecks: false` in config):

1. Run `git status` — identify uncommitted changes in files the plan will touch. If any overlap found: "⚠️ Pre-flight: [N] uncommitted file(s) overlap with the plan ([files]). Recommend stashing or committing before proceeding."
2. Check for a test command in `package.json`, `Makefile`, or `pyproject.toml`. If found and if the plan touches tested code: "ℹ️ Test command found: `[command]`. Consider running tests before proceeding."
3. Check for `.super-plan-mode.json` — already done at startup; confirm config was applied.

Show pre-flight results above the gate.

### Acceptance Gate

**CRITICAL: DO NOT proceed past this point without explicit user response.**

Present the plan (or a link to the saved file), then output this gate exactly:

```
────────────────────────────────────────────
  Plan ready. What would you like to do?

  1  Accept & Build
  2  Reject / Cancel
  3  Modify plan
  4  Accept Phase 1 only  ← (only shown for L/XL estimates)
────────────────────────────────────────────
  Enter number (or type the action):
```

**Wait. Take no implementation action until the user responds.**

**Response handling:**

- **1** / "accept" / "build" / "yes" / "y" → proceed to Phase 5
- **2** / "reject" / "cancel" / "no" / "n" → acknowledge, summarize useful research findings discovered, stop. Do not modify any files.
- **3** / "modify" / "m" → ask what to change. After revision, show a concise diff of what changed in the plan (sections added, removed, or modified). Re-present the gate.
- **4** / "phase 1" / "p1" → execute only Phase 1 steps from the plan, then return to the gate for remaining phases. (L/XL only)

---

## Phase 5: Implementation

**Only reached after explicit user acceptance in Phase 4.**

1. Mark Phase 4 todo complete.
2. Execute each step from the accepted plan in order.
4. Update TodoWrite after completing each major step.
5. If a step reveals unexpected complexity or a blocker, pause and report before continuing.

**For L/XL plans with phases:**
After completing each phase, pause and output a checkpoint report:
```
Phase [N] complete.
✅ Completed: [summary of what was done]
📁 Files changed: [list]
➡️  Next: Phase [N+1] — [name] ([steps])
```
Ask the user to confirm before starting the next phase.

**On full completion:**
```
✅ Implementation complete.
Files created/modified: [list]
Steps completed: [N/N]
Deviations from plan: [none / description]
Suggested next steps: [testing, docs, deployment, etc.]
```

---

## Process Efficiency Notes

- Each exploration agent has a narrow, non-overlapping scope — architecture, affected files, and risk are researched in parallel without duplication
- Agents return structured tables, not prose — focused and scannable output
- References (`plan-template.md`, `agent-compat.md`) are loaded only when needed
