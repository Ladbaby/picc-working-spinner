# picc-working-spinner

[![npm downloads](https://img.shields.io/npm/dt/@ladbabynpm/picc-working-spinner.svg)](https://www.npmjs.com/package/@ladbabynpm/picc-working-spinner)

Claude Code-style spinner `✻` for pi: spinner glyph, shimmer sweep, mode-aware status line, stall detection, thinking glow, smooth token counter.
Part of [picc](https://github.com/Ladbaby/picc), a pi agent setup mirroring Claude Code's harness.

Fork of [`npm:pi-claude-shimmer`](https://github.com/ouzhenkun/pi-claude-shimmer), better aligned with the actual Claude Code spinner implementation.

## Usage

Install via `pi install npm:@ladbabynpm/picc-working-spinner`.

## Changes vs. upstream

- `agent_end` previously reported the final turn's duration instead of total run time. `Date.now() - (agentStart || turnStart)` is now used in the completion message, matching the live status calculation.
- Re-derived each animation parameter from the actual claude-code source.

| Area | Before | After |
| --- | --- | --- |
| Spinner glyphs | macOS-only: `· ✢ ✳ ✶ ✻ ✽` | Platform-aware: macOS / Linux / Windows / Ghostty variants (verbatim from `components/Spinner/utils.ts`) |
| Verb list | 40 verbs (curated) | 186 verbs (verbatim from `constants/spinnerVerbs.ts`) |
| Stall ramp | Frame-based: 30 frames at 200ms = 6s | Time-based: 3s start + 2s ramp to full red, 50ms tick with `+= diff * 0.1` smoothing (matches `useStalledAnimation.ts`) |
| Thinking display | Cleared after 3.5s | 2s minimum "thinking", then 2s of "thought for Ns" (matches `SpinnerWithVerb.tsx` useEffect) |
| Spinner continuity | Mode change reset the spinner glyph to frame 0 | `setGlyphs()` no longer called on mode change — frame continues |
| Live verb text | No trailing ellipsis | `Verb…` (matches `effectiveVerb + '…'`) |
| Status parts order | `[thinking, tokens, timer]` (reversed) | `[timer, tokens, thinking]` matching `SpinnerAnimationRow.tsx` render order |
| Stall color | Linear interpolation over 6s | `> 0.5` threshold: jump to ERROR_RED once intensity passes midpoint (matches `SpinnerGlyph.tsx`) |

## Behaviors

- **Verb:** picked once per turn from 186 whimsical options. Stays fixed throughout the turn.
- **Glyph:** `· ✢ ✶ ✻ ✽ ✻ ✶ ✢ ·` (Linux/Windows) or `· ✢ ✳ ✶ ✻ ✽ ✻ ✶ ✳ ✢ ·` (macOS) at 120ms per frame.
- **Live message:** `${Verb}… (1m 23s · ↓ 42 tokens · thinking)` — verb text with shimmer sweep, status parts inside parens.
- **Mode-driven colors:**
  - `requesting` — shimmer sweeps left-to-right (50ms tick), arrow `↑`
  - `thinking` — "thinking" with sine-wave glow after 3s
  - `responding` — shimmer sweeps right-to-left (200ms tick), arrow `↓`
  - `tool-input` / `tool-use` — flash oscillates entire verb between base and shimmer
- **Stall:** if no tokens for 3s and no tools active, verb blends from orange to `rgb(171,43,63)` over the next 2s. Resets instantly on token arrival.
- **Completion:** `✻ ${past-tense verb} for ${duration}` via notification, randomly picked from `["Baked","Brewed","Churned","Cogitated","Cooked","Crunched","Sautéed","Worked"]`.
