# GDD coverage audit

**Audited 2026-10-01** by reading the live place directly — every service, config module, UI
module and asset folder — and comparing against
[Primal_Rift_MVP_Game_Design_Document.docx](../Primal_Rift_MVP_Game_Design_Document.docx).

This answers a different question from [08-status.md](08-status.md). That file measures
progress against our **deliberately thinned** plan (one boss not five, one sector not four,
six species not fifteen). This file measures against the **GDD's full MVP scope**, which is
the harder and more honest number.

> **The place has moved substantially since the 2026-09-29 status file.** Eight services,
> five config modules, seven UI modules and twenty named creature rigs now exist that were
> not there then. [08-status.md](08-status.md) and `Dino-Hunt-Tasks.xlsx` are both stale and
> need re-syncing — see §6.

---

## 1. The short answer

| Dimension | Coverage |
|---|---|
| **Systems** — does the machinery exist? | **~75%** |
| **Content** — is there enough of it? | **~45%** |
| **Verification** — is it proven to work? | **~25%** |
| **GDD's own acceptance checklist (§12.3)** | **0 of 15 fully met** |

Three numbers rather than one, because a single figure would hide the real shape: the
architecture is well advanced, the content is thin in specific places, and almost none of the
newer work has been observed running.

**The headline:** most of the GDD's *systems* are now built. What is missing is 15 of 20 guns,
all 3 melee weapons, the entire analytics layer, difficulty modes, monetization, onboarding,
the FPS/TPS camera toggle, and — the one most likely to surprise — **creature art that is
imported but not wired up.**

---

## 2. Content counts, measured

| GDD requirement | Required | Present | |
|---|---|---|---|
| Normal dinosaur species | 15 | **15 defined** | ✅ |
| …with distinct rigs | 15 | **0 wired** (all share 4 grey-box rigs) | ❌ |
| Bosses | 5 | **5 defined**, 1 playtested | ⚠️ |
| Guns | 20 | **5** | ❌ |
| Melee weapons | 3 | **0** | ❌ |
| Waves | 20 | **20** | ✅ |
| Reactor levels | 5 | **5** | ✅ |
| Sectors | 4 | **4** (blockout) | ✅ |
| Classes | 3 | **3** | ✅ |
| Run perks | 6 | **6** | ✅ |
| Survivor roles | 5 | **16 across 4 sectors** | ✅ |
| Guarded nests | ≥1 per sector | **4** | ✅ |
| Difficulty modes | 3 | **0** | ❌ |
| Analytics event types | 12 | **0** | ❌ |

### The creature-art finding

`ServerStorage.Assets.Creatures` contains **20 named rigs** — `Compsognathus`, `Velociraptor`,
`Triceratops`, `Allosaurus`, `Therizinosaurus`, plus `Boss_Raptor`, `Boss_Rex`,
`Boss_Pteranodon`, `Boss_Spinosaurus`, `Boss_Triceratops` and the rest.

**None of them are used.** `Config.Creatures` has no per-species `Rig` field, so every species
still resolves through its *role* to one of the four grey-box rigs:

```
Swarmer 2 species · Flanker 5 · Armoured 6 · Ranged 2
```

So GDD §12.3 item 2 — "all 15 normal dinosaur types have **distinct silhouettes**" — currently
fails even though the art exists. Six different species are visually identical boxes.

Wiring it is small: add `Rig = "<name>"` per species in `Config.Creatures.Species` and let it
override the role default. Worth doing before any more balance work, because readability
affects how the game plays.

---

## 3. Section-by-section against the GDD

### §3 Core Match Design — mostly built

| | Status |
|---|---|
| 20-wave match, victory, defeat | ✅ Verified in playtest |
| Reactor as primary objective | ✅ Verified |
| Downed 25s / revive 3s / −15% credits | ✅ Built (`DownedService`, 380 lines) — config matches the GDD exactly |
| 15s intermission | ✅ Verified |
| Exploration 150s, final 180s, warn at 30s/10s | ✅ Built — matches the GDD exactly |
| **FPS/TPS camera toggle (§3.4, a whole section)** | ❌ Essentially absent — one `CameraMode` reference, no first-person implementation |

§3.4 is nine requirement rows in the GDD, including the fairness rule that camera mode never
changes damage or hitboxes. None of it is built.

### §4 Outpost, Classes, Interactive Systems — mostly built

