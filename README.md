# Skills

A curated collection of agent skills sourced from upstream repositories, maintained as first-class git subtrees.

## What's included

Skills are discovered automatically by opencode and Codex via `**/*.md` glob patterns from the skills directory.

### Upstream skills (git subtrees)

| Skill | Source |
|---|---|
| `ce-brainstorm` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-code-review` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-compound` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-dhh-rails-style` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-doc-review` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-plan` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-work` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `frontend-design` | [anthropics/skills](https://github.com/anthropics/skills) |
| `git-commit` | [awesome-copilot](https://github.com/github/awesome-copilot) |
| `swiftui-liquid-glass` | [Dimillian/Skills](https://github.com/Dimillian/Skills) |
| `swiftui-ui-patterns` | [Dimillian/Skills](https://github.com/Dimillian/Skills) |

### Local skills

| Skill | Source |
|---|---|
| `git-commit-and-push` | Copied from local `~/.codex/skills/` |

## Usage

### Install

Clone into your agent's skills directory:

```bash
git clone git@github.com:behrendtio/skills.git ~/.agents/skills
```

Both opencode and Codex discover skills automatically from this location.

### Update from upstream

```bash
bin/update
```

This pulls the latest version of every upstream skill via `git subtree`. Each skill is extracted from its source repository and merged independently, preserving the flat directory structure.

The script will:
1. Clone each upstream repo to a temporary directory
2. Extract only the relevant skill subdirectory via `git subtree split`
3. Pull changes into the local skill folder
4. Clean up temporary files

Run it whenever you want to sync with upstream changes.

## Adding a new skill

1. Add the skill ID to `ALL_SOURCES` in `bin/update`
2. Add the corresponding entries to `source_url()`, `source_prefix()`, and `local_prefix()`
3. Run `bin/update` to pull it in

## Structure

All skills live at the repository root. Each skill is either:
- A **git subtree** (managed by `bin/update`, synced from upstream)
- A **local skill** (committed directly, not synced)
