---
type: CODATUM_NOTEBOOK
id: 6a335790b0846f46a0452839
workspace_id: 674d04a76ab32ef89c104873
name: theLook Sales Dashboard
notebook_param_widgets:
  - id: 6a3357ba81ef5818ace927fd
    type: DATE_RANGE
    label: Period
    settings:
      unit: MONTH
      absoluteValidRange: {}
      relativeValidRange: {}
    defaultValue:
      type: RELATIVE_DATE_RANGE
      relativeDateRangeOffsetStart: -365
      relativeDateRangeOffsetEnd: 0
---

# theLook Sales Dashboard

```yaml {.page}
type: DOC_PAGE
id: 6a3357ba0b57339daf6e3f3d
width:
  width_type: AUTO
notebook_param_widget_values:
  - id: 6a3357ba81ef5818ace927fd
    value:
      - '2025-06-01'
      - '2026-06-30'
```

## Total Sales by Category

Joins `order_items` and `products` from the theLook ecommerce dataset and aggregates total sales (sum of `sale_price`) by product category.

:::sql-block
```yaml {.attrs}
id: 6a3357ba6fb6fe30865bf939
connId: 674d04aa723c39d549407a6c
name: Total Sales by Category
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=order_items
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=products
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.1
    escapeType: DATE
```

```sql
SELECT
  p.category AS category,
  SUM(oi.sale_price) AS total_sales
FROM $1 AS oi
JOIN $2 AS p ON oi.product_id = p.id
WHERE oi.created_at BETWEEN $3 AND $4
GROUP BY category
ORDER BY total_sales DESC
```

```yaml {.result}
height: 300px
```
:::

```yaml {.chart}
id: 6a3357baee16b30875c5ae60
sqlId: 6a3357ba6fb6fe30865bf939
chartSettings:
  template_type: XY_CHART_V2
  component_settings:
    name: Total Sales by Category
    enable_auto_dispatch: true
    x:
      - field: category
    views:
      - view_id: 6a3357ba5b289bd3ca287ff5
        render_type: BAR
        ys:
          - field: total_sales
            aggregator: SUM
    group_by: []
    order_by: VALUE_DESC
  display_settings:
    transpose: true
    x_axis:
      name: Category
    y_axis:
      name: Total Sales ($)
    series:
      6a3357ba5b289bd3ca287ff5-total_sales___sum:
        formatter:
          formatter_number: .3s
        value_label:
          show: true
          formatter:
            formatter_number: .3s
height: 720px
```

::p

## Key KPIs

Total sales, order count, and average order value (AOV) over the selected period.

:::sql-block
```yaml {.attrs}
id: 6a3357ba72bfa5113654ea34
connId: 674d04aa723c39d549407a6c
name: Key KPIs
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=order_items
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.1
    escapeType: DATE
```

```sql
SELECT
  SUM(sale_price) AS total_sales,
  COUNT(DISTINCT order_id) AS order_count,
  SUM(sale_price) / COUNT(DISTINCT order_id) AS avg_order_value
FROM $1
WHERE created_at BETWEEN $2 AND $3
```

```yaml {.result}
height: 150px
```
:::

::::columns
:::column
```yaml {.chart}
id: 6a3357ba55077999e3f476f1
sqlId: 6a3357ba72bfa5113654ea34
chartSettings:
  template_type: BIG_NUMBER_V2
  component_settings:
    name: Total Sales
    enable_auto_dispatch: true
    render_type: TEXT
    primary_values:
      - field: total_sales
        aggregator: SUM
    secondary_values: []
  display_settings:
    primary:
      prefix: $
      formatter:
        formatter_number: .4s
      conditional_styles: []
height: 200px
```
:::

:::column
```yaml {.chart}
id: 6a3357baa5ad70ceaaf36008
sqlId: 6a3357ba72bfa5113654ea34
chartSettings:
  template_type: BIG_NUMBER_V2
  component_settings:
    name: Order Count
    enable_auto_dispatch: true
    render_type: TEXT
    primary_values:
      - field: order_count
        aggregator: SUM
    secondary_values: []
  display_settings:
    primary:
      suffix: ' orders'
      formatter:
        formatter_number: ','
      conditional_styles: []
height: 200px
```
:::

