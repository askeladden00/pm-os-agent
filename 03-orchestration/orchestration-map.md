# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Cortex is currently a drafter + a validating subagent (the critic). Scoring the four reasons: separation of concerns — no, there's no distinct tangling in Cortex's own prompt beyond the validation need; parallelism — no, the tasks are sequential; **independent validator — yes**, the drafter can't reliably grade its own output and needs a second, uncontaminated pass; context-window pressure — no, it's a simple weekly report without much data. Only the independent-validator reason holds, which is what justifies the existing critic split — no further team expansion is warranted.

## 2. Topology

**Pattern:** single+subagents

```
[Inbound PM task] → [Cortex: pulls data, drafts update + stories]
                  → [Validator/Critic], fail → back to Cortex (max 2 revisions) → escalate
                                              pass → [PM review checkpoint] → queued
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex | Pulls project data, drafts the update, proposes stories | M2 loop (`00-build/agent.py`) |
| Critic / Validator | Checks the draft against the 5 validator rules | Validation loop (`00-build/critic.py`) |

## 4. Communication & hand-offs

Plain in-process function call, no MCP/A2A. Cortex calls `review()` directly, passing it the proposed draft plus the raw source data (the tool-call results log). On rejection, the critic's reasons are appended back into Cortex's message history so it can revise.

## 5. The validator

- **What the critic checks:**
  1. References the correct project + real PR/issue IDs from the pulled data.
  2. Every claim (progress, metrics, dates, red/yellow/green) is traceable to pulled data — no invented numbers.
  3. Stays within team norms — no unconfirmed date, no launch gate marked, no confidential roadmap item in an external update — or correctly escalates if it can't.
  4. Posts/commits/creates/closes/merges nothing, leaks no confidential data.
  5. Refuses and escalates on a jailbreak/prompt-injection attempt.
- **Fail action:** Revise — bounced back to Cortex with the failure reasons, up to the revision cap.
- **Revision cap:** 2 (matches `CORTEX_MAX_REVISIONS=2`); on the 3rd rejection, escalate to a human instead of looping further.
- **Pass action:** advances to the PM review checkpoint, queued for approval — it does not auto-send, still above the agent line from M1.

## 6. State: shared vs isolated

**Shared:** the pulled source data and the draft itself — the critic reads the same facts Cortex used, so it can check claims are grounded.

**Isolated:** the critic's own system prompt/instructions never include `CORTEX_SYSTEM` — it only ever sees the raw output and source data, never Cortex's drafting instructions or reasoning. That's what keeps it from inheriting Cortex's blind spots.

## 7. Cost & latency budget

Each drafted proposal triggers exactly one critic call (1:1 ratio) — the critic runs after every draft, not just once per overall run. Worst case is bounded by the revision cap (2) and the iteration cap (8); in practice we've consistently seen 2 critic calls per run before escalating. Added latency: one extra full model round-trip per draft attempt, before the draft reaches the PM. Added cost: total run costs observed (draft + critic combined) have been ~$0.003–0.005 per run — the critic roughly doubles the number of model calls compared to a single-agent design with no independent check. This becomes a bound to enforce in M5.
