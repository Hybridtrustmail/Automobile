---
doc: Data/Vehicle/docs/vehicle_prototypes_drivetrain_plan.md
type: plan
status: active
authority: informative
component: data
summary: Drivetrain data plan for vehicle prototypes.
read-first: []
roadmap: []
---

# Vehicle prototypes — drivetrain topology and electric heavy-duty phase

Date: 2026-08-03
Status: approved, phase A not yet implemented
Builds on: `vehicle_prototypes_design.md` (schema, tiers, validation gates, range
and generation conventions — not repeated here).

**Goal:** make the electric power path legible — how many motors, on which axle,
at what power, through which reduction — and fill the empty electric heavy-duty
region of the dataset (buses, trolleybuses, lorries).

## Why

Two gaps found by a coverage review of the 313-row dataset.

**1. Electrification is car-only.** 40 BEV rows: 38 cars, one pickup, one
motorcycle. 11 PHEVs, all cars. Zero electric buses, zero trolleybuses, zero
electric lorries — while `vehicle_mass_80yr_dataset.csv` models all three.

**2. No drivetrain topology.** 0 of 67 electrified rows carry `gearbox_ratios`;
13 carry `final_drive_ratio`. The extension phase set out to close the PHEV/EV
reduction-gear gap and did not: the one row that solved it, the Outlander PHEV,
holds its front/rear reductions (9.663 / 7.065) in `notes` prose because the
schema has nowhere to put them.

### The EV_sim ratios are not axle ratios

`EV_sim_dataset.csv` is a **lumped single-motor model**. Each vehicle has exactly
one `motor` block and one `drive train / gear_ratio`, including
`Tesla_2022_ModelS_PlaidTriMotorAWD` — a three-motor car reduced to one
equivalent motor and one ratio.

So the 12 `ev_sim` values currently sitting in `final_drive_ratio` (Tesla 9.0-9.1,
Audi 9.205, Taycan 8.05) are **simulation equivalents, not hardware**. Moving them
into a front or rear reduction column would assert something the source never
stated. They get their own column instead.

#### Confirmed during Phase B: the lumped value is one axle's value

This began as an inference from EV_sim's file structure. Phase B research
confirmed it three times independently, and the mechanism turns out to be
specific: **EV_sim adopts one axle's figure as the whole-vehicle equivalent**,
rather than averaging or summing.

| Vehicle | EV_sim lumped value | What it actually equals |
| --- | --- | --- |
| Tesla Model X Long Range | gear_ratio 9.04 | the car's **rear** gearset (front is 7.56) |
| Tesla Model X Plaid | gear_ratio 7.56 | that car's ratio (7.56 on both axles) |
| Audi e-tron 55 quattro | motor_power_kw 140.0 | the **rear** motor's continuous rating exactly (front is 125) |

FASTSim behaves the same way: on the Toyota Highlander Hybrid its
`motor_power_kw` of 123 kW is exactly the researched **front** motor, with the
50 kW rear motor absent from the model entirely.

Two consequences, both already implemented:

- A lumped figure must never be read as a system total or an axle ratio. It is
  one component's value wearing a whole-vehicle label.
- The per-axle power validation gate skips `fastsim` and `ev_sim` rows. Without
  that skip it blanked researched hardware on four rows — including the Model S
  Plaid, where 250+500=750 kW was deleted for failing to match a lumped 205 kW.
  Deleting the best-sourced row in the set to defend a modelling artefact is the
  exact failure this rule now prevents.

## Schema additions

| Column | Meaning |
| --- | --- |
| `motor_count` | total traction motors (1, 2, 3, 4) |
| `motor_layout` | readable topology token: `1-rear`, `1-front+1-rear`, `1-front+2-rear`, `2-front+2-rear` |
| `motor_power_front_kw` | combined motor power on the front axle |
| `motor_power_rear_kw` | combined motor power on the rear axle |
| `motor_reduction_front` | front-axle reduction; semicolon list if multi-speed |
| `motor_reduction_rear` | rear-axle reduction; semicolon list if multi-speed |
| `drivetrain_ratio_lumped` | single equivalent ratio from a lumped simulation model |

Existing columns keep narrowed meanings:

- `final_drive_ratio` — an **engine's** final drive only. Not a motor reduction.
- `motor_power_kw` — system total. The front/rear pair must sum to it where all
  three are known.
