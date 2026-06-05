---
name: orient
description: Manual-only orientation workflow. Use only when the user explicitly invokes $orient and asks Codex to orient around a topic, feature, bug, repo area, or codebase context before doing work. This skill gathers surrounding codebase context, reads applicable agent instructions, identifies likely ports and simulator/device targets, reports what it learned, and then stops without starting implementation or other work.
---

# Orient

Use this skill as a context-gathering pass before work begins. It may inspect files and run read-only discovery commands, but it must not start the requested work.

## Hard Stop

Do not edit files, create commits, start dev servers, run tests, boot simulators, launch apps, install dependencies, migrate databases, or make product/code changes while using this skill.

After reporting orientation findings, wait for the user's next instruction. Do not proceed into planning or implementation unless the user explicitly asks in a later message.

## Workflow

1. Restate the topic or area the user asked about in one sentence.
2. Read the active agent instructions:
   - Check for `AGENTS.md` in the current directory and parent directories.
   - Follow included instruction references such as `@/path/to/file`.
   - Note any command wrappers, sandbox rules, validation expectations, or repo-specific conventions.
3. Identify the repository and project shape:
   - Check `pwd`, `git status --short --branch`, and top-level files.
   - Inspect manifests such as `package.json`, `Gemfile`, `pyproject.toml`, `Cargo.toml`, `Podfile`, `app.json`, `app.config.*`, `eas.json`, `vite.config.*`, `docker-compose.*`, and Procfiles only as relevant.
   - Use `rg` or `rg --files` first for file and text searches.
4. Build a topic map:
   - Search for the user's topic terms, related route/component/service names, config keys, and nearby tests/docs.
   - Read only enough surrounding files to understand ownership boundaries and likely entry points.
   - Prefer actual code paths over assumptions from naming.
5. Determine ports without starting services:
   - Inspect repo env files such as `.agent.env`, `.env.example`, `.env.local`, `.env.development`, and documented setup files.
   - Inspect package scripts, server config, Docker/Kamal/Procfile/Vite/Metro/Rails/Next/Expo settings as applicable.
   - Use read-only listener checks such as `lsof -nP -iTCP -sTCP:LISTEN` when useful.
   - Report both configured ports and currently listening processes when they differ.
6. Determine simulator/device targets without launching them:
   - For iOS/macOS app work, inspect project docs and config for preferred simulator names or OS versions.
   - Use read-only simulator listings such as `xcrun simctl list devices available` or `xcrun simctl list devices booted` when available.
   - For Android, inspect Gradle/Expo/React Native docs and use read-only device listings such as `adb devices` only if available.
   - Report the likely target simulator/device and any ambiguity.
7. Stop with a concise orientation report.

## Report Format

Return these sections:

- `Topic`: The area being oriented around.
- `Instructions`: Relevant agent or repo instructions found.
- `Code Map`: Key files, entry points, and ownership boundaries.
- `Ports`: Configured ports and live listeners relevant to the project.
- `Simulators`: Likely simulator/device targets and current availability.
- `Open Questions`: Only questions that block confident next steps.
- `Stopped`: State that no work has been started and Codex is waiting for the user's next instruction.

Keep the report factual and grounded in files or command output. If a value cannot be found, say so directly and name what was checked.
