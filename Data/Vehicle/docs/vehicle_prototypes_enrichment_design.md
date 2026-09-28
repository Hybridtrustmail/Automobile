---
doc: Data/Vehicle/docs/vehicle_prototypes_enrichment_design.md
type: architecture
status: active
authority: informative
component: data
summary: Vehicle prototypes enrichment design.
read-first: []
roadmap: []
---

# Vehicle prototypes — bulk enrichment, schema fill and directory phase

Date: 2026-08-05
Status: implemented; current schema and provenance rules are maintained in
`vehicle_prototypes_design.md`. Counts below describe the historical starting point.
Builds on: `vehicle_prototypes_design.md` (schema, tiers, provenance, gates),
`vehicle_prototypes_drivetrain_schema_review.md` (identity, sidecar contract).

**Goal:** fill the columns that are empty or thin, remove two schema defects,
tidy the working directory, and enrich from two newly supplied bulk datasets —
growing the dataset from 320 to about 500 rows, every addition carrying a real
`q`.

## Where the data stands

320 rows × 82 columns. Three columns are entirely empty (`drivetrain_config`,
`gradeability_pct`, `transfer_ratios`) and about 30 sit under 25% fill.

## The two new sources

Both are in `Data/Data sources/`. They are complementary, not overlapping.

| | Kaggle *Car Dataset 1945-2020* | NRCan *open_canada* |
| --- | --- | --- |
| Size | 70 823 rows × 78 cols | ~29 000 rows across 4 files |
| Citation | doi.org/10.34740/kaggle/ds/2938279 | open.canada.ca |
| `Generation` | **100%** | — |
| curb / full mass | 76% / 56% | — |
| `payload_kg` | 34% | — |
| axle load | 9% (6 357 rows) | — |
| gears / transmission / drive wheels | 82-85% | transmission only |
| city + highway fuel | ~55% | ~100% |
| CO2 | 2.6% | ~100% |
| **kWh/100 km city/hwy/combined** | 15 rows — unusable | **1 212 rows** |
| electric range, recharge time | ~0 | full |

Kaggle brings masses, generation and gearing but has effectively no EV data.
NRCan brings consumption and CO2 including the electric triplet, but no masses.

### Quality audit, done before designing on this

Kaggle was spot-checked against rows this project had already researched from
manufacturer and period sources. On every directly comparable vehicle it matched
exactly:

| Vehicle | This dataset (source) | Kaggle |
| --- | --- | --- |
| Fiat 126, 1972 | 580 kg (period spec page) | 580, plus full 920 / payload 340 |
| Trabant 601 | 615 kg (period spec page) | 615, plus full 1000 |
| Mitsubishi Outlander PHEV 2014 | 1810 / 2310 (UK brochure PDF) | 1810 / 2310, plus payload 500 |

The Outlander match is the strongest signal: it reproduces a brochure read
page-by-page, and correctly distinguishes the PHEV among 16 Outlander trims.

