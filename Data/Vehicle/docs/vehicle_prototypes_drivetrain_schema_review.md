---
doc: Data/Vehicle/docs/vehicle_prototypes_drivetrain_schema_review.md
type: plan
status: active
authority: informative
component: data
summary: Drivetrain schema review notes.
read-first: []
roadmap: []
---

# Drivetrain configuration — schema design review

Date: 2026-08-04
Scope: how `vehicle_prototypes.csv` should represent a model that ships in several drivetrain configurations.
Status: recommendation. Nothing implemented; no file in the working directory was modified.

---

## Recommendation

**One row per sourced mass set. Drivetrain configurations that do not come with their own published masses go in a long-format sidecar — `drivetrain_configs.csv`, which is `motor_topology.csv` widened and promoted from a build-time override to a published artifact.** The main row carries exactly one configuration: the one the row's mass and dimension figures were actually sourced for, named in a new `drivetrain_config` token that doubles as the sidecar's discriminator.

**Repair `axle_configuration` and `drive_layout` in place — do not rename, do not deprecate, do not add columns.** Both value spaces are already correct in the schema doc; only one loader writes into them wrongly, and every wrong value it writes is a byte-identical duplicate of a correct value in the sibling column. The repair is lossless and touches one function.

**Delete `axle_count` as a sourced column and compute it in `derive()` from `axle_configuration`,** alongside a new derived `driven_axle_count`. The stored column is not thin, it is wrong.

The boundary rule, stated once and mechanically:

> A configuration gets its own **row** if and only if the source states a different value for `curb_mass_kg`, `gross_mass_kg`, `payload_kg`, or any of `length_mm` / `width_mm` / `height_mm` / `wheelbase_mm`.
> Otherwise it gets a **line in the sidecar**.
> It is a different **vehicle** (not a variant) if the nameplate, the `generation` code, or the `body_type` differs — regardless of mass.

For the Solaris Urbino 12 electric this yields: one row, three sidecar lines. That is the right answer, and I explain below why.

---

## Reasoning

### What I verified, and where your framing is wrong

I checked all of this against `vehicle_prototypes.csv` (316 rows), the five loaders, and the upstream `prototype_enrichment.csv` / `niiat_trucks.csv`.

**Defect 1 confirmed, but the diagnosis is much narrower than "five loaders populate them inconsistently".** Cross-tabulating the two columns by tier:

