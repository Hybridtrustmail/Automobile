---
doc: Data/Vehicle/docs/vehicle_prototypes_bus_payload_standard.md
type: reference
status: active
authority: informative
component: data
summary: Bus payload standard for vehicle prototypes.
read-first: []
roadmap: []
---

# Regulatory standard for bus/coach useful load (payload)

Research date: 2026-08-04. Scope: EU/UNECE type-approval framework for M2/M3
vehicles (buses and coaches).

## Summary table

| Class | Description | Per-passenger mass Q (kg) | Standing area per passenger Ssp (m²) | Standing allowed? | Luggage allowance |
|---|---|---|---|---|---|
| Class I / Class A | Urban, standing passengers, frequent movement | **68 kg** | 0.125 m² | Yes | B ≥ 100 × V (kg), V = baggage compartment volume in m³ |
| Class II | Interurban, seated + limited standing (gangway/limited area) | **71 kg** | 0.15 m² | Yes (limited) | B ≥ 100 × V (kg) |
| Class III / Class B | Coach, seated passengers only | **71 kg** | "Not applicable" (no standing area rated) | No | B ≥ 100 × V (kg) |

Crew member mass: **75 kg** (all classes).
Driver mass (separate definition, used for kerb/running-order mass, not payload): **75 kg**, located at the driver's seating reference point.

These figures are the **type-approval requirements** in Commission Regulation
(EU) No 1230/2012, Annex I, Part B — the regulation that implements Regulation
(EC) No 661/2009 for masses and dimensions and that all M2/M3 vehicles must
satisfy for EU type-approval. They are not "common engineering practice"
figures; they are the numbers a manufacturer must use to demonstrate that the
technically permissible maximum laden mass is not exceeded under point 2.2.3.

## Exact quotes and sources

Primary source attempted: EUR-Lex (eur-lex.europa.eu), CELEX 02012R1230.
**EUR-Lex itself could not be reached this session** — every fetch attempt
(WebFetch tool and direct `curl`) was blocked by EUR-Lex's bot-protection
(AWS WAF returned HTTP 202 with `x-amzn-waf-action: challenge` and zero
content; the WebFetch tool returned an empty page). UNECE's own site
(unece.org) was likewise blocked — direct `curl` returned `HTTP 403` from
Cloudflare, and PDF fetch attempts returned a Cloudflare "Just a moment..."
JS-challenge page instead of the document.

As a substitute I fetched the **consolidated text of the same EU Regulation
as republished by legislation.gov.uk** (UK National Archives, the official
statutory database that mirrors retained/EU law verbatim), which loaded
successfully via direct `curl`:

- URL fetched: `https://www.legislation.gov.uk/eur/2012/1230/2019-12-02/data.xht`
  (consolidated text of Commission Regulation (EU) No 1230/2012, Annex I,
  Part B, as amended to 2019-12-02).

Exact quoted text extracted from that fetched document:

> "2.2.3.8. Values of Q and Ssp values
> Vehicle class | Q (kg) | Ssp (m²)
> Class I and A | 68 | 0,125 m²
> Class II | 71 | 0,15 m²
> Class III and B | 71 | Not applicable
> The mass of each crew member shall be 75 kg."

> "2.2.3.9. The number of standing passengers shall not exceed the value
> S1/Ssp, where Ssp is the rated space provided for one standing passenger
> as specified in the table in point 2.2.3.8."

> "2.2.3.10. The value of the maximum permissible mass of the luggage shall
> be not less than:" — followed by an inline formula image. I downloaded the
> image (`http://www.legislation.gov.uk/eur/2012/1230/images/eur_20121230_2019-12-02_en_019`)
> and read it directly: **B = 100 × V**, i.e. minimum permissible luggage
> mass (kg) = 100 kg for every m³ of baggage-compartment volume.

