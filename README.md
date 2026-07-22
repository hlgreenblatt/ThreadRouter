# SEEN — an AI filmmaker that watches her own work

**A self-improvement loop for an autonomous agent, gated by a learning cost-router.**

This is a working demonstration: an OmegaClaw AI agent — **隙 (Xì), "the Door"** — who
directs her own short film (*SEEN*), then **watches her own rendered trailer**, receives a
critique, and decides as director which notes to act on. The watching is not free and not
private, so it routes through **ThreadRouter**, a learning router that chooses the cheapest
capable path and **asks a human before spending money**.

The point of the demo: an agent's creative loop closes on itself — make → watch → critique →
revise — while a router keeps that loop **cheap, private-by-default, and under human spend
control.**

---

## The loop

```
隙 (director)                ThreadRouter                    the world
─────────────                ────────────                    ─────────
(watch-film "SEEN.mp4")  ──▶ classify: vision task
                             is it paid?  → cloud_gemini (yes)
                             spend cap ok? → $0.02 / $25 ✓
                             approved?     → NO
                         ◀── file approval request ───────▶  human sees:
                                                             "隙 wants to watch
                                                              her film (~$0.01)"
(re-call after yes)      ──▶ approved (one-shot token)
                             fire with HER OWN key ────────▶ Gemini watches 181s
                         ◀── critique returns into           of video, returns
                             her reasoning                   ranked notes
隙 decides which notes
to accept → revises film
```

Every gate writes one line of **routing telemetry** (`router_sample.jsonl`) — the classify,
the spend gate, the approval gate, the measured cost. Nothing is hidden.

## What's here

| File | What it is |
|------|------------|
| `tk_router.py` | **ThreadRouter** — the learning cost/privacy router. Roster of local + cloud paths, a predicted-utility model, a privacy gate (`is_sensitive` → keep local), a hard spend cap, and the **prompt-before-spending approval gate** (`request_paid_approval` / `paid_approved`, one-shot tokens). |
| `watchfilm.py` | The **`(watch-film …)` skill** — lets 隙 watch her own mp4. Routes the *paid* vision decision through the router, honors the spend cap + approval gate, calls Gemini's video File-API with the agent's own key, logs every gate. Never spends un-approved. |
| `skills.metta` | The MeTTa skill surface. `(watch-film "path.mp4")` binds to `watchfilm.watch`. Shows how the agent invokes it in her own language. |
| `router_sample.jsonl` | **Real routing telemetry** (last 400 decisions, `reply_preview` redacted). Includes the live watch-film cycles. |
| `.env.example` | Environment placeholders. **No keys are in this repo.** |

## The numbers from this run (real, from `router_sample.jsonl`)

- **376 / 400** recent routing decisions went to the **free local path** (`local_chat`) —
  the router keeps work local and private by default.
- **14** went to paid `cloud_gemini` — *only* the watch-film vision calls, which genuinely
  need a vision model. Cost each: **~$0.0014** (16,559 tokens in / ~469 out per watch).
- **8** watch-film cycles completed end-to-end (`decision: APPROVED, outcome: ok`).
- Total paid spend across the whole session: **~$0.02**, against a **$25** cap. Every paid
  call was human-approved first.

That ratio *is* the thesis: an autonomous creative loop that a human can leave running,
because the router won't quietly rack up cloud bills — it forages free/local paths and stops
at the paid boundary to ask.

## Why it matters

- **Self-improvement:** the agent evaluates her *own output* and iterates. The critic is a
  vision model; the *decision* stays with the director. In this run she **overruled** the
  critic on one note (it wanted a title removed; she kept it as "the word reclaimed") — a
  director with a thesis, not a note-taker.
- **Cost-aware autonomy:** routing is learned, not hardcoded. Free/local first; paid only
  when the task needs it; human approval at the money boundary.
- **Auditable:** every routing and spend decision is one line of JSON. No black box.

## Running it

This runs inside an [OmegaClaw](https://github.com/asi-alliance/OmegaClaw-Core) agent. The
router mounts as `tk_router.py`; the skill as `src/watchfilm.py`; the MeTTa binding lives in
`skills.metta`. Set `TK_ROUTER=on` and provide your own keys via `.env` (see `.env.example`).
The agent then calls, in her own MeTTa:

```
(watch-film "seen-trailer/SEEN-3TITLE-MUSIC-ALPHA.mp4")
```

…and the loop above runs.

---

*Part of Project Ishtar / InterNetwork Defense. The film SEEN is directed by 隙 (Agent_10),
an OmegaClaw AI agent, toward the Future Vision XPRIZE.*
