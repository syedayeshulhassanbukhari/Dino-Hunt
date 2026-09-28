# Dino Hunt — greenfield project plan

**Version** 1.0 · **Written** 2026-09-28 · **Supersedes** docs 02, 04 and 05 (archived)

The previous implementation — a reskin of an existing Area 51 alien shooter — has been
deleted. This is a clean build in a new blank place called **Dino Hunt**, on a modern
toolchain, grey-box first.

**The design is unchanged.** [Primal_Rift_MVP_Game_Design_Document.docx](../Primal_Rift_MVP_Game_Design_Document.docx)
remains the spec, and its known defects are still catalogued in [01-gdd-review.md](01-gdd-review.md).
Only the implementation restarts.

---

## 1. What changed, and why it matters

| | Before | Now |
|---|---|---|
| Codebase | Inherited Area 51 shooter, reskinned | Greenfield |
| Source of truth | The `.rbxl` place | **The filesystem**, synced by Rojo |
| Editing | MCP wrote scripts into the place | Edit files locally; Rojo syncs; MCP inspects and tests |
| Version control | Docs only; game code had none | **All game code in git** |
| Frameworks | None — hand-rolled | **Fusion** (UI) + **Nevermore** (architecture) |
| Art | Reskin existing alien rigs | **Grey-box from nothing** |

The source-of-truth reversal is the important one. Fusion and Nevermore are distributed
through Wally and expect a filesystem project, so adopting them necessarily moves authoring
off the place and into the repo. That closes the version-control gap (old decision D-06)
completely: every line of gameplay code becomes diffable and revertible.

### 1.1 What we keep from the deleted codebase

The code is gone; the good ideas in it are not. These were genuinely well-designed and are
adopted as principles for the new build:

- **Derived stats.** Every creature's numbers descend from a single difficulty weight rather
  than being hand-tuned per field: `baseHP = k₁ × lp`, `damage = k₂ × √lp`, rewards likewise.
  Sub-linear damage keeps late enemies from hitting absurdly hard. One weight per species is
  the only number a designer picks.
- **One server-wide concurrency cap**: `min(perPlayer × livePlayers, MAX_ACTIVE)`. Not a
  per-device cap — AI count is a server property.
- **Directional armour.** Shots blocked within a cone of the creature's facing and landing
  from the flank. This is the Triceratops and Ankylosaurus mechanic, and it worked.
- **Invariants worth protecting**: the wave-1 enemy dies to exactly one starter-weapon shot;
  every creature is slower than the player, so anything can be outrun; nothing one-shots a
  full-health player.
- **Comment the why.** Numbers carry a note explaining their derivation, the date and who
  asked. A bare constant is a code smell.

---

## 2. Technology stack

| Layer | Choice | Role |
|---|---|---|
| Toolchain manager | **Rokit** (or Aftman) | Pins rojo, wally, selene, stylua per project |
| Sync | **Rojo** | Filesystem → Studio |
| Packages | **Wally** | Dependency management |
| UI & reactive state | **Fusion** | All interface, all reactive state |
| Architecture | **Nevermore** | ServiceBag, Binder, Maid, Signal, Rx, utilities |
| Lint / format | **Selene**, **StyLua** | Enforced before commit |
| Types | **Luau strict** where practical | `--!strict` on new modules |
| Studio access | **Official Roblox MCP** | Inspection, play-test, screenshots — *not* authoring |

### 2.1 Division of labour between the two frameworks

They overlap — Nevermore ships Blend for UI and Rx for reactivity — so the split is a rule,
not a preference:

- **Fusion owns the client's view layer.** Every `ScreenGui`, every HUD element, every menu.
  Reactive state that drives pixels lives in Fusion `Value`/`Computed`.
- **Nevermore owns game architecture.** Service lifecycle and dependency injection via
  `ServiceBag`; tag-driven object binding via `Binder`; cleanup via `Maid`; cross-boundary
  events via `Signal` and `Rx`.
- **Do not use Blend.** Do not use Fusion for server-side state. One paradigm per side.

> Exact Wally package names and versions are confirmed at install time in Phase 0, not
> assumed from memory. Fusion's API changed significantly at 0.3 (explicit scopes, `peek`),
> so the version we pin determines how every UI module is written — pin it once, early.

