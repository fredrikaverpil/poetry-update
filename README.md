# poetry-update

GitHub Action which will periodically create PRs if updates are
available for dependencies specified in `poetry.lock`.

## Quickstart

Add a new GitHub Actions workflow in `.github/workflows/poetry.yaml`:

```yaml
name: poetry

on:
  schedule:
    - cron: "0 9 * * 1" # Mondays at 09:00
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: "3.10"
      - uses: fredrikaverpil/setup-pipx@v1.5
      - run: pipx install poetry
      - uses: fredrikaverpil/poetry-update@v1.1
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          delete-branch: true
          branch: poetry-update
          commit-message: "Update poetry dependencies"
          title: "Update poetry dependencies"
          body: |

            ### Outdated dependencies:

            ```bash
            ${{ env.POETRY_OUTDATED }}
            ```

            _Note: there may be dependencies in the table above which were not updated as part of this PR.
            The reason is they require manual updating._
```

Customize the [`peter-evans/create-pull-request` action](https://github.com/peter-evans/create-pull-request) to you heart's content.

## Advanced usage

You can optionally customize the commands of the action.

```yaml
- uses: fredrikaverpil/poetry-update@v1.1
  with:
    cmd_install: 'poetry install'  # initial installation
    cmd_show: 'poetry show --outdated'  # generate the output for the PR description
    cmd_update: 'poetry update'  # perform the desired update
```
