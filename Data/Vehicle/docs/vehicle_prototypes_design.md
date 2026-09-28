---
doc: Data/Vehicle/docs/vehicle_prototypes_design.md
type: architecture
status: active
authority: informative
component: data
summary: Vehicle prototypes dataset design.
read-first: []
roadmap: []
---

# Vehicle prototype dataset — design

Date: 2026-07-31 (last updated 2026-08-05)
Status: implemented — 500 rows / 89 columns / 553 provenance rows

## Purpose

Produce a real-vehicle prototype dataset that can back the traction calculation
(тяговий розрахунок) in `Курсовой проект+OCR.md`, with an explicit powertrain type
so normal / hybrid / electric vehicles can be compared as separate populations.

The coursework needs, per vehicle: own mass `M₀`, gross mass `Mₐ`, payload `Mв`,
tare coefficient `q`, max speed `Vmax`, road resistance `ψ`, aerodynamic drag,
rolling resistance, wheel radius `rк`, final drive `i₀`, gearbox ratios `iкп`,
and transmission efficiency `ηтр`.

## Directory layout

```
Data/Vehicle/
  build/    build_vehicle_prototypes.py, bulk_sources.py, parse_niiat.py, unit tests
  sources/  read-only inputs: prototype_enrichment.csv, niiat_trucks.csv,
            manual_vehicles.csv, drivetrain_configs.csv
  out/      vehicle_prototypes_provenance.csv (build output)
  docs/     this file and the rest of the design/plan history
  vehicle_prototypes.csv   build output, top level (the deliverable)
  archive.zip              superseded prototype_index.csv + enrichment_provenance.csv
```

Run with `python3 build/build_vehicle_prototypes.py` from `Data/Vehicle/`; the
script resolves `sources/`, `out/` and the FASTSim path via `__file__`, not the
working directory, so it also runs correctly from inside `build/`.

## Problem with the current data

`prototype_enrichment.csv` — 95 real vehicles, 54 columns.

- No powertrain type column. `fuel_type` reads `gasoline` for a BMW 530e; the hybrid
  fact sits in free-text `engine_type` (`plugin hybrid`, `hybrid`, `electric motor`).
- `battery_capacity_kwh`, `range_km`, `charging_power_kw`, `gradeability_pct`,
  `max_towing_kg` — 0% filled.
- Weak fill on calculation inputs: `curb_weight_kg` 66%, `gvw_kg` 27%,
  `payload_capacity_kg` 32%, `year` 64%.
- Absent entirely: drag coefficient, frontal area, rolling resistance, wheel radius,
  gear ratios, transmission efficiency.

`prototype_index.csv` — 9354 rows, but 9259 are `family_envelope` synthetic class
estimates. Only the same 95 are real vehicles. It does carry a clean powertrain
vocabulary (ICE / Hybrid / BEV / FCEV / ICE-LPG / ICE-CNG / ICE-LNG /
Trolley-electric), but applies it only to the synthetic rows.

## Sources

### Tier A — FASTSim (local, authoritative, free)

`/media/cigameca/SHARED/Work/Vehicle_simulation/fastsim-fastsim-3/cal_and_val/f2-vehicles/`
67 NREL FASTSim vehicle definitions, YAML. Every one of the following is 67/67 filled:

| FASTSim field | Coursework symbol |
|---|---|
| `veh_pt_type` (Conv 33 / BEV 20 / HEV 9 / PHEV 5) | powertrain type |
| `drag_coef` | `Cx` |
| `frontal_area_m2` | `F` |
| `wheel_rr_coef` | `f` |
| `wheel_radius_m` | `rк` |
| `trans_eff` | `ηтр` |
| `glider_kg`, `cargo_kg` | mass build-up |
| `wheel_base_m`, `veh_cg_m`, `drive_axle_weight_frac` | layout / axle loads |

Includes commercial vehicles: `Class 4 Truck (Isuzu NPR HD)`,
`Regional Delivery Class 8 Truck`, `Line Haul Conv`, `Toyota Hilux Double Cab 4WD`,
`Nissan Navara`, `2020 Chevrolet Colorado 2WD Diesel`, and a BEV pickup
`2022 Ford F-150 Lightning 4WD`.

Does **not** contain: gear ratios (transmission is a single efficiency scalar),
tyre codes (`tire_code` exists in the f3 format but is null for all 67), buses.

