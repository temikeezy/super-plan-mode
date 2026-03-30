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

**Exploration:** Full parallel subagent support via OpenClaw's agent orchestration. Launch `plan-researcher` agents in parallel for architecture, affected-files, and risk angles simultaneously.

**Gate:** Numbered menu in chat. User types `1`, `2`, `3`, or `4`. Full-word alternatives also accepted.

**Model flag:** `--model provider/model` parsed and applied to implementation. Planning agents always use `anthropic/haiku` regardless.

**Plan files:** Saved to `.agents/plans/super-plan-mode-[timestamp].md` (or `.claude/plans/` — either works). Use `--list` to browse, `--resume <file>` to reload.

**Config file:** `.super-plan-mode.json` in the project root. User flags override config values.

**SKILL.md metadata:** The skill follows the full Agent Skills Open Standard frontmatter — `name`, `description`, `license`, and `metadata` block (author, version, organization, date, abstract) — ensuring compatibility with OpenClaw's skill registry and hash verification.

---

## Claude Code ⭐ (Native Plugin)

**Exploration:** Full parallel subagent support. Launch 2–3 `plan-researcher` agents simultaneously using the Task tool. Each agent targets a different angle (architecture, affected files, risk). After completion, read all files they identify as critical.

**Gate:** Numbered menu rendered in chat. User types `1`, `2`, `3`, or `4`. Full-word alternatives also accepted.

**Model flag:** `--model provider/model` is parsed and the implementation model is set accordingly. Planning agents (`plan-researcher`, `risk-analyzer`) always use `anthropic/haiku` regardless of the user flag.

**Plan files:** Saved to `.claude/plans/super-plan-mode-[timestamp].md` in the project root. Use `--list` to browse saved plans. Use `--resume <file>` to reload and execute a previously generated plan.

**Config file:** `.super-plan-mode.json` in the project root is read at startup. User flags override config values.

---

## Codex CLI

**Exploration:** No subagent support. Run exploration inline sequentially:
1. Map architecture and patterns (Glob, Grep, Read)
2. Trace affected files and dependencies
3. Assess risks and run `git status` / `git log`

Follow the same methodology described in `agents/plan-researcher.md` — that file doubles as the inline research guide.

**Gate:** Numbered menu in chat output. User types the number.

**Model flag:** Codex CLI uses `--model` natively. Pass `provider/model` as the value if the CLI supports it; otherwise pass just the model name. Check your Codex CLI version for exact syntax.

**Plan files:** Saved to the project root as `super-plan-mode-[timestamp].md` (no `.claude/plans/` directory in non-Claude Code environments unless you create it).

---

## Cursor Agent

**Exploration:** Cursor Agent does not support subagent spawning. Use inline sequential exploration following the `plan-researcher` methodology. Cursor's multi-step instruction support handles the phased flow naturally.

**Gate:** The numbered menu works in Cursor chat. User types the number. Cursor's built-in "Apply" button can also be used as the acceptance mechanism for code suggestions — but the numbered gate is the canonical approach.

**Model flag:** Set the model via Cursor's model selector (top of chat). The `--model` flag value in the command is advisory — document the requested model in the plan header.

**Plan files:** Saved to `.claude/plans/` if the directory exists, otherwise to project root. Cursor can open and display the plan markdown file natively.

**Tip:** Pin the super-plan-mode command in Cursor's rules or AGENTS.md to make it always available.

---

## Windsurf (Cascade)

**Exploration:** Inline sequential exploration. Windsurf's Cascade model supports multi-step reasoning natively, so the phased exploration flow works without modification.

**Gate:** Cascade's natural pause-for-review behavior aligns well with the numbered gate. Output the numbered menu and Cascade will wait for the user's input before continuing.

**Model flag:** Set via Windsurf's model selector. The `--model` flag is advisory in this environment.

**Plan files:** Saved to `.claude/plans/` or project root. Windsurf renders markdown natively in the workspace.

---

## Aider

**Exploration:** Use Aider's `/read` command to load relevant files before generating the plan. Run `git status` and `git log` via `/run` commands during exploration.

**Gate:** The numbered menu works in Aider's chat. For a stronger guarantee, use `--dry-run` mode:
1. Run Aider with `--dry-run` flag during the planning phases
2. The plan is generated and saved but no code is written
3. Review the plan file
4. Re-run Aider (without `--dry-run`) with `--message "Resume from plan: .claude/plans/super-plan-mode-[timestamp].md"` to execute

**Model flag:** Pass `--model provider/model` to Aider's CLI directly. Aider has native multi-provider model support (OpenAI, Anthropic, Google, Ollama, etc.).

**Plan files:** Saved to project root. Use `/add super-plan-mode-[timestamp].md` to bring the plan into Aider's context for execution.

---

## Model / Provider Reference

The `--model` flag uses `provider/model` format. Known values:

### Anthropic
- `anthropic/claude-opus-4-6`
- `anthropic/claude-sonnet-4-6` (default)
- `anthropic/claude-haiku-4-5`
- Shorthand: `opus`, `sonnet`, `haiku`

### OpenAI
- `openai/gpt-4o`
- `openai/gpt-4o-mini`
- `openai/o3`
- `openai/o4-mini`

### Google
- `google/gemini-2.5-pro`
- `google/gemini-2.5-flash`
- `google/gemini-2.0-flash`

### Mistral
- `mistral/mistral-large`
- `mistral/mistral-small`

### Groq
- `groq/llama-3.3-70b`
- `groq/deepseek-r1-distill-llama-70b`

### Local / Self-hosted
- `ollama/<model-name>` — passthrough (any locally running Ollama model)
- `bedrock/<model-id>` — passthrough (AWS Bedrock)
- `lmstudio/<model-name>` — passthrough (LM Studio local server)

### Unknown models
Unrecognized `provider/model` values are accepted as passthrough with a warning:
> "Note: `provider/model` is not in the known model list. Proceeding — ensure your environment is configured for this model."

---

## Token Efficiency Notes

Planning agents (`plan-researcher`, `risk-analyzer`) always use `anthropic/haiku` regardless of the `--model` flag. This keeps exploration cheap:

- 3 parallel haiku agents ≈ cost of ~0.3 sonnet calls
- Each agent returns structured output (not prose) with hard output caps
- References in `skills/super-plan-mode/references/` are only loaded when needed (not always in context)
- SKILL.md body is kept under 2,000 words — detailed content stays in `references/`