:::column
```yaml {.chart}
id: 6a3357bab69686170ba0e63a
sqlId: 6a3357ba72bfa5113654ea34
chartSettings:
  template_type: BIG_NUMBER_V2
  component_settings:
    name: Average Order Value (AOV)
    enable_auto_dispatch: true
    render_type: TEXT
    primary_values:
      - field: avg_order_value
        aggregator: SUM
    secondary_values: []
  display_settings:
    primary:
      prefix: $
      formatter:
        formatter_number: .4r
      conditional_styles: []
height: 200px
```
:::
::::

::p

## Monthly Sales Trend

Sales trend aggregated by month based on `created_at` in `order_items`.

:::sql-block
```yaml {.attrs}
id: 6a3357bab44876f345f60c48
connId: 674d04aa723c39d549407a6c
name: Monthly Sales Trend
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=order_items
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.1
    escapeType: DATE
```

```sql
SELECT
  DATE_TRUNC(created_at, MONTH) AS month,
  SUM(sale_price) AS total_sales
FROM $1
WHERE created_at BETWEEN $2 AND $3
GROUP BY month
ORDER BY month
```

```yaml {.result}
height: 300px
```
:::

```yaml {.chart}
id: 6a3357ba56dd3a22b0665abe
sqlId: 6a3357bab44876f345f60c48
chartSettings:
  template_type: XY_CHART_V2
  component_settings:
    name: Monthly Sales Trend
    enable_auto_dispatch: true
    x:
      - field: month
    views:
      - view_id: 6a3357ba74fb74a43121c8f5
        render_type: LINE
        ys:
          - field: total_sales
            aggregator: SUM
    group_by: []
    order_by: LABEL_ASC
  display_settings:
    x_axis:
      name: Month
    y_axis:
      name: Total Sales ($)
    series:
      6a3357ba74fb74a43121c8f5-total_sales___sum:
        formatter:
          formatter_number: .3s
height: 320px
```

::p

## Top 10 Sales by Country

Joins `order_items` and `users` and aggregates sales by customer country, showing the top 10 countries.

:::sql-block
```yaml {.attrs}
id: 6a3357ba4a2bbf3ee0354b86
connId: 674d04aa723c39d549407a6c
name: Top 10 Sales by Country
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=order_items
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=users
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.1
    escapeType: DATE
```

```sql
SELECT
  u.country AS country,
  SUM(oi.sale_price) AS total_sales
FROM $1 AS oi
JOIN $2 AS u ON oi.user_id = u.id
WHERE oi.created_at BETWEEN $3 AND $4
GROUP BY country
ORDER BY total_sales DESC
LIMIT 10
```

```yaml {.result}
height: 300px
```
:::

```yaml {.chart}
id: 6a3357ba858bde56c4997540
sqlId: 6a3357ba4a2bbf3ee0354b86
chartSettings:
  template_type: XY_CHART_V2
  component_settings:
    name: Top 10 Sales by Country
    enable_auto_dispatch: true
    x:
      - field: country
    views:
      - view_id: 6a3357baecc2d6cb6f5f9f53
        render_type: BAR
        ys:
          - field: total_sales
            aggregator: SUM
    group_by: []
    order_by: VALUE_DESC
  display_settings:
    transpose: true
    x_axis:
      name: Country
    y_axis:
      name: Total Sales ($)
    series:
      6a3357baecc2d6cb6f5f9f53-total_sales___sum:
        formatter:
          formatter_number: .3s
        value_label:
          show: true
          formatter:
            formatter_number: .3s
height: 360px
```

::p

## Orders by Day of Week × Hour (Heatmap)

Extracts day of week and hour from `created_at` in `order_items` and visualizes order count as a heatmap.

