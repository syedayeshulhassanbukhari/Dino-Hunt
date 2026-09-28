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

**The place.** No Rojo, no Wally, no filesystem sync — decided 2026-09-28. Code is authored
directly in the Dino Hunt place over the Studio MCP, using `set_script_source`,
`edit_script_lines` and `execute_luau`.

**Consequence: the game code is not in git.** This repo holds documentation and the task
tracker only. Periodically export the script tree with `export_rbxm` and commit it, or a
mistake is unrecoverable beyond Studio's own coarse version history.

## Stack

| Layer | Choice |
|---|---|
| UI and reactive state | **Fusion 0.3** — at `ReplicatedStorage.Fusion` |
| Architecture | Hand-written — `Maid`, `Signal`, `ServiceBag` in `ReplicatedStorage.Shared.Util` |
| Types | Luau `--!strict` on new modules |

**Nevermore is not used.** It is Wally-only and ships no `.rbxm`, so it cannot be installed
without a package manager. We wrote the three pieces we actually needed instead.

**Fusion 0.3 is scoped.** Create a scope, pass it as the first argument to every constructor,
destroy the scope to destroy everything made with it, and use `peek()` to read a state object
outside a `Computed`. There is no implicit global scope — do not write 0.2-style Fusion. The
reference component is `StarterPlayerScripts.Client.Bootstrap`; follow its shape.

## Where the code lives

| Concern | Path |
|---|---|
| Fusion | `ReplicatedStorage.Fusion` |
| Shared utilities | `ReplicatedStorage.Shared.Util` — Maid, Signal, ServiceBag |
| **All tunable numbers** | `ReplicatedStorage.Shared.Config` |
| Remote definitions | `ReplicatedStorage.Shared.Net` |
| Server services | `ServerScriptService.Server.Services` |
| Server entry point | `ServerScriptService.Server.Bootstrap` |
| Client entry point | `StarterPlayer.StarterPlayerScripts.Client.Bootstrap` |
| Client UI | `StarterPlayer.StarterPlayerScripts.Client.UI` |

A service is a ModuleScript in `Server.Services` returning a table with optional
`Init(serviceBag)`, `Start()` and `Destroy()`. Bootstrap claims every child automatically —
adding a service needs no bootstrap edit. Claim dependencies in `Init`, do everything else in
`Start`.

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
