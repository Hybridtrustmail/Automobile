---
doc: Data/Vehicle/docs/dynamics.md
type: reference
status: active
authority: informative
component: data
summary: Vehicle dynamics data and method notes.
read-first: []
roadmap: []
---

# Vehicle Dynamics Reference Table

`vehicle_dynamics_reference.csv` holds **published performance figures for named
real vehicle models**. It currently contains **159 rows covering 79 models**
between 1945 and 2024.

## Why this is a separate file

`vehicle_mass_80yr_dataset.csv` contains *modeled* class-envelope masses: every
number there is generated, and no row corresponds to a vehicle that was ever
built. This table is the opposite: every number is a figure stated by a source
that was actually opened, attached to a specific make, model and variant.

Merging the two would recreate the integrity problem the mass dataset was
cleaned up to remove — a reader could not tell which numbers were measured and
which were produced by a generator. The two files are joined, when needed, only
through the `SubClass` column, and the validator enforces that every `SubClass`
used here exists in the mass dataset.

## Layout

The table is long/tidy: **one row per (model, metric)**. Sparsity is the point.
A metric that no consulted source published is simply absent — it is never
filled with an estimate, a placeholder or a class average.

| Column | Meaning |
| --- | --- |
| `VehicleName` | Readable identifier, e.g. `Volkswagen Golf Mk1 GTI 1976 1.6`; derived from the four fields below it |
| `SubClass` | Join key into the modeled mass dataset |
| `Category`, `VehicleDomain` | Family codes, matching the mass dataset |
| `Make`, `Model`, `Variant` | The specific vehicle the figure belongs to |
| `AxleFormula` | `4x2`, `4x4`, `6x6`, `8x8`, ... or `Tracked`; empty if unsourced |
| `TyreSize` | Base fitment as printed by the source; `front / rear` when they differ |
| `ChassisSource_*` | Separate provenance for the two fields above |
| `ChassisNotes` | Caveats on the chassis data specifically |
| `ModelYearStart`, `ModelYearEnd` | Production period |
| `ReferenceYear` | The year the cited specification applies to |
| `Metric`, `Unit` | See below |
| `Value_Min`, `Value_Max` | Equal for a single published value; a genuine range otherwise |
| `Source_Tier` | How strong the provenance is |
| `Source_Reference`, `Source_URL` | The source actually consulted |
| `Retrieved` | Retrieval date |
| `Notes` | Caveats, disagreements between sources, limiter warnings |

### Metrics

`TopSpeed` (km/h), `Accel_0_100_kmh` (s), `Accel_0_60_mph` (s),
`Accel_0_32_kmh` (s, the 0–20 mph figure conventionally published for tracked
armour), `Gradeability` (%), `CurbMass` (kg), `GrossMass` (kg).

`GrossMass` holds combat weight / GVW / GVWR / loaded weight as the source
labels it; `CurbMass` holds empty, kerb or curb weight. Where a source gives a
weight without saying which it is, no mass row is recorded.

At most **two dynamics metrics** are recorded per model variant, by design and
enforced by the validator. Mass rows do not count against that limit.

### Ranges

`Value_Min` < `Value_Max` means one of three things, always explained in
`Notes`:

- the source itself gives a range (Renault Master E-Tech, 90–120 km/h);
- variants differ (John Deere 8R 410, 40 km/h standard / 50 km/h optional);
- **consulted sources disagree** (Leopard 2A7+, 68 vs 72 km/h).

The third case is recorded as a range rather than silently resolved in favour of
one source.

### Source tiers

- `Primary-manufacturer` — the manufacturer's own technical data.
- `Official-handbook` — a state or institutional reference handbook. Currently
  the NIIAT *Kratkiy avtomobilnyy spravochnik* (10th ed., Moscow, 1985), the
  RSFSR road-transport ministry's own specification tables.
- `Contemporary-road-test` — a period magazine road test.
- `Secondary-technical-database` — specialist compilations (LECTURA,
  automobile-catalog, ultimatespecs, EVSpecifications, GlobalSecurity.org).

Of 110 rows, 4 are `Primary-manufacturer`, 32 `Official-handbook`, 2
`Contemporary-road-test` and 72 `Secondary-technical-database`.
Manufacturer sites
frequently either omit performance data or block automated retrieval, and for
pre-1980 vehicles a primary source is often not online at all. The tier column
exists so this weakness is visible in the data rather than hidden.

## Measured tare ratios, and what they say about the model

For trailers the handbook prints own mass, rated payload and gross mass
together, so `q = M0/Mb` is **attested rather than modeled**. The identity
`curb + payload = gross` holds exactly for all 23 trailers recorded, which also
served as the OCR check: four blocks that failed it were dropped, not repaired.

