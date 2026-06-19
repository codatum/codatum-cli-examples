# Manage with GitHub

Keep your Codatum notebooks (`.cnb.md`) in GitHub as the source of truth. Changes go through pull requests, get validated by GitHub Actions, and are pushed to Codatum only after they're merged to `main`. The Codatum notebook is set to "query only" so its structure can't be edited in the UI — structural changes flow through GitHub, while viewers can still adjust parameters and run queries.

This is the worked example for the [use case documentation](https://cli-docs.codatum.com/use-cases/manage-with-github). See that page for the full walkthrough.

## What's here

```
03-manage-with-github/
└── .github/
    └── workflows/
        ├── validate.yml   # validates .cnb.md on pull requests
        └── deploy.yml      # pushes to Codatum on merge to main
```

This use case is about the workflow around your notebooks, so the example ships only the two GitHub Actions workflows. It doesn't include a notebook of its own — bring your own under `notebooks/` in your repo. The dashboard from the [Build a dashboard](https://cli-docs.codatum.com/use-cases/build-dashboard) use case works well as a starting point.

The two workflows:

- **validate.yml** runs on pull requests. It validates schema and syntax (`cdm notebook validate`) and checks that job execution metadata has been stripped (`cdm notebook strip --check`). Formatting (`cdm notebook format`) and stripping (`cdm notebook strip`) rewrite files, so they're done locally before committing rather than in CI. It does not run any SQL, so it stays fast and doesn't touch your data sources. It uses the same `CDM_PAT` secret as the deploy workflow.
- **deploy.yml** runs when changes land on `main`. It pushes the notebooks to Codatum (`cdm notebook push --yes --ignore-job-metadata`). The `--ignore-job-metadata` flag means it reflects only the structure and leaves job execution state untouched, so it doesn't overwrite the latest results that Codatum refreshes on its own. It uses an edit PAT scoped to the target folder.

GitHub manages the notebook *structure* (SQL, charts, parameters, layout). Data refresh (running jobs) happens on the Codatum side, and job execution state is kept out of Git — locally you `cdm notebook strip` before committing, and in CI you push with `--ignore-job-metadata`. This keeps execution churn from showing up as diffs.

## Prerequisites

- A Codatum workspace, with the target notebook(s) set to **"query only"** (notebook menu → Query only) so their structure can only be changed through GitHub, while viewers can still adjust parameters and run queries
- Two PATs (see [PAT permissions](https://cli-docs.codatum.com/cli/pat)):
  - an **edit** PAT, scoped to the target folder — added to GitHub Secrets, used by both workflows
  - a **read-only** PAT — kept only on each member's machine for local work (clone/pull/diff/preview), never added to Secrets, so nobody can push from their own machine by mistake
- The `cdm` CLI installed and authenticated locally with the read-only PAT (`cdm auth login`). See [Getting started](https://cli-docs.codatum.com/guide/getting-started).

## Set it up

In your own repository:

1. **Add the workflows.** Copy `validate.yml` and `deploy.yml` into `.github/workflows/`. Both reference the `CDM_PAT` secret and watch `notebooks/**/*.cnb.md` — adjust the paths and secret name to match your repo.
2. **Add the edit PAT as a repository secret.** In your GitHub repo settings, add `CDM_PAT` — the edit PAT, scoped to the target folder. Both workflows use it. The read-only PAT stays on each member's machine and is never added to Secrets.
3. **Add your notebooks under `notebooks/`.** Clone each query-only notebook, strip its execution state, and commit it:
   ```sh
   cdm notebook clone <notebook> -o notebooks/
   cdm notebook strip notebooks/
   ```
   A cloned `.cnb.md` references a connection ID specific to the original workspace, so point it at your own connection first (see the [Build a dashboard](https://cli-docs.codatum.com/use-cases/build-dashboard) example for how).

Once it's wired up, the flow is: edit locally → open a PR (validation runs) → merge to `main` (deploy runs).
