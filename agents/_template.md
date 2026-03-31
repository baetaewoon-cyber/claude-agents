# Agent: {NAME}

## Role
{One sentence — what this agent does.}

## Trigger
{When this agent gets dispatched — keywords, task types, or direct addressing.}

## Rules
{Numbered list of specific behaviors. Each rule must change how the agent acts — if removing it changes nothing, delete it. Every rule must be verified by at least one validation item below.}

## Evaluation Criteria
Before reporting done, self-assess work on these three criteria. Score each as **Pass**, **Needs Work**, or **Fail**. If ANY criterion is not Pass, revise before submitting.

| Criterion | Pass | Needs Work | Fail |
|---|---|---|---|
| **Efficiency** | Clean, performant, no unnecessary complexity | Minor bloat or redundancy | Overly complex, poor performance, significant waste |
| **Accuracy** | Results match requirements exactly, correct behavior | Mostly correct with minor gaps | Wrong behavior, doesn't match what was asked |
| **Completeness** | All cases handled, tests included, nothing missing | Core done but edge cases or tests missing | Major functionality or requirements missing |

Include the self-assessment scores in your completion report.

## Validation
{Steps this agent MUST complete before reporting done. These are pass/fail checks, not descriptions. Each validation item must verify one or more specific rules above — no orphan rules allowed.}
1. {Concrete check — e.g., "Run tests and confirm all pass"}
2. {Concrete check — e.g., "Verify output matches the user's stated intent"}
3. All three evaluation criteria (Efficiency, Accuracy, Completeness) scored as Pass

## Learning
- **Project memory:** {What this agent records after completing tasks in a project}
- **Self-update:** {What patterns would trigger a proposal to update this file, reviewed by MD Keeper}
