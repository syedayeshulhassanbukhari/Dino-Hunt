# SDLC plan — Primal Rift MVP

**Version** 1.0 · **Written** 2026-09-06 · **Covers** Mon 7 Sep – Fri 2 Oct 2026 (20 working days)

Companion documents: [01-gdd-review.md](01-gdd-review.md) (spec defects),
[02-conversion-map.md](02-conversion-map.md) (service mapping),
[04-inventory.md](04-inventory.md) (what exists), [03-open-decisions.md](03-open-decisions.md)
(blocking decisions).

---

## 0. Read this first

**The full MVP as written in the GDD is not a one-month build.** Twenty creatures needing
rigs and animation sets, five multi-phase bosses, four sectors, five survivor escorts and the
complete collection loop is a multi-month content programme regardless of how the engineering
is organised.

What fits in one month — and is the correct engineering sequence anyway — is a **depth-first
vertical slice**: every system present and working end to end, with content deliberately thin.
One boss instead of five. One sector instead of four. Six species instead of fifteen. The GDD
itself recommends exactly this in its Phase 2 exit criteria.

This plan delivers a **playable, winnable, losable Primal Rift** by 2 October. Section 8 lists
precisely what is deferred and why.

---

## 1. Scope

### 1.1 In scope — the month's deliverable

A complete 20-wave Primal Rift run that can be won or lost, containing:

- Finite 20-wave match with victory and defeat states
- Reactor as the defended objective, with HP, repair and failure
- Amber currency, magnet delivery and the 5-level reactor gauge
- Boss framework, with **Boss 1 (Alpha Raptor Matriarch)** fully implemented
- Sector gating on the dual condition (boss cleared **and** Amber level reached)
- Exploration phase — wave pause, timer, warnings, safe recall
- **Sector 1** with hatchery, one survivor and one guarded nest
- Egg carry, incubation and permanent pet grant
- Minimal Dino Codex
- Three classes with passives and abilities
- Downed state and revive
- **Six** grey-boxed or final dinosaur species
- Currencies renamed and rebalanced: DNA, Credits, Amber

### 1.2 Out of scope this month

Deferred deliberately, not forgotten. See section 8 for reasoning.

Bosses 2–5 · Sectors 2–4 · Survivors 2–5 · Species 7–15 · Melee weapons and stamina ·
Full 20-gun rebalance · Mutations · Difficulty modes · Monetization · Weapon mastery ·
Nightmare mode · Final dinosaur art if the asset pack does not arrive

### 1.3 Assumptions

These are load-bearing. If one is wrong, the timeline moves.

| # | Assumption | If wrong |
|---|---|---|
| A1 | One developer working with Claude Code over the MCP bridge | More hands compresses weeks 3–4 |
| A2 | Dinosaur art is **grey-boxed** from existing rigs; final pack may arrive later | Art-first adds 1–2 weeks before anything is testable |
| A3 | **D-01** resolves to a finite 20-wave match | Endless mode invalidates weeks 1–2 |
| A4 | **D-02** resolves to owning the wave table in-place | Recovering the generator adds 1–2 days |
| A5 | 5 working days per week, 20 total | Pro-rata |
| A6 | The place remains the only source of truth; no Rojo sync | — |

### 1.4 Dependencies and blockers

- **D-01, D-02** must be answered before Week 1 Day 2. They gate all wave work.
- **D-03** (reactor numbers) must be answered before Week 2.
- Dinosaur asset pack: existence and location unknown. A2 assumes we proceed without it.

---

## 2. SDLC model

**Iterative, one-week sprints, each ending in a playable build.** Not waterfall — the spec has
known defects (see 01) and the codebase is inherited, so the plan must absorb discovery.

### 2.1 Phases per sprint

| Phase | Activity |
|---|---|
| Plan | Pick the sprint's FRs. Confirm Definition of Ready. |
| Design | For anything new, decide the data shape and where it lives before writing code. |
| Build | Read-before-write over MCP. One system at a time. |
| Verify | Studio play test, solo and multiplayer, against the FR's acceptance criteria. |
| Record | Update WORKLOG, decisions doc, export `src/`, commit and push. |

