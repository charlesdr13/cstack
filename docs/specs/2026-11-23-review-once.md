# Review once per run, no default round 2

Dispatch logs (45 runs, 121 implements, 81 reviews) show 24 of 45 runs ran 2+
reviews: the extra one is the round-2 re-check after fixes. The fix is already
proved by its finding's `Repro:` plus green, so a second reviewer adds little.

- [ ] **R1** `skills/charles-flow/SKILL.md` step 5: one review per run, after the last implement, never per lane.
- [ ] **R2** Proven findings are fixed inline by the orchestrator; a fix is proved by its `Repro:` passing plus `green.sh`, with no second review. Round 2 stays only for money, auth/security or data-loss changes.
- [ ] **R3** A fix big enough to need an implement lane is a new slice and gets its own review (keeps `flow-status.sh`'s "implement with no later review" gate true without code changes).
- [ ] **R4** `agents/codex-reviewer.md` description no longer says "Use after any implementation"; README diagram no longer says "max 2 rounds".
- [ ] **R5** Version bumped to 3.3.2 in all three places. Green: `bash scripts/selftest.sh && bash scripts/doctor.sh`.

Files in scope: the four above plus `.claude-plugin/*.json`. No script changes.
Grill waived: docs-only wording change, reversible.
