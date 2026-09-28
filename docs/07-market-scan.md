# Market scan — who else is making this on Roblox

Searched 2026-09-28. Roblox catalogue, Rolimon's analytics, DevForum, Wikipedia game list.

The GDD's §14 "Market Reference Snapshot" listed five games as design references. This checks
whether they are still live, what their numbers actually look like now, and — the question
that matters — whether anyone is already shipping Primal Rift's loop.

---

## 1. The closest thing: **Dino Hunters** (Fishgig Game)

The GDD cited this as reference. It is now the nearest competitor, and its numbers are the
most useful thing in this document.

| | |
|---|---|
| Visits | 26.9M |
| Favourites | 59,883 |
| Rating | 92.5% |
| Max server | 8 |
| Created | ~June 2026 (3 months ago) |
| Last update | hours ago — actively maintained |

**Its loop:** shoot and weaken dinosaurs → capture with capsules → train and feed → combine
ranks → evolve named forms → battle with the improved team → missions → bosses → collect eggs
→ unlock stronger areas.

**Its concurrent-player curve is the story:**

| When | CCU |
|---|---|
| All-time peak, 21 Jul 2026 | **16,766** |
| 30-day average | 2,957 |
| 7-day average | 1,821 |
| At time of scan | 1,280 |

**A ~92% fall from peak in roughly two months, while still shipping updates hourly.**

Two conclusions, and they pull in opposite directions:

- **Demand is proven.** A dinosaur shooter with egg collection reached 16.7k concurrent. The
  audience the GDD is aiming at exists and is large.
- **Retention is the whole game.** Dino Hunters has a 92.5% rating, 60k favourites and an
  active developer, and it still lost nine players in ten. Whatever it does, it does not make
  people come back on day 7. This is the same cliff the GDD's own KPI table worries about.

---

## 2. The commodity problem: **Dino Defenses**

Not a hit game — a **game template for sale** on BuiltByBit. "Fully scripted, highly
optimized, mobile-friendly", endless waves of scaling dinosaurs, **boss encounters every 10
waves**, grid-based building, auto-targeting turrets, balanced early-game scaling, ready to
monetise.

That is most of Primal Rift's Phase 1–3 surface, purchasable off the shelf.

**Implication:** the wave-defense layer is not a moat. Anyone can buy it this afternoon. What
cannot be bought is the part of the GDD nobody else is building — see §4.

---

## 3. Adjacent, not competing

| Game | What it is | Overlap |
|---|---|---|
| **Prior Extinction** | Realistic survival where you *play as* a dinosaur, 20+ playable species | Theme only. Different genre. |
| **Dino Egg Farm** | Hatch eggs, mutations, sell dinos, get rich | Collection loop only |
| **Collect Dinos** | Dig eggs, hatch, build a park, earn | Collection loop only |
| **Grow a Garden** (Prehistoric update) | Dino eggs as a limited event | Egg meta, huge audience |
| **Base vs Waves**, **Mutant Plants: Base Defense**, **Defend your base from zombies** | Generic base defense | Wave layer only |
| **Hunty Zombie** | Post-apocalyptic co-op wave shooter, up to 7 players, cash between waves | **Closest structural match** — wave shooter with between-wave upgrades |

The egg-and-mutation collection meta is thoroughly occupied — by Dino Hunters, by the farm
games, and by Grow a Garden's event. **Primal Rift's hatchery and Codex are not
differentiators.** They are table stakes.

---

## 4. What nobody appears to be shipping

No game found combines:

- a **defended objective** that can be lost (our reactor), with
- **timed exploration windows** gated on two conditions (boss cleared **and** a resource gauge), with
- **survivor rescue and escort** as an objective, inside
- a **finite 20-wave run** with a real win state.

Every competitor is either endless (Dino Defenses, Hunty Zombie), open-world progression
(Dino Hunters, Prior Extinction), or pure collection (the farm games).

