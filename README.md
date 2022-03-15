# Github Actions Notes
My personal notes about Github Actions

## Run
* Under Linux, the run command does not need to start with `#!/bin/bash` as this is implied.
* By default, `run` beings with `/usr/bin/bash -e {0}`.
* * The `-e` is similar to `set -e` - meaning the script will abort when any commands return a non-zero exit code.
* * `{0}` refers to an auto-generated shell script such as:
* * *  `/home/runner/work/_temp/a894e763-fea0-48ac-ba15-92e8eda24f4d.sh`
* * * The contents of the `run` command have been saved to this file by Github Actions.

## Action Shell
* A [list of all Github Action shells](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell), including supported platforms

* You can use a different shell for the `run` command:

```yaml
      - name: python command
        run: |
          import platform
          print(platform.processor())
        shell: python
```

* You can use both `powershell` and `bash` under Windows:

```yaml
  run-windows-commands:
    runs-on: windows-latest
    steps:
      - name: Directory PowerShell
        run: Get-Location
      - name: Directory Bash
        run: pwd
        shell: bash
```

## Job Dependencies

* Jobs will run in parallel by default.
* By using `needs` in a step, you can make it depend on a previous step, thus making Github Actions run in serial instead of parallel

![Github_Actions_Run_Needs_Depend](github_actions_run_needs_depend.png)

## Uses
`uses:` is used to run a third-party Github Action.

**Examples:**

* `actions/hello-world-javascript-action@v1`
* `actions/hello-world-javascript-action@master`
* `actions/hello-world-javascript-action@3ee2a0320c9193ab716b86a9dae253f5c16bae62`
* * After the `@` symbol, a `branch name`, `tag` or `commit` must be referenced.

The `with:` keyword provides an input to a `uses` actions:

![Uses Input](github_actions_uses_input.png)

* Note that `uses:` references a Github project: https://github.com/actions/hello-world-javascript-action

`Outputs` can also be obtained by giving your `with` input an `id:` and then referencing it with a GitHub Action generated macro:

* `${{ steps.id_name.outputs.output_name }}`
* * `id_name` is defined in the input step by using `id:`
* * `output_name` is a predefined output from the included Github Action itself
* * * This name will be found in the documentation of the imported action

![Uses Input and Output with ID](github_actions_uses_input_output_with_id.png)

## Checkout Action

By default, Github Actions does not clone your repository into the Github Actions virtual machine.

Github provides an action to do this: https://github.com/actions/checkout

**Example:**

```yaml
    - name: Checkout
      uses: actions/checkout@v1
```

**Alternative:** *Not recommended - just proof of concept!* Instead of using `actions/checkout`, you could run these commands instead.  However, this example demonstrates the use of the built-in `GITHUB_*` environment variables.

```yaml
    steps:
      - name: List Files
        run: |
          pwd
          echo $GITHUB_SHA
          echo $GITHUB_REPOSITORY
          echo $GITHUB_WORKSPACE
          echo "${{ github.token }}"
          git clone https://github.com/$GITHUB_REPOSITORY
          cd *
          pwd
          git checkout $GITHUB_SHA
          ls -la
          md5sum *.md
```

## pull_request

You can run actions when a `PR` occurs by adding `pull_request` to `on:`

```yaml
name: Actions Workflow
on: [push, pull_request]
```

![Github Actions On Pull Request](github_actions_on_pull_request.png)

* By default, including `pull_request` only runs on these *activity types*: `opened`, `reopened`, and `synchronize`, but **not not on** `closed`.
* To trigger workflows by different activity types, use the `types` keyword:

```yaml
on:
  pull_request:
    types: [review_requested]
jobs:
  specific_review_requested:
    runs-on: ubuntu-latest
    if: ${{ github.event.requested_team.name == 'dev-team'}}
    steps:
      - run: echo 'A review from dev-team was requested'
```

* If you use `actions/checkout`, when code is pushed, the action will run on any branch because a branch name is not specified under `on:`
* * `$GITHUB_REF` is the branch that triggered the action
* OTOH, when a pull_request occurs, then:
* * `$GITHUB_REF` is the PR merge branch: `refs/pull/:prNumber/merge`

