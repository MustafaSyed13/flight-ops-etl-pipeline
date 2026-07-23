# Flight Ops ETL Pipeline & Power BI Dashboard

Hey, thanks for stopping by. This is a project I built on my own to practice real data cleaning and dashboarding, not just working with data that's already tidy.

I started with a single Excel file that had 31 tabs in it, one for every day in May, each one a handwritten shift report. It was rough. Different headers on different days, random total rows sitting right in the middle of the real data, numbers stored as text, flight numbers spelled wrong, even a few negative numbers that made no sense. Basically what you'd actually get from a busy operations team typing things up by hand.

My goal was to take that mess and turn it into something usable: one clean table and a dashboard someone could actually read. I ended up with 566 solid flight records, and I built the cleanup process so that if you handed me next month's file, it would run through automatically without me touching a thing.

> Everything shown here is safe to share publicly: flight numbers, times, gate, passenger counts, and origin airport. Nothing confidential is in this repo.

---

## Here's What It Looks Like

**The whole month at once: 27,283 passengers across 566 flights**
![Full dashboard](screenshots/01-dashboard-full-month.png)

**Zoomed into a single week using the slicer**
![Week 1 filtered dashboard](screenshots/02-dashboard-week1-filter.png)

**How I set up the charts behind the scenes**
![Visual builder / field configuration](screenshots/03-visual-builder-slicer.png)

**The Power Query Editor, where most of the real work happened**
![Power Query applied steps](screenshots/04-power-query-applied-steps.png)

---

## The Numbers

566 flights total: 542 arrived, 19 got cancelled, 5 were diverted. They flew out of six airports (EWR, LGA, BOS, MDW, IAD, and BNA), and were split between two carriers, Porter carried about three quarters of the passengers, Air Canada the rest.

---

## Cleaning Up the Mess

This was honestly the hard part. I spent way more time here than on the dashboard itself. A few things I had to figure out:

The 31 daily sheets didn't line up with each other, so first I had to strip out the junk rows (headers, blanks, totals) from all of them at once, then stack them into one table. I set it up so a brand new month's file just flows through the same steps automatically.

Some columns were typed as text when they should've been numbers or times, so I fixed those. Some values were flat out impossible (negative passenger counts from typos), so I wrote a rule to catch and clear those out.

Eighteen flight numbers were consistently mistyped across the sheets, so instead of fixing each one by hand, I wrote a single step that corrects all eighteen at once, like a built-in lookup table.

There wasn't a clean "status" column either. Whether a flight arrived, got cancelled, or diverted was buried in different columns depending on the day, so I wrote logic that scans for the right keywords wherever they show up and builds a proper status for every flight.

And there was a processing-time column in the original data I just didn't trust, it mixed units and had at least one negative number in it. So I threw it out and recalculated it myself straight from the actual timestamps.

What came out the other end: 17 clean, correctly typed columns ready to build a real dashboard on top of.

---

## Telling the Airlines Apart

The raw data never actually said which airline a flight belonged to, just a flight number. So I wrote this bit of DAX to figure it out based on the pattern in the flight number:

```dax
Airline =
IF(
    LEFT(FORMAT('1'[Flight], "0"), 2) = "85" || FORMAT('1'[Flight], "0") = "9804",
    "Air Canada",
    "Porter"
)
```

---

## What's on the Dashboard

A few KPI cards up top for the headline numbers, a line chart tracking passengers day by day, a bar chart showing which airports flights came from, a donut chart splitting passengers by carrier, and a week slicer so you can click through Week 1 to 4 or look at the whole month at once. I also spent time on the visual side, custom colors, card styling, a header banner, so it actually looks like something you'd want to open.

---

## What I Built It With

Power BI Desktop, Power Query (the M language), and DAX, starting from an Excel file. I kept the working files on OneDrive so I could pick up the project from either of my computers.

---

## What's Actually in This Repo

```
flight-ops-etl-pipeline/
├── README.md
└── screenshots/
    ├── 01-dashboard-full-month.png
    ├── 02-dashboard-week1-filter.png
    ├── 03-visual-builder-slicer.png
    └── 04-power-query-applied-steps.png
```

I kept the actual Power BI file and the source Excel workbook on my own computer rather than uploading them, since they hold the full dataset. This page is here to show how I think through a messy data problem and what I ended up building.
