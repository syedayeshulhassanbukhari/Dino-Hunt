# Work log

Newest first. One entry per session that changes code, the docs or a decision.

---

## 2026-09-28 (Phase 1) — Grey-box blockout built

### Done

- **Blockout built in the place**, 63 parts under `Workspace.Map`:
  - **Plaza** 180x180 centred on the reactor, with a 60-stud defense ring marked on the floor
    (the line creatures cross to start attacking — FR-11).
  - **Reactor** at origin: 20x4 base, a neon column topping out at 57 studs with a point light.
    Deliberately tall — it has to read from the far end of a 130-stud lane.
  - **Three lanes** N/E/W, 28 wide x 130 long, 14-high walls, spawn markers at the mouths.
    Each carries an `OpensAtWave` attribute (1 / 6 / 11) matching the GDD's progressive lanes.
  - **Containment wing**, six bays along the south edge, open faces toward the reactor.
  - **Four sectors** on the diagonals behind colour-coded gates, each with `SectorId`,
    `RequiresBoss` and `RequiresReactorLevel` attributes so Phase 4's gating has something to
    read. Nest and survivor markers in each; hatchery pad in Sector 1.
- **Snapshot exported** to `snapshot/DinoHunt-Snapshot.rbxm` with a README — the only version
  control the game code has.

### Three bugs worth recording

- **Cylinder axes.** A Cylinder part's X axis is its length; Y and Z are the diameter. Sizing
  the defense ring `(60, 0.2, 60)` and rotating it upright produced a 60-stud pillar rather
  than a flat disc. Correct is `(thickness, diameter, diameter)`.
- **Hardcoded yaw.** I hardcoded the four sector corridor rotations and got **all four wrong**.
  Derived correctly it is `yaw = atan2(dir.X, dir.Z)`, giving -45 / 45 / 135 / -135.
  Deriving beats guessing even when the geometry looks obvious.
- **Template lighting.** The place ships a custom `Sky`, `Atmosphere`, `Bloom`, `SunRays` and
  `DepthOfField`. They blurred and dimmed the exact sightlines a blockout exists to test, and
  the custom Sky drew a moon regardless of `ClockTime`. Stashed to
  `ServerStorage._StashedLighting` rather than deleted — they are a reasonable starting point
  for the art pass.

### Tracker

DH-010 to DH-013 and DH-073 **Done**. 8 of 74 tasks, 8.1%.

### Next

DH-014: species table and derivation formulas in `Shared.Config`. Then the first creature.

---

## 2026-09-28 (Phase 0) — Fusion installed, core utilities written, first build runs

### Decided

- **No Rojo, no Wally. MCP-only authoring.** Reverses the filesystem-source-of-truth decision
  taken earlier the same day. Code is written directly into the place via `set_script_source`
  and `execute_luau`.
  - **Accepted cost: the game code is not in git.** Only docs and the tracker are. Mitigated
    by DH-073, a periodic `export_rbxm` snapshot committed here.
- **Nevermore dropped.** It ships no `.rbxm` — its releases are per-package Wally tags across
  a ~400-package monorepo whose dependencies only Wally resolves. Uninstallable without a
  package manager. Replaced by three modules we wrote ourselves.
- **Fusion pinned at 0.3** (`v0.3-beta`, the scoped API). Resolves D-13.

### Done

- **Fusion 0.3 installed** from the official GitHub release `.rbxm` into
  `ReplicatedStorage.Fusion`. Verified in-place: `version = {major = 0, minor = 3,
  isRelease = true}`.
- **Built the structure in-place:** `ReplicatedStorage.Shared.{Config,Net,Types,Util}`,
  `ServerScriptService.Server.{Services,Bootstrap}`,
  `StarterPlayerScripts.Client.{Controllers,UI,Bootstrap}`.
- **Wrote the three Nevermore replacements** in `Shared.Util`:
  - `Maid` — deterministic cleanup with named slots; connections disconnected before
    Instances are destroyed, since a connection firing mid-cleanup can touch something
    already gone.
  - `Signal` — pure-Luau event with a reused-thread pool. Chosen over BindableEvent because
    that serialises arguments, so tables cross as copies and metatables are lost — no good
    for services handing each other creature handles.
  - `ServiceBag` — two-phase `Init`/`Start` lifecycle. The split is the point: `Init` claims
    dependencies, `Start` runs only after every service is initialised, so mutually dependent
    services never observe each other half-built.
