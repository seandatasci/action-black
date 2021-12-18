# Action-black GitHub Action

[![Test](https://github.com/rickstaa/action-black/workflows/Test/badge.svg)](https://github.com/rickstaa/action-black/actions?query=workflow%3ATest)
[![release](https://github.com/rickstaa/action-black/workflows/release/badge.svg)](https://github.com/rickstaa/action-black/actions?query=workflow%3Arelease)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/rickstaa/action-black?logo=github\&sort=semver)](https://github.com/rickstaa/action-black/releases)
[![action-bumpr supported](https://img.shields.io/badge/bumpr-supported-ff69b4?logo=github\&link=https://github.com/haya14busa/action-bumpr)](https://github.com/haya14busa/action-bumpr)

This action runs the [black formatter](https://github.com/psf/black) or [jupyter-black formatter](https://github.com/n8henrie/jupyter-black) to check/format your python code and jupyter notebooks on a push or pull request. It is similar to [reviewdog/action-black](https://github.com/reviewdog/action-black) that can annotate the black changes required with [Reviewdog](https://github.com/reviewdog/reviewdog). However, this version also allows you to format the code using GitHub actions (see #advanced-use-cases).

## Quickstart

In it's simplest form this action can be used to check/format your code using the black formatter.

```yaml
name: action-black-py-jupyter
on: [push, pull_request]
jobs:
  linter_name:
    name: black formatter
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: seandatasci/action-black-py-jupyter@v2.0
        with:
          black_args: ". --check"
```

## Inputs

### `black_args`

**optional**: Black input arguments. Defaults to `. --check --diff`.

### `fail_on_error`

**optional**: Exit code when black formatting errors are found \[true, false]. Defaults to 'true'.

## Outputs

### `is_formatted`

Boolean specifying whether any files were formatted using the black formatter.

## Advanced use cases

### Annotate changes

This action can be combined with [reviewdog/action-suggester](https://github.com/reviewdog/action-suggester) also to annotate any possible changes (uses `git diff`).

```yaml
name: action-black-py-jupyter
on: [push, pull_request]
jobs:
  linter_name:
    name: black formatter
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check files using the black formatter
        uses: seandatasci/action-black-py-jupyter@v2.0
        id: action_black_py_jupyter
        with:
          black_args: "."
      - name: Annotate diff changes using reviewdog
        if: steps.action_black_py_jupyter.outputs.is_formatted == 'true'
        uses: reviewdog/action-suggester@v1
        with:
          tool_name: blackfmt
```

### Commit changes or create a pull request

This action can be combined with [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request) to also apply the annotated changes to the repository.

```yaml
name: action-black-py-jupyter
on: [push, pull_request]
jobs:
  linter_name:
    name: runner / black
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check files using the black formatter
        uses: seandatasci/action-black-py-jupyter@v2.0
        id: action_black_py_jupyter
        with:
          black_args: "."
          fail_on_error: "false"
      - name: Create Pull Request
        if: steps.action_black_py_jupyter.outputs.is_formatted == 'true'
        uses: peter-evans/create-pull-request@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          title: "Format Python code with psf/black push"
          commit-message: ":art: Format Python code with psf/black"
          body: |
            There appear to be some python formatting errors in ${{ github.sha }}. This pull request
            uses the [psf/black](https://github.com/psf/black) formatter to fix these issues.
          base: ${{ github.head_ref }} # Creates pull request onto pull request or commit branch
          branch: actions/black
```
