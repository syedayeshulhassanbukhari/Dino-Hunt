# Place snapshots

`DinoHunt-Snapshot.rbxm` is an export of the Dino Hunt place's code and blockout.

**This is the only version control the game code has.** There is no Rojo sync — code is
authored directly in the place over the Studio MCP (decided 2026-09-28), so nothing reaches
git automatically. If the place is lost or a change goes badly wrong, this file and Studio's
own coarse place history are the entire safety net.

## What it contains

| Path | Contents |
|---|---|
| `Workspace.Map` | The grey-box blockout — plaza, reactor, lanes, containment, sectors |
| `ReplicatedStorage.Shared` | Config, Net, Types, and the Util modules (Maid, Signal, ServiceBag) |
| `ServerScriptService.Server` | Bootstrap and Services |
| `StarterPlayer.StarterPlayerScripts.Client` | Bootstrap, Controllers, UI |
| `ReplicatedStorage.MatchState` | Replicated match attributes the HUD reads (Phase, Wave, Remaining, TimerEnd) |
| `StarterPack.Rifle` | The rifle Tool and its client script |
| `ServerStorage.Assets` | Creature rigs the spawner clones |

`ReplicatedStorage.Fusion` is deliberately **not** included — it is a third-party dependency
reinstallable from its GitHub release, and including it would bloat every snapshot.

## Restoring

Import the `.rbxm` into a place and re-parent the seven top-level instances to the services
named above. Fusion 0.3 must be installed separately from
`https://github.com/dphfox/Fusion/releases` (the `Fusion.rbxm` asset).

## Refreshing

Re-export from Studio over MCP with `export_rbxm`, pointing at those same seven paths.
Do this at the end of any session that changed code — DH-073.

## Known limitation

`.rbxm` is a binary format, so git stores it as an opaque blob: no diffs, no line-level
history, no meaningful review. It protects against loss, not against a bad change slipping
through unnoticed. A text export that writes each script to a `.luau` file would give real
diffs and is worth building once there is enough code to justify it.
