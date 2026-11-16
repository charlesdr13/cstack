# Lean path for small, low-risk fixes

Goal: a small bug fix through `/cstack:debug` costs close to a plain session.

Evidence (flow-bench, 2026-11-16, Opus 5.5, real master-inbox fix commits): the
debug flow cost 0.76-1.37 USD per fix against plain's 0.42-0.74, made 20 tool calls
against 10, and read 2.05M input tokens against 0.60M on the same task, with no
higher hidden-test pass rate. The extra spend is the spec doc, run-state
bookkeeping, a baseline full-suite run, loading this skill, and an Opus review that
proved one real bug in five runs. Every extra turn re-reads the whole context.

## Requirements

- [ ] **R1** `commands/debug.md` opens with a small-fix path that applies when the confirmed fix touches at most 2 non-test source files and about 40 changed lines, and touches none of: auth/permissions/tenant scoping, money or billing, migrations or schema, deletes or other data loss, concurrency, the flow's own machinery.
- [ ] **R2** The small-fix path keeps: one command red on the exact symptom, the cause confirmed from source with `file:line`, every caller grepped, the failing test first, the loop going green, then one `green.sh` run at the end; its report keeps the red and green output.
- [ ] **R3** The small-fix path skips: loading the `charles-flow` skill, the spec doc, `run-state.sh` init/phase/close, the baseline `green.sh`, and the review lane.
- [ ] **R4** If the fix outgrows R1 while working (more files, a risky area, a red final green), the path switches to the full loop below it from the plan step, and says so in one line.
- [ ] **R5** `skills/charles-flow/SKILL.md`'s "Small asks skip the flow" line points small debug fixes at that path; the skill stays under its 260-line cap.
- [ ] **R6** `scripts/selftest.sh` asserts debug.md has the small-fix path, names the risk exclusions, and still ends with one `green.sh`.
- [ ] **R7** Version bumped to 3.3.0 in `.claude-plugin/plugin.json` and both `version` fields of `.claude-plugin/marketplace.json`.

Scope: `commands/debug.md`, `skills/charles-flow/SKILL.md`, `scripts/selftest.sh`,
`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, this plan.

Green: `bash scripts/selftest.sh && bash scripts/doctor.sh`.

Conventions: copy the tone and layout of `commands/fast-flow.md`.

Grill waived: prose change to one command, no money, auth or data paths.

Proof after release: rerun the flow-bench cstack arm on the same 10 tasks; target
cost per fix within 1.5x plain, hidden-test pass rate no lower than 3.2.1's.

## Sign-off

- [x] **R1** `commands/debug.md` opens with a small-fix path that applies when the confirmed fix touches at most 2 non-test source files and about 40 changed lines, and touches none of: auth/permissions/tenant scoping, money or billing, migrations or schema, deletes or other data loss, concurrency, the flow's own machinery. — `## Small fix: the short path` is the first section; review added external writes and risky shared-helper callers
- [x] **R2** The small-fix path keeps: one command red on the exact symptom, the cause confirmed from source with `file:line`, every caller grepped, the failing test first, the loop going green, then one `green.sh` run at the end; its report keeps the red and green output. — steps 1-4 of the short path; selftest counts exactly one `$SCRIPTS/green.sh`
- [x] **R3** The small-fix path skips: loading the `charles-flow` skill, the spec doc, `run-state.sh` init/phase/close, the baseline `green.sh`, and the review lane. — "Skip on this path" line; selftest rejects run-state init/phase/close in the section
- [x] **R4** If the fix outgrows R1 while working (more files, a risky area, a red final green), the path switches to the full loop below it from the plan step, and says so in one line. — switch sentence names loading the skill, writing the plan, opening the run
- [x] **R5** `skills/charles-flow/SKILL.md`'s "Small asks skip the flow" line points small debug fixes at that path; the skill stays under its 260-line cap. — selftest: SKILL.md 260 lines (cap 260) PASS
- [x] **R6** `scripts/selftest.sh` asserts debug.md has the small-fix path, names the risk exclusions, and still ends with one `green.sh`. — PASS on the real doc; FAILs when "money or billing" is removed and when a baseline green is added
- [x] **R7** Version bumped to 3.3.0 in `.claude-plugin/plugin.json` and both `version` fields of `.claude-plugin/marketplace.json`. — grep shows 3.3.0 at all three

Green: selftest 484 PASS (baseline 483), doctor 21 ok, 0 failing.
