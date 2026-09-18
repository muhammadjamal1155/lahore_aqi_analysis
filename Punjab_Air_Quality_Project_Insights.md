# Punjab Air Quality Analysis — Project Insights

**Scope:** Daily air quality (PM2.5, PM10, US AQI index) and weather data across 10 Punjab cities, 2022-08-01 to 2026-09-13 (~4 years, ~15,050 city-days)
**Tool:** Power BI (Power Query for ETL/modeling, DAX for measures)

---

## 1. Project Purpose

Punjab's cities, and Lahore in particular, are consistently ranked among the most polluted urban areas in the world. This project set out to answer four questions with data rather than assumption:

1. How much worse is air quality during Punjab's Oct–Feb "smog season" compared to the rest of the year?
2. Which cities are most and least affected?
3. Is air quality getting better or worse over time?
4. What role does weather play in driving pollution up or down?

---

## 2. Data Sources & Honesty Note

- **Air quality:** Open-Meteo Air Quality API (built on the Copernicus Atmosphere Monitoring Service — CAMS — atmospheric model)
- **Weather:** Open-Meteo Historical Weather API (meteorological reanalysis)

These are **modeled estimates**, not direct ground-sensor readings from Pakistan's EPA or PMD networks. They were used because neither agency currently exposes a public bulk-download covering 10 cities across this date range (EPA offers only a live dashboard; PMD's historical data requires a manual, non-instant request process). This is stated plainly rather than presented as official government ground-truth data — a deliberate, documented trade-off between data accessibility and source authority.

---

## 3. Data Model

A star schema built in Power Query: 3 fact tables, 7 dimension tables.

**Facts:** `fact_air_quality_daily`, `fact_weather_daily`, `fact_aqi_alert_daily`
**Dimensions:** `dim_city` (10 rows), `dim_date` (~1,505-day generated calendar), `dim_aqi_category` (7 rows — the 6 official US EPA AQI breakpoints plus a "No Data" category), `dim_weather_condition` (5 rows), `dim_pollutant` (2 rows, PM2.5/PM10 — kept as a reference table only, not relationally connected, since the fact table stores both pollutants as columns rather than separate rows), `dim_station` and `dim_time` (single-row placeholders, included to satisfy the required schema even though this dataset has no station-level or intra-day granularity).

---

## 4. Key Findings

### 4.1 Smog season is measurably and consistently worse
- **Average AQI, smog season (Oct–Feb): 169** vs. **non-smog season: 121** — a **40% increase**.
- This gap holds in **every one of the 5 years studied, with no exceptions** — this is not a one-off seasonal anomaly, it's a structural pattern.
- **% of days classified Unhealthy or worse:** 71% during smog season vs. 19% outside it — roughly **3.7x more likely**.

### 4.2 Pollution is trending worse, not better
- Year-over-year AQI change: **-10% (2023)** → **+4% (2024)** → **+4% (2025)** → **+5% (2026)**.
- After an initial improvement in 2023, every subsequent year has shown pollution increasing relative to the year before.

### 4.3 City-level disparity is significant
- **Worst average AQI:** Lahore, followed closely by Faisalabad and Sheikhupura.
- **Best average AQI:** Rawalpindi and Bahawalpur, consistently the cleanest of the 10 cities.
- Ranking shifts somewhat depending on the season filter — Lahore's relative position worsens further specifically during smog season, suggesting its pollution is more seasonally concentrated than some other cities.

### 4.4 A distinct, non-seasonal extreme event was identified
- **May 23, 2025:** a sudden multi-city spike (city-average AQI 244.6, single highest reading: Lahore at AQI 538) occurred **outside** smog season.
- Cross-checking PM2.5 vs. PM10 on this date shows PM10 (larger dust particles) far exceeding PM2.5 (fine combustion particles) in every affected city, while one unaffected city (Bahawalpur) showed no change at all — the signature of a **regional dust storm**, not smog. This distinguishes two separate pollution mechanisms present in the same dataset.
- **The single highest 7-day rolling average AQI in the entire dataset (257.55) occurred January 20, 2026** — squarely within smog season, and the actual worst sustained period recorded.

### 4.5 Data completeness is high
- Of ~15,050 city-days, only ~40 rows (≈0.27%) have missing readings — nearly all concentrated on the very first day(s) of data collection (2022-08-01), consistent with a data-pipeline warm-up gap rather than ongoing sensor failure.
- True completeness: ≈99.7%.

### 4.6 Open question worth confirming
- Across the full dataset, visual inspection of the category breakdown shows almost no days classified "Good" (AQI 0-50) and very few "Moderate" — the overwhelming majority fall into Unhealthy, Unhealthy for Sensitive Groups, or Very Unhealthy. This should be confirmed with a direct count (`COUNTROWS` filtered to category = "Good") before stating it definitively, but if confirmed, it would mean **Punjab's air quality rarely if ever reaches officially "safe" levels** across this entire 4-year sample — one of the most striking possible findings in the dataset.

---

## 5. Limitations

- Data is **modeled**, not measured on the ground — treat absolute values as estimates, though relative comparisons (city vs. city, season vs. season, year vs. year) are likely robust even if the modeled baseline shifts.
- `dim_pollutant`, `dim_station`, and `dim_time` are structurally present but analytically thin (1-2 rows each) — included to satisfy a required schema, not because they add real dimensional depth in this dataset.
- The May 2025 dust-storm finding is based on a pattern match (PM10 vs. PM2.5 ratio, geographic clustering) rather than confirmed meteorological event data — a reasonable inference, not a certainty.

---

## 6. Suggested Headline Statement

> "Across 10 Punjab cities and 4+ years of data, air quality during smog season (Oct–Feb) averages 40% higher AQI than the rest of the year — a pattern that held in every single year studied, and one that has been getting worse, not better, since 2023."
