# Normalization – from the raw donor file to 3NF

Source: [data/raw/transfusion.data](../../data/raw/transfusion.data) (748 donors). Quality report: [data/dataset_card.md](../../data/dataset_card.md).

The raw file is one wide table. Each row summarises a donor's whole history in five numbers, so the real structure is hidden inside those numbers:

- `Frequency` = 13 means the donor made **13 donations**. That is a list of donations (a repeating group) packed into one count.
- `Time` and `Recency` give the months of the **first** and **last** of those donations.
- `Monetary` is just `Frequency × 250` c.c.
- The last column is the result of one **recall campaign** (March 2007) for this donor.

The same four donors are followed through every step. Their source rows are 1, 2, 3 and 5:

```
row 1:  2 ,50,12500,98 ,1
row 2:  0 ,13,3250,28 ,1
row 3:  1 ,16,4000,35 ,1
row 5:  1 ,24,6000,77 ,0
```

Assumptions used to turn months into dates (used again in Phase IV):

- **Reference date:** 1 March 2007 (the "March 2007" in the data). A donation *k* months before it is dated reference − *k* months.
- **Spacing:** the first donation is `Time` months back and the last is `Recency` months back. The other donations are spaced evenly between them and rounded to whole months. These middle dates are estimates, flagged with `is_date_estimated = Y`.
- **Site:** the file does not record where each donation was made, so every historical donation goes to the Hsin-Chu centre (site 1).

---

## UNF – unnormalised

One row per donor, with the donations as a repeating group (`donations` holds a list, not a single value):

| donor (row) | Recency | Frequency | Monetary | Time | donated Mar 2007 | donations (months before ref.) | site |
|---|---|---|---|---|---|---|---|
| 1 | 2 | 50 | 12500 | 98 | 1 | 98, 96, …, 4, 2 (50 values) | Hsin-Chu centre, Hsin-Chu City |
| 2 | 0 | 13 | 3250 | 28 | 1 | 28, 26, …, 2, 0 (13 values) | Hsin-Chu centre, Hsin-Chu City |
| 3 | 1 | 16 | 4000 | 35 | 1 | 35, 33, …, 3, 1 (16 values) | Hsin-Chu centre, Hsin-Chu City |
| 5 | 1 | 24 | 6000 | 77 | 0 | 77, 74, …, 4, 1 (24 values) | Hsin-Chu centre, Hsin-Chu City |

Problems: the donations column holds many values, so it cannot be searched, counted or linked to blood units. Totals such as `Monetary` repeat information already in `Frequency`.

## 1NF – one value per cell

**Removed: the repeating group.** The list of donations becomes one row per donation, so every column holds a single value. Key: (`donor_id`, `donation_no`).

| donor_id | donation_no | donation_date | volume_cc | Recency | Frequency | Monetary | Time | campaign | campaign_month | donated_in_campaign | site_name | site_city |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 1 | 1999-01-01 | 250 | 2 | 50 | 12500 | 98 | March 2007 recall | 2007-03 | Y | Hsin-Chu centre | Hsin-Chu City |
| 1 | 50 | 2007-01-01 | 250 | 2 | 50 | 12500 | 98 | March 2007 recall | 2007-03 | Y | Hsin-Chu centre | Hsin-Chu City |
| 2 | 1 | 2004-11-01 | 250 | 0 | 13 | 3250 | 28 | March 2007 recall | 2007-03 | Y | Hsin-Chu centre | Hsin-Chu City |
| 2 | 13 | 2007-03-01 | 250 | 0 | 13 | 3250 | 28 | March 2007 recall | 2007-03 | Y | Hsin-Chu centre | Hsin-Chu City |
| 5 | 24 | 2007-02-01 | 250 | 1 | 24 | 6000 | 77 | March 2007 recall | 2007-03 | N | Hsin-Chu centre | Hsin-Chu City |

The 748 donor rows become 4,125 donation rows. Donor values such as `Frequency` and `Time` now repeat on every donation row.

## 2NF – remove partial dependencies

**Removed: partial dependencies on (`donor_id`, `donation_no`).**

- `Recency`, `Frequency`, `Monetary`, `Time`, `campaign`, `campaign_month` and `donated_in_campaign` depend on `donor_id` alone, not on which donation it is. They move to **DONOR**.
- `donation_date`, `volume_cc` and the site depend on the whole key, so they stay in **DONATION**. DONATION gets its own key, `donation_id`.

