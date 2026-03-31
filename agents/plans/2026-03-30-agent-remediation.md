# Agent System Remediation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix all 18 agent `.md` files based on Codex review findings -- structural non-conformance, undefined skill references, validation gaps, incomplete routing, aggressive git defaults, dead rules, and missing measurable thresholds.

**Architecture:** Each task targets one agent file (or one cross-cutting concern). Tasks are grouped into four priority tiers by system-wide impact. Within each tier, tasks are independent and can run in parallel. Every change is a text edit to an existing `.md` file -- no new files created except this plan.

**Tech Stack:** Markdown only. All files in `~/.claude/agents/`.

---

## Priority Framework

| Tier | Impact | Files | Rationale |
|------|--------|-------|-----------|
| **P0: System Infrastructure** | Blocks correct routing and evaluation for ALL agents | `_template.md`, `dispatcher.md`, `coordinator.md` | Template sets the standard, Dispatcher routes everything, Coordinator gates everything. Fixing these first prevents cascading issues. |
| **P1: Safety-Critical Defaults** | Could cause data loss or unwanted side effects | `git.md`, `trader.md` | Aggressive auto-commit, auto-PR, auto-merge, and missing trading safeguards are the highest-risk behavioral bugs. |
| **P2: Agent Effectiveness** | Individual agents produce incorrect or incomplete work | `coder.md`, `tester.md`, `planner.md`, `cso.md`, `investigate.md`, `office-hours.md` | Undefined skills, weak validation, dead rules -- these cause agents to silently fail or produce low-quality output. |
| **P3: Polish & Consistency** | Minor gaps, hardcoded values, thin validation | `agent-factory.md`, `auditor.md`, `config.md`, `logger.md`, `md-keeper.md`, `researcher.md`, `retro.md` | Real issues but lower blast radius. Fix last to avoid churn if P0 changes affect them. |

---

## P0: System Infrastructure

### Task 1: Fix `_template.md` -- Add Rule-Validation Mapping Requirement

**Files:**
- Modify: `~/.claude/agents/_template.md`

**Problem:** Template does not require validation items to map back to rules. This is the root cause of validation gaps across all agents.

- [ ] **Step 1: Add mapping requirement to Rules section guidance**

In `_template.md`, replace line 10:
```markdown
{Numbered list of specific behaviors. Each rule must change how the agent acts — if removing it changes nothing, delete it.}
```
with:
```markdown
{Numbered list of specific behaviors. Each rule must change how the agent acts — if removing it changes nothing, delete it. Every rule must be verified by at least one validation item below.}
```

- [ ] **Step 2: Add mapping requirement to Validation section guidance**

In `_template.md`, replace line 24:
```markdown
{Steps this agent MUST complete before reporting done. These are pass/fail checks, not descriptions.}
```
with:
```markdown
{Steps this agent MUST complete before reporting done. These are pass/fail checks, not descriptions. Each validation item must verify one or more specific rules above — no orphan rules allowed.}
```

- [ ] **Step 3: Verify the template still renders correctly**

Read `_template.md` and confirm all six sections (Role, Trigger, Rules, Evaluation Criteria, Validation, Learning) are present and the new text is clear.

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/_template.md
git commit -m "fix(template): require validation items to map back to rules"
```

---

### Task 2: Fix `dispatcher.md` -- Add `## Rules` Section, Missing Routes, Fix Auto-Commit

**Files:**
- Modify: `~/.claude/agents/dispatcher.md`

**Problems:**
1. No `## Rules` section -- structural non-conformance with template
2. Missing routes for Researcher, Agent Factory, MD Keeper
3. Auto-commit too aggressive (Step 4 dispatches Git after any file change)
4. Completeness criterion says "git committed" unconditionally, contradicting conditional git dispatch

- [ ] **Step 1: Restructure to add `## Rules` section**

Replace the entire `## Routing Flow` through `## Agent Routing Rules` block with a proper `## Rules` section. The routing flow content becomes rules:

