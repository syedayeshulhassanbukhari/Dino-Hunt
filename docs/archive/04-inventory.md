# Inventory — what exists, what is missing

State of the **Dino Valley** place (placeId `88638180383535`) as of 2026-09-06, measured
against the Primal Rift MVP spec. Survey was read-only; nothing was modified.

Companion to [02-conversion-map.md](02-conversion-map.md), which maps spec *services* onto
code. This document is the flatter question: **what can you play today, and what is not there
at all.**

## The one-line answer

The **shooter is built**. The **game around it is not**, and **none of the dinosaur content
exists**. Combat, weapons, enemy AI, waves, economy, saves, UI and analytics are all working
systems that need renaming and retuning. Every objective the spec is actually about — the
reactor, Amber, bosses, exploration, survivors, eggs, the Codex — is absent, and so is every
dinosaur.

---

## Available — built and working

### Combat and weapons
- Server-authoritative hit validation, fire-rate and ammo checks
- Hitscan and projectile weapons, recoil patterns, heat, reload
- **27 authored guns** with data modules, view models, sounds and shot replication
  (AK-47, AWM, SPAS-12, MiniGun, FlameThrower, Quantum Rifle, Void Cannon, M79, KSG,
  Grenade Launcher, Missile Pod, Tactical Shotgun and more)
- **47 gun meshes** staged in `Workspace.All Guns`
- Weak-point / headshot multipliers
- **Directional armour**: shots are blocked within a 35 degree half-angle of an enemy's
  facing and land from the flank — the Triceratops mechanic the spec asks for, already shipping
- First person and third person, mobile camera, ADS, crosshair, hit markers, damage numbers

### Enemy AI
- 13-module AI package: Targeting, Separation, Pathfinding, Movement, Combat, Ranged,
  Charge, Enrage, Summon, WorldState, Animation, Config, Manager
- Global pathfinding budget with a token throttle
- Surround-ring positioning, plus a contact-damage mode for chargers
- Stuck detection and recovery
- Ranged attackers, summoners, enraging elites, charging bruisers

### Wave system
- Wave engine with action and build phases
- **300-wave authored table** across six difficulty cycles, with per-wave composition
- Concurrency cap: min(perPlayer x livePlayers, 60), with a per-wave per-player table
- Batched spawning from a reservoir so the cap is never exceeded
- Spawn doors and spawner plates

### Enemy content
- **26 rigs** in `ServerStorage.ZombieTemplates`, with animations
- Colour-family ladder (green, light blue, orange, blue, purple, lime) x variants
  (standard, shield, big) plus specials: Man in Black, Mimic Clown, Cow, Alien dog, UFO,
  summoner stages
- All stats derived from one difficulty weight — see the formulas in
  [02-conversion-map.md](02-conversion-map.md)

### Economy and progression
- Two currencies: **gems** and **coins**, dropped per kill, values derived from enemy weight
- Weapon upgrade service with tiers
- Mystery box
- Placeable gear including an AutoTurret with its own AI, and a separate gearDamage
  channel so enemies can attack structures
- Badge system with a large authored badge config
- Rank / progression scaffolding

### Persistence and services
- Session-locked profile store, sharing a loadout contract with a separate lobby place
- Telemetry, a 6-module exit-analytics package with an HTTP sender, and run tracking
- Daily login shield (15% damage reduction)

### UI
- HUD, hotbar, mobile gun controls, upgrade menu, gear menu, gem button, profile,
  win/lose screen, AWM scope overlay, arena loading screen
- Damage vignette, combat feedback popups, spawn SFX, footsteps, BGM

### World
- Area 51 map with buildings, fence collision, area volumes, UFO movement paths and blockers
- Two further unused map folders present: `City` and `Mountain Village The Hidden Valley`

---

## Missing — nothing exists

### Every dinosaur
**Zero dinosaur assets are in the place.** No rigs, meshes, animations or textures. Searching
the data model for dino / raptor / rex / trike returns nothing.

