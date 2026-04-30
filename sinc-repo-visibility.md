# Make Repositories Public on Release Date

## Overview

This reusable GitHub Actions workflow automatically manages the **visibility of GitHub repositories** based on a configured release date. It ensures that repositories:

- remain **private before** a defined release date,
- become **public on or after** that release date, and
- are automatically **reverted to private** if they are accidentally made public too early.

The workflow is intended for use in **advice publication and registry repositories**, where release timing must be enforced consistently and auditable.

The workflow is designed to be:
- **Idempotent** (safe to run repeatedly),
- **Fail-soft** (issues with one repository do not stop others), and
- **Reusable** via `workflow_call`.

---

## How It Works

For each entry in `repos.json`, the workflow:

1. Reads the target repository name and release date.
2. Queries the GitHub API to determine the repository’s current visibility.
3. Compares the current time (UTC) with the configured release date.
4. Applies one of the following actions.

### Visibility rules

| Situation | Action |
|---------|--------|
| Repository is **public before** release date | Revert to **private** |
| Repository is **private on or after** release date | Make **public** |
| Repository is **already public on the release day** | No change (reported) |
| Repository already in correct state | No action |

All decisions are logged and summarised.

---

## Input File (`repos.json`)

The workflow expects a top-level JSON **array** with one object per repository.

### Minimal example

```json
[
  {
    "repo_name": "2026_her.27.irls",
    "publish_after_utc": "2026-04-30 10:00"
  }
]
```

### Optional fields

| Field | Description |
|------|-------------|
| `repo_name` | Name of the repository to manage (required) |
| `publish_after_utc` | Release date/time in UTC (required) |
| `org_target` | Target GitHub organisation (optional; defaults to workflow input) |

---

## Workflow Inputs

This workflow is called via `workflow_call` and supports the following inputs:

| Input | Description | Default |
|------|-------------|---------|
| `target_org` | Default target organisation for repositories | `ices-advice` |

---

## Required Secrets

| Secret | Purpose |
|--------|---------|
| `CROSS_ORG_TOKEN` | GitHub token with permission to read and modify repository visibility |
| `TEAMS_WEBHOOK_URL` | *(Optional)* Microsoft Teams incoming webhook URL |

If `TEAMS_WEBHOOK_URL` is not provided, Teams notifications are skipped automatically.

---

## Notifications

If a Teams webhook is provided, the workflow posts a summary message that may include:

- ✅ Newly published repositories
- 🔒 Repositories reverted to private
- ℹ️ Repositories already public on the release day
- ⚠️ Authentication issues (e.g. expired token)

No message is sent if **no action was required**.

---

## Error Handling and Safety

- A `401 Unauthorized` response from the GitHub API is **captured and reported**, but does not stop the workflow.
- Errors affecting a single repository **do not abort the workflow** for other repositories.
- The workflow can be safely run on a schedule (e.g. daily or hourly).

---

## Example Usage (Calling Repository)

```yaml
name: Enforce repository release dates

on:
  workflow_dispatch:
  schedule:
    - cron: "0 12 * * *" # at 12 UTC every day

jobs:
  publish-after-date:
    uses: ices-tools-prod/workflows/.github/workflows/publish-after-date.yml@v1
    with:
      target_org: "ices-advice"
    secrets:
      CROSS_ORG_TOKEN: ${{ secrets.CROSS_ORG_TOKEN }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

---

## Design Notes

- All date comparisons are performed in **UTC**, using calendar-day logic where appropriate.
- The workflow does **not** assume ownership of repository creation — only visibility.
- Defaults are encoded centrally to reduce duplication and configuration errors.
- The workflow is version-pinnable to ensure **reproducibility and auditability**.

---

## Intended Use

This workflow is suitable for:

- Advice release governance
- Automated publication embargoes
- Registry or documentation repositories with strict release controls

It is **not** intended to manage repository content, tags, or releases — only visibility.
