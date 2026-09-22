---
name: worktree-dispatch
description: Structure parallel or multi-task Claude Code work using git worktrees so agents never collide and repo copies never diverge. Use this skill EVERY time more than one Claude Code task is about to run against the same repo — someone says "split this into parallel tasks", "dispatch these fixes", "set up worktrees", "run these in parallel", mentions merge conflicts between agent sessions, or discovers multiple diverged copies of a repo. Also trigger when planning any multi-phase build where more than one workstream will touch the codebase in the same window.
---

# Worktree Dispatch

Four diverged copies of a signal-processing repo are the manual version of the Tangled Loop. The fix is structural: **one canonical clone, N worktrees, zero ad-hoc copies.** A worktree is a built-in git mechanism — one repo, multiple isolated working directories, each on its own branch — so parallel agents are physically separated without duplicating the repo.

## The rules

1. **One canonical clone per machine, ever.** Everything else is a worktree off it. If a second full clone exists, that's an as-built finding, not a workflow.
2. **One task = one worktree = one branch**, named `fix/<slug>` or `feat/<slug>`:
   `git worktree add ../<slug> -b fix/<slug>`
3. **Main is the landing strip, not a desk.** No agent ever works on main. Merges to main happen only after human review — never an agent's.
4. **Cap the parallelism honestly.** `MAX_PARALLEL` is set by "how many diffs can I actually review today," not by machine capacity. Default: **2–3**. More than that and the review debt compounds faster than the work.
5. **Cheaper model + tighter gate beats bigger model, for dispatched work.** Route execution tasks to a cheaper/faster model with an explicit spec and a build-gate check on the way back; reserve a stronger model for the spec and the review.

## Dispatch procedure

1. Read the task list (typically a state file or backlog slice — one row per finding/task).
2. For each task: create the worktree, write a one-paragraph spec into it (goal condition + Stop lines: never merge, never push main, never delete, park uncertainty in `./inbox/`).
3. Launch/queue agents, respecting `MAX_PARALLEL`.
4. On completion, each worktree's diff goes through **build-gate** before merge. PASS → PR/merge candidate; REJECT → reasons written back to the task row.
5. Teardown after merge: `git worktree remove ../<slug>` and delete the branch. Stale worktrees are how divergence sneaks back in.

## State handshake

Every dispatch reads from and writes back to a single state file (see loop-scaffold's schema): task / source / branch / status / verdict. A crashed worktree records `CRASHED — untouched` so tomorrow doesn't re-dispatch blind.