| | Status |
|---|---|
| Reactor Core, Containment Wing, 4 sectors, lanes | ✅ Blockout complete |
| 3 classes with GDD's exact passives and abilities | ✅ Built — Scan Pulse 6s, Shock Fence 7s, Healing Mist 6s, 60s cooldown |
| Reactor levels 1–5, Amber magnet, thresholds | ✅ Verified |
| Boss-clear exploration windows | ✅ Built, unverified |
| Survivor rescue | ✅ Built (`SurvivorService`, 458 lines), unverified |
| **Buildable defenses — Barricade, Ammo Crate, Shock Mine (§4.3)** | ❌ Not built. "Barricade" appears only in a wave event string and a perk description |

### §5 Normal Dinosaur Roster — defined, not differentiated

All 15 species present with the GDD's roles, first-wave numbers and armour arcs. Stats derive
from one `lp` weight per species.

**But:** no distinct silhouettes (see §2), and the GDD's per-species behaviours are not
individually implemented — a Gallimimus "rushes past players toward the reactor", an Oviraptor
"steals deposited samples", a Carnotaurus has a "camouflage burst". Currently every creature
walks its lane and bites what is in reach. Roles exist as stat bands, not as behaviours.

### §6 Boss Encounters — framework done, content 1 of 5

All five GDD bosses are defined by name with phases. `BossService` is 460 lines with a phase
machine, telegraphs, the FR-21 invulnerability ceiling, hunter marking and checkpoint rewards.

**Only Boss 1 has been playtested**, and it needed retuning after the first attempt (every
pounce missed; the fight ran 187s against a 100s target). Bosses 2–5 are unobserved.

### §7 Twenty-Wave Schedule — done

All 20 waves transcribed with the GDD's compositions and events. Scaling formulas match §7.1
except boss health, which **deliberately deviates** — see [01-gdd-review.md](01-gdd-review.md)
defect #4.

### §8 Weapon Roster — the largest content gap

**5 guns of 20. 0 melee of 3.**

The five that exist (`AWP`, `M4A1`, `P320`, `UMP-45`, `Tactical M1014`) are also **not the
GDD's weapons** — the spec lists Standard Pistol, Tactical Revolver, Suppressed Pistol, SMG,
Shotgun, Flame SMG, Toxic Shotgun, Galaxy Rifle, Frost Blaster, Plasma Cannon and so on, with
elemental effects, heat instead of reload, and charge mechanics.

So §8 is not 25% done; it is a different, smaller roster. Decide whether the GDD's 20-weapon
list still stands — it carries the "every weapon has a job" pillar and the elemental status
system, neither of which exists.

Run perks are the exception: **all 6 of the GDD's perks** are implemented with matching text
and stack limits.

### §9 Economy, Collection and Progression — half built

| | Status |
|---|---|
| Run Credits | ✅ Verified |
| Amber | ✅ Verified |
| DNA | ⚠️ Referenced in BossService/CodexService/Survivors; not a tracked currency in MatchState |
| Eggs, nests, carry, deposit | ✅ Built (`HatcheryService`, 561 lines), unverified |
| Incubation 120–180s | ✅ Built — 120/150/180 by rarity |
| Permanent pets | ✅ Built (`PetInventory` in DataService), unverified |
| Dino Codex | ✅ Built (`CodexService` + `UI.Codex`), unverified |
| **Mutations (§9.2)** | ❌ No references anywhere |
| **Hunter Level, Weapon Mastery, Class Mastery (§9.4)** | ❌ No references anywhere |
| **Difficulty modes — Recruit / Standard / Nightmare (§9.5)** | ❌ Not built |
| **Monetization (§9.6)** | ❌ No MarketplaceService usage |

### §10 UI, Onboarding, Audio, VFX — UI strong, onboarding absent

Nine UI modules: `Hud`, `Status`, `Combat`, `Compass`, `Announcer`, `Class`, `Codex`, `Perks`,
`Theme`. That covers most of the GDD's seven HUD areas.

`Config.Audio` exists with shot, reload, hit/weak/kill ticks, small and big growls, boss roar,
reactor alarm and ambience — a real start on §10.4.

**§10.2's eight-step first-session onboarding is entirely absent.** No tutorial references
anywhere. Given the market scan found onboarding named as a retention lever and a competitor
losing 92% of its players, this is a bigger gap than its size suggests.

### §11 Technical Design — strong, one service missing

