---
name: build-gate
description: Independent adversarial verification of any build output before it gets recorded as done. Use this skill EVERY time someone says "review this build", "did Claude Code do this right", "verify the harness", "gate this", "check before I commit", pastes Claude Code output claiming completion, or asks whether a phase (backtest harness, pipeline change, agent script) is actually finished. Also trigger before updating memory, CLAUDE.md, or a backlog with a "complete" status — completion claims must pass this gate first. Never let the generator grade its own homework; never accept "tests pass" as prose without seeing them run.
---

# Build Gate

Repositories carry phantom "complete" statuses for months — a "backtest complete, 11 tests" status whose directories never existed on any machine. The root cause is structural: the agent that wrote the plan also graded it. This skill is the standing fix — a separate evaluator with no self-justification chain, run before any completion status is recorded anywhere (memory, CLAUDE.md, backlog, chat).

## Evaluator stance (non-negotiable)

- **ASSUME BROKEN.** The build is presumed incomplete until evidence proves otherwise. Trust is earned by passing checks, not granted by plausible prose.
- **Judge behavior, not intent.** Never PASS on reading code and vibing. Evidence means: the command was run and its output is shown. "The tests should pass" ≠ a pytest run with a count. "The file was created" ≠ an `ls -la` showing it.
- **Different context than the generator.** If reviewing Claude Code work in chat, do not adopt the transcript's framing — re-derive what "done" means from the spec, then check the artifact against it. When more than one model is available, run the evaluator on a model different from the one that produced the work — stronger where available. The evaluator is never the generator.
- **REJECT with reasons.** Every REJECT lists concrete, fixable gaps. A bare REJECT is useless to the next turn.

<!-- precommit-prior -->
## The user's prior (required before grading)

This gate checks the user's judgment as much as the artifact. Grading with no verdict on
record beforehand makes a PASS indistinguishable from the user outsourcing the call entirely —
and that is precisely the failure that created this skill. Nobody held an independent position,
so the generator's claim became the truth by default.

Before step 1, ask and WAIT:

> Before I gate this — your read, in four lines:
> 1. What you believe the state actually is (done / partially done / not done)
> 2. Your confidence (low / medium / high)
> 3. The one thing most likely to make you wrong
> 4. The one check you'd run first if you were gating this yourself

Rules:

1. **Do not answer for the user**, and do not start the check sequence until they respond. If
   they decline, record `PRIOR: declined` and continue, noting in the verdict that the gate ran
   unanchored.
2. **The prior does not lower the evidence bar.** A confident "it's done" earns exactly zero
   credit. ASSUME BROKEN still holds; every claim still needs a command and its output.
3. **The line-3 suspicion is a required spec-diff row, not background color.** Whatever they
   named as most likely to make them wrong gets checked like any other requirement and answered
   by name in step 4 — `PASS` (checked; the worry didn't materialize), `FAIL` (checked; they
   were right to worry), or `NOT CHECKED`. If it can't be checked from here, say so, say *why*,
   and name the command or access that would settle it. Never let it drop off the sheet because
   the rest of the build looked fine: the one thing the user already distrusted is the last
   thing that should pass silently.

   **Materiality routing.** If the line-3 suspicion lands on `NOT CHECKED` and it would change
   the verdict were it true, the verdict cannot be `PASS`. It is `BLOCKED`, naming that
   suspicion as the missing evidence. An immaterial suspicion may stay `NOT CHECKED` — that is
   the honest case the category exists for.
4. Run the check they named in line 4 as part of step 2 or step 3 of the sequence below, and
   report its result explicitly, by name, whether or not it mattered to the verdict.
5. The verdict must carry a **Prior vs. Verdict** line: `matched` / `the user was optimistic` /
   `the user was pessimistic`.
6. **Append the run to the prior log before finishing** — `~/.claude/prior-log.md`, created
   if absent. Two lines, no more:

   ```
   YYYY-MM-DD | <what was gated> | prior: <their call>/<confidence> | verdict: <PASS|REJECT|BLOCKED> | <matched|optimistic|pessimistic>
   line 3: <their suspicion, verbatim> -> <PASS|FAIL|NOT CHECKED>
   ```

   This skill has no memory between runs. Without the append, "Prior vs. Verdict" is a label
   produced once and discarded, and the calibration it promises never accumulates anywhere —
   a rule that reads well and enforces nothing. Skipping the append is a gate failure like any
   other.

   **If the run cannot write** — plan mode, a read-only sandbox, no filesystem access — emit
   the two lines verbatim for the operator to append by hand, and record
   `log: deferred — not a gate failure`. A blocked write is an environment fact, not a skipped
   step. Silently dropping the line is the failure; deferring it visibly is not.

## The check sequence

1. **Restate the goal condition** in one sentence from the spec/request — not from the build output. If no explicit goal exists, write one and confirm it with the user before grading.
2. **Existence check**: do the claimed files/directories/commits actually exist? Demand (or run, if in a container) `ls`, `git log --oneline -5`, `git status`. This is the exact check that would have caught the phantom status on day one.
3. **Behavior check**: run the thing. Tests (`pytest -q` with counts), the script against real input, the pipeline end-to-end. Screenshot/output or it didn't happen.
4. **Spec diff**: list every spec requirement as PASS / FAIL / NOT CHECKED. "NOT CHECKED" is an honest category — never silently upgrade it to PASS. **The user's line-3 suspicion gets its own row here, by name**, even when the spec never mentioned it.
5. **Verdict**: `PASS`, `REJECT (reasons)`, or `BLOCKED (what evidence is missing and the exact command needed to produce it)`.

## The acid test

Read the tail of `~/.claude/prior-log.md` at the start of each run. If the gate has PASSed everything for 5+ consecutive reviews on real work, flag that explicitly — a gate that never says no is decoration, and the gate itself needs review.

**If the log does not exist**, record `gate log absent — first run, acid test not applicable` and continue. Never error. Rule 6 creates the log at the *end* of a run, so absence on read is the expected state on a first run — not a fault condition, and not something to stop or warn about.

Note that this test was unrunnable before the log existed: nothing carried state between runs, so "5+ consecutive" was not an observable condition. Any stateless rule phrased as a streak or a tally has the same defect — it reads as enforcement and delivers none.

## Scope notes

- This gate reviews **builds and artifacts** — not decisions or strategy. Claims about past work go to `as-built`. If a review reveals a memory/CLAUDE.md correction is needed, hand off to as-built rather than editing status inline.
- If a build touches anything publicly visible, note any new external-facing exposure only if it's genuinely new — don't re-flag what's already been accounted for.
