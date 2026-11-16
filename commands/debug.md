---
description: Debug flow — a short path for small low-risk fixes, else the lean loop; reproduce and confirm the cause from source first
---

Debug: $ARGUMENTS

## Small fix: the short path

Every extra turn re-reads the whole context, so a small fix skips the paperwork.
It is small when the confirmed fix touches at most 2 non-test source files and
about 40 changed lines, and touches none of: auth, permissions or tenant scoping;
money or billing; migrations or schema; deletes or other data loss; concurrency;
sending messages or other external writes; this flow's own machinery. A shared
helper counts as risky when any grepped caller is in one of those areas. It also
needs `codex-run` on PATH and a `.charles.toml`; without either, use the full loop.

1. Steps 1-3 below: one command red on the exact symptom, the cause confirmed
   from source with `file:line`.
2. Grep every caller, write the failing test and watch it fail, fix, re-run the
   loop until green.
3. Run `"$SCRIPTS/green.sh" "$(pwd)"` once, at the end, with
   `SCRIPTS="$(dirname "$(readlink -f "$(command -v codex-run)")")"`.
4. Report as below (symptom, red output, cause, fix, green output).

Skip on this path: loading the `charles-flow` skill, the spec doc, `run-state.sh`,
the baseline `green.sh` and the review lane. If the fix outgrows the size or risk
limit while you work, or the final green is red, say so in one line and switch to
the full loop below: load the skill, write the plan, open the run, carry on.

## Full loop

Run the **lean loop** from the `charles-flow` skill. Differs from the default only here: reproduce first, and confirm the cause from
source yourself before planning. Explore lanes, if any, return a cause with
`file:line` evidence, never a patch. The plan holds the confirmed cause, the fix
scope and the green command; the review asks whether the diff fixes that cause and
nothing else. No grill unless the bug touches money, auth/security or data loss.

**1. Loop before theory.** Get one command that goes red on the user's exact
symptom (failing test, curl, CLI call with a fixture, script replaying a captured
payload) and keep its output. It must show the reported symptom, not a nearby
failure; make it fast and deterministic (pin time, seed randomness, isolate files),
and for a flaky bug loop it until the failure rate is workable. No loop possible:
stop and say what you tried and what you need (access, a log, a captured request);
do not guess-and-patch.

**2. Is it even running your code?** Check before the next hypothesis:

- Installed copy vs source: plugins, packages and global CLIs run a copy, so
  editing the repo changes nothing until reinstall (Claude Code plugins also need a
  version bump and restart).
- PATH order: `type -a <cmd>` lists every copy; an old symlink earlier on PATH wins.
- Stale deploy: edge platforms can serve the previous build for a minute or two;
  check the serving version, wait, retest.
- Config not reloaded: `docker restart` does not re-read `.env`; `docker compose
  up -d` recreates changed containers. Many servers read env only at start.
- Wrong target: the deploy went to another account, project or environment. Print
  the target (`wrangler whoami`, project id, branch-to-env mapping) before trusting
  a result.
- Env var "missing": a secret empty in your shell may be injected only into the
  process; check how the process gets it, without printing the value.

**3. Cause, not symptom.** Change one thing at a time and re-run the loop. State
each hypothesis with the evidence that would kill it, then run that check; keep
survivors with `file:line`. Trust the live system (deployed schema, serving
version, actual env) over docs and repo. "Completed" is not "worked": a job that
swallows its own failure reports success forever, so look at what it produced.
Count for real before calling data empty (planner statistics can say zero rows on
a full table).

**4. Fix once.** Grep every caller of what you change; a guard in the shared
function beats a guard in the caller the ticket names. Failing test first, then the
fix, then re-run the step-1 loop: it must go green on the exact symptom. Remove
temporary logging and debug flags.

Report: symptom, loop command and its red output; root cause with `file:line` and
what ruled out the other hypotheses; the fix, its test and the loop's green output;
any sibling caller or neighbouring system with the same flaw.

Implement per the skill (parallel chunks when the manifest warrants, otherwise a
Bash call with `run_in_background: true`):

```bash
codex-run --lane implement --dir <repo> --req <R1,A3> --timeout 2700 "<task>"
```

Cap green-fix cycles at 3, then record a `FAILED` item and stop, reporting what you ruled out and what you would try next.

No `.charles.toml`: stop and offer `/cstack:init`.