| tier | rows | writes `axle_configuration` | writes `drive_layout` |
|---|---:|---|---|
| enrichment | 95 | 2-axle 37, FWD 6, 4x2 5, RWD 4, 6x2 3, 6x4 3, AWD 3, 8x4 1 | AWD 22, FWD 17, RWD 11, 4x2 5, 6x2 3, 6x4 3, 8x4 1 |
| niiat | 103 | 4x2 49, 4x4 13, 6x6 11, 6x4 8, 8x8 1, 8x4 1 — all well-formed | — |
| fastsim | 67 | 4x4 3, 4x2 2, 6x4 2, `2-wheel` 2 | — |
| ev_sim | 12 | — | — |
| manual / web-cited / brochure | 39 | — (**cannot**: the column is absent from `manual_vehicles.csv`'s header**)** | AWD 1, RWD 3 |

Only `load_enrichment()` writes bad values. And the pairwise cross-tab shows they carry **zero unique information**:

```
('2-axle','AWD') 19   ('2-axle','FWD') 11   ('2-axle','RWD') 7
('FWD','FWD')     6   ('4x2','4x2')     5   ('RWD','RWD')    4
('6x2','6x2')     3   ('6x4','6x4')     3   ('AWD','AWD')    3
('8x4','8x4')     1
```

Twenty-five rows have the two columns byte-identical; the other 37 have `axle_configuration = '2-axle'`, which duplicates `axle_count = 2`. The cause is upstream: `prototype_enrichment.csv`'s `axle_layout` is itself the mixed column, and `load_enrichment()` copies it into `axle_configuration` while separately copying the clean `drive_layout` through. Deleting every bad value loses nothing at all. The only non-enrichment offender is `2-wheel`, from `NON_CAR` in the build script for two motorcycles — and the correct axle formula for a chain-drive motorcycle is `2x1`, which is well-formed, so that needs no special case either.

**Defect 2 confirmed, but the cause is not what you think.** `axle_configuration` is 154/316 mostly because `manual_vehicles.csv` has no such column in its header. All 21 buses and all 38 web-cited rows are structurally unable to carry an axle formula. Adding one header field is the single highest-yield coverage action available and costs nothing.

**Defect 3, which you did not mention: `axle_count` holds the wrong quantity.** Every filled value:

```
car / 2-axle → 2.0        truck / 4x2 → 4.0
truck / 6x2 → 6.0         truck / 6x4 → 6.0
truck / 8x4 → 8.0         motorcycle → 1
```

For the 12 truck rows it is the *wheel-end* count — the first digit of the axle formula — not the axle count. A 4x2 truck has two axles, not four. The column is right for cars by coincidence (2 axles, and 4x2 would give 4, but they were tagged `2-axle` so the estimator produced 2). It is a sourced column holding a value that is 100% derivable from `axle_configuration` and derivable *correctly*, which the stored version is not. Store nothing; derive `axle_count = first/2` and `driven_axle_count = second/2`.

**Defect 4, also unmentioned, and it is the ICE half of your Solaris problem.** In the niiat tier, 66 of 67 `final_drive_ratio` values are non-numeric — they use comma decimal separators (`3,9`, `6,170`) and pass through `load_niiat()` as raw strings with no `num()` call. Worse, 15 of them read like `3,9 или 4,1`, `6,170 или 4,556`, `4,625 или 4,111` — "3.9 **or** 4.1". That is exactly your case: one model, one year, two published final drives, mass figures identical. It is already in the data, encoded as free text in a numeric column, silently breaking the traction calculation. These 15 rows are the natural first customers of the drivetrain sidecar and they are all ICE.

**Your consumer-breakage premise is too pessimistic.** I grepped the whole `Transport/` tree. Nothing reads `vehicle_prototypes.csv` — not the browser app, not any notebook, not any script other than the build itself. `Vehickle_calculations_assets/app.js` reads `catalog.js`, `q-reference.js` and `q-named-reference.js`, which are generated from `vehicle_mass_80yr_dataset.csv` via `build_q_web_reference.py`, a different pipeline. The `curb_mass_kg` / `gross_mass_kg` / `payload_kg` / `tare_ratio_q` contract is real as documentation but is not currently exercised by anything. I still recommend not breaking it — it costs nothing to keep — but the migration risk you are pricing in is not there.

**Corroborating evidence for the repair direction, from the app side.** `catalog.js` `axleOptions` already uses a strict `NxM` vocabulary per category — `4x2, 4x4, 6x2, 6x4, 6x6, 8x2, 8x4, 8x8`, no FWD/AWD anywhere. The eventual consumer's contract is already the clean one. (Note in passing: `catalog.js` line 77, `m1-suv-hybrid`, carries `axle: "AWD"`, which can never match its own M1 option list of `4x2`/`4x4`. Same confusion, app side, pre-existing, out of scope here — flagging it, not touching it.)

### Why the sidecar rather than rows, and why not more columns

Three facts decide it.

**Fact one: drivetrain configurations without their own masses are the common case, and they are the case you actually hit.** Solaris publishes one GVW (20 000 kg) for the Urbino 12 electric — it is a type-approval figure that does not move with the motor choice — and publishes no curb mass at all for any of the three configurations; the row's `curb_mass_kg` is blank for exactly that reason, documented in its `notes`. Three configurations, one mass set, no mass set. Three rows would mean three copies of the same 60 columns differing in four fields, and the row's own `notes` already records the other two configurations in prose, which is the schema telling you it has nowhere to put them.

**Fact two: `q` is insensitive to drivetrain topology and sensitive to payload rating.** `tare_ratio_q` is filled on 72/316 rows: 64 trucks, 8 cars, **zero buses**. Only 8 rows carry both `motor_count` and `q`, and they are the only paired evidence in the set:

| vehicle | curb | payload | q | topology |
|---|---:|---:|---:|---|
| Tesla Model 3 Long Range AWD | 1830 | 471 | 3.885 | 1-front+1-rear |
| Tesla Model 3 Performance AWD | 1836 | 465 | 3.948 | 1-front+1-rear |
| Tesla Model Y Long Range AWD | 1979 | 612 | 3.234 | 1-front+1-rear |
| Tesla Model Y Performance AWD | 1995 | 596 | 3.347 | 1-front+1-rear |
| Porsche Taycan 4S | 2140 | 740 | 2.892 | 1-front+1-rear |
| Porsche Taycan 4S Plus | 2220 | 560 | 3.964 | 1-front+1-rear |

Within a nameplate-year, q moves 1.6% (Model 3) and 3.5% (Model Y) across variants with *identical* topology, and 37% across the two Taycans — also identical topology, driven entirely by the payload rating (740 → 560 kg). Not one of these six spreads is caused by drivetrain topology. Meanwhile the schema already tolerates, inside a single row, a `_min`/`_max` curb spread with a **median of 3.3% and a maximum of 27%** (Honda Civic 600–790 kg, Corolla E30 785–955 kg). A central-motor-to-hub-motor swap on a 12 m bus is order 100–200 kg on ~12 000 kg curb — 1–2% in q, *smaller than the uncertainty the schema already accepts within one row.* So: **a drivetrain variant does not warrant its own row on q grounds.** It warrants one only when a source publishes different masses, which is precisely the boundary rule, and which is a rule about sourcing rather than about physics — which is what makes it mechanical.

**Fact three: more columns cannot express the case at all.** The topology columns are singular by construction — one `motor_count`, one `motor_layout`. Representing three Solaris configurations on one row means `motor_count_1/2/3`, and the drivetrain plan already anticipated and rejected this: *"Should several such rows accumulate, the fix is a long-format drivetrain sidecar keyed on (vehicle, axle, motor index), not more columns on the main table."* The Solaris case is that accumulation arriving. Widening `motor_topology.csv` is the plan's own stated next step, not a new idea.

### Why the per-config mass does **not** go into `_min`/`_max`

Tempting, and wrong, and the dataset already ruled on it. The Solaris row's `notes` carry a controller correction:

> "The source's 3300/3480 pair is WITHOUT vs WITH pantograph — two distinct configurations, not a trim spread — so it was moved out of the `_min`/`_max` range columns: their computed mean (3390 mm) would describe no real bus."

That precedent binds. A range must be a spread of one quantity; two configurations are two quantities. Where a source *does* publish a per-configuration curb mass, it belongs on the sidecar line, not in the range columns. Using `_min`/`_max` for this would manufacture a mean vehicle that does not exist — the exact error the design doc names.

### What the sidecar looks like

`motor_topology.csv` already has the right key `(make, model, year, variant_trim)`, the right loader, and 7 of the needed columns. Add a discriminator and the fields that vary by configuration:

```
make, model, year, variant_trim, drivetrain_config,     # key + discriminator
is_primary,                                             # exactly one TRUE per key
axle_configuration, drive_layout,
motor_count, motor_layout,
motor_power_front_kw, motor_power_rear_kw,
motor_reduction_front, motor_reduction_rear,
drivetrain_ratio_lumped,
engine_power_kw, final_drive_ratio, gearbox_ratios, gear_count,
transmission_type, transfer_ratios,
curb_mass_kg, gross_mass_kg,                            # only if published per config
source_url, quote, notes
```

`drivetrain_config` is a short human token: `cetrax-220`, `axtrax-2x125`, `async-160`; for the niiat trucks, `fd-3.9` / `fd-4.1`. Semantics stay exactly as `motor_topology.csv` has them today — the `is_primary` line is folded into the main row before `derive()`, and non-primary lines are published but not folded. That preserves every existing consumer read: the main CSV keeps one value per column and nothing about `curb_mass_kg` or `tare_ratio_q` changes shape.

Two properties follow for free. Mass and dimensions live in exactly one place per configuration, so there is nothing to drift. And a configuration that is only *partly* known — Solaris's 160 kW asynchronous option, for which no ratio, no mass, and no reduction are published — is one line with three fields filled and the rest blank, which is honest, rather than a row with 55 fields copied from a sibling it may not share.

### Duplication and drift, for the rows that *do* split

When a source does publish per-configuration masses, you get separate rows and ~60 columns duplicated. Do not normalise this away — the flat-CSV constraint is real and a body table plus a join is exactly what a browser consumer should not have to do. Control drift with a check instead of a structure:

Add a build-time gate: rows sharing `(make, model, year, generation, body_type)` must agree on `wheelbase_mm`, `width_mm` and `height_mm` exactly and on `length_mm` within 1%, else log to the provenance sidecar under a new `method = geometry_disagreement` and leave the values in place. Warn, do not blank — a genuine long-wheelbase variant should be a different `body_type` and this gate is how you find out it was not.

That, plus the discipline the dataset already has — anything derivable is recomputed on every build and never stored by hand — is sufficient. Extending that discipline to `axle_count` and `driven_axle_count` removes the only stored-and-wrong pair in this area.

---

## Alternatives rejected

**Row per configuration.** Duplicates ~60 columns for what is usually a 4-field difference, and the multiplier lands where it hurts most: buses and electric heavy duty, the region Phase C is about to populate, where curb mass is often unpublished and payload is a passenger-capacity assumption rather than a source figure. Zero of 21 bus rows have `q` today. Splitting them by drivetrain multiplies rows in the region where the book's central quantity does not exist, and does not add one usable q. It would also silently double-weight those bodies in any population statistic over the CSV.

**More columns on the main row.** Cannot represent N configurations without an N-fold column suffix explosion, and the drivetrain plan already committed against it in writing.

**Sidecar for everything, main row carries no drivetrain at all.** Cleanest on paper, and I rejected it on one hard fact: **41 of 316 rows (13%, all niiat) have blank `make` and blank `model`** — `niiat_trucks.csv` has an empty `model` on 41 of 88 rows, so `load_niiat()` derives no make either. Those rows have no stable identity and cannot be reached by any sidecar keyed on `(make, model, year, variant_trim)`. Thirty of the 72 rows that carry `q` are in that set. A design that puts drivetrain *only* in the sidecar makes drivetrain permanently unreachable for 13% of the dataset, including its best q coverage. Keeping the primary configuration on the row means those rows still work the moment `parse_niiat.py` is fixed, and degrade gracefully until then.

**Rename `axle_configuration` → `axle_formula` with a deprecation window.** I considered this for its value as a forcing function: a consumer reading the old name breaks loudly instead of silently receiving a different value space. Rejected because there are no consumers, so the forcing function is worth nothing and the churn across the design docs, `manual_vehicles.csv`, `COLUMNS` and the app's vocabulary is real. Repair in place, and enforce the value space with a gate.

**Enumerating configurations in `variant_trim`.** This is the current de-facto practice — the Solaris row's `variant_trim` reads `ZF CeTrax 220 kW / Solaris High Energy 420 kWh (UITP 2023 spec)`, and `q-named-reference.js` does the same with `Volkswagen Golf GTI 2.0 TSI DSG`. It conflates drivetrain with battery with market spec in one unparseable string, and it is what makes the dedup key fragile: two rows for the same vehicle with differently-worded trims will never merge. Keep `variant_trim` as the trim, put the configuration in `drivetrain_config`.

---

## Migration plan

Ordered. Each step is independently verifiable, and steps 1–4 are prerequisites that stand on their own merit whatever you decide about drivetrain.

**1. Fix `parse_niiat.py` so no row has a blank `model`.** 41/88 rows currently do. Nothing keyed on stable identity works for them until this lands. Verify: `sum(1 for r in rows if not r['make'] or not r['model']) == 0`, and 316 rows still out.

**2. Fix `load_niiat()`'s numeric parsing.** Normalise the comma decimal separator so `final_drive_ratio` is a number; where the value matches `X или Y`, write nothing to `final_drive_ratio` and emit one sidecar line per option (this is step 7's first payload). Verify: every non-blank `final_drive_ratio` parses as float; 15 rows move to the sidecar; the `final_drive_ratio ∈ 2–12` gate from the design doc becomes enforceable for the first time.

