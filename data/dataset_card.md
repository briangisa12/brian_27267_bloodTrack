# Dataset card – Blood Transfusion Service Center

## Source

| Item | Value |
|---|---|
| Name | Blood Transfusion Service Center |
| Publisher | UCI Machine Learning Repository |
| Creator | I-Cheng Yeh, Chung-Hua University, Hsin-Chu, Taiwan |
| Link | https://archive.ics.uci.edu/dataset/176/blood+transfusion+service+center |
| DOI | https://doi.org/10.24432/C5GS39 |
| Licence | CC BY 4.0 (sharing and adaptation allowed for any purpose, with credit) |
| Format | Comma-separated text (`transfusion.data`), header row, Windows (CRLF) line endings |
| Size | 748 records × 5 fields (12,843 bytes) |
| Date collected | Donor records up to March 2007; published to UCI on 3 October 2008; downloaded on 4 November 2026 |
| Raw files | [raw/transfusion.data](raw/transfusion.data) (data) and [raw/transfusion.names](raw/transfusion.names) (original description) |
| SHA-256 of `transfusion.data` | `96c8e1091b9c037bcaf25a19b24b49d07771cd88689ffd273e056e9e8845ffe7` |

The raw files are kept exactly as downloaded. `.gitattributes` stops Git from changing their line endings.

## What one record represents

One row is **one blood donor**, picked at random from the donor database of the Blood Transfusion Service Center in Hsin-Chu City, Taiwan. The centre also sends its blood bus to one university in the city about every three months. Each row summarises that donor's whole donation history, measured from March 2007.

| Field (header as written) | Meaning | Unit |
|---|---|---|
| `Recency (months)` | Months since the donor's last donation | months |
| `Frequency (times)` | Total number of donations | count |
| `Monetary (c.c. blood)` | Total blood donated | c.c. |
| `Time (months)` | Months since the donor's first donation | months |
| `whether he/she donated blood in March 2007` | 1 = donated in March 2007, 0 = did not | flag |

## Personal data

The records have no names, phone numbers, ID numbers, addresses or birth dates, and no donor ID. They are already anonymous, so nothing needed to be removed. The description file `transfusion.names` contains the dataset author's published academic contact details. They are not donor data and are left unchanged as part of the original download.

## Data quality report (counts, nothing fixed yet)

Checked on all 748 records. Cleaning happens in Phase IV.

| # | Check | Result |
|---|---|---|
| 1 | Records / fields | 748 records, 5 fields, every row has exactly 5 values |
| 2 | Missing values | 0 |
| 3 | Values with a trailing space (stored as text such as `"2 "`) | 1,496: all 748 `Recency` values and all 748 `Time` values |
| 4 | Header names with spaces, brackets or quotes | 5 of 5 (not usable as column names) |
| 5 | Identical rows (all 5 values the same) | 215 extra copies; only 533 distinct rows. There is no donor ID, so these are most likely different donors with the same history, not real duplicates |
| 6 | `Monetary` ≠ `Frequency` × 250 | 0 of 748. `Monetary` is fully derived (every donation is 250 c.c.) |
| 7 | `Time` smaller than `Recency` (first donation after last) | 0 |
| 8 | `Frequency` = 1 but `Recency` ≠ `Time` | 0 |
| 9 | Average gap between donations shorter than 56 days | 48 donors. Months are rounded, so these histories break the 56-day rule on paper |
| 10 | Precision | `Recency` and `Time` are whole months here, but the original paper reports decimals (minimum `Recency` 0.03, `Time` 2.27), so precision was lost |
| 11 | Dates | No real dates: every time value is "months before March 2007" |
| 12 | Donation site | Not recorded per donation (centre or university bus unknown) |
| 13 | Ranges | `Recency` 0–74, `Frequency` 1–50, `Monetary` 250–12,500, `Time` 2–98 |
| 14 | Target balance | 178 donated in March 2007 (23.8%), 570 did not |
| 15 | Total donations described | 4,125 (sum of `Frequency`) |

## Sample records (first 3 rows, as stored)

```
2 ,50,12500,98 ,1
0 ,13,3250,28 ,1
1 ,16,4000,35 ,1
```

## Where the data goes

How each field becomes 3NF tables, and the type conversions needed, is in [../docs/design/normalization.md](../docs/design/normalization.md).
