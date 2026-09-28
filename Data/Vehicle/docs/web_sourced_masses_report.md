---
doc: Data/Vehicle/docs/web_sourced_masses_report.md
type: reference
status: active
authority: informative
component: data
summary: Web-research log for the data/web-sourced-masses gap-filling pass -- what was searched, found, and why each unfilled row was refused.
read-first: []
roadmap: []
---

# Web-sourced masses: 28-row gap-filling pass

Branch `data/web-sourced-masses`. Goal: find real, citable curb/gross masses
for the 28 non-car rows missing one mass field. Result: **1 of 28 filled**,
27 refused. Every refusal is a deliberate outcome of the rule "never write a
number you cannot cite to the exact vehicle" — this report records what was
tried and why each one failed that bar, not a shortfall against a coverage
target.

## Summary

| Metric | Value |
|---|---|
| Rows targeted | 28 |
| Filled (cited) | 1 (`vp-ae64b8`, Ford Ranger, `gross_mass_kg`) |
| Refused | 27 |
| `vp-1ffb60` (ПАЗ-3205-110-80) repaired? | **No** — searched specifically, no reliable citation found for this exact sub-variant |
| Row count | 500 (unchanged) |
| q points before | 299 |
| M3 distinct mass pairs before | 8 |
| q points after | 300 |
| M3 distinct mass pairs after | 8 (the one fill was N1, not M3) |

## What was filled

**`vp-ae64b8` — Ford Ranger Double Cab High Rider XLT 4x4 3.2 TDCi (2019)**,
`gross_mass_kg` = 3200 kg.

