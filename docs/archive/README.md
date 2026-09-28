# Archived documents

These describe the **previous implementation**: a reskin of an existing Area 51 alien wave
shooter in a place called Dino Valley (placeId `88638180383535`). That place was deleted on
or before 2026-09-28 and the project restarted greenfield as **Dino Hunt**.

They are kept because the analysis in them is real work and parts remain useful, not because
they describe anything that still exists.

| File | Still useful for |
|---|---|
| `02-conversion-map.md` | The service decomposition (WaveService, ObjectiveService, …) carried into the new architecture. The derivation formulas in "Numbers already solved". |
| `04-inventory.md` | The effort ranking, and the finding that creature rigs are the long pole regardless of approach. |
| `05-sdlc-plan.md` | The 64 functional requirements, the seven non-functional families, and the exit criteria — **all still valid**, since the design did not change. Only the timeline and module-status sections are obsolete. |

**Do not follow their paths, module names or timelines.** Nothing in `ServerScriptService.ZombieAI`,
`ReplicatedStorage.Params` or `ServerStorage.ZombieTemplates` exists any more.

The current plan is [../06-project-plan.md](../06-project-plan.md).
