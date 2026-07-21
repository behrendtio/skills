---
name: close-out
description: "Orchestrate an end-to-end code-change closeout loop: freeze a ticket-derived scope ledger, run an exhaustive Conductor-style discovery review, fix only findings proven to be introduced by and required for the scoped implementation, test and commit-and-push after each complete fix batch, repeat discovery review until clean, then run single-engine Codex autoreview as the independent final gate. Use only when the user explicitly invokes close-out or asks for a review-until-clean workflow that commits and pushes each fix round. Invocation authorizes only scoped fixes, validation, incremental commits, pulls, safe conflict resolution, and pushes to origin; it does not authorize adjacent cleanup, pre-existing or merely exposed fixes, PR creation, merging, force-pushing, reviewer panels, non-Codex autoreview engines, or other external writes."
---

# Close Out

Drive the current change to a practical review fixpoint. Keep the discovery review and final gate independent; do not replace one with the other. Review broadly, but treat the frozen scope ledger as the strict authority boundary for edits.

## Establish the baseline

1. Read repository instructions and the original task, issue, or specification.
2. Identify the actual target/base branch and inspect the complete intended change, including committed, staged, unstaged, and relevant untracked files.
3. Freeze a scope ledger before reviewing. Record:
   - The behaviors explicitly requested by the original task, issue, or specification.
   - The implementation surfaces changed to deliver each requested behavior.
   - The smallest owner boundary for each requested behavior.
   - Explicit exclusions, incidental diff content, and unrelated pre-existing behavior.
   - The initial changed files and approximate non-test LOC as review metadata, not as authority to edit everything they contain.
4. Derive scope from the original task or specification, not merely from the current diff. A touched file, nearby code, shared owner boundary, or valid reviewer concern is not automatically in scope.
5. If the task contract is unavailable or materially ambiguous, stop and ask the user to confirm scope before accepting or fixing findings. Do not infer broad cleanup authority from an ambiguous task.
6. Run formatting first only when it can be restricted to the scoped files and reviewed lines. Run cheap deterministic checks early when they can expose a broken baseline.
7. Treat explicit invocation of this skill as authorization to commit and push each completed fix batch through `git-commit-and-push`. Do not create or update a PR, merge, force-push, post review comments, fix adjacent or pre-existing issues, or write to other external systems unless separately requested.

## Run the discovery loop

Read [references/discovery-review.md](references/discovery-review.md) before the first discovery review.

For each cycle:

1. Review the entire current change using the discovery contract. Use Conductor workspace-diff tools when available; otherwise inspect the Git merge-base diff plus working-tree changes directly.
2. Complete the full review before editing. Do not stop after the first finding.
3. Verify every finding against the real code path, adjacent callers, tests, types, dependency contracts, and frozen scope ledger when relevant.
4. Classify each finding as:
   - **Accepted in-scope** only when all three conditions are proven:
     1. **Change causality**: the current implementation introduced the defect. Merely exposing, revealing, or making a pre-existing defect easier to notice does not qualify.
     2. **Requirement relevance**: the defect affects a behavior explicitly recorded in the scope ledger or prevents that implementation from working correctly.
     3. **Scoped remediation**: the smallest correct fix stays within the implementation's recorded owner boundary and does not alter unrelated behavior.
   - **Rejected**: incorrect, speculative, intentional, pre-existing, or not worth complicating the codebase for.
   - **Follow-up**: real but outside the current owner boundary or change contract.
   - **Stop and escalate**: requires a new public contract, migration strategy, architecture choice, release-process change, or user decision.
   If any acceptance condition is false or uncertain, do not accept or fix the finding. Classify it as rejected, follow-up, or stop-and-escalate.
