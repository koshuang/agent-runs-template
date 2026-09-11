# Run Checkpoint

## Current state

The invitation form is implemented and wired to the synthetic API contract.
The invitation acceptance page is partially implemented.

## Important decisions

- Keep acceptance UI separate from the invitation creation form.
- Treat expired-token handling as a visible user state, not a silent redirect.

## Verification performed

- Component tests for the invitation form: PASS
- Acceptance-page tests: not run
- Typecheck: not run

## Remaining

1. Finish the acceptance page.
2. Add expired-token UI state.
3. Run tests and typecheck.
4. Update stream state with evidence.

## Relevant artifacts

- `src/web/InvitationForm.tsx`
- `src/web/InvitationAccept.tsx`
- `tests/web/invitation-form.test.tsx`

All identifiers and paths are synthetic.