Known defect to correct on import: `Toyota Mirai` and `2016 Hyundai Tucson Fuel Cell`
are tagged `HEV`; both are FCEV.

### Tier B — existing enrichment

The 95 rows of `prototype_enrichment.csv`. Contributes brand/model identity, EU market
specs, price, CO₂, top speed. Overlaps Tier A on exactly one vehicle (Tesla Model Y).

### Tier C — web research

Buses (zero coverage in A and B), plus tyre codes and gearing. Every value cited with
source URL and supporting quote.

## Row composition

The dataset grew past its original ~181-row target with two later additions:
NIIAT trucks (with dual-final-drive sidecar lines) and a wholesale import of
180 Kaggle rows (see "Bulk sources" below). Current `data_tier` breakdown:

| `data_tier` | Rows |
|---|---:|
| `enrichment` (original `prototype_enrichment.csv`) | 95 |
| `fastsim` | 67 |
| `ev_sim` | 12 |
| `niiat` | 107 |
| `web-cited` (buses, gearing research) | 38 |
| `brochure` | 1 |
| `kaggle-spec` (Task 8 wholesale import) | 180 |
| **total** | **500** |

Motorcycles: the 28 existing rows are carried through unchanged, not extended.

## Schema

Groups, one column per field unless noted.

**Identity** — `prototype_id`, `make`, `model`, `variant_trim`, `generation`,
`year`, `market`

`prototype_id` is a stable id, not a positional or kind-prefixed one:
`"vp-" + sha256("|".join([make, model, year, variant_trim]).lower())[:6]`
(see `stable_id()` in `build_vehicle_prototypes.py`). It is deterministic
across builds — two consecutive builds produce identical ids — because it is
derived from the row's own identity fields rather than row order or a salted
hash. The prefix is always literally `vp-`; it carries no information about
`vehicle_kind`, which is recorded exactly once, in the `vehicle_kind` column
itself. Do not read the id to infer whether a row is a car, truck, bus or
motorcycle.

Collision rule: if two rows hash to the same 6-hex id, the second is widened
to 8 hex digits and a `NOTE: id collision` line is printed. This separates
two *different* keys that happen to collide at 6 hex. It does **not** help
two rows that share the exact same `(make, model, year, variant_trim)` key —
identical keys hash identically at any width, so widening can never split a
true duplicate; only deduplication at load time can (see the Task 8 duplicate
fix below, under "Known findings").

**Classification** — `vehicle_kind`, `regulatory_category` (M1/M2/M3/N1/N2/N3/L,
plus preserved specific values like M1G, L3e), `segment`, `body_type`,
`axle_configuration`, `drive_layout`, `drivetrain_config`

`regulatory_category` is **derived**, in `derive()`, from `vehicle_kind` and
`gross_mass_kg` (`eu_category()`: car → M1, motorcycle → L, truck → N1/N2/N3
by mass, bus → M2/M3 by mass), but only into a row that does not already carry
a category from its source. A pre-existing, more specific value — `M1G`,
`L3e`, or a truck's `"N2, N3"` combination class — is never overwritten by
the generic derivation. It stays blank only where the row's `vehicle_kind` is
mass-dependent (truck/bus) and `gross_mass_kg` itself is blank; there is
nothing to derive from.

`axle_configuration` is a strict wheel-end formula `AxB` (`4x2`, `4x4`, `6x4`,
`6x6`, `8x4`, `8x8`, …), plus `2x1` for a chain/shaft-drive motorcycle (2
wheels, 1 driven). `drive_layout` is one of `FWD`/`RWD`/`AWD`/`4WD`. Both value
spaces are enforced by a build-time gate that blanks anything else, logged
under `method = blanked_implausible`.

`axle_count` and `driven_axle_count` are **derived, never stored** — computed
in `derive()` from `axle_configuration` (`AxB` → `A/2` axles, `B/2` driven; for
cars with no axle formula but a known `drive_layout`, `axle_count=2` and driven
count follows from FWD/RWD/AWD/4WD). Do not add a loader that writes either
column directly.

`drivetrain_config` names the drivetrain configuration the row's own masses
and dimensions were sourced for; see "Drivetrain configurations" below.

