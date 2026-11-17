# Blood Donor Recall and Unit Expiry Tracker

**Student:** Brian Ganza GISAGARA

**Student ID:** 27267

**Course:** INSY 8311 – Database Development with PL/SQL (Group C)


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

Details, column descriptions and the data quality report are in [data/dataset_card.md](data/dataset_card.md). The raw file is in [data/raw/](data/raw/).

## Innovation

**Option:** Smart automation (DBMS_SCHEDULER, alerts)

A nightly `DBMS_SCHEDULER` job will:

- build the **donor recall list**: eligible donors ranked by a return-likelihood score calculated in PL/SQL from their donation history;
- raise **expiry alerts** for every blood unit that expires within the next 3 days.

The business rules stay in PL/SQL; the scheduler only runs them on a timetable. It will be shown live in the final presentation and documented in `innovation/`.

## Tables (logical design, finalised in Phase II)

| Table | Purpose |
|---|---|
| `DONORS` | One row per donor (from the dataset) |
| `DONATION_SITES` | Places where blood is collected |
| `DONATIONS` | Each donation event: donor, site, date, volume |
| `COMPONENT_TYPES` | Red cells, platelets, plasma… with shelf life in days |
| `BLOOD_UNITS` | Each unit produced from a donation, with expiry date and status |
| `HOSPITALS` | Facilities that request blood |
| `UNIT_ISSUES` | When a unit is issued and to which hospital |
| `RECALL_CAMPAIGNS` | A round of calling donors back (nightly list or urgent recall) |
| `DONOR_RECALLS` | Junction: donor recalled in a campaign, with score and outcome |
| `EXPIRY_ALERTS` | Units flagged as expiring soon |
| `HOLIDAYS` | Public holidays used by the restriction rule |
| `AUDIT_LOG` | Who did what, when, and whether it was allowed |