### 2.2 Working rules

1. **Read before write.** The place contains a large working codebase. Never edit a script
   that has not been read in full this session.
2. **Numbers are derived, never hand-tuned.** Every enemy stat descends from its `lp` weight.
   A bare number with no derivation comment fails review.
3. **One-way export.** Before and after each sprint, export every script from the place into
   `src/`, and commit. This is the only version control the game code has (**D-06**).
4. **Server owns truth.** No client-reported hits, rewards or currency.
5. **Comment the why.** Match the existing codebase: what changed, when, and who asked.
6. **The decisions doc is the change log.** No silent scope changes.

### 2.3 Definition of Ready

A requirement may enter a sprint when it has an ID, acceptance criteria, its numbers decided,
and no unresolved blocking decision.

### 2.4 Definition of Done

- Acceptance criteria met in a Studio play test
- Verified **solo and with 2+ players**
- No new errors in the output console across a full wave
- Server-authoritative; the client cannot forge the outcome
- Numbers live in `Params` with a derivation comment
- `src/` exported, WORKLOG updated, committed and pushed

### 2.5 Testing strategy

| Level | Method | When |
|---|---|---|
| Unit-ish | Luau executed over MCP against the live data model | During build |
| System | Studio Play Solo, targeted wave via the existing skip-wave script | Per FR |
| Integration | Studio Start Server + 2 clients | Per sprint |
| Performance | Max-wave load, mobile device emulation, FPS and heartbeat | W4, and W2 spot-check |
| Security | Attempt forged remotes: fake hits, fake rewards, duplicate egg grants | W4 |
| Regression | Full 20-wave run | End of every sprint from W2 |

---

## 3. Module status

Three categories, as requested.

### 3.1 Already developed — use as-is

| Module | Location | Note |
|---|---|---|
| Combat authority | `GunFireServer`, `Blaster.Scripts.Blaster` | Server-side hit validation, fire rate, ammo |
| Directional armour | `Blaster.Scripts.Blaster` | 35° facing block — the Triceratops mechanic, already shipping |
| Weapon framework | `ReplicatedStorage.Blaster` | 27 guns, view models, recoil, heat, touch input |
| Camera | `FirstPersonController`, `MobileCamera`, `CameraRecoiler` | FPS/TPS, mobile, ADS |
| Persistence | `NinjaProfileStore` | Session-locked profiles, lobby loadout contract |
| Analytics | `Telemetry`, `ExitAnalytics` (6 modules), `RunTracker` | Funnels, exit reasons, HTTP sender |
| Pathfinding throttle | `ZombieAI.Pathfinding` | Global token budget |
| Stuck recovery | `StuckAlienRecovery` | Pattern to copy for survivor escorts |
| Concurrency cap | `ZombieSpawner` | `min(perPlayer × live, 60)` — already correct |

### 3.2 Reuse and refactor

| Module | Change required | Est. |
|---|---|---|
| `Params` | Replace 26 alien rows with dinosaur rows; keep the `lp` derivation | 2d |
| `WaveTable` | Fork to a finite 20-wave Primal Rift table; retire cycle multipliers | 2d |
| `WaveManager` / `ZombieSpawner` | Boss waves, victory at 20, exploration pause hook | 2d |
| `ZombieAI` | Add flight, water traversal, grab; rename to `DinoAI` | 4d |
| `Economy` | gems → DNA, coins → Credits; add Amber as run-scoped | 1d |
| `AreaUnlockServer` / `AreaOccupancy` | Re-gate on boss + Amber level; the paid walls were removed 2026-08-03 | 2d |
| `GearServer` | Barricade, Ammo Crate, Shock Mine templates | 1d |
| `NinjaLobby.ClassConfig` | Ranger / Engineer / Medic passives and abilities | 2d |
| HUD | Reactor HP, Amber gauge, wave counter, exploration timer | 2d |
| `NinjaProfileStore` | New keys: DinoCodex, EggInventory, PetInventory, SurvivorLog | 1d |
| Enemy rigs | Grey-box 6 dinosaurs from Cow / Alien dog quadrupeds | 3d |

### 3.3 To be developed — nothing exists

