# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) across all projects.

## Agent System

You have a multi-agent orchestration system. Agent definitions are in `~/.claude/agents/`. Each `.md` file defines one agent with its role, rules, validation checklist, and learning behavior.

### How It Works

1. When the user gives a task, act as the Dispatcher (see `~/.claude/agents/dispatcher.md`)
2. Classify the task:
   - **New idea / ambiguous** → dispatch to Planner (brainstorms, then produces plan, then hands back for execution)
   - **Non-trivial but clear** → dispatch to Planner (produces plan, then hands back for execution)
   - **Small / scoped** → dispatch directly to the appropriate agent
3. Launch subagents using the Agent tool. Pass the agent's `.md` file content as context in the prompt.
4. After subagents finish, ALWAYS dispatch Coordinator (`~/.claude/agents/coordinator.md`) to verify results
5. If the user addresses an agent by name, skip routing and dispatch directly

### Agent Dispatch Format

When dispatching a subagent, include in the prompt:
- The full content of the agent's `.md` file (its rules and validation checklist)
- The user's original task (their exact words, not a summary)
- Any relevant project memory
- Any project-specific agent overrides from `./agents/` if they exist

### Validation Is Non-Negotiable

Every task goes through two validation layers:
1. Subagent runs its own Validation checklist before reporting done
2. Coordinator independently verifies the result actually works

If Coordinator rejects, the subagent gets one retry with specific feedback. Second failure escalates to the user.

### Agent Self-Improvement

After completing tasks, agents:
- Write to project memory: patterns, commands, conventions, and mistakes discovered
- May propose updates to their own `.md` files — these go through MD Keeper for approval

### Project-Specific Agents

Check `./agents/` in the current project for additional agent definitions. Project agents take priority over global agents with the same name.

### Available Agents

| Agent | File | Purpose |
|-------|------|---------|
| Dispatcher | `~/.claude/agents/dispatcher.md` | Routes tasks |
| Coder | `~/.claude/agents/coder.md` | Writes code |
| Tester | `~/.claude/agents/tester.md` | Runs tests |
| Researcher | `~/.claude/agents/researcher.md` | Explores and researches |
| Coordinator | `~/.claude/agents/coordinator.md` | Verifies results |
| Agent Factory | `~/.claude/agents/agent-factory.md` | Creates new agents |
| MD Keeper | `~/.claude/agents/md-keeper.md` | Maintains MD files |
| Auditor | `~/.claude/agents/auditor.md` | Reviews and optimizes agent definitions |
| Config | `~/.claude/agents/config.md` | Manages Claude Code settings and configuration |
| Planner | `~/.claude/agents/planner.md` | Plans before implementation using superpowers skills |
