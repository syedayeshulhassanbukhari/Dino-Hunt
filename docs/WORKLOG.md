# Work log

Newest first. One entry per session that changes code, the docs or a decision.

---

## 2026-09-28 (Phase 4a, playtest 1) — the boss works; three numbers were wrong

Every system fired. `DH-042`, `DH-043`, `DH-044` done. The value of the run was the three
things it proved wrong.

### 1. Every pounce missed. 0 of ~25.

Not luck — arithmetic. The telegraph radius was 12 studs and a player covers
`17.6 x 1.2 = 21 studs` during the warning. **Any** movement escaped, so the attack could
never land, and the flank window (x1.5 damage) sat permanently open.

Radius 12 -> **16**. Escape now needs 13.3 studs/s, 76% of walk speed: deliberate movement
still saves you, standing still and shooting does not. The warning stays at 1.2s because
FR-22 floors it at 1s.

**This is the second time a circle's size has been the bug rather than the code** — the
defense ring was the first. Ground-attack geometry needs checking against player speed, not
eyeballed.

### 2. The fight took 187s against a 100s prediction.

The derivation assumed ~120 effective dps, from 140 raw against a 73% magazine duty cycle.
**Measured: 64 dps.** The estimate ignored everything that is not shooting the boss — killing
adds, repositioning, aiming.

HP 12,000 -> **7,000**, which is ~109s at the measured rate. The comment in `Config.Bosses`
now says to start boss 2 from 64 dps, not from the arithmetic.

### 3. The stall warning fired during a legitimate boss fight.

`wave 4 stalled — no deaths for 90s, 1 alive` — correct by its own rule, wrong in substance:
a 90-150s boss fight with the adds already dead is exactly the intended shape. An active boss
now counts as liveness.

### Instrumentation added, because two things were invisible

- **Pounce hits are now logged, not just misses.** With only misses printed there was no way
  to distinguish a dodgeable attack from an impossible one. Hit rate and effective dps are
  both reported at boss death.
- **Damage to players is now logged**, rolled up per second. **DH-075 is completely
  unverified**: the player never came within AttackRange, so creature-on-player damage has
  still never been observed. Fourteen creatures biting would be ~10 lines a second, hence the
  rollup rather than per-bite logging.

### Also worth noting

The reactor took **3,170 damage during wave 4** and finished at 46%. Most of that is the boss
itself: it walks to the ring like any creature and chews the reactor for ~12.6 dps, which over
187 seconds is ~2,360. Shortening the fight to ~109s should roughly halve it. Whether a boss
*should* attack the reactor is a real design question — the GDD does not say — but the
pressure it creates seems right.

---

## 2026-09-28 (Phase 4a written) — the boss

Boss half of Phase 4 written and load-checked. Exploration follows next.

### A gap in my own task list

**Creatures never damaged players.** `Config.Creatures` has carried `damage`,
`AttackInterval` and `AttackRange` since Phase 1 and nothing used them — creatures walked
past you to the reactor. No task covered it; Phase 6 has downed/revive but nothing said
"creatures can hurt you".

A boss whose telegraphed pounce cannot hurt anyone is a punching bag, so the boss work needed
it. Added as **DH-075** rather than folded in silently. 75 tasks now.

Implemented per GDD §3.2 — "dinosaurs prefer nearby players but attack the reactor when they
reach its defense ring". They do **not** chase: a creature walks its lane and bites whoever
steps into reach. Chasing needs real pathfinding and would break "everything is outrunnable"
in spirit, since a player could never disengage. One Heartbeat loop for all creatures, the
same pattern as the reactor tick.

### Boss design

`Config.Bosses` holds definitions; `BossService` reads it and nothing else, so boss 2 should
be a table entry plus any new attack verbs.

**Alpha Raptor Matriarch**, 12,000 HP solo. Derived: the rifle is 140 dps raw, but a 30-round
magazine against a 1.6s reload is a 73% duty cycle => ~102 dps sustained, ~120 with some weak
points. 12,000 / 120 = **100 seconds**, inside the GDD's 90-150s band.

Three phases on health fractions — Hunt, Pack (summons 6 raptors, 3s invulnerable), Frenzy
(pounce every 2.5s). Every FR-25 verb is present: summons, hunter mark, leap, and a flank
window (x1.5 damage for 4s) when a pounce lands on nobody.

**Player scaling deviates from the GDD deliberately.** §7.1 says boss HP scales
1 + 0.50 x (players-1) = 3.5x at six players against roughly 6x incoming damage, so a full
party kills a boss in half a solo player's time. That inversion is defect #4 in
01-gdd-review. Using 0.85 per extra player gives 5.25x, close enough to linear that party
size stops being an exploit.

