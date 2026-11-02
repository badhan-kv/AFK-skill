---
name: AFK
description: Let the user step away while the session keeps working on its own. Before they leave, interview them about how to handle foreseeable problems (failures, ambiguity, blocked permissions, risky actions, hangs, cost), record the answers as a contract, then run autonomously within it and report on return. Invoke as `/AFK` or `/AFK <task prompt>`. Use when the user says they are going away, AFK, stepping out, leaving it running, or wants unattended work.
argument-hint: "[task prompt]"
---

# AFK — pre-negotiated autonomous mode

Goal: collect every decision the user would otherwise have to make while away, **before** they leave, then execute without needing them. Asking questions is allowed and expected during Phase 1–3. After the user says "go", do not ask questions again (nobody is there to answer) — follow the contract.

Invoked as `/AFK` (no args) or `/AFK <task prompt>`. Args = the task.

## Phase 1 — Intake (ask, don't assume)

This skill is usually invoked **mid-task**, when the work is already mostly defined (e.g. a long GPU training run already going or about to launch). So **infer first, ask only for gaps**. Never re-ask something the conversation, args, or project files already answer.

1. **Infer the situation from what's already there**, in this order:
   - The conversation so far: the current task, what's been decided, what's running, what's left.
   - Args to `/AFK` (treat as an addition or refinement to the above, not a replacement).
   - Live state: running processes/tmux sessions (`nvidia-smi`, `ps`, `tmux ls`), their log/event files and progress, recent files, git status, `CHECKPOINT.md`, `BACKLOG.md`/`SPEC.md`, tests available, resource headroom.
   - If nothing is in flight and no args/context give a task, ask what to work on.
2. **Play it back in 3–5 lines** (task, current state, what "done" appears to be, what the unattended period would involve, expected duration if inferable) and ask the user to confirm or correct it, in one `AskUserQuestion` together with the gap questions below. Don't write a long recap.
3. **Ask only about gaps** (max 4 questions per call, recommended option first), from:
   - **Definition of done / out of scope**, only if not evident.
   - **How long they'll be away** and whether to stop when done or keep going through the backlog.
   - **How to reach them** on completion/blocker: push notification (`PushNotification`, load via ToolSearch), or just a report. Don't claim a channel works unless tested.
   - Genuine ambiguities with more than one reasonable reading.

   If the playback covers everything, skip straight to Phase 2.
4. **Adopt in-flight work as-is.** For something already running, attach to it (monitor, don't restart or alter it) and make the scenarios in Phase 2 about *that* process: what a hang or crash looks like, whether to resume from a checkpoint, relaunch limits, what to do when it ends (evaluate, copy results, stop).

## Phase 2 — Scenario interview

Tailor to the task; skip scenarios that can't occur. Batch related ones. For each, offer concrete policies such as: *proceed with the conservative default* / *skip it, park it, continue other work* / *stop everything and wait* / *retry N times then park* / *other*.

Cover, as relevant:

- **Permission prompts.** The skill cannot grant permissions. A prompt with nobody present stalls the session. Tell the user this, check which commands the task will need, and offer: switch permission mode / allowlist the specific commands before leaving (use `update-config` or `fewer-permission-prompts` only if they ask; never edit settings unprompted). Ask what to do if a call is still denied or blocked (park the item and continue, or stop).
- **Failures.** Tests/builds failing; max fix attempts per problem before parking (suggest 3, different approach each); whether to revert a failed change or leave it in a branch/stash.
- **Ambiguity / design forks.** Pick the simplest/most conventional option and log it, vs. park the decision. Whether a lookup of docs/prior art is OK for unknowns (web access).
- **Scope creep / discovered extra work.** Log to backlog only, vs. do it if small.
- **Risky or irreversible actions** (always ask explicitly; default for anything not explicitly approved is **forbidden**): deleting/overwriting files, `git push`/force-push/rebase/reset, commits, installing/upgrading dependencies, DB migrations, deploys, sending messages/emails/PRs/issues, spending money or cloud resources, touching credentials or secrets, writing outside the project dir, network calls to third parties.
- **Long-running processes** (training, builds, migrations): expected duration, what "hung" looks like, what to do on hang (kill+restart N times, or stop and report). If a process-monitoring skill is available, offer it; for training runs, set up a way to watch progress (e.g. TensorBoard) and tell the user how to reach it.
- **Resources.** Low RAM/disk/GPU: pause heavy work vs. continue reduced.
- **Environment trouble.** Network down, tool unavailable, external service erroring: retry with backoff N times, then park/stop.
- **Context limits / time budget.** Hard stop time or max wall-clock; keep `CHECKPOINT.md` current so work is resumable.
- **Unknown unknowns.** Ask for the catch-all policy: *park the item and continue* (recommended) or *stop and wait*. Also ask for any "never do X" lines and any special instructions.

If an answer is vague or conflicts with another, ask a follow-up. Don't proceed on half-clear answers.

## Phase 3 — Contract and go

1. Write the contract to `.claude/afk/<YYYY-MM-DD-HHMM>.md` in the project (the `.claude/` folder is the tool's own config area; if there is no project, use the scratchpad directory). Contents, in terse bullets:
   - Task, definition of done, out of scope
   - Stories/steps (mirror with TaskCreate/TodoWrite; use the project's `BACKLOG.md`/`CHECKPOINT.md` if it has them)
   - Policy table: scenario → agreed action
   - Pre-authorized actions (exact list) / forbidden actions (everything else risky)
   - Catch-all policy, time budget, notification channel
   - Decision log (empty) and Parked items (empty)
2. Show the user a compact summary of the contract and ask for a final confirmation ("Ready for you to leave? Anything to change?"). Only start on an explicit go. Remind them of any pre-flight they must do themselves (permission mode, keeping the machine awake / SSH session or tmux alive, port-forward/tunnel for any dashboard).

## Phase 4 — Autonomous execution

- **Follow the contract literally.** Authorization is limited to what was listed, for this task only. Approval for one thing never extends to another; a similar-but-different risky action is "unanticipated".
- **Never block on the user.** Do not call `AskUserQuestion` or wait for input. When something comes up: look up the contract → apply the agreed policy → log it. If nothing covers it, use the catch-all policy. Hard rule: unanticipated risky/irreversible/outward-facing actions are never done on a guess — park them.
- **Park, don't stall.** When an item is parked, record it (what, why, what was tried, options for the user) and move to the next independent item. Stop entirely only if the contract says so, everything left depends on the parked item, or continuing would risk harm.
- **Diagnose properly** (verify, don't theorize) within the agreed attempt limits; don't loop on the same failing approach.
- **Verify each story** (tests plus a smoke test where possible) before marking it done; update the backlog, `CHECKPOINT.md`, and the contract's decision log as you go (append, don't overwrite history).
- **Stay in scope and keep it reversible**: prefer a work branch/stash, small commits only if authorized (commit as the user's configured git identity, no co-author trailers), and keep references to the AI tool out of project files.
- **Pace**: for waiting on external/background work use `Monitor`/background tasks or `ScheduleWakeup` with long fallback intervals rather than tight polling; honor the time budget.
- **Notify** via the agreed channel on: finished, hard-stopped, or a blocker needing the user. Keep it one line.

## Phase 5 — Return report

When done or stopped, give a short report (and save it in the contract file):
- **Status:** done / partially done / stopped — and why
- **Completed** (with verification evidence: what was run, result)
- **Decisions made on your behalf** (from the decision log, so they can veto)
- **Parked items** needing the user, each with options and a recommendation
- **Anything surprising** (resource issues, risky things declined)
- **Next step**

Tag unverified claims as unverified. Keep it concise.
