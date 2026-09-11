# Agent Runs Template

[English](README.md) | [繁體中文](README.zh-TW.md)

A reusable, vendor-neutral template for durable AI coding-agent execution state.

This repository is intended to be used as a **GitHub template repository**. Create a separate instance repository for real execution data, especially when logs may contain private engineering context.

> Public-template rule: every example in this repository must be synthetic and de-identified. Do not use real company names, product names, repository names, issue numbers, file paths, customer data, credentials, or internal identifiers.

## Goal

Provide a common state and logging contract for multiple coding-agent runtimes such as Claude Code, Codex, OpenCode, and future tools so that a separate orchestrator can answer:

- What is being worked on?
- Which execution streams are active?
- What is blocked?
- What needs human attention?
- What evidence proves completion?
- Can a new agent/session safely take over?

The design separates durable engineering state from noisy runtime detail.

## Core model

```text
Project
  ↓
Work Item
  ↓
Stream
  ↓
Run
```

- **Work Item**: the actual engineering goal, usually linked to a GitHub Issue or another canonical tracker item.
- **Stream**: one execution lane inside the Work Item, such as `api`, `web`, `migration`, `infra`, or `e2e`.
- **Run**: one agent-runtime execution/session. Runs are replaceable; the Work Item and Stream are durable.

`Run != Task` and `Session != Work Item`.

## Information layers

```text
Issue       = What / Why
State       = Where we are
Checkpoint  = What the next agent needs
Events      = What actually happened
PR + CI     = Evidence
```

### Control plane

The canonical engineering issue should contain human-readable requirements, scope, Acceptance Criteria, important decisions, human-required blockers, PR milestones, and verified completion.

It should **not** become a raw agent transcript or per-tool activity log.

### State

State files answer the current-state question. There are three levels:

```text
work-item/index.json
stream/state.json
run/state.json
```

An orchestrator should read progressively from the highest level and drill down only when needed.

### Checkpoint

A checkpoint is a compact handoff artifact. It should contain the minimum context needed for a fresh agent or post-compaction session to continue safely.

### Events

`events.jsonl` is structured runtime history for audit/debugging. It is not the default human interface and should not be mirrored wholesale into GitHub Issues.

## Repository layout

```text
<org-or-scope>/
  <project>/
    work-items/
      <work-item-id>/
        index.json
        checkpoint.md
        streams/
          <stream>/
            state.json
            runs/
              <run-id>/
                state.json
                checkpoint.md
                events.jsonl

schema/
examples/
```

See `examples/acme/example-app/work-items/WORK-123/` for a fully synthetic fixture.

## ChatGPT / orchestrator read strategy

Use progressive drill-down:

```text
Canonical Issue
    ↓
Work Item index.json
    ↓
Stream state.json
    ↓
Run state.json
    ↓
Checkpoint
    ↓
events.jsonl
```

Read only the minimum layer needed to answer the question.

Examples:

- “What is the current status?” → Issue + `index.json`
- “Why is E2E blocked?” → add `streams/e2e/state.json`
- “What happened before the failure?” → inspect the active run and `events.jsonl`

## Runtime responsibility

Logging should primarily be a **runtime / hook / adapter responsibility**, not a model-memory responsibility.

A provider adapter may map lifecycle events such as session start, tool use, stop, compaction, and session end into a normalized event/state protocol.

The agent should not be required to generate a prose summary after every tool call.

## Security and privacy

- Never log credentials, secret values, auth tokens, private keys, or session cookies.
- Raw transcripts should not be pushed by default.
- Prefer metadata over sensitive payloads.
- Redact tool output before durable storage when needed.
- A public template repository must contain **synthetic examples only**.
- Real execution repositories should use visibility appropriate to the data they contain.

Example safe event:

```json
{
  "event": "credential_used",
  "credential": "EXAMPLE_API_KEY",
  "value": "REDACTED"
}
```

## Completion semantics

An agent claiming “done” is not sufficient evidence.

Completion should require evidence appropriate to the work, for example:

- automated tests
- CI checks
- read-back verification
- deployment verification
- measurable acceptance criteria

Acceptance Criteria should have explicit verification state.

## Rollover / repository growth

When an execution repository becomes too large or expensive to operate:

1. Keep the old repository as immutable historical evidence.
2. Create a new repository from this template.
3. Continue with the same schema/version contract.
4. Do not rewrite or compress old history solely for convenience.

The template repository remains the reusable source for new instances; it does not store production execution logs.

## Initial PoC scope

A minimal implementation can start with:

- one GitHub-backed execution repository
- one coding-agent runtime
- runtime hooks/adapters
- Work Item / Stream / Run state
- checkpoints
- structured events
- orchestrator read-back

Avoid building dashboards, databases, real-time streaming, and complex RBAC before the state model is proven useful.

## Acceptance Criteria for an implementation

- [ ] Work Item, Stream, and Run schemas are documented and machine-readable.
- [ ] A new Run can attach to an existing Stream without losing durable state.
- [ ] Current state can be understood without reading the entire event log.
- [ ] Human-required blockers are distinguishable from agent-actionable blockers.
- [ ] Completion requires evidence, not only an agent status claim.
- [ ] Raw runtime detail does not pollute the canonical engineering Issue.
- [ ] Public examples contain synthetic, de-identified data only.
- [ ] A fresh orchestrator can locate the relevant state via repository files.

## Schema

See:

- `schema/work-item.schema.json`
- `schema/stream-state.schema.json`
- `schema/run-state.schema.json`
- `schema/event.schema.json`

## Example

See:

`examples/acme/example-app/work-items/WORK-123/`

The example intentionally uses fictional names and identifiers.