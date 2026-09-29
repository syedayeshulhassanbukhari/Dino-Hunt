# Project status — and how two people work on this

**As of 2026-09-29.** Updated at the end of any session that changes the numbers.

Written because a second developer is joining. Sections 1–3 are the state of the work;
**section 4 is the part that matters for two people**, because this project has a structural
problem with parallel work that needs deciding before either of you starts.

---

## 1. Where the project is

| | Tasks | Est. days |
|---|---|---|
| **Done** | **40 / 75** | **21.0 / 40.4** |
| In progress | 4 | 2.65 |
| Not started | 27 | 16.75 |
| Cut | 4 | — |

**52% by effort.** Phases 0–3 are complete and playtest-verified. Phase 4 is half built.

| Phase | Done | State |
|---|---|---|
| 0 · Toolchain | 5/11 | Complete in substance — 4 tasks *cut* when Rojo/Wally were dropped |
| 1 · Blockout & creature | 11/11 | ✅ Verified |
| 2 · Wave loop | 10/10 | ✅ Verified |
| 3 · The spine | 11/11 | ✅ Verified |
| 4 · Boss & exploration | 3/12 | Boss built and retuned; **exploration not started** |
| 5 · Collection loop | 0/10 | Not started |
| 6 · Players & hardening | 0/10 | Not started |

> **"Days" are estimated developer-days, not elapsed time.** Phases 0–3 took about two
> calendar days of Claude Code sessions. Treat the estimates as relative sizing only — Phase 4
> already overran its own (the boss fight measured 187s against a 100s prediction).

---

## 2. What actually works right now

A playable run exists. You can press Play and:

- Survive **20 finite waves** with a 15s intermission between each, win at wave 20, or lose
  when the reactor dies.
- Shoot **four grey-box creature types** with server-validated hitscan, weak points and
  directional front armour that must be flanked.
- Watch **Amber** fly from each kill into the reactor and level the gauge through five levels.
- **Repair** the reactor for credits earned from kills.
- Fight **Boss 1**, the Alpha Raptor Matriarch: three phases, telegraphed pounce, hunter mark,
  summoned pack, flank window on a missed pounce.
- Jump the match clock with `/wave N` and force a loss with `/lose` (Studio only).

### Verified by playtest

| System | Evidence |
|---|---|
| Wave loop | Waves 1–5 cleared in sequence; wave 20 reached and cleared; VICTORY fired |
| Concurrency cap | `wave 2 ... held by cap on 16 tick(s)` — the first wave over the solo cap of 18 |
| Lane openings | Wave 20 reported `3 lane(s)`; waves 1–5 reported 1 |
| Reactor | `wave 1 — took 839 damage, peak 14 attacker(s), reactor 4160/5000` |
| Defeat | `reactor destroyed` → `DEFEAT on wave 19` |
| Amber | `REACTOR LEVEL 2 — 255 Amber (threshold 250)` at the predicted wave |
| Repair | Twice, with credit cost and cooldown |
| Boss phases | All three fired on health fractions; invulnerability capped at 3.0s of 8s |

### Known-good but **not** visually reviewed

The HUD is marked done on functional evidence — it mounts clean, zero client errors, and every
attribute it reads is verified server-side. **Nobody has actually looked at it critically.**
If the bars are cramped or mislabelled, that is unreported, not absent.

### Implemented but never observed

**DH-075 — creatures damaging players.** The code is in and load-checked, but across every
playtest the player never stood within `AttackRange`, so it has never once been seen to work.
Instrumentation was added for the next run. Treat it as unproven.

---

## 3. What is left

Grouped by what they touch, which is what matters for splitting work.

### Track A — finish Phase 4 (2.9 days)

| ID | Task | Touches |
|---|---|---|
| DH-046 | Boss 1 re-verify after retune | `Config.Bosses` |
| DH-045 | Boss health bar review | `UI.Hud` |
| DH-047 | Phase respawn (reward already verified) | `BossService` |
| DH-048 | `ExplorationService`: wave pause | **new**, `WaveService` |
| DH-049 | 150s timer, 30s/10s warnings | **new** |
| DH-050 | Safe recall at zero | **new** |
| DH-051 | Sector gating on boss **AND** Amber level | **new**, reads blockout attributes |
| DH-052 | Late gauge fill case | **new** |

**This is the moat.** [07-market-scan.md](07-market-scan.md) found nothing on Roblox combining
a losable objective with timed exploration gated on two conditions. Everything else we have
built is available as an off-the-shelf template.

### Track B — Phase 5, the collection loop (6.3 days)

Survivors, guarded nests, egg carry, hatchery, incubation, pet grant, Codex, profile keys.
Almost entirely **new files**: `SurvivorService`, `HatcheryService`, `DataService`.

Nominally depends on Track A (DH-053 and DH-056 list DH-050 as a dependency), but that
dependency is about *testing* inside an exploration window, not construction. The escort AI and
the nest guardian can be built and unit-tested before exploration exists.

**Highest data-integrity risk in the project**: DH-060, the idempotent pet grant. It writes
permanent inventory from a timer that must survive a reconnect.

### Track C — Phase 6, player systems (6.3 days)

**Unblocked right now.** Classes (DH-063) depends on DH-021 and downed state (DH-065) on
DH-020 — both long since done. Nothing in Phase 6's player half waits on Phase 4 or 5.

Classes, abilities, downed state, revive, next-wave respawn, run perks. Then hardening:
performance, mobile, exploit pass, and a 1-day stabilisation buffer that is reserved — do not
plan work into it.

---

## 4. Two people, one place — read this first

### The problem

