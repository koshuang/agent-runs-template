# Checkpoint

## Current state

The invitation API is complete.
The invitation form is complete.
The invitation acceptance page is still in progress.
E2E verification is waiting on the acceptance page.

## Important decisions

- Invitation tokens expire after 7 days.
- Existing users accept invitations after authentication.
- New users are redirected to account creation before acceptance.

## Verification

- API tests: PASS
- Web tests: pending
- E2E: blocked by incomplete UI

## Remaining

1. Complete the invitation acceptance page.
2. Run web tests and typecheck.
3. Run E2E coverage.
4. Open or update the implementation PR.
5. Verify CI.
6. Verify each Acceptance Criterion before completion.

## Relevant artifacts

- `src/api/invitations.ts`
- `src/web/InvitationAccept.tsx`
- `tests/invitation-flow.spec.ts`

All paths above are synthetic example paths.
