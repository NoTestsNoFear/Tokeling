# TokePal Roadmap

> **Positioning:** Buddy is a creature you meet. TokePal is a creature you raise.

Every version below is evaluated against that single sentence. If a feature doesn't strengthen *raise*, it is cut or deferred.

---

## V0 — Foundations

Decisions and scaffolding before any user-facing code.

1. **Surface:** Claude Code hook (plugin). Lowest install friction, complementary to `/buddy`, reads session transcripts directly. See [DECISIONS.md](DECISIONS.md) for rationale.
2. **Storage:** Local JSON at `~/.tokepal/state.json`. No server, no account, no telemetry.
3. **Token ingestion:** `SessionEnd` (or equivalent) hook reads Claude Code transcript, sums input + output tokens, writes to state.
4. **XP formula (draft):** `xp_gained = min(floor(tokens_used / 100), session_cap)` where `session_cap` is ~1000 to prevent gaming. Tune after dogfood.
5. **Behavior classifier (draft):** Label each session from tool-call mix in the transcript:
   - **Debug-heavy** — many failed `Bash` calls, test runs, error messages in tool results
   - **Ship-heavy** — `Edit`/`Write` dominant, `git commit` at end, new files created
   - **Refactor-heavy** — repeated edits to same files, low new-file ratio
   - **Explore-heavy** — `Read`/`Grep`/`Glob` dominant, few writes
   - Store rolling tallies; evolution branch picks the dominant label at threshold.

---

## V1 — The Relationship

**Goal:** After 2 weeks of use, the creature feels like yours.
**Stop gate:** If users aren't attached after dogfooding, do not proceed to V1.5.

1. Single egg on first run; hatches into one creature at first XP threshold.
2. Token usage → XP → leveling with a visible curve.
3. **Behavior-driven evolution** — at least 2 branches at the first evolution threshold (debug-path vs. ship-path). This is the moat.
4. Persistent state across sessions: days-alive, total sessions, current streak, last-seen.
5. `tokepal status` — ASCII creature, name, level, XP bar, days-alive, streak.
6. Subtle idle animation (2–3 frame cycle) + level-up burst.
7. One hardcoded species per branch (2 total). No cosmetics, no currency, no shop.

*Out of scope for V1:* multiple species, collection view, cards, currency, cosmetics, social features.

---

## V1.5 — Depth of Identity

**Goal:** The creature isn't just leveling, it's becoming specific to you.

8. 3–5 species with distinct ASCII identities (sourced from OpenGameArt CC0, see below).
9. Expanded evolution tree (4–6 end-state creatures across branches).
10. Trait rolls — 2–3 personality tags per creature (Curious, Meticulous, Chaotic, Sleepy, etc.) affecting flavor text and idle behavior.
11. Session reactions — happy/sleepy/excited/focused based on session pattern.
12. Creature card — `tokepal card` produces a shareable terminal block + exportable PNG.
13. Collection / dex view — visible once the user has hatched more than one.
14. Milestone rewards — eggs granted at levels 5 / 10 / 20. First time new eggs enter the system.

---

## V2 — Social + Light Economy

**Goal:** Creatures connect to other people without becoming a marketplace.

15. Soft currency introduced (Token Dust / Shells / Embers). Only now — after the relationship is proven.
16. Species-specific cosmetics, small earnable set. No real-money shop.
17. Web profile / card URL for sharing outside the terminal.
18. Gifting eggs between users (requires light identity; see V2 prerequisites below).
19. Friend bonuses / shared hatch events.
20. Seasonal / limited events (Halloween eggs, bug-week creatures, etc.) — first real retention lever.
21. Team incubators for squads / orgs.

**V2 prerequisites (new infrastructure that didn't exist in V1):**

- Lightweight identity (GitHub OAuth, anonymous by default)
- Minimal sync endpoint for social features (cards, gifting) — everything non-social stays local
- Export format for creature state (so sync is additive, not migratory)

---

## Explicitly deferred to V3+

- Breeding / fusion mechanics
- Full trading marketplace
- Cross-user economy
- Real-money cosmetics
- Cross-device sync as a core feature (single-device is a stance in V1–V2)

---

## Art strategy

We don't need to draw anything from scratch to ship V1.

**Pipeline:**

1. Start with CC0 creature sprites from [OpenGameArt.org](https://opengameart.org/content/cc0-resources) (collections: "32x32 pixel art creatures", "Basic Green Monster", "Pixel Sci-Fi Monster", "50 monochromatic 32x32 critters").
2. Convert PNG → ASCII programmatically with [`image-to-ascii`](https://www.npmjs.com/package/image-to-ascii) (npm) or [`ascii-art`](https://github.com/khrome/ascii-art).
3. Ship both forms: ASCII in `tokepal status`, PNG in the creature card.
4. Author custom sprites in [Piskel](https://www.piskelapp.com/) (free, open-source) once the collection grows past CC0 starters.

This means V1 art cost = zero. We only commission originals when the collection needs identity beyond what CC0 can provide.

---

## Validation gates

Do not advance versions without passing the gate.

- **V1 → V1.5 gate:** Dogfood group uses TokePal for 2 weeks. Do they still `tokepal status` voluntarily in week 2? Do they reference their creature by name? If no, fix V1 before adding V1.5 features.
- **V1.5 → V2 gate:** Do users voluntarily share their creature card? Can they describe *why* their creature evolved the way it did? If no, behavior classification is too noisy — tune before adding social.
- **V2 gate:** Before opening any social features, verify no data leaves the machine in V1.5 and document the exact delta V2 introduces.
