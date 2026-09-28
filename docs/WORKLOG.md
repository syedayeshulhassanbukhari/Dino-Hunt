# Work log

Newest first. One entry per session that changes code, the docs or a decision.

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