| Module | Description | Est. |
|---|---|---|
| `ObjectiveService` — Reactor | HP, damage intake, repair, failure state | 2d |
| `ObjectiveService` — Amber | Server drops, magnet VFX, gauge, 5 levels | 2d |
| `BossService` | Phase state machine, telegraphs, invuln cap, health bar, checkpoint reward | 3d |
| Boss 1 content | Alpha Raptor Matriarch — summons, mark, wall-leap, flank window | 2d |
| `ExplorationService` | Wave pause, 150s timer, warnings, safe recall | 2d |
| `SurvivorService` | Escort NPC, authored waypoints, stuck recovery, safe room | 2d |
| `HatcheryService` | Nest, guardian, egg carry, incubation, idempotent pet grant | 3d |
| Dino Codex | Species states, variant slots, compact in-match overlay | 2d |
| Downed & revive | 25s bleed-out, 3s revive, next-wave respawn | 1d |
| Run perks | Milestone 3-choice draft, 6 perks | 1d |

**Raw estimate: 22d refactor + 20d new = 42 developer-days against 20 available.** This is
precisely why the month is scoped depth-first. Section 8 shows the cut.

---

## 4. Functional requirements

Priority: **M** must-have this month · **S** should-have · **C** could-have · **W** won't (deferred).

### 4.1 Match structure

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-01 | M | The match runs exactly 20 waves | Wave counter reaches 20; no wave 21 spawns |
| FR-02 | M | Clearing wave 20 wins the run | Victory screen; rewards granted once |
| FR-03 | M | Reactor at 0 HP ends the run in defeat | Defeat screen; checkpoint rewards retained |
| FR-04 | M | Waves 4/8/12/16/20 are boss waves | Boss spawns; normal spawning suspended |
| FR-05 | M | Normal intermission is 15s | Timer visible; buy actions available |
| FR-06 | S | Milestone waves offer a 3-perk choice | Three distinct perks; one selectable; stacks respected |

### 4.2 Reactor and Amber

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-10 | M | A reactor exists with server-owned HP | HP replicates; not client-writable |
| FR-11 | M | Dinosaurs reaching the defense ring attack the reactor | Reactor HP falls; uses the existing `gearDamage` channel |
| FR-12 | M | Players can repair the reactor | Hold-interact; costs Credits; rate from `Params` |
| FR-13 | M | Killed dinosaurs drop Amber, server-owned | Amber never enters a backpack |
| FR-14 | M | Amber flies to the reactor and adds to the gauge | 0.5–1.2s curve; value added once; VFX culled on low-end |
| FR-15 | M | Reactor levels at 250 / 600 / 1,050 / 1,600 cumulative | Level-up fires once per threshold |
| FR-16 | M | Amber thresholds scale with party size | Solo and 6-player both reach L2 by a comparable wave |

> FR-16 addresses the flat-threshold defect in [01-gdd-review.md](01-gdd-review.md) §3.

### 4.3 Bosses

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-20 | M | A boss runs a multi-phase state machine | Phases advance on HP thresholds; recover from lost target |
| FR-21 | M | No boss is invulnerable for more than 8 consecutive seconds | Instrumented and logged |
| FR-22 | M | Attacks that can down a full-health player telegraph ≥1s | Ground indicator plus audio cue |
| FR-23 | M | A boss grants exactly one checkpoint reward | Duplicate grant IDs rejected |
| FR-24 | M | Defeated players respawn once at a phase transition | Verified in multiplayer |
| FR-25 | M | Boss 1 — Alpha Raptor Matriarch — is fully playable | Summons, hunter mark, wall-leap, flank window after missed pounce |
| FR-26 | W | Bosses 2–5 | Deferred |

### 4.4 Sectors and exploration

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-30 | M | A sector opens only on boss cleared **and** Amber level reached | Neither alone opens it |
| FR-31 | M | If the gauge fills later, the sector opens then | Defines the mid-wave case the GDD leaves open |
| FR-32 | M | Exploration pauses wave spawning and reactor damage | No spawns, no reactor HP loss |
| FR-33 | M | Exploration lasts 150s with 30s and 10s warnings | Timer visible; warnings fire once |
| FR-34 | M | At zero, distant players are safely recalled, not killed | No death, no fall damage |
| FR-35 | M | Unfinished objectives persist to the next window | Survivor and nest state retained |