```markdown
## Rules

### Routing Sequence
Every task follows this four-step sequence:

1. **Pre-scope:** Dispatch Coordinator in pre-scope mode with the user's task description. Coordinator returns: `Scope: [Trivial|Standard|Complex] -- <reason>`
2. **Classify and route:** Read the user's task and classify:
   - **New idea / ambiguous** -- dispatch to Office Hours first, then Planner
   - **Non-trivial but clear** -- dispatch to Planner, then execute
   - **Small / scoped** -- dispatch directly to the appropriate agent
3. **Post-verify:** After the working agent reports done, dispatch Coordinator in post-verify mode with the user's original request, the working agent's completion report, and the pre-scope line. Post-verify is not optional.
4. **Git (conditional):** Dispatch Git agent only when the user asked for git operations, or when the task explicitly requires a commit. Do not auto-commit after every file change.

When dispatching the working agent, include:
- The user's original words (not a summary)
- The Coordinator's pre-scope line verbatim

### Agent Pipelines

For complex tasks, chain agents in sequence. Each agent's output feeds the next.

| Pipeline | Sequence | When to Use |
|---|---|---|
| **Idea to Ship** | Office Hours -> Planner -> Coder -> Tester -> Coordinator | New product ideas or features |
| **Debug** | Investigate -> Coder -> Tester -> Coordinator | Bug reports and failures |
| **Security Review** | CSO -> Coder (fixes) -> CSO (re-verify) -> Coordinator | Before deploy or on security concerns |
| **Retro** | Retro -> (optionally) Planner for action items | End of sprint or milestone |
| **Standard** | Planner -> Coder -> Tester -> Coordinator | Non-trivial clear tasks |
| **Research** | Researcher -> (handoff as needed) -> Coordinator | Information gathering, documentation questions |

Execute pipelines step by step -- each agent must complete and pass evaluation before the next starts. If any agent fails evaluation after retry, halt and escalate to user.

### Agent Routing Table
1. Check for project-specific agents in `./agents/` before defaulting to global agents in `~/.claude/agents/`
2. When a task has independent parts, launch multiple agents in parallel (all receive the same pre-scope line)
3. Route trading tasks (trading, backtest, Alpaca, stock, portfolio, watchlist, strategy, buy, sell, market) to Trader (`~/.claude/agents/trader.md`)
4. Route config tasks (settings.json, permissions, hooks, env vars, MCP servers, allow/deny) to Config (`~/.claude/agents/config.md`)
5. Route agent review/audit tasks (audit agents, optimize agent files, review agent definitions, score agents) to Auditor (`~/.claude/agents/auditor.md`)
6. Route git tasks (commit, push, pull, branch, merge, rebase, PR, clone, git status) to Git (`~/.claude/agents/git.md`)
7. Route logging tasks (log progress, log session, log today, summarize today, daily summary, dev-journal) to Logger (`~/.claude/agents/logger.md`)
8. Route new ideas, brainstorms, "is this worth building" to Office Hours (`~/.claude/agents/office-hours.md`)
9. Route security audits, vulnerability checks, "is this secure" to CSO (`~/.claude/agents/cso.md`)
10. Route debugging, "why is this broken", test failures, unexpected behavior to Investigate (`~/.claude/agents/investigate.md`)
11. Route retrospectives, "what did I accomplish", weekly summaries to Retro (`~/.claude/agents/retro.md`)
12. Route research, exploration, "how does X work", documentation lookups to Researcher (`~/.claude/agents/researcher.md`)
13. Route "create a new agent", recurring task pattern with no matching agent to Agent Factory (`~/.claude/agents/agent-factory.md`)
14. Route "clean up agent files", agent self-update proposals, `.md` file maintenance to MD Keeper (`~/.claude/agents/md-keeper.md`)
```

- [ ] **Step 2: Fix Completeness criterion in Evaluation Criteria table**

Replace:
```markdown
| **Completeness** | Full pipeline executed, pre-scope + post-verify both ran, git committed | Missing a pipeline step | Skipped pre-scope or post-verify |
```
with:
```markdown
| **Completeness** | Full pipeline executed, pre-scope + post-verify both ran, git dispatched if user requested | Missing a pipeline step | Skipped pre-scope or post-verify |
```

- [ ] **Step 3: Fix Validation item 7 to match conditional git**

Replace:
```markdown
7. Git agent was dispatched after successful post-verify (if file changes exist)
```
with:
```markdown
7. Git agent was dispatched only when user requested git operations or task explicitly requires a commit
```

- [ ] **Step 4: Read the full file and verify all six template sections are present**

Confirm: Role, Trigger, Rules, Evaluation Criteria, Validation, Learning all exist as `##` headings.

- [ ] **Step 5: Commit**

```bash
git add ~/.claude/agents/dispatcher.md
git commit -m "fix(dispatcher): add Rules section, missing routes, conditional git dispatch"
```

---

### Task 3: Fix `coordinator.md` -- Structural Conformance, Dead Rules, Contradictions

**Files:**
- Modify: `~/.claude/agents/coordinator.md`

**Problems:**
1. Mode sections appear before `## Rules` -- template expects Rules right after Trigger
2. `git diff --name-only` as primary source of truth is unsafe in dirty worktrees
3. "word-for-word" is a dead rule
4. "not more, not fewer" contradicts Complex including Standard

- [ ] **Step 1: Restructure to put Rules before modes**

The file currently has the structure: Role -> Trigger -> Mode 1: Pre-Scope -> Mode 2: Post-Verify (with Rules nested inside) -> Evaluation Criteria -> Validation -> Learning.

Restructure to: Role -> Trigger -> Rules (top-level, containing the mode descriptions) -> Evaluation Criteria -> Validation -> Learning.

Replace the `---` separator after Trigger (line 10) and the `## Mode 1: Pre-Scope` heading (line 12) with:

```markdown
## Rules

### Mode 1: Pre-Scope
```

Replace `## Mode 2: Post-Verify` (line 47) with:

```markdown
### Mode 2: Post-Verify
```

Remove the nested `### Rules` heading (line 96) entirely -- the rules under it become part of the top-level Rules section, as sub-items of Mode 2. Replace `### Rules` with:

```markdown
### Post-Verify Checks
```

Remove the three `---` horizontal rule separators (lines 10, 44, 126) as they are not part of the template structure.

- [ ] **Step 2: Fix git diff unsafe assumption**

Replace line 55:
```markdown
**Step 1:** Identify files changed. Always run `git diff --name-only` to determine what actually changed — this is the primary source of truth. Compare the result against the subagent's claimed changes; flag any discrepancies. Do not trust the subagent's report alone.
```
with:
```markdown
**Step 1:** Identify files changed. Run `git diff --name-only` against the baseline (use `git status --porcelain` first to check for a clean worktree — empty output means clean). Compare the result against the subagent's claimed changes; flag any discrepancies. If the worktree is dirty from unrelated changes, scope the diff to the subagent's claimed files only. Do not trust the subagent's report alone.
```

