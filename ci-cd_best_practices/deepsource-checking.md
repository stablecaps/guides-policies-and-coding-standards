# Deep Source checking.


### Setup

1. Add a yaml called `.deepsource.toml` in root of repo to configure automatic PR review.
2. go to [deepsource](https://app.deepsource.com/) and activate repo for scanning of issues. This should show up in PRs.


```yaml
version = 1

exclude_patterns = [
  "docs*/**",
  "venv/*",
  "__pycache__/*",
  "templates/*",
]

test_patterns = [
  "tests/**/*.py",
  "tests/*.py",
  "test_*.py"
]

[[analyzers]]
name = "test-coverage"
enabled = true

[[analyzers]]
name = "shell"
enabled = true

[[analyzers]]
name = "python"
enabled = true

  [analyzers.meta]
  runtime_version = "3.x.x"

[[transformers]]
name = "black"
enabled = true

[[transformers]]
name = "isort"
enabled = true
```