- `gearbox_ratios` — multi-speed **gearbox** ratios, not motor reductions.

Rules:

- Fill `motor_reduction_*` only from a source stating a per-axle figure.
- A single-motor vehicle fills only its own axle's pair and sets `motor_count=1`.
- Where a source gives one ratio for a multi-motor car without saying which axle,
  put it in `drivetrain_ratio_lumped`, not in an axle column.
- Multi-speed reductions use the existing semicolon convention (`15.56;8.05`).

### Motor counts above two, and the per-axle limit

`motor_count` is a plain integer, so 1, 2, 3 and 4 motors are all representable.
Three- and four-motor cars are already in scope: the Ferrari SF90 Stradale is
stored as `2-front+1-rear`, and the Tesla Model S Plaid is a tri-motor.

The power and reduction columns are **per axle, not per motor**. Where an axle
carries two motors, `motor_power_front_kw` (or `_rear_kw`) is their combined
power, and the axle's reduction column holds the ratio they share.

Known limitation, accepted deliberately: a vehicle with genuinely independent
per-wheel gearing — two motors on one axle driving through *different* ratios,
as on the Rimac Nevera — cannot express that here. If such a vehicle is added,
record the axle-level figures, put the per-wheel detail in `notes`, and do not
invent an axle ratio that neither wheel actually uses. Should several such rows
accumulate, the fix is a long-format drivetrain sidecar keyed on
(vehicle, axle, motor index), not more columns on the main table.

## Identity: how a sidecar attaches

`prototype_id` is assigned in `main()` as a per-kind counter **after** merge, so
it is positional and shifts if earlier loaders change. It must not be used as a
join key.

The stable identity is the dedup key already used in `main()`:

```python
(make.lower(), model.lower(), year, variant_trim.lower())
```

Researched topology for rows that do not live in `manual_vehicles.csv` (the
`fastsim`, `enrichment` and `ev_sim` tiers) therefore attaches through a
sidecar keyed on those four fields, applied after merge and before `derive()`.

**Superseded by the drivetrain-configuration migration:** that sidecar shipped
as `motor_topology.csv` in Phase A and was later renamed `drivetrain_configs.csv`
and widened to carry a `drivetrain_config` discriminator, `is_primary`, axle
geometry, gearing and per-configuration masses — see
`vehicle_prototypes_design.md`'s "Drivetrain configurations" section for the
current contract (boundary rule, `is_primary` semantics) and
`vehicle_prototypes_drivetrain_schema_review.md` for the reasoning. Everything
below in this Phase A section describes the mechanism as first built; the
column set and file name have moved on but the fold-on-`is_primary` behaviour
is unchanged.

## Phase A — schema, migration, override mechanism

No web research. Deterministic only.

1. Add the seven columns to `COLUMNS`, grouped after `motor_power_kw`.
2. Add the same columns to `manual_vehicles.csv`'s header.
3. `load_ev_sim()`: write its `gear_ratio` to `drivetrain_ratio_lumped` instead of
   `final_drive_ratio`, and note in each row that it is a lumped equivalent.
