# dot-malloy-example — UK property prices

An example [Malloy](https://malloydata.dev) semantic model, used to demonstrate and
test **Dot's Malloy connector**. It models HM Land Registry "Price Paid" data for
UK residential property (~5.5M monthly-aggregated rows, 1995–2026) living in Dot's
FoxDB warehouse.

## Using it with Dot

In Dot: **Settings → Connections → Malloy**, point it at this repo. Dot loads
`index.malloy`, syncs each source as a queryable table, and **injects the warehouse
credentials at query time** — so no secrets live here. Then just ask questions in
natural language; Dot writes the Malloy.

## The model (`index.malloy`)

Three explores over the `foxdb` connection — a fact and two pre-aggregated rollups:

- **`property_sales`** (`public.fact_monthly_sales`) — the central transaction fact.
  - dimensions: `county`, `town`, `district`, `property_type`, `tenure`,
    `is_new_build`, `sale_year`, `sale_quarter` (year/quarter derived from the
    fact's own `month` date — no calendar join needed)
  - measures: `transaction_count`, `total_sales_value`, `avg_sale_price`
    (value-weighted), `max_sale_price`
  - **joins** (`join_one`, unique keys): `county_benchmark` → `county_annual`,
    `town_benchmark` → `town_trends`, so any slice can be compared to its
    county-year and town-year benchmark.
  - views: `by_county`, `by_property_type`, `price_trend_by_year`,
    `new_vs_existing`, `top_towns`, `vs_county_benchmark`
- **`county_annual`** (`public.agg_county_annual`) — annual sales per county.
  - measures: `transaction_count`, `total_sales_value`, `avg_sale_price`,
    `new_build_count`, `new_build_share`
  - views: `by_county`, `top_new_build_counties`, `yearly_trend`
- **`town_trends`** (`public.agg_town_price_index`) — per-town yearly average price
  with precomputed year-over-year change.
  - measures: `transaction_count`, `avg_sale_price`, `avg_yoy_change_pct`
  - views: `fastest_growing_towns`, `by_year`

## Running standalone

The connection is named `foxdb` (Postgres wire). To run with the
[Malloy CLI](https://github.com/malloydata/malloy-cli) or Publisher, configure a
`foxdb` Postgres connection to `db.getdot.ai:5432`, database `db`, user `default`,
password = your FoxDB token, `sslmode=require`. Tables are schema-qualified
(`public.<table>`); columns that collide with Malloy keywords (`month`, `year`,
`quarter`, `type`) are backticked in the model.

## Example queries

```malloy
run: property_sales -> by_county
run: property_sales -> price_trend_by_year
run: property_sales -> vs_county_benchmark + { where: county = 'GREATER LONDON' }

run: county_annual -> top_new_build_counties
run: town_trends -> fastest_growing_towns
```
