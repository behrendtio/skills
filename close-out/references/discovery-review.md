# Discovery Review Contract

Perform a high-recall review of the complete current change. Review read-only and finish the full finding set before any fixes begin. Record qualifying issues broadly, but label scope separately: a valid finding is not automatically authorized work. Only findings that pass the close-out scope acceptance test may be fixed.

## Finding threshold

Report every issue the author would likely fix if they understood it. Report nothing when no such issue exists.

A qualifying review finding must be:

1. Introduced by the current change, or concretely exposed by it and therefore worth reporting; exposure alone never authorizes a fix.
2. Discrete, concrete, and actionable.
3. Demonstrably harmful to correctness, security, performance, operability, compatibility, accessibility, or maintainability at the repository's existing quality bar.
4. Supported by a specific affected scenario, input, environment, state transition, caller, or contract.
5. Clearly unintended, without relying on guesses about product intent.

Do not report trivial style, vague hardening, speculative interactions, broad rewrites, or issues whose alleged downstream effect cannot be identified.

## Scope classification

After recording a valid finding, classify it against the frozen scope ledger. Accept it for repair only when all three are proven:

1. The current implementation introduced the defect; a merely exposed or pre-existing defect fails this condition.
2. The defect affects an explicitly scoped behavior or prevents that implementation from working correctly.
3. The smallest correct repair remains inside the recorded implementation owner boundary without changing unrelated behavior.

A changed file, adjacent caller, shared abstraction, repeated pattern, or severe priority does not independently establish scope. When any condition is false or uncertain, mark the finding rejected, follow-up, or stop-and-escalate; do not edit for it.

## Review method

1. Inspect the diff stat and full changed-file list.
2. Review the merge-base-to-HEAD diff together with staged, unstaged, and relevant untracked changes.
3. Trace changed code through callers, callees, types, tests, schemas, configuration, migrations, and dependency contracts as needed.
4. Check all changed surfaces, not only the first suspicious file.
5. Continue until every qualifying finding in the current pass is recorded.
6. Do not edit code during this review pass.

Consider, where relevant:

- Normal behavior, boundary inputs, failure paths, retries, cancellation, and partial completion.
- State lifecycle, stale state, ordering, concurrency, idempotency, and cleanup.
- Authentication, authorization, tenant/data isolation, validation, secrets, and trust boundaries.
- API/type/schema compatibility, migrations, rollback, serialization, and older clients or persisted data.
- Resource use, query behavior, hot paths, unbounded work, and leaks.
- UI layout, loading/error/empty states, keyboard and assistive access, and platform-specific behavior.
- Test assertions that can pass while the behavior is broken, plus missing regression coverage for concrete risks.

Security findings must describe a concrete exploitable or safety-reducing path; do not recommend disabling legitimate functionality without evidence.

## Finding format

Record one entry per unique issue:

- Priority: P0, P1, P2, or P3.
- Title: imperative and no more than roughly 80 characters.
- Location: the narrowest useful changed-file line or range.
- Scenario: the exact conditions required to trigger the problem.
- Impact: what becomes incorrect, unsafe, broken, or materially harder to maintain.
- Evidence: the relevant code path or contract that makes the impact demonstrable.
- Scope status: accepted in-scope, rejected, follow-up, or stop-and-escalate, with the scope-ledger behavior that justifies the classification.
- Fix boundary: for accepted findings only, the smallest ownership boundary in which the scoped introduced instance can be corrected.

Keep each finding concise and matter-of-fact. Do not duplicate one root cause across multiple comments unless independently fixing one site would leave other introduced failures within the frozen scope ledger. Do not turn repeated out-of-scope instances into authorized work.
