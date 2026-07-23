# Flight Ops ETL Pipeline & Power BI Dashboard

An end-to-end data engineering project that transforms a month of messy, real-world-style daily airline shift reports into a clean relational dataset and an interactive Power BI dashboard — built entirely with **Power Query (M)**, **DAX**, and **Power BI Desktop**.

The raw source is a single Excel workbook with **31 separate sheets** (one per day of May) written by shift staff by hand: inconsistent headers, embedded subtotal rows, mixed data types in the same column, typos in flight numbers, and negative values from data-entry mistakes. This project builds a repeatable ETL recipe that turns that into **566 clean flight records** and a live reporting dashboard — so a new month's file can be dropped in and flow through the same pipeline with zero manual rework.

> Only non-sensitive, publicly shareable fields (flight numbers, timestamps, gate, passenger counts, origin airport) are included. No proprietary carrier or confidential operational data is published in this repo.

---

## Dashboard Preview

**Full month view — 27,283 passengers across 566 flights, May 1–31**
![Full dashboard](screenshots/01-dashboard-full-month.png)

**Week-filtered view — the Week slicer drives every visual on the page**
![Week 1 filtered dashboard](screenshots/02-dashboard-week1-filter.png)

**Report Builder view — field wells behind the line chart and slicer**
![Visual builder / field configuration](screenshots/03-visual-builder-slicer.png)

**Power Query Editor — the ~30-step applied-steps recipe that produces the model table**
![Power Query applied steps](screenshots/04-power-query-applied-steps.png)

---

## Data Scope

| | |
|---|---|
| Source | 1 Excel workbook, 31 sheets (one per day of May) |
| Raw sheet format | "Daily Shift Report" — inconsistent headers, embedded totals rows, mixed data types, typos |
| Final cleaned dataset | 566 individual flight records, May 1–31 |
| Breakdown | 542 Arrived · 19 Cancelled · 5 Diverted |
| Origins covered | EWR, LGA, BOS, MDW, IAD, BNA (6 airports) |
| Carriers | Porter (75.6% share) · Air Canada (24.4% share) |

---

## ETL Pipeline (Power Query / M)

Built a ~30-step applied-steps recipe (visible in the Power Query Editor screenshot above) that turns 31 inconsistent daily sheets into one clean fact table:

- **Structural cleanup** — filtered out header rows, blank filler rows, and totals rows across all 31 sheets simultaneously using `Table.SelectRows`.
- **Combine step** — used `Table.ExpandTableColumn` to unpivot and stack all 31 daily sheets into a single table automatically, so a new month's workbook flows through the same query with no manual editing.
- **Data type correction** — fixed columns that Excel/import had mistyped as `Text` instead of `Whole Number` / `Decimal` / `Time`.
- **Custom M logic**:
  - A `Table.ReplaceValue` call with a custom replacer function to null out impossible negative values caused by data-entry typos.
  - A `List.Accumulate` batch-replace across 18 wrong → correct flight-number pairs in a single pass — equivalent to a lookup table, applied without 18 manual point-and-click replace steps.
  - A derived `Status` column using `Text.Contains` logic across multiple columns to classify each flight as Arrived / Cancelled / Diverted, since the source data placed status flags inconsistently between columns.
  - A recalculated `ProcessMinutes` column computed directly from timestamp differences with `Duration.TotalMinutes`, replacing the source's unreliable pre-computed field (which mixed units and contained at least one negative value).
- **Result** — 17 clean, correctly-typed columns feeding the model:
  `Day, Flight, Origin, ETA, ATA, Crew, Pax, FirstPax, LastPax, ProcessMinutes, Gate, BSO, RefCust, RefImm, RefCFIA, RefExams, Status, Date`

---

## Data Modeling (DAX)

A calculated column classifies each flight by carrier using flight-number pattern matching:

```dax
Airline =
IF(
    LEFT(FORMAT('1'[Flight], "0"), 2) = "85" || FORMAT('1'[Flight], "0") = "9804",
    "Air Canada",
    "Porter"
)
```

---

## Dashboard (Power BI Report)

- **3 KPI cards** — Total Passengers (27,283), Total Flights (566), Cancelled Flights (19)
- **Line chart** — daily passenger volume trend across the month, sorted chronologically
- **Clustered bar chart** — passengers by origin airport (6 origins)
- **Donut chart** — passenger share by carrier
- **Interactive slicer** — date-based Week filter (Week 1–4) enabling on-demand weekly or full-month views from a single report page
- **Styling pass** — custom theme, card effects, header banner, formatted titles/labels

---

## Tools

Power BI Desktop · Power Query (M) · DAX · Excel — with OneDrive used to sync the working `.pbix`/`.xlsx` files across two machines during development.

---

## Repository Contents

```
flight-ops-etl-pipeline/
├── README.md
└── screenshots/
    ├── 01-dashboard-full-month.png
    ├── 02-dashboard-week1-filter.png
    ├── 03-visual-builder-slicer.png
    └── 04-power-query-applied-steps.png
```

The `.pbix` file and source workbook are kept locally and are not included in this repo, since they contain the full operational dataset. This repo documents the pipeline design, DAX logic, and dashboard output.