**3. Repair the two columns.** In `load_enrichment()`: write `axle_configuration` only when `axle_layout` matches `^\d+x\d+$`; otherwise leave it blank (the value is already in `drive_layout`, verified duplicate). Change `NON_CAR`'s motorcycle entries from `2-wheel` to `2x1`. Add a validation gate that blanks any `axle_configuration` not matching `^\d+x\d+$` and any `drive_layout` not in `{FWD, RWD, AWD, 4WD}`, logging under the existing `method = blanked_implausible`. Verify: `axle_configuration` value set is `{4x2, 4x4, 6x2, 6x4, 6x6, 8x4, 8x8, 2x1}`; `drive_layout` value set is `{FWD, RWD, AWD}`; `axle_configuration` filled count drops 154 → 100; `drive_layout` unchanged at 66; **no information lost**, which is checkable — every blanked value must be reproducible from the sibling column on the same row.

**4. Derive `axle_count`, add `driven_axle_count`.** Remove `axle_count` from `load_enrichment()`; compute both in `derive()` from `axle_configuration`. Verify: 12 truck rows change from 4/6/8 to 2/3/4 — these were wrong before and are right now; `driven_axle_count` newly filled wherever `axle_configuration` is.

**5. Add `axle_configuration` to `manual_vehicles.csv`'s header** and populate it for the 21 buses and the researched rows where a source states it. Verify: coverage rises from ~100 toward ~140 with no unsourced value added.