> Notations (point 2.2.3.1): "'V' total volume of baggage compartments in m³
> including luggage compartments, racks and ski-box;" "'B' maximum
> permissible mass of the luggage in kg stated by the manufacturer,
> including the maximum permissible mass (B') that may be transported in
> the ski-box if any."

> Article 2, definition (20): "'mass of the driver' means a mass rated at
> 75 kg located at the driver's seating reference point."

> Article 2, definition (19): "'mass of the passengers' means a rated mass
> depending on the vehicle category multiplied by the number of seating
> positions including, if any, the seating positions for crew members and
> the number of standees, but not incl[uding the driver]..."

> Annex I Part B, point 2.2: "The technically permissible maximum laden mass
> of the vehicle shall not be less than the mass of the vehicle in running
> order plus the mass of the passengers plus the mass of the optional
> equipment plus the mass of the coupling if not included in the mass in
> running order."

> Annex I Part B, point 13 (Special provisions for buses and coaches),
> confirms class labelling used throughout: "13.1. Class of vehicle: Class
> I/Class II/Class III/Class A/Class B."

**Caveat on provenance**: legislation.gov.uk is the UK government's official
statute-law database and reproduces the EU regulation's operative text
verbatim (as retained EU law), so this is treated as a **near-primary**
source. It was NOT possible this session to cross-check the identical
wording directly on EUR-Lex due to the WAF block described above. The
consolidated version fetched carries date-stamp 2019-12-02, i.e. it includes
amendments up to that point; I did not separately verify whether any later
amendment (post-2019) altered these specific figures (2.2.3.8/2.2.3.10 are
long-standing and have not been reported as amended in what I could find,
but this is not independently confirmed against a later consolidated text).

## Class definitions (UNECE Regulation No. 107) — partially confirmed

Fetch attempts against unece.org (both the regulation landing pages and the
direct PDF `R107r7e.pdf`) were blocked by Cloudflare's JS challenge page
every time — no regulation text was retrieved from unece.org this session.

I did successfully fetch one document that discusses R107 class definitions:
**FIA "Fact File 89 — UNECE Regulation 107"** (https://www.fia.uk.com/static/d965dea0-6278-4f8a-b580ee29126ea56d/FACT-FILE-89-UNECE-Regulation-107.pdf),
a PDF from the FIA (a technical/regulatory body, not the primary UNECE text
but a near-primary secondary summary aimed at compliance). Retrieved via
WebFetch and converted to text with `pdftotext`. Exact quotes from that
fetched PDF:

> "Vehicle Class I - vehicles constructed with areas for standing
> passengers, to allow frequent passenger movement."

> "Vehicle Class II - vehicles constructed principally for the carriage of
> seated passengers and designed to allow the carriage of standing
> passengers in the gangway and/or in an area which does not exceed the
> space provided for two double seats."

> "Vehicle Class III - vehicles constructed exclusively for the carriage of
> seated Passengers."

This document did not define Class A or Class B (it is a fire-suppression
compliance-deadline fact sheet, not a comprehensive class-definition
reference), so those two are **not confirmed from a fetched document** in
this session — see "Not confirmed" section below.

## Not confirmed (marked explicitly, do not treat as established)

- **Class A / Class B verbal definitions** ("Class A = minibus with standing
  provision, Class B = minibus without standing provision" or similar,
  capacity ≤ 22 passengers). This appeared only in a search-engine-generated
  summary (not a document I fetched and could quote from). I attempted to
  fetch the UK Vehicle Certification Agency's category-definitions page for
  corroboration; it returned "Your IP Address is blocked. Reason: Malware
  Detected" — no content retrieved. **Unconfirmed** — do not cite the Class
  A/B verbal definitions as established without a fetched primary source.
  (The numeric mass/space table for Class A and B, by contrast, IS confirmed
  — it comes directly from the fetched Regulation (EU) 1230/2012 text above,
  where Class A is grouped with Class I and Class B is grouped with Class
  III for Q/Ssp purposes.)
- **UNECE Regulation No. 107 primary text** — could not be fetched at all
  this session (Cloudflare-blocked on every attempt, both landing pages and
  direct PDF links). The class definitions above are sourced from the FIA
  fact file, not from R107 itself. If bit-exact R107 wording is required,
  it still needs to be fetched from a working mirror.
- **EU Directive 97/27/EC** and **Directive 2001/85/EC** — not fetched this
  session (out of time-budget given the repeated blocks above); Regulation
  (EU) 1230/2012 above supersedes 97/27/EC for this purpose and was
  confirmed instead. If the 2001/85/EC class-definition wording specifically
  is needed (rather than the UNECE R107 wording, which by now is the
  operative reference since 2001/85/EC was repealed/absorbed into the
  Regulation 2018/858 framework, with R107 as the technical annex), that is
  a further unconfirmed gap.
- Whether Regulation (EU) 1230/2012 point 2.2.3.8/2.2.3.10 figures have been
  amended in any consolidation after 2019-12-02 was not checked (see caveat
  above).

## The 75 kg discrepancy with weight.md

The project's `weight.md` (in the working directory, not modified) states:

> "`Passenger_Mass_kg`: up to 75 kg per modeled passenger."

The regulatory figure actually confirmed above is **68 kg (Class I/A) or
71 kg (Class II/III/B)** per passenger — i.e. 4–7 kg **lower** than the
75 kg the dataset uses for passengers.

Note carefully what 75 kg actually IS in the regulation: it is the mass
assigned to the **driver** (Article 2, definition 20) and to **each crew
member** (point 2.2.3.8), not to an ordinary passenger. So:

- If `weight.md`'s "75 kg per modeled passenger" is describing ordinary bus
  passengers, it is **7 kg above** the Class II/III regulatory figure (71 kg)
  and **7 kg above** the Class I/A figure (68 kg) — a discrepancy of roughly
  +10% to +9.9%, respectively.
- 75 kg exactly matches the regulatory mass for the **driver/crew**, not for
  passengers. It is possible the dataset's 75 kg figure was carried over
  from the driver/crew convention, or from a different (non-bus-specific,
  e.g. generic ISO/M1 passenger-car) convention, rather than the M2/M3-specific
  passenger table.
- Both figures now exist as documented facts: the project's stated 75 kg,
  and the regulation's 68/71 kg. This report does not decide which the
  project should use — that is a decision for the dataset maintainer to make
  with this discrepancy in view.

## What this means for a payload (Mb) calculation, if the project wants to
## follow the regulation exactly (informational — not a recommendation)

Per the fetched regulatory text, the regulatory "mass of the passengers" for
an M2/M3 vehicle would be: (number of seats, including crew seats) × Q,
using Q = 68 kg for Class I/A or Q = 71 kg for Class II/III/B, PLUS, if
standing is permitted (Class I/II/A only), (number of rated standees) × Q,
PLUS a minimum luggage mass of 100 kg per m³ of baggage-compartment volume
(B = 100 × V), where V is manufacturer-stated. Crew seats, if any beyond the
driver, use 75 kg each; the driver (75 kg) is part of unladen/running-order
mass, not part of the payload in most conventions, though the regulation's
"mass of the passengers" definition explicitly includes crew seating
positions in the passenger count at their own 75 kg rate — the driver's own
75 kg is already inside "mass of vehicle in running order," not inside
"mass of the passengers." This distinction affects whether a driver's 75 kg
should be added to curb mass or subtracted from payload in the project's own
q = curb / payload formula; that is a modeling decision, not something the
regulation states in "payload" terms (the regulation never uses "payload,"
only "pay-mass" for goods vehicles, and "technically permissible maximum
laden mass" for buses/coaches — see below).

**Important scoping note**: the regulation does not define a bus/coach
"payload" as a labeled quantity in the way it defines truck "pay-mass"
(definition 21, N-category only: "'pay-mass' means the difference between
the technically permissible maximum laden mass and the mass in running
order increased by the mass of the passengers and the mass of the optional
equipment"). For M2/M3 vehicles the regulation only requires that laden mass
(driver + passengers + crew + optional equipment + minimum luggage) not
exceed the technically permissible maximum laden mass — it does not itself
hand you a single "payload" number; that has to be derived (as the project
apparently already does) as gross mass minus curb mass, using the component
masses above.

## Files/URLs fetched this session (for audit trail)

- `https://www.legislation.gov.uk/eur/2012/1230/2019-12-02/data.xht` — fetched successfully via curl, saved locally as `/tmp/topo/leg1230.xht` (637,508 bytes), parsed to `/tmp/topo/leg1230.txt`.
- `http://www.legislation.gov.uk/eur/2012/1230/images/eur_20121230_2019-12-02_en_019` — formula image, fetched successfully, read directly (B = 100 × V).
- `https://www.fia.uk.com/static/d965dea0-6278-4f8a-b580ee29126ea56d/FACT-FILE-89-UNECE-Regulation-107.pdf` — fetched successfully via WebFetch tool, converted with `pdftotext`.
- `https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX%3A02012R1230-20170727` — blocked/empty.
- `https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A02012R1230-20170727` — blocked/empty (WebFetch).
- `https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:02012R1230-20191202` — HTTP 202, WAF challenge, 0 bytes.
- `https://unece.org/sites/default/files/2021-05/R107r7e.pdf` — Cloudflare JS challenge page returned instead of PDF.
- `https://unece.org/transport/documents/2021/05/standards/un-regulation-no-107-rev7` — HTTP 403 (Cloudflare).
- `https://www.vehicle-certification-agency.gov.uk/vehicle-type-approval/what-is-vehicle-type-approval/type-approval-category-definitions/` — blocked, "IP Address is blocked. Reason: Malware Detected."
- `https://wiki.unece.org/download/attachments/113345566/STCBC-04-02%20Bus%20Clasification_rev1.pdf?api=v2` — blocked/HTML challenge page instead of PDF.
- `https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX%3A42018X0237&from=DE` — blocked/empty.
