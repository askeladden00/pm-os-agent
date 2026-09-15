# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | · |
| Decide relevant context | M | M | M | Below | required |
| Draft the update | H | L | M | Below | spot-check |
| Decide tone/commitment level | L | H | L | Above | required |
| Flag at-risk/escalation | L | H | M | Above | required |
| Choose what to escalate | L | H | M | Above | required |
| Propose a story batch (capped) | H | L | H | Below | spot-check |
| Post an update / approve a company-wide one | L | H | L | Above | required |

## Agent anatomy (sketch)

- **Model:** `gpt-4o-mini` as the default for routine runs; escalate to a frontier model after the critic rejects a draft twice (matches `CORTEX_MAX_REVISIONS=2`) — two failed passes signals the routine model isn't handling the nuance.
- **Tools:** `get_task` · `get_project` · `get_activity` · `search_past_updates` · `get_roadmap` · `get_norms` · `propose_stories` (capped, queued for approval). Deliberately absent: any `post_update`, `create_issue`, `merge_pr`, or `commit_ship_date` tool — the agent line is enforced in code, not just in the prompt.
- **Memory:** read-only persistent context only — norms, roadmap, and past updates are re-read fresh each run for precedent and grounding; Cortex keeps no run-to-run memory of its own.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

1. *Pull project state + activity* sits **below** the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: **blast radius** (nothing to damage on a read).
2. *Decide relevant context* sits **below** the line (with required HITL) because it's medium to reverse, has a medium blast radius, and is only medium to verify — deciding factor: **measurability** (can't fully confirm nothing material was omitted, hence the approval gate).
3. *Draft the update* sits **below** the line because it's highly reversible, has a low blast radius, and is medium to verify — deciding factor: **blast radius** (unpublished, so a rough draft costs nothing).
4. *Decide tone/commitment level* sits **above** the line because it's low to reverse, has a high blast radius, and is low to verify — deciding factor: **blast radius** (a stated commitment reaches leadership directly).
5. *Flag at-risk/escalation* sits **above** the line because it's low to reverse, has a high blast radius, and is medium to verify — deciding factor: **reversibility** (a missed flag can't be un-missed once time passes).
6. *Choose what to escalate* sits **above** the line because it's low to reverse, has a high blast radius, and is medium to verify — deciding factor: **blast radius** (escalating the wrong thing, or missing the right thing, does real damage).
7. *Propose a story batch (capped)* sits **below** the line because it's highly reversible, has a low blast radius, and is highly measurable — deciding factor: **blast radius** (the cap bounds the damage even if wrong).
8. *Post an update / approve a company-wide one* sits **above** the line because it's low to reverse, has a high blast radius, and is low to verify — deciding factor: **reversibility** (you can't unpublish it).

## Hardest call

**#2, "Decide relevant context,"** was the hardest call. First instinct placed it fully above the line, but the scores (Med/Med/Med) landed squarely on the golden rule's own borderline case. **Measurability** was the deciding factor — there's no reliable way to confirm after the fact that nothing material was left out of the drafted context, so instead of keeping it fully human-owned, it moved to below-the-line-with-required-HITL: Cortex proposes the relevant context, a human approves it before drafting proceeds.
