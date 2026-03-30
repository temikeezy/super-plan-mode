---
description: Shorthand alias for /super-plan-mode. Generates a rich implementation plan with explicit accept/build gate before any code changes.
argument-hint: "[--model provider/model] [--dry-run] [--resume <plan-file>] [--list] <task description>"
---

# SPM (Super Plan Mode)

This is a shorthand alias for `/super-plan-mode`. All flags and behavior are identical.

**Arguments:** $ARGUMENTS

Execute the full super-plan-mode workflow with the provided arguments. Follow all instructions in `commands/super-plan-mode.md` exactly, passing `$ARGUMENTS` as the task input.

Quick reference:
- `/spm add OAuth login` — full plan + gate workflow
- `/spm --dry-run add OAuth login` — generate plan only, no execution
- `/spm --resume .claude/plans/super-plan-mode-1234567890.md` — resume from saved plan
- `/spm --list` — browse previously generated plans
- `/spm --model provider/model add OAuth login` — use a specific model for implementation
