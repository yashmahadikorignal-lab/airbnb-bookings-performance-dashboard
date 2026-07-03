# Airbnb Bookings Performance Dashboard

An end-to-end Power BI analytics project that cleans a 100K+ row Airbnb bookings dataset and turns it into a two-page executive dashboard tracking revenue, occupancy, and host/property performance.

## 🎯 Overview

This project analyzes Airbnb booking activity across 80 listings, 50 hosts, and 20 neighbourhoods over a 3-year window (Jan 2023–Jan 2026). The raw export had the kind of real-world messiness you'd actually get from a booking system extract — mixed currency formatting, inconsistent percentage fields, invalid negative values, and duplicate records — so a meaningful chunk of the work was building a repeatable cleaning process before any KPI could be trusted. The final output is a two-page Power BI report: an Executive Overview for revenue/occupancy trends and a Host & Property page for performance segmentation.

## 📊 Dashboard / Key Visuals

**Page 1 — Executive Overview:** KPI cards (Total Revenue, Avg Nightly Rate, Occupancy Rate, Avg Review Score, Cancellation Rate), monthly revenue trend, bookings by neighbourhood, bookings by room type, seasonal occupancy trend, RevPAR trend.

**Page 2 — Host & Property:** bookings by property type, Superhost vs. Regular split (by bookings and by revenue), price-vs-review scatter by property type, a revenue decomposition tree (neighbourhood → property type → room type → host status), and a host performance leaderboard table.

![Executive Overview](images\Executives.jpg)
![Host and Property](images\Host.jpg)

## 🗂️ Data

Star-schema layout, one fact table + four dimension tables:

| Table | Rows | Grain |
|---|---|---|
| `fact_bookings` | 100,200 (100,000 after dedup) | one row per booking |
| `dim_property` | 80 | one row per listing |
| `dim_host` | 50 | one row per host |
| `dim_location` | 20 | one row per neighbourhood |
| `dim_date` | 1,127 | one row per calendar day, Jan 2023 – Jan 2026 |

All foreign keys (`listing_id`, `host_id`, `location_id`, `date_key`) were validated against their dimension tables with zero orphaned records — the model joins cleanly.

## 🧹 Data Cleaning Process

This is the core of the project — the raw `fact_bookings` export needed the following fixes before it was reliable enough to build measures on:

1. **Removed exact duplicate bookings** — 200 fully duplicated rows in `fact_bookings` (0.2% of records) were dropped; each `booking_id` should be unique.
2. **Standardized the `price` field** — ~3,000 rows (3%) stored price as a quoted currency string (e.g. `"$193.00"`) instead of a plain number like the other 97,189 rows. Stripped the `$` and quote characters and cast the whole column to a consistent numeric type.
3. **Standardized `host_response_rate`** — 47 of 50 hosts had this stored as a percentage string ("92%"), 3 as a raw decimal ("0.88"). Normalized all values to the same numeric scale so response rate could be aggregated and compared across hosts.
4. **Fixed invalid `minimum_nights` values** — 536 rows (~0.5%) had negative values (e.g. -2), which isn't physically possible for a nights-stayed field. Filtered these out as data entry errors rather than imputing a guessed value.
5. **Capped out-of-range `review_score`** — Airbnb review scores run 0–5, but 1,503 rows exceeded 5 (up to 5.9). Capped at the theoretical maximum rather than dropping the rows, since the booking itself was still valid.
6. **Corrected `availability_365` outliers** — 600 rows exceeded the possible 0–365 day range (up to 999), which reads as a placeholder/sentinel value from the source system rather than a real figure. Excluded these from occupancy-rate calculations.
7. **Handled missing values** — 3 of 50 `host_name` values and 5 of 80 `bedrooms` values were null; flagged for review rather than silently imputed, since guessing a host's name isn't defensible.
8. **Reformatted `host_since`** — stored as `DD-MM-YYYY` text; parsed into a proper date type so tenure could be calculated.
9. **Validated the revenue formula** — spot-checked that `calculated_revenue = price × minimum_nights` held consistently across sampled rows before trusting it as a base measure for DAX.
10. **Verified referential integrity** — confirmed every fact-table foreign key resolves to a real dimension row (zero orphans across all four relationships) before building the model.

## 🛠️ Tools & Techniques

- **Power Query (M)** — for the cleaning steps above: type casting, string parsing, deduplication, outlier filtering.
- **Power BI data model** — star schema with four dimension tables joined to one fact table.
- **DAX measures**, including: `Total Revenue`, `Avg Nightly Rate`, `Occupancy Rate`, `Avg Review Score`, `Cancellation Rate`, `RevPAR`, `Total Bookings`, `Avg Revenue Per Booking`.
- **Visual types**: KPI cards, line charts, clustered bar chart, pie/donut charts, scatter chart, decomposition tree, detail table, slicers.

## 🔍 Key Findings / Insights

- Roughly **1 in 4 bookings (27%)** fall under a strict or super-strict cancellation policy, which is worth factoring into any revenue-reliability analysis.
- Average nightly rate across the cleaned dataset sits around **$236**, with a wide spread ($35–$714) reflecting the mix of room types from shared rooms to entire homes.
- The Host & Property decomposition tree makes it possible to trace revenue down to a specific neighbourhood → property type → host-status combination, useful for spotting which segments are Superhost-driven vs. Regular-host-driven.

## 📁 Repository Structure

```
airbnb-bookings-performance-dashboard/
├── README.md
├── data/            # dim_date.csv, dim_host.csv, dim_location.csv, dim_property.csv, fact_bookings.csv
├── dashboards/       # Airbnbreport.pbix
└── images/           # exported screenshots of both report pages
```

## 📈 Results / Impact

- Consolidated **5 raw CSV extracts (~100K rows total)** into a single validated star-schema model.
- Identified and resolved **10 distinct data quality issues** spanning formatting, duplicates, invalid values, and missing data before any KPI was trusted.
- Delivered a 2-page, 20-visual interactive dashboard covering 8 core business measures for revenue, occupancy, and host performance.
