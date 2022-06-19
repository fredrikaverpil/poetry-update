# poetry-update

GitHub Action which will periodically create PRs if updates are available for dependencies specified in `poetry.lock`.

The motivation behind this PR is that there can sometimes be a considerable amount of fatigue in handling individual [dependabot](https://github.com/dependabot) updates.

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
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v3
        with:
          python-version: "3.10"
      - uses: fredrikaverpil/setup-pipx@v1.5
      - run: pipx install poetry
      - uses: fredrikaverpil/poetry-update@v1.1
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
        with:
          delete-branch: true
          branch: poetry-update
          add-paths: poetry.lock
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

## Custom inputs

You can optionally customize the commands of the action.

```yaml
- uses: fredrikaverpil/poetry-update@v1.1
  with:
    cmd_install: 'poetry install'  # initial installation
    cmd_show: 'poetry show --outdated'  # generate the output for the PR description
    cmd_update: 'poetry update'  # perform the desired update
```

## Dependency groups

Instead of updating all dependencies, you limit the updates to a specific [dependency group](https://python-poetry.org/docs/master/managing-dependencies/#dependency-groups).

⚠️ Warnings:

- The required poetry arguments are currently in beta.
- You could end up having unwanted dependencies updated which are part of another dependency group. It is currently unclear if this is a bug. For more information, see https://github.com/python-poetry/poetry/issues/5876.

```yaml
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: fredrikaverpil/setup-pipx@v1.5

      - run: pipx install --suffix=@1.2.0b2 --force git+https://github.com/python-poetry/poetry.git@1.2.0b2

      - name: Create PR for dev dependencies only
        uses: fredrikaverpil/poetry-update@v1.1
        cmd_show: 'poetry@1.2.0b2 show --outdated --only dev'
        cmd_update: 'poetry@1.2.0b2 update --only dev'
```
