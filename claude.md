# CLAUDE.md — ReschedulingEngine (Team Aurora)

## What this service is
- Customer rescheduling on the Zero Touch critical path.
- Flow: POST /reschedule → eligibility check → SchedulerV2 Reschedule RPC
  → publish job.rescheduled to Kafka → NotificationService.
- Stack: Python, gRPC, Kafka, PostgreSQL. Feature flags via Statsig.

## Known risks
- job.rescheduled can fire twice (AURORA-1247). Any change to the
  scheduler call or the Kafka publish must explain how it prevents duplicates.
- ~15% of traffic still goes through OldSchedulerV1 via the legacy adapter.

## How to work
- If you see a risk in my approach, push back. Don't agree by default.
- For any change touching more than 2 files, write a short plan first
  and wait for my OK before editing.
- Keep changes small. One PR does one thing.
- Every bug fix starts with a failing test.
- Changes to the reschedule path need tests for: a retried request,
  a partial failure, and a scheduler error.

## Before adding anything new, make the case first
Before creating any of these, explain in 2-3 lines why it's needed and
why an existing one can't be reused. Then wait for my OK.
- A new file or module: which existing file did you consider, and why doesn't it fit?
- A new class or entity: what does it own that nothing else owns today?
- A new database table, column, or collection: what query needs it,
  and how will existing data be migrated?
- A new dependency or library: what does it give us that the standard
  library or our current stack doesn't?
- A new Kafka topic or event: who consumes it, and what happens if it's
  delivered twice?
If you can't make a clear case, don't add it. Extend what exists.

## Design principles
- Start with the simplest thing that works. No interfaces, base classes,
  or factories until there's a second real use case.
- Apply SOLID when it solves a real problem, not by default:
  - Single responsibility: if a function does I/O, business rules, and
    formatting together, split it.
  - Open/closed: if an if/else on a type keeps growing, use a lookup
    of handlers instead of adding another branch.
  - Dependency inversion: external clients (scheduler, Kafka, DB) are
    passed in, not created inside business logic, so they can be mocked in tests.
- If you apply a pattern, name it and say in one line what problem it removes.
- No dead code. If you read something, use it. If you add something, call it.
- Name things for what they do. No "utils", "helpers", or "manager" files.

## Reliability rules
- Every external call needs a timeout and handled errors.
- No blocking calls inside async routes.
- Anything that can be retried must be safe to run twice.
- Never swallow an exception. Log it with the job ID, or let it fail loudly.

## What you can do on your own
- Read code, run tests, make local changes, draft PRs.

## Ask first
- Storage or schema changes, feature flags, shared clients,
  new files or entities (see above), anything deploy-related.

## Never
- Put secrets in this file or in code. Secrets come from the environment.
- Run deploy scripts.
- Add a dependency without asking.
