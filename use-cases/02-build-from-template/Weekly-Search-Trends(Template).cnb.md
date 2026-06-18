---
type: CODATUM_NOTEBOOK
id: 6a33777e26f486d9eebb4160
workspace_id: 674d04a76ab32ef89c104873
name: Weekly Search Trends Report (Template)
notebook_param_widgets:
  - id: 6a33777ee2d06b5b770b75a4
    type: TEXT_SELECT
    label: Country
    settings:
      optionsSourceType: TEXT
      optionsSourceText: |-
        Argentina
        Australia
        Austria
        Belgium
        Brazil
        Canada
        Chile
        Colombia
        Czech Republic
        Denmark
        Egypt
        Finland
        France
        Germany
        Hungary
        India
        Indonesia
        Israel
        Italy
        Japan
        Malaysia
        Mexico
        Netherlands
        New Zealand
        Nigeria
        Norway
        Philippines
        Poland
        Portugal
        Romania
        Saudi Arabia
        South Africa
        South Korea
        Spain
        Sweden
        Switzerland
        Taiwan
        Thailand
        Turkey
        Ukraine
        United Kingdom
        Vietnam
      description: Target country for the report (Google Trends country_name).
      placeholder: Select a country
    defaultValue:
      type: FIXED
      fixedValue: Japan
  - id: 6a33777f96290b5786ce4335
    type: DATE_RANGE
    label: Week range
    settings:
      description: >-
        Weeks within this range are included. The latest week in the range is
        treated as "this week".
      unit: WEEK
      dayStartOfWeek: 0
    defaultValue:
      type: RELATIVE_DATE_RANGE
      relativeDateRangeOffsetStart: -84
      relativeDateRangeOffsetEnd: 0
---

# Weekly Search Trends Report (Template)

```yaml {.page}
type: DOC_PAGE
id: 6a33777e26f486d9eebb4161
width:
  width_type: FIXED
  fixed_width: 1024
notebook_param_widget_values:
  - id: 6a33777ee2d06b5b770b75a4
    value: United Kingdom
  - id: 6a33777f96290b5786ce4335
    value:
      - '2026-01-01'
      - '2026-06-30'
```

This report summarizes the **weekly search trends** for a selected country, based on the BigQuery public dataset **Google Trends** (`bigquery-public-data.google_trends`).

- **Data sources**:`international_top_terms` (top search terms) /`international_top_rising_terms` (rising terms)
- **Aggregation basis**: All figures are computed against the latest`refresh_date` partition of each table (a partition filter is always applied).
- **About countries and regions**: Each country (`country_name`) contains multiple regions (`region_name`), and terms are ranked per region. This report derives a nationwide ranking from a**national reach score** — the sum of each term's`score` across all regions (reach × popularity).
- **Definition of "this week"**: Among the weeks included in the "Week range" parameter below, the**latest week** is treated as "this week", and the week immediately before it as the "previous week".

> **How to use this template**
> Just change the **Country** and **Week range** parameters at the top to reuse this as a report for a different country or period. Because the queries reference the parameters and the latest `refresh_date`, every section re-aggregates automatically when you change them.

::p

## 1. Top Search Terms This Week

A nationwide ranking of the terms most searched in the target country during this week (the latest week in the selected range). `national_reach_score` is the sum of each term's `score` across all regions, reflecting both popularity and geographic breadth.

:::sql-block
```yaml {.attrs}
id: 6a33777ff0d566022c66ae9b
connId: 674e8bc1941af1739414a874
name: Top Search Terms This Week
defaultViewMode: SHOW_RESULTS
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674e8bc1941af1739414a874/pj=bigquery-public-data/ds=google_trends/tb=international_top_terms
  - type: PARAM_REF
    refParamKey: 6a33777ee2d06b5b770b75a4
    escapeType: STRING
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.1
    escapeType: DATE
```

```sql
-- Identify the latest week in range (= this week) from the latest refresh_date partition
WITH this_week AS (
  SELECT MAX(week) AS w
  FROM $1
  WHERE refresh_date = (SELECT MAX(refresh_date) FROM $1)
    AND country_name = $2
    AND week BETWEEN $3 AND $4
)
SELECT
  RANK() OVER (ORDER BY SUM(score) DESC) AS rank,
  term,
  SUM(score)                  AS national_reach_score,
  COUNT(DISTINCT region_name) AS regions
FROM $1
WHERE refresh_date = (SELECT MAX(refresh_date) FROM $1)
  AND country_name = $2
  AND week = (SELECT w FROM this_week)
GROUP BY term
ORDER BY national_reach_score DESC, term
LIMIT 20
```

```yaml {.result}
height: 330px
```
:::

```yaml {.chart}
id: 6a33777f745f9ec84a4b986f
sqlId: 6a33777ff0d566022c66ae9b
chartSettings:
  template_type: XY_CHART_V2
  component_settings:
    name: Top Search Terms This Week (national reach score)
    enable_auto_dispatch: true
    x:
      - field: term
    views:
      - view_id: 6a33777f0040d6afff21a864
        render_type: BAR
        ys:
          - field: national_reach_score
            aggregator: SUM
    group_by: []
    order_by: VALUE_DESC
  display_settings:
    transpose: true
    x_axis:
      name: Term
    y_axis:
      name: National reach score
    series:
      6a33777f0040d6afff21a864-national_reach_score___sum:
        formatter:
          formatter_number: ','
        value_label:
          show: true
          formatter:
            formatter_number: ','
height: 560px
```