:::sql-block
```yaml {.attrs}
id: 6a3357ba797ca03847c1fe8d
connId: 674d04aa723c39d549407a6c
name: Orders by Day of Week × Hour
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674d04aa723c39d549407a6c/pj=bigquery-public-data/ds=thelook_ecommerce/tb=order_items
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a3357ba81ef5818ace927fd.1
    escapeType: DATE
```

```sql
SELECT
  FORMAT('%02d', EXTRACT(HOUR FROM created_at)) AS hour_of_day,
  CONCAT(CAST(EXTRACT(DAYOFWEEK FROM created_at) AS STRING), ' ', FORMAT_DATE('%a', DATE(created_at))) AS weekday,
  COUNT(*) AS order_count
FROM $1
WHERE created_at BETWEEN $2 AND $3
GROUP BY hour_of_day, weekday
ORDER BY weekday, hour_of_day
```

```yaml {.result}
height: 300px
```
:::

```yaml {.chart}
id: 6a3357bac397d655670a0b42
sqlId: 6a3357ba797ca03847c1fe8d
chartSettings:
  template_type: XYZ_CHART_V2
  component_settings:
    name: Orders by Day of Week × Hour
    enable_auto_dispatch: true
    render_type: HEATMAP
    x:
      - field: hour_of_day
    'y':
      - field: weekday
    z:
      - field: order_count
        aggregator: SUM
  display_settings:
    x_axis:
      name: Hour
    y_axis:
      name: Day of Week
    z_axis:
      formatter:
        formatter_number: ','
height: 360px
```

::p

# Dashboard

```yaml {.page}
type: GRID_PAGE
id: 6a3357ba3c4afcd8eb7a8f7a
width:
  width_type: RANGE
  min_width: 1024
  max_width: 1600
notebook_param_widget_values:
  - id: 6a3357ba81ef5818ace927fd
    value:
      - '2025-06-01'
      - '2026-06-30'
```

:::grid-item
```yaml {.attrs}
type: HEADING
id: 6a3357ba9778f49d817a2b45
```

## theLook Sales Dashboard
:::

:::grid-item
```yaml {.attrs}
type: RICH_TEXT
id: 6a3357baf6c802901640a859
name: About this dashboard
hideFrame: true
```

- A dashboard visualizing sales from the theLook ecommerce dataset (`bigquery-public-data.thelook_ecommerce`).
- Use the "Period" parameter at the top to filter the target period (default: last 12 months).
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357babc595babf1b8e3f6
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357ba55077999e3f476f1
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357ba9fd070105534ef9e
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357baa5ad70ceaaf36008
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357bafff7cc5ad4f8f3d7
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357bab69686170ba0e63a
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357bad859c690d0fbaec3
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357ba56dd3a22b0665abe
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357bad2959f47cf4e3e8c
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357baee16b30875c5ae60
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357ba9a0761d2342eddf1
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357ba858bde56c4997540
```
:::

:::grid-item
```yaml {.attrs}
type: CHART
id: 6a3357ba1a3bb24b0e4f93c0
pageId: 6a3357ba0b57339daf6e3f3d
chartId: 6a3357bac397d655670a0b42
```
:::

```yaml {.grid-layout}
- type: columns
  children:
    - type: leaf
      width: 8
      ref: 6a3357ba9778f49d817a2b45
      height: 2
    - type: leaf
      width: 4
      ref: 6a3357baf6c802901640a859
      height: 3
- type: columns
  children:
    - type: leaf
      width: 4
      ref: 6a3357babc595babf1b8e3f6
      height: 3
    - type: leaf
      width: 4
      ref: 6a3357ba9fd070105534ef9e
      height: 3
    - type: leaf
      width: 4
      ref: 6a3357bafff7cc5ad4f8f3d7
      height: 3
- type: leaf
  ref: 6a3357bad859c690d0fbaec3
  height: 7
- type: columns
  children:
    - type: leaf
      width: 6
      ref: 6a3357bad2959f47cf4e3e8c
      height: 12
    - type: leaf
      width: 6
      ref: 6a3357ba9a0761d2342eddf1
      height: 8
- type: leaf
  ref: 6a3357ba1a3bb24b0e4f93c0
  height: 6
```