**Powertrain** — `powertrain` (ICE / Hybrid / BEV / FCEV / ICE-LPG / ICE-CNG /
ICE-LNG / Trolley-electric), `powertrain_detail` (MHEV / HEV / PHEV / REx / BEV /
FCEV), `energy_carrier`, `engine_displacement_cc`, `power_kw`, `torque_nm`,
`battery_capacity_kwh`, `motor_power_kw`, `motor_count`, `motor_layout`,
`motor_power_front_kw`, `motor_power_rear_kw`, `motor_reduction_front`,
`motor_reduction_rear`, `drivetrain_ratio_lumped`

The `motor_*` and `drivetrain_ratio_lumped` fields are the topology fields
folded in from `drivetrain_configs.csv`'s `is_primary` sidecar line; see
"Drivetrain configurations" below.

**Mass** — `curb_mass_kg` (`M₀`), `gross_mass_kg` (`Mₐ`), `payload_kg` (`Mв`),
`glider_kg`, `cargo_kg`, `seats`, `max_towing_kg`

**Geometry** — `length_mm`, `width_mm`, `height_mm`, `wheelbase_mm`,
`track_width_mm`, `cg_height_m`, `drive_axle_weight_frac`

**Range pairs** — five fields additionally carry `<field>_min` and `<field>_max`:
`curb_mass_kg`, `gross_mass_kg`, `length_mm`, `width_mm`, `height_mm`.
See "Range-valued fields" below.

**Wheels and gearing** — `tyre_code` (e.g. `205/55 R16`),
`wheel_radius_static_m`, `wheel_radius_dynamic_m`, `num_wheels`,
`final_drive_ratio` (`i₀`), `gearbox_ratios` (semicolon list `i₁;i₂;…;iₙ`),
`gear_count`, `transfer_ratios`, `transmission_type`, `transmission_eff` (`ηтр`)

**Resistance** — `drag_coef` (`Cx`), `frontal_area_m2` (`F`),
`rolling_resistance_coef` (`f`)

**Performance** — `top_speed_kmh` (`Vmax`), `acceleration_0_100_s`,
`gradeability_pct`, `co2_emissions_gkm`, `range_km`

**Consumption** — `fuel_city_l_100km`, `fuel_highway_l_100km`,
`fuel_combined_l_100km`, `energy_city_kwh_100km`, `energy_highway_kwh_100km`,
`energy_combined_kwh_100km`, `electric_range_km`, `recharge_time_h`.
See "Consumption columns" below.

**Derived** (computed, flagged as such) — `cda_m2` = `drag_coef` × `frontal_area_m2`
(фактор обтічності), `payload_kg` = `gross_mass_kg` − `curb_mass_kg` where not
directly sourced (never overwrites a sourced `payload_kg`), `tare_ratio_q` =
`curb_mass_kg` / `payload_kg` (feeds Рис. 1 of the coursework),
`specific_power_kw_per_t`, `axle_count`, `driven_axle_count`

**Provenance** — `data_tier` per row, `source_file`, `source_url`, `last_updated`,
`notes`

## Consumption columns

There are two parallel triplets, one per energy carrier, each in the same
per-100-km convention:

- Fuel: `fuel_city_l_100km`, `fuel_highway_l_100km`, `fuel_combined_l_100km`
- Electric: `energy_city_kwh_100km`, `energy_highway_kwh_100km`,
  `energy_combined_kwh_100km`

`kWh/100 km` is used rather than `kWh/km` for two reasons: it is the unit
NRCan's own bulk-source columns are published in (`City (kWh/100 km)`, etc.,
so no conversion or rounding is introduced on ingest), and it keeps electric
consumption on the same scale and city/highway/combined structure as the fuel
triplet, so the two can be read side by side rather than requiring a mental
unit conversion.

A plug-in hybrid legitimately fills **both** triplets — it burns fuel in
charge-sustaining mode and draws from the grid in charge-depleting mode, and
both figures are real, independently measured numbers for the same vehicle.
Do not treat a populated electric triplet as evidence a row's fuel triplet is
wrong, or vice versa.

`electric_range_km` and `recharge_time_h` are BEV/PHEV-specific fields
alongside the triplets, filled by NRCan where available.

## Range-valued fields

Sources frequently give a mass or dimension as a range rather than one number —
because the figure varies by trim, drivetrain or market. Collapsing such a range
to its midpoint destroys information and asserts a precision the source never
stated, so the range is kept as data.

`RANGED_FIELDS` in `build_vehicle_prototypes.py` lists the five fields this
applies to: `curb_mass_kg`, `gross_mass_kg`, `length_mm`, `width_mm`,
`height_mm`.

