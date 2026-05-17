---
name: git-commit-and-push
description: 'Commit local changes with the git-commit conventional-commit workflow, pull the current branch from origin and resolve safe merge conflicts, then push the branch to origin. Use when a user asks to commit and push in one pass (for example: "commit and push", "ship this branch", or "run /commit then push").'
---

# Git Commit and Push

Execute this workflow to create one safe conventional commit, reconcile the current branch with `origin`, and then push the branch to `origin`.

## Steps

1. Confirm repository state.
- Run `git rev-parse --is-inside-work-tree` and stop if not in a git repository.
- Run `git status --porcelain` and `git diff --staged` to inspect pending changes.
- If there are no staged or unstaged changes, report that there is nothing to commit and stop.

2. Run the git-commit workflow.
- Use the `$git-commit` skill at `/Users/mario/.agents/skills/git-commit/SKILL.md`.
- Follow its safety protocol and conventional commit format.
- Stage only intended files.
- Create exactly one logical commit unless the user explicitly asks for multiple commits.

3. Resolve the current branch name.
- Run `git symbolic-ref --quiet --short HEAD`.
- If this fails (detached HEAD), stop and tell the user to check out a branch first.

4. Verify push target.
- Run `git remote get-url origin`.
- If `origin` does not exist, stop and ask the user which remote to use.

5. Pull the current branch from origin.
- Run `git pull origin <branch>` after the local commit exists.
- If the pull completes cleanly, continue to the push step.
- If Git reports merge conflicts, inspect each conflict and resolve only when the intended resolution is clear from the committed local changes, incoming changes, tests, and surrounding code.
- If any conflict is ambiguous, high-risk, or requires product/domain judgment, stop and ask the user how to resolve it. Include the conflicted file paths, the competing local/incoming changes, and the specific decision needed.
- After resolving conflicts, run `git status --porcelain` and commit the merge resolution if Git requires it.
- If the pull fails for reasons other than merge conflicts, stop and report the exact git error with next-step options.

6. Push to origin.
- Push the current branch with `git push -u origin <branch>`.
- If upstream is already set and `-u` is unnecessary, `git push origin <branch>` is also acceptable.

## Safety Rules

- Never use `--force` or `--force-with-lease` unless the user explicitly asks.
- Never use `--no-verify` unless the user explicitly asks.
- Never push if commit creation failed.
- Never push while the worktree has unresolved merge conflicts.
- Never guess on conflict resolution. Resolve clear mechanical conflicts directly, but ask the user before choosing between competing behaviors or unclear intent.
- If push is rejected (non-fast-forward, auth, protected branch, hooks), stop and report the exact git error with next-step options.

## Output

Report:
- commit hash and commit subject
- pull result and any merge/conflict resolution summary
- pushed branch name
- remote (`origin`)
- whether upstream tracking was set
