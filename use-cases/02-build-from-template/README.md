# Build from a template

A reusable Codatum notebook (`.cnb.md`) template for a recurring report. Each time you need an updated report, Claude Code clones this template, points it at the latest data, runs it, researches the background of notable results on the web, writes up the findings, and pushes — all from a single prompt.

This is the worked example for the [use case documentation](https://cli-docs.codatum.com/use-cases/build-from-template). See that page for the full walkthrough.

## What's here

```
02-build-from-template/
├── Weekly-Search-Trends(Template).cnb.md   # the report template
└── Weekly-Search-Trends(Template).png       # what a generated report looks like
```

The template builds a weekly search-trends report from the public BigQuery dataset `bigquery-public-data.google_trends`. It has two parameters — the target country and the week range — and three sections: the week's top search terms, the top rising terms, and the rank changes from the previous week (new entries and climbers). All queries are anchored to the latest `refresh_date` partition, so each run reflects the most recent data.

The last section, "This Week's Notes", is left as a placeholder. When you run the report, Claude Code reads the results, looks up the background of the rising terms on the web, and fills this section with its findings. The committed template keeps this section empty on purpose — the commentary is meant to be generated per run, not shipped in the repo.

The `.cnb.md` is committed with execution results stripped (`cdm notebook strip`), so it contains the parameters, SQL, and chart definitions — but no data. Run it yourself to populate it.

## Prerequisites

- A Codatum workspace with a BigQuery connection that can query `bigquery-public-data`
- The `cdm` CLI installed and authenticated (`cdm auth login`). See [Getting started](https://cli-docs.codatum.com/guide/getting-started).
- An AI coding agent set up to use `cdm`. This repo includes the cdm skill for Claude Code (`.claude/skills/cdm/SKILL.md`) and Cursor (`.cursor/skills/cdm/SKILL.md`) in the repo root — open your agent from the repo root so it picks up the skill. This walkthrough uses Claude Code; see [Claude Code setup](https://cli-docs.codatum.com/guide/editor-setup/claude-code).
- This builds on the [Build a dashboard](https://cli-docs.codatum.com/use-cases/build-dashboard) use case — start there if you're new to `cdm`.

## Try it

The template references a connection ID and table resource IDs that are specific to the original workspace. Before running it, point it at your own connection.

The easiest way is to ask Claude Code, from inside this folder:

> Update `Weekly-Search-Trends(Template).cnb.md` to use my own BigQuery connection for the google_trends tables, then run it.

Claude Code will look up your connection with `cdm connection list`, rewrite the `connId` and table resource IDs, and run the notebook to populate it.

Once it runs, the intended workflow is to keep this template on the server and generate a fresh report from it each time with a single prompt — clone the template, update the period, run, research the rising terms on the web, write the notes, push, and render. See the [use case documentation](https://cli-docs.codatum.com/use-cases/build-from-template) for the full prompt.

If you'd rather run the template directly by hand:

1. Find your connection ID:
   ```sh
   cdm connection list
   ```
2. In `Weekly-Search-Trends(Template).cnb.md`, replace the existing connection ID (in each SQL block's `connId` and in the `tableResourceId` bindings) with yours.
3. Run it to populate the report:
   ```sh
   cdm notebook run "Weekly-Search-Trends(Template).cnb.md"
   ```
4. Open it in the browser to view and keep editing:
   ```sh
   cdm notebook preview "Weekly-Search-Trends(Template).cnb.md"
   ```
   