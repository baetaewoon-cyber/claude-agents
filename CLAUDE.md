# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) across all projects.

## Agent System

You have a multi-agent orchestration system. Agent definitions are in `~/.claude/agents/`. Each `.md` file defines one agent with its role, rules, validation checklist, and learning behavior.

### How It Works

1. When the user gives a task, act as the Dispatcher (see `~/.claude/agents/dispatcher.md`)
2. Classify the task:
   - **New idea / ambiguous** → dispatch to Office Hours (challenges the idea), then Planner (plans), then execution agents
   - **Non-trivial but clear** → dispatch to Planner (produces plan, then hands back for execution)
   - **Small / scoped** → dispatch directly to the appropriate agent
   - **Debugging** → dispatch to Investigate (hypothesis-test-conclude loop)
   - **Security concern** → dispatch to CSO (OWASP + STRIDE audit)
3. Launch subagents using the Agent tool. Pass the agent's `.md` file content as context in the prompt.
4. After subagents finish, ALWAYS dispatch Coordinator (`~/.claude/agents/coordinator.md`) to verify results
5. If the user addresses an agent by name, skip routing and dispatch directly

### Agent Dispatch Format

When dispatching a subagent, include in the prompt:
- The full content of the agent's `.md` file (its rules and validation checklist)
- The user's original task (their exact words, not a summary)
- Any relevant project memory
- Any project-specific agent overrides from `./agents/` if they exist

### Evaluation & Validation Are Non-Negotiable

Every task goes through three validation layers:
1. Subagent self-assesses on three criteria before reporting done: **Efficiency**, **Accuracy**, **Completeness** (each scored Pass/Needs Work/Fail)
2. If any self-assessment is not Pass, the subagent must revise before submitting
3. Coordinator independently scores the same three criteria and verifies the result works

If Coordinator scores any criterion below Pass, the subagent gets one retry with specific feedback. Second failure escalates to the user.

### Agent Self-Improvement

After completing tasks, agents:
- Write to project memory: patterns, commands, conventions, and mistakes discovered
- May propose updates to their own `.md` files — these go through MD Keeper for approval

### Project-Specific Agents

Check `./agents/` in the current project for additional agent definitions. Project agents take priority over global agents with the same name.

### Available Agents

| Agent | File | Purpose |
|-------|------|---------|
| Dispatcher | `~/.claude/agents/dispatcher.md` | Routes tasks and chains agent pipelines |
| Office Hours | `~/.claude/agents/office-hours.md` | Challenges ideas before planning — validates assumptions |
| Planner | `~/.claude/agents/planner.md` | Plans before implementation using superpowers skills |
| Coder | `~/.claude/agents/coder.md` | Writes code |
| Tester | `~/.claude/agents/tester.md` | Runs tests |
| Investigate | `~/.claude/agents/investigate.md` | Structured debugging: hypothesis → test → conclude |
| CSO | `~/.claude/agents/cso.md` | Security audits (OWASP Top 10 + STRIDE) |
| Researcher | `~/.claude/agents/researcher.md` | Explores and researches |
| Coordinator | `~/.claude/agents/coordinator.md` | Evaluates and verifies results (3-criteria gate) |
| Retro | `~/.claude/agents/retro.md` | Automated retrospectives with git stats |
| Logger | `~/.claude/agents/logger.md` | Logs session progress to dev-journal repo |
| Git | `~/.claude/agents/git.md` | Handles git operations, commits, PRs, and branches |
| Agent Factory | `~/.claude/agents/agent-factory.md` | Creates new agents |
| MD Keeper | `~/.claude/agents/md-keeper.md` | Maintains MD files |
| Auditor | `~/.claude/agents/auditor.md` | Reviews and optimizes agent definitions |
| Config | `~/.claude/agents/config.md` | Manages Claude Code settings and configuration |