5. If there are no accepted in-scope findings, mark the discovery review clean and leave the loop.
6. Implement **all** accepted findings from the completed review batch. Do not rerun review midway through the batch. Fix only the introduced instances required for the scoped implementation. Do not repair sibling instances, adjacent defects, generalized bug classes, pre-existing occurrences, or cleanup opportunities unless the scope ledger explicitly includes them. If the smallest safe fix exceeds the recorded scope, stop and request authorization.
7. Add or update regression tests for accepted findings where a stable automated assertion is feasible.
8. After the entire batch is implemented, run formatting, focused tests, and proportionate lint, typecheck, build, broader tests, API/browser/simulator, or other direct behavior proof.
9. After the batch proof passes, use the `git-commit-and-push` skill at `/Users/mario/.agents/skills/git-commit-and-push/SKILL.md`. Follow it faithfully to create one logical conventional commit for the completed round, pull the current branch from `origin`, resolve only safe unambiguous conflicts, and push.
10. If the pull or conflict resolution changes code beyond the tested batch, treat that result as part of the review surface and rerun the relevant proof before continuing. Stop and report any ambiguous conflict, failed commit, failed pull, or failed push instead of bypassing the workflow.
11. Rerun the full discovery review on the updated complete branch diff. Any code change invalidates the previous clean state.

After two non-converging fix cycles, pause to recheck scope and root causes. Continue automatically only when every remaining accepted finding still passes all three acceptance conditions. Report follow-ups separately; do not broaden the change merely to satisfy a reviewer.

## Run the final autoreview gate

After one clean discovery review:

1. Use the available `autoreview` skill and follow its instructions faithfully except that this workflow always fixes the engine selection to Codex only.
2. Invoke the helper with `--engine codex`. Do not use `--panel`, `--reviewers`, Claude, Pi, or any other review engine, even for high-risk changes or when environment defaults select another engine.
3. Allow only autoreview's documented Codex Sol-to-Terra account-access fallback. On capacity, rate-limit, authentication, transport, or unrelated engine failure, keep Codex selected and stop with the exact failure; never switch engines or create a panel.
4. Select a review target that covers the complete intended change. Never treat an empty local diff as proof when committed branch changes remain.
5. Run focused tests in parallel with autoreview when appropriate, or run them separately when credentials, isolation, or project tooling require it.
6. Verify every autoreview finding before editing and apply the same frozen scope ledger and three-part acceptance test used in discovery. Autoreview grants no broader authority: a finding is not acceptable merely because it is valid, severe, located in a changed file, or recommended by the reviewer.
7. Treat valid out-of-scope autoreview findings as follow-ups. Do not let them cause code, test, formatting, generated-file, or configuration changes.
8. If autoreview produces accepted findings, finish the complete accepted batch, add regression coverage, run the relevant proof, use `git-commit-and-push` for that completed round, invalidate the earlier discovery result, and restart the discovery loop from a fresh full review.
9. Reject invalid findings explicitly and briefly. Never change correct or out-of-scope code solely to make a reviewer quiet.

## Stop condition

Stop only when all of these describe the same unchanged diff:

- One full discovery review has no accepted in-scope findings.
- A single-engine Codex autoreview run using `--engine codex` has no accepted/actionable findings remaining.
- Required deterministic checks pass.
- Direct behavior validation passes when the change affects observable UI, CLI, API, runtime, or generated artifacts.
- Every accepted fix batch has been committed and pushed successfully through `git-commit-and-push`.
- The current branch is pushed to `origin` with no remaining intended local changes.
- No code, generated file, formatting, or test-fixture change occurred after those results.

If any file changes after a clean result, restart at the discovery review. A clean model pass is practical evidence, not proof that defects cannot exist.

## Final report

Report concisely:

- The reviewed target/base and scope.
- The frozen scope ledger, including explicit exclusions.
- Discovery cycles and findings accepted, rejected, deferred, or escalated.
- Fixes made by batch.
- Tests and direct behavior proof run.
- Commit hashes and subjects, pull results, pushed branch, remote, and upstream-tracking result for every fix round.
- The final discovery result and the exact single-engine Codex autoreview command/result on the unchanged diff.
- Any conscious follow-ups or blockers.

Do not claim the change was shipped, merged, or included in a PR unless that separate work was explicitly requested and completed.
