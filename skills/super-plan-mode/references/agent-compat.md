# Multi-Agent Compatibility Guide

Super-plan-mode is designed to work with any AI coding assistant. The core workflow is text-based behavioral instruction — not tool-level enforcement. The accept gate works because it is a behavioral contract embedded in the workflow, not a system lock.

This file documents how each supported environment wires up the workflow, exploration strategy, model selection, and plan file handling.

---

## Universal Pattern

Regardless of agent environment, the flow is always:

```
1. Understand request (clarify if needed)
2. Explore codebase (read-only, parallel or sequential)
3. Generate plan file (save to disk)
4. Present numbered gate (halt, wait for response)
5. Implement only after explicit acceptance
```

---

## OpenClaw ⭐ (Native)

Super-plan-mode is built as an [Agent Skills Open Standard](https://agentskills.io) skill for OpenClaw. Drop `skills/super-plan-mode/` into your project's `.agents/skills/` directory and it is discovered automatically.

**Skill discovery:** OpenClaw scans `.agents/skills/`, `.agent/skills/`, and `.claude/skills/` directories. Place the `super-plan-mode/` folder in any of these and it loads on the next session.

**Exploration:** Full parallel subagent support via OpenClaw's agent orchestration. Launch 3 parallel explore agents for architecture, affected-files, and risk angles simultaneously.

**Gate:** Numbered menu in chat. User types `1`, `2`, `3`, or `4`. Full-word alternatives also accepted.

**Model flag:** `--model provider/model` parsed and applied to implementation.

**Plan files:** Saved to `.agents/plans/super-plan-mode-[timestamp].md` (or `.claude/plans/` — either works). Use `--list` to browse, `--resume <file>` to reload.

**Config file:** `.super-plan-mode.json` in the project root. User flags override config values.

**Slash command autocomplete:** The skill includes a `commands/` directory (`commands/super-plan-mode.md` and `commands/spm.md`). OpenClaw reads these when the skill loads and registers `/super-plan-mode` and `/spm` in the `/` autocomplete dropdown. Both commands appear with their description and argument hint as soon as the skill is installed.

**SKILL.md metadata:** The skill follows the full Agent Skills Open Standard frontmatter — `name`, `description`, `license`, and `metadata` block (author, version, organization, date, abstract) — ensuring compatibility with OpenClaw's skill registry and hash verification.

---

## Claude Code ⭐ (Native Plugin)

**Exploration:** Full parallel subagent support. Launch 2–3 parallel `Explore` agents simultaneously. Each agent targets a different angle (architecture, affected files, risk). After completion, read all files they identify as critical.

**Gate:** Numbered menu rendered in chat. User types `1`, `2`, `3`, or `4`. Full-word alternatives also accepted.

**Model flag:** `--model provider/model` is parsed and the implementation model is set accordingly.

**Plan files:** Saved to `.claude/plans/super-plan-mode-[timestamp].md` in the project root. Use `--list` to browse saved plans. Use `--resume <file>` to reload and execute a previously generated plan.

**Config file:** `.super-plan-mode.json` in the project root is read at startup. User flags override config values.

---

## Cursor Agent

**Exploration:** Full parallel subagent support since Cursor 2.4 (January 2026). Launch 2–3 parallel explore agents — each runs in its own isolated workspace using git worktrees. Subagents can also spawn their own sub-subagents and run asynchronously, so the parent agent continues while researchers complete.

**Gate:** Numbered menu in Cursor chat. User types the number. Cursor's built-in "Apply" button can serve as the acceptance trigger for code suggestions — but the numbered gate is the canonical approach.

**Model flag:** Pass `--model provider/model` in the command arguments.

**Plan files:** Saved to `.claude/plans/` if the directory exists, otherwise to project root. Cursor renders markdown natively.

**Tip:** Pin the super-plan-mode command in Cursor's `.cursorrules` or `AGENTS.md` to make it always available.

---

## Windsurf (Cascade)

**Exploration:** Full parallel agent support since Wave 13. Launch up to 5 parallel explore agents simultaneously, each in its own pane. Cascade's multi-agent interface lets you monitor them side by side.

**Gate:** Cascade's natural pause-for-review behavior aligns with the numbered gate. Output the numbered menu — Cascade waits for input before continuing.

**Model flag:** Set via Windsurf's model selector or pass `--model` in the command. The flag is documented in the plan header for reference.

**Plan files:** Saved to `.claude/plans/` or project root. Windsurf renders markdown natively in the workspace.

---

## Codex CLI

**Exploration:** Full parallel subagent support via the OpenAI Agents SDK. Spawn parallel explore agents for architecture, affected-files, and risk angles simultaneously.

**Gate:** Numbered menu in chat output. User types the number.

**Model flag:** Pass `--model provider/model` to the CLI. Codex natively supports OpenAI models; for other providers use the Agents SDK orchestration layer.

**Plan files:** Saved to the project root as `super-plan-mode-[timestamp].md` (create `.claude/plans/` manually if preferred).

---

## Aider

**Exploration:** Use Aider's `/read` command to load relevant files before generating the plan. Run `git status` and `git log` via `/run` commands during exploration.

**Gate:** The numbered menu works in Aider's chat. For a stronger guarantee, use `--dry-run` mode:
1. Run Aider with `--dry-run` flag during the planning phases
2. The plan is generated and saved but no code is written
3. Review the plan file
4. Re-run Aider (without `--dry-run`) with `--message "Resume from plan: .claude/plans/super-plan-mode-[timestamp].md"` to execute

**Model flag:** Pass `--model provider/model` to Aider's CLI directly.

**Plan files:** Saved to project root. Use `/add super-plan-mode-[timestamp].md` to bring the plan into Aider's context for execution.

---

## Process Efficiency Notes

- Exploration agents have narrow, non-overlapping scopes — no duplicated research
- Agents return structured tables, not prose
- References in `skills/super-plan-mode/references/` are loaded only when needed
- SKILL.md body stays lean — detailed content lives in `references/`