- **Verified in a playtest, not assumed.** Server log `[Dino Hunt] Server started — 0
  service(s)`; client log `[Dino Hunt] Client HUD live — Fusion 0.3, label reads "Wave 1"`.
  The label reading "Wave 1" rather than "Pre-wave" proves the Computed followed the Value.
  Screenshot confirms the HUD renders. No errors.

### Tracker

DH-005, DH-006, DH-008 **Done**. DH-001, DH-002, DH-003, DH-007 **Cut** — void without
Rojo/Wally. Added DH-073 (export snapshots) and DH-074 (prove the hand-written utilities).
74 tasks, 39.90 days, Phase 0 at 33%.

### Next

Phase 1. DH-010: blockout the reactor core and defense ring.

---

## 2026-09-28 (later) — Studio bridge live, task tracker built

### Done

- **Connected the Roblox Studio MCP** (`@chrrxs/robloxstudio-mcp`). Plugin installed to
  `AppData/Local/Roblox/Plugins/MCPPlugin.rbxmx`. Bridge verified against the live place.
- **Confirmed the place** — Dino Hunt, placeId `96665470525866`, gameId `10768413738`. Saved to
  Roblox. Essentially blank: ServerScriptService, ReplicatedStorage and StarterGui all empty.
  This resolves **D-11**.
- **Built [Dino-Hunt-Tasks.xlsx](../Dino-Hunt-Tasks.xlsx)** — 72 tasks across the seven phases,
  with dependencies, estimates, priorities, acceptance criteria and FR references. Three sheets:
  Tasks, Phases (formula rollups), Legend. Verified through Excel: 0 formula errors.

### Estimate revised upward

Decomposing the plan into individual tasks produced **39.15 days against the 26–31 day
phase-level estimate** — about 30% more, roughly a day per phase. The bottom-up figure is the
one to trust; `06-project-plan.md` has been corrected rather than left contradicting the
tracker.

Eight working weeks, not six. The time is in the systems, not in trimmable extras — a
meaningful cut means dropping a phase, and Phase 5 (the collection loop) is the only one whose
removal still leaves something playable.

### Next

Phase 0. Task DH-001: install and pin the Rokit toolchain.

---

## 2026-09-28 — Greenfield restart

The Dino Valley place was deleted and the project restarted from scratch in a new blank place
called **Dino Hunt**. The design is unchanged; only the implementation.

### Decided

- **Greenfield rebuild** on Fusion (UI) and Nevermore (architecture).
- **The filesystem is the source of truth** — Rojo syncs `src/` into Studio, so all gameplay
  code now lives in git. This reverses the old MCP-authoring arrangement, because Fusion and
  Nevermore are Wally-distributed and expect a filesystem project. It also closes the
  version-control gap the old D-06 raised.
- **The GDD remains the spec.** The defect review and the 64 functional requirements carry
  forward untouched.
- **Framework split is a rule:** Fusion owns the client view layer; Nevermore owns ServiceBag,
  Binder, Maid, Signal and Rx. Blend is not used.

### Done

- Wrote [06-project-plan.md](06-project-plan.md) — stack, repo layout, conventions, grey-box
  scope and a six-phase plan (26–31 days).
- **Rewrote CLAUDE.md**, which described the deleted codebase and would have misled every
  future session.
- Archived `02-conversion-map.md`, `04-inventory.md` and `05-sdlc-plan.md` to `docs/archive/`
  with a README explaining what in them survives. The 64 FRs and the exit criteria in 05 are
  still valid — only its timeline and module-status sections are dead.
- Reset the decisions doc: D-01…D-06 marked superseded, four new open items raised.

### Carried forward from the deleted codebase

The code is gone, the good ideas are not: derived stats from a single `lp` weight with
sub-linear damage; one server-wide concurrency cap; directional armour; and the invariants
(wave-1 dies to one shot, everything is outrunnable, nothing one-shots a full-health player).

