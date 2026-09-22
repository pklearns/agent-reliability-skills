---
name: loop-audit
description: Structured health audit of autonomous agents and scheduled jobs (mail and notification agents, watcher scripts, signal/analysis agents, an LLM routing layer, cron/LaunchDaemon jobs on an always-on machine). Use this skill EVERY time someone says "audit my agents", "loop audit", "why did the cron fail", "is the fleet healthy", "check my automations", reports a silent agent failure or stale output, or is deciding whether to add a NEW scheduled agent to the stack. Also run it quarterly as a fleet review, and whenever an observability tool is being evaluated — this skill defines what the tooling must answer.
---

# Loop Audit

An autonomous agent that fails silently is worse than no agent — it keeps producing trust while producing nothing. Fleets accumulate known open failure modes (cron failures queued behind an observability backlog, notes missing detail, duplicate processing, rate-limit gaps). This skill audits any agent — or the whole fleet — against six checks and names the disease.

## The six checks (per agent)

1. **Discovery** — Does it decide what's worth acting on itself, or does its output only ever contain what was already known? An agent that never surprises is a Blind Loop: automated execution, manual choosing.
2. **Isolation** — Can two runs (or two agents) collide on the same files/records? Duplicate-processing in an ingestion watcher is this disease already observed. Check for lockfiles, idempotency, or worktree-style separation.
3. **Verification** — What says "no"? Is there any check between "agent produced output" and "output lands in the inbox/email/store"? An agent whose output is never rejected by anything is a Nodding Loop. **Acid test: when did this agent's output last get flagged, rejected, or corrected? If never — the gate is decoration.**
4. **Persistence** — Where does run state live? If the answer involves context or nothing, it's an Amnesiac Loop: it can't know it already processed something, can't resume, can't be audited. Demand a state file/log with at least: item / source / status / timestamp.
5. **Scheduling + caps** — Does the trigger actually fire (check last-run timestamps — "last ran on demo day" = Manual Loop)? Are there caps: per-run timeout, daily budget/volume, max retries? A loop without caps has outsourced its spending and failure authority to its own bugs. LaunchDaemon jobs: verify with `launchctl list | grep <label>` and log mtimes.
6. **Human door** — Where does uncertainty land for review (inbox, digest, flagged section)? And is it actually being sampled? One sample per day per fleet is the minimum; the day no one can explain what an agent changed, the picture has fallen behind.

## Fleet-level pass

- **The four debts**: unverified output → eroded understanding → reflexive trust → uncapped runtime. Score the fleet honestly on each; they compound in that order.
- **New-agent gate**: no new scheduled agent ships until all six checks have a written answer. This is also the acceptance spec for any observability tool — if a tool can't shorten the distance between "failure happens" and "someone sees it," it doesn't earn a slot.

## Output shape

Per agent: one line per check (PASS / GAP / UNKNOWN + evidence), disease name if any, single next action. Fleet: the top 3 gaps ranked by blast radius, not by ease. No padding — an audit that flags nothing on a real fleet failed the acid test itself.
