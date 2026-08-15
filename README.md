Olist E-Commerce Analytics — Medallion Pipeline & Power BI Dashboard

End-to-end data engineering and BI project on the [Olist Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), built on Microsoft Fabric. Raw Kaggle CSVs are ingested, cleaned, and modeled through a Bronze → Silver → Gold Medallion Architecture, then served to an interactive six-page Power BI dashboard via Direct Lake mode.

<img width="1040" height="916" alt="olist_infographic" src="https://github.com/user-attachments/assets/2d05718a-7b9f-4d4b-9ebb-53e209447d2b" />

## Overview

Nine raw CSVs (~100K orders, 2016–2018, ~R$16M in transaction value) are transformed into a star-schema Gold layer and surfaced through a semantic model with DAX measures. The analysis focuses on six business questions: sales performance, delivery delays, seller/product performance, logistics bottlenecks, customer behaviour and payment trends.

| Metric | Value |
|---|---|
| Total orders | 99,441 |
| Total revenue | R$16,008,872.12 |
| On-time delivery rate | 91.9% |
| Late delivery rate | 6.65% |
| Avg. delay (late orders) | 10.64 days |
| Active sellers | 3,095 |
| Avg. review score | 4.09 / 5.00 |
| Product categories | 73 |

---

## Architecture

The pipeline follows the Medallion Architecture end to end in Microsoft Fabric:

```
Kaggle CSVs → Bronze (raw Delta) → Silver (cleaned + engineered) → Gold (star schema) → Semantic Model → Power BI
```

- **Bronze** — 9 raw tables loaded via `spark.read.csv()` with schema inference and ingestion timestamps. Left unaltered as the immutable source of truth.
- **Silver** — PySpark cleaning: null handling, deduplication (including composite-key dedup on `order_items`), ZIP-code geolocation collapse, and foreign-key validation via left-anti joins. A custom **Haversine UDF** derives `distance_km`; `delay_days` is engineered from purchase/delivery timestamps.
- **Gold** — One fact table (`gold_fact_delivery`, 103,200 rows / 38 columns) and four dimensions (`gold_dim_customer`, `gold_dim_seller`, `gold_dim_product`, `gold_dim_date`), all single-direction 1:N into the fact — no bi-directional filters or snowflaking, by design.
- **Semantic model → Power BI** — Direct Lake mode for near-real-time analytics without data duplication, feeding six dashboard pages.

See [`docs/architecture_diagram.png`](docs/) and [`docs/erd.png`](docs/) for the full diagrams from the project report.

---

## Key Findings

- **Distance drives delay.** Late delivery rate rises steadily from 4.3% (0–50km) to 10.3% (1000+km).
- **Geography is uneven.** High-volume states (SP, RJ) have low late-rates but contribute the most late orders in absolute terms; low-volume remote states (ES, MA) run far hotter — up to 20%.
- **Cost doesn't buy speed.** Late orders average R$22.87 in freight vs. R$19.32 for on-time orders — a marginal gap that doesn't support "pay more, arrive faster."
- **Weight compounds regional risk.** Heavier parcels cluster disproportionately in states that already have weaker logistics infrastructure.
- **Seller performance varies widely.** A subset of sellers run 15–30%+ late rates, with some remote-state sellers at 100% on low volumes; routes like AM–AL, MA–AL, and MS–BA see late rates of 60–100%.

## Recommendations

1. **Regional distribution hubs** in northern/northeastern states to cut last-mile distance on the worst routes.
2. **Seller enablement** — flag underperforming sellers for SLA support and dispatch/packaging training.
3. **Specialist heavy-freight carriers** for 5+ kg parcels routed into high-risk states.
4. **Performance-based freight pricing** tied to actual delivery SLAs, since higher freight cost alone doesn't correlate with faster delivery.
5. **Better delivery-time estimation** and **automated monitoring/alerts** for routes and sellers with rising delay rates.

---

## Dashboard Pages

**1. Initial Summary** — orders, late delivery rate, delay days, freight value, and monthly trend.
![Initial Summary](docs/screenshots/01_initial_summary.png)

**2. Geographic Analysis** — late delivery orders by state, mapped across Brazil.
![Geographic Analysis](docs/screenshots/02_geographic_analysis.png)

**3. Distance vs Delay** — late delivery rate and avg. delay days by distance band.
![Distance vs Delay](docs/screenshots/03_distance_vs_delay.png)

**4. Weight & Freight Analysis** — late rate by freight band, freight value vs. delay days.
![Weight & Freight Analysis](docs/screenshots/04_weight_freight_analysis.png)

**5. Seller Performance** — seller-level late rates, top 20 late sellers, late rate by route.
![Seller Performance](docs/screenshots/05_seller_performance.png)

**6. Correlation Analysis** — location, freight cost, and weight correlation with late deliveries.
![Correlation Analysis](docs/screenshots/06_correlation_analysis.png)

---

## Tech Stack

`Microsoft Fabric` · `PySpark` · `Delta Lake` · `Power BI (Direct Lake, DAX)` · `Pandas / NumPy` · `Haversine (custom UDF)` · `PostgreSQL` (validation)

---

## Repo Structure

```
├── notebooks/
│   ├── 01_bronze_ingestion_notebook.html
│   ├── 02_silver_transformation_notebook.html
│   └── 03_gold_final_notebook.html
├── docs/
│   ├── architecture_diagram.png
│   ├── erd.png
│   └── screenshots/
├── olist_infographic.png
