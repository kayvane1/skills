---
name: building-modal-apps
description: Scaffold and structure Modal.com Python applications using the "thin wrapper" pattern — thin Modal endpoints around pure-Python business logic, with smoke tests on a dev environment. Use when creating a new Modal service, adding a Modal endpoint, building an LLM or document-processing pipeline on Modal, migrating a workload to Modal, or refactoring a Modal app to separate business logic from infrastructure.
---

# Building Modal Apps

Modal endpoints are **thin wrappers** around **pure-Python business logic**. Endpoints handle infrastructure (images, secrets, timeouts, GPUs, concurrency). Services hold logic and stay testable without Modal.

## Architecture

```
endpoints/<name>.py    Modal wrapper (~50-100 lines): @app.cls/@app.function, @modal.enter, @modal.method
    ↓ instantiates in @modal.enter()
services/<name>.py     Pure Python (500+ lines): all business logic, no modal imports
```

**Why:** unit tests run locally without Modal, business logic is portable (local / Lambda / Modal), infrastructure concerns stay in one file.

## Standard project structure

```
<app_name>/
├── <module>/
│   ├── common.py            # modal.App, images, secrets, mounts
│   ├── deploy.py            # imports endpoint apps, calls app.include()
│   ├── endpoints/           # thin Modal wrappers
│   ├── services/            # pure Python business logic
│   ├── schemas/             # Pydantic models
│   └── db/                  # database code (optional)
├── test/
│   ├── unit/                # tests services/ directly — no Modal
│   ├── integration/
│   └── smoke/               # tests deployed Modal endpoints via .lookup()
├── Makefile
├── pyproject.toml
└── README.md
```

## Workflow (TDD with smoke tests)

1. **Gather requirements** — module name, purpose, dependencies, required secrets, GPU?, S3 mounts?
2. **Create directories + `__init__.py`** for every package
3. **Write `common.py`** — `modal.App(...)`, secrets, CPU/GPU images, S3 mounts
4. **Write service class** in `services/<name>.py` — pure Python, no `modal` imports
5. **Write unit tests** in `test/unit/test_<service>.py` — cover service class directly
6. **Write Modal endpoint** in `endpoints/<name>.py` — `@modal.enter()` instantiates the service; `@modal.method()` async pass-through
7. **Write `deploy.py`** — `from <module>.common import app` and `app.include(<endpoint_app>)`
8. **Write smoke test** in `test/smoke/test_<endpoint>_e2e.py` — `modal.App.lookup(env="dev")` then call `.remote()`
9. **Add `Makefile` + `pyproject.toml`**
10. **Run unit tests locally** — must pass before deploying
11. **Deploy to dev** — `modal deploy <module>/deploy.py --env=dev`
12. **Run smoke tests** — verify they pass against the deployed env

See [REFERENCE.md](REFERENCE.md) for full code templates and antipatterns.

## Non-negotiables

- **Methods on Modal classes must be `async`** — `@modal.method()` doesn't drive sync functions reliably.
- **No business logic in the endpoint file.** If `endpoints/foo.py` exceeds ~100 lines, split out a service.
- **Never hardcode secrets.** Use `modal.Secret.from_name(...)`; `modal secret list` to discover what exists.
- **Images:** `.add_local_python_source()` for app code (fast iteration), `.add_local_dir(..., copy=True)` only for stable shared libs. Using `copy=True` for app code kills iteration speed.
- **Smoke tests must use `modal.App.lookup()`** to hit the deployed app — never call the local code.
- **Pin package versions** in `pyproject.toml` and in image `pip_install` calls.

## When the user provides a partial app

If asked to add an endpoint to an existing Modal project:

1. Read `common.py` to learn the existing app name, image, and secrets.
2. Add the new service in `services/` first, with unit tests.
3. Add the thin endpoint wrapper, importing existing images/secrets from `common.py` rather than redefining.
4. Register it in `deploy.py` via `app.include()`.
5. Add a smoke test that uses the same `App.lookup()` pattern as siblings.
