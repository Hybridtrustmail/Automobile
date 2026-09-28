---
doc: Data/Vehicle/docs/embedded-table-audit.md
type: reference
status: active
authority: informative
component: vehicle-data
summary: Records which calculation tables belong in canonical datasets and which remain beside their equations.
read-first: []
roadmap: []
---

# Embedded calculation-table audit

Reviewed 2026-09-22. Large records that evolve independently of equations are
stored under `Data/Vehicle` and generated for the browser. This now includes
vehicle prototypes, drive cycles, electric-drive components, efficiency maps,
the tyre product catalogue, tyre load indices and tyre speed categories.

The following tables remain beside their equations because they define a
specific published method or a short finite choice rather than a reusable
catalogue: engine-curve coefficients, transmission efficiency ranges, fuel
properties, braking test factors, clutch lining dimensions, gear tooth-form
factors and module series, leaf-spring standard series, and layout/topology
enumerations. Moving these small tables to runtime CSV would separate each
equation from its versioned method without making catalogue maintenance easier.

Generated `*-data.js`, `*.json` datasets and `vehicle_prototypes.csv` must not
be edited by hand when a builder or source CSV exists. Source locators belong
in the canonical CSV or its companion provenance file.