**That combination is the moat.** Specifically the *exploration window*: the GDD's idea that
beating a boss buys you 150 seconds of calm to go and do something else entirely, then return
to defend. Nothing in this scan does that, and it is the single most distinctive thing in the
document. It is also — not coincidentally — Phase 4, the next phase.

---

## 5. Actionable: the name collides

**Our place is called "Dino Hunt". The competitor is "Dino Hunters", 26.9M visits, created
three months ago, updated hourly.**

- Roblox search for "Dino Hunt" will surface Dino Hunters above us indefinitely.
- Players who find us may reasonably read us as a clone of it.
- The GDD's §2.1 branding rule anticipated franchise confusion, not collision with a live
  Roblox hit.

**This is already solved and just needs applying:** the GDD's product name is
**Primal Rift: Outpost Siege**, which is distinctive and collides with nothing found. Rename
the place before it is ever published. `Dino Hunt` / `Dino-Hunt` can stay as the internal
directory and repo name — see the four-names table in the README.

Raised as **D-14** in [03-open-decisions.md](03-open-decisions.md).

---

## 6. Retention advice from developers who have hit this wall

From the DevForum thread on a wave-based zombie shooter with good session time and conversion
but poor D1/D7 — the same shape of problem Dino Hunters appears to have:

1. **"Daily rewards do not increase retention very well. The key point is whether the game is
   really fun."** Cosmetic and login systems do not fix a repetition problem.
2. **Repetitiveness is the named killer.** Advice: more randomness, and audit whether
   progression difficulty and reward satisfaction actually motivate a return.
3. **Onboarding is a retention lever** — early abandonment is the hardest thing to diagnose
   and the most valuable to fix.
4. **UI disconnected from theme** reads as unfinished and costs trust.
5. **Ask the players who quit.** "You miss the most obvious stuff because you're looking at
   them from the point of view of a developer."

### How this lands on our existing risks

Point 2 is already risk #3 in the archived SDLC plan ("Twenty waves may feel samey") — this
scan promotes it from a theoretical risk to **the observed cause of a competitor's collapse**.

And it sharpens a number from the Phase 3 playtest: waves are clearing in **14–32 seconds**
against the GDD's 40–65s target, with wave composition as the only thing that changes between
them. Twenty short, structurally identical waves is precisely the repetition failure described
above. The GDD's own mitigations — lane openings, objective events, boss checkpoints, perks,
role-mixed compositions — are mostly **not yet built**, and several sit in Phase 6.

**Recommendation:** treat the per-wave *event* column in `Config.Waves` as load-bearing
content rather than flavour text. Waves 6, 11, 18 and 19 already carry authored events (a lane
opening, a power outage, no repair until clear). Those are the variety, and they are currently
printed to the log and otherwise ignored.

---

## Sources

- [Dino Hunters — Roblox](https://www.roblox.com/games/119031147890918/Dino-Hunters) · [Rolimon's analytics](https://www.rolimons.com/game/119031147890918)
- [Dino Defenses — BuiltByBit template](https://builtbybit.com/resources/dino-defenses.107159/)
- [Prior Extinction](https://www.roblox.com/games/6698800091/Prior-Extinction-Dinosaur-Survival)
- [Dino Egg Farm](https://www.roblox.com/games/129829790324321/Dino-Egg-Farm) · [Collect Dinos](https://www.roblox.com/games/132169393925482/Collect-Dinos)
- [Base vs Waves](https://www.roblox.com/games/138263805195687/Base-vs-Waves) · [Mutant Plants: Base Defense](https://www.roblox.com/games/105954652742326/Mutant-Plants-Base-Defense)
- [DevForum — retention for a wave-based zombie shooter](https://devforum.roblox.com/t/looking-for-advice-on-improving-retention-for-a-wave-based-zombie-shooter/3307981)
- [DevForum — survival game retention & monetization](https://devforum.roblox.com/t/survival-game-retention-monetization/3022564)
- [Roblox Grow a Garden prehistoric pets](https://www.pocketgamer.com/roblox/dino-egg-pets-in-grow-a-garden/)
