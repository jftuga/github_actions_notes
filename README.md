# Github Actions Notes
My personal notes about Github Actions

## Run
* Under Linux, the run command does not need to start with `#!/bin/bash` as this is implied.
* By default, `run` beings with `/usr/bin/bash -e {0}`.
* * The `-e` is similar to `set -e` - meaning the script will abort when any commands return a non-zero exit code.
* * `{0}` refers to an auto-generated shell script such as:
* * *  `/home/runner/work/_temp/a894e763-fea0-48ac-ba15-92e8eda24f4d.sh`
* * * The contents of the `run` action have been saved to this file by Github Actions.

## Action Shell
* A [list of all Github Action shells](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell), including supported platforms

* You can use a different shell for the `run` action:

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

## Debug
* [If the workflow logs do not provide enough detail to diagnose why a workflow, job, or step is not working as expected, you can enable additional debug logging.](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)

![Github Actions Debug Howto](github_actions_debug_howto.png)

![Github Actions Debug Output](github_actions_debug_output.png)

* You can download the `log archive`:
* * If `ACTIONS_RUNNER_DEBUG` and `ACTIONS_STEP_DEBUG` set to `true`, then your downloaded `logs.zip` file will contain an additional folder: `runner-diagnostic-logs`

![Github Actions Debug Download Log Archive](github_actions_debug_download_log_archive.png)