- [ ] **Step 3: Fix "word-for-word" dead rule**

Replace line 99:
```markdown
1. Compare the result against the user's original request word-for-word -- did it do what was asked?
```
with:
```markdown
1. Compare the result against the user's stated requirements and constraints -- did it do what was asked?
```

- [ ] **Step 4: Fix "not more, not fewer" contradiction**

Replace line 144:
```markdown
5. **Post-verify mode:** Ran the review steps appropriate to the actual tier (not more, not fewer)
```
with:
```markdown
5. **Post-verify mode:** Ran at least the review steps appropriate to the actual tier (Complex includes Standard checks)
```

- [ ] **Step 5: Read the full file and verify structural conformance**

Confirm: Role, Trigger, Rules (as top-level `##`), Evaluation Criteria, Validation, Learning all present. No `## Mode` headings at the same level as Rules.

- [ ] **Step 6: Commit**

```bash
git add ~/.claude/agents/coordinator.md
git commit -m "fix(coordinator): restructure for template conformance, fix dead rules and contradictions"
```

---

## P1: Safety-Critical Defaults

### Task 4: Fix `git.md` -- Clone Logic, Auto-PR/Merge, Dead Rules, Duplicate Numbering

**Files:**
- Modify: `~/.claude/agents/git.md`

**Problems:**
1. "Always clone first" is wrong inside existing repos
2. "After pushing any branch, always create a PR" is overbroad
3. Auto-merge for personal repos is too aggressive as a default
4. `Claude Opus 4.6` trailer is a hardcoded dead rule
5. Duplicate rule numbering (two rule 10s)

- [ ] **Step 1: Fix rule 1 (clone logic)**

Replace:
```markdown
1. Always clone first if a GitHub repo exists — never `git init` alongside an existing remote
```
with:
```markdown
1. If the repo is not present locally, clone it — never `git init` alongside an existing remote. If already inside a repo, operate in the existing worktree.
```

- [ ] **Step 2: Fix rule 6 (hardcoded trailer)**

Replace:
```markdown
6. Include `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>` trailer in every commit
```
with:
```markdown
6. Include `Co-Authored-By` trailer with current model name in every commit
```

- [ ] **Step 3: Fix rule 8 (auto-PR)**

Replace:
```markdown
8. After pushing any branch, always create a PR using `gh pr create`
```
with:
```markdown
8. After pushing a non-default branch to a GitHub-hosted remote, create a PR using `gh pr create`. Skip for local-only repos or direct-to-default workflows.
```

- [ ] **Step 4: Fix rule 9 (auto-merge)**

Replace:
```markdown
9. After creating a PR, check if the repo is a personal repo (single owner, no required reviewers). If so, auto-merge with `gh pr merge --merge --delete-branch`. If not personal or has branch protections, leave the PR open for review.
```
with:
```markdown
9. After creating a PR, leave it open for review by default. Only auto-merge if the user explicitly requests it.
```

- [ ] **Step 5: Fix duplicate rule numbering**

The file has two rules numbered `10`. Renumber from rule 10 onward:
```markdown
10. Never force push without explicit user confirmation — ask first, explain the risk
11. Clean up merged branches: delete local and remote branches after merge is confirmed
12. When resolving merge conflicts, show the conflict to the user and explain the resolution before applying
13. Use `gh` CLI for all GitHub operations (PRs, repo creation, issues, releases)
```

- [ ] **Step 6: Update validation item 4 to match new PR rule**

Replace:
```markdown
4. PR exists on GitHub for every pushed non-main branch
```
with:
```markdown
4. PR exists on GitHub for pushed non-default branches on GitHub-hosted remotes (skipped for local-only repos)
```

- [ ] **Step 7: Commit**

```bash
git add ~/.claude/agents/git.md
git commit -m "fix(git): safer clone logic, conditional PR/merge, fix dead rules and numbering"
```

---

### Task 5: Fix `trader.md` -- Missing Tester Handoff, Safety Safeguards, Validation Gaps

**Files:**
- Modify: `~/.claude/agents/trader.md`

**Problems:**
1. No explicit Tester handoff after code changes
2. No position sizing, duplicate-order prevention, or market-hours handling rules
3. Validation misses rules 5 and 7

- [ ] **Step 1: Add Tester handoff rule after rule 3**

After rule 3, insert new rules (renumber subsequent rules):
```markdown
4. After Coder delivers code, dispatch Tester to verify trading logic with unit tests and a paper trade dry run
5. Enforce position sizing limits: never open a position larger than the max size defined in `Trading/docs/risk-rules.md`. If no limit is defined, halt and ask the user for an explicit position sizing limit before proceeding.
6. Before submitting any order, check for existing open orders on the same symbol to prevent duplicates
7. Check market hours before placing orders -- for equities, warn if outside regular trading hours (9:30-16:00 ET) unless the user explicitly requests extended hours
```

Renumber old rules 4-8 to 8-12:
```markdown
8. Enforce all rules in `Trading/docs/risk-rules.md` — refuse to proceed if a proposed action violates any active rule
9. Before executing the first trade in any session, confirm with the user (safety gate)
10. Ensure all trades, signals, and activity are logged to `Trading/reports/`
11. Support three execution modes based on user request:
    - **On-demand:** Check specific symbol(s) and trade if signal warrants
    - **Scheduled:** Build/update a runner script that checks watchlist on interval
    - **Continuous:** Build a streaming monitor for a defined period
12. Paper trading only — refuse any attempt to switch to live trading unless the user explicitly changes config
```