### 2.2 Studio MCP — what it is and is not for

**Not for authoring.** Code lives in files. MCP is used to:

- inspect the live data model when something does not match expectation
- run play-tests and read the output console
- capture screenshots of grey-box layouts
- place and adjust world geometry that is genuinely easier to position visually than to
  describe in code

World geometry is the honest exception. Blocking out a map by typing `CFrame` values is
miserable; doing it in Studio and reading the result back is not. Grey-box geometry therefore
lives in the place and is committed as a `.rbxmx` model export, not as Luau.

---

## 3. Repository layout

```
Dino-Hunt/
├── rokit.toml                  pinned tool versions
├── wally.toml                  package dependencies
├── default.project.json        Rojo mapping
├── selene.toml  stylua.toml    lint + format config
├── src/
│   ├── shared/                 → ReplicatedStorage.Shared
│   │   ├── Config/             THE numbers. Species, waves, weapons, reactor.
│   │   ├── Net/                remote definitions, shared by both sides
│   │   ├── Types/              shared Luau type definitions
│   │   └── Util/
│   ├── server/                 → ServerScriptService.Server
│   │   ├── init.server.luau    bootstrap: build the ServiceBag, start services
│   │   └── Services/
│   │       ├── WaveService/
│   │       ├── CreatureService/
│   │       ├── CombatService/
│   │       ├── ObjectiveService/     reactor + Amber
│   │       ├── ExplorationService/
│   │       ├── HatcheryService/
│   │       ├── SurvivorService/
│   │       ├── RewardService/
│   │       └── DataService/
│   ├── client/                 → StarterPlayer.StarterPlayerScripts.Client
│   │   ├── init.client.luau
│   │   ├── Controllers/        input, camera, effects
│   │   └── UI/                 Fusion components
│   └── assets/                 → ServerStorage.Assets (rigs, .rbxmx exports)
└── docs/
```

### 3.1 Conventions

- **`src/shared/Config` is the single source of truth for numbers.** No gameplay constant
  is declared anywhere else. This replaces the old `Params` module and inherits its rule:
  every non-obvious number carries a derivation comment.
- One folder per service, `init.luau` plus siblings. No 3,000-line modules.
- Services declare their dependencies and are started by `ServiceBag`; nothing reaches across
  via `require` of a running singleton.
- Remotes are defined once in `shared/Net` and validated server-side without exception.
- `--!strict` on all new modules unless a package forces otherwise.

---

## 4. Grey-box scope

Grey-boxing means **untextured, unanimated, correct-in-shape-and-scale**. Success is that the
game is legible and playable, not that it looks like anything.

### 4.1 World blockout

| Element | Grey-box form |
|---|---|
| Reactor core | Central cylinder with a visible defense ring on the floor |
| Containment wing | Wall of cage volumes with door frames |
| Sectors 1–4 | Four bounded areas behind gate volumes |
| Jungle Gate / lanes | Two to three clearly-read approach lanes with spawn markers |
| Hatchery | Marked pad in Sector 1 |
| Nest / survivor points | Coloured markers |

Scale and sightlines matter here and nothing else. Lane width, reactor sightlines and cover
spacing are the things a blockout exists to test.

### 4.2 Placeholder creatures

Part-built rigs, distinct **silhouette per role**, one flat colour each. No meshes, no
animation beyond a walk cycle.

| Role | Grey-box form | Reads as |
|---|---|---|
| Swarmer | Small, low, fast | Compsognathus |
| Flanker | Medium, lean | Velociraptor |
| Armoured | Wide, tall front plate | Triceratops |
| Ranged | Medium with a marked head | Dilophosaurus |

Four is enough to prove wave composition and target-priority reading. Two of these carry
Wave 1.

### 4.3 Placeholder combat

One hitscan rifle, server-validated. No recoil curves, no attachments, no second weapon.
Enough to prove the damage path, weak points and directional armour.

---

## 5. Phase plan

Phases, not calendar dates — the clock starts when Phase 0 completes. Estimates assume one
developer working with Claude Code.

### Phase 0 · Toolchain — 1–2 days