**The source of truth is a single live Roblox place, edited over MCP. There are no branches,
no merges and no locking.** Two people editing `CreatureService` at the same time means one
silently overwrites the other. The `.rbxm` snapshot in `snapshot/` is a binary blob — git
cannot merge it either, so two people committing snapshots produces a conflict only one side
can win.

This is a direct consequence of dropping Rojo (decided 2026-09-28). It was the right call for
one person. **It does not scale to two without a protocol.**

### Raised as D-16 — needs a decision before either of you starts

Three workable answers, in order of preference:

**(a) Team Create + strict file ownership.** Enable Roblox Team Create on the place. Roblox
locks a script while someone has it open, which prevents the worst collisions. Then assign
*ownership by service* so you are rarely in the same file at all. One person runs the
snapshot commits.

**(b) Reinstate Rojo after all.** Code returns to the filesystem, git does the merging, and
both of you work normally. Costs the Phase 0 toolchain work that was cut (DH-001…004,
DH-007 — about a day) and changes the workflow everyone has learned. Genuinely the correct
long-term answer if this project runs for months.

**(c) Split by place.** One person owns the main place; the other works on separable assets
(creature rigs, map geometry, UI) in their own place and hands over `.rbxm` files. Zero
collision risk, but the second person cannot test anything in context.

### If you take (a), the ownership split that minimises collisions

| | Person A | Person B |
|---|---|---|
| **Track** | A then B — Phase 4 exploration, then Phase 5 collection | C — Phase 6 player systems |
| **Owns** | `WaveService`, `BossService`, `ExplorationService`, `SurvivorService`, `HatcheryService`, `ObjectiveService`, `Config.Waves`, `Config.Bosses` | `PlayerService`, `CombatService`, new `ClassService`, `Config.Weapons`, `StarterPack.Rifle` |
| **Shared — coordinate** | `CreatureService`, `Config.Creatures`, `Config.Game`, `UI.Hud` | same |

**Do this before splitting: break `UI.Hud` into one module per panel.** It is already the
worst hotspot — every feature adds to it — and it is a single 400-line file. `UI/WavePanel`,
`UI/ObjectivePanel`, `UI/BossPanel`, each mounted by `Hud`, means two people adding UI never
touch the same file. Half a day, and it removes the most likely source of lost work.

`CreatureService` is the other hotspot and cannot be split as cleanly. Rule of thumb: whoever
needs to change it announces it first.

---

## 5. Onboarding a second developer

Read in this order:

1. **[CLAUDE.md](../CLAUDE.md)** — conventions, invariants, where code lives. Short and
   load-bearing.
2. **[01-gdd-review.md](01-gdd-review.md)** — the spec's defects. Several requirements
   deliberately deviate from the GDD; this says which and why.
3. **[07-market-scan.md](07-market-scan.md)** — what we are competing with, and the one
   competitor whose player count fell 92% in two months.
4. **[03-open-decisions.md](03-open-decisions.md)** — what is undecided.
5. **[WORKLOG.md](WORKLOG.md)** — newest first. The bug entries are the useful part.

### Things that will catch you out

- **The user runs the playtests, not Claude.** Verification is by reading logs. Print lines
  are the test surface, so make them specific.
- **Load-check by cloning, never by requiring in place.** The Studio edit peer caches
  `require` per ModuleScript for the whole session — an edited module returns the stale table
  and never re-parses, so a syntax error passes a "load check" silently.
- **`TextChannel:SendAsync()` never returns for a command message.** Anything after it in the
  same thread is dead code.
- **A Cylinder part's X axis is its length**, Y and Z are the diameter. Getting this wrong
  stands a pillar up where you wanted a flat disc. It has cost time twice.
- **Ground-attack geometry must be checked against player speed.** A 12-stud telegraph with a
  1.2s warning is undodgeable-to-land, because a player covers 21 studs in that time. Every
  pounce in the first boss playtest missed for this reason.
- **Estimates derived from arithmetic have been wrong twice.** Effective player dps measured
  64, not the 120 derived from magazine duty cycle. Measure, then tune.

### The five invariants

Non-negotiable, inherited from the deleted codebase where they were load-bearing. All are
asserted at load where possible:

1. The wave-1 creature dies to **exactly one** starter-weapon shot.
2. **Every creature is slower than the player** (17.6) — anything can be outrun.
3. **Nothing one-shots a full-health player.**
4. Concurrency is `min(perPlayer × live, 60)` — **one server-wide cap**.
5. The server owns damage, ammo, currency, drops and saves. **The client never reports a hit.**

---

## 6. Open decisions

Full detail in [03-open-decisions.md](03-open-decisions.md).

| ID | Blocks | Summary |
|---|---|---|
| **D-16** | **Both developers** | How do two people share one live place? See §4. |
| D-14 | Publishing | The place is named "Dino Hunt"; "Dino Hunters" has 26.9M visits. Rename to *Primal Rift: Outpost Siege*. |
| D-15 | Nothing yet | Authored per-wave events (`Power outage 25s`, `No repair until clear`) are printed and ignored. They are the GDD's own answer to repetition — the named cause of a competitor's churn. |
| D-12 | Phase 5 art | Creature rigging approach: grey-box, asset pack, or Blender. |
| D-13 | Nothing | Package versions are pinned; Fusion 0.3 recorded. Effectively resolved. |

### Two design questions nobody has answered

- **Should a lone player be able to hold a lane?** 14 of 17 creatures reached the reactor on
  wave 1 while the player was actively shooting. Either that is intended — the reactor absorbs
  damage and repair is a core loop — or kill speed needs work. Decide it deliberately.
- **Should a boss attack the reactor?** Boss 1 walks to the ring like any creature and chewed
  ~2,360 of the reactor's 5,000 HP over a 187s fight. The GDD does not say. The pressure feels
  right, but it makes a long boss fight doubly punishing.
