# TokePal — Build Plan & Collaboration Walkthrough

> This doc is written for a collaborator (and their agent) joining the project.
> Read it top-to-bottom. Every step has a **Decided** section (what's settled) and an **Open** section (what needs your input). At the end there's a consolidated list of open questions — please share thoughts on each.

---

## Context

TokePal is a terminal-based collectible creature system. You code → your creature grows. It differentiates from Anthropic's built-in `/buddy` by being **persistent, behavior-driven, and relational** rather than a cute-but-stateless Easter egg.

**One-line positioning:** *Buddy is a creature you meet. TokePal is a creature you raise.*

Before diving in, skim:
- [README.md](../README.md) — what the product is
- [ROADMAP.md](ROADMAP.md) — V0 → V2 plan with stop gates
- [DECISIONS.md](DECISIONS.md) — settled decisions with reasoning

This doc zooms in on **V0 and V1** — the near-term actionable work.

---

## Architecture at a glance

```
  ┌─────────────────────────┐         ┌───────────────────┐
  │ Claude Code session     │ ──────▶ │ ~/.tokepal/       │
  │   (SessionEnd hook)     │  write  │   state.json      │
  │   silent, no output     │         │   transcripts/    │
  └─────────────────────────┘         └─────────┬─────────┘
                                                │ read
                                                ▼
              ┌─────────────────┬───────────────────────┐
              │                 │                       │
    ┌─────────▼────────┐  ┌─────▼──────┐    ┌──────────▼──────────┐
    │ tokepal status   │  │ tokepal    │    │ (V1.5+) menu bar    │
    │ on-demand snap   │  │ watch      │    │ (V2+) web dashboard │
    └──────────────────┘  │ live TUI   │    └─────────────────────┘
                          └────────────┘
```

**Key split:** capture (silent hook) vs. display (user-chosen). Never auto-spawn windows. The user picks the viewer; the hook just accumulates state.

---

## V0 — Foundations (no user-visible product yet)

### V0.1 — Verify the Claude Code hook surface

**Goal:** Confirm we can actually get token counts and tool-call data from a `SessionEnd` (or equivalent) hook.

**What we'll do:**
- Write a minimal hook that logs `{ session_id, timestamp, tokens_in, tokens_out, tool_calls[] }` to a file.
- Install it locally. Run a few real sessions. Verify data is complete and reliable.

**Decided:**
- Surface is a Claude Code hook, not a standalone daemon or extension.
- Hook is silent — no output in the Claude Code session.

**Open:**
- Which hook exactly? `SessionEnd`? `Stop`? Something else? We need to check the current Claude Code hook catalog.
- Is the transcript directly accessible to the hook, or do we need to parse it from disk?
- Are token counts present in hook context, or do we have to re-derive them from the transcript?
- What happens if a session ends abnormally (crash, Ctrl-C)? Is there a hook for that?

**Collaborator question:** Have you already worked with Claude Code hooks? Any gotchas you've hit — silent failures, schema drift between versions, permission issues?

---

### V0.2 — Design the state schema

**Goal:** Decide the shape of `~/.tokepal/state.json` before writing anything to it. Schema changes hurt later.

**Proposed schema (draft):**

```json
{
  "version": 1,
  "creature": {
    "id": "uuid",
    "name": "…optional nickname…",
    "species": "egg | blob | glitchling | …",
    "stage": "egg | hatched | evolved",
    "level": 1,
    "xp": 0,
    "xp_to_next": 100,
    "created_at": "iso-8601",
    "hatched_at": "iso-8601 | null",
    "last_seen": "iso-8601"
  },
  "behavior_tallies": {
    "debug": 0,
    "ship": 0,
    "refactor": 0,
    "explore": 0
  },
  "stats": {
    "total_sessions": 0,
    "total_tokens": 0,
    "days_alive": 0,
    "current_streak": 0,
    "longest_streak": 0
  },
  "history": [
    { "session_id": "…", "ended_at": "…", "xp_gained": 42, "label": "ship" }
  ]
}
```

**Decided:**
- Local JSON file only, no database.
- Single creature per machine for V1 (no multi-creature inventory yet).

**Open:**
- Do we cap `history` length, or keep all sessions forever? (All sessions = biggest, but enables cool retrospective views.)
- Is `name` user-editable via `tokepal rename`, or derived from species? Gut: user-editable — makes the creature feel owned.
- Do we store raw transcripts at all, or only extracted labels? Storing raw = re-labelable if classifier improves; not storing = smaller + more private.

**Collaborator question:** Any fields you'd add or drop? Anything here that'll bite us in V1.5 when species/traits expand?

---

### V0.3 — Behavior classifier prototype

**Goal:** Prove we can actually label a session as debug/ship/refactor/explore from hook data — without this, the whole "raise, don't meet" moat evaporates.

**What we'll do:**
- Take 10–20 real sessions from dogfood.
- Hand-label each one (gut check: what was I doing?).
- Try a rule-based classifier using tool-call mix + error signals + commit presence.
- Measure agreement between classifier and hand label. Target >70% before moving on.

**Draft rules:**

| Label | Primary signal | Secondary signal |
|-------|----------------|------------------|
| Debug-heavy | >3 failed `Bash` calls OR test-run commands with non-zero exit | Error strings in tool results, repeat-edits-after-failure |
| Ship-heavy | `git commit` near session end OR >3 new files created | High `Edit`/`Write` count, low `Read` count |
| Refactor-heavy | Repeated edits to same files, low new-file count | Rename patterns, symbol-wide replaces |
| Explore-heavy | `Read`/`Grep`/`Glob` dominant (>60% of tool calls) | Few writes, no commits |

Tie-breaking: whichever has the highest normalized score. Floor: if no label scores above threshold, mark `neutral` and don't change tallies.

**Decided:**
- Rule-based classifier first, not ML. Fast to ship, fast to explain, easy to tune.
- Labels are applied once at session end and never revised retroactively.

**Open:**
- What's the minimum session size to count? (5 tool calls? 1000 tokens?)
- Should `neutral` still grant XP but skip behavior tallies? (Probably yes — users shouldn't be punished for a reading session.)
- Are 4 labels enough, or do we need more (e.g., `testing`, `docs`, `config`)? Gut: 4 is enough for V1; expand if needed in V1.5.

**Collaborator question:** Anything you'd measure instead? Do you have instincts about signals specific to your own coding sessions that the above would miss?

---

### V0.4 — Art pipeline spike

**Goal:** Prove we can go from CC0 sprite → ASCII render in a few lines of code, so the visual side isn't a surprise cost later.

**What we'll do:**
- Pick one CC0 creature sprite from OpenGameArt (e.g., "Basic Green Monster" or "Animated Pixel Slug").
- Run it through [`image-to-ascii`](https://www.npmjs.com/package/image-to-ascii) at a few sizes.
- Tune palette + width until it looks good in an 80-column terminal.
- Save both forms (PNG + ASCII) side by side.

**Decided:**
- CC0 sprites → programmatic ASCII conversion → ship both forms.
- V1 art cost = $0. Custom art comes after V1.5 once the CC0 well runs dry.
- Target: 24–32 char wide ASCII for in-terminal display.

**Open:**
- Monochrome ASCII or ANSI-colored ASCII? Colored is prettier; monochrome is universally readable (no-color terminals, piped output).
- Do we want animation (multi-frame ASCII cycle) from day one, or start static? Static is simpler but less alive.
- Do we pre-render ASCII at build time (checked into repo) or runtime (`image-to-ascii` called on first run)? Pre-render is faster + offline-friendly; runtime lets us tune per-terminal width.

**Collaborator question:** Do you have opinions on ASCII aesthetic — simplified/cute (Tamagotchi-ish) vs. detailed/gritty (roguelike)? The choice affects which source sprites we gravitate toward.

---

## V1 — The Relationship

V1 is where the product actually exists. The bar is high: after 2 weeks, the creature should feel like yours. If dogfood users don't voluntarily check on it in week 2, V1 is not done.

### V1.1 — Egg and hatch

**Goal:** First run creates an egg. Enough XP hatches it into a creature.

**Flow:**
1. Hook runs → no state file → create one with `stage: egg`.
2. Subsequent sessions add XP silently.
3. When `xp >= hatch_threshold`, set `stage: hatched`, pick a species (V1: only one option per evolution branch; picked at first *evolution*, not hatch), record `hatched_at`.
4. Next time user runs `tokepal status`, they see the hatched creature + a "just hatched!" banner.

**Decided:**
- Hatch happens automatically based on XP, not on user command.
- The hatch reveal is deferred until the user next opens the viewer — it's a surprise, not an interruption.

**Open:**
- What's the hatch threshold? A few hundred tokens? A full coding day? A week? Affects first-impression emotional weight. Gut: 1–2 sessions of real use.
- Does the egg itself have any visual progression (hairline cracks as XP grows), or is it static until it pops? Progression is more satisfying; static is simpler.

**Collaborator question:** How long should a user wait before the first hatch? Too quick = feels arbitrary. Too slow = they give up. What's your instinct?

---

### V1.2 — XP and leveling

**Goal:** Post-hatch, sessions keep adding XP and creatures level up with a visible curve.

**Draft formula:**
- `xp_gained = min(floor(tokens_used / 100), 1000)` per session
- `xp_to_next(level) = 100 * level^1.5`

**Decided:**
- Per-session cap prevents grinding / token-burn gaming.
- Level-up events are detected at hook time, surfaced at next viewer open.

**Open:**
- Tuning numbers above are guesses. Real numbers come from dogfood.
- Do we want session-quality multipliers (streaks, variety of tool use) on top of the base formula? Adds depth; risks making the formula opaque.
- Hard level cap, or unbounded?

**Collaborator question:** Do you want raw token count to be the only XP driver, or should session quality matter? E.g., a session with a successful commit grants a small bonus.

---

### V1.3 — Behavior-driven evolution (the moat)

**Goal:** At the first evolution threshold (draft: level 5), the creature evolves into one of several species based on the user's dominant behavior label so far.

**Flow:**
1. At each session end, increment the dominant-label tally for that session.
2. When `level == 5` (or whatever threshold), evolution triggers.
3. Pick the highest tally: debug → Glitchling, ship → Shippermon, refactor → Refactorab, explore → Wanderlith (placeholder names).
4. Set `species`, bump `stage: evolved`, record evolved_at.

**Decided:**
- Evolution reflects who you were *up to* the threshold. No retroactive reclassification.
- V1 has 2 branches minimum (to ship), V1.5 expands to 4+.

**Open:**
- What exactly is the threshold — level, days alive, session count, or a combination?
- Can a user "steer" their evolution (e.g., see their current dominant label before evolution triggers), or is it a surprise? Tradeoff: visibility respects the user; surprise is more magical.
- What if tallies are tied or near-tied? Coin flip? User choice? A rare "hybrid" species?

**Collaborator question:** Do you want evolution to be deterministic (same behavior → same species every time) or include a rarity roll (same behavior → usually species A, sometimes rare species B)?

---

### V1.4 — Persistence: days-alive, streaks, sessions

**Goal:** The creature has a history you can see. This is the single biggest structural differentiator from `/buddy`.

**Decided:**
- `days_alive` = calendar days since `created_at`.
- `current_streak` = consecutive days with at least one session ending.
- `longest_streak` tracked separately.
- Last-seen shown in status ("last session 3 hours ago").

**Open:**
- What counts as "a day" — local time zone or UTC? Time zone is nicer but dev tools traditionally use UTC.
- Streak break grace period — does missing one day break the streak immediately, or is there a 1-day forgiveness? Forgiveness = kinder; no forgiveness = more meaningful streaks.
- Do we show any "neglect" state (creature looks tired / sleepy) after long absence, or is the creature state-agnostic to time-gaps? Neglect state is emotionally strong but edges toward guilt mechanics.

**Collaborator question:** How do you feel about gentle neglect signals? E.g., if no session in 3 days, status shows a sleepy version. Emotional or annoying?

---

### V1.5 — The `tokepal status` command

**Goal:** One command. Shows everything that matters. This is the V1 product.

**Draft output:**

```
  ╭──────────────── Kipper ────────────────╮
  │                                        │
  │              ⌒⌒⌒                       │
  │             ( ^ω^)          Glitchling │
  │              /  \           Level 7    │
  │                                        │
  │  XP: ▓▓▓▓▓▓▓▓░░░░░░░░  612 / 1000      │
  │  days alive: 18    streak: 5           │
  │  last session: 2h ago                  │
  │                                        │
  ╰────────────────────────────────────────╯
```

**Decided:**
- Single-screen output, no paging.
- Works in a non-color terminal (degrades gracefully).
- Idle animation handled by `tokepal watch` (the live TUI), not `status`.

**Open:**
- Exact box-drawing aesthetic — heavy (╭╮), light (┌┐), ASCII-only (+-|)? Affects portability vs. polish.
- Do we show behavior tallies ("mostly shipping lately") or keep that hidden for evolution surprise?
- Does `status` have flags (`--json`, `--quiet`, `--one-line`) for scripting use cases?

**Collaborator question:** Anything missing from the draft output? Anything there that feels unnecessary?

---

### V1.6 — The `tokepal watch` live TUI

**Goal:** A dedicated pane for users who want the creature visible while they work.

**Decided:**
- Live-updates state changes (XP gain, level up, evolution) without manual refresh.
- Subtle idle animation (2–3 frame ASCII cycle, slow).
- Level-up burst on state transition.
- Same content as `status` but reactive.

**Open:**
- Refresh mechanism — file-watch on state.json, or polling every N seconds? File-watch is cleaner on macOS/Linux, flakier on Windows.
- Do we show a session-end pop-in when a hook just fired ("+42 XP gained from last session")? Satisfying but risky if it lags the actual session end.

**Collaborator question:** Have you built TUIs before? Any opinion on the stack — Ink (React for CLIs), blessed, bubbletea (Go), textual (Python), or vanilla ANSI? The choice gates contributor ramp-up.

---

## Consolidated Open Questions

Quick index so you don't have to scroll. Please share thoughts on any of these:

### Technical
1. Exact Claude Code hook to use for capture (V0.1).
2. Whether to store raw transcripts or only extracted labels (V0.2).
3. History retention — cap or unbounded (V0.2).
4. Pre-render ASCII at build time vs. runtime (V0.4).
5. State file refresh mechanism for `watch` — file-watch vs. polling (V1.6).
6. TUI stack choice (V1.6).

### Product / design
7. Monochrome vs. ANSI-colored ASCII (V0.4).
8. Static vs. animated ASCII from day one (V0.4).
9. ASCII aesthetic — cute vs. gritty (V0.4).
10. Hatch threshold — how long to first hatch (V1.1).
11. Egg visual progression vs. static egg (V1.1).
12. Session-quality multipliers on XP (V1.2).
13. Hard level cap or unbounded (V1.2).
14. Evolution threshold — level / days / session count / combo (V1.3).
15. Transparent vs. surprise evolution direction (V1.3).
16. Deterministic vs. rarity-rolled evolution (V1.3).
17. Time-zone for "day" calculation (V1.4).
18. Streak-break grace period (V1.4).
19. Neglect state (sleepy creature after absence) — emotional or annoying? (V1.4).
20. Show behavior tallies in status, or keep hidden (V1.5).
21. Box-drawing aesthetic for status output (V1.5).
22. `status` flags for scripting (V1.5).

### Strategic
23. Number of initial species in V1 — 2 branches minimum, but how many total?
24. Dogfood group — who's in it? How do we recruit?
25. First public surface — internal dogfood only, or invite-beta from the start?

---

## What we'd like from your agent

When your agent reads this doc, please have it:

1. **Flag open questions it has opinions on** — even weak ones help us converge.
2. **Raise anything we missed** — gaps, risks, bad assumptions.
3. **Push back on the "raise, don't meet" framing** if it thinks we're overselling the differentiation. We'd rather hear it now.
4. **Propose a starting point** — given the V0.1–V0.4 scaffolding, which does your agent think is the safest first thing to build?

We'll consolidate both sides' answers and update [DECISIONS.md](DECISIONS.md) with the next wave of settled calls.