Nothing else can start until this is done.

- Install and pin Rokit, Rojo, Wally, Selene, StyLua
- `wally.toml` with Fusion and the Nevermore packages we actually need; **pin versions**
- `default.project.json` mapping the tree in §3
- Connect the **official Roblox Studio MCP** and verify it reaches the Dino Hunt place
- Verify round trip: edit a file → Rojo syncs → change visible in Studio
- Commit the skeleton

**Exit:** a trivial Fusion HUD element renders in-game from a file edit.

### Phase 1 · Blockout and creature stub — 3–4 days

- World blockout per §4.1, exported to `src/assets` as `.rbxmx`
- One placeholder creature that spawns, paths to the reactor and can be shot dead
- `shared/Config` seeded with the species table and the derivation formulas
- `CreatureService` with pooling and the concurrency cap from day one

**Exit:** a creature walks a lane and dies to one rifle shot.

### Phase 2 · The wave loop — 4–5 days

- `WaveService`: 20 finite waves, composition table, batched spawning, intermission
- Four placeholder creature roles wired to waves
- Victory at Wave 20, defeat state stubbed

**Exit:** a full 20-wave run completes.

### Phase 3 · The spine — 4–5 days

- Reactor: HP, damage intake, repair, failure
- Amber: server drops, magnet flight, gauge, five levels, **party-scaled thresholds**
- Sector gating on boss cleared **and** Amber level
- Fusion HUD: reactor HP, Amber gauge, wave counter

**Exit:** a run can be won and lost; the reactor is the objective.

### Phase 4 · Boss and exploration — 5 days

- `BossService`: phase machine, telegraphs, invulnerability cap, checkpoint reward
- Boss 1 — Alpha Raptor Matriarch — grey-boxed
- `ExplorationService`: wave pause, 150s timer, warnings, safe recall
- Sector 1 opens and is enterable

**Exit:** Wave 4 boss → sector opens → exploration → safe return.

### Phase 5 · The collection loop — 5 days

- Survivor escort with authored waypoints and stuck recovery
- Guarded nest, egg carry, hatchery deposit
- Incubation on server timestamps, **idempotent** pet grant
- Minimal Dino Codex

**Exit:** an egg survives a disconnect and hatches into exactly one pet.

### Phase 6 · Players and hardening — 4–5 days

- Three classes with passives and abilities
- Downed state and revive
- Performance pass, mobile test, exploit pass

**Exit:** the MVP slice exit criteria in [05-sdlc-plan.md](archive/05-sdlc-plan.md) §9.

**Total: 26–31 days.** Roughly six working weeks, not four — the greenfield restart costs
about two weeks against the reskin plan, bought back in a codebase that is version-controlled,
typed and framework-backed rather than inherited.

---

## 6. Deferred

Unchanged from the previous plan, and for the same reason — build every system once, then
multiply content across systems that work.

Bosses 2–5 · Sectors 2–4 · Survivors 2–5 · Species 7–15 · Melee weapons and stamina ·
The full 20-weapon roster · Mutations · Difficulty modes · Monetization · Final art

---

## 7. Risks

| Risk | Mitigation |
|---|---|
| Fusion 0.3 scope API is unfamiliar and easy to misuse | Pin the version in Phase 0 and write one reference component before building many |
| Nevermore is large; pulling too much of it in | Take only ServiceBag, Binder, Maid, Signal, Rx. Add more only when a need is demonstrated |
| Rojo/Wally setup friction on Windows | Phase 0 is explicitly budgeted for it; no gameplay work until the round trip is proven |
| Grey-box becomes permanent | Blockout geometry is `.rbxmx` in `assets/`, replaceable without touching code |
| Rebuilding loses hard-won balance knowledge | §1.1 carries the derivations and invariants forward explicitly |
| Studio MCP used for authoring by habit | Code lives in files. MCP writes nothing except world geometry |

---

## 8. Immediate next actions

1. **Connect the official Roblox Studio MCP** and confirm it sees the Dino Hunt place —
   currently no Studio server is reachable from this session.
2. Confirm the Dino Hunt place is saved somewhere known, and whether it is a new place in an
   existing universe or a new universe entirely.
3. Begin Phase 0.