### Rules enforced in code, not trusted to data

- **FR-21** the 8-second invulnerability ceiling is clamped in `BossService`, with a warn if a
  phase asks for more. The longest window actually used is printed at boss death.
- **FR-22** telegraph lead time. The indicator marks where the target is *now* and resolves
  there after the warning — a promise about a place, not a homing missile, so moving out is
  always the right answer.
- **FR-23** the checkpoint reward is granted on death, immediately, so losing wave 17 cannot
  erase having beaten boss 4.
- **FR-24** phase transitions respawn defeated players.

Statically verified: boss slower than the player (16.5 < 17.6), pounce does not one-shot
(34 < 100), warning >= 1s, every phase under the invulnerability cap.

### Design note

The boss is registered as a **normal CreatureService record** with `isBoss`, so CombatService
shoots it, the armour arc applies, ObjectiveService sees it hit the reactor, and the wave
clear check counts it — all unchanged. A separate boss entity would have forced every
consumer to check two places for "is this shootable". Bosses skip the pool, because a scaled
model returned to it would come back out at the wrong size.

### What to look for

```
[BossService] Alpha Raptor Matriarch spawned on LaneNorth_JungleGate — 12000 HP (1 player(s)), 3 phases
[BossService] Alpha Raptor Matriarch phase 1/3 — Hunt
[BossService] marked hunter: <name>
[BossService] pounce missed — flank open for 4.0s (x1.50 damage)
[BossService] Alpha Raptor Matriarch phase 2/3 — Pack
[BossService] summoned 6/6 Velociraptor, invulnerable 3.0s
[BossService] Alpha Raptor Matriarch DEFEATED in 104s — longest invulnerability 3.0s (cap 8s), 1 boss(es) cleared
[BossService] checkpoint reward — 250 credits to 1 player(s)
```

---

## 2026-09-28 (market scan) — a competitor's collapse is the most useful data we have

Searched the Roblox catalogue, Rolimon's, DevForum and Wikipedia's game list to answer "does
this already exist". Written up in [07-market-scan.md](07-market-scan.md).

### The headline

**Dino Hunters** (Fishgig Game) — cited in the GDD's own §14 as a reference — is now the
nearest competitor, and its curve is the finding:

| | |
|---|---|
| Peak CCU, 21 Jul 2026 | **16,766** |
| 30-day average | 2,957 |
| 7-day average | 1,821 |
| At scan | 1,280 |

26.9M visits, 92.5% rating, 60k favourites, updated hourly — and still down ~92% from peak in
two months. **Demand for a dinosaur shooter with egg collection is proven at 16.7k concurrent;
retention is where it dies.**

### Three things that change our thinking

1. **The collection loop is table stakes, not a differentiator.** Eggs, hatching, rarity and
   mutations are already occupied by Dino Hunters, Dino Egg Farm, Collect Dinos and Grow a
   Garden's prehistoric event. The GDD treats the hatchery as the retention hook; the market
   says it is the price of entry.
2. **The wave layer is a commodity.** "Dino Defenses" is a *template for sale* on BuiltByBit —
   endless scaling dinosaur waves, bosses every 10 waves, grid building, turrets, mobile-ready.
   Most of our Phases 1-3 can be bought this afternoon.
3. **The moat is the exploration window.** Nothing found combines a losable defended objective,
   timed exploration gated on boss *and* resource, survivor escort, and a finite run with a
   real win. Competitors are endless, open-world, or pure collection. The GDD's most
   distinctive idea is also the next thing we build.

### Retention advice, and how it lands on us

DevForum, from developers with the same D1/D7 problem: daily rewards do not fix retention;
**repetitiveness is the named killer**; onboarding is the lever; ask the players who quit.

This promotes our archived risk #3 ("twenty waves may feel samey") from theoretical to
**observed cause of a competitor's collapse** — and it sharpens the Phase 3 number: waves
clear in **14-32s** with composition as the only variable. Twenty short, structurally
identical waves is exactly that failure.

Raised **D-15**: `Config.Waves` already carries an authored `Event` per wave — "River Breach
opens", "Power outage 25s", "No repair until clear". Only the lane openings are implemented;
the rest print and do nothing. Those events are the GDD's own variety mechanism and are cheap
now, expensive after twenty waves are tuned.

### Also raised

**D-14 — the place name collides.** "Dino Hunt" versus "Dino Hunters" (26.9M visits, created
three months ago). Search will bury us and players may read us as a clone. Already solved by
the GDD's own product name, **Primal Rift: Outpost Siege**; the place just needs renaming
before it is ever published.