DONOR (2NF):

| donor_id | Recency | Frequency | Monetary | Time | campaign | campaign_month | donated_in_campaign |
|---|---|---|---|---|---|---|---|
| 1 | 2 | 50 | 12500 | 98 | March 2007 recall | 2007-03 | Y |
| 2 | 0 | 13 | 3250 | 28 | March 2007 recall | 2007-03 | Y |
| 3 | 1 | 16 | 4000 | 35 | March 2007 recall | 2007-03 | Y |
| 5 | 1 | 24 | 6000 | 77 | March 2007 recall | 2007-03 | N |

DONATION (2NF):

| donation_id | donor_id | donation_date | volume_cc | site_name | site_city |
|---|---|---|---|---|---|
| 1 | 1 | 1999-01-01 | 250 | Hsin-Chu centre | Hsin-Chu City |
| 50 | 1 | 2007-01-01 | 250 | Hsin-Chu centre | Hsin-Chu City |
| 51 | 2 | 2004-11-01 | 250 | Hsin-Chu centre | Hsin-Chu City |
| 63 | 2 | 2007-03-01 | 250 | Hsin-Chu centre | Hsin-Chu City |

## 3NF – remove transitive dependencies

1. **`Monetary` (transitive dependency).** `donor_id → Frequency → Monetary`: total volume is determined by the number of donations (× 250 c.c.), so it is dropped. The volume of each donation is kept in DONATION.
2. **`Recency`, `Frequency`, `Time` (derived values, not a dependency split).** They can be computed from the donor's DONATION rows: months since the latest date, COUNT, and months since the earliest date. Storing them would let them disagree with the donation rows, so they are not stored.
3. **Campaign (transitive dependency).** `donor_id → campaign → campaign_month`: the month belongs to the campaign, not to the donor, so the campaign moves to **RECALL_CAMPAIGNS**. One donor can be recalled in many campaigns and one campaign recalls many donors, so this many-to-many relationship is resolved with the junction table **DONOR_RECALLS**. The outcome (`donated_in_campaign`) belongs to the pair, so it moves there.
4. **Site (transitive dependency).** `donation_id → site_name → site_city`: the city depends on the site, not on the donation, so the site moves to **DONATION_SITES** and DONATION keeps only `site_id`.

DONORS (3NF):

| donor_id | source_row_no | status |
|---|---|---|
| 1 | 1 | ACTIVE |
| 2 | 2 | ACTIVE |
| 3 | 3 | ACTIVE |
| 5 | 5 | ACTIVE |

DONATIONS (3NF):

| donation_id | donor_id | site_id | donation_date | volume_cc | is_date_estimated |
|---|---|---|---|---|---|
| 1 | 1 | 1 | 1999-01-01 | 250 | N |
| 2 | 1 | 1 | 1999-03-01 | 250 | Y |
| 51 | 2 | 1 | 2004-11-01 | 250 | N |
| 63 | 2 | 1 | 2007-03-01 | 250 | N |

DONATION_SITES (3NF):

| site_id | site_name | city | site_type |
|---|---|---|---|
| 1 | Hsin-Chu Blood Transfusion Service Center | Hsin-Chu City | CENTRE |
| 2 | University blood bus | Hsin-Chu City | MOBILE |

RECALL_CAMPAIGNS (3NF):

| campaign_id | campaign_name | campaign_type | start_date |
|---|---|---|---|
| 1 | March 2007 recall | ROUTINE | 2007-03-01 |

DONOR_RECALLS (3NF, junction between DONORS and RECALL_CAMPAIGNS):

| recall_id | donor_id | campaign_id | outcome |
|---|---|---|---|
| 1 | 1 | 1 | DONATED |
| 2 | 2 | 1 | DONATED |
| 3 | 3 | 1 | DONATED |
| 5 | 5 | 1 | NOT_DONATED |

### Blood units: a list inside each donation

Each whole-blood donation is separated into components, so one donation holds a **list of units** (a repeating group). The list becomes the child table **BLOOD_UNITS**.

**Removed: transitive dependency.** `unit_id → component → shelf_life_days`: shelf life depends on the component, not on the unit, so components move to **COMPONENT_TYPES**.

