# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook

**Why this type?** Cortex only reacts when someone actually asks — there's an inbound request that starts the work, not a schedule or a continuous poll.

**Ruled out:**
- *Heartbeat* — this is a batch process that runs only once a week, not something needing continuous polling.
- *Cron* — would fire even if nobody asked, producing an update nobody requested.
- *Goal* — Cortex's "done" is a fixed checklist (update drafted + stories proposed), not an open-ended objective it has to search for.

**Idempotency:** if the same inbound message fires the hook twice, Cortex dedupes by message/task ID and skips redrafting if it's already been processed.

## 2. Goal / definition of done

Status update drafted and backlog stories proposed (within the queue cap), both queued for human approval; critic has passed the draft; nothing is posted or created in the tracker.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | The critic returns a pass verdict on the drafted update, within the revision cap. | Loop ends; draft + proposed stories held for human review. |
| **Stuck / give up** | The critic rejects the draft enough times to hit `CORTEX_MAX_REVISIONS=2`, or the loop reaches `CORTEX_MAX_ITERATIONS=8` without a passing draft — either one independently ends the run. | Stop, log the last draft, escalate to a human. |
| **Escalate to human** | The requested project doesn't exist or a firm ship-date is demanded (`task-missing-data`); the request asks Cortex to do something above the line — post company-wide, mark a gate green, share embargoed content (`task-jailbreak`); or a proposed story batch exceeds the queue cap. | Refuse/flag, hold the draft, escalate — matches the HITL checkpoints from `agent-line-map.md`. |

## 4. State

Within a single hook-triggered run, the loop tracks the current draft, critic feedback, and iteration/revision counts — all scoped to the one requested project (`get_project`/`get_activity` are always scoped by `project_id`). Nothing persists across runs: project state, activity, roadmap, and norms are re-read fresh from fixtures every time. Per the roadmap tool's own warning, CONFIDENTIAL items must never leak into an external or cross-project update.

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP), otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed yet, because Cortex only reads fixtures and writes one draft file locally (`run-output/`); no branching or isolated git workspace per run. |
| **Skills** (reusable capabilities) | Not needed yet, because the whole task (pull → draft → critique) fits in one tool-calling loop; no separate reusable skill module required. |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet — everything today reads mock JSON fixtures, no real Jira key or Google MCP connected. Plan: wire a real GitHub/Jira connector when Cortex moves off the fixtures. |
| **Subagents** (independent check when the loop can't grade itself) | Placeholder → M3 orchestration-map.md. |
| **State tracking** | Locked, always-on (see §4). |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

`00-build/agent.py`