::p

## 2. Rising Terms This Week

Terms whose searches surged in the target country this week. A higher `max_percent_gain` (the maximum regional growth rate) indicates a term that drew sudden attention over a short period.

:::sql-block
```yaml {.attrs}
id: 6a3377b403e6ef4cea328567
connId: 674e8bc1941af1739414a874
name: Rising Terms This Week
defaultViewMode: SHOW_RESULTS
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674e8bc1941af1739414a874/pj=bigquery-public-data/ds=google_trends/tb=international_top_rising_terms
  - type: PARAM_REF
    refParamKey: 6a33777ee2d06b5b770b75a4
    escapeType: STRING
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.1
    escapeType: DATE
```

```sql
WITH this_week AS (
  SELECT MAX(week) AS w
  FROM $1
  WHERE refresh_date = (SELECT MAX(refresh_date) FROM $1)
    AND country_name = $2
    AND week BETWEEN $3 AND $4
)
SELECT
  RANK() OVER (ORDER BY SUM(score) DESC, MAX(percent_gain) DESC) AS rank,
  term,
  SUM(score)                  AS national_reach_score,
  MAX(percent_gain)           AS max_percent_gain,
  COUNT(DISTINCT region_name) AS regions
FROM $1
WHERE refresh_date = (SELECT MAX(refresh_date) FROM $1)
  AND country_name = $2
  AND week = (SELECT w FROM this_week)
GROUP BY term
ORDER BY national_reach_score DESC, max_percent_gain DESC, term
LIMIT 20
```

```yaml {.result}
height: 330px
```
:::

::p

## 3. Rank Changes vs. Previous Week (New Entries & Climbers)

Compares the nationwide ranking of this week and the previous week to surface **newly ranked-in terms (NEW)** and **terms that climbed (UP)**. `rank_change` is "previous-week rank − this-week rank"; a larger value means a bigger jump up the ranking (NEW means the term had no data for the previous week).

:::sql-block
```yaml {.attrs}
id: 6a3377b4fd24887320e310f7
connId: 674e8bc1941af1739414a874
name: Rank Changes vs. Previous Week
defaultViewMode: SHOW_RESULTS
bindings:
  - type: TABLE_REF
    tableResourceId: >-
      bq/cn=674e8bc1941af1739414a874/pj=bigquery-public-data/ds=google_trends/tb=international_top_terms
  - type: PARAM_REF
    refParamKey: 6a33777ee2d06b5b770b75a4
    escapeType: STRING
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.0
    escapeType: DATE
  - type: PARAM_REF
    refParamKey: 6a33777f96290b5786ce4335.1
    escapeType: DATE
```

```sql
WITH rd AS (
  SELECT MAX(refresh_date) AS d FROM $1
),
-- The latest week in range (= this week)
this_week AS (
  SELECT MAX(week) AS w
  FROM $1
  WHERE refresh_date = (SELECT d FROM rd)
    AND country_name = $2
    AND week BETWEEN $3 AND $4
),
-- The week immediately before this week (= previous week)
prev_week AS (
  SELECT MAX(week) AS w
  FROM $1
  WHERE refresh_date = (SELECT d FROM rd)
    AND country_name = $2
    AND week < (SELECT w FROM this_week)
),
-- Nationwide rank for this week and the previous week
ranked AS (
  SELECT
    week,
    term,
    RANK() OVER (PARTITION BY week ORDER BY SUM(score) DESC) AS rk
  FROM $1
  WHERE refresh_date = (SELECT d FROM rd)
    AND country_name = $2
    AND week IN ((SELECT w FROM this_week), (SELECT w FROM prev_week))
  GROUP BY week, term
),
cur AS (SELECT term, rk FROM ranked WHERE week = (SELECT w FROM this_week)),
prv AS (SELECT term, rk FROM ranked WHERE week = (SELECT w FROM prev_week))
SELECT
  cur.rk                                    AS this_week_rank,
  cur.term,
  prv.rk                                    AS prev_week_rank,
  IF(prv.rk IS NULL, NULL, prv.rk - cur.rk) AS rank_change,
  IF(prv.rk IS NULL, 'NEW', 'UP')           AS status
FROM cur
LEFT JOIN prv USING (term)
-- Keep only terms in this week's top tier that are new entries or climbers
WHERE cur.rk <= 25
  AND (prv.rk IS NULL OR prv.rk - cur.rk > 0)
ORDER BY (prv.rk IS NULL) DESC, rank_change DESC, this_week_rank
LIMIT 25
```

```yaml {.result}
height: 360px
```
:::

::p

## 4. This Week's Notes

> This section is a free-text area. Use it to capture what you noticed about this week's trends — observations, likely background causes, and next actions.

**This week's highlights**

- (e.g.) The top ranking was dominated by ◯◯-related terms; sports / current-event topics had a strong impact.

**What the rising terms suggest**

**Changes vs. last week / points to watch**

**Next actions / memo**

::p
