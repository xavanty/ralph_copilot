# Ralph for GitHub Copilot CLI

> Adaptation of [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) for GitHub Copilot CLI.  
> An autonomous AI loop with intelligent exit detection, multi-agent orchestration, and native support for Copilot CLI agents.

---

## What is Ralph?

Ralph is an autonomous AI development loop, originally created by [Frank Bria](https://github.com/frankbria) for Claude Code. This adaptation brings the same capability to **GitHub Copilot CLI** (`copilot` command), with three core improvements:

1. **3 critical bugs fixed** to work correctly with Copilot CLI
2. **Agent support** (`~/.copilot/agents/`) with automatic phase-based switching
3. **Multi-agent orchestration via `@fix_plan.md`** — zero extra configuration

---

## Installation

```bash
git clone https://github.com/xavanty/ralph_copilot.git
cd ralph_copilot
chmod +x install.sh
./install.sh
```

**Prerequisite:** GitHub Copilot CLI installed and authenticated.
```bash
copilot --version   # version ≥ 1.0.0
```

This installs three global commands: `ralph`, `ralph-setup`, `ralph-monitor`.

---

## Quick Start

```bash
# 1. Create a new project
ralph-setup my-project
cd my-project

# 2. Edit PROMPT.md with your project goal
# 3. Edit @fix_plan.md with your tasks (see format below)
# 4. Run
ralph -v
```

---

## The Two Core Files

### `PROMPT.md` — Agent Instructions

`PROMPT.md` is read every loop and provides:
- **Project context**: what is being built or generated
- **Per-agent instructions**: what each agent should do when activated
- **Available tools**: `view`, `create`, `edit`, `bash`, `glob`, `grep`
- **RALPH_STATUS block**: required at the end of every response

```markdown
# Ralph Instructions — [Project Name]

## Project Context
[Describe the project and goal]

## If you are the `[agent-name-1]` agent:
[Specific instructions for this agent]

## If you are the `[agent-name-2]` agent:
[Specific instructions for this agent]

## Status Reporting (REQUIRED at end of every response)

---RALPH_STATUS---
STATUS: IN_PROGRESS | COMPLETE | BLOCKED
TASKS_COMPLETED_THIS_LOOP: <number>
FILES_MODIFIED: <number>
TESTS_STATUS: NOT_RUN | PASSING | FAILING
WORK_TYPE: RESEARCH | GENERATION | DOCUMENTATION | PUBLICATION
EXIT_SIGNAL: false | true
COMPONENTS_PROCESSED: <done>/<total>
RECOMMENDATION: <next action in one line>
---END_RALPH_STATUS---

EXIT_SIGNAL: true only when ALL [ ] tasks in your section of @fix_plan.md are marked [x].
```

See `PROMPT.md.example` for a full annotated template.

---

### `@fix_plan.md` — Task List with Per-Section Agents

`@fix_plan.md` is the central control file. It defines:
- **Tasks** to execute (`- [ ]` pending, `- [x]` done)
- **The responsible agent** per section (syntax: `## [agent-name] Title`)

Ralph reads this file every loop and automatically activates the correct agent.

```markdown
# Fix Plan — [Project Name]

## [agent-name-1] Phase 1: Research
- [ ] Gather information on [TOPIC]
- [ ] Save findings to docs/research/[topic].md

## [agent-name-2] Phase 2: Generation
- [ ] Read research and generate output document
- [ ] Verify output is complete

## [agent-name-3] Phase 3: Review
- [ ] Review the generated document
- [ ] Record completion
```

**How agent detection works:**
1. Ralph reads `@fix_plan.md` line by line at the start of each loop
2. Finds the first `## [agent-name]` section with at least one incomplete `- [ ]` task
3. Activates that agent via `--agent <name>` when calling Copilot CLI
4. When the section is fully done (all `[x]`), Ralph automatically:
   - Resets exit signals
   - Starts a new Copilot session
   - Loads the next agent

---

## Multi-Agent Flow

```
@fix_plan.md                  Ralph                Copilot CLI
───────────────────────────────────────────────────────────────
## [agent-1] Phase 1          ──→  --agent agent-1  ──→  works
  - [x] Task 1 ✓                                          on tasks
  - [x] Task 2 ✓              ←── EXIT_SIGNAL: true ←──

                               resets signals, new session
                               detects next incomplete section

## [agent-2] Phase 2          ──→  --agent agent-2  ──→  works
  - [ ] Task 3                                            on tasks
  - [ ] Task 4                ←── EXIT_SIGNAL: true ←──

                               project complete — exit 0
```

---

## RALPH_STATUS Block — Required Format

Every `PROMPT.md` must instruct the agent to end **every response** with:

```
---RALPH_STATUS---
STATUS: IN_PROGRESS | COMPLETE | BLOCKED
TASKS_COMPLETED_THIS_LOOP: <number>
FILES_MODIFIED: <number>
TESTS_STATUS: PASSING | FAILING | NOT_RUN
WORK_TYPE: RESEARCH | GENERATION | DOCUMENTATION | PUBLICATION
EXIT_SIGNAL: false | true
COMPONENTS_PROCESSED: <done>/<total>
RECOMMENDATION: <next action in one line>
---END_RALPH_STATUS---
```

| Field | When to use |
|-------|-------------|
| `EXIT_SIGNAL: true` | ALL `[ ]` tasks in the current section are now `[x]` |
| `STATUS: BLOCKED` | Missing external dependency (credentials, file, API) |
| `STATUS: COMPLETE` | Entire project is done |

---

## Per-Project Configuration (`.ralph.env`)

Create `.ralph.env` in your project directory to override defaults:

```bash
# .ralph.env
COPILOT_AGENT="my-agent"                               # fixed agent (fallback when no ## [agent] in fix_plan)
COPILOT_ALLOWED_TOOLS="create,view,edit,bash,glob,grep"  # available tools
```

> When `@fix_plan.md` uses `## [agent]` syntax, `COPILOT_AGENT` in `.ralph.env` is ignored — the agent is always read from the fix_plan.

---

## Examples

### `examples/multi-agent/`
A complete 3-phase pipeline using generic agents:
```
researcher → developer → reviewer
  Phase 1     Phase 2    Phase 3
```
Includes `@fix_plan.md`, per-phase prompt files, and `.ralph.env`.

### `examples/rest-api/` and `examples/simple-cli-tool/`
Single-agent examples from the original frankbria project.

---

## Differences from the Original Project

| Aspect | frankbria/ralph-claude-code | xavanty/ralph_copilot |
|--------|-----------------------------|-----------------------|
| AI runtime | Claude Code | GitHub Copilot CLI |
| Tool names | `write`, `read`, `shell` | `create`, `view`, `edit`, `bash` |
| Session continuity | `--continue` / `--resume` | `COPILOT_USE_CONTINUE=false` |
| Exit detection | keywords + EXIT_SIGNAL | EXIT_SIGNAL explicit (keywords ignored when `EXIT_SIGNAL: false`) |
| Agents | Claude Code subagents | `~/.copilot/agents/` + `--agent` flag |
| Multi-agent orchestration | not native | via `## [agent]` in `@fix_plan.md` |

## Project Structure

```
ralph_copilot/
├── ralph_loop.sh             # Main loop — reads @fix_plan.md, detects agent, calls copilot
├── install.sh                # Installs ralph, ralph-setup, ralph-monitor globally
├── setup.sh                  # Used by ralph-setup to scaffold a new project
├── lib/
│   ├── response_analyzer.sh  # Parses copilot JSONL output, detects RALPH_STATUS
│   ├── circuit_breaker.sh    # Prevents infinite loops
│   └── ...
├── templates/
│   ├── PROMPT.md             # PROMPT.md template for new projects
│   └── fix_plan.md           # @fix_plan.md template for new projects
├── examples/
│   ├── multi-agent/          # 3-phase multi-agent pipeline example
│   ├── rest-api/             # Single-agent REST API example
│   └── simple-cli-tool/      # Single-agent CLI tool example
└── PROMPT.md.example         # Full annotated PROMPT.md template
```

---

## Credits

Based on [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) by [Frank Bria](https://github.com/frankbria).  
This fork adapts Ralph for GitHub Copilot CLI and adds multi-agent orchestration via `@fix_plan.md`.

---

## License

MIT — see [LICENSE](LICENSE).