### 4.5 Survivors, eggs and collection

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-40 | M | One survivor can be found, escorted and credited | Team-wide credit; appears in the safe room |
| FR-41 | M | A stuck survivor relocates after 8s | Returns to last safe waypoint |
| FR-42 | M | A guarded nest yields one egg per run | Kill **or** distract both work |
| FR-43 | M | Carrying an egg is visible and slows sprint | Icon above carrier |
| FR-44 | M | Depositing an egg secures it against run failure | Survives a forced run loss |
| FR-45 | M | Incubation is server-timed and survives reconnect | Timestamp-based, not tick-based |
| FR-46 | M | Hatching grants exactly one permanent pet | Idempotent; duplicate IDs rejected |
| FR-47 | S | The Codex records seen / defeated / egg / hatched | Persists across sessions |

> FR-44 through FR-46 are the highest data-integrity risk in the project.

### 4.6 Player systems

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-50 | M | Three classes with distinct passives and abilities | 60s cooldown; duplicates allowed |
| FR-51 | M | Downed state lasts 25s | Not instant death |
| FR-52 | M | Teammates revive in 3s, interrupted by damage | Verified in multiplayer |
| FR-53 | M | Expired downed respawns at the next wave, −15% unspent Credits | Solo case explicitly defined |
| FR-54 | S | Barricade, Ammo Crate and Shock Mine are placeable | Caps enforced server-side |
| FR-55 | W | Melee weapons and stamina | Deferred |

> FR-53 resolves the solo contradiction in [01-gdd-review.md](01-gdd-review.md) §6.

### 4.7 Content

| ID | Pri | Requirement | Acceptance criteria |
|---|---|---|---|
| FR-60 | M | Six dinosaur species are spawnable with distinct behaviour | Distinct silhouette, readable attack, valid server hitbox |
| FR-61 | M | Wave-1 species dies to one starter-weapon shot | Preserves the existing invariant |
| FR-62 | M | Every species is slower than the player's 17.6 | All are outrunnable |
| FR-63 | S | One flying species | New locomotion path |
| FR-64 | W | Species 7–15 | Deferred |

---

## 5. Non-functional requirements

### 5.1 Performance

| ID | Requirement | Measure |
|---|---|---|
| NFR-P1 | ≥50 client FPS on reference mid-tier mobile at low graphics, max wave load | FPS sample at wave 19–20 |
| NFR-P2 | Active AI never exceeds `min(perPlayer × live, 60)` | Instrumented; one server-wide cap |
| NFR-P3 | Server heartbeat stable under maximum wave load | No visible rubber-banding |
| NFR-P4 | Corpses despawn within 3–5s; no unbounded connections or path objects | Memory flat across 20 waves |
| NFR-P5 | All enemies, projectiles, tracers and damage numbers are pooled | No per-spawn instantiation spikes |

### 5.2 Security and authority

| ID | Requirement |
|---|---|
| NFR-S1 | The server owns damage, ammo, currency, drops, wave state, unlocks and saves |
| NFR-S2 | The client may request an action but never reports a hit or reward as truth |
| NFR-S3 | Every remote validates distance, state, cooldown and ownership |
| NFR-S4 | Remotes are rate-limited; a flooding client is throttled, not trusted |
| NFR-S5 | Reward grants are idempotent with unique IDs; duplicates rejected |
| NFR-S6 | No client reward remote exists at all |

### 5.3 Data integrity

| ID | Requirement |
|---|---|
| NFR-D1 | Profiles are session-locked; concurrent servers cannot both write |
| NFR-D2 | Saves survive disconnect, rejoin and forced shutdown |
| NFR-D3 | Egg and pet grants are atomic — an egg cannot be duplicated or lost mid-hatch |
| NFR-D4 | Incubation is timestamp-based, not tick-based, so it survives reconnects |
| NFR-D5 | Profile schema is versioned with a migration path |

### 5.4 Compatibility

