---
description: "Python project git hygiene: .gitignore, what to commit, pre-commit hooks"
applyTo: "**"
---

# Python Git Practices

## `.gitignore` Minimum Entries

Every Python project must include at minimum:

```gitignore
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.venv/
.env
*.log
.mypy_cache/
.pytest_cache/
.ruff_cache/
```

Never commit `.env` files or anything containing secrets. Use a secrets manager (e.g. Vault) or
`python-dotenv` with `.env` in `.gitignore`.

## What Not to Commit

- **Generated artefacts** — compiled outputs, generated docs, coverage reports
- **Notebook outputs** — use `nbstripout` as a pre-commit hook to strip cell outputs automatically
- **Large data files** — data belongs in object storage; only schema and config are versioned

## What to Commit

- **Lock files** (`requirements.txt` pinned, `poetry.lock`, `uv.lock`) — these are reproducibility
  artefacts, not disposable generated files. Always commit them.

## Pre-commit Hooks

Enforce formatting and linting before commits land to keep history clean and remove style noise
from diffs. Use [`prek`](https://prek.j178.dev/) — a fast, dependency-free drop-in replacement
for `pre-commit`, with native `uv` integration. Install via `prek install` to set up the git
hook.

`prek` reads `.pre-commit-config.yaml` (fully compatible with standard pre-commit hooks) or
`prek.toml` (native format). Example `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    rev: 0.9.18
    hooks:
      - id: uv-lock

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.14.9
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: debug-statements
      - id: check-added-large-files
      - id: check-toml
      - id: check-json
      - id: detect-private-key
```

The `debug-statements` hook catches leftover `breakpoint()` and `pdb` calls — particularly
important for DAG and `KubernetesPodOperator` entrypoint code where a stray debugger call causes
a silent runtime failure.
