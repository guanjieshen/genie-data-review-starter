# Genie Data Review Starter

> ⚠️ **Archived**
> This repository is no longer maintained. Use the [Databricks GitHub Actions template](https://github.com/guanjieshen/databricks-github-actions-template) to start a project, and the [demo in the action repository](https://github.com/guanjieshen/genie_code_code_review_gh_action/tree/main/examples/demo) to see Genie Data Review on a pull request with planted issues.

![Template](https://img.shields.io/badge/GitHub-template-blue)
![Action](https://img.shields.io/badge/action-genie__code__code__review__gh__action-orange)

## Overview

This repository is a starter template for **[Genie Data Review](https://github.com/guanjieshen/genie_code_code_review_gh_action)**, a GitHub Action that reviews Databricks pull requests against the live workspace using **Genie Code** and **Genie One**.

It contains a small Databricks Asset Bundle (an orders ETL job), the workflow that runs the review, example reviewer settings, and a demo that sets up sample data and opens a pull request with 11 planted data issues. Use it to see the review working in your own workspace in about 15 minutes, then copy the workflow into your real repositories.

```
.
├── databricks.yml                      bundle (set your catalog, schema, warehouse here)
├── resources/orders_etl.job.yml        job: silver (Python + shared helper) → gold (SQL)
├── src/                                job code
├── .github/workflows/genie-data-review.yml   the review workflow
├── .genie-review/                      reviewer config and team instructions
└── demo/                               sample data, demo pull request, expected findings
```

## Prerequisites

- A Databricks workspace with Unity Catalog and Unity Gateway, where Genie Code is available.
- A catalog and schema you can create tables in, and a SQL warehouse.
- The [Databricks CLI](https://docs.databricks.com/aws/en/dev-tools/cli/install) authenticated to the workspace, and the [GitHub CLI](https://cli.github.com/).

## Getting started

### 1. Create your copy

Select **Use this template** on GitHub to create a repository, then clone it.

### 2. Point the bundle at your workspace

Edit the `staging` target in `databricks.yml`:

```yaml
    variables:
      catalog: main                 # a catalog you can create a schema in
      schema: genie_review_demo     # created by the demo setup
      warehouse_id: 1234abcd5678ef90
```

### 3. Set up the demo data

The setup script creates the demo tables (with comments, tags, a deprecated table, and a governed metric view), deploys and runs the job once so lineage is recorded, and then edits the deployed job to simulate an on-call hotfix.

```bash
DATABRICKS_CONFIG_PROFILE=<your-profile> python3 demo/setup.py
```

Commit the `databricks.yml` change and push it to `main`.

### 4. Create the reviewer identity

Create a read-only service principal and grant it access to the demo schema, following the [setup guide](https://github.com/guanjieshen/genie_code_code_review_gh_action/blob/main/docs/setup.md). For this demo:

```sql
GRANT USE CATALOG, BROWSE ON CATALOG main TO `<application-id>`;
GRANT USE SCHEMA ON SCHEMA main.genie_review_demo TO `<application-id>`;
```

Also give it `CAN_VIEW` on the `genie_review_demo_orders_etl` job and `CAN_USE` on the warehouse.

### 5. Add repository variables and secrets

| Name | Type | Value |
|---|---|---|
| `DATABRICKS_HOST` | Variable | Workspace URL |
| `GENIE_REVIEWER_CLIENT_ID` | Variable | Service principal application ID |
| `GENIE_REVIEWER_CLIENT_SECRET` | Secret | Service principal OAuth secret |
| `GENIE_MODEL_HOST` | Variable (optional) | A workspace that serves the Genie Code model, if yours does not |

### 6. Open the demo pull request

```bash
./demo/create_demo_pr.sh
```

The review posts as a comment within a few minutes. Compare it with [demo/EXPECTED.md](demo/EXPECTED.md).

> 💡 **Tip**
> Turn on GitHub Copilot code review on the same pull request to see the two layers side by side. Copilot covers the plain logic bug (revenue counted once per order item); Genie Data Review covers the issues that need the workspace.

## Use it in your own repository

Copy `.github/workflows/genie-data-review.yml` into your bundle repository, set `bundle-target` to the target you deploy to, and add the same variables and secrets. Optionally copy `.genie-review/` and edit the team instructions.

> ⚠️ **Warning**
> This demo deploys to `/Workspace/Shared` so the reviewer can find the deployed job. In real repositories, deploy to a restricted folder and give the reviewer identity `CAN_READ` on it.

## Clean up

```bash
databricks bundle destroy
```

Then drop the demo schema: `DROP SCHEMA <catalog>.<schema> CASCADE`.
