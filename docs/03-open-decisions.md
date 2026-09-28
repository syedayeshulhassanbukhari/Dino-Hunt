# Decisions

Blocking questions first, then the log of what has been settled.
When a decision is made, move it down with the date and the reasoning.

> **Reset 2026-09-28.** The project restarted greenfield. Decisions D-01 through D-06 concerned
> the deleted Area 51 codebase and are recorded at the bottom as superseded — several resolved
> themselves by the restart.

---

## Open

### D-12 — Creature rigging approach

The previous project's fatal content problem was that 20 creatures need non-humanoid rigs and
animation sets, and nothing existed to reuse. That has not changed — greenfield makes it
starker, since there are now no rigs at all.

**Options.** (a) Part-built grey-box rigs now, commission or buy final art later.
(b) Buy a low-poly dinosaur asset pack up front and rig against its skeletons.
(c) Build custom rigs in Blender — a Blender MCP is available in this session.

**Blocks:** nothing before Phase 1, but it determines whether Phase 1's placeholder work is
throwaway or foundational.

---

### D-13 — Package versions

Fusion's API changed significantly at 0.3 (explicit scopes, `peek`). The pinned version
determines how every UI module is written, and Nevermore's package set needs deciding rather
than pulling wholesale.

**Resolve during Phase 0**, and record the pins here.

---

## Decided

### 2026-09-28 — D-10 resolved: Studio MCP connected

`@chrrxs/robloxstudio-mcp` added to the project's MCP config; `MCPPlugin.rbxmx` installed to
`AppData/Local/Roblox/Plugins/`. Bridge verified against the live place.

Note this is a community package, not Roblox's own `studio-rust-mcp-server`. Worth revisiting
if it proves unreliable.

### 2026-09-28 — D-11 resolved: the Dino Hunt place

**placeId `96665470525866`, gameId `10768413738`**, saved to Roblox. Effectively blank —
ServerScriptService, ReplicatedStorage and StarterGui all empty. This is the Rojo sync target
and the publish destination.

### 2026-09-28 — Greenfield rebuild on Fusion and Nevermore

The Area 51 reskin is abandoned; the place was deleted. Rebuild from scratch in the new
**Dino Hunt** place using Fusion for UI and Nevermore for architecture.

**Reasoning:** user judgement that the inherited codebase was not good enough to carry.

### 2026-09-28 — The filesystem is the source of truth

Code lives in `src/`, Rojo syncs into Studio, and all gameplay code is in git. The Studio MCP
inspects and tests but does not author, except for world geometry committed as `.rbxmx`.

**Reasoning:** Fusion and Nevermore are Wally-distributed and expect a filesystem project, so
the toolchain forces this — and it closes the version-control gap that superseded D-06 raised.

### 2026-09-28 — The GDD remains the spec

Same game design, new codebase. The defect review in
[01-gdd-review.md](01-gdd-review.md) and the 64 functional requirements in
[archive/05-sdlc-plan.md](archive/05-sdlc-plan.md) both still apply.

### 2026-09-28 — Fusion for UI, Nevermore for architecture

The two overlap; the split is a rule. Fusion owns the client view layer. Nevermore owns
ServiceBag, Binder, Maid, Signal and Rx. Blend is not used.

**Reasoning:** avoids two reactive paradigms tangled through one codebase.

### 2026-09-06 — Version control on GitHub

Repo is `syedayeshulhassanbukhari/Dino-Hunt`, branch `master`. Now holds game code as well as
documentation.

---

## Superseded by the 2026-09-28 restart

| ID | Was | Outcome |
|---|---|---|
| D-01 | 300 endless waves vs 20 finite | Moot — greenfield builds 20 finite per the spec |
| D-02 | Wave table generator ownership | Moot — no inherited table exists |
| D-03 | Defended objective | Still required, now a clean build (Phase 3) |
| D-04 | Dead SWAT framework layer | Moot — deleted with the place |
| D-05 | Rojo sync vs MCP iteration | **Reversed.** Rojo now syncs; see 2026-09-28 above |
| D-06 | Game code had no version control | **Resolved** by the filesystem source of truth |
