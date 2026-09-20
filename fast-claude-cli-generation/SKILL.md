---
name: fast-claude-cli-generation
description: >
  Reduce latency when an app calls the `claude` CLI as a subprocess (Sonnet or any model) —
  faster AI turns / responses without changing the model. Use when generation "feels slow",
  a Claude-CLI-backed app lags, someone compares it unfavorably to a local LLM, or you're
  asked to speed up / optimize per-call LLM latency, time-to-first-token, or throughput.
  Logic and decision order only — not implementation. Distilled from a real optimization that
  cut median latency ~30% and killed random multi-second freezes with no quality loss.
date_added: "2026-07-15"
---

# Making `claude` CLI generation fast (the logic)

## Mental model — the tax, not the model

The slowness is almost never Sonnet the *model*; it's the **machinery around each call**. A
local LLM feels fast because it's warm, resident, and streams. A `claude -p` subprocess piles
avoidable taxes on the same model. Strip the taxes first; only then consider the model.

## Rule 0 — Benchmark before you touch anything

Do not optimize on intuition. It's easy to spend a day refactoring something that changes
latency by zero. Before and after every lever:

- **Replay real, heaviest prompts.** Log real `{system, user, response}` payloads and replay the
  worst-case ones (largest context the app actually produces). Never benchmark a toy prompt.
- **Measure two numbers:** time-to-first-token (what *feels* fast) and total wall-clock.
- **Report median AND p90.** The freezes hide in the tail; the mean lies.
- **Interleave variants** (A,B,C,A,B,C…), never in blocks — the first call of a session eats the
  cold-start and would frame whichever recipe ran first as slow.
- **Change one factor at a time** so you know which lever earned the win.

## The levers, in priority order

1. **Neutral working directory (biggest, cheapest win).** Spawn the subprocess in an *empty
   directory outside any repo*. Inside a repo the CLI loads that project's `CLAUDE.md` and
   enumerates `.claude/skills/` on **every boot** — pure dead weight the call never uses (it
   passes its own system prompt). This alone cut ~30% off median and removed the worst tail spikes.
2. **Lower reasoning effort.** The CLI's default effort is `high` (coding-tuned). For short or
   generative tasks (a chat line, a JSON decision, a summary) use `low`/`medium`. **Always verify
   quality** by diffing real outputs at each effort — in practice short-task quality is
   indistinguishable, and it tightens the latency tail.
3. **Warm process pool.** Keep a few pre-booted processes idle and feed prompts to them, skipping
   the ~1–2.5s CLI startup on a hit. Bake the chosen effort into the pooled process so warm +
   effort coexist (a per-call effort flag otherwise forces a cold spawn).
4. **Shorten the output.** Generation time is dominated by *output* tokens — this is the only
   prompt-side lever that actually moves latency. Tighten length caps where the task allows.
5. **Stream tokens for perceived speed.** If a human is watching, read incremental deltas and show
   text as it generates. Total time is unchanged, but perceived wait collapses to time-to-first-token.

## Traps (verified the hard way)

- **`--bare` breaks subscription auth.** It strips the login context with the project context, so
  the call returns "Not logged in" almost instantly — a *fake-fast* result that isn't a real
  generation. Use a **neutral cwd** to shed the repo tax while keeping auth.
- **Don't assume prompt caching helps — measure it.** On the CLI subscription path, repeated
  stable-prefix calls did **not** lower time-to-first-token (they got slower under load). We
  cancelled a large prompt-restructure that was justified only by this assumption.
- **Stacked retries multiply latency.** A retry loop inside another retry loop can 4× the worst
  case. Keep a single retry layer.
- **Respect the output-bound floor.** ~200 output tokens ≈ a few seconds, roughly the physical
  floor per call. You can't beat it per-call without a smaller model or streaming (perceived).
  If you have N *sequential* calls, wall-clock ≈ N × floor — attack the sequence (make it
  concurrent, opt-in, or shorter), not each call.

## Separate "faster" from "better"

Latency is usually **output-bound**, so trimming *input* (memory, context) is a **quality/cost**
play, not a speed play. Keep the two motivations separate: optimize context for sharper decisions,
optimize the machinery (cwd, effort, warm pool, streaming) for speed. Don't justify an input
refactor with a latency claim you haven't measured.

## The one-line formula

**Benchmark real heaviest prompts (median+p90, TTFT+total, interleaved) → neutral cwd → lower
effort (verify quality) → warm pool → stream if watched → shorten outputs. Never `--bare`, never
assume caching, never stack retries.**