### Blocked

- **D-10** — no Roblox Studio MCP is connected. The previous bridge disconnected and the
  official server has not replaced it. Phase 0 cannot complete without it.
- **D-11** — unknown where the Dino Hunt place lives (new universe or existing) and whether
  it is saved to Roblox.

### Next

Phase 0: connect the MCP, install and pin the toolchain, prove the Rojo round trip.

---

## 2026-09-06 — Inventory doc, GitHub connected

**Nothing in the Roblox place was modified.** Still read-only against Studio.

### Done

- **Wrote [04-inventory.md](04-inventory.md)** — a flat what-exists / what-is-missing audit,
  complementing the service-level conversion map.
  - Headline: the shooter is built, the game around it is not, and **zero dinosaur assets
    exist in the place**.
  - **24 of 26 enemy rigs are humanoid bipeds.** Only Cow and Alien dog are quadruped
    MeshPart rigs, so the existing skeletons are the wrong *shape* for dinosaurs, not just
    the wrong skin. This is the largest remaining item and it gates testing.
  - Recommended grey-boxing two existing rigs as stand-in dinosaurs so systems work can
    proceed before art lands.
- **Connected the GitHub remote and pushed.** The repo existed with a single hand-uploaded
  commit on `master` (`.gitignore`, `CLAUDE.md`, `README.md`, the design doc). Rebased the
  local commits on top of it rather than force-pushing, so that commit is preserved.
- **Renamed the local branch `main` -> `master`** to match the remote's default.
- **Added `.gitattributes`** to normalise line endings — the hand-uploaded files were CRLF
  and the local ones LF, which made identical files look different.

### Gotcha worth remembering

The push initially failed with `403: Permission to syedayeshulhassanbukhari/Dino-Hunt.git
denied to ayeshulhassan`. There are **two GitHub accounts** on this machine and the stored
credential is for the wrong one. Fixed by scoping the remote URL to the owning account:

```
https://syedayeshulhassanbukhari@github.com/syedayeshulhassanbukhari/Dino-Hunt.git
```

`gh` is installed (2.100.0) but still not authenticated; Git Credential Manager is now
configured as the global credential helper.

### Still blocked / unknown

- **D-01** match shape and **D-02** wave table ownership. Both block all wave work.
- Does the supplied dinosaur asset pack exist yet, and where?
- Is `Roblox-Zombie-Arena` still Rojo-syncing into the Dino Valley place? If it is, direct
  MCP edits to the place could be overwritten, and the real source of truth is that repo.

### Next

Settle D-01 through D-04, decide grey-box vs art-first, then fork the wave pipeline to a
finite 20-wave table.

---

## 2026-09-05 — Spec review, place survey, repo set up

**Nothing in the Roblox place was modified.** This session was read-only against Studio.

### Done

- **Reviewed the MVP design document** (v1.1, 26 Aug 2026, 14 sections). Found 7 critical
  problems, 4 balance issues and 6 undefined states. Written up in
  [01-gdd-review.md](01-gdd-review.md).
  - Headline: the pacing budget overruns its own 28-38 min target by its own numbers
    (34.3-45.5 min), and all three currencies have no values at all.
- **Verified the Studio MCP bridge.** One instance attached: Dino Valley, placeId
  `88638180383535`, Edit mode.
- **Surveyed the place** and discovered it is not empty — it runs a complete Area 51
  alien/zombie wave shooter with ~50 server scripts, a 13-module enemy AI package, a 27-gun
  weapon framework and a 300-wave table. Mapped every spec system onto it in
  [02-conversion-map.md](02-conversion-map.md): **4 reuse, 7 extend, 4 build new**.
- **Raised 4 blocking decisions** in [03-open-decisions.md](03-open-decisions.md), and
  recorded the 4 already settled.
- **Initialised this repo** with README, CLAUDE.md and docs.

### Decided this session

- Build target is the Dino Valley place.
- Approach is reskin in place, incrementally — not a fork, not greenfield.
- Map the existing systems before writing any code.