**6. Add `drivetrain_config` to `COLUMNS`** and to `manual_vehicles.csv`'s header. Leave it blank everywhere. This is a no-op build that establishes the column. Verify: 316 rows, 82 columns, byte-identical otherwise.

**7. Rename `motor_topology.csv` → `drivetrain_configs.csv`, widen it, add `is_primary`.** `load_topology_overrides()` becomes `load_drivetrain_configs()`: fold only `is_primary` lines into the main row, exactly as today; carry the rest through to a published sidecar. Add a gate that each key has at most one `is_primary`. Verify: with every existing line marked `is_primary`, the main CSV is byte-identical to step 6's output. That equality is the whole safety argument for this step — do not proceed past it without seeing it.

**8. Enter the real multi-configuration cases.** Solaris Urbino 12 electric: three lines (`cetrax-220` primary, `async-160`, `axtrax-2x125`), and trim the two-sentence configuration digression from the row's `notes`, keeping the citation. The 15 niiat dual-final-drive trucks from step 2. Verify: sidecar has >1 line for exactly those keys; main CSV unchanged from step 7.

**9. Add the geometry-agreement gate** described above. Verify: it either logs nothing or logs something worth reading.

**10. Update `vehicle_prototypes_design.md` and `vehicle_prototypes_drivetrain_plan.md`** with the value spaces, the boundary rule, and the sidecar contract.

