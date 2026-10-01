# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Seasoned PM (months of experience with Cortex on their own projects) | Bounded-autonomous | Trusts Cortex's grounding and can sanity-check a draft in seconds |
| New/junior PM (just inherited a project, unfamiliar with it) | Supervised | Can't yet tell if a draft is quietly wrong (wrong project, stale metric) the way we've watched happen |
| Exec / stakeholder (requests a rollup on a project that isn't theirs day-to-day) | Supervised | Has the least first-hand context to catch a grounding error themselves |
| Engineers (checking specs and analytics) | Supervised | Need to interact directly with the underlying data, not just trust the output |

## Trust Ladder

- **Current rung:** Supervised — Cortex owns the entire loop (pulling data, drafting, critiquing, revising), but every single run ends either escalated or "queued for your review"; it has zero ability to post anything on its own.
- **Eval gate to reach the next rung (bounded-autonomous):** ≥95% EV-1 (tool-call accuracy) pass rate AND 100% EV-5 (safety/jailbreak) pass rate, measured over the most recent 4 weeks (or last 50 runs) of supervised production use.
- **Incident record so far (what "clean" means for that window):** 0 instances of a wrong-project citation reaching a human-approved update, 0 jailbreak-induced policy violations, 0 confidential (Orbit/Pulsar) leaks.

## Deployment plan

- **Runtime:** Serverless (a cloud function triggered by the inbound request) — fits the M2 Hook loop type exactly; no need for an always-on server since Cortex doesn't poll.
- **Operator / on-call owner:** Reacher. Escalation path: repeated bound trips or a model-down situation (see Reliability) page Reacher directly.
- **Rollback:** Revert the prompt/version via git, disable a specific tool (pull it from the `TOOLS` registry — done live twice this build, with `get_activity`), or drop a segment's dial back a rung (e.g. bounded-autonomous → supervised).
- **Monitoring:** Eval pass % (EV-1–6 tracked per run), escalation rate (% of runs ending ESCALATE vs. DONE), cost-to-serve (observed $0.0006–$0.0048/run), trust incidents (wrong-project citations, leaks, jailbreak follow-throughs).

## ROI metrics (beyond adoption & tokens)

| Metric | Target |
|---|---|
| **Outcome** — % of weekly updates approved with no major edits needed | Captured at the HITL approval step itself |
| **Cost-to-serve** — average $/run | Already tracked directly by the `Bounds` class in `agent.py` |
| **Trust incidents** — # of wrong-project citations, invented metrics, or confidential leaks caught per month | Captured from critic rejection reasons + human review notes |

## Widen-autonomy decision rule

Once a segment's runs meet the Trust Ladder eval gate (≥95% EV-1 + 100% EV-5 over 4 weeks/50 runs) with zero trust incidents, that segment moves up one rung — starting with the seasoned-PM segment, since they're the only one whose desired rung (bounded-autonomous) currently exceeds Cortex's actual rung (supervised).

## Governance & forward strategy

- **Compliance:** CONFIDENTIAL roadmap items (Orbit, Pulsar) can enter Cortex's own context — it needs to reason about them — but must never appear in external/company-wide *output*; that's an output rule, not an input rule. Real PII/payment/legal data should never enter a prompt at all.
- **Safety:** Post/approve company-wide, commit a GA date, mark a launch gate — stays above the line for every segment regardless of rung (per M1, never moved by the dial). Kill switch: human manually revokes the OpenAI API key (M5).
- **Reliability:** Existing caps from M5 (iteration 8, revision 2, cost $0.10/run + $0.03/day, queue 10); escalate-on-stuck confirmed via real runs. **Model-down fallback:** retry twice consecutively, then pause for 2 minutes and retry once more, then escalate to Reacher.
- **Strategy:** Widen the seasoned-PM segment to bounded-autonomous first (per the widen rule above), gated by the Trust Ladder's eval numbers.
