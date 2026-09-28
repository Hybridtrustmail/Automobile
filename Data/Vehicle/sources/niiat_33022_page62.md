---
doc: Data/Vehicle/sources/niiat_33022_page62.md
type: reference
status: active
authority: informative
component: vehicle-data
summary: Reviewed NIIAT model 33022 torque columns and importer correction.
read-first: ["Data/Vehicle/README.md"]
roadmap: []
---

# NIIAT model 33022: reviewed source excerpt

[1] НИИАТ. Краткий автомобильный справочник. Том 2, 2006.
Printed page 62 (PDF page 62), engine specification table in the model
33022 section beginning on page 60. Local original:
`Literature/Transport/Автомобили/Spravochnik/НИИАТ/kratkii_avtomobilnyi_spravochnik_tom_2_2006.clean.pdf`.

This is a reviewed excerpt, not a transcription of the whole book.

| Property [1, p. 62] | First engine column | Second engine column |
| --- | --- | --- |
| Engine | УМЗ-4215.10-10 | УМЗ-4215.10 |
| Displacement, cm³ | 2890 | 2890 |
| Maximum power, kW | 65.4 | 70.5 |
| Maximum torque, N·m | **196** | **206** |
| Torque speed, rpm | 2200–2500 | 2200–2500 |
| Fuel | Gasoline | Gasoline |

## Correction and rebuild rule

The extracted row contains separate columns:

```text
максимальный крутящий               196   206    164,6 172,0 168,5 172,0
```

`196206` is **not** a torque from the book. The former `first_num` helper
joined whitespace-separated numbers. `parse_niiat.py` now reads the first
torque column independently, with regression tests against this row.

The existing CSV record adopts the first engine column (65.4 kW, 2890 cm³),
so its matching torque is 196 N·m. Do not replace this with 206 N·m from the
second variant, or infer that every modification has the first column's data.
The correction applies to `sources/niiat_trucks.csv` model `33022` and
`vehicle_prototypes.csv` record `vp-7e795f`.

When regenerating OCR-derived tables, run `npm run test:data` before building
the UI dataset. Keep this reviewed source note with the data so a later import
does not silently reinstate the joined value.
