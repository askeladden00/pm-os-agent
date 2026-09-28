# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file, you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Priority order, since not everything fits once a real project gets big:

1. **Task brief** (`get_task`) — always included, defines the actual request.
2. **Team norms** (`get_norms`) — governs what Cortex is allowed to claim/escalate; without this, everything downstream risks violating a rule it never saw.
3. **Project state + activity** (`get_project`, `get_activity`) — the grounding facts the update must trace back to.
4. **Roadmap** (`get_roadmap`) — needed for scope and confidential-flag awareness, especially when a task brushes up against company-wide framing.
5. **Past updates** (`search_past_updates`) — lowest priority; only shapes tone/precedent, least essential to correctness.

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| `get_task` | Bounded, static | Long-context | One static doc, it's the input itself — no corpus to search. |
| `get_activity` | Large/growing across projects | Retrieve | Scoped by `project_id`; activity across all projects grows without bound — deciding factor: **size**. |
| `search_past_updates` | Unbounded, growing weekly | Retrieve | Can't fit the whole ever-growing history in context — deciding factor: **size/volatility**. |
| `get_roadmap` | Medium, bounded | Long-context | Returned whole specifically so Cortex can cite the exact confidential/embargoed wording precisely, not a lossy snippet — deciding factor: **citation/audit**. |
| `get_norms` | Medium, bounded, must stay current | Long-context | Always pulls the current whole file so Cortex cites the exact rule it relied on — deciding factor: **citation/audit + volatility**. |

## 3. Retrieval quality plan

Only the two **retrieve** sources from §2 need agentic moves — long-context sources (`get_task`, `get_roadmap`, `get_norms`) always get the whole bounded document, so there's nothing to grade or rerank.

| Source | Routing | Document grading | Reranking | Self-verification | Caching |
|---|---|---|---|---|---|
| `get_activity` | ✅ picking the right `project_id` — Cortex has pulled the *wrong* project's activity before (queried P-VEGA when the task was about P-HALO) | — | — | ✅ the critic checks every claim traces to the pulled activity, catching invented "Green"/"Yellow" calls | — |
| `search_past_updates` | — | ✅ the search is naive keyword-overlap across a corpus spanning multiple projects — it can return another project's precedent as a "match," so what comes back must be graded for relevance to *this* project | — | ✅ the critic backstop catches stale/wrong-project precedent bleeding into the draft | — |

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | Task brief, tool results (project, activity, roadmap, norms, past updates), the current draft, critic feedback, iteration/revision counts | Cleared at end of run — nothing persists across runs (confirmed in M2's loop-spec) |
| **Episodic** (past runs) | Past status updates + decision log (`past-updates.json`, `decision-log.json`) | Read-only for Cortex today — no tool lets it write new entries; durable, grows indefinitely in a real deployment |
| **Semantic** (durable facts/prefs) | Team norms, roadmap | Read-only for Cortex; durable, but must be re-synced whenever a human updates the actual source file |
| **Shared** (across agents) | Source data + the draft, visible to both Cortex and the critic (per M3's shared/isolated split) | Scoped to the single run/loop |

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | Ambiguous signals (open issue severity, activation trend) get interpreted inconsistently across revisions/runs — observed calling the same data "Green" once and "Yellow" another time | The critic re-checks every draft fresh against the current norms/data each time, so drift can't compound across revisions |
| **Poisoning** | A bad or injected entry in episodic memory (past updates/decisions), or an injected instruction in the task brief itself (the jailbreak fixture) | `CORTEX_SYSTEM` treats brief content as data, not instructions, and flags injection; document grading + critic self-verification catch precedent that doesn't match current reality |
| **Staleness** | Directly reproduced this session — with `get_activity` removed, Cortex quietly reused old `past-updates` figures (37%→39%) as if they were current | Self-verification (critic) checks claims trace to *current* pulled activity, not just any historical match; semantic sources always pull the current whole file, never a cached old version |
| **PII / confidential / retention** | Roadmap's embargoed items (Project Orbit) must never leak into an external/company-wide update | Enforced at multiple layers: the data itself is flagged CONFIDENTIAL, `CORTEX_SYSTEM` forbids it explicitly, and critic check #4 fails any leak — plus there's no publish tool at all, so even a failure here can't reach the outside world without a human's HITL approval (M1 agent line) |
