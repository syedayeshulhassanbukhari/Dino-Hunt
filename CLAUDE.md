# Orientation for Claude Code

## What this project is

**Dino Hunt** — a cooperative dinosaur wave-survival shooter for Roblox. 1–6 players defend
an Amber-powered research outpost through 20 waves, clear bosses, rescue survivors, recover
eggs and hatch permanent companions.

The design spec is [Primal_Rift_MVP_Game_Design_Document.docx](Primal_Rift_MVP_Game_Design_Document.docx)
("Primal Rift: Outpost Siege"). Its known defects are catalogued in
[docs/01-gdd-review.md](docs/01-gdd-review.md) — read that before implementing anything it
touches, because several requirements deliberately deviate from the spec.

**This is a greenfield build.** An earlier implementation reskinned an existing Area 51 alien
shooter; that place was deleted and the project restarted on 2026-09-28. Anything in
`docs/archive/` describes code that no longer exists.

Current plan: [docs/06-project-plan.md](docs/06-project-plan.md).

## Source of truth

**The filesystem.** Code lives in `src/`, Rojo syncs it into Studio. This is the reverse of
the old arrangement and it is deliberate — it puts all gameplay code in git.

**The Studio MCP does not author code.** It is for inspecting the live data model,
running play-tests, reading the console and capturing screenshots. The single exception is
world geometry: blockout is built in Studio and committed as `.rbxmx` under `src/assets/`,
because positioning a map by typing CFrames is worse than doing it visually.

## Stack

| Layer | Choice |
|---|---|
| Tools | Rokit → Rojo, Wally, Selene, StyLua |
| UI and reactive state | **Fusion** |
| Architecture | **Nevermore** — ServiceBag, Binder, Maid, Signal, Rx |
| Types | Luau `--!strict` on new modules |

**The framework split is a rule, not a preference.** Fusion owns the client view layer and
nothing else. Nevermore owns service lifecycle, tag binding, cleanup and cross-boundary
events. Do not use Blend. Do not use Fusion for server state.

Fusion's API changed significantly at 0.3 (explicit scopes, `peek`). Check the pinned version
in `wally.toml` before writing a component — do not write from memory of a different version.

## Conventions

- **`src/shared/Config` is the single source of truth for numbers.** No gameplay constant is
  declared anywhere else.
- **Stats are derived, not hand-tuned.** Each species gets one difficulty weight `lp`;
  everything else follows: `baseHP = k₁ × lp`, `damage = k₂ × √lp`, rewards likewise. Damage
  is sub-linear on purpose — linear scaling makes late creatures hit absurdly hard.
- **Every non-obvious number carries a comment** saying why, when and who asked.
- Services declare dependencies and are started by `ServiceBag`. No reaching across via
  `require` of a running singleton.
- Remotes are defined once in `src/shared/Net` and validated server-side without exception.
- One folder per service, `init.luau` plus siblings. No 3,000-line modules.

## Invariants to protect

Carried forward from the deleted codebase, where they were load-bearing:

- The wave-1 creature dies to **exactly one** starter-weapon shot.
- **Every creature is slower than the player** — anything can be outrun.
- **Nothing one-shots a full-health player.**
- Concurrency is `min(perPlayer × livePlayers, MAX_ACTIVE)` — **one server-wide cap**. AI
  count is a server property; it cannot be tiered per device.
- The server owns damage, ammo, currency, drops and saves. The client never reports a hit.

## Working agreement

- Update [docs/WORKLOG.md](docs/WORKLOG.md) when a session changes code or a decision. Newest
  entry first.
- When a blocking question resolves, move it from "Open" to "Decided" in
  [docs/03-open-decisions.md](docs/03-open-decisions.md) with the date and reasoning.
- Push doc and worklog updates to `master` as part of finishing a session. Ask first before
  force pushes, branch changes or history rewrites.
- The remote URL must carry the owning account or pushes 403 — there are two GitHub accounts
  on this machine. See [docs/WORKLOG.md](docs/WORKLOG.md) 2026-09-06.
