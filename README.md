# TokePal

A coding companion ecosystem with many collectible species that hatch, grow, and evolve from real developer activity.

> **Buddy is a creature you meet. TokePal is a creature you raise.**

TokePals live in your terminal. You start with an egg. As you code, it grows. Over time it hatches into one of many species, levels up, evolves, and earns cosmetics — all driven by how you actually work, not by a separate game loop bolted on the side.

**Privacy stance:** TokePal runs locally. Nothing leaves your machine. No accounts, no telemetry — ever.

---

## Table of Contents

- [What It Is](#what-it-is)
- [Design](#design)
- [Early Thinking](#early-thinking)
  - [Core Loop: XP and Currency](#core-loop-xp-and-currency)
  - [Eggs](#eggs)
  - [Species Discovery and Evolution](#species-discovery-and-evolution)
  - [Cosmetics](#cosmetics)
  - [Trading](#trading)
  - [Breeding](#breeding)
  - [Additional Features Worth Building](#additional-features-worth-building)
  - [Economy Warning](#economy-warning)
- [Roadmap](#roadmap)

---

## What It Is

TokePal is a dev-native collectible creature system. You code, your TokePal grows. It is not a game layered awkwardly on top of a coding tool — it is a lightweight companion that sits inside the developer workflow and reflects real activity back at you as personality, progress, and collection depth.

The central mechanic: **token usage fuels XP and a soft currency**. XP makes a TokePal grow. Currency lets you participate in the economy (cosmetics, shop items, eggs). Keeping these separate matters — if growth and spending are the same resource, users feel punished every time they buy something.

Species, evolution, and traits tie progression to *how* you work, not just *how much*. A debugging-heavy user and a shipping-heavy user should not end up with identical creatures.

---

## Design

V1 is designed as a dev-native collectible creature system that blends terminal charm with polished 2D sprite identity. Instead of feeling like a full game layered awkwardly on top of a coding tool, the experience should feel lightweight, modern, and naturally embedded in the developer workflow: users receive an egg, hatch one of many species through real coding activity, and watch their creature grow, level up, and evolve over time.

The visual direction combines simplified ASCII companions for in-session presence with expressive 2D sprites for hatch reveals, profiles, and collection views, creating a balance between hacker-friendly personality and collectible appeal.

The overall tone should be slightly mischievous, retro-inspired, and highly legible, with subtle idle motion, clean progress indicators, and compact creature cards that highlight species, level, rarity, and evolution without overwhelming the user with game-like complexity.

---

## Early Thinking

This section captures the initial design reasoning behind TokePals — what the system should do, what it should avoid, and why. It is intentionally exploratory. Treat it as the design log, not the spec.

### Core Loop: XP and Currency

TokePals gain XP from token usage should absolutely be a core mechanic. It is elegant, intuitive, and directly tied to real behavior. But raw token spend cannot be the only driver, or users will just realize they are feeding a meter. The stronger version is: token usage is the main fuel, but the system also has drops, milestones, rarity, events, and choices layered on top.

A strong foundation:

- Token usage → **XP** (growth) + **soft currency** (spending).
- Soft currency could be Token Dust, Shells, Bits, or Embers.
- XP makes a TokePal grow; currency lets users interact with the economy.

That separation is important. Growth and spending should not be the exact same thing, otherwise users feel punished for buying cosmetics or eggs because they are sacrificing progression. Caps and balancing on both sides.

### Eggs

Eggs need to feel meaningful and a little scarce. Sources:

- Onboarding
- Milestone rewards
- Streaks
- Level thresholds
- Rare drops after sessions
- Referrals
- Seasonal events
- "Completed project" moments

Eggs should not be purely buyable from day one — hatching loses its magic if you can just buy it. Good pattern: every user starts with one egg, then earns more through progress. A marketplace or shop can later offer specific egg types or cosmetic-only variants.

### Species Discovery and Evolution

This is the emotional core. TokePals should not just level; they should change in ways that reflect how the user works. V1 can start simple, but the architecture should support branching evolution later:

- Debugging-heavy users
- Shipping-heavy users
- Frontend users
- Infra goblins
- Chaotic refactorers

That makes people care far more than if all creatures are just reskinned badges.

### Cosmetics

Cosmetics are a very good idea, but they need to be framed correctly. Categories:

- Skins
- Accessories
- Idle effects
- Aura colors
- Hatch effects
- Frames
- Nameplates
- Background habitats
- Alternate ASCII expressions

Cosmetics create desirability without breaking balance. Make them **species-specific** where possible — a "glitch aura" for a Glitchling or a "terminal-green shell" for a shell-type creature is more interesting than generic hats slapped on everything.

### Trading

Trading is probably strong, but not immediately. It creates community and flex value, but also farming, bots, alt-account abuse, and economy distortion. Do not launch with a full free market.

Start with lighter social mechanics:

- Gifting eggs
- Limited swaps
- Friend bonuses
- Shared hatch events
- Team incubators

That captures the social energy without instantly forcing a game-economy solution.

### Breeding

Interesting, but not for early versions. Breeding implies inheritance, rare combinations, and long-term collection depth — but also genetics, cooldowns, balance, rarity inflation, exploits, storage, inventory logic, and optimization behavior. The risk is turning a charming dev progression system into a spreadsheet meta-game too early.

**Good V3 idea. Bad V1 idea.**

A simpler early substitute: **fusion events** or **paired incubations** — two TokePals generate a special egg or a temporary variant, without a full breeding system.

### Additional Features Worth Building

- **Trait rolls and personality tags.** Each TokePal could have 2–4 traits like Curious, Chaotic, Meticulous, Sleepy, Gremlin, Stable, Speedy. Affects flavor, idle animations, or evolution paths. Adds uniqueness within the same species.
- **Session reactions.** After a coding session, the TokePal reacts: happy bounce, sleepy face, excited spark, level-up burst. Tiny feature, big attachment impact.
- **Milestone rewards.** At levels 5, 10, 20, etc., users get something: an egg, a cosmetic, a title, a background, a species shard, an evolution item, or a special interaction. Level-ups should never feel empty.
- **Collections and dex.** A Pokédex-style collection log. Even a simple "species discovered / evolutions unlocked / rare variants owned" view creates long-term goals.
- **Limited events and seasonal drops.** One of the best retention levers later. Halloween eggs, bug week, refactor festival, open-source month.
- **Creature pages and cards.** Every TokePal should have a shareable card with species, level, traits, rarity, and evolution stage. This is where social spread happens.
- **Light habitats.** Not a whole Tamagotchi room, but a background or biome: terminal cave, repo forest, merge swamp, cloud citadel. Habitats can be collectible cosmetics too.

### Economy Warning

**Do not reward raw token burn too aggressively.** If the fastest path is spamming prompts or wasting tokens, people will game it and the system will quietly rot.

A healthier formula: token usage contributes a lot, but XP and currency are shaped by session quality, milestones, caps, or diminishing returns. Genuine use should feel rewarding. Spam should not.

---

## Roadmap

Every version is evaluated against a single sentence: *does this strengthen "raise, don't meet"?* If not, it's cut or deferred. The full plan with stop-gates and rationale lives in [docs/ROADMAP.md](docs/ROADMAP.md). Open questions and decisions live in [docs/DECISIONS.md](docs/DECISIONS.md). A step-by-step V0–V1 walkthrough for collaborators is in [docs/PLAN.md](docs/PLAN.md).

### V0 — Foundations

Decisions and scaffolding before user-facing code: Claude Code hook as the surface, local-only storage, token ingestion from session transcripts, draft XP formula, behavior classifier sketch.

### V1 — The Relationship

Prove the core loop before adding anything else.

- Single egg → hatch → one creature
- Token usage → XP → leveling
- **Behavior-driven evolution** (debug-path vs. ship-path) — the moat
- Persistent state: days-alive, session count, streak
- `tokepal status` with idle animation and level-up burst
- 2 species at first evolution threshold; no currency, no cosmetics, no social

**Stop gate:** dogfood 2 weeks. If users don't voluntarily check on their creature in week 2, fix V1 before moving on.

### V1.5 — Depth of Identity

- 3–5 species with distinct ASCII identities
- Expanded evolution tree (4–6 end-states)
- Trait rolls / personality tags
- Session reactions
- Creature card (terminal block + exportable PNG)
- Collection / dex view
- Milestone rewards (eggs at levels 5 / 10 / 20)

### V2 — Social + Light Economy

- Soft currency introduced (only now, after the relationship is proven)
- Species-specific cosmetics
- Web profile / shareable card URL
- Gifting eggs, friend bonuses, shared hatch events
- Seasonal / limited events
- Team incubators

### V3+ — Deferred

Breeding / fusion, full marketplace, cross-user economy, real-money cosmetics.

### Art strategy (V1 cost: zero)

CC0 sprites from [OpenGameArt](https://opengameart.org/content/cc0-resources) → programmatic ASCII conversion via [`image-to-ascii`](https://www.npmjs.com/package/image-to-ascii) → ship both forms. Custom sprites later via [Piskel](https://www.piskelapp.com/) once the collection outgrows CC0 starters.

---

## License

See [LICENSE](LICENSE).
