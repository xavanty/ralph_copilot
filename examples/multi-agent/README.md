# Example: Multi-Agent Pipeline

This example shows how to use Ralph with multiple agents, each handling a different phase of a project.

## How It Works

```
researcher → developer → reviewer
  Phase 1     Phase 2    Phase 3
```

Ralph reads `@fix_plan.md` each loop and activates the agent whose section has incomplete tasks.
When a section is fully done (`[x]`), Ralph automatically switches to the next agent.

## Setup

1. Define your agents in `~/.copilot/agents/` (one `.md` file per agent)
2. Edit `@fix_plan.md` — replace `[TOPIC]`, `[topic]`, and agent names with your own
3. Edit `PROMPT.md` — describe what each agent should do
4. Run:
   ```bash
   ralph -v
   ```

## Customizing Agent Names

The agent names in `@fix_plan.md` must match filenames in `~/.copilot/agents/`.
For example, `## [researcher]` activates `~/.copilot/agents/researcher.md`.

Change the names to whatever agents you have configured:
```markdown
## [my-agent-1] Phase 1: Data Gathering
## [my-agent-2] Phase 2: Processing
## [my-agent-3] Phase 3: Output
```

## Reset State

To restart from scratch:
```bash
rm -f .exit_signals .circuit_breaker_state .ralph_last_agent .copilot_session_id
sed -i 's/- \[x\]/- [ ]/g' @fix_plan.md
```
