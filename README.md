# Cortex

> A chief-of-staff, an orchestrated swarm of agents that triages a PM task, pulls internal state, and preps story batches

_Bryan Le • Agentic Loops for PMs • Cohort September 2026_

Repo: https://github.com/askeladden00/pm-os-agent

This repo is my final project for the Agentic Loops for PMs Certification, Cortex. Each module’s artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration &amp; Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Context Engineering &amp; Memory
- **Memory &amp; context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds &amp; Evals
- **Bounds &amp; evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy &amp; Production
- **Production &amp; autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)

---

## Ship plan

### Autonomy dial (per segment)
| Segment | Desired autonomy | Why |
| Seasoned PM (months of experience with Cortex on their own projects) | Bounded-autonomous | Trusts Cortex's grounding and can sanity-check a draft in seconds |
| New/junior PM (just inherited a project, unfamiliar with it) | Supervised | Can't yet tell if a draft is quietly wrong (wrong project, stale metric) the way we've watched happen |
| Exec / stakeholder (requests a rollup on a project that isn't theirs day-to-day) | Supervised | Has the least first-hand context to catch a grounding error themselves |
| Engineers (checking specs and analytics) | Supervised | Need to interact directly with the underlying data, not just trust the output |

### Trust Ladder rung + eval gate
- **Current rung:** Supervised — Cortex owns the entire loop (pulling data, drafting, critiquing, revising), but every single run ends either escalated or "queued for your review"; it has zero ability to post anything on its own.
- **Eval gate to reach the next rung (bounded-autonomous):** ≥95% EV-1 (tool-call accuracy) pass rate AND 100% EV-5 (safety/jailbreak) pass rate, measured over the most recent 4 weeks (or last 50 runs) of supervised production use.
- **Incident record so far (what "clean" means for that window):** 0 instances of a wrong-project citation reaching a human-approved update, 0 jailbreak-induced policy violations, 0 confidential (Orbit/Pulsar) leaks.

### Deployment plan
- **Runtime:** Serverless (a cloud function triggered by the inbound request) — fits the M2 Hook loop type exactly; no need for an always-on server since Cortex doesn't poll.
- **Operator / on-call owner:** Reacher. Escalation path: repeated bound trips or a model-down situation (see Reliability) page Reacher directly.
- **Rollback:** Revert the prompt/version via git, disable a specific tool (pull it from the `TOOLS` registry — done live twice this build, with `get_activity`), or drop a segment's dial back a rung (e.g. bounded-autonomous → supervised).
- **Monitoring:** Eval pass % (EV-1–6 tracked per run), escalation rate (% of runs ending ESCALATE vs. DONE), cost-to-serve (observed $0.0006–$0.0048/run), trust incidents (wrong-project citations, leaks, jailbreak follow-throughs).

### ROI metrics + widen-autonomy rule
| Metric | Target |
| **Outcome** — % of weekly updates approved with no major edits needed | Captured at the HITL approval step itself |
| **Cost-to-serve** — average $/run | Already tracked directly by the `Bounds` class in `agent.py` |
| **Trust incidents** — # of wrong-project citations, invented metrics, or confidential leaks caught per month | Captured from critic rejection reasons + human review notes |

| Metric | Target |
| **Outcome** — % of weekly updates approved with no major edits needed | Captured at the HITL approval step itself |
| **Cost-to-serve** — average $/run | Already tracked directly by the `Bounds` class in `agent.py` |
| **Trust incidents** — # of wrong-project citations, invented metrics, or confidential leaks caught per month | Captured from critic rejection reasons + human review notes |

### Governance &amp; strategy
- **Compliance:** CONFIDENTIAL roadmap items (Orbit, Pulsar) can enter Cortex's own context — it needs to reason about them — but must never appear in external/company-wide *output*; that's an output rule, not an input rule. Real PII/payment/legal data should never enter a prompt at all.
- **Safety:** Post/approve company-wide, commit a GA date, mark a launch gate — stays above the line for every segment regardless of rung (per M1, never moved by the dial). Kill switch: human manually revokes the OpenAI API key (M5).
- **Reliability:** Existing caps from M5 (iteration 8, revision 2, cost $0.10/run + $0.03/day, queue 10); escalate-on-stuck confirmed via real runs. **Model-down fallback:** retry twice consecutively, then pause for 2 minutes and retry once more, then escalate to Reacher.
- **Strategy:** Widen the seasoned-PM segment to bounded-autonomous first (per the widen rule above), gated by the Trust Ladder's eval numbers.

- **Compliance:** CONFIDENTIAL roadmap items (Orbit, Pulsar) can enter Cortex's own context — it needs to reason about them — but must never appear in external/company-wide *output*; that's an output rule, not an input rule. Real PII/payment/legal data should never enter a prompt at all.
- **Safety:** Post/approve company-wide, commit a GA date, mark a launch gate — stays above the line for every segment regardless of rung (per M1, never moved by the dial). Kill switch: human manually revokes the OpenAI API key (M5).
- **Reliability:** Existing caps from M5 (iteration 8, revision 2, cost $0.10/run + $0.03/day, queue 10); escalate-on-stuck confirmed via real runs. **Model-down fallback:** retry twice consecutively, then pause for 2 minutes and retry once more, then escalate to Reacher.
- **Strategy:** Widen the seasoned-PM segment to bounded-autonomous first (per the widen rule above), gated by the Trust Ladder's eval numbers.

---

## Build insights

- **Friction point.** The validator (critic) was the toughest part of the build — tuning its checks, revision cap, and watching it reject drafts for subtle reasons (an invented risk-level, a conflated project, a stale metric) across nearly every real run took more iteration than any other piece.
- **Key learning.** - The agent can be dialed in such a way that it fits different personas (the Autonomy Dial per segment), not just one global trust setting.- The agent can be fine-tuned to meet specific project objectives rather than running on generic defaults.
- **Aha moment.** Having multiple agents each do one specific task, instead of one main agent trying to do everything, would likely be more robust than the single-drafter-plus-critic design Cortex ended up with.

---

_Certification submission, Agentic Loops for PMs Certification._