- [ ] **Step 2: Add missing validation items**

Add after existing validation item 5:
```markdown
6. User confirmation was captured before the first trade in the session (rule 9)
7. Execution mode was explicitly selected (on-demand, scheduled, or continuous) (rule 11)
8. No duplicate orders were submitted for the same symbol (rule 6)
9. Position sizing limits were checked before order submission (rule 5)
10. Tester was dispatched after code changes to verify trading logic (rule 4)
```

Renumber the existing final validation item (evaluation criteria) accordingly.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/agents/trader.md
git commit -m "fix(trader): add Tester handoff, position sizing, duplicate-order prevention, market hours"
```

---

## P2: Agent Effectiveness

### Task 6: Fix `coder.md` -- Skill References, Plan Adherence, Weak Validation

**Files:**
- Modify: `~/.claude/agents/coder.md`

**Problems:**
1. "TDD skill" and "systematic-debugging skill" are undefined, no fallback
2. "follow the plan steps exactly" blocks sensible adaptation
3. Validation too weak -- "code runs without syntax errors" doesn't catch broken behavior

- [ ] **Step 1: Fix rule 3 (TDD skill reference)**

Replace:
```markdown
3. Use TDD skill when writing new functionality: write failing test first, then implement
```
with:
```markdown
3. Use `superpowers:test-driven-development` skill when writing new functionality. If unavailable, follow TDD manually: write a failing test first, then implement the minimal code to pass it.
```

- [ ] **Step 2: Fix rule 4 (debugging skill reference)**

Replace:
```markdown
4. Use systematic-debugging skill when fixing bugs: reproduce first, then fix
```
with:
```markdown
4. Use `superpowers:systematic-debugging` skill when fixing bugs. If unavailable, reproduce the bug first with exact error output, then fix.
```

- [ ] **Step 3: Fix rule 7 (plan adherence)**

Replace:
```markdown
7. If the task came with a plan, follow the plan steps exactly
```
with:
```markdown
7. If the task came with a plan, follow the plan steps unless direct repo evidence requires a deviation. Report any deviations explicitly in the completion report.
```

- [ ] **Step 4: Strengthen validation item 1**

Replace:
```markdown
1. Code runs without syntax errors (execute or lint it)
```
with:
```markdown
1. Code runs without errors -- execute it or run the project's linter/formatter if configured
```

- [ ] **Step 5: Add validation item for formatter/linter**

Add after validation item 2:
```markdown
3. If the project has a configured formatter or linter, it was run and output is clean
4. Any plan deviations were reported in the completion report (rule 7)
```

Renumber subsequent items (old 3 becomes 5, old 4 becomes 6, old 5 becomes 7).

- [ ] **Step 6: Commit**

```bash
git add ~/.claude/agents/coder.md
git commit -m "fix(coder): name exact skills with fallbacks, flexible plan adherence, stronger validation"
```

---

### Task 7: Fix `tester.md` -- Coder Coordination, Traceback Rule, Validation Gaps

**Files:**
- Modify: `~/.claude/agents/tester.md`

**Problems:**
1. "If no tests exist, write them" pushes into code-authoring without Coder coordination
2. "full traceback" is too absolute for huge outputs
3. Validation doesn't cover rules 5 or 6

- [ ] **Step 1: Fix rule 3 (test writing scope)**

Replace:
```markdown
3. If no tests exist for the changed code, write them
```
with:
```markdown
3. If no tests exist for the changed code, write them. If the task is verification-only (not implementation), hand back a concrete missing-test spec to Coder instead of writing implementation-coupled tests.
```

- [ ] **Step 2: Fix rule 4 (traceback)**

Replace:
```markdown
4. Report exact error output, not summaries — include the full traceback or failure message
```
with:
```markdown
4. Report exact error output, not summaries — include the failing excerpt sufficient to diagnose (truncate very large tracebacks to the relevant frames)
```

- [ ] **Step 3: Add missing validation items**

Add after validation item 4:
```markdown
5. At least one edge case or failure case was tested (rule 5)
6. No mocks were used on the thing being tested -- mocks are for external dependencies only (rule 6)
```

Renumber old item 5 (evaluation criteria) to 7.

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/tester.md
git commit -m "fix(tester): clarify test-writing scope, truncate large tracebacks, add missing validation"
```

---

### Task 8: Fix `planner.md` -- Plan Mode Fallback, Codebase Reading, Minimal Plan Definition

**Files:**
- Modify: `~/.claude/agents/planner.md`

**Problems:**
1. No fallback if EnterPlanMode/ExitPlanMode are unavailable
2. No rule to read codebase before naming files
3. "Minimal plan" is vague

- [ ] **Step 1: Fix rule 1 (Plan Mode fallback)**

Replace:
```markdown
1. Call `EnterPlanMode` as your first action — this enforces read-only mode and prevents accidental code writes during planning
```
with:
```markdown
1. Call `EnterPlanMode` as your first action to enforce read-only mode. If Plan Mode tools are unavailable, proceed in read-only discipline and state that explicitly in your report.
```

- [ ] **Step 2: Add codebase reading rule after rule 4**

Insert new rule 5 (renumber subsequent rules):
```markdown
5. Before naming exact files or functions in the plan, read the codebase to verify they exist (or confirm the directory where new files should go). Do not hallucinate paths.
```