The VW Golf appears to disagree (Kaggle 910/940 vs this dataset's 750) but does
not: those are the 1.5/1.6 trims against our 1.1, which Kaggle lacks. The 2CV is
absent entirely, most likely an accent-spelling issue (`Citroen` vs `Citroën`) —
which is exactly the matching hazard this design has to handle.

## Directory

```
Data/Vehicle/
  vehicle_prototypes.csv        the deliverable
  build/    build_vehicle_prototypes.py, parse_niiat.py
  sources/  manual_vehicles.csv, niiat_trucks.csv,
            drivetrain_configs.csv, prototype_enrichment.csv
  out/      vehicle_prototypes_provenance.csv
  docs/     the .md files
  archive.zip
```

`archive.zip` takes `prototype_index.csv` (4.1 MB artifact whose generator no
longer exists), `enrichment_provenance.csv` (superseded), and `__pycache__`.

Moved OUT of `Vehicle/`, up to `Data/`: `vehicle_mass_80yr_dataset.csv` and
`vehicle_dynamics_reference.csv`. They belong to the other pipeline —
`build_q_web_reference.py` reads the mass dataset to generate the browser's
`q-reference.js` — and are not inputs to this build.

Delete the two empty directories `Data/Data` and `Data/Engines`.

All paths in `build_vehicle_prototypes.py` and `parse_niiat.py` must be updated
in the same change, and the build re-run to prove nothing broke.

## Identity

`prototype_id` becomes `vp-` plus the first 6 hex characters of
`sha256(key)` where `key` is the stable identity joined by `|` and lowercased:

```python
key = "|".join([make, model, str(year), variant_trim]).strip().lower()
pid = "vp-" + hashlib.sha256(key.encode("utf-8")).hexdigest()[:6]
```

`sha256` is specified rather than Python's `hash()` because the latter is salted
per process and would produce different ids on every run — the precise failure
this change exists to remove.

This fixes two things at once. The old `car-0001` form duplicated
`vehicle_kind`, and — more seriously — it was a per-kind counter assigned after
merge, so every id shifted whenever an earlier loader changed. Ids are now
stable across rebuilds and safe to reference.

`vehicle_kind` remains the single place vehicle kind is recorded.

On collision (two vehicles hashing to the same 6 hex chars), extend that id to 8
characters rather than renumbering — a collision must never silently move an
existing id.

## Schema additions

Replace `fuel_consumption_combined` with parallel triplets, all per 100 km:

| Column | |
| --- | --- |
| `fuel_city_l_100km`, `fuel_highway_l_100km`, `fuel_combined_l_100km` | ICE |
| `energy_city_kwh_100km`, `energy_highway_kwh_100km`, `energy_combined_kwh_100km` | electric |
| `electric_range_km`, `recharge_time_h` | electric |

`co2_emissions_gkm` stays as it is.

A plug-in hybrid legitimately fills **both** triplets. That is the case a single
shared consumption column cannot express, and the reason for the split.

`kWh/100 km` is used rather than kWh/km because it is NRCan's own published unit
and mirrors L/100 km, so the two read on the same scale.

## Columns filled without research

**`regulatory_category`** — derived in `derive()` from `vehicle_kind` and
`gross_mass_kg`, using the boundaries the build already enforces elsewhere:

| kind | rule |
| --- | --- |
| car | M1 |
| truck | N1 ≤ 3500 kg; N2 ≤ 12 000; N3 > 12 000 |
| bus | M2 ≤ 5000 kg; M3 > 5000 |
| motorcycle | L |

Derived, never stored, so it cannot drift from the mass it depends on. Where
gross mass is unknown the category stays blank rather than being guessed.

**`drivetrain_config`** — filled on the main row from its `is_primary` sidecar
line where one exists; blank otherwise. Do not mint a token per vehicle: the
column means "which of several configurations this row's masses describe", and
a vehicle with only one configuration has no such distinction to record.

**`gradeability_pct` and `transfer_ratios`** stay empty. Neither bulk source
carries them, and inventing them would defeat the point. Recorded here so their
emptiness is a known state rather than an oversight.

## Enrichment rules

**Cited beats bulk, always.** Where a bulk source disagrees with a value this
project fetched and quoted, the existing value stands and the disagreement is
logged under a new provenance method `bulk_disagreement`. It is never applied.
The Outlander's brochure figures do not get overwritten because Kaggle agrees —
and would not be overwritten if Kaggle disagreed.

**Bulk gets its own provenance method.** `method = bulk_dataset`, one citation
per source. Never `cited`: in this project `cited` means a page was fetched and
a quote captured *for that value*. Blurring the two would silently devalue every
row researched by hand.

**Fill blanks only.** A bulk source may populate an empty cell. It may not
overwrite a populated one.

**Matching must be conservative.** A wrong match is worse than no match, because
it attaches one vehicle's masses to another. The rule is mechanical:

1. **Normalise** make and model on both sides — lowercase, strip accents
   (`Citroën` → `citroen`, which is what lost the 2CV), strip punctuation and
   collapse whitespace. Kaggle's column is misspelled `Modle`.
2. **Make and model must match exactly** after normalisation. No fuzzy matching
   on these two fields.
3. **Year** must fall inside `Year_from…Year_to` inclusive (Kaggle), or match
   `Model year` exactly (NRCan).
4. **Trim** decides only when steps 1-3 leave more than one candidate. If
   several remain and no trim string matches, take **no** match and log the row
   as ambiguous with its candidates. Never pick the first, and never pick the
   closest.

Report three counts: matched, unmatched, ambiguous. Ambiguity is a signal the
key is too coarse, not a nuisance to be tuned away.

## Expansion to ~500 rows

Add about 180 rows from Kaggle, selected by:

1. **Both `curb_weight_kg` and `full_weight_kg` present** — so every new row
   carries a real `q`, which is what the book computes. Kaggle has ~39 000 such
   rows, so the pool is deep.
2. **Stratified** across decades and body types, to avoid the set collapsing
   onto modern hatchbacks.
3. Not already present under any existing key.

New rows enter at tier `kaggle-spec` with `method = bulk_dataset` provenance.
They are explicitly a different evidential class from the web-cited rows, and
any population statistic over the dataset must be able to separate them.

## Success criteria

1. Build runs clean, no warnings, from the reorganised directory.
2. `prototype_id` is stable: rebuilding twice with no input change produces
   identical ids.
3. No cited value is overwritten by a bulk source anywhere; every disagreement
   is logged rather than applied.
4. `regulatory_category` filled wherever kind and gross mass allow, blank
   otherwise.
5. Row count ~500, and every added row has a non-empty `tare_ratio_q`.
6. Consumption triplets populated wherever NRCan matched; PHEV rows carry both.
7. `vehicle_mass_80yr_dataset.csv` is byte-identical after the move.

## Open questions

- The unmatched rate against both sources is unknown until the matcher runs. If
  it is high, the matching rules need loosening — but deliberately, with the
  loosened rule stated, not by lowering a threshold until numbers look better.
- NRCan is Canadian-market data. Where it disagrees with an EU figure already in
  a row, that is a market difference rather than an error, and `cited beats
  bulk` handles it — but it is worth watching how often it happens.
