---
name: fix-ci
description: Manually run every CI/CD action relevant to the current branch that can be executed locally, excluding deploy, publish, release, production, and infrastructure mutation actions; fix any failures, rerun until green, run autoreview, then use git-commit-and-push. Use only when the user explicitly invokes $fix-ci or directly asks to use the fix-ci skill; never invoke this skill automatically or implicitly.
---

# Fix CI

## Core Rule

Use this skill only after explicit user invocation. Do not infer it from ordinary requests to run tests, fix builds, review code, commit, or push.

Run local equivalents of every current-branch CI/CD action that is relevant to the branch, except deploy-related actions. If anything fails, fix it immediately, rerun the affected action, and continue until all runnable non-deploy actions are green or a real external blocker remains.

## Workflow

1. Establish repository state.
   - Check the current branch, worktree status, and recent changes.
   - Read repository instructions such as `AGENTS.md`, `.codex/RTK.md`, `README`, package manager files, Makefiles, task runners, and CI configuration.
   - Preserve unrelated user changes. Do not revert files you did not intentionally change.

2. Discover relevant CI/CD actions.
   - Inspect workflow definitions, including `.github/workflows`, `.gitlab-ci.yml`, `.circleci/config.yml`, `buildkite`, `azure-pipelines.yml`, `Jenkinsfile`, `bitrise.yml`, `codemagic.yaml`, and repo-specific scripts.
   - Include jobs that validate the branch locally: lint, typecheck, test, build, format check, static analysis, security/audit checks, migrations in check mode, generated-code checks, docs checks, mobile/web build checks, and any repo-specific required validation.
   - Exclude any action whose purpose is deployment, publishing, release creation, package upload, app store upload, production data mutation, production service mutation, infrastructure apply/destroy, credential rotation, or environment promotion.
   - If a workflow mixes validation and deploy steps, run only the validation commands that are safe locally.

3. Map actions to local commands.
   - Prefer exact commands from CI job definitions.
   - Use the repo's package manager and task runner conventions instead of inventing alternate commands.
   - If a job requires secrets or hosted-only services, run the closest safe local equivalent and record the limitation.
   - Ask the user only when the action cannot be classified safely and choosing incorrectly could deploy, publish, mutate infrastructure, or consume paid external resources.

4. Run first-pass CI locally.
   - Run each discovered non-deploy action.
   - Track first-pass status for every action before making fixes.
   - Capture enough failure output to diagnose root cause, but keep user-facing summaries concise.

5. Fix failures immediately.
   - Diagnose the actual failing command and root cause before editing.
   - Make the smallest coherent code, test, configuration, or fixture change that resolves the failure.
   - Rerun the failed action after each fix.
   - Rerun any dependent or broader validation when the fix could affect more than the original failure.
   - Continue until all runnable actions pass or an unavoidable external blocker remains.

6. Run autoreview after fixes.
   - Invoke the `$autoreview` skill once local non-deploy CI actions are green.
   - Address any concrete autoreview findings that require code changes.
   - Rerun affected validation after autoreview fixes.

7. Commit and push.
   - Invoke the `$git-commit-and-push` skill after CI and autoreview are complete.
   - Let that skill handle staging, commit message selection, pull/rebase or merge handling, and pushing according to its own instructions.

## Final Response

Always include a compact table with one row per discovered CI/CD action:

| Action | First pass | Result | Failure cause | Fix |
| --- | --- | --- | --- | --- |
| `test` | ✅ | Passed | N/A | N/A |
| `typecheck` | ❌ | Passed after fix | Type error in changed module | Updated the prop type and reran `typecheck` |

Use `✅` when an action was already green on the first pass, `❌` when it failed on the first pass, and `⚠️` when it could not be run locally. For failures, explain why it failed and how it was fixed. For skipped or blocked actions, explain the local execution limitation and whether a safe equivalent was run.

Also summarize:

- The final branch and pushed commit.
- The autoreview result and any fixes made because of it.
- Any actions that remained blocked, skipped, or only partially verified.
