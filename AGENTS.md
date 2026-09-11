# Agent Instructions

This repository defines the reusable contract for durable AI-agent execution state.

## Scope

This template stores schemas, rules, and synthetic examples. It must not contain real company, product, customer, repository, issue, credential, or internal-path data.

## Core invariants

1. Preserve the hierarchy: `Project → Work Item → Stream → Run`.
2. A Run is ephemeral; Work Item and Stream state are durable.
3. GitHub Issues are the human-facing control plane, not the raw execution log.
4. `index.json` is the aggregate Work Item state and the normal orchestrator entry point.
5. `state.json` files represent current state, not append-only history.
6. `checkpoint.md` is a handoff artifact, not a transcript.
7. `events.jsonl` is append-oriented structured runtime history.
8. Completion requires evidence appropriate to the Acceptance Criteria.
9. Raw transcripts are not durable by default.
10. Never persist secrets or sensitive payloads.

## Public-template de-identification rule

All examples in this repository must be fictional and synthetic.

Allowed examples:

- `acme/example-app`
- `WORK-123`
- `api`, `web`, `e2e`
- `EXAMPLE_API_KEY`

Do not use:

- real employer/client/company names
- real product names
- real repository names
- real issue or PR numbers
- real branch names tied to private work
- real customer/user information
- credentials, secrets, internal URLs, hostnames, IPs, or production identifiers

When contributing an example derived from real work, generalize it before committing.

## State-writing rules

- Update only the narrowest state layer that changed, then refresh aggregate parents as needed.
- Keep state concise enough for an orchestrator to read cheaply.
- Use explicit statuses rather than ambiguous prose.
- Mark human attention separately from ordinary blockers.
- Link to evidence instead of copying large logs into state.

## Suggested statuses

Work Item / Stream / Run statuses should prefer a small controlled vocabulary such as:

- `pending`
- `in_progress`
- `blocked`
- `verification_pending`
- `completed`
- `cancelled`

Do not use `done` as a substitute for verified completion.

## Evidence rules

Before setting `completed`, record evidence appropriate to the work, for example:

- tests passed
- CI passed
- read-back verified
- deployment verified
- Acceptance Criteria individually verified

An agent assertion alone is insufficient.

## Event rules

Prefer structured events over prose diaries.

Good:

```json
{"event":"test_failed","suite":"InvitationFlowTest","exit_code":1}
```

Avoid long free-form reasoning or transcript dumps.

## Handoff rules

A checkpoint should let a fresh agent continue without replaying the full transcript. Include:

- current state
- important decisions
- completed work
- remaining work
- blockers
- verification already performed
- relevant artifacts/files

## Schema compatibility

Schema changes should be backward-conscious. If a breaking change becomes necessary, add an explicit schema version and document migration/compatibility expectations in the README before updating examples.