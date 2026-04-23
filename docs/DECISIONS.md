# TokePal — Open Questions & Decisions

Living doc. Each entry has a status, the current answer, and the reasoning so future revisits don't re-litigate settled points.

---

## 1. Surface — where does TokePal live?

**Decision:** Split capture from display.

- **Capture (silent):** Claude Code hook writes session data to `~/.tokepal/state.json`. No output in the Claude Code session. TokePal never fights `/buddy` for screen space.
- **Display (user-chosen):** The user opens a viewer wherever they want.
  - V1: `tokepal status` (on-demand snapshot in any terminal) and `tokepal watch` (live-updating TUI in a dedicated pane or second terminal).
  - V1.5+: optional menu-bar app (macOS) / tray icon (Win/Linux).
  - V2+: optional local web dashboard (`tokepal serve`) for richer card views and sharing.

**Why:**
- Auto-spawning terminal windows is invasive, cross-platform messy, and reads as malware-like behavior to devs. Hard no on that as a default.
- Decoupling capture from display means TokePal works for tmux users, multi-monitor users, menu-bar-only users, and "I only check it sometimes" users without picking one winner.
- The hook stays silent and invisible in Claude Code — no visual competition with `/buddy`.

**Deferred:** Standalone CLI for non-Claude-Code sessions, VS Code extension, auto-spawn-window flag (`tokepal watch --spawn`) as opt-in for users who want it.

---

## 2. Token source — where do token counts actually come from?

**Decision:** Claude Code session hook (`SessionEnd` or equivalent). Read the transcript, sum input + output tokens, classify tool-call mix.

**Why:** Transcripts are the authoritative record of a session. No API keys, no polling, no external service.

**Open:** Exact hook name and transcript schema across Claude Code versions. Lock down during V0 implementation.

---

## 3. Behavior classification — how do we tell debug vs. ship vs. refactor vs. explore?

**Decision:** Classify from tool-call distribution in each session's transcript.

**Signals (draft):**
| Label | Signals |
|-------|---------|
| Debug-heavy | Failed `Bash` calls, test-run commands, error strings in tool results, repeat edits to the same file after failures |
| Ship-heavy | `Edit` / `Write` dominant, new files created, `git commit` near session end |
| Refactor-heavy | Many edits to existing files, low new-file ratio, rename patterns |
| Explore-heavy | `Read` / `Grep` / `Glob` dominant, few writes |

**Approach:** Store rolling tallies per session. Evolution branch at the first threshold picks the dominant label. Do *not* reclassify retroactively — your creature reflects who you were when it evolved.

**Open:** Exact weights, tie-breaking rules, minimum-session filters. Tune during dogfood.

---

## 4. Local-only vs. server-backed

**Decision:** Local-only through V1 and V1.5. `~/.tokepal/state.json`. No telemetry. No phone-home. No account.

**Why:**
- Dev-audience trust. "Nothing leaves your machine" is a first-class feature, not a footnote.
- Zero ops cost.
- Simplifies privacy posture to a single sentence.

**V2:** Introduces an optional, minimal sync endpoint for social features only (card sharing, gifting). Non-social state stays local. See [ROADMAP.md](ROADMAP.md) V2 prerequisites.

---

## 5. Multi-machine identity

**Decision:** One creature per machine in V1–V1.5. Single-device is a stance, not a limitation.

**Why:** Cross-device sync requires auth, a server, and a migration path. None of that is worth the complexity before the core loop is validated.

**V2:** Export/import lets users move a creature to a new machine manually. Full cross-device sync stays deferred.

---

## 6. Privacy stance

**Decision:** Local-only, zero telemetry, explicit in README.

**Posture (for public-facing copy):**
> TokePal reads your Claude Code transcripts on your machine. Nothing leaves it. No accounts, no telemetry, no analytics — ever.

**Why:** Developer trust is hard to earn and cheap to lose. Commit to the stance publicly so it's structurally hard to walk back.

---

## 7. Relationship to Anthropic's `/buddy`

**Decision:** Complementary, non-overlapping. TokePal does not claim, replace, or wrap `/buddy`.

**Positioning line:** *"Buddy is a creature you meet. TokePal is a creature you raise."*

**Why:**
- Buddy has distribution we can't match; fighting it loses.
- The products have different emotional promises (moment vs. relationship). Position on difference, not feature-count.

---

## 8. Art pipeline

**Decision:** CC0 source sprites → programmatic ASCII conversion → ship both forms.

**Stack:**
- **Sprites:** [OpenGameArt.org CC0 collections](https://opengameart.org/content/cc0-resources) — specifically "32x32 pixel art creatures", "Basic Green Monster", "Pixel Sci-Fi Monster", "50 monochromatic 32x32 critters".
- **Also useful:** [itch.io CC0 asset packs](https://itch.io/game-assets/free/tag-creatures), [Truly Truly Public Domain](https://opengameart.org/content/truly-truly-public-domain) collection on OGA.
- **ASCII conversion:** [`image-to-ascii`](https://www.npmjs.com/package/image-to-ascii) npm library (configurable palette, ANSI color) or [`ascii-art`](https://github.com/khrome/ascii-art).
- **Custom sprite authoring (later):** [Piskel](https://www.piskelapp.com/) — free, open-source, browser-based.

**Why:** V1 art cost is zero. Collection depth scales with CC0 availability. We only commission originals once the collection outgrows what CC0 can cover distinctively.

**Attribution:** CC0 requires no attribution, but we'll credit anyway in `docs/ATTRIBUTIONS.md` (future) as a courtesy.

---

## 9. Monetization

**Decision:** Free. Indefinitely for V1–V2.

**Why:** Free removes the largest barrier to adoption in a dev-tool audience and keeps the product honest — we're not optimizing for spend, we're optimizing for the relationship. Revenue model is a V3+ conversation.

---

## 10. Naming / trademarks

**Decision:** Not worrying about this yet.

**Note for later:** Before any public launch, run a basic trademark check on "TokePal" and adjacent terms. "Toke" has connotations in some markets — worth a smell test at launch time, not now.

---

## 11. Distribution channel

**Decision:** Claude Code plugin registry as the primary channel. Secondary: npm (if we ship a standalone CLI later) and Homebrew.

**Why:**
- Plugin registry is where the target audience is already looking.
- Aligns with the hook-based surface decision (#1).
- Zero-config discovery — users who run `/buddy` will encounter TokePal adjacent.

---

## 12. Validation plan

**Decision:** Dogfood group (small, private) for 2 weeks after V1 ships. Do not build V1.5 until the V1 → V1.5 gate in [ROADMAP.md](ROADMAP.md) passes.

**Why:** The only honest test of "does this feel like a relationship" is *time*. No A/B test substitutes for whether people voluntarily run `tokepal status` in week 2.

---

## Still genuinely open

These don't have decisions yet and need input before V0 implementation:

- **Exact Claude Code hook API surface** — need to verify which hooks fire, what data they provide, and whether token counts are in the transcript by default.
- **XP curve tuning** — draft formula exists; real numbers come from dogfood.
- **Evolution thresholds** — what level does first evolution trigger? Gut says level 5–8 but needs playtest.
- **Failure modes** — what happens if the state file is corrupted? If the hook fails silently? Decide before V1 ships, not after.
