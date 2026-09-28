---
doc: Data/Vehicle/docs/vehicle_prototypes_extension_design.md
type: architecture
status: active
authority: informative
component: data
summary: Vehicle prototypes extension design.
read-first: []
roadmap: []
---

# Vehicle prototype dataset — extension phase

Date: 2026-08-01
Status: implemented 2026-08-03
Builds on: `vehicle_prototypes_design.md` (same schema, same tier/provenance conventions,
same validation gates — not repeated here).

## Purpose

Broaden `vehicle_prototypes.csv` (currently 246 rows: 95 enrichment + 67 fastsim +
84 niiat trucks) with more real, popular, easy-to-source vehicles, and specifically
close the gap on PHEV/EV reduction-gear ratios, which the original build left almost
empty (`final_drive_ratio` filled on 66/246 rows, `gearbox_ratios` on 79/246, nearly
all from the ICE side).

## Recon result

Surveyed `/media/cigameca/SHARED/Work/Vehicle_simulation/` (beyond FASTSim) and the
two named Spravochnik PDFs for additional structured sources:

| Source | Verdict |
|---|---|
| `EV_sim-main.zip` → `EV_sim/data/EV/EV_dataset.csv` | Use — 13 real EVs |
| `My_PHEV/config/outlander_2014.yaml` | Use as cross-check for Outlander row |
| Outlander PHEV 2014 UK brochure (pp. 24–25) | Use — fully specced |
| NIIAT tom 1 (buses), OCR text | Use — needs new parser, different layout than tom 2 |
| Prihodko spravochnik OCR | Skip — repair/diagnostics manual, not a per-model spec catalog |
| `EPA_ALPHA`, `carculator-master.zip`, `sumo-main.zip` | Skip — engine dyno maps, generic archetypes, traffic sim |

## Work streams

### 1. EV_sim import (Tier D: `ev_sim`)

13 rows from `EV_sim/data/EV/EV_dataset.csv`: Chevrolet Bolt/Volt-era, Tesla Model
3/S/X/Y trims, Audi e-tron 55, Porsche Taycan 4S/4S Plus. Columns map directly:
`drag_coef`, `frontal_area_m2`, `curb_mass_kg`, `wheel_radius_*_m`,
`final_drive_ratio` (single-speed reduction gear), `transmission_eff`,
`motor_power_kw`. Skip Tesla Model Y if it duplicates the existing enrichment-tier
row (design doc notes Tesla Model Y as the one overlap between tiers) — keep the
enrichment-tier row, do not double up.

### 2. Outlander PHEV 2014 row (Tier E: `brochure`)

One new row, `powertrain=Hybrid`, `powertrain_detail=PHEV`. From the UK brochure,
cross-checked against the user's own fitted `outlander_2014.yaml`:

- Engine 1998cc, 89kW/190Nm; front motor 60kW/137Nm; rear motor 60kW/195Nm;
  battery 12kWh Li-ion
- Ratios: engine final drive 3.425, front motor reduction 9.663, rear motor
  reduction 7.065 — recorded as `final_drive_ratio` = engine ratio, with the two
  motor ratios in `notes` (schema has one final-drive column; this is the first
  row needing more than one, so rather than add columns for a single vehicle,
  note the split ratios in text and flag it in provenance)
- Kerb 1810kg, GVW 2310kg, tyres 225/55R18, drag/frontal-area/rolling-resistance
  from the yaml cross-check (brochure doesn't state these), flagged `data_tier`
  mixed brochure+model-fit — recorded per-field in the provenance sidecar

### 3. NIIAT tom 1 buses (Tier `niiat`, same as trucks)

New small parser for tom 1's OCR layout (headers aren't 8-space-indented and lack
the "ТРАНСПОРТНЫЕ СРЕДСТВА" running header that `parse_niiat.py` keys on — confirmed
in recon). Target ~15–25 bus entries (M2/M3), same field extraction pattern as
`parse_niiat.py` (mass, dimensions, engine, axle config). Written as
`parse_niiat_buses.py`, output merged into the same `niiat` tier via
`build_vehicle_prototypes.py`.

### 4. Web-researched popular cars across decades (Tier C: `web-cited`)

~30–50 additional passenger car rows, chosen for being genuinely high-volume /
well-known nameplates (not curiosities), spanning from ~1970s–1990s icons (e.g.
VW Golf/Beetle, Lada 2101–2107, Toyota Corolla early gens, Ford Escort, Fiat 126)
through 2000s–2020s bestsellers not already covered by the enrichment or FASTSim
tiers (Toyota RAV4/Camry, Honda Civic/CR-V, Ford F-150, VW Golf Mk8, Tesla Model 3
base if not already in EV_sim). Every value cited with source URL and supporting
quote in `vehicle_prototypes_provenance.csv`, same convention as the original
design's Tier C. Older vehicles (pre-1990s) will have thinner fill on
drag_coef/rolling_resistance/gearing — left blank rather than guessed, consistent
with the existing validation-gate policy.

## Non-goals (unchanged from base design, restated)

- No synthetic rows. No modification of `prototype_enrichment.csv`,
  `prototype_index.csv`, or `vehicle_mass_80yr_dataset.csv`.
- No use of the Prihodko spravochnik as a row source (recon: not a spec catalog).
- No new schema columns for the Outlander split ratios — captured in `notes` +
  provenance instead, since it's a one-off case, not adding a general column for
  a single vehicle.