![Github Actions On Pull Request 2](github_actions_on_pull_request_2.png)

* See also: [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

## Schedule

You can use a *cron schedule expression* to schedule GitHub Actions.
* See also: [Create a repository dispatch event](https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event)


```yaml
on:
  schedule:
    # run at 12 PM each day
    - cron: "0 12 * * *"
    # at 04:30 on every 2nd day-of-month
    - cron: "30 4 */2 * *"
```

* The **minimum time** is every 5 minutes.
* See also: [Crontab Guru](https://crontab.guru/)

___

## repository_dispatch

This event can be manually triggered by sending a POST request.  A `GitHub Token` is required in order to authorize the POST request.

* Settings -> Developer Settings -> Personal Access Tokens
* Generate new token
* Scope: `repo` checkbox for all 4 sub-repo permissions


The POST request URL is:
* https://api.github.com/repos/USERNAME/YOUR-REPO/dispatches
* Authorization:
* * Basic Auth with newly generated token
* Required Headers:
* * Key: `Accept`
* * Value: `application/vnd.github.v3+json`
* * Key: `Content-Type`
* * Value: `application/json`
* Body:
* * send raw JSON
* Example:

```json
{
  "event_type": "whatever_you_like (such as 'build')"
  "client_payload": {
    "env": "production",
    "example_unit": false,
    "example_integration": true
  }
}
```

actions.yml:
```yml
repository_dispatch:
  # this match "event_type" in the JSON listed above
  types: [build]
```

In The `actions.yml` the `client_payload` will be available in the `github` object:

![Github Actions Repository Dispatch Client Payload](github_actions_repository_dispatch_client_payload.png)

___

## Filtering Workflows by Branches, Tags, and Paths

Only run workflow when pushing to main:

```yml
on:
  push:
    branches:
      - main
```

Only run workflow if a PR is requesting to merge with main:

```yml
on:
  pull_request:
    branches:
      - main
```

Branches can also be pattern based:

```yml
on:
  push:
    branches:
      - main
      - 'feature/*'
```

The above would **not** match `feature/abc/xyz` as the `*` does not match a `/`. It has to be one word.  Otherwise, you will need to use: `- 'feature/**'` to match nested slashes.

See also: [Filter pattern cheat sheet](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet)

**Negation / Exclusions**

Use `branches-ignore` to run on **all** branches except for those listed:

```yml
on:
  push:
    branches-ignore:
      - main
```

You can **not** use both `branches:` and `branches-ignore:` simultaneously. To overcome this limitation, use a `!` within `branches`.

This will ignore the `featC` branch but still work on all other `feature` branches.  The negation entry **must come last** in the list:

```yml
on:
  push:
    branches-ignore:
      - main
      - 'feature/**'
      - '!feature/featC'
```

**tags**

These work exactly like `branches`, including `tags-ignore`:

```yml
on:
  push:
    branches:
      - master
      - 'feature/**'
  tags:
    - v1.*
```

**paths**

For example, only run the workflow when Javascript files have been pushed, except for `filename.js`:

```yml
on:
  push:
    branches:
      - master
      - 'feature/**'
  tags:
    - v1.*
  paths:
    - '**.js'
    - '!filename.js'
```

You can also use `paths-ignore` but not at the same time that you use `paths`:

```yml
   paths-ignore:
     - 'docs/**'
```




___

## Marketplace

[Github Actions Marketplace](https://github.com/marketplace?type=actions)

___

## Debug
* [If the workflow logs do not provide enough detail to diagnose why a workflow, job, or step is not working as expected, you can enable additional debug logging.](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)

![Github Actions Debug Howto](github_actions_debug_howto.png)

![Github Actions Debug Output](github_actions_debug_output.png)

* You can download the `log archive`:
* * If `ACTIONS_RUNNER_DEBUG` and `ACTIONS_STEP_DEBUG` set to `true`, then your downloaded `logs.zip` file will contain an additional folder: `runner-diagnostic-logs`

![Github Actions Debug Download Log Archive](github_actions_debug_download_log_archive.png)
