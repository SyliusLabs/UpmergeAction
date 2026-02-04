# Upmerge Action

A GitHub Action that creates upmerge PRs between branches when there are changes to merge.

## Usage

```yaml
name: Upmerge PR

on:
    schedule:
        - cron: "0 2 * * *"
    workflow_dispatch: ~

permissions:
    contents: write
    pull-requests: write

jobs:
    upmerge:
        runs-on: ubuntu-latest
        strategy:
            fail-fast: false
            matrix:
                include:
                    - base_branch: "1.0"
                      target_branch: "1.1"
                    - base_branch: "1.1"
                      target_branch: "1.2"

        steps:
            - uses: actions/checkout@v4
              with:
                  ref: ${{ matrix.target_branch }}

            - uses: SyliusLabs/UpmergeAction@v1
              with:
                  base_branch: ${{ matrix.base_branch }}
                  target_branch: ${{ matrix.target_branch }}
                  token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `base_branch` | Yes | Source branch to upmerge from (e.g., `1.0`) |
| `target_branch` | Yes | Target branch to upmerge to (e.g., `1.1`) |
| `token` | Yes | GitHub token with permissions to create PRs |

## Outputs

| Output | Description |
|--------|-------------|
| `created` | Whether a PR was created (`true` or `false`) |
| `pr_number` | PR number if created, empty otherwise |
| `pr_url` | PR URL if created, empty otherwise |

## Behavior

- Creates a PR from `upmerge/{base}_{target}` branch to merge changes from base to target
- If target already contains all changes from base, succeeds without creating a PR
- If an upmerge branch was manually modified (e.g. conflict resolution), it is preserved — the action skips to avoid overwriting work
- Unmodified upmerge branches are recreated with the latest base to keep them fresh
- Uses `gh` CLI for PR creation (no external action dependencies)