This is the long pole. Worse, the existing rigs are the wrong shape: **24 of 26 templates are
humanoid bipeds** built from Parts and CharacterMeshes with standard R15-style joints. Only
**Cow** and **Alien dog** are non-humanoid quadruped MeshPart rigs, and they are the only
usable precedent for a dinosaur body plan.

The spec needs 15 normal species plus 5 bosses, spanning quadrupeds, bipeds with tails,
flyers and a swimmer. Almost none of that maps onto the current skeletons.

### Every objective system
- **Reactor** — no central defended object, no HP, no repair, no failure state
- **Amber** — no third currency, no magnet-to-gauge behaviour, no reactor levels
- **Sector unlocking** — area volumes exist, but the paid unlock walls were deliberately
  removed on 2026-08-03 and all spawner doors are permanently open
- **Exploration phase** — nothing pauses waves, warns, or recalls distant players
- **Survivors** — no escort NPCs, no waypoint following, no safe room
- **Eggs, nests, hatchery** — nothing; no carry state, no incubation, no hatch grant
- **Dino Codex** — no collection book, no species research states, no variant tracking
- **Baby pets** — no companion system

### Boss encounters
No boss framework of any kind. The UFO carries a difficulty weight 117.8x a wave-1 enemy and
Big White debuts at wave 9, but both are ordinary AI with large numbers. Missing: multi-phase
state machines, phase transitions, telegraphed ground indicators, invulnerability windows and
the 8-second invulnerability cap, boss health bars, and the guaranteed checkpoint reward.

### Match structure
- **No victory condition.** The game is endless; the spec ends at Wave 20 with a win.
- **No boss gating.** Waves 4/8/12/16/20 are not special.
- **No intermission shop flow** in the shape the spec describes (15s buy window, 3-perk
  milestone choice)
- **No run perks** — the six-perk list has no counterpart

### Player systems
- **No classes.** A class config exists in the lobby module, but Ranger / Engineer / Medic
  and their three abilities are unimplemented.
- **No downed state or revive.** No 25s bleed-out, no 3s teammate revive, no next-wave respawn
- **No melee weapons at all.** Spear, Heavy Axe and Energy Blade are new, as is the entire
  stamina system.
- **No DNA currency**, no weapon mastery, no Hunter Level in the shape the spec describes

### Defenses
Barricade, Ammo Crate and Shock Mine do not exist as templates. The placement system they
would sit on does exist.

---

## Where the effort actually goes

Ranked by remaining work, largest first.

1. **Dinosaur art and rigs** — 20 creatures, mostly non-humanoid, plus animation sets.
   Nothing to reuse. This gates everything else being testable.
2. **Boss encounters** — 5 multi-phase fights on a framework that does not exist.
3. **The collection loop** — eggs, nests, hatchery, incubation, pets, Codex. Entirely new,
   and the riskiest for data integrity because it writes permanent inventory from timers.
4. **Exploration and survivors** — new match state, new escort AI, new map sectors.
5. **Reactor and Amber** — conceptually simple, but it is the spine everything gates on.
6. **New locomotion** — flight, water traversal and grab attacks for four species.
7. **Match restructure** — finite 20 waves, boss gates, victory.
8. **Renaming and retuning** — weapons, enemy stats, currencies, UI strings. Large but
   mechanical, and the derivation formulas make the stat work fast.

## What this means for the plan

The vertical slice the spec asks for (Waves 1-4, Boss 1, Sector 1, one survivor, one egg)
cannot be tested without at least a handful of dinosaur rigs. Two options:

- **Grey-box it.** Reskin two existing rigs as stand-in dinosaurs and build the systems
  against them. Systems progress immediately; art lands later.
- **Art first.** Wait for the supplied asset pack before building anything creature-facing.

Grey-boxing is the faster path, and it matches how the existing codebase already handles a
partly-delivered art set — its spawner silently skips any enemy type whose rig is missing,
so a half-delivered roster still ships.