---

## 2026-09-28 (Phase 3 complete) — the reactor is the objective

**Phase 3 is 11/11. The project is 37/74 (48%).** Exit gate met: a run can be won (Phase 2's
VICTORY) and lost (`DEFEAT on wave 19 — reactor destroyed`), and the reactor is what decides it.

### Verified from the user's log

```
wave 1 — took 839 damage, repaired 0, peak 14 attacker(s), reactor 4160/5000 (83%)
ALARM — reactor below 35% (1642/5000)
reactor destroyed  ->  DEFEAT on wave 19 — reactor destroyed
```

DH-031, DH-032 and DH-034 done. The HUD tasks (039-041) are marked done on functional
evidence — mounted clean across three runs with zero client errors, and every attribute it
reads verified server-side — but **the visual layout has not been reviewed by eye**.

### Balance data, the real value of this run

First real numbers on reactor pressure:

| | Observed | Predicted |
|---|---|---|
| Wave 1 damage taken | 839 (17% of the bar) | — |
| Wave 1 creatures reaching the ring | 14 of 17 | — |
| Wave 19 reactor lifetime | 26.5s from wave start | ~10s at full contact |

The wave 19 figure reconciles: creatures need ~12s to walk in, so there were only ~14s of
real contact, averaging ~357 dps against the 524 predicted for all 24 at the ring
simultaneously. The derivation holds.

**Repair economics work out on paper.** Wave 1 pays roughly 100 credits, which is 2.5 repairs
= 1250 HP against 839 taken. Sustainable, but not comfortably — which is probably correct.

**The open design question for Phase 6:** 14 of 17 creatures reached the ring on wave 1 while
the player was actively shooting. Either that is intended — the reactor is meant to take
damage and be repaired, and the player cannot hold a lane alone — or kill speed and creature
speed need work. Worth deciding deliberately rather than letting the numbers decide it.

Wave 1 clear time is now **25s**, up from 15s in Phase 2 but still short of the GDD's 40-65s.
Expected to keep rising as creatures gain real attacks.

### Not implemented, noted

Wave 19's authored event is "No repair until clear". It is printed but not enforced. Phase 6.

### Fixed this session

The per-wave summary only fired on `WaveCleared`, so the wave that destroys the reactor — the
one whose numbers most need explaining — produced no summary at all. It now also fires on
`MatchLost`, tagged `LOST` rather than `cleared`.

### Next

Phase 4 — boss and exploration. DH-042: `BossService`, the phase state machine.

---

## 2026-09-28 (Phase 3, playtest 1) — Amber and repair verified, damage instrumented

### Verified from the user's log

- **DH-033 repair.** `Shayankhankhan7 repaired +500 (5000/5000) for 40 credits`, twice.
  Prompt, hold, credit cost and cooldown all work — and the credits were earned from kills,
  so the award path works too.
- **DH-035 / DH-036 / DH-037 Amber.** `REACTOR LEVEL 2 — 255 Amber (threshold 250)` at ~97s
  into the match, which is the wave 4-5 window the roster sums predicted. Amber only accrues
  from `CreatureDied` drops credited on shard arrival, so reaching 255 exercises the whole
  chain: drop, flight, credit, threshold, level-up.

Phase 3 is 5/11; the project is 31/74.

### What the log could not show, and why

**Reactor damage has no log line.** DH-031 and DH-032 could only be *inferred*: the repair
guard returns early at full health, so the reactor must have been below 5000 both times.
That is a weak form of evidence, and the `+500 (5000/5000)` line was actively misleading —
it printed the nominal repair amount, not what was actually restored, so a top-up of 200 HP
looked like a full 500.

This is the second time instrumentation has been the bottleneck rather than the code. Logging
every damage tick would bury the run, so the fix is roll-ups:

- **One summary line per wave**: damage taken, damage repaired, peak simultaneous attackers,
  and the reactor's closing health and percentage.
- **An ALARM warn** the first time health crosses below 35%, re-armed when a repair lifts it
  back out. That crossing is the moment a player is meant to change what they are doing, so
  it earns its own line; nothing else about damage does.
- **Repair now reports what was actually restored**, computed from before/after.

### Still unverified

DH-034 (defeat at 0 HP) and the three HUD tasks. Defeat needs a run where the reactor is
allowed to die — `/wave 19` and standing back is the fast route, since wave 19 is ~524 dps
unopposed against 5000 HP.

---

## 2026-09-28 (Phase 3 written) — the reactor and the Amber gauge

All twelve modules load-checked clean. Awaiting playtest logs.

### The three numbers the GDD never gave

