# poetry-update-action

GitHub Action which will periodically create PRs if updates are
available for dependencies specified in `poetry.lock`.

## Quickstart

Add a new GitHub Actions workflow in `.github/workflows/poetry.yaml`:

```yaml
name: poetry

on:
  schedule:
    - cron: "0 8 * * *" # every morning at 08:00
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: "3.10"
      - uses: fredrikaverpil/pipx-action@v1.3
      - run: pipx install poetry
      - uses: fredrikaverpil/poetry-update-action@v1

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          delete-branch: true
          branch: poetry-update
          commit-message: "Update poetry dependencies"
          title: "Update poetry dependencies"
          body: |

            ```bash
            $ poetry show --outdated

            ${{ env.POETRY_OUTDATED }}
            ```

            _Note: there may be dependencies in the table above which were not updated as part of this PR.
            The reason is they require manual updating._
```

Customize the `peter-evans/create-pull-request` action to you heart's content.
