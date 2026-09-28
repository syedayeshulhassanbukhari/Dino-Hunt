# Conversion map — spec systems onto the Dino Valley place

Surveyed 2026-09-05, read-only over the Studio MCP bridge against the `Edit` data model.
Nothing was modified.

Published as a page: https://claude.ai/code/artifact/a5fa582e-d947-4b34-90ce-180d42b61f6f

## Headline

The place is **not empty**. It runs a complete, working Area 51 alien/zombie wave shooter.
Roughly half the spec's Section 11 architecture already exists under alien-themed names.

**4 reuse as-is · 7 extend · 4 build new.**

## The mapping

| Spec service | Exists as | Verdict |
|---|---|---|
| WaveService | `WaveManager`, `ZombieSpawner`, `ReplicatedStorage.WaveTable` | **Extend** |
| EnemyService | `ServerScriptService.ZombieAI` (13 modules) | **Extend** |
| CombatService | `GunFireServer`, `Blaster.Scripts.Blaster`, `WeaponFireConfig` | **Reuse** |
| WeaponService | `GunSystem`, `ReplicatedStorage.Blaster` (27 guns) | **Extend** |
| ObjectiveService | `AreaUnlockServer`, `AreaOccupancy`, `Workspace.AreaVolumes` | **Extend** |
| ExplorationService | — | **New** |
| SurvivorService | — | **New** |
| HatcheryService | — | **New** |
| DataService | `NinjaProfileStore` | **Reuse** |
| RewardService | `Economy`, `UpgradeService`, `BadgeService` | **Extend** |
| AnalyticsService | `Telemetry`, `ExitAnalytics` (6 modules), `RunTracker` | **Reuse** |
| Boss encounters (5) | — (no phase framework) | **New** |
| Classes (3) | `ReplicatedStorage.NinjaLobby.ClassConfig` | **Extend** |
| Buildable defenses | `GearServer`, `ServerStorage.GearTemplates.AutoTurret` | **Extend** |
| Camera FPS/TPS | `FirstPersonController`, `MobileCamera`, `CameraRecoiler` | **Reuse** |

## Notable wins

- **Armour weak points already ship.** The alien shield blocks shots within a **35 degree
  half-angle of the enemy's facing** and lets them land from the flank
  (`Blaster.Scripts.Blaster`, `SHIELD_BLOCK_HALF_ANGLE = 35`). That is the spec's
  Triceratops / Ankylosaurus mechanic, working today.
- **27 authored guns** with view models, recoil, heat, mobile touch input and shot
  replication. The spec wants 20. Renaming and rebalancing, not building.
- **Structure damage already exists** — every enemy carries a separate `gearDamage` stat for
  hitting player-placed gear. Redirecting that at a reactor is a small change.
- **Stuck-recovery pattern exists** (`StuckAlienRecovery`) and should be copied wholesale for
  the spec's 8-second survivor escort recovery rule.

## Notable gaps

- **No boss framework.** The UFO carries a difficulty weight of 117.8x a wave-1 alien and
  Big White debuts at wave 9, but both are ordinary AI with big numbers. Multi-phase state
  machines, telegraphs and the 8-second invulnerability cap are unbuilt.
- **No flight, no water traversal, no grab attack.** Pteranodon, Quetzalcoatlus, Baryonyx and
  Spinosaurus need new locomotion, not new stats.
- **No defended objective.** Today's mode is open arena survival.
- **No melee weapons at all.**

## Numbers already solved

`ReplicatedStorage.Params` declares itself "single source of truth for ALL numbers". Every
enemy stat descends from one difficulty weight `lp`, itself solved from the design table's
own totals column with **zero residual across all 50 first-cycle waves**:

```
baseHP     = 15  x lp        -- linear
damage     = 4.5 x sqrt(lp)  -- sub-linear, deliberately
gearDamage = 20  x sqrt(lp)
gemReward  = 10  x sqrt(lp)
coinReward = 5   x sqrt(lp)
speed      = from role, not derived
```

The square root is deliberate: linear damage scaling made late enemies hit for 58.

Two invariants are load-bearing and should carry into Primal Rift:

- the wave-1 enemy dies to **exactly one starter-weapon shot**
- **every enemy is slower than the player's 17.6**, so anything can be outrun

Concurrency is `min(perPlayer x livePlayers, MAX_ACTIVE_ZOMBIES = 60)` with a per-wave table
of per-player shares — which is also the correct answer to the spec's unimplementable
"45 on mobile, 60 on PC" cap.

**Recommendation:** adopt this derivation for the 15 dinosaurs. One `Params` row per species
with a weight, and every stat falls out. That is the numbers appendix
[01-gdd-review.md](01-gdd-review.md) says the spec is missing, in the format the engine
already reads.

## Cleanup inventory

Dead weight that will be copied forward if not dealt with deliberately. Not urgent — some
diagnostics are the fastest way to test a wave, so sweep *after* the vertical slice verifies.

`ZZZ_DiagMonitor_TEMP`, `ZZZ_SpawnDeathDiag`, `ZZZ_StraightLine_TEMP`, `ZZZ_DiagClient_TEMP`,
`_DEBUG_ArmVisualizer`, `_DEBUG_CameraTrace`, `_QA_GiveAllWeapons`, `SkipWaveServer`,
`_BACKUP_StuckAlienRecovery`, `_BACKUP_ZombieTemplates_20260819`, `_QA_Backup_Glock17`,
`MIBTestSpawner`, `CowTestSpawner`.

Enemy templates are duplicated across `Workspace`, `ReplicatedStorage`, `ReplicatedFirst` and
`ServerStorage` plus a dated backup. The spawner clones from `ServerStorage.ZombieTemplates`;
the others replicate to every client for no benefit.

## Recommended order

1. Settle the four open decisions in [03-open-decisions.md](03-open-decisions.md).
2. Fork the wave pipeline to a finite 20-wave table, keeping the `lp` derivation and the
   concurrency cap. Retire the cycle multipliers.
3. Write the `Params` rows for the 15 dinosaurs.
4. Build the reactor and Amber gauge — the spine, and the prerequisite for sector gating,
   boss unlocks and exploration.
5. Boss 1 as a phase-machine prototype. The Alpha Raptor Matriarch proves the pattern the
   other four reuse.
6. Exploration window, then one survivor and one nest. This completes the Phase 2 vertical
   slice the spec itself asks for before content expansion.
7. Sweep the diagnostics and duplicate templates.
