# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you, by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

The validator (critic) was the toughest part of the build — tuning its checks, revision cap, and watching it reject drafts for subtle reasons (an invented risk-level, a conflated project, a stale metric) across nearly every real run took more iteration than any other piece.

## Learning

- The agent can be dialed in such a way that it fits different personas (the Autonomy Dial per segment), not just one global trust setting.
- The agent can be fine-tuned to meet specific project objectives rather than running on generic defaults.

## Aha moment

Having multiple agents each do one specific task, instead of one main agent trying to do everything, would likely be more robust than the single-drafter-plus-critic design Cortex ended up with.

## What you'd do differently

Start with who Cortex is for and what problem is actually being solved for them (the user segments and their needs) before building out the mechanics — that would have saved real cycle time spent fine-tuning the agent's behavior later.
