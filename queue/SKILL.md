---
name: queue
description: Manual-only prompt-readiness gate that checks whether a user's prompt is clear enough to act on before any work begins. Use only when the user explicitly invokes $queue by name; never trigger this skill automatically or implicitly. Respond by briefly playing back the task as understood, then give a clear yes/no understood decision and any required numbered follow-up questions instead of implementation, research, planning, file edits, or tool use.
---

# Queue

## Overview

Use this skill as a manual prompt-readiness gate. Decide whether the request is sufficiently clear to start work, then respond only with a concise playback of the task, a yes/no understood decision, and any necessary follow-up questions.

Only use this skill when the user explicitly mentions `$queue`. Do not infer that this skill should run from ordinary ambiguity, clarification needs, planning requests, or similar wording.

## Non-Negotiables

- Do not trigger this skill automatically or implicitly.
- Do not start the requested work.
- Do not edit files, run commands, browse, inspect repositories, create plans, or call tools.
- Do not answer the underlying task.
- Do not provide implementation details, suggestions, summaries, or analysis beyond the task playback and any necessary follow-up questions.
- Do not ask follow-up questions when a reasonable assumption would make the request clear enough and the assumption is low-risk.

## Decision Rule

Answer `Understood: Yes` when the prompt provides enough information for Codex to begin work responsibly. Minor missing details are acceptable when Codex can make conventional assumptions without changing the likely outcome.

Answer `Understood: No` when the prompt is blocked by missing, contradictory, or high-impact information. Ask only for details that materially affect what Codex should do next.

Treat these as common reasons to answer `No`:

- The desired output or success condition is unclear.
- The prompt names multiple possible targets and does not say which one matters.
- The user asks for a change but omits the file, repo, artifact, account, environment, or system to change.
- The prompt contains conflicting instructions.
- The task has legal, financial, medical, security, production, destructive, or privacy impact and lacks required boundaries or consent.
- The user asks Codex to choose among meaningfully different approaches where preference changes the work.

## Output Format

When the prompt is clear enough:

```text
Task: Briefly restate the task you believe the user is asking for.
Understood: Yes
Follow-up questions: No
```

When follow-up questions are needed:

```text
Task: Briefly restate the task you believe the user is asking for, including any ambiguity if relevant.
Understood: No
Follow-up questions: Yes
1. First necessary question?
2. Second necessary question?
```

## Question Guidelines

- Ask the smallest set of questions needed to unblock work.
- Prefer one to three questions.
- Number questions starting at `1.`.
- Make each question concrete and answerable.
- Do not include optional curiosity questions.
- Do not include a preamble, closing note, or any sections beyond `Task`, `Understood`, and `Follow-up questions`.

## Examples

User asks: "Refactor the auth code."

```text
Task: Refactor the auth code.
Understood: No
Follow-up questions: Yes
1. Which repository, app, or files contain the auth code you want refactored?
2. What outcome should the refactor optimize for: readability, testability, performance, bug fixes, or a specific behavior change?
```

User asks: "In the current repo, rename the Calendar tab to Tasks everywhere and update tests."

```text
Task: Rename the Calendar tab to Tasks throughout the current repository and update the related tests.
Understood: Yes
Follow-up questions: No
```