Main relationships: a donor has many donations; a donation produces one or more units; a unit has one component type; a unit is issued at most once, to one hospital; donors and campaigns are many-to-many through `DONOR_RECALLS`. Full details are in [docs/design/](docs/design/).

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
docs/business_process/  Phase I diagram and explanation (PDF)
docs/design/         Phase II normalization, ERD and data dictionary
data/raw/            raw dataset exactly as downloaded
data/dataset_card.md source, licence, size and quality report
```

Later phases add more folders (SQL scripts, diagrams, innovation).

## Phase I - Business Process Modeling

**Selected process:** donor recall, blood collection, stocking and issue of blood units. Every night a DBMS_SCHEDULER job builds a ranked recall list of donors who are eligible again (56+ days) and flags units expiring within 3 days. The recall officer calls donors; the collection nurse checks eligibility and records the donation; the lab technician tests it and creates dated blood units; the inventory officer stores them, removes expired units and issues the unit that expires first when a hospital asks for blood. The blood bank manager reviews recall outcomes, stock and expired units each morning to decide on donation campaigns.

The diagram uses 8 swimlanes (Oracle system, recall officer, donor, collection nurse, lab technician, inventory officer, requesting hospital, blood bank manager), 5 decision points, and marks every Create, Retrieve, Update and Delete/archive activity with the table it touches.

- **Modeling tool:** draw.io (diagrams.net)
- **Published diagram:** [open the published diagram](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=business_process_diagram.io&dark=auto#R%3Cmxfile%3E%3Cdiagram%20name%3D%22Business%20process%22%20id%3D%22KwGAdX82bBuHPjgOklQg%22%3E7V3Zcts4Fv0aVbkfeoq7qEdtSXvKHaesuGf6KQWRkIQJRahJyrLz9Y2NFEXCbkRFcInHqdgUuArnrgfA5cie758%2FJuCw%2Bx2HMBpZRvg8shcjy7It3yd%2FaMuLaDFMh7dsExTyNvPcsELfoWg0ROsRhTC9ODDDOMrQ4bIxwHEMg%2ByiDSQJPl0etsHR5V0PYAtrDasARPXW%2F6Aw2%2FFW3zXO7b9BtN3ldzYNsWcP8oNFQ7oDIT6VmuzlyJ4nGGd8a%2F88hxHtvbxf%2BHkfXtlbPFgC40zlhNu%2FsLP%2Ffvrfx9n83nx8%2FP4FBMmvkquIpjR7yfsgg89k32yX7SPSYJLNDY4zgRW5q%2FgsTqD7QYS2MdmO4Iae%2BQSTDJE%2BnYrmPQpDeuyM3%2BoJREdxq1mEcUiaFjjGCfn7AMlp9FuAmDY%2Fxigjf5bPB5S8kI0vCQi%2BQXrgaGmNfGM0scn27JiiGKYp%2FWIJDuiWPaVCKC6aiIvOmdxEEZEbhGP%2BOc1w8A3F2%2BKWKE3Jw1kG3pBfa%2FF4R%2FIcqXh48t3gc6nrRO9%2FhHgPM%2FaUu5KAOEIaTmdhMr1cZoSS%2FOqJz0AI77a41hlfsiEg%2FgG4LRW40xPaRyCm%2BOxwgr4TbAEFngKdZiDJkXcMiVCUhKAkJKZNP6MomuOIQEBvY2%2BCjbdx6TXF%2FT5U9rMfds8Ef4OlPZOJYZimTHzuiTxEFK30Jc3gnmzcLGa%2Fr76u5r8tF493ywcqKdaMGpEEbbcwSX%2B5CkXTr8NomxNXE2x2Hba34C1hSS5OLCW8BAqkB24qN%2BgZhnVkQhf6oSPred9a255X7MktIumuWQTWMPqMU8R0yV4E5FGJZp6V%2F65ywBpnGd6XjEXtjNxcZPhQESdDBv4nik9EocrQnhmFG6JY9tQwrgNZoqp5U34R81JxbV1661wtAAk%2BxiEFmSF%2F2qEMrg4goHtPxGW%2FataZflXlAkB%2FE8jkwgt8uN7IQBlZXkRdwJpsbOnGyJ09jNxF3k6647zLMj4gZnSZpab2GxIRQGum0mALUExBdT2uwiF4oeor7HSA94djBpl5z44JPTINcAKLB0jOt%2BEtKG9Y3H%2B6f1hx%2B0%2B2p19u7z%2BtSg%2BIzmdeIUaeU7f4FTmyLPdCjixdcuT2Q47%2B2b4oydH8dTmaJxBwYQDxNxiWnL4RoTRTFYqvD8v59O6uVWGwJ0Y7wuD1QxhaMyoiaDMgjR55iHdC2Y4ZFVvYEwXBmN3d3y%2B%2BPn66%2FdKqWLhuSzZi3A%2BxaNFGwDydIFlfkqnIwPK%2Fn28f%2Fvw6JRFlu1IwtloyDn67IefGD2AgtQJFsH8Zcjq9CDn%2FjYlAGRsUo3RHvYyWKHNiX2q%2BtihzUse8u%2BzQpf%2FeyA4B%2Ffdj2WFBJuDNBgU0Q2g89XupHNQ4RqZRB%2BktMIfrteccqyIVoAyNke1gPZajcfuOIATphzzqJ1qZ9DvQa82jmxKWrwOZ2WygJ7fx4XiyNqQGViYzj6%2FLDNFwnNBYDx8zkhBCwfvBIEIxywJoR1pGjFnamJ6oDeivgLTm7E0JL6goIO%2FJ2xf%2BI4hwOnx3b0poxZ%2BKDRaDCLqcvK2NpjdV%2Bb4cwrLF3uH9%2Bpj%2BmLU2ZNZ6Y71irb2150oTsOk2gZD66gxzz03zK%2FvDVRCMvbp9tKv28RKQiaMLD1XeTIZHox70NSXw2I8MkznpgwISHj4xYNiIl0GMJdTlvny%2FEhQb2vRFQmX9VJnLvDRKacTHJL0ONCWz5jraYFKllnI8h5u8PMAtSU3YINQTStmgtRim2MHg2%2Bg8Fr0jh2Hat%2F0fpajqs75wVJV8kslJh%2B5vmQ9WUZfHx6kEqcwyVAgi0rVCBK4bklRwiqZRSStNbV5Rwhd1oc5t5pUh3MAkAREjIUBK7bGy4ranqUQGLjNHfa7XUuWjJELwnnLHfEITFyBtyaNpeRXtN7Uhr8oq6fbmLYwU8fBrdJ4Axp15kpuFalytYhPad%2BCmZbbkwS0JofRTUQvEXlDYYbCLUYBArC8cH%2BsDSXlamTX0cPwLHxrgWptC8o35yC%2BKWVpM5w7hmHSCythvN6pr%2By1xwZbyXDOJVHQYfK%2BCBMKYT%2Bs4gJTys%2FqYJ9OtBli6gmxLecqWXiV9Pexa%2B67jKgfZi9eVdIHSAEjdqXEDkmCHniDvYXLo7Wo%2BfVgsF7%2F0d8KOSbqlrSBceSpXXUjeVRDOJUxf9D32KmZBX%2FStPE1Ls%2FNucZ5WZfUFm71H%2FhTTt0J%2B2CbBdOZ%2FuoMRXbMRoY3KJOCurMTYbMuxS1i1n4okv42fyBdjZKr%2BGT4TfcZclVbLAR0urbbK%2BFQdodB8%2FRXIjlS5p39Mb%2B%2Bms7tlj3XXH7eUT9uqLJtEJt6Tg79lY5p0CZ8mD28Zldn52mZn2Kr0mmYz0Mpg2ROCp7InFxOx2d49ZnF%2FDEHya3FAHgKURrWJ249V5vt1Nne7RVbelvBvnfgQ%2FVnjAxQCwmQDluPDatbIkO93zlhlaDWKiDL7975dykqkEWzUVl%2Fm6NhtZY62MsE3eMciFn5x%2F0Aug3LrwBpBVlgNah3EfHHj5sPyw32PbQSJQdqKOpW5R4mkdEgF%2F3GJNHgCiNgRMS9DDyFs2eOWZl3YylyfZvXVT%2FzcitIbAscbUbNhxJig4FuxlvPs9FXUlirs19vV6nFJ1FZUg3gk%2F7tSZ9duS52VKcPBS04xXyfd4SSjJX7ywT%2BQ0tl3GzYX45hsaUec1xP1dykIsS%2Bt5Q7KSz%2Ffe2BYiFYCD2RbW3BoudWZ1PqCw16tAdUwceMB%2FnWEacbHiXc4PaCMzea7gc9EGmIQ6SgIVOistmWhjipBmAP8TmsEfQYZ4jY%2FhjBMR8Wgkp6sbmK1tJbLUWULJfh3vUJlBcVkOqaZVBeL%2BTh8TOCvI4gzlL1cp5oq3tVwLr2rthjMUSXmeogTCaogZ9PyxIpn1wmI083xyiUpKuiMjbbQUeXEmrKiAw19HgpVJf2uK%2BiZmG3ZTgkh9lMNjOdFNdcgpsnPHsRgq7X6haVtZayjSknloL7TOGdJzqPY7XHC50Tq0VGnUqFAn46qsl4S3AdGWhejoTyr5CUsioonorBFWqpWyzdrw18%2FSmbMqxwYbZg%2BLm6%2FfL27%2F%2FhzUmKOKiUmE6suJztz3GkpbPI7omWl9VHbY6MlattRZZ40K%2Fk1wfHniC4CMcDoYt5yAPYHQA23Lk3xvbY0RXVWWlOOd6Ch8QIGKOXg70F4XU70z47XNvyWHK%2BrSiy9c9w%2FsZJSokRDOwmRPhbYFekPDGsvR6inSPiYBPCNi9VLotDL5ikRiXB2eEssZrQ8t84urfkbvhTG4ZS%2B5YFCSscEeRPNmcQJNSniPwWEJHPbwjflWhLPycBMYESs%2FtNlfzWPjNUgMhKNHRIybr%2BQsRtExh02Ml6%2FkHEaREZSxWBIyIz7hYzbIDLjYSPj9wsZb4DWDB9gfAEM3R8CVpr5FZSKmJJe9nUy9R%2FRy0OxEnyVIrUdYjluEMv8iw5VzcyexXN%2Bk9gMPKDL6xRX0qoO0Zk0iU59TGlY6Fi90px8iOn%2FmlMqUFrC5k%2BYdgpPkxSCOfB8yOxX2O01SSKYA4%2B7zX4F3l6TNIIpmaE7JGysekDQtVlrkksYOjx5AdDexGtek3xC%2Fu2Gio5VT0k7xaZJRkEyP2tY2Nj9wqZJhsAa%2BICP1S%2BGwGuSIbAGnudY9UC664CgSYpg8PDU09BuA4JxkySBNfAs1OrXsNy4SYbAGngWatVD6U6xaZIhsAYeSNv9CqTHTTIEbalNh0NzeSGtEny1F%2F52iGaThII98LQoL1vVF01rkk7Iv9tgselXyjpukk5wBj6o7dRNXKfYNEknOG3ZtC5dVJ1yKFZDdwhjk8yDPfDU1u7XbNNxk7SDPfDE1q4nth2zQn6TvMPg4amTdt2yQn6jyxsGzjzY%2FRr%2F9ptkHuy2Zmt3GDvkFUVK%2BPHyHB1i2CRDkX%2B%2FoeqX06%2B8yW%2BSb3AG7pmcfjHifpN8gzPwlSpOv%2BbM%2BY3yDQOPGZx6zNB1wN0o5TBweNz6gEXHAXeTTIIz8FlzeSWFvhg2pYqfGXzOJB33Rl2r2pe8g1tW6I81%2F9jieLu%2BOL5S%2F8B0xroWw08kFRDqHTSwAsnFO9XA6Pwa42uQkdQoyVd1iqtM2kPqDasy4HJQWYIgK7tIe4e%2BrDZOj9F1ZfcU4LKqrzDUiJfVNl4tvNPs8RC2pVm1d4ZohEpSGVMzVC28cBZGMDsrVvG2ID1oeUZ7aClVtOy6ZNY0INENyl5G%2FE2e%2BEgLihLlAboQGFdr6GtEQFKnUjmqKnVzXp4ogptM1ocrzF91AljMSiSZthopHTyMA%2FZWtIi3z63RbExlnpOS1DTVz0PxBif7vHhZ7VQE9piV%2FOVHh%2BdSV8UxH%2BlLuKmTQkkQwfMD0QKp5eMe2N0rR10bHloS6I2qtrnaytZNJLUpG8C6HGBbMuzzGp9FLMfNWOGA%2BMfCyDGgLgo8igLOly9EQbQoOohSWs%2FqlKAsY3iy9%2ByVikEa6xfh4lit7qOoBJ2%2Fn%2BXmtMO8gOSJvWaJb9ErzdnViWQxAWCv4QhhjGD4y78aw96qYZ8PcvwA9uRjgnFW2veRGMXd7ziE9Ii%2FAQ%3D%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E)
- **Diagram (PDF export):** [docs/business_process/business_process_diagram.pdf](docs/business_process/business_process_diagram.pdf)
- **Supporting explanation:** [docs/business_process/business_process_explanation.pdf](docs/business_process/business_process_explanation.pdf)

## Phase II - Data Sourcing, Normalization and Logical Design

The raw UCI file (748 donors) is stored unchanged in [data/raw/](data/raw/) and described, with a quality report of counts, in [data/dataset_card.md](data/dataset_card.md). Each raw row summarises a donor's whole history. Normalizing it shows a hidden list of donations (repeating group), donor-level values that depend on part of the key (partial dependencies), and values that depend on other non-key values, such as total volume on the number of donations (transitive dependencies). The result is 12 tables in 3NF, including the junction table `DONOR_RECALLS` for the many-to-many link between donors and recall campaigns.

- **Normalization, UNF → 3NF with real rows, field mapping and cleaning checklist:** [docs/design/normalization.md](docs/design/normalization.md)
- **Logical ERD (crow's-foot):** [docs/design/erd.png](docs/design/erd.png)
- **Data dictionary:** [docs/design/data_dictionary.md](docs/design/data_dictionary.md)

## References

- Beliën, J., & Forcé, H. (2012). Supply chain management of blood products: A literature review. *European Journal of Operational Research, 217*(1), 1–16. https://doi.org/10.1016/j.ejor.2011.05.026
- Nisingizwe, M. P., Ndishimye, P., Swaibu, K., Nshimiyimana, L., Karame, P., Dushimiyimana, V., Musabyimana, J. P., Musanabaganwa, C., Nsanzimana, S., & Law, M. R. (2022). Effect of unmanned aerial vehicle (drone) delivery on blood product delivery time and wastage in Rwanda: A retrospective, cross-sectional study and time series analysis. *The Lancet Global Health, 10*(4), e564–e569. https://doi.org/10.1016/S2214-109X(22)00048-1
- Oracle. (n.d.). DBMS_SCHEDULER. In *Oracle Database PL/SQL Packages and Types Reference* (19c). https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SCHEDULER.html
- World Health Organization. (2026, June 12). *Blood safety and availability* [Fact sheet]. https://www.who.int/news-room/fact-sheets/detail/blood-safety-and-availability
- Yeh, I.-C. (2008). *Blood Transfusion Service Center* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5GS39
- Yeh, I.-C., Yang, K.-J., & Ting, T.-M. (2009). Knowledge discovery on RFM model using Bernoulli sequence. *Expert Systems with Applications, 36*(3), 5866–5871. https://doi.org/10.1016/j.eswa.2008.07.018

The full list of sources is in [docs/research/research_log.md](docs/research/research_log.md).
