# Decisions

Blocking questions first, then the log of what has been settled.
When a decision is made, move it down with the date and the reasoning.

---

## Open

### D-01 — Match shape: 300 endless waves vs 20 finite

**Conflict.** The whole wave pipeline — table, generator, cycle multipliers, per-wave caps —
assumes an endless ladder across six difficulty cycles. Primal Rift's pacing, unlock gates
and KPI targets all assume a finite 28-38 minute run ending in victory at Wave 20.

Waves 1-20 of the existing table are not transferable either: existing Wave 1 spawns
**5 aliens**, the spec's Wave 1 wants **17 dinosaurs**.

**Options.** (a) Author a new finite 20-wave table in the same shape and retire the cycles.
(b) Keep the endless ladder and treat Wave 20 as a soft milestone rather than a win.

**Blocks:** all wave work, boss gating, exploration timing, every KPI.

---

### D-02 — Wave table ownership

**Conflict.** `ReplicatedStorage.WaveTable` is generated from `docs/AREA51_WAVE_TABLE.md` by
`tools/gen_wavetable.js`. Its header states plainly that hand-editing the numbers will drift
the design file and the game apart. **Neither the doc nor the tool is in this repo.**

**Options.** (a) Bring that generator repo into this project and keep the pipeline.
(b) Primal Rift formally takes ownership of the table in-place and the generator is retired.

Not both. Whichever is chosen, the WaveTable header comment must be rewritten to say so.

**Blocks:** D-01, and any edit to wave composition.

---

### D-03 — Defended objective

**Conflict.** There is no central objective in the place today, so the spec's primary fail
condition (reactor reaches zero HP) has no implementation. It also has no numbers: reactor
HP, per-enemy reactor damage and repair rate/cost are undefined in the spec too.

Enemies already carry a `gearDamage` stat for hitting structures, so the targeting change is
small. The state machine, failure flow and the numbers are not.

**Needs:** a decision to build it, plus the three missing numbers.

**Blocks:** sector gating, the Amber gauge, the entire loss condition.

---

### D-04 — The second, unused codebase

**Conflict.** `ReplicatedStorage.Shared` holds a parallel, largely dead scaffold from an
unrelated SWAT project: `Net`, `Validate`, `RateLimiter`, `Signal`, `Loader`, `Config`,
`Enums`, `WeaponConfig`, `GearConfig`, `AttachmentConfig`, `MissionConfig`, `Progression`,
`RankConfig`. Its GearConfig sells a "Standard-issue SWAT uniform" and carries phase markers
`P2-1` / `P4-5`. `Blaster.Constants` notes a field there exists but "no code reads it".

**Options.** (a) Adopt it as Primal Rift's service spine and migrate onto it.
(b) Delete it.

Leaving two shared layers in place is how the codebase becomes unnavigable.

**Blocks:** nothing immediately, but gets more expensive to resolve the longer it waits.

---

## Also unresolved — from the spec itself

These are recorded in [01-gdd-review.md](01-gdd-review.md) and need answers before the
systems that depend on them are built. They are not blocking today.

- Pacing budget overruns its own target (34-46 min against a stated 28-38).
- Amber thresholds are flat while income scales with party size.
- Boss HP scaling makes co-op easier than solo.
- Solo downed-state rules contradict each other.
- Weapon gating: by wave or by Hunter Level?
- Reactor HP, repair rate and cost — no numbers.
- Player defense progression — none exists across 20 waves.
- Late Amber gauge fill: does an exploration window open mid-wave?
- Parallel egg incubation: one hatchery slot or many?
- DNA carry and theft mechanics.

---

## Decided

### 2026-09-05 — Build target is the Dino Valley place

Primal Rift ships into the existing **Dino Valley** place, placeId `88638180383535`.
Confirmed by the user.

### 2026-09-05 — Reskin in place, incrementally

Keep `GameManager` / `WaveManager` / `ZombieAI` / `Blaster` and evolve them into the spec,
renaming and extending rather than rewriting.

**Reasoning:** fastest path to a playable dino build, and roughly half the spec's
architecture already exists. **Accepted cost:** the alien game is overwritten as we go, and
there is no preserved fallback copy.

### 2026-09-05 — Map before building

First work item is a system-by-system gap analysis rather than starting the vertical slice.
Delivered as [02-conversion-map.md](02-conversion-map.md).

### 2026-09-05 — Version control on GitHub

Repo is `syedayeshulhassanbukhari/Dino-Hunt`. This repo holds documentation, decisions and
design data — not a Rojo source sync. Place edits go through the Studio MCP bridge.
