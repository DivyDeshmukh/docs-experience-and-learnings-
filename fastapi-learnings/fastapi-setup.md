# FastAPI Service Setup – TestSmith AI (`ai-service`)

These are the **conventions, tools, and structure** we’ll follow for the `ai-service` (FastAPI) part of **TestSmith AI**.

The goals:

- Clean, modular structure
- Strong typing and validation
- Good tooling (lint, format, types, tests)
- Production-friendly from day one

---

## 1. Stack Overview

### 1.1 Runtime / Core Libraries

- **FastAPI**
  - Web framework for building APIs in Python.
  - Async-first, high performance.
  - Auto-generates OpenAPI docs and `/docs` UI from type hints.
  - Great match for strongly-typed, schema-heavy APIs (LLM tools, test generation, etc.).

- **Uvicorn** (`uvicorn[standard]`)
  - ASGI server that actually runs the FastAPI app.
  - Recommended by FastAPI.
  - Handles HTTP & WebSocket, async I/O.

- **Pydantic Settings** (`pydantic-settings`)
  - Strongly-typed app configuration via environment variables / `.env`.
  - Central place for things like:
    - `OPENAI_API_KEY`
    - `ENVIRONMENT`
    - DB URLs (later)
  - Fails fast if required config is missing.

- **httpx**
  - Async HTTP client.
  - Used for calling external APIs (e.g., LLM providers) from FastAPI routes or services.
  - Fits naturally with `async def` functions.

---

## 2. Dev / Quality Tooling

### 2.1 Ruff – Linting + Formatting

- Acts as:
  - Linter (like Flake8),
  - Import sorter (like isort),
  - Formatter (similar to Black, via `ruff format`).
- Why:
  - Keeps the codebase **consistent and clean** automatically.
  - Catches obvious mistakes (unused imports, unused vars, some bug patterns).
  - Very fast → no excuse not to run it.

**Commands (conceptual):**

- Lint: `ruff check app tests`
- Format: `ruff format app tests`

---

### 2.2 mypy – Static Type Checking

- Reads Python type hints and checks that they are used correctly.
- Why:
  - Catches type-related bugs **before** running code.
  - Works well with FastAPI + Pydantic, which are already type-heavy.
  - Important as the AI service grows and request/response structures become complex.

**Command:**

- `mypy app`

---

### 2.3 pytest (+ pytest-asyncio)

- **pytest**
  - Standard testing framework for Python.
  - Simple `test_*.py` + `assert` pattern.
  - Supports fixtures, parametrization, plugins.

- **pytest-asyncio**
  - Adds support for `async def` tests.
  - Lets you test async functions (FastAPI routes, async services) cleanly.

**Example async test:**

```python
import pytest

@pytest.mark.asyncio
async def test_generate_tests():
    # call async function here, e.g.:
    # result = await generate_tests(...)
    # assert result is not None
    ...
````

**Command:**

* `pytest`

---

### 2.4 Pre-commit Hooks (Optional but Recommended)

* Use [`pre-commit`](https://pre-commit.com/) to run Ruff, mypy, etc. before each commit.
* Why:

  * Ensures bad/unformatted code never gets into the repo.
  * Automates “best practices” instead of relying on memory.

Typical setup steps:

```bash
pip install pre-commit
pre-commit install
```

Then configure hooks to run:

* `ruff` / `ruff-format`
* `mypy`
* (optionally) tests or security tools (like `bandit`)

---

## 3. Project Structure (`ai-service`)

Inside the monorepo:

```bash
apps/
  ai-service/
    app/
      api/        # FastAPI routers per domain (test generation, flakiness, etc.)
      core/       # config, startup, shared infra
      models/     # ORM models (if/when we add a DB)
      schemas/    # Pydantic models (request/response DTOs)
      services/   # business logic (LLM calls, analysis logic)
      main.py     # FastAPI application entrypoint
    tests/
      # unit + integration tests
