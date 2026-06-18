# Build a dashboard through conversation

Create a Codatum notebook (`.cnb.md`) from scratch by talking to Claude Code. You describe what you want to see; Claude Code explores the tables, writes the SQL, builds the charts, and checks the result by rendering it to an image — while you review the live preview in your browser.

This is the worked example for the [use case documentation](https://cli-docs.codatum.com/use-cases/build-dashboard). See that page for the full walkthrough.

## What's here

```
01-build-dashboard/
├── theLook-Sales-Dashboard.cnb.md        # the finished dashboard
├── theLook-Sales-Dashboard-page1-doc.png  # the doc page (chart definitions)
└── theLook-Sales-Dashboard-page2-grid.png # the grid page (dashboard layout)
```

The dashboard shows sales by category, key KPIs (total sales / order count / AOV), a monthly sales trend, sales by country, and an order heatmap by weekday and hour. It has a date-range parameter to filter the period, and a grid page that lays everything out as a dashboard.

It's built on the public BigQuery dataset `bigquery-public-data.thelook_ecommerce` (a fictitious e-commerce store), so you can reproduce it with your own BigQuery connection.

The `.cnb.md` is committed with execution results stripped (`cdm notebook strip`), so it contains the chart definitions, SQL, parameters, and grid layout — but no data. Run it yourself to populate the charts.

## Prerequisites

- A Codatum workspace with a BigQuery connection that can query `bigquery-public-data`
- The `cdm` CLI installed and authenticated (`cdm auth login`). See [Getting started](https://cli-docs.codatum.com/guide/getting-started).
- An AI coding agent set up to use `cdm`. This repo includes the cdm skill for Claude Code (`.claude/skills/cdm/SKILL.md`) and Cursor (`.cursor/skills/cdm/SKILL.md`) in the repo root — open your agent from the repo root so it picks up the skill. This walkthrough uses Claude Code; see [Claude Code setup](https://cli-docs.codatum.com/guide/editor-setup/claude-code).

## Try it

The notebook references a connection ID and table resource IDs that are specific to the original workspace. Before running it, point it at your own connection.

The easiest way is to ask Claude Code, from inside this folder:

> Update `theLook-Sales-Dashboard.cnb.md` to use my own BigQuery connection for the theLook tables, then run it.

Claude Code will look up your connection with `cdm connection list`, rewrite the `connId` and table resource IDs, and run the notebook to populate the charts.

If you'd rather do it by hand:

1. Find your connection ID:
   ```sh
   cdm connection list
   ```
2. In `theLook-Sales-Dashboard.cnb.md`, replace the existing connection ID (in each SQL block's `connId` and in the `tableResourceId` bindings) with yours.
3. Run the notebook to populate the charts:
   ```sh
   cdm notebook run theLook-Sales-Dashboard.cnb.md
   ```
4. Open it in the browser to view and keep editing:
   ```sh
   cdm notebook preview theLook-Sales-Dashboard.cnb.md
   ```
   