Old rules 5-9 become 6-10.

- [ ] **Step 3: Fix old rule 9 (now rule 10) -- define minimal plan**

Replace:
```markdown
9. If brainstorming reveals the task is small (single file, obvious change), produce a minimal plan
```
with:
```markdown
10. If brainstorming reveals the task is small (single file, obvious change), produce a minimal plan: at minimum, list the file(s) to change, the steps to take, and one test check to verify the change works.
```

- [ ] **Step 4: Fix rule 8 (now 9) for Plan Mode fallback**

Replace:
```markdown
8. After the plan is complete, call `ExitPlanMode` to restore write capability, then hand off to Dispatcher for execution routing
```
with:
```markdown
9. After the plan is complete, call `ExitPlanMode` to restore write capability (skip if Plan Mode was unavailable), then hand off to Dispatcher for execution routing
```

- [ ] **Step 5: Update validation item 1 for fallback**

Replace:
```markdown
1. `EnterPlanMode` was called as the first action
```
with:
```markdown
1. `EnterPlanMode` was called as the first action (or read-only discipline was stated if Plan Mode unavailable)
```

- [ ] **Step 6: Update validation item 6 for fallback**

Replace:
```markdown
6. `ExitPlanMode` was called after plan completion, before handoff
```
with:
```markdown
6. `ExitPlanMode` was called after plan completion (or skipped if Plan Mode was unavailable)
```

- [ ] **Step 7: Add validation item for codebase reading**

Add after validation item 6:
```markdown
7. File and function names in the plan were verified against the codebase (rule 5)
```

- [ ] **Step 8: Commit**

```bash
git add ~/.claude/agents/planner.md
git commit -m "fix(planner): add Plan Mode fallback, require codebase reading, define minimal plan"
```

---

### Task 9: Fix `cso.md` -- Severity Rubric, Grep Patterns, Coverage Validation

**Files:**
- Modify: `~/.claude/agents/cso.md`

**Problems:**
1. No severity rubric for Critical/High/Medium/Low
2. "use grep patterns" for secrets is too weak
3. Validation doesn't require OWASP/STRIDE coverage proof

- [ ] **Step 1: Add severity rubric after rule 4**

Insert new rule 5 (renumber subsequent rules):
```markdown
5. Severity rubric:
   - **Critical:** Exploitable remotely without authentication, leads to full system compromise or data breach
   - **High:** Exploitable with low-privilege access, leads to significant data exposure or privilege escalation
   - **Medium:** Requires specific conditions to exploit, limited impact
   - **Low:** Informational, defense-in-depth improvement, no direct exploit path
```

Old rules 5-7 become 6-8.

- [ ] **Step 2: Fix old rule 6 (now 7) -- secrets scanning**

Replace:
```markdown
6. Check for secrets in code: API keys, passwords, tokens, connection strings — use grep patterns
```
with:
```markdown
7. Check for secrets in code: run at least one broad pattern scan (e.g., `rg -i "(api_key|secret|password|token|credential)"`) plus one targeted scan based on project conventions (e.g., `.env` files, config files with connection strings)
```

- [ ] **Step 3: Add OWASP/STRIDE coverage validation**

Replace validation item 2:
```markdown
2. STRIDE model was applied to at least the main components
```
with:
```markdown
2. STRIDE model was applied to at least the main components -- report includes a checklist showing each STRIDE category as "checked/no issue" or "finding: <description>"
```

Add after validation item 1:
```markdown
2. Report includes a checklist showing each OWASP category as "checked/no issue" or "finding: <description>"
```

Renumber accordingly (old 2 becomes 3, old 3-6 become 4-7).

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/cso.md
git commit -m "fix(cso): add severity rubric, improve secrets scanning, require coverage checklists"
```

---

### Task 10: Fix `investigate.md` -- Flexible Iteration Limit, Unreproducible Bug Rule

**Files:**
- Modify: `~/.claude/agents/investigate.md`

**Problems:**
1. "Max 5 iterations" is too rigid
2. No rule for unreproducible bugs

- [ ] **Step 1: Fix iteration limit in rule 2**

Replace:
```markdown
   - Repeat until root cause is found — max 5 iterations before escalating to user
```
with:
```markdown
   - Repeat until root cause is found — escalate after 5 iterations if no new evidence is being produced. Continue past 5 if each iteration yields new evidence.
```

- [ ] **Step 2: Add unreproducible bug rule after rule 1**

Insert new rule 2 (renumber subsequent rules):
```markdown
2. If the bug cannot be reproduced locally, document the missing inputs, environment, or conditions needed for reproduction. Collect whatever evidence is available (logs, screenshots, error reports) and escalate with that context rather than guessing at fixes.
```

Old rules 2-7 become 3-8.

- [ ] **Step 3: Update existing validation item 1 to account for unreproducible bugs**

Replace:
```markdown
1. Bug was reproduced with exact error output captured
```
with:
```markdown
1. Bug was reproduced with exact error output captured, or if unreproducible, missing reproduction conditions were documented (rule 2)
```

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/investigate.md
git commit -m "fix(investigate): flexible iteration limit, add unreproducible bug handling"
```

---

### Task 11: Fix `office-hours.md` -- Allow "Don't Proceed", Fix Question Count, Measurable Brief

**Files:**
- Modify: `~/.claude/agents/office-hours.md`

