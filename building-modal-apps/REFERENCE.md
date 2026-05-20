# Modal Apps — Reference Templates

Templates use `<module>`, `<service>`, `<endpoint>`, `<app-name>` as placeholders.
Replace with concrete names. Pin all versions.

## common.py — Modal app, images, secrets

```python
"""Modal app configuration: app, images, secrets, mounts."""

import modal

APP_NAME = "<app-name>"

app = modal.App(
    APP_NAME,
    secrets=[
        modal.Secret.from_name("<db-secret>"),
        modal.Secret.from_name("<llm-api-secret>"),
    ],
)

# CPU image — orchestration, lightweight processing
cpu_image = (
    modal.Image.debian_slim(python_version="3.13")
    .pip_install(
        "pydantic>=2.12.0",
        "openai>=1.0.0",
        # pin everything
    )
    # stable shared libs use copy=True for cache
    .add_local_dir("../../libs/<shared>", "/pkg/<shared>", copy=True)
    .env({"PYTHONPATH": "/pkg"})
    # app code last — no copy, so edits don't bust the image cache
    .add_local_python_source("<module>")
)

# GPU image (only if needed)
gpu_image = (
    modal.Image.debian_slim(python_version="3.13")
    .pip_install("torch>=2.5.0", "transformers>=4.46.0")
    .add_local_python_source("<module>")
)
```

### CloudBucketMount for S3 (OIDC, no credentials)

```python
S3_BUCKET_NAME = "<bucket>"
S3_MOUNT_PATH = "/mnt/s3"
ROLE_ARN = "arn:aws:iam::<account>:role/<role>"

s3_mount = modal.CloudBucketMount(
    bucket_name=S3_BUCKET_NAME,
    oidc_auth_role_arn=ROLE_ARN,
)
```

Pass to a function/class via `cloud_bucket_mounts={S3_MOUNT_PATH: s3_mount}`.

## services/<service>.py — pure-Python business logic

```python
"""Business logic for <functionality>. No Modal imports — unit testable."""

from typing import Any


class DataProcessor:
    def __init__(self, config: dict | None = None):
        self.config = config or {}

    async def process(self, data: dict) -> dict[str, Any]:
        if not data:
            raise ValueError("Data cannot be empty")
        return {"status": "success", "result": self._do_processing(data)}

    def _do_processing(self, data: dict) -> dict:
        return data
```

## endpoints/<endpoint>.py — thin Modal wrapper

```python
"""Modal endpoint — thin wrapper around DataProcessor."""

import modal
from <module>.common import app, cpu_image

endpoint_app = modal.App("<app-name>-<endpoint>")


@endpoint_app.cls(image=cpu_image, timeout=600)
@modal.concurrent(max_inputs=128)
class DataProcessorEndpoint:
    @modal.enter()
    def initialize(self):
        from <module>.services.<service> import DataProcessor
        self.processor = DataProcessor(config={"option": "value"})

    @modal.method()
    async def process(self, data: dict) -> dict:  # MUST be async
        return await self.processor.process(data)
```

### GPU endpoint variant

```python
@endpoint_app.cls(
    image=gpu_image,
    gpu="A100",
    timeout=600,
    scaledown_window=15 * 60,
    max_containers=5,
    volumes={"/models": model_volume},
)
@modal.concurrent(max_inputs=32)
class ModelServerEndpoint:
    @modal.enter()
    def initialize(self):
        from <module>.services.model_server import ModelServer
        self.server = ModelServer(model_path="/models/model.pt")

    @modal.method()
    async def generate(self, prompt: str) -> str:
        return await self.server.generate(prompt)
```

## deploy.py — deployment entry point

```python
from <module>.common import app
from <module>.endpoints.<endpoint> import endpoint_app

app.include(endpoint_app)
```

Add one `app.include(...)` per endpoint app.

## test/unit/test_<service>.py — fast, no Modal

```python
import pytest
from <module>.services.<service> import DataProcessor


@pytest.mark.asyncio
async def test_processor_success():
    result = await DataProcessor().process({"key": "value"})
    assert result["status"] == "success"
    assert result["result"]["key"] == "value"


@pytest.mark.asyncio
async def test_processor_rejects_empty():
    with pytest.raises(ValueError, match="Data cannot be empty"):
        await DataProcessor().process({})
```

## test/smoke/test_<endpoint>_e2e.py — hits deployed Modal

```python
import modal
import pytest


@pytest.fixture
def modal_app():
    return modal.App.lookup("<app-name>-<endpoint>", environment_name="dev")


def test_endpoint_deployed(modal_app):
    endpoint = modal_app.registered_entrypoints["DataProcessorEndpoint"]
    result = endpoint.process.remote({"test": "data"})
    assert result["status"] == "success"
```

For function-style endpoints use `modal_app.registered_functions["<name>"].remote(...)`.

## Makefile

```makefile
.PHONY: help install test lint format deploy smoke clean

help:
	@echo "install | test | lint | format | deploy | smoke | clean"

install:
	uv pip install -e .

test:
	uv run pytest test/unit -v

lint:
	uv run ruff check .

format:
	uv run ruff format .

deploy:
	uv run modal deploy <module>/deploy.py --env=dev

smoke:
	uv run pytest test/smoke -v

clean:
	rm -rf .pytest_cache **/__pycache__
```

## pyproject.toml

```toml
[project]
name = "<module>"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    "pydantic>=2.12.0",
    "openai>=1.0.0",
    "modal>=1.1.4",
]

[tool.uv]
dev-dependencies = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.23.0",
    "ruff>=0.1.9",
]

[tool.ruff]
line-length = 100
target-version = "py313"

[tool.pytest.ini_options]
testpaths = ["test"]
asyncio_mode = "auto"
```

## Antipatterns

| Wrong | Right | Why |
|---|---|---|
| `.add_local_python_source("<module>", copy=True)` | `.add_local_python_source("<module>")` | `copy=True` for app code busts cache on every edit |
| `api_key = "sk-..."` inline | `modal.Secret.from_name("<name>")` | Secrets must never live in source |
| `def process(self, data)` on `@modal.method()` | `async def process(self, data)` | `@modal.method()` requires async |
| Business logic inside the endpoint class | Service class in `services/`, endpoint delegates | Unit tests then need Modal — slow, fragile |
| `modal deploy ...` without smoke test | `modal deploy ... && pytest test/smoke -v` | Catches image/secret/network failures unit tests miss |
| Smoke test instantiates the local class | `modal.App.lookup(env="dev")` then `.remote()` | Local instantiation skips the actual deployment |
| Re-defining `app` / `image` / `secrets` in each endpoint | Import them from `common.py` | One source of truth |
| Endpoint file > 100 lines | Move logic to `services/`, keep wrapper thin | Wrapper exists for infra, not logic |

## What a smoke test catches that unit tests miss

- Image dependencies (missing packages, wrong `PYTHONPATH`)
- Secret names / keys
- Resource allocation (OOM, timeouts, GPU type)
- Networking (VPC, API egress, private DNS)
- Modal-specific decorator wiring (`@modal.enter`, `@modal.method`, `@modal.concurrent`)
