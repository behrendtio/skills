# Skills

A curated collection of agent skills sourced from upstream repositories, maintained as first-class git subtrees.

## What's included

Skills are discovered automatically by opencode and Codex via `**/*.md` glob patterns from the skills directory.

### Upstream skills

| Skill | Source |
|---|---|
| `ce-brainstorm` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-code-review` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-compound` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-debug` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-doc-review` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-plan` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `ce-work` | [compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) |
| `autoreview` | [openclaw/agent-skills](https://github.com/openclaw/agent-skills) |
| `frontend-design` | [anthropics/skills](https://github.com/anthropics/skills) |
| `git-commit` | [awesome-copilot](https://github.com/github/awesome-copilot) |
| `swiftui-liquid-glass` | [Dimillian/Skills](https://github.com/Dimillian/Skills) |
| `swiftui-ui-patterns` | [Dimillian/Skills](https://github.com/Dimillian/Skills) |

### Local skills

| Skill | Description |
|---|---|
| `ce-dhh-rails-style` | Legacy Rails style skill retained locally; not currently published as a top-level skill by compound-engineering-plugin |
| `git-commit-and-push` | Commits changes with a conventional commit message, pulls from origin to reconcile the branch, resolves safe merge conflicts, and pushes to origin |
| `playwright` | Automates real browsers from the terminal — navigation, form filling, snapshots, screenshots, data extraction, and UI-flow debugging via a bundled CLI wrapper script |

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
bin/update --dry-run ce-plan
bin/update ce-brainstorm ce-code-review ce-compound ce-doc-review ce-plan ce-debug ce-work
```

This pulls the latest version of every upstream-managed skill listed in `sources.tsv`. You can pass one or more skill IDs to update only those skills.

The script will:
1. Clone each upstream repo to a temporary directory
2. Validate each configured source subdirectory
3. Copy the source subdirectory into the local skill folder
4. Clean up temporary files

Run it whenever you want to sync with upstream changes.

## Adding a new skill

### From an upstream repository

1. Add one entry to `sources.tsv`
2. Run `bin/update --dry-run <skill-id>` to verify the configured source
3. Run `bin/update <skill-id>` to pull it in

### Manually (local skill)

1. Create a new directory at the repo root:
   ```bash
   mkdir my-skill
   ```
2. Add a `SKILL.md` with YAML frontmatter (name, description) and the skill instructions
3. Add any supporting files (scripts, references, agents configs) inside the same directory
4. Commit directly to this repo:
   ```bash
   git add my-skill
   git commit -m "Add my-skill"
   ```

Local skills are not managed by `bin/update` and will never be overwritten by upstream syncs.

## Structure

All skills live at the repository root. Each skill is either:
- An **upstream-managed skill** (listed in `sources.tsv`, synced by `bin/update`)
- A **local skill** (committed directly, not synced)
