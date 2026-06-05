---
name: create-pr
description: Create a GitHub pull request for the current branch with an automatically drafted title and exhaustive concise body. Use only when the user explicitly invokes $create-pr or directly asks to use the create-pr skill; do not use implicitly for ordinary GitHub, git, PR, or branch tasks.
---

# Create PR

## Purpose

Create a pull request for the current Git branch after inspecting the branch diff, commit history, and repository state. Draft a short descriptive title and a concise but exhaustive body that explains the work in user-facing terms where possible.

## Workflow

1. Confirm repository context:
   - Run `git status --short --branch`.
   - Run `git branch --show-current`.
   - Stop and ask the user what to do if the current branch is empty, detached, `main`, `master`, or the repository has no GitHub remote.
   - Run `gh pr view --json url,title,body,baseRefName,headRefName` if available. If a PR already exists for the branch, report it instead of creating a duplicate unless the user explicitly asked to refresh or replace it.

2. Determine the comparison base:
   - Prefer the repository default branch from `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
   - Fall back to `git symbolic-ref --short refs/remotes/origin/HEAD` and strip `origin/`.
   - If the base is still ambiguous, ask the user.
   - Run `git fetch origin <base>` when possible, then compute the merge base with `git merge-base HEAD origin/<base>`.

3. Gather branch evidence before writing:
   - `git log --oneline --decorate <merge-base>..HEAD`
   - `git diff --stat <merge-base>...HEAD`
   - `git diff --name-status <merge-base>...HEAD`
   - `git diff <merge-base>...HEAD` or targeted diffs for changed files
   - Relevant test, docs, migration, generated-file, and config changes

4. Draft the PR title:
   - Make it a short descriptive headline for the overall branch.
   - Use sentence case.
   - Prefer user-facing outcomes over implementation details.
   - Avoid vague titles such as "Updates", "Fixes", or "Misc changes".

5. Draft the PR body:
   - Use Markdown.
   - Include an opening one-sentence summary.
   - Include exhaustive but concise bullet lists of all meaningful work in the branch.
   - Group bullets by user-facing areas when useful, such as `Product`, `Experience`, `Reliability`, `Backend`, `API`, `Tests`, or `Docs`.
   - Word bullets as user-facing features or impacts where possible.
   - Include technical-only bullets when they matter for reviewers, but explain their practical impact.
   - Include a `Validation` section listing commands actually run and results. If validation was not run, say so plainly.
   - Avoid dumping raw commit messages, file lists, or implementation minutiae unless they are necessary to make the PR complete.

6. Create the PR:
   - Push the branch if it has no upstream: `git push -u origin HEAD`.
   - Write the drafted body to a temporary file to preserve Markdown formatting.
   - Run `gh pr create --base <base> --head <branch> --title "<title>" --body-file <body-file>`.
   - If the user requested a draft PR, add `--draft`.
   - Return the PR URL and a brief note of any validation gaps.

## Quality Bar

- Do not invent changes. Every title and body claim must be grounded in the branch evidence.
- Prefer an exhaustive branch-wide explanation over a narrow summary of only the latest commit.
- Keep the body concise by grouping related changes and removing duplicate bullets.
- If the branch includes generated artifacts, mention the source change and the generated artifact only when reviewers need to know both.
- If the branch includes risky changes such as migrations, auth, billing, permissions, deploy config, data deletion, or public API changes, call out the review impact explicitly.
