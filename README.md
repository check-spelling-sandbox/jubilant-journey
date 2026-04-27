# update-action-runtime

GitHub actions are maintained by individuals and organizations often on a best effort basis. Sometimes projects don't get around to updating their [`runs.using`](https://docs.github.com/en/actions/reference/workflows-and-actions/metadata-syntax?versionId=free-pro-team%40latest&productId=actions&restPage=reference%2Cworkflows-and-actions%2Cworkflow-syntax#runsusing-for-javascript-actions) field before GitHub triggers a deprecation round.

You can use this action in a workflow to override the `runs.using` field of another action that has not run yet (as long as the action is not only referenced by a dynamically triggered composite action -- see [Using with dynamic actions](#using-with-dynamic-actions) for how to make it work).

Note that one constraint people have is support for GHES (e.g. [GHES 3.20](https://docs.github.com/en/enterprise-server@3.20)) where a newer version for `using` may not be available until long after the [old version is deprecated](https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/) for ["FPT" (https://github.com)](https://docs.github.com/en). It's possible to address this by using an `if: ${{ github.server_url == 'https://github.com' }}` guard.

## inputs

### `config`

JSON array of objects

Key|Value
-|-
`path`|Regular Expression to limit actions to check (directories are of the form `owner/repository/reference/(optional-sub-path/)`; and the file will be either `action.yml` or `action.yaml`)
`old`|The old runtime (e.g. `node20`)
`new`|The new runtime (e.g. `node24`)

#### Example
```jsonc
[
    {
        "path": "github/dependabot-action/v2",
        "old": "node20",
        "new": "node24"
    }
    ...
]
```

## outputs

### `updated-actions`

JSON array of update actions


#### Example

```jsonc
[{ "path": "path-of-updated-action", "old": "node20", "new": "node24"}]
```

## Sample workflow

```yml
name: Test

on:
  push:
  workflow_dispatch:

permissions: {}

jobs:
  test:
    name: test
    permissions:
      contents: read
      actions: read
    runs-on: ubuntu-latest
    steps:
      - name: Update action runtime
        uses: check-spelling-sandbox/jubilant-journey@main
        with:
          config: |
            [
              {
                "path": "github/dependabot-action",
                "old": "node20",
                "new": "node24"
              }
            ]
        # restrict update to github.com (leaving GHES instances running the older version)
        if: ${{ github.server_url == 'https://github.com' }}

      - name: checkout
        uses: actions/checkout@v6

      - name: dependabot-action
        uses: github/dependabot-action@v2
```

## Using with dynamic actions

If you will be using a composite action that uses a local composite action that in turn uses an action, you will need to trigger a download in order for this to work.

```yml
name: Test

on:
  push:

permissions: {}

jobs:
  test:
    name: test
    permissions:
      contents: read
      actions: read
    runs-on: ubuntu-latest
    steps:
      - name: Update action runtime
        uses: check-spelling-sandbox/jubilant-journey@main
        with:
          config: |
            [
              {
                "path": "github/dependabot-action",
                "old": "node20",
                "new": "node24"
              }
            ]

      - name: checkout
        uses: actions/checkout@v6

      - name: Use an action that dynamically triggers dependabot-action
        uses: ./call-dependabot-action

      - name: dependabot-action
        uses: github/dependabot-action@v2
        if: false # this will force the actions runtime to download the action even though it won't run
```
