# Setting up Pre-commit hooks

## Overview

To help ensure that the codebase is clean and without error, pre-commit hooks should be used to quality-check committed code. See:
1. [More information](https://pre-commit.com)
2. [more hooks](https://pre-commit.com/hooks.html)


### Example Python pre-commit

1. Add a `.pre-commit-config.yaml` file to the root of the repo
2. Run `poetry add --group=dev pre-commit`
3. Run `pre-commit install`


```yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
    -   id: check-merge-conflict
    -   id: check-ast

- repo: local
  hooks:
    - id: pytest-check
      name: pytest-check
      entry: pytest
      language: system
      pass_filenames: false
      always_run: true

```