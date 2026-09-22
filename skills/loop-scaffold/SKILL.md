---
name: loop-scaffold
description: Convert a recurring manual workflow into a complete autonomous loop with all six parts installed. Use this skill EVERY time someone says "automate this", "turn this into a loop", "make this run on its own", "I keep doing this every week", "schedule this", or describes any task performed on a repeating cadence (weekly data-quality checks, verification passes against an external data source, staleness checks on a cached dataset, price watchlists, backlog sweeps). Also trigger whenever a review of usage patterns or an open backlog surfaces a workflow being rebuilt across sessions — rebuilt-N-times is the strongest signal a loop should exist. Never ship a bare cron job with a pasted prompt; scaffold the full structure or don't automate it.
---

# Loop Scaffold

A loop is not a cron job with a prompt pasted in — pasted prompts rot because nobody updates something buried in a schedule. A real loop has six parts. This skill takes one recurring workflow and scaffolds all six, on the available infrastructure — for example, an always-on machine, a scheduler such as launchd/cron or a CI scheduler like GitHub Actions, a notes vault, an existing skill directory.

## Qualify first

Before scaffolding, answer three questions (fast — don't interrogate):
1. **Is the choosing automatable, or only the doing?** If picking the work each run still requires a person, scaffold discovery too or acknowledge it's a script, not a loop.
2. **What's the cost of a wrong run?** Determines gate strictness and whether output goes live or to inbox. If output is published anywhere public, default to draft-to-inbox rather than auto-publish.
3. **Can it leave the laptop?** Local timer = "runs while I'm around." An always-on machine defaults to LaunchDaemon there; GitHub Actions for repo-anchored work.

## The six parts to scaffold (all of them, every time)

1. **Skill, not prompt** — write `.claude/skills/<name>/SKILL.md` with three load-bearing sections:
   - **Read**: concrete sources down to the command (`gh run list --status failure`, a file glob, an RSS list) — "read the context" is not a source.
   - **Judge**: the actionable-vs-noise criteria. This is the ceiling of the whole loop.
   - **Stop**: the red lines, minimum three. Never publish, never delete, never push main, take at most N items per run, anything uncertain → `./inbox/`. The loop will do everything the skill says and only what it says — an unwritten "never" is a scheduled incident.
2. **State file** — `./state/<name>.md` with minimum columns: item / source / priority / status. Memory is disk, not context; tomorrow's run reads status first.
3. **Gate** — what checks output before it lands (build-gate pattern, a deterministic test, or schema validation). Prefer a small model + strict deterministic gate over a big model ungated.
4. **Trigger** — launchd plist or Actions cron. Verify it fires ≥3 times (dry runs count) before trusting it.
5. **Caps, set before first solo run** — per-run timeout, daily budget, max retries. Concrete numbers, not intentions. A cap is a circuit breaker, not a cost optimization.
6. **Human door** — where uncertainty lands (inbox note, digest section) and the sampling contract: which output gets read daily/weekly and when. Write the time down.

## Ship checklist (all boxes or it doesn't ship)

- [ ] Stop section has ≥3 lines
- [ ] State file readable by next run; status column named
- [ ] Gate has rejected something in dry runs (if it never says no, fix the gate, not the checklist)
- [ ] Caps: three numbers written into the config
- [ ] Trigger verified fired without anyone pressing anything
- [ ] Human door + sampling time recorded

## Handoffs

Parallel execution inside a loop → **worktree-dispatch**. Post-ship health → **loop-audit** (quarterly). Whether to build the loop at all, when it's expensive → weigh it deliberately before committing, rather than defaulting to automate.
