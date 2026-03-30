```
 ███████╗██╗   ██╗██████╗ ███████╗██████╗
 ██╔════╝██║   ██║██╔══██╗██╔════╝██╔══██╗
 ███████╗██║   ██║██████╔╝█████╗  ██████╔╝
 ╚════██║██║   ██║██╔═══╝ ██╔══╝  ██╔══██╗
 ███████║╚██████╔╝██║     ███████╗██║  ██║
 ╚══════╝ ╚═════╝ ╚═╝     ╚══════╝╚═╝  ╚═╝

 ██████╗ ██╗      █████╗ ███╗   ██╗    ███╗   ███╗ ██████╗ ██████╗ ███████╗
 ██╔══██╗██║     ██╔══██╗████╗  ██║    ████╗ ████║██╔═══██╗██╔══██╗██╔════╝
 ██████╔╝██║     ███████║██╔██╗ ██║    ██╔████╔██║██║   ██║██║  ██║█████╗
 ██╔═══╝ ██║     ██╔══██║██║╚██╗██║    ██║╚██╔╝██║██║   ██║██║  ██║██╔══╝
 ██║     ███████╗██║  ██║██║ ╚████║    ██║ ╚═╝ ██║╚██████╔╝██████╔╝███████╗
 ╚═╝     ╚══════╝╚═╝  ╚═╝╚═╝  ╚═══╝   ╚═╝     ╚═╝ ╚═════╝ ╚═════╝ ╚══════╝
```

<div align="center">

**Plan first. Build with confidence.**

