# Blood Donor Recall and Unit Expiry Tracker

**Student:** Brian Ganza GISAGARA
**Student ID:** 27267
**Course:** INSY 8311 – Database Development with PL/SQL (Group C)
**Instructor:** Eric Maniraguha
**Academic year:** 2026–2027, Semester I

---

## Problem statement

Blood banks have two opposite problems at the same time:

1. **Waste.** Donated blood does not last. Red cells keep for about 42 days and platelets for only about 5 days. Units that are not issued before their expiry date are thrown away.
2. **Shortage.** Stock still runs low, because the blood bank does not know which past donors are eligible to donate again and which of them are likely to come back.

When donor records and unit stock are kept on paper or in disconnected spreadsheets, nobody sees which units are about to expire or which donors should be called back.

## Objectives

**Main objective:** build an Oracle database with PL/SQL that reduces blood unit waste and improves donor return by tracking donors, donations and blood units in one place.

**Specific objectives:**

1. Store donors, donations, blood components and units in normalized (3NF) tables loaded from a real dataset.
2. Identify donors who are eligible again (the 56-day waiting period has passed) and rank them by how likely they are to return, using their donation history (recency, frequency, time since first donation).
3. Enforce stock rules in PL/SQL: an expired unit can never be issued, and units are issued oldest-safe-first (FEFO – first expiring, first out).
4. Automatically flag units expiring within 3 days and log every sensitive action in an audit table.

## Data source

- **Dataset:** Blood Transfusion Service Center – UCI Machine Learning Repository
- **Link:** https://archive.ics.uci.edu/dataset/176/blood+transfusion+service+center
- **DOI:** https://doi.org/10.24432/C5GS39
- **Size:** 748 donors, 5 columns, no missing values
- **Licence:** Creative Commons Attribution 4.0 (CC BY 4.0) – allows educational use with credit
- **Personal data:** none (no names, contacts or identifiers)

Details and column descriptions are in [data/README.md](data/README.md).

## Innovation

**Option:** Smart automation (DBMS_SCHEDULER, alerts)

A nightly `DBMS_SCHEDULER` job will:

- build the **donor recall list**: eligible donors ranked by a return-likelihood score calculated in PL/SQL from their donation history;
- raise **expiry alerts** for every blood unit that expires within the next 3 days.

The business rules stay in PL/SQL; the scheduler only runs them on a timetable. It will be shown live in the final presentation and documented in `innovation/`.

## Planned tables (draft – finalised in Phase II)

| Table | Purpose |
|---|---|
| `DONORS` | One row per donor (from the dataset) |
| `DONATION_SITES` | Places where blood is collected |
| `DONATIONS` | Each donation event: donor, site, date, volume |
| `COMPONENT_TYPES` | Red cells, platelets, plasma… with shelf life in days |
| `BLOOD_UNITS` | Each unit produced from a donation, with expiry date and status |
| `UNIT_ISSUES` | When a unit is issued and to which facility |
| `DONOR_RECALLS` | Recall list entries: donor, date, score, outcome |
| `EXPIRY_ALERTS` | Units flagged as expiring soon |
| `HOLIDAYS` | Public holidays used by the restriction rule |
| `AUDIT_LOG` | Who did what, when, and whether it was allowed |

Main relationships: a donor has many donations; a donation produces one or more units; a unit has one component type; a unit is issued at most once.

## Planned PL/SQL features

| Feature | Where it is used |
|---|---|
| Procedure | `record_donation` adds a donation and creates its blood units with the correct expiry date |
| Procedure | `issue_unit` issues the oldest safe unit of the requested component (FEFO) |
| Function | `donor_return_score` returns a score from recency, frequency and time since first donation |
| Function | `is_donor_eligible` returns whether 56 days have passed since the last donation |
| Cursor | Loops over eligible donors to build the nightly recall list |
| Package | `blood_bank_pkg` groups the donation, issue and recall procedures and functions |
| Trigger | Blocks issuing any unit whose expiry date has passed |
| Trigger | Blocks a donation from a donor who is not yet eligible again |
| Compound trigger | Writes every INSERT/UPDATE/DELETE on units and issues to `AUDIT_LOG` |
| Restriction rule | Blocks changes on weekdays and public holidays (course rule, Phase VI) |
| Exceptions | Custom errors such as "unit expired" and "donor not eligible" |
| Scheduler | Nightly job for recall list and expiry alerts (innovation) |

## Repository structure

```
docs/research/       research log and sources
docs/proposal/       2-page proposal (.docx)
docs/presentation/   Phase 0 slides
data/                dataset and its README
```

Later phases add more folders (SQL scripts, diagrams, innovation).

## References

- Yeh, I.-C. (2008). *Blood Transfusion Service Center* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5GS39

The full list of sources is in [docs/research/research_log.md](docs/research/research_log.md).
