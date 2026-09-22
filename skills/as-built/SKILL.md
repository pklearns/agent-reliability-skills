---
name: as-built
description: Forensic reconciliation of claimed state vs. actual state on disk. Use this skill EVERY time someone questions whether something was really built — "did we actually build this", "does this directory exist", "which copy is canonical", "is this status stale", "forensic check", "plan-as-built", "reconcile memory against the repo" — or whenever a discrepancy surfaces between memory/CLAUDE.md/backlog and reality. Also trigger proactively before restarting old work from history and before any major status update to a backlog or project CLAUDE.md files — verify first, record second. This is the codified version of a workflow commonly rebuilt from scratch.
---

# As-Built

Plans, chat summaries, and memory files record *intent*. Disk, git history, and Claude Code transcripts record *what happened*. When they disagree, disk wins. This skill exists because the gap burns people (a "backtest complete" claim — three directories that never existed on any of three machines) and because the same forensic commands get re-derived each time.

## The evidence hierarchy (strongest → weakest)

1. **Git history** — commits, diffs, tags. `git log --oneline --all`, `git show --stat <hash>`.
2. **Filesystem** — files that exist right now. `ls -la`, `find . -name "<pattern>" -newer <ref>`.
3. **Claude Code transcripts** — what was actually touched in sessions:
   - `grep -o '"file_path": *"[^"]*"' <transcript>.jsonl | sort -u` → every file a session touched
   - `ls ~/.claude/projects/` → instant working-directory inventory on macOS
4. **Chat summaries / memory / CLAUDE.md** — claims only. Never evidence; always the thing being audited.

## Procedure

1. **State the claim precisely.** "Backtest complete" is not auditable; "`backtest_agent/` exists with 11 passing tests as of April" is.
2. **Pick the cheapest decisive check** from the hierarchy and run it (or surface the exact command to run if the machine isn't reachable from this session).
3. **Classify the claim**: `AS-BUILT` (evidence confirms), `PLAN-AS-BUILT` (planned, recorded as done, never executed), `DIVERGED` (built, but differently than recorded), or `UNVERIFIABLE` (evidence unreachable from here — name what to run and where).
4. **Multi-copy reconciliation**: when the same repo exists in multiple places, canonical = the copy holding the latest verified milestone commits, not the most recently touched one. List each copy's tip commit and declare canon explicitly.
5. **Write the correction** everywhere the stale claim lives: memory edit, CLAUDE.md diff, backlog line. A forensic finding that doesn't propagate just gets rediscovered next quarter.

## Standing rules

- Never soften a PLAN-AS-BUILT finding. "It was probably built somewhere" is exactly the error class this skill kills.
- Time-box: one decisive check beats five suggestive ones. If step 2's check settles it, stop.
- Output shape: claim → evidence → classification → correction applied/queued. Prose, short, no report headers.