Source: [Autonet Helderberg spec sheet, Ford Ranger 3.2TDCi XLT 4X4 A/T P/U D/C](https://www.autonet.co.za/helderberg/car-specification/FORD-RANGER-3-2TDCi-XLT-4X4-AT-PU-DC--26975)
(dealer spec sheet, "Gross Laden Mass" 3200 kg, "Licencing Mass" 2118 kg).

This page describes the automatic derivative, not the row's manual
derivative (whose curb mass, 2159 kg, was already confirmed independently by
[carfolio.com](https://www.carfolio.com/ford-ranger-double-cab-high-rider-3.2-tdci-xlt-4x4-645227)
for the exact 3.2 TDCi XLT 4x4 manual spec). GVM on the Ranger T6 platform is
an axle/chassis rating, not powertrain-dependent — cross-checked against an
unrelated listing for the 2.0 SiT XLT 4x4 double cab (different engine,
tare 2044 kg) which independently states the identical 3200 kg GVM, showing
the rating does not move with engine or transmission on this cab/trim.
Implied payload: 3200 − 2159 = 1041 kg, plausible for a 4x4 double-cab bakkie.

Recorded in `Data/Vehicle/sources/web_sourced_masses.csv` and applied by the
new `apply_web_sourced_masses()` import step in `build_vehicle_prototypes.py`.

## The 27 refusals

| prototype_id | vehicle | missing | what was searched | why refused |
|---|---|---|---|---|
| vp-1ffb60 | ПАЗ-3205-110-80 (M3) | gross | Russian bus-spec sites (autoopt.ru, ru.wikipedia.org, spectekhnika.info, targeted queries for "3205-110-80") | autoopt.ru's 2005 catalog page (the best candidate) is CAPTCHA-gated and unreadable. A figure of 8060 kg surfaces on spectekhnika.info but is attributed to "the base model" generically, never tied to the `-110-80` suffix specifically — the same failure mode that produced the original bad 5990 kg value. Refused rather than repeat the error. |
| vp-3363f3 | Solaris Urbino 12 electric, ZF CeTrax 220kW/420kWh (M3) | curb | Solaris's own UITP-2023 press-kit PDF (manufacturer primary source, matches the row's exact spec string) | The PDF confirms GVW 20 000 kg (already in the row) but states no kerb/empty weight at all — Solaris's spec sheet for this exact variant simply doesn't publish it. |
| vp-eaba4e | Škoda 32Tr SOR, low-entry 3-door (M3) | gross | en.wikipedia.org, czwiki.cz (unreachable), targeted "celková hmotnost"/GVW queries | Only curb weight (10 180 kg, matching the row) is published anywhere found; no source states a GVW/technically-permissible-maximum figure for this trolleybus. |
| vp-ca9371 | Mercedes-Benz Actros 4141 K (N3) | curb + gross | truck1.eu spec page, lectura-specs, multiple used-tipper listings | truck1.eu states GVW 40 000 kg (self-consistent with its own payload figure) while other summaries/the type designation imply 41 000 kg — a ~2.5% disagreement between sources on the same nominal spec. Curb mass only available from used-equipment listings (14 600–15 030 kg range) which describe as-built tippers with a mounted body, not the bare chassis-cab "curb mass" this dataset needs. No single authoritative figure for either field. |
| vp-afa499 | Volvo FH D12D420 (N3) | curb + gross | Volvo spec pages, mymotorlist.com (engine only) | Found only a general FH kerb-weight range (6570–6945 kg) not attributed to the D12D420-engined variant specifically. No gross mass found at all. |
| vp-72742f | Scania 114/124/164 (N3) | curb + gross | — | Source string names three distinct model series (114/124/164) spanning very different GVW classes; this is not one vehicle. No exact-variant search is possible without a specific model+chassis. |
| vp-d62ca0 | Mercedes-Benz eActros 300, 4x2 rigid, 3-battery (N3) | curb | Daimler Truck press materials, lectura-specs (403), hylane spec sheet | Found a curb weight for the unrelated eActros 600 LS tractor (11 546 kg) but nothing for the eActros 300 rigid 3-battery configuration this row states. |
| vp-76fb6a | Renault Rapid Van (F40) 1.6 D (F404) 55 Hp | curb + gross | truck-data.com, truck1.eu — both list this exact model string | Both pages carry the exact matching spec string but neither populates a weight field (403/empty on fetch). No usable figure found. |
| vp-6c6027 | Renault Trafic Van (TXX) 1.6 65 Hp | curb + gross | truck1.eu (exact model string match) | Page has full engine data but no weight fields populated. |
| vp-6a5159 | Chevrolet Colorado 2WD Diesel (2020) | gross | auto123.com, thecarconnection.com | Two sources agree GVWR is 6200 lb for "the diesel" Colorado, but a third (auto123's trim list) states the 2.8L Duramax was Crew-Cab-4WD-only for this model year, contradicting the row's stated "2WD Diesel" configuration. Could not confirm this exact 2WD+diesel combination existed with that GVWR. |
| vp-c23578 | Ford F-150 Lightning 4WD (2022) | gross | tfltruck.com, jdemmerford.com and others (curb/GVWR by battery pack) | Every real Lightning trim's curb weight (6015–6590 lb / 2728–2989 kg) runs meaningfully above this row's existing curb (2592.4 kg / 5715 lb) — a FASTSim simulation figure, not a documented production trim. No trim matches closely enough to attach a cited GVWR to. |
| vp-fcace9 | Class 4 Truck (Isuzu NPR HD) (N2) | gross | — | FASTSim archetype name, not one production vehicle/trim/year. The row's own curb (6590 kg) already sits near typical NPR-HD GVWR territory, a sign the "curb" figure itself is a simulation approximation rather than a real spec — attaching a cited real-world GVWR to it would misrepresent the row. |
| vp-7ee625 | Line Haul Conv (N3) | gross | — | Generic FASTSim simulation archetype, not a real, identifiable production vehicle. No exact vehicle to search for. |
| vp-7717a5 | Nissan Navara (2016) | gross | 4x4australia.com.au, nissan-cdn.net spec sheets, cars-data.com | Found GVM figures (2910 kg) but only for specific dual-cab 4x4 trims; the row is FASTSim's generic "Nissan Navara" with no cab/trim identified, and its curb (2064 kg) doesn't closely match any single documented trim. |
| vp-d114a1 | Regional Delivery Class 8 Truck (N3) | gross | — | Generic FASTSim simulation archetype, not a real vehicle. |
| vp-2b190f | Toyota Hilux Double Cab 4WD (2021) | gross | — | Generic FASTSim archetype name; no cab/engine/trim to pin an exact GVM to. |
| vp-00abcc | ГАЗ 33022/33028/2779 | curb | (already-cited handbook PDF has a full breakdown, see below) | Source row itself names 7 distinct sub-modifications (vans/pickups/ambulance) whose curb masses span 2020–2830 kg (>40% spread) against one shared 3500 kg GVW — a genuinely variant-dependent figure, and the row carries no `variant_trim` to pick one. The one place this breakdown exists is the same offline PDF already cited for this row (`kratkii_avtomobilnyi_spravochnik_tom_2_2006.pdf`, pp. 68–69), which has no stable public URL — out of scope for a *web*-sourced citation. |
| vp-7a4688 | Citroën Berlingo | curb | Russian spec aggregators (cartechnic.ru-style sites) | Row has no year/generation/trim; Berlingo curb mass varies materially across two generations and multiple body/roof options in the handbook's ~2006 window. Refused for variant ambiguity. |
| vp-defc48 | Hyundai H-1 | curb | — | Same handbook table (line 6704 ff.) shows 4 distinct H-1 sub-variants (3-seat/6-seat × short/long wheelbase) with materially different curb masses; row carries no trim to disambiguate. Refused. |
| vp-33dc0f | Mazda B2500 | curb | — | No year/cab/trim given; B2500 spans single/double cab and multiple GVW classes. Refused for variant ambiguity. |
| vp-8824df | Mercedes-Benz "ТО-У" | curb | — | Source model string itself is not a standard nameplate (likely a garbled/abbreviated catalog entry); could not even confirm which real Mercedes model this refers to. Refused. |
| vp-917b94 | Peugeot Boxer | curb | cartechnic.ru, 110km.ru | Found a Boxer curb/gross pair (1895 kg / 3000 kg) for one generic trim, but the row's gross (3500 kg) implies a heavier/longer wheelbase variant with a different curb mass — picking the 1895 kg figure would silently graft the wrong trim's curb onto this row's stated 3500 kg gross. Refused. |
| vp-2d5b2a | Peugeot Partner | curb | — | No year/trim; Partner spans multiple generations and payload classes across the handbook's window. Refused. |
| vp-a0dbef | Renault Kangoo Express | curb | — | Same ambiguity as Partner/Berlingo. Refused. |
| vp-14d185 | Volkswagen LT28/LT35 | curb | — | Source string itself names two distinct GVW-class variants (LT28 vs LT35); no single curb mass applies to both. Refused. |
| vp-ae64b8 (curb was already present) | — | — | — | n/a — only `gross_mass_kg` was missing here; see "What was filled" above. |
| vp-30d053 | Chevrolet Silverado 1500 Crew Cab 6'6" 5.3L V8 (2019) | gross | auto123.com (found a full spec block for a *different* trim/cab) | The only detailed GVWR/curb pair found (auto123, RST 4WD **Double Cab** std bed) is a different cab style than the row's Crew Cab, with a curb weight (2184 kg) that doesn't match the row's existing curb (2224 kg) closely enough to trust the paired GVWR for this exact configuration. Refused. |
| vp-195eec | Toyota Hilux Single Cab 2.8 GD-6 4x4 Legend 50 (2019) | curb + gross | carfolio.com, cars.co.za, Toyota SA dealer PDFs, autotrader.co.za | Every source found publishes engine/performance data for this exact trim string but leaves the weight fields blank; a GVM of 2910 kg was found but only for the Double Cab Legend, not Single Cab. Refused both fields. |

## Verification output

```
$ cd Data/Vehicle && python3 build/build_vehicle_prototypes.py
Web-sourced masses: filled 1, refused 0, unmatched 0
...
wrote .../vehicle_prototypes.csv: 500 rows
wrote .../vehicle_prototypes_provenance.csv: 688 provenance rows (40 blanked)

$ python3 build/build_q_dataset.py
...
300 q points -> .../q-dataset.json

$ python3 -m unittest discover -s build -p 'test_*.py'
Ran 106 tests in 0.002s
OK

$ cd ../.. && npm run check
... vitest/node --test: 26/26 pass ...
check-docs: all documentation files pass
✓ built in 207ms
```

Row count: 500 (unchanged). q points: 299 → 300. M3 distinct mass pairs: 8 → 8
(the one fill landed on an N1 row, not M3).

Row count later dropped to 497 in a follow-up cleanup that removed three
FASTSim simulation archetypes (`vp-fcace9`, `vp-7ee625`, `vp-d114a1` above)
from the build entirely — see `ARCHETYPE_NAMES` in
`build_vehicle_prototypes.py`. None of the three ever produced a q point (see
their refusal reasons above: all three were refused for `gross_mass_kg`), so
that removal cost no q data. `vp-72742f` (Scania 114/124/164, refused above
for the same "not one vehicle" reason) and `vp-00abcc` (ГАЗ 33022/33028/2779)
were kept and marked `model_scope=family` in that same cleanup rather than
deleted, since — unlike the archetypes — they are real sourced handbook
readings, just readings that cover more than one model.

## Assessment for continuing to the 111 car rows

Effort-per-fill on this pass was roughly 25–30 tool calls (search + fetch)
per successfully-cited value, against a background refusal rate of 96%
(27/28). The car population is likely to convert at a meaningfully higher
rate than these non-car rows did, because:

- Car curb/GVW figures are far more commonly published by manufacturers and
  spec aggregators than commercial-vehicle GVM/GVW figures, which are often
  behind dealer configurators or absent from spec pages entirely.
- Many of this batch's refusals were structural, not just hard-to-find: FASTSim
  simulation archetypes ("Line Haul Conv", generic "Nissan Navara") and
  multi-model catalog headings ("ГАЗ 33022. 33028. 2779", "Scania
  114/124/164") have no single exact vehicle to cite in the first place. The
  car rows should not carry this same structural problem as heavily, though
  some nameplate-collision risk will remain per the dataset's documented
  history.

Recommend continuing, but budget for a similar multi-search-per-row cost and
expect a non-trivial refusal rate — the value of this work is in what gets
correctly refused as much as what gets filled.

---

# Batch 1 of 4: electrified vehicles (54 rows)

Branch `data/web-sourced-masses`, continuing from the 28-row non-car pass above.
Target: the 54 non-motorcycle rows with an identifiable make/model/year, an
electrified powertrain, and no usable curb/gross pair. Result: **12 of 54
filled**, 42 refused. PHEV and Hybrid rows were worked first as instructed.

## Summary

| Metric | Before | After |
|---|---|---|
| Row count | 497 | 497 |
| q points | 300 | 312 |
| M1 + `hybrid` (PHEV+HEV) distinct mass pairs | 2 | 7 |
| M1 + `electric` (BEV) distinct mass pairs | 10 | 13 |
| Rows targeted | 54 | — |
| Filled (cited) | — | 12 |
| Refused | — | 42 |

`MIN_CURVE_POINTS` is 8 distinct (curbMass, grossMass) pairs. **M1+electric
was already over the bar (10) and is now comfortably clear at 13. M1+hybrid
rose from 2 to 7 — one confirmed pair short of its own curve.** The blockers
are named under "Refusals" below; none could be cleared without writing a
number that fails the citation or the basis test.

## Method: the Dutch national vehicle register (RDW) as a primary source

Every `market=EU` row in this batch was resolved against **RDW open data,
dataset `m9d7-ebf2` ("Gekentekende voertuigen")** — the Netherlands' national
registration database, which carries per-vehicle
`technische_max_massa_voertuig` (the EU type-approval *technically permissible
maximum laden mass*) and `massa_ledig_voertuig` (unladen mass). This is
category 2 on the acceptable-source list.

Two properties make it unusually safe here:

1. **It reports the row's own curb mass back.** For most enrichment rows the
   dataset's `curb_mass_kg` matches RDW `massa_ledig_voertuig` *exactly*
   (BMW 530e 1810, Mazda MX-30 1620, Lexus RX450h 2075, Subaru Forester 1631)
   or within a few kg (Tesla Model Y 1950 vs 1956). That is a direct,
   independent confirmation that the right variant was found — the failure
   mode that corrupted this dataset twice before is exactly what an exact
   curb match rules out.
2. **It shows whether the figure is trim-dependent.** Grouping hundreds or
   thousands of registrations by type-approval `variant` shows immediately
   whether one permissible mass covers all equipment levels. Every fill below
   was made only where a single `variant` carried a single figure across all
   its `uitvoering` codes.

The method was validated against manufacturer documents on three vehicles
before being trusted on the rest, and agreed every time:

- **Mazda MX-30** — RDW 2119 kg; Mazda-derived spec page states 2119 kg.
- **Lexus RX 450h** — RDW 2715 kg; Lexus UK's own technical-specification
  sheet (engine 2GR-FXS) states "Gross vehicle weight (kg) 2,715".
- **VW Golf GTE** — RDW 2040 kg; Volkswagen's MY2020 *Technik und Preise*
  brochure states "zul. Gesamtgewicht 2.040", Leergewicht 1.615 kg (9 kg from
  the row's 1624 kg) and Zuladung 403–585 kg, bracketing the implied 416 kg.

For the last two, the manufacturer document is cited as the source and RDW as
the cross-check.

## What was filled (12 rows)

| prototype_id | vehicle | gross_mass_kg | payload vs row curb | source |
|---|---|---|---|---|
| vp-53afb4 | BMW 530e (G30 LCI), 2020 — **PHEV** | 2505 | 695 | RDW variant 11AG, 894 registrations at massa ledig 1810 (= row curb exactly) |
| vp-f491d1 | VW Golf GTE 1.4 eHybrid 245, 2020 — **PHEV** | 2040 | 416 | VW MY2020 *Technik und Preise* brochure; RDW variant ACDGEAX0 confirms |
| vp-676e93 | Lexus RX 450h Business Line, 2019 — **HEV** | 2715 | 640 | Lexus UK tech-spec sheet; RDW variant GYL25(W) confirms |
| vp-d59965 | Lexus RX 450h Executive Line, 2019 — **HEV** | 2715 | 640 | same type approval, equipment trim only |
| vp-711639 | Subaru Forester 2.0i e-Boxer, 2019 — **HEV** | 2185 | 554 | RDW variant SKE; UK towing guide quoting Subaru agrees |
| vp-c8872f | Toyota Camry Hybrid Active (EU), 2021 — **HEV** | 2100 | 505 | RDW variant AXVH71(E), 375 registrations, one figure |
| vp-02597b | Mazda MX-30 First Edition, 2020 — BEV | 2119 | 499 | RDW variant 1WB; Mazda-derived spec page agrees |
| vp-c33f36 | Mazda MX-30 Luxury, 2020 — BEV | 2119 | 499 | same single variant 1WB |
| vp-361605 | Audi e-tron Sportback 50 quattro Business Edition Plus, 2020 — BEV | 3040 | 665 | RDW variant SBM2RQ1, 792 registrations |
| vp-941da3 | Audi e-tron Sportback 50 quattro Business Edition, 2020 — BEV | 3040 | 665 | same variant SBM2RQ1 |
| vp-c0d302 | Tesla Model Y Long Range AWD (EU), 2021 — BEV | 2371 | 421 | RDW variant Y5CD |
| vp-cd6621 | Tesla Model Y Performance AWD (EU), 2021 — BEV | 2371 | 421 | same EU type approval |

All twelve pass the payload sanity band (350–700 kg) and gross > curb.

### Grouped rows, and where a shared figure was *not* spread

Five make/model/year pairs appear twice as equipment trims. In each case the
two rows were checked to be one type-approval variant before the figure was
shared: Audi e-tron Sportback 50 (SBM2RQ1), Lexus RX 450h (GYL25(W)), Mazda
MX-30 (1WB), Tesla Model Y (Y5CD). Deliberately **not** spread:

- **Audi e-tron Sportback 55** (SBM1JQ1, 3150–3170 kg) is a different mass
  class from the 50 and was excluded by filtering on the 50's unladen mass.
- **Lexus RX 450h L** (GYL26, 2840 kg) is the 7-seat long body — a different
  figure, excluded.
- **BMW 530e xDrive** (variant 31CG, 2610 kg) is not the row's rear-drive
  530e; the mass filter excludes it.
- **Honda e** has two distinct RDW type approvals (1855 and 1870 kg) — see
  refusals.

## Refusals (42 rows)

### The 33 FASTSim rows: a structural refusal

`data_tier=fastsim` rows carry `cargo_kg = 136` (or 136.8) and a
`market` of "US/EU (FASTSim)". Their `curb_mass_kg` is the **simulation test
mass with that 136 kg of occupant/cargo already inside it**, not a curb mass:

- The lb-derived ones land on round simulation values — 1814.4 kg is exactly
  4000 lb and is stated identically for the Ford C-MAX HEV, Hyundai Sonata
  PHEV *and* Toyota RAV4 Hybrid LE, three vehicles of different real mass.
- The f3-resolved European ones diverge from the registered unladen mass by
  90–260 kg in the same direction: Cupra Born 1927 vs RDW 1711–1738; BMW iX
  xDrive40 2600 vs 2340; Polestar 2 2173 vs 1969–2088; Volvo XC40 Recharge
  twin 2132 vs 1975; Volvo C40 2237 vs 1995; BYD Atto 3 1813 vs 1725.

Pairing a real type-approval gross mass with such a figure understates
Mкор = gross − curb by roughly the cargo allowance — on a car with ~500 kg of
real useful load that is a ~25–30% error in q, and it would pass the payload
floor while doing it, so nothing downstream would catch it. `market` being
undetermined between US and EU compounds it, since GVWR and the EU
technically-permissible mass are different numbers for the same car. Refused
as a class. This is the same reasoning the pilot used to refuse
`vp-c23578` (F-150 Lightning), now generalised with the cargo column as
direct evidence.

Rows refused on this basis: vp-262cad, vp-118a2d, vp-283497, vp-89314d,
vp-4ebdad, vp-648f3f, vp-b5fcc9, vp-24b2fe, vp-5c56cb, vp-9837d6, vp-c9f0f7,
vp-9fe796, vp-da4b78, vp-026c10, vp-141613, vp-719c4c, vp-df71be, vp-cbeffa,
vp-76cf5c, vp-c23578, vp-1e7c52, vp-fc6e18, vp-35fe17, vp-662c7e, vp-ef0d12,
vp-1b12e8, vp-1607a0, vp-3b2367, vp-ad3e77, vp-4b8ae9, vp-d16b2b, vp-1c9c27,
vp-777972, vp-9402d9.

Two further notes from the searching that was done anyway: the **BYD Dolphin
Active** (44.9 kWh) has no RDW type approval at all — only the 60 kWh
Comfort/Design (leeg 1633 / max 2068), a different variant — and the
**VinFast VF e34** is not registered in the Netherlands (only VF 8 and VF 6),
so both would have been refused for want of the exact variant regardless.

### Individually refused rows

| prototype_id | vehicle | what was searched | why refused |
|---|---|---|---|
| vp-a33823 | Honda e Advance, 2020 | RDW (merk HONDA), spec aggregators quoting Honda figures | RDW holds **two** Honda e type approvals: `uitvoering` 2/4 at leeg 1495 / max **1870** kg and `uitvoering` 3 at leeg 1488 / max **1855** kg. The row's curb (1525 kg) matches neither, so it cannot arbitrate. Worse, the sources contradict each other on direction: the Advance is the heavier 113 kW car and should take the higher figure, but the spec pages found pair "Advance" with the *lower* 1855 kg. Two candidate values, no way to tell which is this row's — refused. |
| vp-05024e | Honda e "specs", 2020 | same | Same ambiguity, and this row's `variant_trim` is the placeholder "specs", so it does not even name a trim to resolve. Refused. |
| vp-696d02 | Ferrari SF90 Stradale, 2020 | RDW (variant HF: leeg 1720/1750, max 2085 kg) | The gross mass is available and consistent (2085 kg for both SF90 Stradale uitvoeringen), but the row's `curb_mass_kg` of 1570 kg is Ferrari's **dry weight** (*peso a secco*, no fluids) — 150–180 kg below the registered unladen mass. Pairing 2085 with 1570 yields an implied useful load of 515 kg where the real figure is ~335–365 kg: a ~50% overstatement of Mкор that would pass the payload band unnoticed. Filling this would have taken M1+hybrid to the 8-pair curve threshold; refused anyway, because a curve fitted through a wrong point is worse than no curve. |
| vp-37ede4 | VW Golf 1.5 eTSI (Mk8 launch, mild hybrid), 2019 | RDW, filtered to merk VOLKSWAGEN, handelsbenaming GOLF, cilinderinhoud 1498, first admission 2019-10 to 2021-06 | This row needs *both* masses, so an RDW pair would have been self-consistent and would have created the eighth hybrid point. But 1498 cc Mk8 Golfs spread across at least nine type-approval variants with unladen masses 1206–1325 kg and permissible masses 1800–1970 kg, and the register does not separate the 48V eTSI from the plain 1.5 TSI or the 110 kW from the 96 kW output at this level. No way to pin the exact variant — refused for variant ambiguity. |
| vp-534f66 | Chevrolet Chevette 1.6L OHC, 1976 (US) | — | Caught by the batch selector only because "CHEVETTE" contains the letters "EV"; it is an ICE row and out of scope for this electrified batch. Left for a later batch. |
| vp-3363f3 | Solaris Urbino 12 electric, 2023 | — | Already searched and refused in the pilot pass above (Solaris's own press kit publishes no kerb mass for this variant). Unchanged. |
| vp-eaba4e | Škoda 32Tr SOR trolleybus, 2018 | — | Already searched and refused in the pilot pass above. Unchanged. |
| vp-d62ca0 | Mercedes-Benz eActros 300, 2021 | — | Already searched and refused in the pilot pass above. Unchanged. |

## Verification output

```
$ cd Data/Vehicle && python3 build/build_vehicle_prototypes.py
Web-sourced masses: filled 13, refused 0, unmatched 0
wrote .../vehicle_prototypes.csv: 497 rows
wrote .../vehicle_prototypes_provenance.csv: 701 provenance rows (40 blanked)

$ python3 build/build_q_dataset.py
312 q points -> .../q-dataset.json

$ python3 -m unittest discover -s build -p 'test_*.py'
Ran 120 tests in 2.201s
OK

$ cd ../.. && npm run check
... node --test: 27/27 pass ...
check-docs: all documentation files pass
✓ built in 255ms
```

("filled 13" counts the pilot's Ford Ranger row plus this batch's 12.)

## Assessment for the remaining 65 rows (mostly 2012–2016 mainstream petrol cars)

Cost this pass was roughly **3–4 lookups per confirmed value** — an order of
magnitude better than the pilot's 25–30, almost entirely because RDW answers
"what is the type-approval mass, and does it depend on trim?" in one query
and confirms the variant by handing back the row's own curb mass.

Expect a **better** rate on the remaining 65 rows than the 22% here, and
possibly much better, for one reason and with one caveat:

- **Reason:** this batch's refusal rate was dominated by the 33 FASTSim rows,
  a data-provenance problem rather than a research problem. If the remaining
  rows are mainstream 2012–2016 cars from the enrichment/manual tiers rather
  than FASTSim, that structural drag disappears, and the enrichment rows in
  this batch converted at 12 of 15 (80%).
- **Caveat:** RDW only helps for cars sold in the Netherlands. A 2012–2016
  US-market row gets no benefit and falls back to per-vehicle searching at
  the pilot's cost. Check `market` before budgeting.

**Recommended before the next batch:** decide what to do with the FASTSim
tier as a whole. Thirty-three rows in this batch alone cannot yield a q point
without either (a) subtracting the documented `cargo_kg` from `curb_mass_kg`
to recover a real curb mass — a build-step change, not a research task — or
(b) accepting that the tier is for simulation parameters and excluding it
from the q chart the way the three archetypes already were. Researching them
one at a time will keep producing refusals whatever the effort spent.

## Batch 2 — electrified and modern petrol rows

Method: RDW open data (Netherlands national vehicle register, dataset
`m9d7-ebf2`) publishes both the EU type-approval permissible maximum mass and
the unladen mass. The second field confirms the variant, because it returns the
row's own curb figure. Cost fell from 25-30 lookups per confirmed value in
batch 1 to 3-4 here.

### Filled

| row | vehicle | curb | gross | source basis |
|---|---|---|---|---|
| vp-37ede4 | VW Golf VIII 1.5 eTSI ACT 150 PS DSG | 1380 | 1880 | ADAC Autokatalog (build 01/20-09/20, 110 kW, 1498 cc, 7-speed DSG); reproduces the table's own 500 kg Zuladung; corroborated by angurten.de and RDW variant ACDFYAX0 |
| vp-c6971d | Mercedes E 200 (W213 FL) | 1600 | 2360 | RDW variant U08KT0, 336 registrations, one figure across 11 uitvoering codes; automoli W213-FL table agrees |
| vp-26f825 | Mercedes E 300 (W213 FL) | 1630 | 2390 | RDW variant U08LT0, 1991 cc sedan only; automoli agrees |
| vp-6f90d7 | BMW M550i xDrive (G30 LCI) | 1890 | 2560 | RDW variant 11BK, 58 registrations, 4395 cc sedan, single figure |
| vp-a9d57a | Ford Puma 1.0 EB 125 ST-Line | 1205 | 1760 | angurten.de; RDW B7JB12X agrees |
| vp-d40e97 | Ford Puma 1.0 EB 125 Titanium X | 1205 | 1760 | same type approval, equipment level differs only |
| vp-b65958 | Škoda Octavia 1.5 TSI 110 kW | 1338 | 1888 | RDW variant ACDHFAX0 at matching unladen mass; within Škoda's published 1850-1910 kg range |

The Golf VIII took M1/hybrid from 7 to 8 distinct curb/gross pairs, reaching
`MIN_CURVE_POINTS` — that grouping can now be fitted a regression curve.

The two Mercedes rows imply a 760 kg useful load, above the 350-700 kg rule of
thumb used as a sanity check. Judged genuine: Mercedes' own stated Zuladung is
735 kg and the row's curb figure sits 25 kg from the type-approval unladen
basis. Recorded here rather than silently accepted.

### Refused

| vehicle | reason |
|---|---|
| Ferrari F8 Spider, Lamborghini Aventador S, Lamborghini Urus | the row's curb figure is the manufacturer's *dry* weight, 150-180 kg below registered unladen mass — the same defect that disqualified the Ferrari SF90 in batch 1. Refused on basis, not for want of a gross figure |
| Porsche Panamera GTS, Panamera Turbo S | RDW returns two variants at the row's unladen mass with different maxima (2665 vs 2585; 2560 vs 2585). Ambiguous |
| VW Tiguan 1.5 TSI | permissible mass spreads 2140-2330 kg at the row's exact unladen mass. Ambiguous |
| Nissan Juke, Kia Sportage, Peugeot 208 | no usable RDW population at the row's curb mass |
| Renault Clio TCe 90 | 1675 vs 1680 across several variants, unresolved |

### Not attempted

Roughly 30 rows remain: the 1964-1984 classics (Trabant, 2CV, R4, R5, Mini,
Kadett C, Golf Mk1, Beetle, Escort Mk1, Lada 2101, Corolla E30, Sunny B310,
Civic 1st generation, Chevette) and US-market moderns needing GVWR (Honda
Civic, CR-V, Accord, RAV4, Tucson, Santa Fe, Silverado). RDW does not cover
either group well: the classics predate the register's useful coverage and the
US rows are outside it entirely.

## Batch 3 — manufacturer documents

A set of 21 locally-supplied manufacturer PDFs (plus one Russian-language
workshop manual) in `.../Transport/Автомобили/by brands` was checked against
every candidate row for Ferrari, Lamborghini, Chevrolet, Honda, Toyota,
Trabant and Lada that is still missing a mass. **Result: 0 rows filled, 18
refused.** No row in `sources/web_sourced_masses.csv` changed, so the dataset
is unchanged at 497 prototypes and 319 q points.

The outcome is not for want of documents. It is that the documents in hand
are, without exception, either (a) press releases and brochures that publish
a *dry* weight and no permissible mass at all, or (b) correctly-dated
documents for a *different market's* version of the nameplate, whose
drivetrain and rated masses differ from the row's.

### The supercar rows — no kerb weight is stated anywhere

The valuable case was to replace the manufacturer dry weight in
`curb_mass_kg` with a stated kerb weight and add the gross. **Not one of the
supercar documents states a kerb weight, and not one states a permissible
maximum mass.** They state dry weight only, which is the figure already in
the dataset and already known to be the wrong basis.

| Row | Document chosen | Page | What it states | Verdict |
|---|---|---|---|---|
| Ferrari F8 Spider 2020 (vp-46a79e) | `Doc7477.pdf` (Ferrari press kit, 09/2019) | p. 7, "Dimensions and weight" | `Dry weight** 1400 kg` — dry basis only | Refused: no kerb weight, no gross. The 1400 kg confirms the row's `curb_mass_kg` is dry weight |
| Ferrari SF90 Stradale 2020 (vp-696d02) | — | — | — | Refused: no document covers the SF90 |
| Lamborghini Aventador S 2017 (vp-f48f34) | `2018-Lamborghini_Aventador-S_optionlist.pdf` | p. 10 | `Dry weight 1575 Kg` (coupé) / `1625 Kg` (Roadster) | Refused: dry basis only, no gross. The 1575 kg confirms the row's curb is the coupé dry weight |
| Lamborghini Urus 2018 (vp-f50917) | `URUS2017.pdf` (press release, 12/2017) | p. 11, "Dimensions and weight" | `Curb weight < 2,200 kg` — an upper bound, not a value | Refused: a "less than" bound is not a stated figure, and no gross appears |

Why the other Ferrari and Lamborghini documents were rejected rather than used:

- `cs_reveal_ferrari_f8_tributo_gbr.pdf` — the **Tributo** coupé, not the
  **Spider** the row names. Different body style. (`Doc7477.pdf` turned out
  to be the Spider press kit and was used in its place — for identification
  only, as it carries no usable figure.)
- `FerrariF8.pdf` — despite the filename, this is a **Ferrari North America
  service bulletin** ("2020 Technical Documentation Update #5", July 2020)
  announcing a technical-documentation release. It contains no vehicle
  specifications of any kind.
- `Lamborghini_int Aventador_2014-lp700.pdf` — Aventador **LP700-4
  Roadster, 2014**. Different variant (LP700 not S) and body (Roadster);
  states `Dry weight 1625 kg`, which is the Roadster dry figure, not the
  row's coupé.
- `Lamborghini URUS_PHEV_DIGITAL_BROCHURE_EN_2026_WCAG.pdf` and
  `Brochure_Lamborghini_Urus PA2_ENG_WCAG.pdf` — the **Urus SE / SE
  Performante PHEV, model year 2026** (812 PS combined, `DRY WEIGHT 2.478
  kg`). Different generation and a different powertrain from the 2018 V8.
- `URUS PERFORMANTE_DIGITAL_BROCHURE_WCAG_EN_2024.pdf` — **Urus
  Performante, 2024** (`dry weight 2157 kg`). A different variant and model
  year; the Performante is a deliberately lightened car (-47 kg on its
  predecessor), so its figures do not describe the base 2018 Urus.
- `Lamborghini_int Huracan LP 610-4 Avio_2017.pdf` — Huracán. No Huracán
  row exists in the dataset.

### US-market rows — the documents are for other markets

Each of these rows now has a *correctly dated* document, which removes the
model-year objection but exposes a market objection that is just as fatal.

| Row | Document chosen | Page | What it states | Verdict |
|---|---|---|---|---|
| Honda CR-V EX-L FWD 2023, USA (vp-265046), curb 1599 kg | `2023_SPECS_CRV_updated-1.pdf` | p. 4 | EX-L: curb 1656 kg, GVWR 2175 kg | Refused: **Canadian** spec sheet (footnote 8 cites the Government of Canada 5-cycle method). Its trim ladder is LX-B 2WD / LX 2WD / LX-B / LX / Sport-B / Sport / EX-L / Touring Hybrid — the Canadian EX-L is **AWD only**, curb 1656 kg, 57 kg above the row's FWD 1599 kg. The document's two FWD columns (curb 1583, GVWR 2160) are LX trims, not EX-L. GVWR here is drivetrain-dependent (2160 FWD vs 2175 AWD), so it is not a chassis constant that could be borrowed across. The document simply does not state a GVWR for a FWD EX-L, because Canada did not sell one |
| Toyota RAV4 LE FWD 2022, USA (vp-9fae94), curb 1529 kg | `2022-Toyota-RAV4-UK.pdf` | p. 42, "Weights & towing capacity" | Gross vehicle weight 2130 kg (FWD) / 2215 kg (AWD-i) | Refused: the **UK** range is 2.5 Petrol **Hybrid** only (both columns on the performance page are "2.5 Petrol Hybrid Automatic"). The row is the US 2.5 L non-hybrid LE. Different powertrain; the hybrid carries a battery and a rear motor the row's car does not have |
| Chevrolet Silverado 1500 Crew Cab 6'6", 5.3 V8, 2019, USA (vp-30d053), curb 2224 kg | `2019_silverado_classic.pdf` | p. 6 | 2WD GVWR 7000 lb (3175 kg); 4x4 7200 lb (3266 kg) | Refused: this is the **Silverado 1500 LD** — the carried-over K2 "Legacy Duty" truck sold alongside the new T1 in 2019 — and its MODEL OVERVIEW offers **4-door Double Cab only** (wheelbase 3645 mm / 143.5 in). The row is a T1 **Crew Cab** (147.4 in wheelbase). Also a Canadian brochure (mychevrolet.ca, nrcan.gc.ca). The 7000 lb figure is a near-perfect trap: it is also the US T1 Crew Cab's GVWR, but the document does not state it *for that truck*, and one photo caption in the brochure even mislabels the Double Cab as a "Crew Cab" |
| (same row, alternative) | `Blackwells_GMSV_Chevrolet_Silverado_Model_Spec_0821.pdf` | p. 2 | LT Trail Boss: kerb 2469 kg, GVM 3221 kg; LTZ: kerb 2540 kg, GVM 3300 kg | Refused: **New Zealand GMSV** right-hand-drive remanufactured trucks, 6.2 L V8 / 10-speed, dated 08/2021. Different market, engine, transmission and model year; the RHD conversion itself adds mass, which is why kerb is 2469 kg against the row's 2224 kg |
| Honda Civic Sedan Sport 2.0 2023, USA (vp-0166fb), curb 1331 kg | `Honda1370HMECivic5drBro2020_SE.pdf` | p. 22 | Tjänstevikt 1352-1446 kg, Totalvikt 1775 kg | Refused: **Swedish** brochure for the **10th-generation (FK) 5-door hatchback**, model year 2020. The row is an 11th-generation (FE) **sedan**, US market, 2023. Generation, body style, market and model year all differ |
| (same row, alternative) | `Honda-Civic-2016-UK.pdf` | p. 22 | Maximum permissible weight 1680-1870 kg across trims | Refused: **UK 2016**, 10th generation. Wrong generation, market and year |
| Honda Accord LX 2023, USA (vp-1f02fe) | — | — | — | Refused: no document covers the Accord |

### Toyota RAV4 — the two remaining RAV4 documents

- `pricelist.pdf` is **Toyota Sweden's current RAV4 price list and brochure**
  ("RAV4 – prislista", prices in SEK, saved 2026-08-14). It covers the RAV4
  Hybrid (`Totalvikt 2230 kg`) and the Plug-in Hybrid (`Totalvikt 2570 kg`).
  Swedish market, hybrid and PHEV only, and a model year four years after the
  row. Refused.
- `RAV4_pf_web_1311_1.pdf` is a **Swedish RAV4 price list dated 11/2013**
  (Tjänstevikt 1645-1810, Totalvikt 2100-2240). Refused on model year: this
  is the XA40 generation, two generations before the row's 2022 XA50.

Neither is the US-market XA50 the row describes.

### Classics

| Row | Document | Verdict |
|---|---|---|
| Trabant 601 saloon, 1964 (vp-d1b9e0), curb 615 kg | `Trabant-1.1-Universal-444-Edition-1994-GER.pdf`, p. 3 "Technische Daten" | Refused. The document is the **Trabant 1.1 Universal "444 Edition", 1994**: a four-cylinder four-stroke 1043 cm³ VW engine, 30 kW, estate body — `Leermasse, fahrfertig 735 kg`, `Nutzmasse 385 kg`, `Zul. Gesamtmasse 1120 kg`. The row is the 601 **saloon** of 1964 with the 595 cm³ two-stroke twin. Different model, engine, body style and era; the recorded figures are logged here only so the document does not have to be re-opened |
| Lada (VAZ) 2101, 1970 (vp-4ae922), curb 945 kg | `vaz210102011013_rukovodstvo_po_ustroistvu_i_ekspluatatsii.zip` | Refused. The archive holds a single Word document, "Руководство по устройству и эксплуатации ВАЗ-2101-02-011-013". Converted in full (≈193 000 characters) and searched: it is a *construction and operation* manual — chapter by chapter on the engine, carburettor, clutch, gearbox, axles, suspension, brakes, electrics and body — and contains **no technical-characteristics table and no vehicle mass of any kind**. The strings `снаряж`, `Полная`, `Собственн`, `грузоподъем` and `Техническая характеристика` do not occur anywhere in it |
| Toyota Corolla E30 1979, Honda Civic 1st gen 1972, Chevrolet Chevette 1976 | — | Refused: no document covers them |

### Rows with no candidate document at all

Honda e (vp-a33823, vp-05024e), Honda CB 550 SC, Honda CLR CityFly, Honda V30
Magna, Toyota Hilux Single Cab 2019 (South Africa), Toyota Land Cruiser 300
2021. Nothing in the supplied set addresses these.

### What would actually close these rows

- **Supercars.** A dry weight is published freely; a kerb weight and a
  permissible maximum are not. What is needed is a type-approval or
  registration record — a Certificate of Conformity, an Italian *carta di
  circolazione*, or a national register entry — which states *massa in ordine
  di marcia* and *massa massima a pieno carico* together. A brochure will not
  supply them.
- **US-market rows.** The figure required is the GVWR from the door-jamb
  certification label, or a US-market press/spec kit. Canadian and UK
  brochures for the same nameplate are genuinely different vehicles here, not
  merely different presentations of one.

## Batch 4 — classic-car manuals

Nine owner's/workshop manuals for 1946-1982 cars, freshly renamed under the
conventions in the `by brands` folder's own `README.md`, checked against the
ten classic rows that still lack a mass. **0 of 10 filled, 10 refused.**

Year matching was relaxed as instructed — a manual from anywhere inside an
unchanged production run describes the same car — and it is *not* what
blocked these rows. Every refusal below is a mismatch of engine variant,
body style, market, or a document that states no weights at all.

| Row | Document | What it states, and where | Verdict |
|---|---|---|---|
| `vp-d8ce2d` VW Golf Mk1 1.1L 50 PS **5-door**, 1974, curb 750 kg | `Volkswagen_Golf-Mk1_manual.pdf`, p. 92 ("Weights") | 2-door 1.1: unladen 750 / permissible load 450 / **permissible total 1200 kg**. 4-door 1.1: unladen 775 / load 425 / **total 1250 kg**. Unladen weight is stated "ready for road, full tank"; the footnote adds "These figures are for Germany only" | Refused. The row is the 5-door (VW's "4-door" column) but its curb mass, 750 kg, is exactly the **2-door** figure — the manual gives 775 kg for the 4-door. Pairing the 4-door's 1250 kg against a 2-door curb yields a 500 kg useful load, above both the ~450 kg ceiling for a car this size and the manual's own stated 425 kg. Pairing the 2-door's 1200 kg is self-consistent (450 kg exactly) but describes the wrong body. Either way two bases would be mixed. Independently, this January 1979 edition documents a **later body**: p. 93 gives length 3815/3835 mm against the row's 3708 mm, so the relaxation does not reach back to 1974 for dimensions either |
| `vp-f0ebfe` VW Beetle (Type 1) 1600 US-spec sedan, 1974, curb 816 kg | `Volkswagen_Beetle_1946_owners-manual.pdf`, pp. 7-8 (English and German technical-data pages) | Typ 11: `Eigengewicht 695 kg`, `Gesamtgewicht 1105 kg` (English page: unladen 1600 lb, laden 2440 lb). Typ 51: 730 / 1140 kg | Refused. The 1946 Typ 11 is the 1131 cm³, 25 PS split-window car; the row is the 1974 US-market 1600 (1641 cm³, 34 kW) with impact bumpers. Not one unchanged production run — the row's curb mass is 121 kg (17%) above the document's. The trap is precise: 1105 − 816 = 289 kg would clear the payload floor and produce a plausible-looking q from two different cars |
| `vp-09a7e4` Citroën 2CV6 (602 cm³), 1970, curb 600 kg | `Citroen_2CV-375-425cc_1960_SE-FR_service-manual.pdf` | Swedish 2CV club service manual, Stockholm, August 1960 (foreword, p. 5), built from Citroën's French workshop plates. Checked front and back: repair procedures, lubrication charts and chassis jigs — **no vehicle technical-data table and no weights** | Refused twice over: the 602 cm³ 2CV6 did not exist in 1960 (this book covers the 375/425 cm³ cars), and the document states no mass in any case |
| `vp-179447` Renault 4 **GTL**, 1984, curb 720 kg | `Renault_R4_1969-1977_NL_vraagbaak.pdf`, "MATEN EN GEWICHTEN" | `GEWICHTEN (kg)`: Berline/Luxe/Grand Luxe/Export 630 / 980; L en Safari 665 / 1095; **TL 695 / 1095** (`rijklaar gewicht` and `max. toelaatbaar totaalgewicht`). A footnote warns the factory figure is not the one on the registration document | Refused. The book covers 1969-1977 (Berline, Luxe, Grand Luxe, Export, L, TL, Safari) with the 845 cm³ engine. The GTL is the later 1108 cm³ car, introduced after this range ends; it appears in no column. Its 720 kg curb also exceeds every figure in the table |
| `vp-b100e8` Ford Escort Mk1 1100 Saloon, 1968, UK, curb 805 kg | `Ford_Escort-Mk1_DE_owners-manual.pdf`, pp. 43-45 + 48 ("Technische Daten") | Engine data (1098/1298 cm³, 44-72 PS), `ABMESSUNGEN` (Typ PKW: Radstand 2400, Spur vorn 1257 / hinten 1282, Breite 1570, Höhe 1408, Länge 4052 mm), `FÜLLMENGEN`, `LAMPENTABELLE`, `REIFENGRÖSSE/REIFENDRUCK`, `DACHLASTEN 50 kg` | Refused. The Technische Daten section contains **no `Leergewicht` and no `zulässiges Gesamtgewicht`** — the strings do not occur anywhere in the manual. The only mass it states is the 50 kg roof load. (Dimensions are present but are German-market and partly disagree with the row: 4052 mm against 3978 mm. `track_width_mm` was not filled either: the car has different front and rear tracks and the column does not say which it holds) |
| `vp-d25ace` Datsun Sunny B310 1.4 A14, **JDM**, 1977, curb 907 kg | `Datsun_Sunny_haynes-workshop-manual.pdf`, pp. 9 and 224 ("Capacities, dimensions and weights") | Kerb weights only, by body and engine: 4-door saloon A12 840 kg / A14 875 kg; 2-door 830-875 kg; coupé 890 kg; estate 925 kg; North American sedan 1270 kg, hatchback 1295 kg, wagon 1260 kg; vans 845-895 kg | Refused. The row needs a **gross** mass and the manual states none — the words "gross vehicle weight" do not appear in it. Its coverage is "Aug '78 to May '82, all models", i.e. UK and North American B310s; the row is the JDM car, and the North American figures (1270 kg for a sedan) show how far market spec moves the mass |
| `vp-534f66` Chevrolet Chevette 1.6 OHC, North America, 1976, curb 880 kg | `Vauxhall_Chevette_haynes-workshop-manual.pdf`, p. 8 | `Kerb weight (nominal)` 826-885 kg across eleven body/trim combinations, and `Maximum gross vehicle weight`: hatchback and saloon **1314 kg**, estate 1335 kg, van 1350 kg | Refused. This is the **British** T-car: "all versions of the Chevette Hatchback, Saloon & Estate and Bedford Chevanne with 1256 cc engine". The row is the North American Chevrolet Chevette with the 1.6 L OHC engine — a different make, market and engine on a shared platform. This is the one document in the batch that does publish a GVW, which makes it the most dangerous to borrow |
| `vp-ee27ff` Opel Kadett C 1.2 52 PS, 1973, curb 785 kg | `Opel_Kadett-C-Coupe-1.2_manual.pdf` | All 50 pages rendered and OCR'd: the file is a **photograph album** (a BilGalleri.dk gallery of one Danish-registered Kadett C coupé), not a manual. No text, no technical data | Refused. The file is misnamed at source; it should carry the folder's `_NO-SPECS` suffix so nobody opens it again |
| `vp-4ae922` Lada (VAZ) 2101, 1970 | `Lada-VAZ_2101-2011-2013_manual_RU_NO-SPEC-TABLE.zip` | — | Refused in batch 3 and not re-opened; the filename records the finding |
| `vp-d1b9e0` Trabant 601 saloon, 1964 | (only the 1994 Trabant 1.1 Universal brochure exists) | — | Refused in batch 3; different model, engine, body and era |

`Volkswagen_New-Beetle_2009_INT_brochure.pdf` was not used: no dataset row
describes a New Beetle. (Three rows — `vp-b8a9e9`, `vp-4c07e7`, `vp-e348be` —
carry Kaggle-sourced "Volkswagen Beetle, 1 generation, 1946" masses of
1160-1290 kg against 1620-1630 kg gross, which are New Beetle figures under a
mislabelled generation year. They already hold complete, `ok`-status pairs and
were outside this pass's scope; flagged here as a lead worth checking.)

### Non-mass fields

The instruction to harvest ratios, tyre codes and wheel radii while these
manuals were open produced nothing writable, for two reasons. First, the
documents: the only gearbox and final-drive ratios in the batch are in the
Renault vraagbaak (I 3,82; II 2,235; III 1,458; IV 1,027; final drive 4,125)
and they belong to the 845 cm³ types, not the row's GTL; the Golf and Escort
owner's manuals carry no ratios at all, and their tyre entries list three
alternative sizes for one engine (Golf 1.1: 145 SR 13, 155 SR 13, 175/70 SR
13) with no way to say which the row's car wore. Second, the machinery: the
`web_sourced_masses.csv` import applies `WEB_SOURCED_MASS_FIELDS` only —
curb, gross and their min/max — so a non-mass value could not be imported
without new build code, which this pass was told not to write.

### Integrity fix — the dry-weight supercar rows

`mass_pair_status` gained a fourth value, `dry-weight-basis`, applied to
`vp-46a79e` (Ferrari F8 Spider, 1400 kg), `vp-696d02` (Ferrari SF90 Stradale,
1570 kg), `vp-f48f34` (Lamborghini Aventador S, 1575 kg) and `vp-f50917`
(Lamborghini Urus, 2200 kg). Batch 3 confirmed against the manufacturer
documents that each of these `curb_mass_kg` values is the maker's **dry
weight** — 150-180 kg below registered unladen mass — and that neither maker
publishes a permissible maximum for these cars.

None of the four carries a gross mass, so none produces a q point and the
chart does not move. The marking exists for the future: pairing a correctly
sourced gross mass against a dry weight would divide q by a denominator
inflated by the missing fluids, and the result would still clear the payload
floor — invisible, exactly the failure mode `simulation-test-mass` guards
against. Like that ruling it is made from the row's provenance, so it holds
whether or not a gross mass is present, and `build_q_dataset.py` refuses the
rows and reports them in `q-dataset.json` under `excludedDryWeightBasis`.

## Verification output (batch 4)

```
$ cd Data/Vehicle && python3 build/build_vehicle_prototypes.py
Web-sourced masses: filled 21, refused 0, unmatched 0
wrote .../vehicle_prototypes.csv: 497 rows
wrote .../vehicle_prototypes_provenance.csv: 707 provenance rows (40 blanked)

$ python3 build/build_q_dataset.py
62 FASTSim row(s) excluded: curb_mass_kg is a rounded simulation test mass, not a curb mass
4 row(s) excluded: curb_mass_kg is a manufacturer dry weight, not a curb mass
  vp-46a79e Ferrari F8 Spider: dry weight 1400.0 kg
  vp-696d02 Ferrari SF90 Stradale: dry weight 1570.0 kg
  vp-f48f34 Lamborghini Aventador: dry weight 1575.0 kg
  vp-f50917 Lamborghini Urus: dry weight 2200.0 kg
319 q points -> .../q-dataset.json

$ python3 -m unittest discover -s build -p 'test_*.py'
Ran 130 tests in 2.378s
OK

$ cd ../.. && npm run check
# tests 27
# pass 27
# fail 0
check-docs: all documentation files pass
✓ built in 296ms
```

Row count: 497 (unchanged). q points: 319 → 319 (no fills; the four
dry-weight rows never contributed a point).

---

# Batch 5 — range-extended vehicles (REEV / REx)

Branch `data/web-sourced-masses`. This batch **adds rows** rather than filling
gaps in existing ones. Before it, the dataset held exactly one range-extended
vehicle — `vp-262cad`, FASTSim's "2016 BMW i3 REx PHEV" — whose curb mass of
1530.9 kg is exactly 3375 lb, a simulation test mass, and which carries no
gross mass at all. It is barred from the tare-coefficient chart
(`mass_pair_status = simulation-test-mass`) and remains so; nothing about it
was changed. The range-extender architecture was therefore absent from the
analysis entirely.

**Result: 5 REEV rows added, all 5 producing a q point.** All five are
`powertrain_detail = REx`, which the build promotes to `powertrain = PHEV`.

## Summary

| Metric | Before | After |
|---|---|---|
| Row count | 497 | 502 |
| q points | 319 | 324 |
| REEV rows in dataset | 1 | 6 |
| REEV rows producing a q point | 0 | 5 |

## The five records

| Vehicle | curb (kg) | gross (kg) | payload | q | RDW variant / approval |
|---|---|---|---|---|---|
| BMW i3 REx 60 Ah (2015) | 1290 | 1730 | 440 | **2.932** | variant `1Z41`, 647 cc |
| BMW i3 REx 94 Ah (2017) | 1340 | 1760 | 420 | **3.190** | variant `1Z81`, 647 cc |
| Opel Ampera (2012) | 1632 | 2000 | 368 | **4.435** | `e13*2007/46*1159*01` |
| Mazda MX-30 R-EV (2023) | 1753 | 2251 | 498 | **3.520** | variant `6WJ` |
| LEVC TX (2018) | 2205 | 2900 | 695 | **3.173** | variant `A1`, `e11*2007/46*4190*00/*01` |

Four of the five sit in a tight 2.93–3.52 band around the M1 median of ~3 —
a REEV carries both a battery and an engine, so a heavy curb against a normal
homologated payload is exactly the expected shape. The Ampera is the outlier
at 4.44 and the reason is real, not an error: the Voltec car is a documented
**4-seat** design with a T-shaped battery through the centre tunnel, and its
368 kg useful load is genuinely small for a 4.5 m car.

## Source and basis

Every one of the five came from **RDW open data, dataset `m9d7-ebf2`
("Gekentekende voertuigen")**, the Netherlands national vehicle register —
the same method that carried batches 1–4. All were sold in the Netherlands.

- `curb_mass_kg` = RDW `massa_ledig_voertuig`, the register's **unladen mass**
  (vehicle with fluids, **without** the 75 kg driver).
- `gross_mass_kg` = RDW `technische_max_massa_voertuig`, the EU type-approval
  **technically permissible maximum laden mass**.
- Both come from the same registration record, so curb and gross are on one
  basis, and that basis matches the rest of this file.
- RDW also publishes `massa_rijklaar` = `massa_ledig` + 100 kg, the Dutch
  running-order convention. **It is not used here.** It matters below.

Two structural properties of the register did the variant-matching work:

1. **The range extender is visible in the record.** A REEV carries an engine,
   so it has a `cilinderinhoud` where the battery-only sibling has none. That
   one field cleanly separated the i3 REx from the i3 BEV (647 cc, 2 cyl) and
   the MX-30 R-EV from the MX-30 BEV.
2. **Grouping by type-approval variant shows whether the figure is
   trim-dependent.** Every fill below was made only where one variant carried
   one permissible mass across every `uitvoering` code.

### Per vehicle

**BMW i3 REx, 60 Ah — variant `1Z41`, 1290 / 1730 kg.**
Searched RDW for `merk = BMW I`, `handelsbenaming = I3`, `cilinderinhoud > 0`,
grouped by variant. 590 registrations match `1Z41` at 1290 kg unladen and
**all 590** state 1730 kg, across three extensions of one approval
(`e1*2007/46*1213*02/*03/*04`). First admissions 2013-10-29 to 2017-05-30.
[query](https://opendata.rdw.nl/resource/m9d7-ebf2.json?merk=BMW+I&handelsbenaming=I3&cilinderinhoud=647&variant=1Z41&massa_ledig_voertuig=1290)

**BMW i3 REx, 94 Ah — variant `1Z81`, 1340 / 1760 kg.**
67 registrations, all stating 1760 kg, first admissions 2016-08-22 to
2018-12-17 (`e1*2007/46*1213*05/*06`). The 60 Ah and 94 Ah cars are separated
by date: `1Z41` registrations end 2017-05 and `1Z81` begin 2016-08, matching
the 94 Ah launch in July 2016. Sibling variant `7Z41` carries the **identical**
1340/1760 pair over 24 further registrations (2017-11 to 2020-08), confirming
the figure is not specific to one variant code within the 94 Ah car. The 50 kg
unladen and 30 kg gross step over the 60 Ah car is consistent with the larger
battery. Treating the two battery sizes as separate variants, as briefed, was
correct — they genuinely differ.
[query](https://opendata.rdw.nl/resource/m9d7-ebf2.json?merk=BMW+I&handelsbenaming=I3&cilinderinhoud=647&variant=1Z81&massa_ledig_voertuig=1340)

**Opel Ampera — approval `e13*2007/46*1159*01`, 1632 / 2000 kg.**
This one needed care. The Ampera's RDW `variant` field is the placeholder
`AAAA` for every registration, so it cannot disambiguate anything. Grouping by
`typegoedkeuringsnummer` instead revealed the problem: the **same** unladen
mass, 1632 kg, appears under two extensions of one approval with **two
different** permissible maxima — `*01` → 2000 kg (959 registrations,
2011-11 to 2014-03) and `*02`/`*04` → 2135 kg (398 registrations, 2012-10 to
2015-12). The two date windows overlap, so a model year alone cannot pick
between them. The row is therefore pinned to the approval number, not the
year. `*01` is the launch approval and the larger cluster.
Independently corroborated by the sibling **Chevrolet Volt**, registered in
the Netherlands under its own `handelsbenaming`, which states the identical
1632 / 2000 kg pair.
[query](https://opendata.rdw.nl/resource/m9d7-ebf2.json?merk=OPEL&handelsbenaming=AMPERA&typegoedkeuringsnummer=e13*2007/46*1159*01&massa_ledig_voertuig=1632)

**Mazda MX-30 R-EV — variant `6WJ`, 1753 / 2251 kg.**
418 registrations, all stating 2251 kg (`e13*2007/46*2300*10/*11`), first
admissions 2023-09-20 to 2025-07-03. Separated from the battery-only MX-30
(variant `1WB`, 1620 / 2119 kg, 2687 registrations) by the presence of an
engine capacity. A second, smaller cluster (1749 kg unladen, 55 registrations
from 2025-02) states the same 2251 kg gross; the dominant 1753 kg figure was
taken. `engine_displacement_cc` is deliberately **left empty**: RDW states
1660 cc with `aantal_cilinders` 1, which is the homologation convention of
twice the chamber volume for a Wankel, while Mazda names the 8C engine 830 cc.
Rather than write either number under an ambiguous definition, the field is
blank. It plays no part in the tare calculation.
[query](https://opendata.rdw.nl/resource/m9d7-ebf2.json?merk=MAZDA&handelsbenaming=MAZDA+MX-30&variant=6WJ&massa_ledig_voertuig=1753)

**LEVC TX — variant `A1`, approval `e11*2007/46*4190`, 2205 / 2900 kg.**
The London black cab as a series range-extender, registered **M1** despite
seven seats, with a 1477 cc 3-cylinder range-extender engine. Only 7 Dutch
registrations carry this pair — a small sample, noted as such — but every one
is identical on both masses and all sit under one approval family
(`*00` ×6, `*01` ×1), first admissions 2018-04-19 to 2018-11-15. Restricted to
that 2018 approval on purpose: a later homologation (`e5*2007/46*1068`,
registrations from 2021) states a **different** unladen mass, 2133 kg, against
the same 2900 kg maximum, and is not mixed in. The 695 kg useful load is the
highest of the five and is appropriate for a vehicle homologated to carry six
passengers plus driver and luggage.
[query](https://opendata.rdw.nl/resource/m9d7-ebf2.json?merk=LEVC&handelsbenaming=TX&variant=A1&massa_ledig_voertuig=2205)

## A basis collision, and why `payload_kg` is stated on the Ampera row

On the first build the Ampera came out `mass_pair_status = inconsistent-triple`
and produced **no** q point, despite a clean, single-source, single-basis
curb/gross pair. The cause is worth recording because it is exactly the class
of error this project keeps guarding against.

The Kaggle spec dataset enrichment grafted `payload_kg = 268` onto the row.
That figure is not wrong — it is on a **different basis**. Kaggle pairs it
with a curb mass of **1732 kg**, which is RDW `massa_rijklaar`, i.e.
`massa_ledig` + 100 kg. So:

- Kaggle, running-order basis: 1732 + 268 = 2000 kg ✓
- This file, unladen basis:    1632 + 368 = 2000 kg ✓

Both are internally consistent; mixing them is not. `mass_pair_status` did its
job and refused the row. The fix was to state `payload_kg = 368` explicitly on
the manual row — which is not an invented number, but precisely what
`build_vehicle_prototypes.py` derives itself (`payload_kg = gross − curb`)
when a source gives both masses and no payload. The manual value takes
precedence and Kaggle's 268 is recorded as a disagreement rather than applied.

This is the only place in the batch where a number was written that the source
did not state in those words, and it is arithmetic on two figures from one
record, not a borrowed value.

## Refused

| Vehicle | why refused |
|---|---|
| **Chevrolet Volt** (2011-2015) | Not refused for want of data — refused as a **duplicate**. The EU-registered Volt states the identical 1632 / 2000 kg pair as the Opel Ampera above, because it is the same car under a different badge. Adding it would put the same physical vehicle into the q population twice and weight the Voltec platform double. It is used above as an independent cross-check on the Ampera instead, which is the more valuable use of it. |
| **Fisker Karma** | No Dutch registrations at all (`merk LIKE 'FISKER%'` returns nothing in `m9d7-ebf2`), so the RDW route is closed. No alternative source was pursued before the research budget was capped. |
| **Chevrolet Volt 2016-2019 (2nd gen)** | US-market only; not sold in the EU, so absent from RDW. Not pursued further. |
| **Opel Ampera under approval `*02`/`*04`** (gross 2135 kg) | Not refused on evidence — the 1632 / 2135 pair is as well attested as the `*01` pair (398 registrations, all agreeing). Deliberately **not** added, because writing both would put the same nameplate and the same unladen mass into the q population twice under two different gross masses (q 4.44 and q 3.24), and no source found explains what changed between the extensions. See "Unfinished" below. |

## Unfinished — pick-up notes for the next run

Research was capped mid-batch. Nothing below was half-written into the data;
these are leads, not partial records.

1. **Opel Ampera, type-approval extension `*02`/`*04`.** Both masses are
   already in hand and verified — 1632 kg unladen, 2135 kg permissible maximum
   (`q = 3.24`), 398 NL registrations, all agreeing, first admissions 2012-10
   to 2015-12. What is **missing** is not a number but an explanation: what
   physically changed between `*01` (2000 kg) and `*02` (2135 kg) at an
   unchanged unladen mass. A raised axle rating, a towing package, or a
   homologation correction are all plausible and lead to different editorial
   calls about whether this is a second vehicle or a re-rating of the first.
   **Most promising source:** the EU type-approval certificate itself, or
   Opel's own 2013/2014 Ampera brochure or price list (`Opel Ampera
   Technische Daten`), which would state the *zulässiges Gesamtgewicht*
   alongside a model year. Decide, then either add the row or record the
   re-rating in the existing row's notes.
2. **Fisker Karma.** RDW is a confirmed dead end — zero registrations, do not
   re-run that query. **Most promising source:** the US EPA / NHTSA VIN
   decoder or the 2012 Karma press kit for GVWR; note that a US curb weight
   excludes the driver while the EU kerb basis includes 75 kg, so a US-sourced
   pair must be recorded as its own basis and not mixed with the RDW rows.
3. **Chevrolet Volt, 2nd generation (2016-2019).** Not in RDW (US-only).
   **Most promising source:** GM's own US spec sheet, which publishes curb
   weight and GVWR together on one page — the same basis for both, which is
   what this dataset needs. Would be the first US-basis REEV row.
4. **LEVC TX, later homologation `e5*2007/46*1068`** (2021+), unladen 2133 kg
   against the same 2900 kg maximum, q = 3.78. Only 3 NL registrations found,
   which is thin. **Most promising source:** LEVC's own UK technical
   specification sheet for the post-2020 TX, to confirm whether 2133 kg is a
   real lightening of the vehicle or a differently-drawn unladen figure.
5. **BMW i3 REx 120 Ah (2019-2022).** Not searched at all. It should exist in
   RDW under `merk = BMW I`, `handelsbenaming = I3`, `cilinderinhoud = 647`,
   under a variant code beyond `1Z41`/`1Z81`/`7Z41` — note that `7Z41`
   registrations already run to 2020-08 at the 94 Ah figures, so the 120 Ah
   car needs positive identification rather than assumption from date alone.
   Would complete the i3 REx battery ladder.

**Ruled out already — do not repeat:** Fisker in RDW (empty); using RDW
`massa_rijklaar` as the curb figure (it is `massa_ledig` + 100 kg, a different
basis from every other row in this file); using the Ampera's RDW `variant`
field to disambiguate anything (it is the placeholder `AAAA` on every
registration — group by `typegoedkeuringsnummer` instead); and trusting the
Kaggle spec dataset's Ampera curb/payload figures, which are on the
running-order basis (see above).

## Verification output

```
$ cd Data/Vehicle && python3 build/build_vehicle_prototypes.py
Web-sourced masses: filled 21, refused 0, unmatched 0
Kaggle: {'matched': 83, 'ambiguous': 137, 'unmatched': 282, 'filled': 23,
         'disagreements': 13, 'refused': 1}
wrote .../vehicle_prototypes.csv: 502 rows
wrote .../vehicle_prototypes_provenance.csv: 731 provenance rows (43 blanked)

$ python3 build/build_q_dataset.py
324 q points -> .../q-dataset.json

$ python3 -m unittest discover -s build -p 'test_*.py'
Ran 156 tests in 2.261s
OK

$ cd ../.. && npm run check
# tests 133
# pass 133
# fail 0
check-docs: all documentation files pass
manuscript-contract: 51 controls, 119 DOM lookups, 9 citations pass
✓ built in 375ms
```

Row count: 497 → 502 (+5, exactly the rows added). q points: 319 → 324 (+5,
one per added row). No pre-existing row changed status.
