# Codatum CLI examples

Worked examples and use cases for the [Codatum CLI (`cdm`)](https://cli-docs.codatum.com/) — a command-line tool for editing Codatum notebooks (`.cnb.md`) locally and syncing them with the server.

Each use case is a self-contained example you can clone, point at your own Codatum workspace, and run. They're designed to be driven by an AI coding agent: you describe what you want, and the agent uses `cdm` to explore data, write SQL, build charts, and sync your work. This repo includes agent setup for both [Claude Code](https://cli-docs.codatum.com/guide/editor-setup/claude-code) and Cursor, and the examples are walked through using Claude Code.

- **CLI documentation**: https://cli-docs.codatum.com/
- **Codatum**: https://codatum.com/

## Use cases

| # | Use case | What it shows |
|---|----------|---------------|
| 01 | [Build a dashboard through conversation](./use-cases/01-build-dashboard/) | Create a notebook from scratch by talking to Claude Code — explore tables, build charts, add parameters, and lay out a dashboard. |
| 02 | [Build from a template](./use-cases/02-build-from-template/) | Turn a recurring report into a reusable template, then regenerate it from a single prompt — including web research and write-up. |
| 03 | [Manage with GitHub](./use-cases/03-manage-with-github/) | Keep notebooks in GitHub as the source of truth — validate changes in pull requests and deploy to Codatum on merge to `main`. |

Each use case has its own README with the full walkthrough and a link to the corresponding [documentation](https://cli-docs.codatum.com/use-cases/build-dashboard).

## Getting started

1. **Install and authenticate the `cdm` CLI.** See [Getting started](https://cli-docs.codatum.com/guide/getting-started).
2. **Set up your agent.** This repo includes the cdm skill for both Claude Code (`.claude/skills/cdm/SKILL.md`) and Cursor (`.cursor/skills/cdm/SKILL.md`) in the repo root. Open your agent from the repo root so it picks up the skill. The walkthroughs use Claude Code; see [Claude Code setup](https://cli-docs.codatum.com/guide/editor-setup/claude-code).
3. **Pick a use case** from the table above and follow its README.

You'll need a Codatum workspace with a BigQuery connection that can query `bigquery-public-data`, since the examples are built on public datasets.

## Layout

```
codatum-cli-examples/
├── .claude/skills/cdm/SKILL.md   # cdm skill for Claude Code
├── .cursor/skills/cdm/SKILL.md   # cdm skill for Cursor
└── use-cases/
    ├── 01-build-dashboard/
    ├── 02-build-from-template/
    └── 03-manage-with-github/
```

The notebooks are committed with execution results stripped (`cdm notebook strip`) — they contain the SQL, chart definitions, parameters, and layout, but no data. Each one references a connection ID specific to the original workspace, so you'll point it at your own connection before running (each README explains how).

## License

See [LICENSE](./LICENSE).
