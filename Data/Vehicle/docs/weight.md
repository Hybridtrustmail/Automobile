---
doc: Data/Vehicle/docs/weight.md
type: reference
status: active
authority: informative
component: data
summary: Vehicle weight data and method notes.
read-first: []
roadmap: []
---

# Vehicle Mass Family Dataset, 1945-2025

The single canonical dataset contains **9,259 deterministic modeled records**
covering **72 vehicle families** and every year from **1945 through 2025**.

These rows are **class-envelope estimates**, not measured homologation records
for particular makes and models. `Make`, `Model`, `Variant_Trim`, and
`NumericalDataNature` state this explicitly. This avoids the former error of
assigning current vehicle names to years before those models existed.

## Scope

Included families:

- Passenger cars: A through luxury/performance, estate/MPV, SUV and M1G.
- M2/M3 buses: minibuses, coaches, city, articulated, intercity, double-deck,
  school, trolley and off-road buses.
- N1/N2/N3 goods vehicles across their mass ranges, including vans, pickups,
  tractor units, rigid cargo bodies, tippers, refuse trucks, mixers, tankers,
  timber trucks, transporters, recovery vehicles and civilian G subclasses.
- O1-O4 trailers and semi-trailers as records separate from tractor units.
- Special-purpose ambulances, motor caravans, hearses, accessible vehicles,
  fire appliances, mobile cranes and armoured road vehicles.
- Agricultural/forestry T1-T4.2, C, R1-R4 and S families.
- Military wheeled and tracked utility, cargo, APC, IFV, tank, recovery,
  engineering and uncrewed families.
- ICE gasoline/diesel, LPG, CNG, LNG, hybrid, BEV, FCEV, trolley-electric and
  unpowered variants where applicable.

Intentionally excluded at the user's request:

- Construction and mining machinery.
- Powered two- and three-wheelers and quadricycles (category L).

## Taxonomy

`Category` is the dataset family code. `RegulatoryCategory` contains the
applicable EU category. Civilian off-road categories therefore use the proper
suffixes `M1G`, `M2G`, `M3G`, `N1G`, `N2G`, and `N3G`.

Military-only records use `Category = MIL-W` or `MIL-T` and
`RegulatoryCategory = Not applicable`, because military-only and tracked vehicles are
outside the scope of EU Regulation 2018/858. Military status is no longer
incorrectly encoded as `N3G`.

Special-purpose body codes are stored separately in `SpecialPurposeCode`;
other rows contain `Not applicable`.

## Mass Definitions

- `CurbMass_M0_kg`: modeled unladen mass, \(M_0\).
- `Payload_Mb_kg`: modeled passenger and cargo payload, \(M_b\).
- `GrossMass_Ma_kg`: \(M_a = M_0 + M_b\).
- `TareRatio_q`: \(q = M_0 / M_b\).
- `Passenger_Mass_kg`: up to 75 kg per modeled passenger.
- `Luggage_Cargo_Mass_kg`: remaining payload.
- `EnergyCarrier`: descriptive carrier only; no consumption is inferred.
- `PowertrainMassAdjustment_kg`: modeled curb-mass addition relative to the
  same family/year ICE baseline at unchanged gross mass.

The generator validates the M2/M3, N1/N2/N3 and O1/O2/O3/O4 gross-mass
boundaries on every run.

Powertrain mass additions apply to curb mass while gross mass remains fixed.
Payload therefore decreases and `TareRatio_q` increases for hybrid, BEV, FCEV,
and trolley-electric variants. Gross mass is sampled within category boundaries
rather than clipped onto them.

The historical envelope uses explicit piecewise transitions at 1960, 1973,
1982, 1990, 2010 and 2025. These transitions are modeling assumptions, not
empirical discoveries.

## Trailer and Combination Limits

The former `TowingCapacity_kg` estimate was removed because it was calculated as
a fixed proportion of curb mass and was not a permissible legal limit.

The replacement fields are:

- `TrailerTowCapability`: `Yes`, `No`, `Unknown`, or `Not applicable`.
- `MaxTowableMass_Braked_kg`: homologated braked-trailer limit.
- `MaxTowableMass_Unbraked_kg`: homologated unbraked-trailer limit.
- `MaxCombinationMass_kg`: homologated maximum laden combination mass.
- `MaxCouplingLoad_kg`: homologated static coupling-point load.
- `CouplingType`: fifth wheel, agricultural hitch, other known type, or unknown.
- `TowDataNature`: provenance status for the towing fields.
- `TowDataSource_Reference` and `TowSource_URL`: definition source.

The four numeric homologation fields are intentionally empty. These limits vary
by exact vehicle variant, coupling, body completion and approval, so a class
estimate cannot provide a legally valid value. Tractor units, agricultural
tractors and recovery vehicles are marked as designed to tow; other powered
families remain `Unknown` until a model-specific source is added.

## Sources and Limitations

Provenance is separated so a classification regulation cannot be mistaken for
evidence supporting a numerical value:

- `ClassificationSource_Reference` and `ClassificationSource_URL` identify the
  authoritative category-definition regulation.
- `NumericalDataNature` identifies values as modeled estimates.
- `NumericalMethod_Reference` points to the local deterministic generator.
- `EmpiricalNumericSource_Reference` and `EmpiricalNumericSource_URL` explicitly
  state when no model-specific empirical source is available.

Classification references are:

- EU Regulation 2018/858 for M, N, O, G and special-purpose road vehicles.
- EU Regulation 167/2013 for T, C, R and S agricultural/forestry vehicles.
- EU Regulation 2021/535 for towable mass, combination mass and coupling load.

The numerical values are reproducible engineering class estimates with
deterministic variation. They must not be cited as exact manufacturer,
registration or certification observations. The unsupported power,
displacement, battery, consumption and range columns were removed. The remaining
mass-envelope endpoints also lack empirical model-level sources and must not be
used as observational evidence or for statistical inference. Model-level
research requires a separate source-per-vehicle archival dataset.

## Regeneration

```bash
python3 build_full_80yr_dataset.py
python3 validate_vehicle_mass_dataset.py
python3 plot_80yr_dynamics.py
```
