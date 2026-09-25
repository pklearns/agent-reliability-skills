# build-gate

`build-gate` is a Claude skill that checks a build before anyone records it as done. Coding
agents write convincing completion reports ("phase complete, all tests pass") whether or not
the work exists, because the agent that did the work is also the one grading it. `build-gate`
makes a separate evaluator presume the build is broken, re-derive what "done" means from the
spec, and demand evidence: the files listed on disk, the tests actually run with their counts,
every requirement marked PASS / FAIL / NOT CHECKED. Before it gates real work, it runs once
against a planted-defect canary. If it passes the canary, the gate is blind and it stops.

## Who it's for

People building with Claude Code who can't independently verify a completion claim, whether
because they aren't developers, the codebase is too large to read, or there's too much to
check. If you can't tell whether "tests pass" is true, this gate makes the agent show you.

## Install

**Claude Code: personal (all projects):**
```bash
git clone https://github.com/pklearns/agent-reliability-skills.git
mkdir -p ~/.claude/skills
cp -r agent-reliability-skills/skills/build-gate ~/.claude/skills/
```

**Claude Code: project-scoped (checked into <your-repo>):**
```bash
mkdir -p .claude/skills
cp -r /path/to/agent-reliability-skills/skills/build-gate .claude/skills/
```

Copy the whole folder, including `fixtures/canary/`, because the self-test needs it.

**Check that it loaded:** ask Claude to "gate this build" against the canary claim in
`fixtures/canary/completion-claim.md`. It should return `REJECT` and name the failing test.

## A real example

<!-- To be written. -->
