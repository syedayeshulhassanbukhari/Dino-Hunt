# GDD review — Primal Rift MVP v1.1

Reviewed 2026-09-05 against `Primal_Rift_MVP_Game_Design_Document.docx`
(v1.1, 26 Aug 2026, 14 sections).

The spec is strong on architecture and player-experience guardrails, and weak on numbers —
the better of the two failure modes at this stage, since numbers are cheap to add and
architecture is not.

## Strengths

- **Server-authority model is genuinely well specified.** S11.1/11.3 name every service,
  every remote and its validation checks, and state explicitly: "No client reward remote."
- **The dual-gate unlock (boss kill AND Amber level) is a smart design** — it couples the
  combat loop to the exploration loop so neither can be skipped.
- **Anti-frustration rules are consistent**: safe recalls instead of death, boss-phase
  respawns, unfinished objectives carry forward, stuck-survivor teleport recovery,
  protected checkpoint rewards.
- **Scope discipline exists** — S12.4 out-of-scope list, 7-phase roadmap gated on a
  Wave 1-4 vertical slice.
- **Ethical monetization is a hard constraint** (S8.4, S9.6).

## Critical problems

### 1. The pacing budget does not add up

Target is 28-38 min (S7.2). Summing the spec's own numbers:

| Component | Min | Max |
|---|---|---|
| 15 normal waves @ 40-65s | 600s | 975s |
| 5 bosses @ 90-150s | 450s | 750s |
| 15 intermissions @ 15s | 225s | 225s |
| Exploration (4x150 + 180) | 780s | 780s |
| **Total** | **34.3 min** | **45.5 min** |

The floor already sits near the stated ceiling. Exploration alone is 13 minutes — 34% of a
best-case match. Fix by cutting exploration to 90-120s, dropping to 3 windows, or restating
the target as 34-46 min.

### 2. All three currencies have zero numbers

Amber per kill, DNA per kill, Run Credits per kill/assist/wave, and every shop price are
undefined — yet the whole progression spine depends on them. Amber thresholds are
250 / 600 / 1,050 / 1,600 cumulative; solo kill counts to those waves are roughly
64 / 130 / 200 / 250, implying ~4 to 6.5 Amber per kill on a rising curve.

**Note:** the existing place already solves this class of problem by derivation. See
[02-conversion-map.md](02-conversion-map.md), "Numbers already solved".

### 3. Amber income scales with players but thresholds are flat

Spawn budget is x2.25 at six players, so Amber income roughly doubles while the gauge
requirement stays at 1,600. Sectors unlock trivially in a full group and may be unreachable
solo. Amber requirements need a player-count term, or Amber-per-kill needs dividing by
party size.

### 4. Boss scaling is backwards for co-op

Boss HP is `x[1 + 0.50 x (P-1)]` = 3.5x at six players, but six players bring ~6x DPS.
Bosses die in about half the solo time at full party. Normals fare similarly: 1.75x HP with
2.25x budget = 3.9x effective pool against 6x DPS. Co-op is meaningfully *easier* than solo,
inverting the curve the wave table was tuned against.

### 5. "Active AI cap 45 on mobile, 60 on PC/console" is not implementable

AI count is a server-wide property under the stated server-authority model; one server hosts
mixed devices. Only rendering and effects can be device-tiered.

**Resolved in practice:** the existing place already does this correctly —
`min(perPlayer x livePlayers, 60)` with a per-wave table of per-player shares.

### 6. Solo downed-state rules contradict each other

S3.3 says the run ends when "all living players are downed at the same time", and also that
an expired downed timer respawns you next wave. For a solo player these produce opposite
outcomes from the same event.

### 7. Two conflicting weapon-gating systems

S8 gates guns by wave ("Unlock W14"); S9.4 gates them by Hunter Level. Is the W-column an
in-run purchase requirement or an account unlock? This changes the entire first-session
power curve.

## Balance issues before content production

- **Dual Energy Cannon is an outlier.** 22x2 @ 840 RPM = ~616 sustained DPS, against Energy
  Rifle 341 (the designated boss-DPS gun) and Assault Rifle 216. Heat is never quantified
  anywhere, so the late-game crowd gun is also the best boss gun — breaking the
  "every weapon has a job" pillar.
- **No player defense progression.** Health is flat 100 across all 20 waves while enemy
  damage scales x1.665. Late elites (18-30 base) hit for 30-50 — a 2-3 hit down, with only
  one perk (Field Medic) offering any HP.
- **Reactor HP is never stated.** Neither is dino damage-to-reactor, nor repair rate or cost.
  This is the primary fail condition and has no numbers at all.
- **Diminishing returns are defined for stuns only** — not freeze or slow, which the Frost
  Blaster and Shock Fence both apply.

## Undefined states engineering will hit immediately

- **Late gauge fill.** If the Amber requirement completes mid-Wave 6, does an exploration
  window open then (pausing waves), or queue to the next boss?
- **Egg with no hatchery.** Boss 1 awards an egg, but the hatchery needs Boss 1 *and*
  Reactor L2. If L2 is not reached, the egg is uninsurable against run failure.
- **Parallel incubation.** Up to 5 eggs per run at 120-180s each — one hatchery slot or many?
  This directly determines the ">=35% first egg hatch" KPI.
- **DNA carry mechanics.** Oviraptor "steals deposited field samples" and S3.5 has you
  depositing DNA at intermission, so DNA is carried and losable — but nothing defines how it
  is carried, how much is at risk, or how theft resolves. Amber's rules are explicit; DNA's
  are not.
- **Per-player or per-team?** Barricade max 6 and Ammo Crate max 1 are unscoped. Melee
  stamina has no values.
- **Hitscan lag compensation is missing.** Rewind is specified for melee only. 19 of 20 guns
  are hitscan against fast-flanking raptors on mobile ping.

## Smaller notes

- S1.2's document map stops at section 12; the document has 14 sections.
- Nest guardian species are only partly assigned (Sector 2 Baryonyx implied; 1/3/4 vague).
- No Robux product list, price points or VIP contents — no revenue model to validate.
