# Dino Hunt

Cooperative dinosaur wave-survival shooter for Roblox. 1–6 players defend an Amber-powered
research outpost through 20 waves, clear bosses, rescue survivors, recover eggs from guarded
nests and hatch permanent baby dinosaur companions.

**Status:** Phases 0-3 complete and playtest-verified, Phase 4 half built. **40 of 75 tasks, 52% by effort.**
See [docs/08-status.md](docs/08-status.md).

## Stack

Rojo · Wally · **Fusion** (UI) · **Nevermore** (architecture) · Luau strict

Code lives in `src/` and syncs into Studio via Rojo. The filesystem is the source of truth.

## Documents

| File | Contents |
|---|---|
| [docs/08-status.md](docs/08-status.md) | **Start here.** How much is done, what is left, and how two people split the work. |
| [docs/06-project-plan.md](docs/06-project-plan.md) | Stack, architecture, repo layout, grey-box scope, phase plan. |
| [Primal_Rift_MVP_Game_Design_Document.docx](Primal_Rift_MVP_Game_Design_Document.docx) | The design spec. v1.1, 26 Aug 2026. |
| [docs/01-gdd-review.md](docs/01-gdd-review.md) | Review of that spec — contradictions, missing numbers, balance defects. Read before implementing anything it touches. |
| [docs/03-open-decisions.md](docs/03-open-decisions.md) | Decisions that block work, and the log of settled ones. |
| [docs/07-market-scan.md](docs/07-market-scan.md) | Who else is making this on Roblox, and what their numbers say. |
| [docs/WORKLOG.md](docs/WORKLOG.md) | Dated record of work, newest first. |
| [docs/archive/](docs/archive/) | The previous implementation's analysis. Describes a deleted codebase — see the archive README. |
| [CLAUDE.md](CLAUDE.md) | Orientation for Claude Code sessions. |

## History

An earlier implementation reskinned an existing Area 51 alien wave shooter in a place called
Dino Valley. That place was deleted and the project restarted from scratch on 2026-09-28.
The design did not change; only the implementation.

## Naming

The directory is `Deno-Hunt`, the repo is `Dino-Hunt`, the Roblox place is `Dino Hunt`, and
the design document says `Primal Rift: Outpost Siege`. All four are this project.