`expiry_date` is kept in BLOOD_UNITS on purpose. It is fixed when the unit is made, and must not change if a shelf-life rule changes later.

COMPONENT_TYPES (3NF):

| component_code | component_name | shelf_life_days |
|---|---|---|
| RBC | Red blood cells | 42 |
| PLT | Platelets | 5 |
| FFP | Fresh frozen plasma | 365 |

BLOOD_UNITS (3NF), units from donation 63 (donor 2, 2007-03-01):

| unit_id | donation_id | component_code | expiry_date | status |
|---|---|---|---|---|
| 1 | 63 | RBC | 2007-04-12 | AVAILABLE |
| 2 | 63 | PLT | 2007-03-06 | AVAILABLE |
| 3 | 63 | FFP | 2008-02-29 | AVAILABLE |

Shelf lives are the standard values planned since Phase 0 and are checked again in Phase IV.

### Operational tables (filled while the system runs, not from the raw file)

These come from the Phase I business process. They are already in 3NF because each one describes a single thing:

- **HOSPITALS:** who requests blood. Name and district depend on the hospital, so they are not stored on each issue.
- **UNIT_ISSUES:** one row when a unit is issued to a hospital.
- **EXPIRY_ALERTS:** units flagged by the nightly job. "Days left" is not stored because it is derived from `expiry_date`.
- **HOLIDAYS:** dates used by the Phase VI restriction rule.
- **AUDIT_LOG:** who did what, when, and whether it was allowed.

Final design: **12 tables**. Seven are filled from the raw data (DONORS, DONATIONS, DONATION_SITES, RECALL_CAMPAIGNS, DONOR_RECALLS, COMPONENT_TYPES, BLOOD_UNITS) and five are operational. See [erd.png](erd.png) and [data_dictionary.md](data_dictionary.md).

---

## Field mapping and cleaning checklist (for Phase IV)

| Raw field | Target table.column | How | Type conversion needed |
|---|---|---|---|
| (row position 1–748) | `DONORS.source_row_no` | Kept so any donor can be traced back to its raw row | number |
| `Recency (months)` | `DONATIONS.donation_date` of the **last** donation | Reference date − Recency months; not stored itself (derived) | trim trailing space → NUMBER → DATE |
| `Frequency (times)` | Number of `DONATIONS` rows created for the donor | One row per donation; not stored itself (COUNT) | text → NUMBER |
| `Monetary (c.c. blood)` | Dropped | Derived: always Frequency × 250 (checked on all 748 rows). Gives `DONATIONS.volume_cc = 250` | text → NUMBER (only to check) |
| `Time (months)` | `DONATIONS.donation_date` of the **first** donation | Reference date − Time months; middle donations spaced evenly and flagged `is_date_estimated = Y` | trim trailing space → NUMBER → DATE |
| `whether he/she donated blood in March 2007` | `DONOR_RECALLS.outcome` for campaign "March 2007 recall" | 1 → DONATED, 0 → NOT_DONATED | 0/1 → text code |
| Header row | Dropped | Column names are replaced by the names above | – |

Other Phase IV cleaning items from the quality report:

- **215 identical rows:** keep them as separate donors. There is no ID to prove they are the same person, and each row keeps its own `source_row_no`.
- **48 donors whose average gap is under 56 days:** load them as they are, and list them in a validation report. The eligibility trigger only applies to new donations.
- **Every historical donation is assigned to site 1,** because the file doesn't record the site.

## Demo: one record from raw to 3NF

Raw row 2: `0 ,13,3250,28 ,1`

| Raw value | Ends up in |
|---|---|
| row number 2 | `DONORS` row: donor_id 2, source_row_no 2 |
| `Time` = 28 | `DONATIONS` row 51: first donation, 2004-11-01 |
| `Frequency` = 13 | 13 `DONATIONS` rows (ids 51–63), each 250 c.c. at site 1 |
| `Recency` = 0 | `DONATIONS` row 63: last donation, 2007-03-01 |
| `Monetary` = 3250 | Not stored; equals 13 × 250, the sum of `volume_cc` |
| March 2007 = 1 | `DONOR_RECALLS`: donor 2, campaign 1, outcome DONATED |
| (donation 63) | `BLOOD_UNITS` 1–3: RBC, PLT and FFP with their expiry dates |