*Built natively for [OpenClaw](https://openclaw.dev) and [Claude Code](https://claude.ai/code) · Works with Cursor, Windsurf, Codex CLI, Aider, and more.*

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/temikeezy/super-plan-mode)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![OpenClaw Native](https://img.shields.io/badge/OpenClaw-Native-orange.svg)](https://openclaw.dev)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Native-blueviolet.svg)](https://claude.ai/code)
[![Agent Skills Standard](https://img.shields.io/badge/Agent%20Skills-Open%20Standard-brightgreen.svg)](https://agentskills.io)

</div>

---

## What It Is

**Super Plan Mode** is a universal agent skill that generates a rich implementation plan and **halts until you explicitly approve it** before writing a single line of code.

Built natively for [OpenClaw](https://openclaw.dev) and [Claude Code](https://claude.ai/code) following the [Agent Skills Open Standard](https://agentskills.io). Compatible with any AI coding agent — Cursor, Windsurf, Codex CLI, Aider, and more.

No passive guardrails. No soft suggestions. A hard stop, a structured plan document, and a numbered menu waiting for your decision.

---

## Features

- 🛑 **Explicit accept/build gate** — numbered menu halts execution; nothing runs until you say so
- 📋 **Rich plan artifacts** — problem statement, affected files, implementation steps, risk matrix, effort estimate, acceptance criteria, rollback notes, and alternative approaches — all in one saved document
- 🟢🟡🔴 **Confidence scores** — every implementation step rated by how well-understood it is based on codebase exploration
- 🔍 **Parallel codebase exploration** — multiple read-only agents research architecture, affected files, and risks simultaneously (supported in OpenClaw, Claude Code, Cursor 2.4+, Windsurf Wave 13+, and Codex CLI)
- ⚠️ **Pre-flight checks** — warns if uncommitted changes overlap with the plan before you accept
- 🔁 **Modify loop** — type `3` to revise the plan; see a diff of what changed before re-approving
- 💾 **Plan persistence** — plans saved to `.claude/plans/` for reference, sharing, and resuming
- ▶️ **Resume saved plans** — `--resume <file>` loads a past plan and jumps straight to the gate
- 📁 **Plan history** — `--list` shows all saved plans for the current project
- 🌐 **Multi-agent compatible** — works across Claude Code, Codex CLI, Cursor Agent, Windsurf, and Aider
- 🤖 **Model + provider flag** — `--model provider/model` lets you choose the implementation model from any provider
- 💨 **Dry-run mode** — `--dry-run` generates and saves the plan without offering to execute
- ⚡ **Token optimized** — exploration agents use `haiku` by default; structured output keeps costs low
- ⚙️ **Project config** — `.super-plan-mode.json` sets per-project defaults for model, save path, and checkpoints

---

## Quick Start

```bash
# Full plan → gate → build workflow
/super-plan-mode add OAuth login with Google and GitHub

# Shorthand alias (same thing)
/spm add a payment webhook handler

# Generate plan only, build later
/spm --dry-run refactor the auth middleware

# Resume a saved plan
/spm --resume .claude/plans/super-plan-mode-1234567890.md

# List all saved plans
/spm --list

# Choose implementation model
/spm --model openai/gpt-4o add a caching layer
/spm --model anthropic/claude-opus-4-6 redesign the API router
/spm --model ollama/llama3.3 add input validation
```

Or trigger passively — just tell the agent:

> "Let's plan this before writing any code"
> "Show me the plan first"
> "Don't touch anything yet, just plan it out"

---

## How It Works

### Phase 1 — Understand

Clarifies the request. Asks focused questions if anything is ambiguous. Confirms understanding before moving on.

### Phase 2 — Explore (Read-Only)

Launches parallel `plan-researcher` agents (where supported) to map the codebase from three angles:

- **Architecture** — patterns, conventions, existing similar features
- **Affected Files** — what changes, what depends on it, blast radius
- **Risk / Conflicts** — uncommitted changes, fragile files, recent churn

In single-agent environments (Cursor, Codex, Windsurf, Aider), the same research runs inline sequentially. Same result either way.

### Phase 3 — Generate Plan

Produces a complete plan document and saves it to `.claude/plans/`. The plan includes:

| Section | Description |
|---------|-------------|
| Problem Statement | What's broken or missing |
| Goal | Verifiable outcome |
| Affected Files | Every file, change type, and reason |
| Implementation Steps | Ordered steps with confidence scores |
| Risk Assessment | Overall rating + risk matrix |
| Effort Estimate | S / M / L / XL with reasoning |
| Acceptance Criteria | Testable checklist |
| Rollback Notes | How to undo (required for Medium/High risk) |
| Alternative Approaches | At least one alternative with trade-offs |
| Phases | Checkpoints for Large/XL work |

### Phase 4 — The Gate

Pre-flight checks run first (uncommitted file conflicts, test command detection). Then the plan is presented with this menu:

```
────────────────────────────────────────────
  Plan ready. What would you like to do?

  1  Accept & Build
  2  Reject / Cancel
  3  Modify plan
  4  Accept Phase 1 only  ← (large plans only)
────────────────────────────────────────────
  Enter number (or type the action):
```

**Nothing happens until you respond.** Type `3` to iterate; see what changed in the plan before re-approving. Type `2` to cancel cleanly.

### Phase 5 — Implementation

Executes the approved plan step by step. For Large/XL plans, pauses at each phase checkpoint to report progress and confirm continuation. Ends with a summary of everything that changed.

---

## Confidence Scores

Every implementation step is rated based on how thoroughly it was understood during exploration:

| Score | Meaning |
|-------|---------|
| 🟢 High | Well-understood. Clear file paths, existing patterns to follow. |
| 🟡 Medium | Some uncertainty. May need additional discovery mid-implementation. |
| 🔴 Low | Significant unknowns. Review carefully before accepting. |

---

## Model Flag

Use any model from any provider for implementation:

```bash
# Anthropic (default)
/spm --model anthropic/claude-sonnet-4-6 <task>
/spm --model sonnet <task>        # shorthand

# OpenAI
/spm --model openai/gpt-4o <task>
/spm --model openai/o3 <task>

# Google
/spm --model google/gemini-2.5-pro <task>

# Local
/spm --model ollama/llama3.3 <task>
/spm --model lmstudio/my-model <task>
```

> Planning agents always use `anthropic/haiku` regardless of your `--model` flag — keeping exploration fast and cheap.

Unknown model names are accepted as passthrough with a configuration note.

---

## Project Config

Add `.super-plan-mode.json` to your project root to set defaults:

```json
{
  "defaultModel": "anthropic/claude-sonnet-4-6",
  "planSaveDir": ".claude/plans",
  "autoPhaseCheckpoints": true,
  "preflightChecks": true
}
```

CLI flags always override config file values.

---

## Works With

| Agent | Support | Subagents | Gate |
|-------|---------|-----------|------|
| **OpenClaw** ⭐ | Native skill (agentskills.io) | ✅ Parallel | Numbered menu |
| **Claude Code** ⭐ | Native plugin | ✅ Parallel | Numbered menu |
| **Cursor Agent** | Compatible (v2.4+) | ✅ Parallel + async | Numbered menu |
| **Windsurf (Cascade)** | Compatible (Wave 13+) | ✅ Parallel (5 agents) | Numbered menu |
| **Codex CLI** | Compatible | ✅ Parallel (Agents SDK) | Numbered menu |
| **Aider** | Compatible | ➡️ Inline sequential | Numbered menu |

See `skills/plan-mode/references/agent-compat.md` for detailed per-agent wiring instructions.

---

## File Structure

```
super-plan-mode/
├── .claude-plugin/
│   └── plugin.json              Plugin metadata
├── commands/
│   ├── super-plan-mode.md       /super-plan-mode command
│   └── spm.md                   /spm shorthand alias
├── agents/
│   ├── plan-researcher.md       Read-only exploration agent
│   └── risk-analyzer.md         Git-forensics risk agent
├── skills/
│   └── super-plan-mode/
│       ├── SKILL.md             Passive trigger skill
│       └── references/
│           ├── plan-template.md Canonical plan format
│           └── agent-compat.md  Multi-agent guide
└── .super-plan-mode.json        Example project config
```

---

## Installation

### OpenClaw (Native — Agent Skills Open Standard)

Drop `skills/super-plan-mode/` into your project's `.agents/skills/` directory:

```bash
# Via OpenClaw skill manager
/skills install temikeezy/super-plan-mode

# Or manually
cp -r skills/super-plan-mode /your-project/.agents/skills/super-plan-mode
```

The skill is [Agent Skills Open Standard](https://agentskills.io) compatible — it loads automatically when OpenClaw scans the `.agents/skills/` directory.

### Claude Code (native plugin)

```bash
# Install from GitHub
/plugins install temikeezy/super-plan-mode
```

### Manual (any agent)

Clone the repo and copy the command file into your project's `AGENTS.md`, `CLAUDE.md`, or equivalent system prompt file:

```bash
git clone https://github.com/temikeezy/super-plan-mode
# Then paste commands/super-plan-mode.md into your agent config
```

### Cursor / Windsurf / Aider

Copy the content of `commands/super-plan-mode.md` into your project rules file (`.cursorrules`, `AGENTS.md`, Windsurf system prompt, or Aider `CONVENTIONS.md`). The workflow and gate operate entirely through text instructions.

---

## Contributing

Issues and pull requests welcome at [github.com/temikeezy/super-plan-mode](https://github.com/temikeezy/super-plan-mode).

To add a new agent environment to the compatibility guide, open a PR editing `skills/super-plan-mode/references/agent-compat.md`.

---

## License

MIT — see [LICENSE](LICENSE).