| | Status |
|---|---|
| Server authority over damage, ammo, currency, drops | ✅ Verified |
| Remote validation | ✅ Built |
| Concurrency cap, pooling | ✅ Verified |
| `DataService` | ✅ Real: DataStoreService, `UpdateAsync`, session locking, schema versioning, 60s autosave, `Profiles_v1` |
| **`AnalyticsService` (§11.1 lists it as required)** | ❌ Does not exist; no references anywhere |

### §12 Analytics and Acceptance — not started

**None of the GDD's 12 required event types are recorded.** No funnel data means the KPI
targets in §12.2 cannot be measured at all.

---

## 4. The GDD's own acceptance checklist (§12.3)

The spec's definition of done. **0 of 15 items fully met**, 9 substantially built.

| # | Item | Status |
|---|---|---|
| 1 | 20 waves complete solo **and six-player** | ⚠️ Solo verified; multiplayer never tested |
| 2 | 15 types, **distinct silhouettes**, valid hitboxes | ❌ Six species share one box |
| 3 | 5 bosses all phases, one checkpoint reward each | ⚠️ 1 of 5 playtested |
| 4 | 20 guns + 3 melee pass validation | ❌ 5 guns, 0 melee |
| 5 | FPS/TPS switch on all inputs, no combat change | ❌ Not built |
| 6 | Reactor, repair, **barricade**, downed, revive in multiplayer | ⚠️ Barricade missing; multiplayer untested |
| 7 | Amber delivered once; sectors need boss **and** level | ⚠️ Amber verified; gating unverified |
| 8 | Exploration pauses damage, warns, recalls safely | ⚠️ Built, unverified |
| 9 | Five survivor roles found, escorted, credited, displayed | ⚠️ Built, unverified |
| 10 | Nests support kill **and** distraction; eggs cannot duplicate | ⚠️ Built, unverified |
| 11 | Hatchery timers survive reconnect; exactly one pet | ⚠️ Built, unverified |
| 12 | Persistent rewards survive disconnect; duplicate IDs rejected | ⚠️ Built, unverified |
| 13 | Max wave playable on reference mobile | ❌ Not tested |
| 14 | UI supports aspect ratios, never blocks mobile controls | ❌ Not tested |
| 15 | Analytics record all funnels | ❌ No analytics |

**Nine of fifteen are "built but never observed."** That is the single most important fact in
this audit. The gap between written and verified is now the project's biggest risk — bigger
than any missing feature, because unverified code is of unknown value.

---

## 5. What to do, in order

1. **Wire the creature rigs.** Small change, large effect: fixes acceptance item 2 and makes
   every later balance observation meaningful. Six identical boxes cannot be told apart, so
   nothing learned about target priority is trustworthy.
2. **Verify the nine built-but-unobserved systems.** Exploration, survivors, nests, eggs,
   incubation, pets, Codex, persistence, sector gating. Most need one playtest each with the
   right dev commands. This converts the most value for the least work.
3. **Decide the weapon roster (§8).** 5 real-world guns versus the GDD's 20 elemental ones is
   not a gap to fill incrementally — it is a design decision. Raise it before adding gun six.
4. **Add `AnalyticsService`.** It is a listed requirement, it blocks every KPI, and the market
   scan says retention is what kills games in this genre. Cheap now.
5. **Onboarding (§10.2).** Eight scripted steps. Named as a retention lever by developers who
   have hit this wall.
6. **Multiplayer test.** Nothing has ever run with two players. Six acceptance items say
   "in multiplayer".

Deliberately *not* urgent: difficulty modes, monetization, mutations, mastery tracks. All are
GDD scope but all sit behind a stable, verified core.

---

## 6. Housekeeping this audit exposed

- **[08-status.md](08-status.md) and `Dino-Hunt-Tasks.xlsx` are stale.** They report 40/75
  tasks with Phase 4 half built; the place now contains Phase 5 and Phase 6 systems
  (`HatcheryService`, `SurvivorService`, `DownedService`, `ClassService`, `PerkService`,
  `CodexService`, `DataService`). The tracker needs re-syncing against the place.
- **`README.md`'s Stack section is wrong** and actively misleading for a new developer: it
  still says Rojo, Wally and Nevermore, and "code lives in `src/` and syncs into Studio via
  Rojo. The filesystem is the source of truth." All of that was reversed on 2026-09-28.
  Fixed in this commit.
- **The snapshot is behind the place.** `snapshot/DinoHunt-Snapshot.rbxm` predates everything
  described above, so the only version control the code has does not contain it.
