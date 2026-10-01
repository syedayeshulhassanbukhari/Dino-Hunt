# Remaining tasks

**Rebuilt 2026-10-01** from the [GDD coverage audit](09-gdd-coverage.md) — read against the
live place, not against the old plan. The tracker
[`Dino-Hunt-Tasks.xlsx`](../Dino-Hunt-Tasks.xlsx) now agrees with this file; it grew from 75
tasks to **109** because the audit found both more done and more missing than the old plan
described.

---

## Where things stand

| Status | Tasks | Days |
|---|---|---|
| **Done** — verified by playtest | **41** | 22.25 |
| **Built (unverified)** — code exists, never observed running | **24** | 13.40 |
| **Not started** | **40** | 25.10 |
| Cut | 4 | — |
| | **109** | **62.00** |

**~38.5 days remain**, of which **13.4 is verifying code that already exists.** That is the
cheapest third of the remaining work and the most valuable: unverified code is of unknown
value, and nine of the GDD's fifteen acceptance items are currently in exactly that state.

Remaining by phase: P0 1.1 · P1 0.25 · P2 0 · P3 0.25 · **P4 5.8 · P5 8.05 · P6 10.55 · P7 12.5**

---

## Priority 1 — Make what exists trustworthy (6.0 days)

Highest value per day in the project. Nothing new is built; a large amount becomes real.

| ID | Task | Est |
|---|---|---|
| **DH-076** | **Wire species to their named rigs** — 20 rigs sit unused; 15 species share 4 grey-box bodies | 0.25 |
| **DH-077** | **Dev commands to reach each unverified system** — `/sector N`, `/survivor`, `/nest`, `/egg`, `/downed`, `/class X` | 0.50 |
| DH-078 | VERIFY exploration window — pause, 150s, warnings, safe recall | 0.25 |
| DH-079 | VERIFY sector gating — boss **and** Amber, plus the late-fill case | 0.25 |
| DH-080 | VERIFY survivor rescue — find, escort, stuck recovery, safe room, team credit | 0.25 |
| DH-081 | VERIFY nest and egg carry — kill **and** distraction both work | 0.25 |
| DH-082 | VERIFY incubation survives reconnect | 0.25 |
| **DH-083** | **VERIFY idempotent pet grant** — highest data-integrity risk in the project | 0.50 |
| DH-084 | VERIFY persistence across disconnect; duplicate reward IDs rejected | 0.50 |
| DH-085 | VERIFY downed, revive, next-wave respawn, −15% credits | 0.25 |
| DH-086 | VERIFY classes and the three abilities | 0.25 |
| DH-087 | VERIFY perk draft and stack limits | 0.25 |
| DH-089 | HUD visual review — done on functional evidence only; nobody has looked | 0.25 |
| DH-103 | **Multiplayer test, 2+ players** — nothing has ever run with two | 0.50 |
| DH-088 | Playtest bosses 2–5 — all defined, only boss 1 ever fought | 1.00 |

**Do DH-076 and DH-077 first.** Wiring the rigs makes six identical boxes distinguishable, so
every later observation about target priority becomes trustworthy. The dev commands turn each
verification from a full run into a minute.

---

## Priority 2 — Decisions that block work (0 days of build, needed now)

| ID / Decision | Question |
|---|---|
| **DH-099 / D-17** | **The weapon roster.** 5 guns exist and *none* are the GDD's. Its 20 elemental weapons, heat-instead-of-reload and charge mechanics do not exist. Either adopt the spec or write down the replacement — this is a design call, not a gap to fill one gun at a time. |
| **D-16** | **Two people, one live place.** No branching, no merging, no locking. Blocks both developers. See [08-status.md](08-status.md) §4. |
| D-14 / DH-109 | Rename the place — "Dino Hunt" collides with "Dino Hunters" (26.9M visits). |
| D-15 | Authored per-wave events are printed and ignored. They are the GDD's own answer to repetition. |
| D-12 | Creature rigging approach — partly answered by the rigs that have appeared. |