`01-gdd-review` flagged that reactor HP, damage-to-reactor and repair rate are undefined.
Derived rather than guessed, from one design statement recorded in `Config.Game`:

> "A full solo wave that reaches the reactor unopposed destroys it in about 30 seconds at
> wave 1, and about 10 at wave 19."

- **MaxHealth 5000.** Verified by computation: wave 1 unopposed is **159 dps → 31s**; wave 19
  is **524 dps → 9s**. Both ends land where the statement says.
- **AttackInterval 2.5s, uniform** — deliberately NOT the per-role interval. A swarmer
  nibbling a building every 1.2s makes a crowd delete it faster than any elite, which reads
  as wrong. This preserves "bigger creature hurts the objective more".
- **Repair +500 HP for 40 Credits**, 1s cooldown, 26-stud range. 10% of the bar per press:
  worth crossing the plaza for, not enough to out-heal a wave.

### Amber

Thresholds 250 / 600 / 1050 / 1600 (GDD §4.4) checked against the actual roster — solo income
reaches **339 / 740 / 1287 / 1910** by waves 4 / 8 / 12 / 16, so every level lands just before
its boss. **DH-038 Done**: `Game.amberThreshold` scales them with party size (six players:
563 / 1350 / 2363 / 3600, ×2.25 matching the spawn-budget income), which corrects the GDD's
flat thresholds — the defect raised in `01-gdd-review` §3.

Amber is server-owned end to end: no pickup, no backpack. A shard tweens to the reactor and
the value is credited **on arrival**, so the gauge never runs ahead of the visual. The
level-up check is a `while`, not an `if` — a late shard can cross two thresholds at once.

### Design notes

- **One attacker set, one loop.** Creatures at the ring join a set that a single Heartbeat
  tick drains; at 60 concurrent creatures that is one timer instead of sixty.
- **Creatures turn to face the reactor on arrival.** Not cosmetic: the armour arc is measured
  off facing, so flanking a Triceratops at the ring has to mean something.
- Damage is gated to `Phase == Active`, which is also how Phase 4's exploration window will
  suspend it for free (FR-32).
- `MatchLost` clears the attacker set — a wipe fires no deaths, so nothing else would.
- HUD moved out of Bootstrap into `Client.UI.Hud` + `Client.UI.Theme`. Bars are sprung so a
  hit reads as movement; the Amber gauge fills toward the *next* threshold rather than from
  zero, or it would barely move between levels 4 and 5.

### A tooling trap that invalidated earlier checks

**The edit peer caches `require` per ModuleScript instance for the whole Studio session.**
Re-requiring an edited module returns the stale table and never re-parses the source — so a
syntax error passes a "load check" silently, and stale values come back (this surfaced as
`Game.amberThreshold` being nil right after it was added). Load checks now **clone** the
containing folders into a temp parent and require the clones. Recorded in CLAUDE.md.

### What to look for

```
[ObjectiveService] reactor online — 5000 HP, level 1, next Amber 250
[ObjectiveService] REACTOR LEVEL 2 — 251 Amber (threshold 250)
[ObjectiveService] <name> repaired +500 (3200/5000) for 40 credits
[ObjectiveService] reactor destroyed
[WaveService] DEFEAT on wave N — reactor destroyed
```

---

## 2026-09-28 (Phase 2 complete) — victory and defeat verified

Phase 2 is **10 of 10**. The user asked Claude to run these two tests directly.

### Victory (DH-029)

`/wave 20` sent through the real chat path → `DevCommands` → `WaveService:JumpToWave`:

```
[WaveService] dev jump — wave 1 abandoned, next is 20
[WaveService] wave 20 started — 16 creatures across 3 lane(s), hp x2.33 dmg x1.67 — Final — BOSS ChronoRex not implemented until Phase 4
[WaveService] wave 20 cleared in 157s
[WaveService] VICTORY — 20 waves cleared in 3m 27s
```

HUD read VICTORY / All 20 waves cleared in green. **3 lane(s)** also verifies the lane-opening
logic across all three thresholds. A scripted client shooter did the killing: 552 shots,
standing 10 studs behind each target — the Triceratops took `hit`s, not `blocked`, which is
the armour flank rule working in play.

### Defeat (DH-030)

`/lose` on wave 1 with 17 creatures on the field: **17 → 0 instantly**, phase Active → Defeat,
`DEFEAT on wave 1 — dev command /lose`, HUD in red, no deaths fired, no stall warning.

**Bug found and fixed:** `Defeat()` published `Remaining` *before* `DespawnAll()`, so the
replicated attribute still read 17 after the wipe. Cosmetic today (the Defeat HUD line does
not show it); wrong for anything else that reads it. Despawn now precedes publish.