- When the source states a range, put the endpoints in `<field>_min` and
  `<field>_max` and leave the base field blank in the input CSV. The build
  computes the base field as the mean of the two.
- When the source states one number, put it in the base field and leave the pair
  blank.
- If nothing is sourced, all three stay blank. Never write a base value with no
  source behind it.

The base field therefore always holds a usable number, so consumers that read
`curb_mass_kg` need no change, while the endpoints remain available for anything
that must respect the true spread. Because the mean is recomputed on every build
it cannot drift away from its endpoints.

A range must be a genuine spread of the same quantity. Two things that look like
ranges in source text but are not: rpm bands (`max torque 700 Nm at 1600-2600 rpm`
is a speed band, not a torque range) and OCR line references in `notes`.

Where a source gives a range so wide that no single figure is meaningful, record
the range and let the mean stand for it — this is preferable to the earlier
practice of discarding the value entirely. The Toyota Corolla E30 (785-955 kg)
and Honda Civic 1st generation (600-790 kg) curb masses are in the dataset for
exactly this reason.

## Drivetrain configurations

A model that ships with more than one drivetrain (final drive, motor count,
motor power) is handled by a boundary rule, applied mechanically:

> A configuration gets its own **row** if and only if the source states a
> different value for `curb_mass_kg`, `gross_mass_kg`, `payload_kg`, or any of
> `length_mm` / `width_mm` / `height_mm` / `wheelbase_mm`.
> Otherwise it gets a **line in the sidecar**, `drivetrain_configs.csv`.
> It is a different **vehicle** (not a variant) if the nameplate, the
> `generation` code, or the `body_type` differs — regardless of mass.

`drivetrain_configs.csv` is keyed on the same dedup key `main()` uses to merge
rows — `(make, model, year, variant_trim)`, case-folded — plus a
`drivetrain_config` discriminator token (e.g. `cetrax-220`, `fd-6.7`). Its
contract:

- Exactly one line per key may be `is_primary=TRUE`; a build-time gate warns
  on more than one.
- The `is_primary` line's topology fields fold into the matching main row
  before `derive()` runs, exactly as `motor_topology.csv` did before it was
  widened. Non-primary lines are published in the sidecar but never folded.
- **No line at all may be `is_primary`** when the mass source names no
  configuration — folding one in that case would assert a default the source
  never stated. A row with no primary sidecar line simply keeps whatever
  drivetrain fields it already carries.
- Mass and dimension fields on a sidecar line describe that configuration only
  when a source publishes them per-configuration; otherwise leave them blank
  rather than copying the row's own mass, which the boundary rule above has
  already established does not vary by configuration.

See `vehicle_prototypes_drivetrain_schema_review.md` for the full reasoning
and worked examples (the Solaris Urbino 12 electric, the NIIAT dual-final-drive
trucks).

## Generation notation

`generation` identifies which generation of a nameplate a row describes. It
matters because the dataset holds several rows per nameplate: five Volkswagen
Golf rows span 1974-2020, and `year` alone does not separate a facelift from a
new platform.

Two notations are permitted, in this order of preference:

1. **The manufacturer's own generation or chassis code**, where a source states
   it — `E30`, `Mk8`, `T1XX`, `J300`, `B310`, `XA50`. This is the canonical
   identifier and differs legitimately by marque: Volkswagen numbers marks,
   Toyota uses factory codes, Dacia uses roman numerals.
2. **`Nth generation`**, only where no manufacturer code could be sourced.
   Always numeric-ordinal (`14th generation`), never spelled out.

Some marques code at generation level (`T1XX`, `J300`, `P702`, `E30`) and some at
variant level within a generation (Honda `FE2` is the 11th-gen Civic's 2.0 NA
sedan; `RS3` a 6th-gen CR-V variant; `CY1` an 11th-gen Accord variant). Both are
acceptable — a variant code still identifies its generation unambiguously, which
is what the column is for. Prefer the most specific code the source actually
supports, and if a source names a code pair without saying which member applies
(`RS3/RS4`), say so in `notes` rather than picking one silently.

`notes` must not claim more than the fetched page states. Where a code's
generation is cited but its sub-variant is reasoned rather than quoted, mark that
part `INFERRED:` — the CR-V and Accord rows carry this.

Never infer a generation from a model year, and never write a chassis code from
memory — a code attached to the wrong generation is worse than an empty cell.
Leave the field blank when the source is silent; the `fastsim` and `enrichment`
tier rows are mostly blank for this reason, which is correct.

