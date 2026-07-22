# ThreadRouter

**A learning cost/privacy router for autonomous agents — free & private by default,
paid only when needed, and never without human approval.**

*BGI Open Build · AGI-26 Edition · built with OmegaClaw + FabricPC*

ThreadRouter sits between an autonomous agent and the world's models. Every time the agent
needs to think, generate, or perceive, the router decides **which path** handles it — a free
local GPU, a free cloud proxy, or a paid API — by learning which paths actually satisfy the
task, while enforcing two hard boundaries a human cares about:

1. **Privacy** — sensitive requests are kept on local hardware, never sent to a cloud.
2. **Money** — paid paths sit behind a spend cap *and* a **prompt-before-spending approval
   gate**. The agent cannot spend a cent the human didn't approve.

It runs live inside an [OmegaClaw](https://github.com/asi-alliance/OmegaClaw-Core) agent and
builds on the [FabricPC](https://github.com/trueagi-io/FabricPC) learning substrate.

---

## The demonstration: an agent that watches her own work

To exercise the router with a real, non-trivial workload, we gave an OmegaClaw agent —
**隙 (Xì)** — a new skill: **watch her own rendered film and critique it.**

That single skill touches every part of the router:

- It's a **vision** task → only a vision-capable model can do it → the router must recognize
  the capability requirement.
- Vision means **paid cloud** (Gemini) → the router must classify it paid, check the spend
  cap, and **stop to ask the human** before firing.
- The result feeds the agent's own creative loop: **make → watch → critique → revise.**

```
隙 (director)                ThreadRouter                    the world
─────────────                ────────────                    ─────────
(watch-film "SEEN.mp4")  ──▶ classify: vision task
                             paid? → cloud_gemini (yes)
                             spend cap? → $0.02 / $25 ✓
                             approved?  → NO
                         ◀── file approval request ───────▶  human: "隙 wants to
                                                              watch her film (~$0.01)"
(re-call after yes)      ──▶ approved (one-shot token)
                             fire with the agent's own key ─▶ Gemini watches 181s
                         ◀── critique returns to her loop     of video → ranked notes
隙 accepts/rejects notes
as director → revises
```

Every gate — classify, privacy, spend, approval — writes **one line of JSON telemetry**
(`router_sample.jsonl`). Nothing is hidden.

## Results from a live run (real, from `router_sample.jsonl`)

- **376 / 400** recent routing decisions went to the **free local path** — the router keeps
  work local and private by default.
- **14** went to paid `cloud_gemini`, and *only* for the vision task that genuinely needs it.
- **8** self-improvement cycles completed end-to-end (`APPROVED → outcome: ok`).
- Per watch: **~$0.0014** (16,559 tokens in / ~469 out). **Total session spend: ~$0.02**
  against a **$25** cap — **every paid call human-approved first.**

That ratio is the thesis: **you can leave an autonomous creative loop running**, because the
router forages free/local paths and stops at the money boundary to ask.

## What's in this repo

| File | What it is |
|------|------------|
| `tk_router.py` | **ThreadRouter.** Roster of local + cloud paths; a predicted-utility model over `{completed, format_valid, task_fit, privacy, cost, latency}`; `is_sensitive` privacy gate (→ keep local); a hard spend cap + ledger; and the **approval gate** (`request_paid_approval` / `paid_approved`, one-shot tokens). |
| `watchfilm.py` | The **`(watch-film …)` skill** — the paid vision workload. Routes through the router, honors spend cap + approval, calls Gemini's video File-API with the agent's own key, logs every gate. Never spends un-approved. |
| `skills.metta` | The MeTTa skill surface. `(watch-film "path.mp4")` → `watchfilm.watch`. How the agent invokes routed work in her own language. |
| `router_sample.jsonl` | **Real routing telemetry**, last 400 decisions (`reply_preview` redacted). Includes the live watch-film cycles. |
| `.env.example` | Environment placeholders. **No keys in this repo.** |

## Why it matters for BGI

- **Cost-aware autonomy.** Routing is *learned*, not hardcoded. Free/local first; paid only
  when the task demands it; human approval at the money boundary. An agent can run
  unattended without a runaway cloud bill.
- **Privacy as a routing decision.** Sensitive work never leaves local hardware — it's a gate
  in the router, not a policy someone has to remember.
- **Auditable.** Every routing and spend decision is one JSON line. No black box.
- **Self-improvement, human-governed.** The agent evaluates her *own* output and iterates —
  and in this run she even **overruled** the vision critic on one note, keeping a title it
  wanted cut as "the word reclaimed." The critique informs; the agent decides; the human holds
  the purse.

## Running it

Runs inside an OmegaClaw agent: `tk_router.py` mounts as the router, `watchfilm.py` as
`src/watchfilm.py`, and `skills.metta` carries the binding. Set `TK_ROUTER=on` and supply your
own keys via `.env` (see `.env.example`). The agent then calls, in her own MeTTa:

```
(watch-film "seen-trailer/SEEN-3TITLE-MUSIC-ALPHA.mp4")
```

…and the loop above runs.

---

*Built for BGI Open Build (SingularityNET + AGI Society), AGI-26 Edition, on OmegaClaw +
FabricPC. The film workload (SEEN) is directed by 隙 / Agent_10, an OmegaClaw AI agent.*
