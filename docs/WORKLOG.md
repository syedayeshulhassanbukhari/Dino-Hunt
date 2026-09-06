# Work log

Newest first. One entry per session that changes the place, the docs or a decision.

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