**Problems:**
1. Always hands off to Planner -- no "don't proceed" option
2. Rule says "six questions" but validation says "at least 4 of 6" -- contradiction
3. "One-page design brief" is not measurable

- [ ] **Step 1: Fix rule 5 (allow don't-proceed outcome)**

Replace:
```markdown
5. Hand off the design brief to Planner for implementation planning
```
with:
```markdown
5. If the idea survives questioning, hand off the design brief to Planner for implementation planning. If questioning reveals the idea is not worth building (no demand, no insight, or better solved by existing tools), recommend not proceeding and explain why.
```

- [ ] **Step 2: Fix rule 2 (question requirement)**

Replace:
```markdown
2. Ask these six forcing questions (adapt to context, skip if already answered):
```
with:
```markdown
2. Address all six forcing questions below (ask only those not already answered by the user):
```

- [ ] **Step 3: Fix validation item 1 to match rule 2**

Replace:
```markdown
1. At least 4 of 6 forcing questions were asked (some may be pre-answered)
```
with:
```markdown
1. All six forcing questions were addressed (asked or confirmed as pre-answered)
```

- [ ] **Step 4: Fix rule 4 (measurable brief)**

Replace:
```markdown
4. After questioning, produce a one-page design brief:
```
with:
```markdown
4. After questioning, produce a design brief with these required sections:
```

- [ ] **Step 5: Add validation for don't-proceed outcome**

Add after validation item 4:
```markdown
5. If recommending not to proceed, explanation includes which forcing questions revealed the issue
```

Renumber old item 5 (evaluation criteria) to 6.

- [ ] **Step 6: Update existing validation item 4 for conditional handoff**

Replace:
```markdown
4. Brief was handed off to Planner (or back to Dispatcher for routing)
```
with:
```markdown
4. Brief was handed off to Planner, or don't-proceed recommendation was delivered with reasoning citing specific forcing questions (rule 5)
```

- [ ] **Step 7: Commit**

```bash
git add ~/.claude/agents/office-hours.md
git commit -m "fix(office-hours): allow don't-proceed, fix question count, measurable brief"
```

---

## P3: Polish & Consistency

### Task 12: Fix `agent-factory.md` -- Name Exact Skill, Define Approval Format

**Files:**
- Modify: `~/.claude/agents/agent-factory.md`

**Problems:**
1. "Use the brainstorming skill" doesn't name the exact skill
2. MD Keeper approval has no defined format

- [ ] **Step 1: Fix rule 2 (name exact skill)**

Replace:
```markdown
2. Use the brainstorming skill to clarify what the new agent should do before writing it
```
with:
```markdown
2. Use `superpowers:brainstorming` skill to clarify what the new agent should do before writing it. If unavailable, ask the three questions in rule 3 directly.
```

- [ ] **Step 2: Fix rule 5 (define approval format)**

Replace:
```markdown
5. After creating the agent, invoke MD Keeper to review the new file
```
with:
```markdown
5. After creating the agent, invoke MD Keeper to review the new file. MD Keeper must return explicit `Approved` or a findings list with required changes.
```

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/agents/agent-factory.md
git commit -m "fix(agent-factory): name exact brainstorming skill, define MD Keeper approval format"
```

---

### Task 13: Fix `auditor.md` -- Scope Default, Score Mapping, Dead Rule

**Files:**
- Modify: `~/.claude/agents/auditor.md`

**Problems:**
1. "Read all files" is too broad for targeted audits
2. 1-5 scale doesn't map to Pass/Needs Work/Fail
3. Rule 8 is dead for review-only runs

- [ ] **Step 1: Fix rule 1 (scope)**

Replace:
```markdown
1. Read all files in `~/.claude/agents/` and `./agents/` (if present) before producing any findings
```
with:
```markdown
1. Read the files in scope: if the task targets specific agents, read only those. If the task says "all agents" or does not specify, read all files in `~/.claude/agents/` and `./agents/` (if present).
```

- [ ] **Step 2: Add score mapping to rule 2**

After the last criterion in rule 2 (Measurability), add:
```markdown
   Score mapping: 5 = Pass, 3-4 = Needs Work, 1-2 = Fail
```

- [ ] **Step 3: Fix rule 8 (conditional editing)**

Replace:
```markdown
8. When asked to apply fixes, edit files directly — do not just report
```
with:
```markdown
8. When asked to apply fixes (not just review), edit files directly — do not just report
```

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/auditor.md
git commit -m "fix(auditor): scope-aware reading, add score mapping, clarify edit rule"
```

---

### Task 14: Fix `config.md` -- Schema Drift, Array Rule, Missing Validation

**Files:**
- Modify: `~/.claude/agents/config.md`

**Problems:**
1. Hardcoded schema will drift
2. "Never replace arrays" too absolute
3. Missing validation for unrecognized keys

- [ ] **Step 1: Fix rule 6 (schema drift)**

Replace the opening of rule 6:
```markdown
6. Know the full settings schema and only write recognized keys:
```
with:
```markdown
6. Use the current documented schema and only write recognized keys. Reject unknown keys unless the user explicitly asks for experimental ones. Current known keys:
```

- [ ] **Step 2: Fix rule 3 (array replacement)**

Replace:
```markdown
3. Never replace arrays — always merge with existing entries (permissions.allow, permissions.deny, hooks, etc.)
```
with:
```markdown
3. Do not replace arrays unless the user explicitly requests replacement — default to merging with existing entries (permissions.allow, permissions.deny, hooks, etc.)
```

- [ ] **Step 3: Add validation for unrecognized keys**

Add after validation item 4:
```markdown
5. No unrecognized top-level keys were introduced (rule 6)
```

Renumber old items 5-6 to 6-7.

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/config.md
git commit -m "fix(config): handle schema drift, conditional array replacement, validate keys"
```

---

### Task 15: Fix `logger.md` -- Time Conflict, Hardcoded Branch, Validation Gaps

**Files:**
- Modify: `~/.claude/agents/logger.md`

**Problems:**
1. "Last 8 hours" conflicts with triggers like "log today"
2. Hardcoded `origin/master`
3. Validation doesn't check rules 8 or 9

- [ ] **Step 1: Fix rule 3 (time range)**

Replace:
```markdown
3. For session entries: scan git activity from the last 8 hours across all repos listed in CLAUDE.md using `git log --oneline --since="8 hours ago"` and `git diff --stat`
```
with:
```markdown
3. For session entries: scan git activity across all repos listed in CLAUDE.md. Use the user's stated time range first ("today", "this session", explicit range). Default to last 8 hours only if no range is specified.
```

- [ ] **Step 2: Fix rule 7 (hardcoded branch)**

Replace:
```markdown
7. Always commit and push after writing entries using the commit message format specified in CLAUDE.md
```

This rule itself doesn't mention origin/master -- check validation item 4:

Replace:
```markdown
4. Git commit was created and pushed to origin/master
```
with:
```markdown
4. Git commit was created and pushed to the repo's default branch
```

- [ ] **Step 3: Add missing validation items**

Add after validation item 5:
```markdown
6. If no meaningful work was done, entry explicitly states so (rule 8)
7. Tags are lowercase and hyphenated (rule 9)
8. Time range used matches user's stated range or defaults to 8 hours (rule 3)
```

Renumber old item 6 (evaluation criteria) to 9.

- [ ] **Step 4: Commit**

```bash
git add ~/.claude/agents/logger.md
git commit -m "fix(logger): flexible time range, default branch, add missing validation"
```

---

### Task 16: Fix `md-keeper.md` -- Absolute Rule, Missing Redundancy Validation

**Files:**
- Modify: `~/.claude/agents/md-keeper.md`

**Problems:**
1. "Every line must change behavior" conflicts with template prose
2. Validation never checks redundancy or CLAUDE.md review

- [ ] **Step 1: Fix rule 1**

Replace:
```markdown
1. Every line in an agent `.md` file must change behavior — if removing it changes nothing, remove it
```
with:
```markdown
1. Every rule line in an agent `.md` file must change behavior — if removing it changes nothing, remove it. Explanatory template text (section headings, evaluation table) is exempt.
```

- [ ] **Step 2: Add redundancy validation items**

Add after validation item 3:
```markdown
4. Cross-agent redundancy was checked -- duplicated rules flagged for potential move to CLAUDE.md (rule 3)
5. If a CLAUDE.md review was in scope, it was reviewed with the same clarity standards (rule 7)
```

Renumber old item 4 (evaluation criteria) to 6.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/agents/md-keeper.md
git commit -m "fix(md-keeper): exempt template prose from behavior rule, add redundancy validation"
```

---

### Task 17: Fix `researcher.md` -- Specific Grep, Source Quality, Better Validation

**Files:**
- Modify: `~/.claude/agents/researcher.md`

**Problems:**
1. Generic "grep" -- should specify tool
2. No source quality rule
3. Thin validation -- no granularity on source references

- [ ] **Step 1: Fix rule 1 (specify tool)**

Replace:
```markdown
1. Start with the most specific search — grep for exact names before doing broad exploration
```
with:
```markdown
1. Start with the most specific search — use the Grep tool (or `rg` if in Bash) for exact names before doing broad exploration
```

- [ ] **Step 2: Add source quality rule after rule 3**

Insert new rule 4 (renumber subsequent rules):
```markdown
4. Prefer primary and official sources for technical claims — official docs over blog posts, source code over third-party summaries
```

Old rules 4-5 become 5-6.

- [ ] **Step 3: Strengthen validation item 1**

Replace:
```markdown
1. Findings include source references (file paths, URLs, line numbers)
```
with:
```markdown
1. Findings include specific source references: file paths with line numbers for code, URLs with page titles for web sources
```

- [ ] **Step 4: Add validation item for source quality**

Add after validation item 1:
```markdown
2. Primary/official sources were preferred over secondary sources for technical claims (rule 4)
```

Renumber subsequent items.

- [ ] **Step 5: Commit**

```bash
git add ~/.claude/agents/researcher.md
git commit -m "fix(researcher): specify search tool, add source quality rule, stronger validation"
```

---

### Task 18: Fix `retro.md` -- Define HEAD~N, Validate Action Items

**Files:**
- Modify: `~/.claude/agents/retro.md`

**Problems:**
1. `HEAD~<N>` undefined
2. Validation doesn't check action item quality

- [ ] **Step 1: Fix rule 2 diff stats**

Replace:
```markdown
   - **Diff stats:** `git diff --stat HEAD~<N>` or `git log --stat --since="<period>"` for LOC metrics
```
with:
```markdown
   - **Diff stats:** `git log --stat --since="<period>"` for LOC metrics
```

(Remove the ambiguous `HEAD~<N>` form entirely -- the `--since` form is sufficient and unambiguous.)

- [ ] **Step 2: Add action item validation**

Add after validation item 6:
```markdown
7. Action items are max 3, each is specific and actionable (not vague like "do better")
```

Renumber old item 7 (evaluation criteria) to 8.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/agents/retro.md
git commit -m "fix(retro): remove ambiguous HEAD~N, validate action item quality"
```

---

## Cross-Cutting Verification: Task 19

After all individual file fixes are applied, run this verification pass across all files.

**Files:**
- Read all: `~/.claude/agents/*.md`

- [ ] **Step 1: Template conformance check**

Read every agent `.md` file. Verify each has these sections as `##` headings in order:
1. `## Role`
2. `## Trigger`
3. `## Rules`
4. `## Evaluation Criteria`
5. `## Validation`
6. `## Learning`

Flag any file that deviates.

- [ ] **Step 2: Rule-validation mapping check**

For each agent file, enumerate all rules and all validation items. Verify:
- Every rule is covered by at least one validation item
- No validation item references a nonexistent rule

Flag gaps.

- [ ] **Step 3: Skill reference check**

Search all files for skill references (patterns like `skill`, `EnterPlanMode`, `ExitPlanMode`). Verify each reference either names the exact skill path or has a fallback clause.

- [ ] **Step 4: Dispatcher routing completeness check**

Read `dispatcher.md` routing table. Compare against the list of all agent files. Verify every agent file has a routing rule.

- [ ] **Step 5: Dead rule scan**

Search for known dead-rule patterns:
- "word-for-word"
- "every line must" (without exemption)
- Hardcoded branch names (`origin/master`, `main`)
- Hardcoded model names (`Claude Opus 4.6`)
- "always" / "never" without exception clause (review for overly absolute phrasing)

- [ ] **Step 6: Evaluation criteria threshold check**

For each agent's Evaluation Criteria table, verify that Pass/Needs Work/Fail definitions contain measurable or observable criteria (not purely subjective language).

- [ ] **Step 7: Report findings**

Produce a summary table:

| Check | Status | Issues Found |
|-------|--------|--------------|
| Template conformance | Pass/Fail | ... |
| Rule-validation mapping | Pass/Fail | ... |
| Skill references | Pass/Fail | ... |
| Routing completeness | Pass/Fail | ... |
| Dead rules | Pass/Fail | ... |
| Evaluation thresholds | Pass/Fail | ... |

---

## Test Strategy

### How to verify fixes work

Each fix category needs a different verification approach:

**1. Structural conformance (Tasks 1-3)**
- Parse each `.md` file for `##` headings. Assert the order matches: Role, Trigger, Rules, Evaluation Criteria, Validation, Learning.
- Automated: `grep "^## " ~/.claude/agents/*.md` and verify ordering per file.

**2. Routing completeness (Task 2)**
- List all `.md` files in `~/.claude/agents/` (excluding `_template.md`).
- For each file, search `dispatcher.md` for a routing rule referencing that agent. Exclude pipeline-only agents (Coordinator, Planner, Coder, Tester) that are invoked within pipelines, not directly routed.
- Expect: every agent file has at least one route.

**3. Rule-validation coverage (Tasks 1-18)**
- For each agent file, count rules and count validation items.
- Manually verify each rule maps to at least one validation item.
- This is the most important check and must be done per-file.

**4. Safety defaults (Tasks 4-5)**
- Read `git.md` rules 8-9 and verify they are conditional (not "always").
- Read `trader.md` and verify position sizing, duplicate-order, and market-hours rules exist.
- Read `dispatcher.md` step 4 and verify git dispatch is conditional.

**5. Skill reference resolution (Tasks 6, 8, 12)**
- Search all files for skill references.
- Each must either name the exact skill path (e.g., `superpowers:brainstorming`) or include a fallback clause.

**6. Dead rule elimination (Tasks 3, 4, 15, 16)**
- Search for the specific strings identified as dead: "word-for-word", "Claude Opus 4.6", "origin/master", "every line in an agent .md file must change behavior".
- Expect: none found (or replaced with corrected versions).

**7. Cross-cutting verification (Task 19)**
- Run Task 19 as the final validation gate. All six checks must pass.

---

## Execution Order

```
Tier P0 (do first -- everything depends on these):
  Task 1: _template.md
  Task 2: dispatcher.md
  Task 3: coordinator.md
  → Run Task 19 Steps 1-2 (template conformance + orphan rule check) as intermediate gate

Tier P1 (do second -- safety critical):
  Task 4: git.md          (parallel with Task 5)
  Task 5: trader.md       (parallel with Task 4)
  Task 8: planner.md      (after Tasks 4-5 complete)
  → Run Task 19 Steps 1-2 (template conformance + orphan rule check) as intermediate gate

Tier P2 (do third -- effectiveness):
  Task 6: coder.md        \
  Task 7: tester.md        |
  Task 9: cso.md           |  all parallel
  Task 10: investigate.md  |
  Task 11: office-hours.md /
  → Run Task 19 Steps 1-2 (template conformance + orphan rule check) as intermediate gate

Tier P3 (do fourth -- polish):
  Task 12: agent-factory.md \
  Task 13: auditor.md        |
  Task 14: config.md         |
  Task 15: logger.md         |  all parallel
  Task 16: md-keeper.md      |
  Task 17: researcher.md     |
  Task 18: retro.md         /
  → Run Task 19 Steps 1-2 (template conformance + orphan rule check) as intermediate gate

Final gate:
  Task 19: Cross-cutting verification
```

Total: 19 tasks across 18 files + 1 verification pass.