Two design questions nobody has answered: **should a lone player be able to hold a lane?**
(14 of 17 creatures reached the reactor on wave 1 while being shot at), and **should a boss
attack the reactor?** (Boss 1 chewed ~2,360 of 5,000 HP over a 187s fight).

---

## Priority 3 — Missing GDD systems (Phase 7, 12.5 days)

Everything the spec asks for that was never started.

| ID | Task | Est | Why it matters |
|---|---|---|---|
| **DH-090** | `AnalyticsService` + the 12 GDD event types | 1.00 | §11.1 lists it as required. **No KPI in §12.2 is measurable without it.** |
| **DH-092** | First-session onboarding, 8 steps | 1.00 | Named a retention lever by developers in the [market scan](07-market-scan.md) |
| DH-091 | FPS/TPS camera toggle | 1.00 | §3.4 is nine requirement rows; none built |
| DH-093 | Barricade, Ammo Crate, Shock Mine | 1.00 | Barricade is named in acceptance item 6 |
| DH-094 | Per-species behaviours | 1.50 | Gallimimus rush, Oviraptor theft, Carnotaurus camouflage, Stegosaurus sweep, Pteranodon dive |
| DH-100 | Build out the gun roster | 2.00 | Blocked on DH-099 |
| DH-101 | Three melee weapons + stamina | 1.00 | Zero melee exists; stamina is wholly new |
| DH-102 | Elemental status system | 1.00 | Carries the "every weapon has a job" pillar |
| DH-095 | Difficulty modes — Recruit / Standard / Nightmare | 0.50 | §9.5, no references anywhere |
| DH-096 | Egg mutations | 0.50 | §9.2 |
| DH-097 | Hunter Level, Weapon Mastery, Class Mastery | 1.00 | §9.4 |
| DH-098 | Monetization scaffolding | 1.00 | §9.6, cosmetic-only |

---

## Priority 4 — Hardening (3.5 days)

| ID | Task | Est |
|---|---|---|
| DH-104 | Performance pass — 50+ FPS on reference mobile at wave 20 | 0.75 |
| DH-105 | Mobile and aspect-ratio test | 0.50 |
| DH-106 | Exploit pass — forged hits, forged rewards, duplicate grants | 0.75 |
| DH-107 | Stabilisation buffer — **reserved, do not plan work into it** | 1.00 |

---

## Priority 5 — Process (1.1 days)

| ID | Task | Est |
|---|---|---|
| **DH-108** | **Split `UI.Hud` into per-panel modules** — prerequisite before two people add UI | 0.50 |
| DH-109 | Rename the place | 0.10 |
| DH-004 | Selene and StyLua config (can run against exported source) | 0.25 |
| DH-009 | Commit the skeleton | 0.25 |

---

## A suggested order

1. **DH-076** wire the rigs · **DH-077** dev commands — half a day, unlocks everything below
2. **DH-078 → DH-087, DH-089** the verification sweep — 3 days, converts 24 built tasks into known-good
3. **DH-099 / D-16 / D-14** the decisions — no build time, but they shape what follows
4. **DH-103** multiplayer test — six acceptance items depend on it
5. **DH-090** analytics · **DH-092** onboarding — the two the market scan says decide retention
6. **DH-088** bosses 2–5 · **DH-094** per-species behaviours — the content that makes 20 waves feel different
7. Weapons, then the long tail: difficulty, mutations, mastery, monetization
8. Hardening last

### If two people split it

Per [08-status.md](08-status.md) §4, and only after **DH-108**:

- **Person A:** Priority 1 verification sweep, then Phase 7 systems (analytics, onboarding, camera)
- **Person B:** weapons track (DH-099 → DH-100 → DH-101 → DH-102) — mostly `Config.Weapons`
  and `CombatService`, barely touching anything in Person A's path

The verification sweep is hard to parallelise — it is one person playing and reading logs — so
Person B taking weapons keeps them out of the way while it happens.
