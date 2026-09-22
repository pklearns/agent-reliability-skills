# Agent Reliability Skills

Five Claude skills for the failure mode nobody warns you about: **agents that report success
they didn't earn, and scheduled jobs that fail silently while you keep trusting them.**

Most published skills make an agent do more. These make it prove what it did.

---

## The problem

Coding agents are good at producing plausible completion reports. A status line that says
"phase complete, 11 tests passing" can survive in a README, a `CLAUDE.md`, and a project backlog
for months while the directories it names have never existed on any machine.

This isn't sloppiness. It's structural: **the agent that writes the plan also grades it.** Self
assessment inside a single reasoning chain will rationalize whatever the chain already claimed.
The same structure shows up one layer out, in scheduled work — an agent that has never had a
single output rejected by anything isn't reliable, it's unaudited.

Every skill here is one countermeasure to that pattern.

<!-- OPTIONAL: replace the paragraph above, or add below it, with your own account of how you
     hit this. A concrete war story is the most persuasive thing in a repo like this, and it
     should be in your voice, not a generic one. -->

---

## The five skills

| Skill | Answers |
|---|---|
| [`build-gate`](skills/build-gate/SKILL.md) | Is this build actually done, right now, on evidence? |
| [`as-built`](skills/as-built/SKILL.md) | Was this thing ever really built, whatever the record says? |
| [`loop-scaffold`](skills/loop-scaffold/SKILL.md) | What has to exist before a workflow is allowed to run unattended? |
| [`loop-audit`](skills/loop-audit/SKILL.md) | Is this running agent healthy, or quietly dead? |
| [`worktree-dispatch`](skills/worktree-dispatch/SKILL.md) | How do parallel agents work on one repo without colliding? |

They cross-reference each other and are designed to be installed together, though each stands
alone.

### How they chain

```
        planning a recurring job
                  │
           loop-scaffold ──────────┐
                  │                │  (parallel execution)
                  ▼                ▼
             build-gate  ◄─── worktree-dispatch
                  │
        ┌─────────┴──────────┐
        ▼                    ▼
   status recorded       loop-audit
        │              (quarterly, or on
        ▼               silent failure)
    as-built
 (when the record and
  the disk disagree)
```

Rough rule: `build-gate` checks work that just happened. `as-built` checks work that supposedly
happened a while ago. `loop-scaffold` and `loop-audit` are the before and after of unattended
work. `worktree-dispatch` keeps concurrent agents from writing over each other.

---

## The prior step

`build-gate` doesn't start grading immediately. Before it runs a single check, it asks the
person who requested the review for their own read — state, confidence, the one thing most
likely to make them wrong, and the one check they'd run first — and waits for an answer. Skip it
and the run is recorded as unanchored.

The point isn't ceremony. Without a prior on record, a PASS is indistinguishable from the user
outsourcing the judgment call entirely — and an agent grading its own optimism back to itself is
the exact failure this repo exists to catch, just moved one level up.

Every run appends two lines to `~/.claude/prior-log.md`: the prior, the verdict, whether they
matched, and how the person's named suspicion (line 3) actually resolved. That log is the only
place calibration accumulates — whether someone's high-confidence calls get overturned more
than their low-confidence ones is a question the log can answer and a single run never can. A
run that can't write to disk (plan mode, a read-only sandbox) still prints the two lines for
manual append and says so explicitly — it never just drops them.

---

## Vocabulary

Naming a failure is most of catching it. These are the labels the skills use.

**Claim classifications** (`as-built`):

| Term | Meaning |
|---|---|
| **AS-BUILT** | Evidence confirms the claim as recorded. |
| **PLAN-AS-BUILT** | Planned, recorded as done, never executed. |
| **DIVERGED** | Built, but materially differently than recorded. |
| **UNVERIFIABLE** | Evidence unreachable from here. |

**Loop diseases** (`loop-audit`):

| Term | Meaning |
|---|---|
| **Blind Loop** | Automated execution, manual choosing. It never surprises you. |
| **Nodding Loop** | Nothing has ever rejected its output. |
| **Amnesiac Loop** | No durable state; can't resume, can't be audited. |
| **Manual Loop** | The trigger has only ever fired when someone pressed a button. |
| **Tangled Loop** | Shared mutable state, no isolation between runs. |
| **Uncapped Loop** | No timeout, no budget ceiling, no retry limit. |

**The four debts**, which compound in order:

> unverified output → eroded understanding → reflexive trust → uncapped runtime

---

## Acid tests

Each gate in this set carries a test aimed at itself, because a check that never fires is worse
than no check — it produces confidence without producing safety.

- A **build gate** that has PASSed five consecutive real reviews needs its own review.
- A **loop audit** that flags nothing on a live fleet has failed its own acid test.
- A **verification step** whose output has never once been rejected is decoration.
- The **prior log** (`~/.claude/prior-log.md`) is what makes the first acid test checkable at
  all — read its tail before every run.

---

## Not in this release

Two things referenced in earlier internal design notes for this skill set are planned but not
built:

- **`build-gate`'s TRACES step** — a structured trace of exactly which commands/evidence backed
  each spec-diff row, beyond the prose currently required.
- **`loop-audit`'s per-run trajectory grade** — scoring a single run's decision path, not just
  the six-check fleet snapshot this release ships.

Both are absent from the skills as published here. Treat any reference to either elsewhere as
forward-looking, not current behavior.

---

## Install

Skills live in a `skills/` directory that Claude reads. Drop the five folders in.

**Claude Code — personal (all projects):**
```bash
git clone https://github.com/pklearns/agent-reliability-skills.git
mkdir -p ~/.claude/skills
cp -r agent-reliability-skills/skills/* ~/.claude/skills/
```

**Claude Code — project-scoped (checked into a repo, shared with your team):**
```bash
mkdir -p .claude/skills
cp -r /path/to/agent-reliability-skills/skills/* .claude/skills/
```

**Claude apps:** upload the individual skill folders through the skills interface in settings.

Verify they loaded by asking Claude to list its available skills, or by triggering one directly
— "gate this build" should invoke `build-gate`.

### Installing selectively

The skills reference each other by relative path in their handoff sections. If you install only
some of them, those links will dangle — harmless, but worth deleting the orphaned handoff lines
so the model doesn't route to something that isn't there.

---

## Adapting these

They're deliberately unopinionated about stack, but a few things are worth setting to your own
environment:

- **`worktree-dispatch`** — `MAX_PARALLEL` defaults to 2–3. Set it to how many diffs you will
  genuinely review in a day, not what your machine can run.
- **`loop-scaffold`** — the scheduling section assumes launchd, systemd, or GitHub Actions.
  Swap in whatever you actually run.
- **`loop-audit`** — the liveness commands are macOS and Linux. Add your own if you're elsewhere.
- **Model tiering** — several skills say "cheaper model builds, stronger model gates." That
  separation matters more than which specific models you name.
- **`~/.claude/prior-log.md`** — the path is a default, not a requirement. Point it anywhere
  writable; just keep it the same path across skills that share it.

The one thing not to adapt away: **the evaluator must not be the generator.** Every other detail
here is negotiable.

---

## Contributing

Issues and PRs welcome, particularly:

- Additional liveness and forensic commands for platforms not covered
- Failure modes that don't fit any of the six loop diseases
- Cases where a gate passed something it shouldn't have — those are the most useful reports

---

## License

MIT. See [LICENSE](LICENSE).