## Field provenance

Each field group carries a tier marker so a reader can tell modelled from measured:

- `fastsim` — imported from the NREL FASTSim vehicle definitions
- `enrichment` — carried from the original `prototype_enrichment.csv`
- `web-cited` — researched, with a row in the provenance sidecar giving URL and quote
- `nrcan` / `kaggle-spec` — bulk-sourced; see "Bulk sources" below
- `derived` — computed from other columns in the same row

`vehicle_prototypes_provenance.csv` follows the existing `enrichment_provenance.csv`
shape: `prototype_id, field, value, unit, source_url, quote, method, fetched_at`.

## Bulk sources

Two whole-dataset sources supplement the curated rows, loaded and matched in
`bulk_sources.py`. Each is one citation covering every value it contributes —
individual values from these sources are never `cited` (see "Provenance
methods" below).

- **`nrcan`** — Natural Resources Canada fuel/energy consumption ratings
  (<https://open.canada.ca/data/en/dataset/98f1a129-f628-4ce4-b24d-6f16bf24dd64>).
  Six files spanning 1995-2026, read in an explicit priority order
  (`NRCAN_FILE_ORDER`) rather than alphabetical `glob()` order, because a
  plain sort lets the older 2-cycle test method's ratings overwrite the
  better 5-cycle ones for the overlapping 1995-2014 window. Matched on
  normalised `(make, model, year)`. Fills the consumption triplets, CO2,
  electric range and recharge time. Match rate is low — 37/500 in the current
  build — because NRCan is Canadian-market data against a fleet that is
  largely European and Soviet-era; see "Known findings" below.
- **`kaggle-spec`** — "Car Specification Dataset 1945-2020"
  (<https://doi.org/10.34740/kaggle/ds/2938279>). Used two ways: (1) enrichment
  — matched on normalised `(make, model)` plus a year-range check, filling
  mass, seats, torque, gearing count, top speed, acceleration, wheelbase,
  body type and the fuel triplet where blank; trim only disambiguates
  multiple candidates, it is never fuzzy-matched (see the trim-vocabulary
  finding below); (2) wholesale row import — `select_kaggle_rows()` picked
  180 additional rows (Task 8) stratified across decade and body type,
  requiring both curb and gross mass so every added row yields a real
  `tare_ratio_q`.

Both obey the same fill rule as `cited` values: a bulk source may only fill
a blank cell, never overwrite a populated one.

## Provenance methods

Three methods populate `vehicle_prototypes_provenance.csv`, plus the
pre-existing `blanked_implausible` and `geometry_disagreement`:

- **`cited`** — a page was fetched and a quote captured **for that specific
  value**. This is the strict meaning of the word in this project; it is
  never used for a bulk-dataset fill.
- **`bulk_dataset`** — one citation covers an entire source (NRCan or
  Kaggle), not a single fetched value. Used both when a bulk source fills a
  blank cell and when it adds a whole new row (the Task 8 Kaggle import).
- **`bulk_disagreement`** — a bulk-sourced figure contradicted a value
  already populated in that cell. The disagreement is logged with both
  figures, but the bulk value is **not applied** — the original, sourced
  value is retained untouched.

**Cited beats bulk, always.** A bulk source may fill an empty cell. It may
never overwrite a populated one, regardless of which method populated that
cell first.

Current counts: `bulk_dataset` 313, `cited` 122, `bulk_disagreement` 59,
`blanked_implausible` 30, `geometry_disagreement` 29 (553 total).

## Gearing research scope

Full gearing — `final_drive_ratio` + complete `gearbox_ratios` + `tyre_code` — for a
curated subset of ~45 vehicles chosen to span regulatory category × powertrain × era,
so that every coursework variant has at least one fully worked prototype.

All remaining rows get `tyre_code` and `final_drive_ratio` only; `gearbox_ratios` left
empty rather than guessed.

Rationale: gearbox ratio sets are poorly published for trucks and for older models,
so researching all ~180 would produce large gaps regardless while costing several times
as much. Per-gear traction curves are only needed for worked examples, not for the
population statistics.

## Validation gates

Values failing these are blanked, not silently kept, with the reason recorded in the
provenance sidecar under `method = blanked_implausible` — matching the existing
convention in `enrichment_provenance.csv`. (`GATES` in `build_vehicle_prototypes.py`
is the single list this section documents.)

- `drag_coef` ∈ 0.20–1.00
- `rolling_resistance_coef` ∈ 0.005–0.030
- `wheel_radius_dynamic_m` ∈ 0.20–0.60
- `tare_ratio_q` ∈ 0.5–10.0
- `gross_mass_kg` ≥ `curb_mass_kg`
- `axle_configuration` matches `^\d+x\d+$` (plus `2x1` for motorcycles)
- `drive_layout` ∈ {FWD, RWD, AWD, 4WD}

Plus, ahead of the `tare_ratio_q` gate, a car-specific **payload floor**: in
`derive()`, before `tare_ratio_q` is computed, a car's `payload_kg` below
150 kg (less than two 75 kg occupants) is blanked as a data error rather than
a real homologated capacity — motorcycles are exempt, since they legitimately
carry less. This floor, not the `tare_ratio_q` ceiling, is now the primary
defence against physically impossible payloads.

### Why the `tare_ratio_q` range changed: 0.5–4.0 → 0.5–10.0

The original 0.5–4.0 ceiling was calibrated on a truck/bus-heavy population:
a truck carries its own weight again in cargo, so its `q` (curb/payload) is
low. Task 8 added 180 passenger cars, which carry a few occupants in a
comparatively heavy body — cars legitimately run a much higher `q`.

Evidence from the actual rejects: values in the 4.01–36.18 range were being
blanked. A Porsche 911 at `q = 4.05` (curb 1560 kg, payload 385 kg) is a
normal car and was being wrongly blanked. An Aston Martin at `q = 36.18`
(payload 55 kg — less than one adult) is a genuine data error, not a car with
an unusually light payload.

The fix widens the ceiling to 10.0 as a backstop, but the real gate is the
150 kg payload floor above: it catches the Aston-Martin-style error (via the
implausible payload itself) without also rejecting legitimately high-`q`
sports cars. Within the 180 new Task 8 rows, the fix raised the count
carrying `tare_ratio_q` from 130 to 178, with zero regressions on
previously-valid rows; across the full 500-row dataset, 278 rows carry
`tare_ratio_q`, spanning 0.53–8.76.

A separate gate does not blank but **warns**: rows sharing `(make, model, year,
generation, body_type)` must agree exactly on `wheelbase_mm`, `width_mm` and
`height_mm`, and on `length_mm` within 1%. A disagreement is logged under
`method = geometry_disagreement` and the values are left in place — a genuine
long-wheelbase variant should carry a different `body_type`, and this gate is
how that gets discovered rather than silently accepted as noise.

## Known findings about the bulk sources

Properties of the two bulk sources worth knowing before touching the
matchers again — none of these are defects to fix:

- **Trim-vocabulary incompatibility (Kaggle enrichment).** Kaggle's `Trim`
  column is an engine+transmission code (`"2.0 AT"`, `"1.0 AMT"`); this
  dataset's `variant_trim` is a marketing name (`"530e"`,
  `"2.0T 280hp Veloce"`, `"1.5 T-GDI 48V N Line Sky"`). The two vocabularies
  do not overlap and cannot be matched. This is why `match_kaggle()` yields
  few enrichment matches even with an exact make/model/year hit, and why
  Kaggle's real value in this dataset is the wholesale row import (Task 8),
  not enriching existing rows. Do not loosen the trim comparison to force
  more matches — a looser match attaches another vehicle's mass and
  performance figures to this row.
- **Kaggle's internal duplication.** Within the ~39,000-row pool eligible for
  selection, 17,548 rows are duplicate candidates across 9,046 colliding
  `(make, model, year, trim)` keys — roughly 45%. The source is not 70,823
  distinct vehicles; `select_kaggle_rows()` dedupes within its own selection
  for exactly this reason (see Task 8 in `progress.md`).
- **NRCan's low match rate (37/500) is a market mismatch, not a defect.**
  NRCan is Canadian-market fuel/energy consumption data; this fleet is
  largely European and Soviet-era. A low match rate here reflects the
  populations not overlapping, not a matching bug.

## Non-goals

- No new synthetic `family_envelope` rows. Real vehicles only.
- No modification of `prototype_enrichment.csv`, `prototype_index.csv`, or
  `vehicle_mass_80yr_dataset.csv` — all are read-only inputs.
- No statistics report or chart output in this phase. The CSV is the deliverable.
- No motorcycle expansion.
