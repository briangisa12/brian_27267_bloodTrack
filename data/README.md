# Data source

| Item | Value |
|---|---|
| Name | Blood Transfusion Service Center |
| Publisher | UCI Machine Learning Repository |
| Creator | I-Cheng Yeh |
| Link | https://archive.ics.uci.edu/dataset/176/blood+transfusion+service+center |
| DOI | https://doi.org/10.24432/C5GS39 |
| Licence | CC BY 4.0 (educational use allowed with credit) |
| Origin | Donor database of the Blood Transfusion Service Center, Hsin-Chu City, Taiwan |
| Size | 748 rows (donors), 5 columns, no missing values |
| File | `transfusion.data` (CSV with header) |
| Date downloaded | 2026-11-04 |

## Columns

| Column | Meaning | Unit |
|---|---|---|
| Recency | Months since the donor's last donation | months |
| Frequency | Total number of donations | count |
| Monetary | Total blood donated | c.c. |
| Time | Months since the donor's first donation | months |
| Donated in March 2007 | Whether the donor came back in March 2007 (1 = yes, 0 = no) | binary |

## Personal data check

The file has no names, phone numbers, emails, addresses or ID numbers. Each row is an anonymous donor.

## Notes for normalization (Phase II)

- `Monetary` is always `Frequency × 250` c.c., so it is a **derived** value and will not be stored as a column.
- The dataset describes donors, not individual blood units. Donation dates will be rebuilt from `Recency`, `Frequency` and `Time`, and blood units will be created from each donation using standard shelf-life rules (stored in `COMPONENT_TYPES`). These generated rows will be clearly labelled as derived from the real donor data.

## Sample records

| Recency (months) | Frequency | Monetary (c.c.) | Time (months) | Donated March 2007 |
|---|---|---|---|---|
| 2 | 50 | 12500 | 98 | 1 |
| 0 | 13 | 3250 | 28 | 1 |
| 1 | 16 | 4000 | 35 | 1 |

## Checks done on the file

- 748 data rows plus 1 header row, 5 columns
- `Monetary = Frequency × 250` on all 748 rows (verified)
- 178 donors (23.8%) returned in March 2007; 570 did not
- `transfusion.names` is the original description file from UCI
