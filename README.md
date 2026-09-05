# Primal Rift — Outpost Siege

Roblox cooperative dinosaur wave-survival shooter. 1–6 players defend an Amber-powered
research outpost through 20 waves, clear 5 bosses, rescue survivors, recover eggs and
hatch permanent baby dinosaur companions.

## Three names, one project

These all refer to the same thing and none of them match. Do not be confused by this:

| Thing | Name |
|---|---|
| Local working directory | `D:\Game Development\Roblox\Games\Deno-Hunt` |
| GitHub repository | `syedayeshulhassanbukhari/Dino-Hunt` |
| Product / design doc | **Primal Rift: Outpost Siege** |
| Roblox place (build target) | **Dino Valley**, placeId `88638180383535` |

## Build target

Primal Rift is being built **into the existing Dino Valley place**, which currently runs a
complete and working Area 51 alien/zombie wave shooter. This is a **reskin in place**, not
greenfield development — the approach was confirmed on 2026-09-05.

Roughly half the spec's architecture already exists under alien-themed names. See
[docs/02-conversion-map.md](docs/02-conversion-map.md) for the full system-by-system mapping.

## Documents

| File | Contents |
|---|---|
| [Primal_Rift_MVP_Game_Design_Document.docx](Primal_Rift_MVP_Game_Design_Document.docx) | The MVP spec. v1.1, 26 Aug 2026. Source of truth for intent. |
| [docs/01-gdd-review.md](docs/01-gdd-review.md) | Review of that spec — contradictions, missing numbers, balance issues. |
| [docs/02-conversion-map.md](docs/02-conversion-map.md) | Every spec system mapped onto what exists in Dino Valley. |
| [docs/03-open-decisions.md](docs/03-open-decisions.md) | Decisions that block work, and the log of ones already made. |
| [docs/WORKLOG.md](docs/WORKLOG.md) | Dated record of work done, newest first. |
| [CLAUDE.md](CLAUDE.md) | Orientation for Claude Code sessions. |

## Working method

Claude Code connects to Roblox Studio over the Studio MCP bridge and edits the Dino Valley
place directly. This repo holds the **documentation, decisions and design data** — it is not
a Rojo project and does not currently sync source into the place.