Comparing those against the modeled 1985 values in `vehicle_mass_80yr_dataset.csv`:

| Family | n | measured median q | modeled q | ratio |
| --- | --- | --- | --- | --- |
| O1 Light Trailer | 3 | 6.50 | 0.32 | 20.3 |
| O2 Utility and Caravan | 1 | 0.68 | 0.33 | 2.1 |
| O3 Medium Trailer | 6 | 0.48 | 0.23 | 2.1 |
| O4 Box and Curtain-side | 12 | 0.32 | 0.23 | 1.4 |
| O4 Tank and Bulk | 1 | 0.30 | 0.27 | 1.1 |

**The model understates trailer tare ratio, and increasingly so as the trailer
gets smaller.** The O1 figure of 20x overstates the case: two of those three are
caravans, whose payload is trivial by design. Excluding them, the one genuine
light cargo trailer (MMZ-81021) still shows q = 0.88 against a modeled 0.32,
about 2.7x.

Sample sizes are far too small to recalibrate the mass model from — this is a
flag for investigation, not a correction factor.

## Known limitations

- **Coverage is uneven by era and origin.** Of 159 rows, 97 are civilian road,
  55 military, 4 civilian off-road and 3 agricultural. The civilian side is
  dominated by a single year: 44 of the 57 civilian models are 1985 entries from
  one handbook. Western commercial vehicles from 1990 onward and special-purpose
  bodies remain barely represented; the bus, truck and agricultural web-sourcing
  passes never completed.
- **The Soviet-bloc rows are dated 1985–1985 on purpose.** A specification
  table attests to its own edition, not to a production span, so no production
  period is claimed beyond what the handbook itself states (only the LiAZ-677's
  1967 start year is given). Do not read those rows as covering the model's
  whole life.
- **Top speeds for heavy vehicles are usually limiter values.** The eActros 600
  at 90 km/h and the Citaro at 100 km/h reflect regulation and configuration,
  not capability. These are flagged in `Notes` and must not be read as
  performance measurements.
- **Acceleration figures are rare outside passenger cars.** Buses, trucks,
  tractors and even tracked armour almost never publish them — no source
  consulted publishes a 0–20 mph figure for any tracked vehicle except the
  M1A2. Gradeability is the meaningful second metric for those families and is
  now populated for 16 models, all military.
- **Gradeability is not one quantity.** The BvS10's 100% is a *stop-and-restart*
  gradient, the Bv 206S's 30–100% is terrain-dependent, and the HEMTT's 30–60%
  depends on whether a trailer is towed. Never compare these figures without
  reading `Notes`.
- **Chassis data carries its own provenance**, because it almost never comes
  from the source that published the performance figure. `AxleFormula` is set
  for 55 of 56 models, `TyreSize` for 25. For military vehicles the axle
  formula is taken from the model designation printed in the cited performance
  source ("M35 2.5-ton 6x6"), so `ChassisSource_URL` is empty there by design.
- **Three tyre entries are weaker than they look**, each flagged in
  `ChassisNotes`: the 300 SL's 6.50-15 comes from a 1955-dated entry for a 1954
  reference year and conflicts with a widely quoted 6.70-15; the eActros 600's
  front/rear split is one configured customer build, not a base fitment; the
  Master E-Tech's 205/75 R16C is the fitment of a single press vehicle in a road
  test. No tyre size is recorded for the 1957 Massey Ferguson 35 at all — the
  two databases consulted disagree and neither covers the UK-built tractor.
- **Test standards are not harmonised.** A 1959 road-test 0–60 mph, a 1970s
  manufacturer claim and a modern WLTP-era 0–100 km/h are not directly
  comparable. Treat cross-era comparisons as indicative only.
- Some rows record an approximate published phrasing (the 1949 Beetle's "around
  60 mph" is stored as 96–97 km/h). This is noted per row.

## Validation

`validate_vehicle_dynamics_reference.py` checks metric names, units,
per-metric plausibility bounds, `Value_Min <= Value_Max`, year ordering and
scope, `ReferenceYear` inside the production period, source tier vocabulary,
`https://` source URLs, duplicate model/metric rows, the two-metric limit,
`AxleFormula` format, per-model consistency of the chassis attributes, and
`SubClass` cross-linkage into the mass dataset.

Unlike the mass-dataset validator, these checks are not tautological: the table
is hand-curated, so the checks catch real transcription and provenance errors.
Each of the checks above has been confirmed to fail on a deliberately corrupted
copy of the table.

## Regeneration

```bash
python3 build_vehicle_dynamics_reference.py
python3 validate_vehicle_dynamics_reference.py
```

New records are added to `RECORDS` in `build_vehicle_dynamics_reference.py`,
with the source registered once in the `S` dictionary.
