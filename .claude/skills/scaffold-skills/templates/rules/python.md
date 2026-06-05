---
applies_to:
  - "**/*.py"
  - "**/pyproject.toml"
  - "**/requirements*.txt"
---

# Python conventions for this project

- **Framework:** {{PYTHON_FRAMEWORK}}

## Layout

```
.
├── pyproject.toml          # PEP 621 metadata; single source of truth
├── src/<package>/          # src layout — package not importable from repo root
│   ├── __init__.py
│   └── ...
└── tests/
    └── test_<thing>.py
```

- `src/` layout is mandatory. It catches "works because the cwd is in `sys.path`" bugs before they ship.
- One package per repo unless the project genuinely ships multiple.

## Types

- Type hints on every public function signature. Internal helpers as needed.
- `from __future__ import annotations` at the top of every module (cheaper + future-proof string annotations).
- `mypy --strict` is the goal. Use `# type: ignore[code]` with the specific error code and a comment when you must.
- Prefer `TypedDict` / `dataclass` / `pydantic.BaseModel` over loose dicts.

## Style

- `ruff format` (Black-compatible) is the formatter. `ruff check --fix` is the linter.
- Imports: stdlib, third-party, first-party — `ruff` (isort) sorts them.
- Naming: `snake_case` functions and variables, `PascalCase` classes, `UPPER_SNAKE` constants.

## Errors

- Raise specific exceptions. `raise ValueError("...")` over `raise Exception("...")`.
- Define a project-level exception base class for domain errors (`class AppError(Exception): ...`).
- Never `except:` bare. `except Exception:` only at top-level entry points (CLI, request handler, job runner).
- Don't swallow exceptions to "be robust." Let them propagate; log at the boundary.

## Concurrency

- Prefer `asyncio` for I/O-bound. `concurrent.futures.ProcessPoolExecutor` for CPU-bound.
- Don't mix sync and async in the same call path. If you must, use a single bridge function and document it.
- Thread-locals and module-level state are session-killers in async code. Pass context explicitly.

<!-- IF:PYTHON_FASTAPI -->
## FastAPI

- Routers per resource (`routers/users.py`), not god-routers.
- Pydantic models for request/response bodies. Don't return ORM objects directly.
- Dependencies (`Depends(...)`) for auth, db sessions, config. Keep handlers thin.
- `async def` handlers when the underlying I/O is async; sync `def` otherwise — don't fake it.
<!-- ENDIF -->
<!-- IF:PYTHON_DJANGO -->
## Django

- Apps split by domain, not by layer. Each app owns its models, views, urls, admin.
- Migrations are source-controlled and never edited after merge. Squash deliberately, not casually.
- Use `select_related` / `prefetch_related` to kill N+1s. Audit query counts in tests via `assertNumQueries`.
- `Django REST Framework` viewsets only when the resource is genuinely CRUD; otherwise plain APIViews.
<!-- ENDIF -->
<!-- IF:PYTHON_FLASK -->
## Flask

- Blueprints per resource. App factory pattern (`create_app()`) — no module-level `app = Flask(...)`.
- Use `flask-marshmallow` or `pydantic` for body validation; never `request.json` raw.
- Extensions initialized in the factory, not at import time.
<!-- ENDIF -->
<!-- IF:PYTHON_DATA_SCIENCE -->
## Data-science conventions

- Notebooks are scratchpads, not deliverables. Promote stable code to `src/<package>/` and import it from notebooks.
- Pin dependencies (`pyproject.toml` or `requirements.txt`) — Conda environments included.
- Random seeds set explicitly at the top of any reproducible script.
- Data lives outside the repo. Use a `data/` directory in `.gitignore` and document where it comes from.
<!-- ENDIF -->

## Tests

- `pytest` only. No `unittest.TestCase` subclasses in new code.
- Fixtures in `conftest.py` at the closest meaningful scope (per-package, not project-wide).
- `pytest-mark` for slow/integration/external tests, runnable independently.
- Parametrize over loops: `@pytest.mark.parametrize` reports each case separately.

## Before declaring done

- [ ] `ruff check`
- [ ] `ruff format --check`
- [ ] `mypy src/` (or `--strict`)
- [ ] `pytest -q`

## Anti-patterns

- Mutable default arguments (`def f(x=[]):`). Use `None` and assign inside.
- `*args, **kwargs` to forward everything. Be explicit about what you accept.
- `import *` anywhere except a deliberate package `__init__.py` re-export.
- Catching `BaseException`. You'll swallow `KeyboardInterrupt` and `SystemExit`.