### Tooling notes

- `TextChannel:SendAsync()` **never returns for a command message**. Fire it inside
  `task.spawn` or everything after it is dead code — the first attempt lost the shooter
  thread that way.
- A blocking client eval that polls an attribute for ~27s is a usable sleep: the MCP
  waiter times out at 30s, and there is no other way to wait from this side.

### Phase 2 exit gate

"A full 20-wave run completes" — met. Waves 1–5 cleared by the user's own play; wave 20
reached by dev jump and cleared by scripted fire; every intermission, scaling multiplier and
lane threshold matched its config exactly.

### Next

Phase 3 — the spine. DH-031: reactor instance with server-owned HP. Creatures at the ring
finally get something to attack, which is also what will bring wave times up from the
current 14–32s toward the 40–65s target.

---

## 2026-09-28 (Phase 2, playtest 2) — five waves verified, dev commands added

### From the user's log (20:37–20:40, waves 1–5)

- **DH-021 Done.** Waves 1–5 cleared in 15 / 30 / 24 / 14 / 32s, each `cleared` firing on the
  last death, intermission then next wave every time. Scaling exact: hp ×1.07 → ×1.28,
  dmg ×1.03 → ×1.14 across waves 2–5.
- **DH-023 Done.** `wave 2 fully spawned — 22 creatures in 12.7s, held by cap on 16 tick(s)`:
  the first wave over the solo cap of 18 was throttled; the four under it held 0 ticks.
- Intermissions 15.01s. Reload cycle 1.64s against a 1.6s config. Boss-wave placeholder
  printed on wave 4. No stalls, no errors.
- Ammo feedback worked as designed: `empty — reloading` → `reloaded (30)`, repeatedly.

### Two findings from the same log

- **23 consecutive `blocked` on wave 3.** The Pachycephalosaurus had a 25° armour arc I
  added; the GDD gives it a headbutt, not a shield, and its armour tutorial is the
  Triceratops at wave 7. Blocked shots at wave 3 with no hit-marker feedback is bad
  onboarding. **Armour removed from Pachycephalosaurus.** Triceratops 35° and
  Ankylosaurus 50° stand.
- **Waves clear in 14–32s against the GDD's 40–65s target.** Expected at this stage —
  creatures do not attack yet, so there is no pressure to manage. Balance is Phase 6.
  Also noted for then: the GDD says front armour *reduces* damage; the inherited mechanic
  *blocks*. Blocking is more readable in grey-box; revisit with real hit feedback.

### Added

- **`DevCommands`** — Studio-only chat commands, guarded on `RunService:IsStudio()`:
  `/wave N` jumps the match clock (abandons the current wave with no deaths fired, so
  nothing is rewarded), `/lose` triggers `WaveService:Defeat`. Exists so victory and defeat
  can be verified without a 25-minute run.
- `WaveService` match loop now drives `_nextWave` instead of a for-counter so a jump can
  redirect it.

### A tooling note

The edit peer's `require` cache is stale for any module loaded earlier in the same Studio
session — a post-edit `require` returned the pre-edit table and its memoised stats. Verify
edits by reading `.Source`, not by requiring.

### Still to verify

VICTORY (`/wave 20`, then clear it) and DEFEAT (`/lose`) — DH-029, DH-030. The second lane
opening at wave 6.

---

## 2026-09-28 (Phase 2, playtest 1) — ammo feedback

### From the user's log

- Verified: boot (4 services), match start, 10s intermission exactly, **wave 1 = 17
  creatures on 1 lane**, HUD phase `Intermission`, 9 kills in ~3s through the real path.
  **DH-022 and DH-024 Done.**
- Not reached: `wave 1 cleared`. The user stopped at +31s with 8 alive, after **22 seconds
  with no shots landing**. 18 confirmed hits plus misses against a 30-round magazine: almost
  certainly an empty mag. The server dropped shots silently, the client kept drawing tracers,
  and nothing said "press R". The gun looked broken.

### Fixed

- `HitConfirm` now carries `ammoLeft` on every reply, and two new kinds: `empty` (throttled
  to one per second — auto-fire would otherwise send seven) and `reloaded`.
- `RifleClient` auto-requests a reload on `empty` and prints ammo on every confirm:
  `[Rifle] hit (24 left)`, `[Rifle] empty — reloading`, `[Rifle] reloaded (30)`.
- `WaveService` prints `wave N fully spawned — 17 creatures in 2.5s, held by cap on 0 tick(s)`
  so batched spawning (DH-023) has log evidence rather than an assumption.

Both server modules load clean in edit. Snapshot re-exported.

### Still to verify