Nothing in steps 1–10 changes `curb_mass_kg`, `gross_mass_kg`, `payload_kg` or `tare_ratio_q` on any row. The documented consumer contract is preserved throughout.

---

## Risks and open questions

**The boundary rule, applied retroactively, would collapse 16 existing rows into 8.** Eight enrichment groups are pure trim pairs with identical masses — Ford Puma ST-Line / Titanium X (both 1205 kg), Lexus RX Business / Executive (both 2075), Mazda MX-30 First Edition / Luxury (both 1620), Tesla Model Y LR / Performance (both 1950), Toyota C-HR Active / Dynamic (both 1350), Volvo V90 Inscription / R-Design (both 2000), Audi e-tron Business Edition / Plus (both 2375), Honda e Advance / "specs" (both 1525). **I recommend leaving them.** They carry no `payload_kg` and therefore no `q`, so they cannot distort the population statistic the book computes, and the enrichment tier is a carry-through the design doc treats as read-only in spirit. Apply the rule to new data only. If you disagree, collapsing is safe and cheap — but it is a separate decision from this one.

The rule reproduces every *other* existing multi-row group correctly, which is the best validation available: Fiat Doblo (payloads 595/555/723), VW LT 28 short vs average base (1000/1118), Tesla Model 3 LR/Performance (1830/1836), all four Taycan and Model X rows. That is not a coincidence; it is the rule being descriptive of practice already followed.

**`is_primary` is a judgement call and I have not made it mechanical.** For Solaris, "the configuration the masses were sourced for" picks `cetrax-220` unambiguously, because the UITP spec sheet is the source of the GVW. For a vehicle whose masses come from a source that never names a drivetrain — most of the enrichment tier — there is no principled primary. Provisional rule: if the mass source names a configuration, that one is primary; if it names none, no line is `is_primary` and the main row's topology columns stay blank. I am not confident this survives contact with 20 more heavy-duty rows, and it is the part of this design I would most expect to revise.

**I have not verified the Solaris mass claim.** I am asserting that Solaris publishes one GVW across all three motor options and no curb mass for any of them, on the strength of the row's own `notes` and the fetched UITP PDF cited there. If a Solaris source does give per-configuration curb masses, the case becomes three rows under my own rule and the triggering example stops being an argument for the sidecar. The rule still holds; the example changes sides. Worth one fetch before step 8.

**The hub-motor mass delta is my estimate, not a sourced figure.** The 100–200 kg figure for AxTrax-vs-CeTrax is engineering judgement about portal-axle hardware, used only to argue a magnitude. It must not enter the dataset, and if the ~1–2% q claim matters to a decision, source it.

**Unsprung mass has nowhere to live.** You named it as a real difference between hub-motor and central-motor configurations, and it is — for ride and for tyre-load variation. The schema has no unsprung-mass column and I am not proposing one; nothing in the traction, gradeability or q calculations uses it. If the book's chapter on hub motors needs it, that is a separate column on the sidecar line, added when there is a source for it.

**Buses have curb mass but no q.** Twenty of 21 bus rows have `curb_mass_kg`, none have `payload_kg`, so none have `q`. For M3 the useful load is a passenger-capacity calculation (`seats`, standees, `passenger_mass_kg`), not a published payload. This is unrelated to drivetrain, but it is the largest single hole in q coverage and it will get worse as Phase C adds 15–20 more buses and lorries. Worth its own decision soon.

**Open: should `drivetrain_config` tokens be a controlled vocabulary?** I have proposed free-form short tokens. Controlled vocabularies are better for querying and worse for honesty when a real configuration does not fit. Given 15 niiat final-drive pairs plus a handful of bus cases, the population is too small to design a vocabulary against. Revisit at ~50 sidecar lines.
