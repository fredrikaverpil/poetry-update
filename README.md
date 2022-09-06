# poetry-update

GitHub action to periodically update all (or a subset) of your Poetry project's dependencies.

The motivation behind this action is that there can sometimes be a considerable amount of fatigue in handling individual [dependabot](https://github.com/dependabot) updates.

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
      - uses: fredrikaverpil/poetry-update@v1.2
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
        with:
          delete-branch: true
          branch: poetry-update
          add-paths: |
            poetry.lock
          commit-message: "Update poetry dependencies"
          title: "Update poetry dependencies"
          body: |

            ### Updated dependencies:

            ```bash
            ${{ env.POETRY_UPDATED }}
            ```   

            ### Outdated dependencies _before_ PR:

            ```bash
            ${{ env.POETRY_OUTDATED_BEFORE }}
            ```

            ### Outdated dependencies _after_ PR:

            ```bash
            ${{ env.POETRY_OUTDATED_AFTER }}
            ```

            _Note: there may be dependencies in the table above which were not updated as part of this PR.
            The reason is they require manual updating due to the way they are pinned._
```

Customize the [`peter-evans/create-pull-request` action](https://github.com/peter-evans/create-pull-request) to you heart's content.

## Custom inputs and environment variables

You can optionally customize the commands of the action.

```yaml
- uses: fredrikaverpil/poetry-update@v1.2
  with:
    install_command: 'poetry install'  # initial installation
    show_outdated_command: 'poetry show --outdated'  # generate the output for the PR description
    update_command: 'poetry update'  # perform the desired update
```



## Dependency groups

Instead of updating all dependencies, you can limit the updates to one or more [dependency group](https://python-poetry.org/docs/master/managing-dependencies/#dependency-groups) using the `--only` `--with` or `--without` arguments to the `poetry update` command.

- ✋ This requires Poetry 1.2.0 or later.
- You could end up having unwanted dependencies updated which are part of another dependency group. It is currently unclear if this is a bug. For more information, see https://github.com/python-poetry/poetry/issues/5876.

```yaml
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: fredrikaverpil/setup-pipx@v1.5

      - name: Create PR for dev dependencies only (dependency group "dev")
        uses: fredrikaverpil/poetry-update@v1.2
        with:
          show_outdated_command: 'poetry show --outdated --only dev'
          update_command: 'poetry update --only dev'
```