`wave 1 cleared`, the 15s intermission before wave 2, the River Breach lane opening at 6,
and (eventually) VICTORY. DH-021/023/029/030 stay In progress.

---

## 2026-09-28 (Phase 2 written) — the wave loop, awaiting playtest logs

**Working agreement changed:** the user runs the game; Claude does not start playtests.
Verification is by the logs the user provides. Recorded in CLAUDE.md. Print lines are now the
test surface, so every WaveService state change prints one specific line.

### Built (not yet verified in play)

- **`Config.Waves`** — GDD §7 transcribed: 20 waves, solo counts, boss placeholders, events.
  Species keys are asserted against `Config.Creatures` when WaveService loads, so a typo
  fails the boot instead of silently skipping a group. 344 solo creatures in total.
- **`WaveService`** — the match clock. Each wave is a *reservoir* of (species, lane) pairs
  drained 4 per 0.5s tick while the cap allows; cleared when the reservoir is empty **and**
  nothing is alive, triggered by `CreatureDied` rather than a timer. 15s intermissions (10s
  before wave 1). Victory after wave 20. `Defeat(reason)` is real — clears the reservoir,
  despawns everything, fires `MatchLost` — but nothing calls it until Phase 3's reactor.
  A stall watch warns if an active wave sees no deaths for 90s.
- **Multiplayer scaling** per GDD §7.1: budget ×(1 + 0.25·extra), health ×(1 + 0.07·(w−1))
  ×(1 + 0.15·extra), damage ×(1 + 0.035·(w−1)). `CreatureService:Spawn` takes the
  multipliers; the record keeps scaled `maxHealth`/`damage` beside the base stats.
- **Lane opening** reads the blockout's `OpensAtWave` attributes — Jungle Gate 1, River
  Breach 6, Maintenance Tunnel 11. Spawns round-robin across open lanes.
- **`ReplicatedStorage.MatchState`** — Phase / Wave / Remaining / TimerEnd as attributes.
  Server writes, client observes; no remotes. `Client.Bootstrap` now renders real match
  state: "Wave 3 in 12s", "Wave 3 — 14 remaining", VICTORY, DEFEAT.
- `_DevSpawner` deleted.

### Boss waves

Waves 4/8/12/16/20 spawn only their adds (8 / 6 / 10 / 9 / 16 creatures) and print
`BOSS <name> not implemented until Phase 4`. They will clear fast until then.

### A tooling trap, recorded

An edit-mode eval that `require`d `CombatService` hung, because `Remotes` took the client
branch in edit. That hung thread then blocked **every later require of the same module** —
Luau waits for an in-progress load — so a second eval also timed out even after the source
was fixed. The fix was to create the RemoteEvents in the edit DataModel by hand so the stuck
`WaitForChild` returned. Lesson: a module that yields at top level can wedge the whole edit
peer's require cache; `Remotes` now treats "not running" as the authority.

### Log lines to look for

```
[WaveService] match started — 1 player(s), 20 waves
[WaveService] intermission 10s before wave 1
[WaveService] wave 1 started — 17 creatures across 1 lane(s), hp x1.00 dmg x1.00 — Jungle Gate only
[WaveService] wave 1 cleared in 41s
[WaveService] wave 6 started — 22 creatures across 2 lane(s) ...   <- River Breach open
[WaveService] wave 4 started — ... BOSS AlphaRaptorMatriarch not implemented until Phase 4
[WaveService] VICTORY — 20 waves cleared in 24m 10s
```
No `[WaveService] wave N stalled` warnings, and no red errors.

### Tracker

DH-025 to DH-028 Done (delivered in Phase 1). DH-021 to DH-024, DH-029, DH-030 **In progress**
until the logs confirm them.

---

## 2026-09-28 (Phase 1 complete) — a creature walks a lane and dies to one shot

Exit gate met, with evidence rather than assumption. Phase 1 is 11/11; 16 of 74 overall.

### Built

- **`Shared.Config.{Game,Creatures,Weapons}`** — the numbers. Fifteen species, one `lp` each;
  `HP = 20 × lp`, `damage = 5 × √lp`, rewards on `√lp`. Load-time asserts fail loudly if any
  role is not slower than the player's 17.6, or any species could one-shot a player.
  The GDD's absolute HP bands (§5.1) are deliberately not used — their *ratios* are, because
  the one-shot invariant fixes the absolute scale. Basis: Compsognathus 20 HP = rifle 20 dmg.
- **Four grey-box rigs** in `ServerStorage.Assets.Creatures` — Swarmer, Flanker, Armoured,
  Ranged. Distinct silhouettes, one flat colour each, no animation. One skeleton contract:
  `HumanoidRootPart` primary, `Head` is the weak point, `ArmourPlate` on the Armoured rig.