```

---

## 4. Directory Roles

### `app/main.py`

* Creates the `FastAPI` app.
* Includes router registration (later when we add routers).
* Exposes a basic `/health` endpoint.

### `app/core/config.py`

* Defines a `Settings` class using `BaseSettings` (from `pydantic-settings`).
* Handles env vars and `.env` loading.
* Single source of truth for config (environment, API keys, etc.).

### `app/api/`

* Contains domain-specific routers, for example:

  * `test_generation.py`
  * `flakiness.py`
  * (later) `auth.py`, etc.
* The **HTTP layer** (request/response, endpoints) lives here.
* Each router focuses on a specific domain.

### `app/schemas/`

* Pydantic models for:

  * Request bodies
  * Response payloads
  * Internal DTOs if needed
* Keeps schemas:

  * **Reusable**
  * **Explicit**
  * Clearly separate from DB models and business logic.

### `app/services/`

* Business logic, separate from HTTP layer:

  * LLM clients/wrappers (e.g., calling OpenAI or other providers).
  * Flakiness analysis algorithms.
  * Any orchestration logic that combines multiple operations.
* Makes it easy to:

  * Unit-test logic without going through HTTP.
  * Keep routes thin and clean (controllers just call services).

### `app/models/`

* For ORM models if we add a DB later (e.g., SQLAlchemy or another ORM).
* Clean separation between:

  * DB models (`models/`)
  * API schemas (`schemas/`)
* Helps avoid mixing persistence concerns with API shapes.

---

## 5. Minimal Config & App Example

### 5.1 `app/core/config.py`

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    app_name: str = "TestSmith AI Service"
    environment: str = "local"
    openai_api_key: str | None = None  # set via env var

    class Config:
        env_file = ".env"


settings = Settings()
```

### 5.2 `app/main.py`

```python
from fastapi import FastAPI
from .core.config import settings

app = FastAPI(title=settings.app_name)


@app.get("/health", tags=["health"])
async def health_check() -> dict:
    return {"status": "ok", "env": settings.environment}
```

### 5.3 Running the Service Locally

```bash
uvicorn app.main:app --reload --port 8000
```

Then open:

* `http://localhost:8000/health` – basic health check
* `http://localhost:8000/docs` – auto-generated Swagger UI

---

## 6. Tooling Config (pyproject + Makefile)

### 6.1 `pyproject.toml` (example)

```toml
[project]
name = "testsmith-ai-service"
version = "0.1.0"
description = "AI service for TestSmith AI (test generation & flakiness analysis)"
requires-python = ">=3.11"
dependencies = [
  "fastapi",
  "uvicorn[standard]",
  "pydantic-settings",
  "httpx",
]

[project.optional-dependencies]
dev = [
  "ruff",
  "mypy",
  "pytest",
  "pytest-asyncio",
]

[tool.ruff]
line-length = 100
target-version = "py311"
exclude = ["tests"]

[tool.ruff.lint]
select = ["E", "F", "I", "B"]
ignore = []

[tool.mypy]
python_version = "3.11"
strict = false
disallow_untyped_defs = true
warn_unused_ignores = true
warn_return_any = true
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

> You can install dev dependencies with:
>
> ```bash
> pip install -e ".[dev]"
> ```

---

### 6.2 `Makefile` (optional but convenient)

```makefile
.PHONY: lint format typecheck test run

lint:
	ruff check app tests

format:
	ruff format app tests

typecheck:
	mypy app

test:
	pytest

run:
	uvicorn app.main:app --reload --port 8000
```

Usage:

```bash
make format
make lint
make typecheck
make test
make run
```

## 7. Mental Model Summary

* **FastAPI + Uvicorn** → modern async API framework + server.
* **Pydantic Settings** → typed, env-driven configuration (no hardcoded secrets).
* **httpx** → async HTTP for calling LLM / external APIs.
* **Ruff** → fast linter + formatter; enforces clean, consistent code.
* **mypy** → static type checking; catches bugs before runtime.
* **pytest (+ pytest-asyncio)** → testing layer for both sync and async code.
* **Project structure** (`api/`, `schemas/`, `services/`, `core/`, `models/`) → clean separation of:

  * HTTP layer
  * Data schemas
  * Business logic
  * Configuration
  * Persistence

This gives **TestSmith AI’s `ai-service`** a **senior-level baseline** from day one:

* Easy to extend
* Easy to test
* Easy to keep clean as the project grows
