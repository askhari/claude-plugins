---
name: python-package
description: Use when creating a new Python package from scratch or restructuring one to match this project's conventions — src layout, PEP 621 metadata, ruff + mypy + pytest configured, and the project's framework defaults applied where relevant.
context: fork
---

# Create a Python package

This skill walks through creating a new package that matches the conventions defined in `.claude/rules/python.md`.

## Step 1 — Clarify scope

Before writing anything, confirm with the user:

1. What does this package do? (one cohesive responsibility, not a monorepo)
2. Is it a library (importable) or an application (entry point)?
3. Required Python version? (`>=3.11` is the modern floor unless told otherwise.)
4. Any optional dependency groups expected? (e.g., `[dev]`, `[test]`, `[docs]`)

## Step 2 — Scaffold the layout

```bash
mkdir -p src/<package_name> tests
cd <project-root>
touch pyproject.toml README.md src/<package_name>/__init__.py tests/__init__.py tests/test_smoke.py
```

The `src/` layout is mandatory — it forces an installed-package import path, catching "works because cwd is on `sys.path`" bugs.

## Step 3 — Write `pyproject.toml`

PEP 621 metadata is the single source of truth.

```toml
[project]
name = "<package-name>"
version = "0.1.0"
description = "<one-line description>"
requires-python = ">=3.11"
dependencies = []

[project.optional-dependencies]
dev = ["ruff", "mypy", "pytest", "pytest-cov"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "N", "SIM", "RUF"]

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-q --strict-markers"
```

<!-- IF:PYTHON_FASTAPI -->
Add FastAPI dependencies to `[project.dependencies]`: `fastapi`, `uvicorn[standard]`. Add an app entry point at `src/<package_name>/main.py`.
<!-- ENDIF -->
<!-- IF:PYTHON_DJANGO -->
Add Django dependencies. Use `django-admin startproject` inside `src/<package_name>/` rather than scaffolding by hand — the generated files match Django's expectations exactly.
<!-- ENDIF -->
<!-- IF:PYTHON_FLASK -->
Add Flask dependencies and an app factory at `src/<package_name>/app.py` exposing `create_app()`. No module-level `app = Flask(...)`.
<!-- ENDIF -->

## Step 4 — Write `src/<package_name>/__init__.py`

Set the version and (optionally) re-export the small public surface:

```python
"""<one-line description>."""
from __future__ import annotations

__version__ = "0.1.0"
```

Don't re-export everything. Public API is a deliberate choice.

## Step 5 — Write a smoke test

`tests/test_smoke.py` imports the package and asserts something trivial. The point is to catch packaging errors (missing files, broken `pyproject.toml`) on day one.

```python
from __future__ import annotations

import <package_name>


def test_version_is_set() -> None:
    assert <package_name>.__version__
```

## Step 6 — Install in editable mode

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

If `pip install -e .` fails, the `pyproject.toml` is wrong — fix it before continuing.

## Step 7 — Validate

```bash
ruff check
ruff format --check
mypy src/
pytest -q
```

All four must pass before the scaffold is "done."

## Step 8 — Hand back to the user

Summarize what was created, the public API (currently `__version__`), and the optional dependency groups. Note any commands the user should run next (`pre-commit install`, `pip install -e ".[docs]"`, etc.).