4. Outlander PHEV row in `manual_vehicles.csv`: set `motor_count=2`,
   `motor_layout=1-front+1-rear`, `motor_power_front_kw=60`,
   `motor_power_rear_kw=60`, `motor_reduction_front=9.663`,
   `motor_reduction_rear=7.065`. Keep `final_drive_ratio=3.425` (the engine's).
   Trim the now-redundant ratio sentence from `notes`, keeping the citation.
5. New `motor_topology.csv` + `load_topology_overrides()` applying it on the
   dedup key. Ship it with a header and zero data rows.
6. Validation gate: where `motor_count`, `motor_power_front_kw`,
   `motor_power_rear_kw` and `motor_power_kw` are all present, front+rear must
   equal the total within 1 kW, else blank the pair and log it.

Verify: 313 rows; `final_drive_ratio` filled count drops by 12 (the ev_sim rows
move to `drivetrain_ratio_lumped`); Outlander shows a complete power path;
provenance blanked count unchanged at 16.

## Phase B — research topology for the electrified rows

Target: the 40 BEV + 25 Hybrid + 2 FCEV rows. Populate `motor_count`,
`motor_layout`, per-axle power and per-axle reduction where a source states them.

Rows in `manual_vehicles.csv` are edited in place. Rows in other tiers get a line
in `motor_topology.csv`.

Reduction ratios are often unpublished; `motor_count` and `motor_layout` are
usually findable even when ratios are not. Partial rows are expected and correct
— never infer a ratio from a motor count.

## Phase B result — what is actually findable

Phase B researched 65 electrified rows. The yield was very uneven by field, and
the pattern is stable enough to plan against.

**Reliably findable**

- `motor_count` and `motor_layout` — almost always confirmable, even where power
  figures were murky. 65 of 65 rows carry both.
- Single-axle power on single-motor cars.
- Per-axle power on dual-motor cars **when the source labels "Electric motor
  #1/#2" with a Location field**. Tesla, Porsche, Audi, Volvo and Polestar pages
  all do this; it is far more trustworthy than a search-synthesised combined
  figure.

**Unreliable or absent**

- **Reduction ratios are the hardest field by a wide margin** — 15 of 65 rows.
  Most makers never publish them (Mazda, Honda, Renault, Mini, BYD, Toyota
  hybrids, VinFast: all blank). Where a number did surface it was usually a
  single undifferentiated "axle ratio" with no front/rear attribution.
- **Per-axle power on unlabelled dual-motor systems** — the Ford F-150 Lightning
  and BMW iX publish only combined system totals despite being dual-motor, so
  their power columns stay blank.
- **Power-split and mild-hybrid architectures** do not map cleanly onto a
  front/rear schema. The Chevrolet Volt needed a `2-front` token invented for it
  (two motor-generators on one axle, no rear motor), and deciding which figure is
  "the" traction rating takes a judgement call each time.

**The attribution bar earned its place.** Filling a ratio column only when the
source ties the ratio to a specific axle is what kept the Tesla Model 3 and
Model Y rows honest — both publish an "axle ratio ≈9" that says nothing about
which axle. It cleared only for the Tesla S/X pages and the German makers.

**Rows carrying a caveat that must be read before use**

- Porsche Taycan 4S and 4S Plus: rear first gear is `16`, but sources give only
  "roughly 16:1" / "~15 revolutions" — a 15-16 spread, not a stated value.
- Porsche Taycan 4S Plus: both ratios are carried from the base 4S on
  architecture-level corroboration, not a trim-specific fetch.
- Tesla Model 3 LR and Model Y LR: unconfirmed higher-output "EU spec"
  alternatives exist (158/208 and 158/220). Both rows use the non-EU figures, so
  they are at least mutually consistent.
- Toyota Corolla Cross Hybrid: FWD rests on one variant page; whether a 2022
  E-Four AWD existed is unresolved.

## Bus and coach useful load — the regulated basis

Buses are the largest hole in `q` coverage: 20 of 21 bus rows have a curb mass
and none has a payload, so none has a `q`. Unlike a truck, a bus has no
published payload figure — it must be derived from passenger capacity.

The derivation differs by bus class, and this is regulated rather than a matter
of judgement. A city bus is capacity-limited by standing area with negligible
luggage; a coach is seated-only with substantial underfloor baggage. One blanket
figure would be wrong for both.

Source: **Commission Regulation (EU) No 1230/2012, Annex I Part B**. Full quotes,
URLs and the fetch audit trail are in
`vehicle_prototypes_bus_payload_standard.md`.

| Class | Per-passenger mass `Q` | Standing space `Ssp` | Standing |
| --- | --- | --- | --- |
| I / A — urban | **68 kg** | 0.125 m² | yes |
| II — interurban | **71 kg** | 0.15 m² | limited |
| III / B — coach | **71 kg** | not applicable | no |

- Driver: **75 kg**. Crew member: **75 kg**.
- Luggage: **B = 100 × V**, i.e. 100 kg per m³ of baggage-compartment volume.

### The rule for this dataset — masses first, capacity only as fallback

**Preferred, and used wherever possible:**

```
payload_kg = gross_mass_kg − curb_mass_kg
```

Take it from the model's own published masses. This is the same derivation the
rest of the dataset already uses for trucks and cars, it introduces no
assumption about occupancy, and it needs no regional `Q` at all.

It also covers far more of the fleet than expected: **19 of the 21 bus rows
already carry both masses**, so their payload and `q` follow directly with
nothing assumed. Computed this way they give `q` between 1.53 and 3.38 — the
NIIAT ГАЗ minibuses at 2.61 and the ПАЗ-3205 city buses at 1.53-1.94.

**Fallback, only when a mass is genuinely unavailable:**

```
payload_kg = (seated + standing) × Q(zone, class)   +   100 × V_baggage_m3
```

Use this only for rows where `gross_mass_kg` or `curb_mass_kg` cannot be
sourced — currently just the two EU rows: Solaris Urbino 12 electric (gross
20 000 kg, no published curb) and Škoda 32Tr SOR (curb 10 180 kg, no published
gross). Prefer sourcing the missing mass over falling back.

A payload derived by the fallback must say so in `notes`, naming the zone, class
and capacity figures used, because it rests on assumptions the mass subtraction
does not.

- `Q` is chosen by the row's bus class, not by a single global constant.
- Standing capacity is only counted for Class I and II. For Class III it is zero
  by definition, not merely unobserved.
- The luggage term applies only where a source states a baggage-compartment
  volume. Where none is stated, omit the term — do not assume one.
- The **driver is not payload.** 75 kg of driver belongs to running-order mass,
  which is consistent with this project's existing convention that
  `M₀ = EU kerb mass − 75 kg driver`.
- Record the class in `notes` alongside the capacity figures used, so the
  arithmetic is reproducible from the row.

### `Q` is regional, not global

The 68/71 kg figures are **EU figures**, binding where Regulation (EU) 1230/2012
and UNECE R107 apply. They are not a universal constant, and applying them to a
vehicle homologated elsewhere would be as wrong as ignoring them inside the EU.

`Q` is therefore selected by the row's `market`, not by one dataset-wide value:

| Zone | `Q` | Basis |
| --- | --- | --- |
| EU / UNECE | 68 kg Class I/A; 71 kg Class II/III/B | Reg (EU) 1230/2012 Annex I Part B — fetched and quoted in `vehicle_prototypes_bus_payload_standard.md` |
| Post-Soviet / GOST (`RU/CIS`) | **75 kg** | project convention, `weight.md:55`; **GOST citation still to be added** |
| Other markets | not yet established | research before use; do not default to either figure |

So `weight.md`'s 75 kg is not an error — it is the post-Soviet figure, and the
modelled dataset's families are largely of that origin. The earlier framing of
this as a discrepancy to reconcile was wrong: there is nothing to reconcile
between two zones that genuinely specify different masses.

What does still matter:

- Every derived bus payload must record **which zone's `Q` was used** and why, in
  `notes`. A `q` computed at 68 kg and one at 75 kg are not comparable, and
  nothing else in the row reveals which was applied.
- The 19 NIIAT buses are `RU/CIS` → 75 kg. The Solaris and Škoda are EU → 68 kg
  (both Class I, urban, standing permitted).
- The GOST figure currently rests on project convention rather than a fetched
  standard. That citation is a gap worth closing, and until it is, the 75 kg
  rows should say so rather than implying a regulatory source.
- Coincidentally, 75 kg is also the EU figure for **driver and crew**. Do not let
  that coincidence blur the two: the driver is not a passenger and is not
  payload, in either zone.

### Scope note

Regulation 1230/2012 never defines a bus "payload" — the term "pay-mass" is
N-category only. For M2/M3 it defines the mass components (passengers, crew,
driver, luggage) that must fit under the technically permissible maximum laden
mass. Any bus payload in this dataset is therefore **derived, not regulated**,
and must be labelled as such in provenance: `method = derived`, never `cited`.

## Phase C — electric heavy duty

Add ~15-20 rows to `manual_vehicles.csv`, entered with Phase A's topology
columns populated from the start.

- **Electric buses** — BYD K9, Yutong E12, Solaris Urbino 12 electric,
  Mercedes eCitaro, VDL Citea SLF-120 Electric
- **Trolleybuses** — Bogdan T701, Skoda 32Tr, Trolza-5265 Megapolis
  (`powertrain=Trolley-electric` per the base design's powertrain list)
- **Electric lorries** — Mercedes eActros, Volvo FE/FL Electric, Freightliner
  eCascadia, MAN eTGM, Renault D Z.E.

Same citation rules as all web-researched rows: fetched `source_url`, a `quote`
containing the figures, blanks rather than estimates.

## Non-goals

- Motor efficiency maps, inverter or battery-pack internals — out of scope; the
  FASTSim and EV_sim files already hold that for anyone who needs it.
- Backfilling topology onto ICE rows.
- Changing `prototype_id` to a stable scheme. Recorded as a known limitation:
  ids are positional and must not be used as an external key.