- **`CreatureService`** — pool per rig, one server-wide cap `min(18 × live, 60)`, two derived
  waypoints per lane, damage and death owned in one place. Signals: `CreatureSpawned`,
  `CreatureDied`, `CreatureReachedReactor` (Phase 3's ObjectiveService hooks the last).
- **`CombatService`** — validates weapon, fire interval (10% slack), ammo, origin drift;
  raycasts server-side; friendly fire excluded. Directional armour factored into a pure
  `isBlockedByArmour()`.
- **`PlayerService`** — applies the 17.6 walk speed so the invariant actually holds.
- **Rifle** tool + `RifleClient`; **`_DevSpawner`** (throwaway, gated on `workspace.DevSpawn`).

### Evidence

- Pathing: **6/6 creatures reached the defense ring** in each of three runs.
- One-shot: a Compsognathus died on a single `kill` confirm during the user's own playtest;
  every other kill's arithmetic is exact (Triceratops 8 hits + kill = 9 × 20 = 180).
- Armour: **10/10 unit cases** — ±34° blocked, 36°+ lands, elevation ignored, no-armour
  species never blocked, Ankylosaurus 50° arc holds at 45°.
- Boot: `Server started — 4 service(s)`, zero errors, after every edit.

### Bugs found and fixed

- **Luau parse:** `(task :: () -> ())()` — a call on a parenthesised cast to a function
  type — is a syntax error. Cast to a local first. This took the server down on first boot;
  the symptom was "no Server started line", not a helpful error at the call site.
- **`SetStateEnabled` is runtime state, not a property.** Disabling `Dead` on the template
  rig does nothing for clones; it has to be re-applied on every spawn or the Humanoid kills
  itself at 0 HP before the service can.
- **MoveTo's 8-second timeout** on a 9-second lane. Re-issued before the deadline. And
  `MoveToFinished:Wait()` never fires for a creature parented to nil mid-walk — that would
  leak the thread — so arrival is polled.
- **Edit-mode `require` hung.** `RunService:IsServer()` is false in Studio edit, so Remotes
  took the client branch and `WaitForChild`'d forever; the MCP eval wedged. Anything not
  actively running now acts as the authority.

### Two facts about testing over the MCP worth keeping

- The server peer's `require` has **its own module cache** — it cannot see a live service's
  state. Test pure functions that way; test integration through the real remote path.
- A human play-tester will kill your test subjects. The scripted armour shot started 5s after
  the user had already killed the Triceratops. Factoring the logic pure was the right answer,
  not a longer wait.

### Next

Phase 2, DH-021: `WaveService` with 20 finite waves. `_DevSpawner` is deleted when it lands.

---

## 2026-09-28 (Phase 1) — Grey-box blockout built

### Done

- **Blockout built in the place**, 63 parts under `Workspace.Map`:
  - **Plaza** 180x180 centred on the reactor, with a 60-stud defense ring marked on the floor
    (the line creatures cross to start attacking — FR-11).
  - **Reactor** at origin: 20x4 base, a neon column topping out at 57 studs with a point light.
    Deliberately tall — it has to read from the far end of a 130-stud lane.
  - **Three lanes** N/E/W, 28 wide x 130 long, 14-high walls, spawn markers at the mouths.
    Each carries an `OpensAtWave` attribute (1 / 6 / 11) matching the GDD's progressive lanes.
  - **Containment wing**, six bays along the south edge, open faces toward the reactor.
  - **Four sectors** on the diagonals behind colour-coded gates, each with `SectorId`,
    `RequiresBoss` and `RequiresReactorLevel` attributes so Phase 4's gating has something to
    read. Nest and survivor markers in each; hatchery pad in Sector 1.
- **Snapshot exported** to `snapshot/DinoHunt-Snapshot.rbxm` with a README — the only version
  control the game code has.

### Three bugs worth recording

- **Cylinder axes.** A Cylinder part's X axis is its length; Y and Z are the diameter. Sizing
  the defense ring `(60, 0.2, 60)` and rotating it upright produced a 60-stud pillar rather
  than a flat disc. Correct is `(thickness, diameter, diameter)`.
- **Hardcoded yaw.** I hardcoded the four sector corridor rotations and got **all four wrong**.
  Derived correctly it is `yaw = atan2(dir.X, dir.Z)`, giving -45 / 45 / 135 / -135.
  Deriving beats guessing even when the geometry looks obvious.
- **Template lighting.** The place ships a custom `Sky`, `Atmosphere`, `Bloom`, `SunRays` and
  `DepthOfField`. They blurred and dimmed the exact sightlines a blockout exists to test, and
  the custom Sky drew a moon regardless of `ClockTime`. Stashed to
  `ServerStorage._StashedLighting` rather than deleted — they are a reasonable starting point
  for the art pass.

### Tracker

DH-010 to DH-013 and DH-073 **Done**. 8 of 74 tasks, 8.1%.

### Next

DH-014: species table and derivation formulas in `Shared.Config`. Then the first creature.

---

## 2026-09-28 (Phase 0) — Fusion installed, core utilities written, first build runs

### Decided

- **No Rojo, no Wally. MCP-only authoring.** Reverses the filesystem-source-of-truth decision
  taken earlier the same day. Code is written directly into the place via `set_script_source`
  and `execute_luau`.
  - **Accepted cost: the game code is not in git.** Only docs and the tracker are. Mitigated
    by DH-073, a periodic `export_rbxm` snapshot committed here.
- **Nevermore dropped.** It ships no `.rbxm` — its releases are per-package Wally tags across
  a ~400-package monorepo whose dependencies only Wally resolves. Uninstallable without a
  package manager. Replaced by three modules we wrote ourselves.
- **Fusion pinned at 0.3** (`v0.3-beta`, the scoped API). Resolves D-13.

### Done

- **Fusion 0.3 installed** from the official GitHub release `.rbxm` into
  `ReplicatedStorage.Fusion`. Verified in-place: `version = {major = 0, minor = 3,
  isRelease = true}`.
- **Built the structure in-place:** `ReplicatedStorage.Shared.{Config,Net,Types,Util}`,
  `ServerScriptService.Server.{Services,Bootstrap}`,
  `StarterPlayerScripts.Client.{Controllers,UI,Bootstrap}`.
- **Wrote the three Nevermore replacements** in `Shared.Util`:
  - `Maid` — deterministic cleanup with named slots; connections disconnected before
    Instances are destroyed, since a connection firing mid-cleanup can touch something
    already gone.
  - `Signal` — pure-Luau event with a reused-thread pool. Chosen over BindableEvent because
    that serialises arguments, so tables cross as copies and metatables are lost — no good
    for services handing each other creature handles.
  - `ServiceBag` — two-phase `Init`/`Start` lifecycle. The split is the point: `Init` claims
    dependencies, `Start` runs only after every service is initialised, so mutually dependent
    services never observe each other half-built.
- **Verified in a playtest, not assumed.** Server log `[Dino Hunt] Server started — 0
  service(s)`; client log `[Dino Hunt] Client HUD live — Fusion 0.3, label reads "Wave 1"`.
  The label reading "Wave 1" rather than "Pre-wave" proves the Computed followed the Value.
  Screenshot confirms the HUD renders. No errors.

### Tracker

DH-005, DH-006, DH-008 **Done**. DH-001, DH-002, DH-003, DH-007 **Cut** — void without
Rojo/Wally. Added DH-073 (export snapshots) and DH-074 (prove the hand-written utilities).
74 tasks, 39.90 days, Phase 0 at 33%.

### Next

Phase 1. DH-010: blockout the reactor core and defense ring.

---

## 2026-09-28 (later) — Studio bridge live, task tracker built

### Done

- **Connected the Roblox Studio MCP** (`@chrrxs/robloxstudio-mcp`). Plugin installed to
  `AppData/Local/Roblox/Plugins/MCPPlugin.rbxmx`. Bridge verified against the live place.
- **Confirmed the place** — Dino Hunt, placeId `96665470525866`, gameId `10768413738`. Saved to
  Roblox. Essentially blank: ServerScriptService, ReplicatedStorage and StarterGui all empty.
  This resolves **D-11**.
- **Built [Dino-Hunt-Tasks.xlsx](../Dino-Hunt-Tasks.xlsx)** — 72 tasks across the seven phases,
  with dependencies, estimates, priorities, acceptance criteria and FR references. Three sheets:
  Tasks, Phases (formula rollups), Legend. Verified through Excel: 0 formula errors.

### Estimate revised upward

Decomposing the plan into individual tasks produced **39.15 days against the 26–31 day
phase-level estimate** — about 30% more, roughly a day per phase. The bottom-up figure is the
one to trust; `06-project-plan.md` has been corrected rather than left contradicting the
tracker.

Eight working weeks, not six. The time is in the systems, not in trimmable extras — a
meaningful cut means dropping a phase, and Phase 5 (the collection loop) is the only one whose
removal still leaves something playable.

### Next

Phase 0. Task DH-001: install and pin the Rokit toolchain.

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
