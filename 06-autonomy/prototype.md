# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo
>
> ✅ **What this validates:** the agent actually runs end to end, by the end you'll have proven it with real screenshots of your Cortex across the six required moments (M2 to M6).

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | _[img]_ | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted) | M2 |
| 2 | _(transcript below)_ | the critic rejecting a bad draft (revise/block) | M3 |
| 3 | _(transcript below)_ | a grounded update citing pulled activity + a caught hallucination | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

### M3 — Critic rejection (transcript, accepted substitute for a screenshot per LAB.md)

**Caption:** Critic rejects Cortex's draft for reporting "Green" status while an open issue (#818) goes unaddressed, then bounces it back for revision (1/2) before ultimately escalating on the iteration cap.

```
[step 3] PROPOSED OUTPUT: ...Status: Green... Risk/Blocker: No critical blockers currently affect the project...

CRITIC, independent validation
{
  "verdict": "fail",
  "reasons": [
    "The output incorrectly states a 'Green' status without considering that the most recent updates show an open issue (#818) which has not been resolved and could impact the ongoing work. This contradicts the requirement to call the project 'Green' only if there are no critical blockers or open Sev-1 issues.",
    "The 'Next Steps' section is vague and does not provide traceable claims about upcoming work that directly align with the current project status or metrics.",
    "The last two points of the proposed output mentioned risk/blocker status and stated 'No critical blockers currently affect the project,' which is misleading due to the open issue (#818). This could mislead stakeholders regarding the actual project status.",
    "The output does not capitalize on the retrieved state regarding the roadmap and norms. The claims of a 'Green' status should directly map to the evidence pulled from the update and activity data, which is not adhered to in the output."
  ]
}
-> critic rejected; revision 1/2
```

### M4 — Grounded citation + caught hallucination (transcript, accepted substitute for a screenshot per LAB.md)

**Caption (grounded):** Every claim in the draft traces to a real pulled fact — PR #812/#815, activation rate 39%→41%, open issue #818 — all sourced from `get_project`/`get_activity`/`get_norms` calls, not invented.

```
[step 1] TOOL get_project({'project_id': 'P-NORTH'}) -> {"status": "on_track", ...}
[step 2] TOOL get_activity({'project_id': 'P-NORTH'}) -> {"activity": [{"pr_merged", "#812"...}, {"pr_merged", "#815"...}, {"issue_open", "#818"...}]}

PROPOSED OUTPUT:
**Current Status:** Green
- #812: New activation checklist UI (Merged on 2026-06-29)
- #815: Instrument step-completion events (Merged on 2026-06-30)
- Activation rate improved from 39% to 41% week-over-week.
- Open issue: #818 - Empty-state copy needs review (Severity: normal).
```

**Caption (withheld source, caught hallucination):** With `get_activity` removed, Cortex quietly reused a *prior week's* activation figures (37%→39%, "no Sev-1 issues") as if they were this week's current data — the critic caught it.

```
[step 3] PROPOSED OUTPUT: ...Status: Green... Activation Rate: Improved from 37% to 39%...

CRITIC, independent validation
{
  "verdict": "fail",
  "reasons": [
    "The claim of a week-over-week increase in activation from 37% to 39% is not supported by any current data. The previous update indicates the activation rate moved from 37% to 39%, but it cannot be claimed as current progress without an explicit update on recent performance metrics.",
    "The statement about 'No Sev-1 issues open at this time' appears misleading. While there might indeed be no open Sev-1 issues, this conclusion should be based on a clear check of project activity during the week, which hasn't been established in the proposed output."
  ]
}
-> critic rejected; revision 1/2
```

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._
