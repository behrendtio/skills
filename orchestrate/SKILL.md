---
name: orchestrate
description: Explicit-only orchestration of implementation through bounded sub-agents while the current selected model coordinates, monitors workers every 15 minutes, and independently verifies the integrated result. Invoke only when the user explicitly selects $orchestrate; never select it automatically.
---

# Orchestrate

Use this skill only when explicitly invoked as `$orchestrate`. Never infer or automatically activate it from the shape or complexity of a task.

Keep the current selected model as the orchestrator. The orchestrator owns decomposition, dispatch, coordination, review, integration decisions, and final verification. Delegate implementation to sub-agents; do not switch the orchestrator itself to the worker model.

## Dispatch

Split the request into the smallest useful set of independently verifiable work packages. Parallelise only packages that can proceed safely at the same time. For shared worktrees, give concurrent workers disjoint file or module ownership; sequence overlapping changes through dependencies or assign one owner.

Each worker prompt must include:

- a concrete objective and explicit ownership boundary;
- relevant user requirements, repository instructions, and dependency findings;
- acceptance criteria and the checks it should run;
- a reminder that other agents may be editing the same codebase, so it must preserve unrelated work and accommodate concurrent changes;
- a required handoff containing the outcome, files changed, checks run and results, assumptions, and blockers.

Unless the user specifies otherwise, spawn each worker with `gpt-5.6-luna` and `max` reasoning. Because model overrides cannot use a full-history fork, use the smallest positive `fork_turns` value that supplies useful recent context, or `none` and put all required context in the prompt. Honour explicit user model or reasoning choices and task-specific constraints.

Retain a task-to-agent map with ownership, dependencies, status, and last meaningful update. Reuse a worker for follow-up changes in its owned area instead of spawning a replacement without need.

## Monitor and course-correct

Act on worker completion or requests for attention as soon as they arrive. While any worker remains active, perform a heartbeat at least every 15 minutes:

1. Inspect every running worker's status and latest substantive update.
2. Compare its direction with the assigned scope, acceptance criteria, dependencies, and current shared-worktree state.
3. Send a concise correction or missing context only when there is evidence of drift, conflict, stalled progress, or a newly available dependency. Do not interrupt healthy work merely to announce the heartbeat.
4. Update the task map and the time of the check.

Use bounded waits between events and heartbeats so the orchestrator remains responsive. A completion event resets that worker's monitoring need, but not the 15-minute schedule for other running workers. Do not abandon live workers when ending a turn unless the user asks to stop or progress is genuinely blocked.

## Review and verification

Treat worker reports as handoffs, not proof. When a worker finishes:

1. Inspect the resulting diff and relevant surrounding code against the original request and repository instructions.
2. Check for ownership overlap, regressions, missing error paths, security or tenancy issues, shallow tests, and accidental scope expansion.
3. Run coordinator-owned targeted verification, then the applicable broader project gates in proportion to risk. Do not rely solely on commands the worker says it ran.
4. If verification fails, send precise findings back to the worker that owns the area, wait for its correction, and verify again. The orchestrator may make trivial integration-only edits, but feature-bearing corrections stay delegated unless the user directs otherwise.

Finish only when all required packages are integrated, no worker is still active, and the coordinator's checks support the claimed result. Report the implementation outcome, material files or modules, independent checks and results, and any unresolved limitations. Distinguish verified facts from worker claims or unverified environmental constraints.