| ID | Requirement |
|---|---|
| NFR-C1 | Playable on PC, mobile, tablet and console |
| NFR-C2 | Keyboard, gamepad and touch reach every action |
| NFR-C3 | Camera mode never changes damage, fire rate, recoil, awareness or hitbox size |
| NFR-C4 | UI supports common aspect ratios and never blocks mobile controls |

### 5.5 Usability and accessibility

| ID | Requirement |
|---|---|
| NFR-U1 | A new player understands the objective within 60 seconds |
| NFR-U2 | Independent music, effects, voice and UI volume sliders |
| NFR-U3 | Subtitles for critical warnings |
| NFR-U4 | Sensitivity, FOV, camera-shake and reduced-motion options |
| NFR-U5 | Boss indicators readable at the lowest graphics setting |
| NFR-U6 | Threat conveyed by audio and shape, not colour alone |

### 5.6 Maintainability

| ID | Requirement |
|---|---|
| NFR-M1 | `Params` remains the single source of truth for all tunable numbers |
| NFR-M2 | Stats are derived from `lp`, never hand-tuned in isolation |
| NFR-M3 | Every non-obvious number carries a comment saying why, when and who asked |
| NFR-M4 | All place scripts are exported one-way into `src/` and committed each sprint |
| NFR-M5 | No second parallel service layer — **D-04** must resolve |

### 5.7 Compliance

| ID | Requirement |
|---|---|
| NFR-X1 | Original name, creatures, audio and art — no implied Jurassic franchise link |
| NFR-X2 | Roblox platform policy compliance |
| NFR-X3 | No paid item exceeds the best earnable item in both damage and utility |

---

## 6. Timeline — 7 Sep to 2 Oct 2026

Each week ends in a playable build and a `src/` export.

### Week 1 · Mon 7 – Fri 11 Sep · **Foundations**

> **Goal:** dinosaurs spawn, waves are finite, nothing is lost if we break something.

| Day | Work |
|---|---|
| Mon 7 | Resolve D-01…D-04. Baseline export of every script to `src/`; commit. |
| Tue 8 | Fork the wave pipeline: finite 20-wave table; retire cycle multipliers. |
| Wed 9 | `Params` — six dinosaur rows via the `lp` derivation. Currency rename. |
| Thu 10 | Grey-box 3 species from the Cow / Alien dog quadruped rigs. |
| Fri 11 | Wire species to waves. Full 20-wave run. Export, commit, push. |

**M1 exit:** a 20-wave run completes with dinosaurs; wave-1 enemy dies to one starter shot.
**Covers:** FR-01, FR-04, FR-60, FR-61, FR-62.

### Week 2 · Mon 14 – Fri 18 Sep · **The spine**

> **Goal:** there is something to defend, and the run can be won or lost.

| Day | Work |
|---|---|
| Mon 14 | Reactor object: HP, damage intake via `gearDamage`, defense ring targeting. |
| Tue 15 | Repair interaction. Defeat state at 0 HP. Victory at wave 20. |
| Wed 16 | Amber: server drops, magnet flight, gauge accumulation. |
| Thu 17 | Reactor levels 1–5, party-scaled thresholds (FR-16). |
| Fri 18 | HUD: reactor HP, Amber gauge, wave counter. Regression run. Export, push. |

**M2 exit:** a full run can be won **and** lost; the reactor is the objective.
**Covers:** FR-02, FR-03, FR-10…FR-16.

### Week 3 · Mon 21 – Fri 25 Sep · **Boss and exploration**

> **Goal:** the loop the game is actually about.

| Day | Work |
|---|---|
| Mon 21 | `BossService`: phase machine, invuln cap, telegraphs, health bar. |
| Tue 22 | Boss 1 — Alpha Raptor Matriarch. Checkpoint reward, phase respawn. |
| Wed 23 | `ExplorationService`: wave pause, 150s timer, warnings, safe recall. |
| Thu 24 | Sector 1 gating on boss **and** Amber level, including the late-fill case. |
| Fri 25 | Sector 1 layout: hatchery, survivor point, nest. Export, push. |

**M3 exit:** wave 4 boss → sector opens → 150s exploration → safe return.
**Covers:** FR-20…FR-25, FR-30…FR-35.

