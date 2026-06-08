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

Two sources over the `foxdb` connection:

- **`property_sales`** (`public.fact_monthly_sales`) — the central fact.
  - dimensions: `county`, `district`, `town`, `postcode1`, `property_type`,
    `tenure`, `is_new_build`, `sale_year`, `sale_quarter`
  - measures: `transaction_count`, `total_sales_value`, `avg_sale_price`
    (value-weighted), `max_sale_price`
  - views: `by_county`, `by_property_type`, `price_trend_by_year`,
    `new_vs_existing`, `top_towns`
- **`dim_calendar`** (`public.dim_calendar`) — joined on `month = month_start` for
  year/quarter.

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
run: property_sales -> {
  where: county = 'GREATER LONDON'
  group_by: sale_year
  aggregate: avg_sale_price
  order_by: sale_year desc
}
```