### Week 4 · Mon 28 Sep – Fri 2 Oct · **Collection, classes, hardening**

> **Goal:** the retention loop, then make it hold up.

| Day | Work |
|---|---|
| Mon 28 | Survivor escort: waypoints, stuck recovery, safe room, team credit. |
| Tue 29 | Nest guardian, egg carry, hatchery deposit. **Idempotent grant IDs.** |
| Wed 30 | Incubation timers, hatch reveal, pet grant, minimal Codex. |
| Thu 1 Oct | Three classes, downed and revive. |
| Fri 2 Oct | Performance pass, mobile test, exploit pass, bug fix. Final export, push. |

**M4 exit:** end-to-end playable MVP slice meeting the acceptance criteria.
**Covers:** FR-40…FR-47, FR-50…FR-53, all NFRs verified.

### Milestones

| ID | Date | Gate |
|---|---|---|
| M1 | Fri 11 Sep | Finite 20-wave run with dinosaurs |
| M2 | Fri 18 Sep | Reactor objective; win and loss states |
| M3 | Fri 25 Sep | Boss → sector → exploration loop |
| M4 | Fri 2 Oct | Playable MVP slice, hardened |

---

## 7. Risk register

| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| Dinosaur art never arrives | High | Medium | A2 — grey-box from day one; the spawner already skips missing rigs |
| D-01/D-02 unresolved past Day 2 | High | Medium | Week 1 Day 1 is reserved for decisions; default to finite + own-in-place |
| Egg or pet duplication | High | Medium | Unique grant IDs, atomic receipts, idempotent writes; dedicated exploit pass W4 |
| Breaking the live alien game irrecoverably | High | Medium | One-way `src/` export before the first destructive change (D-06) |
| Flight and water locomotion overruns | Medium | High | FR-63 is Should, not Must; ground species carry M1 |
| Mobile performance regression | Medium | Medium | Existing caps and pooling are sound; spot-check W2, full pass W4 |
| Scope creep from spec defects | Medium | High | Defects are catalogued in 01; changes go through the decisions doc |
| Two parallel service layers | Low | High | D-04 forced to a decision in Week 1 |

---

## 8. What is deferred, and why

| Deferred | Reason | Earliest |
|---|---|---|
| Bosses 2–5 | 2d each on top of the framework; the framework is the hard part and lands in W3 | Month 2 |
| Sectors 2–4 | Each needs layout, gating, survivor and nest content | Month 2 |
| Survivors 2–5 | Same escort system, new placement and rewards — cheap once one works | Month 2 |
| Species 7–15 | Art-gated, not code-gated | With the asset pack |
| Melee weapons | Entirely new system including stamina; no existing melee at all | Month 2 |
| Full 20-gun rebalance | 27 guns exist and work; renaming is cosmetic and can wait | Month 2 |
| Mutations, difficulty modes, monetization | Depend on a stable core that does not yet exist | Month 3 |

The shape of this is deliberate: **month 1 builds every system once; month 2 multiplies content
across systems that already work.** That is far cheaper than building 5 bosses before knowing
whether the boss framework is right.

---

## 9. Exit criteria for the month

The month is complete when all of the following hold in a Studio play test:

- [ ] A 20-wave run can be won, and lost by reactor destruction
- [ ] Six dinosaur species spawn with distinct, readable behaviour
- [ ] Amber delivers to the reactor once and levels the gauge
- [ ] Sector 1 opens only on boss cleared **and** Amber level reached
- [ ] Boss 1 completes all phases and grants exactly one checkpoint reward
- [ ] Exploration pauses waves, warns, and recalls safely
- [ ] One survivor can be rescued and credited team-wide
- [ ] An egg can be recovered, deposited, incubated and hatched into exactly one pet
- [ ] Eggs and pets survive a disconnect/rejoin test with no duplication
- [ ] Three classes function; downed and revive work in multiplayer
- [ ] The reference mobile tier holds ≥50 FPS at wave 20
- [ ] No client remote can forge a hit, a reward or a duplicate grant
- [ ] `src/` reflects the place, and WORKLOG records every session
