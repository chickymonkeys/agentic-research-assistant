---
name: python-development
description: Expert guidance for modern Python development. Use for initializing projects, managing dependencies (uv), structuring code (src-layout), enforcing style (ruff/mypy), and writing tests (pytest). Covers the full development lifecycle from setup to CI/CD.
---

# Python Development

This skill provides a comprehensive workflow for building, testing, and maintaining Python projects using modern tooling standards (2025+).

## Core Principles

1.  **Modern Tooling**: Use **uv** for everything (projects, packages, python versions).
2.  **Standard Layout**: Always use the **`src` layout** for robust packaging.
3.  **Strict Quality**: Enforce types (`mypy`) and style (`ruff`) from day one.
4.  **Reproducibility**: Lock dependencies (`uv.lock`) and pin Python versions.

## 1. Project Setup & Workflow (uv)

We use **uv** as the single tool for Python development.

**See [references/uv.md](references/uv.md) for detailed commands.**

### Quick Start

```bash
# Initialize application
uv init myapp --app --python 3.12
cd myapp

# Add dependencies
uv add fastapi pydantic
uv add --dev pytest ruff mypy

# Run
uv run main.py
```

## 2. Project Structure

Adhere to the `src` layout to avoid import side-effects and ensure proper packaging.

**See [references/structure.md](references/structure.md) for full layout and `pyproject.toml` templates.**

```text
myproject/
├── src/
│   └── mypackage/      # Source code here
├── tests/              # Tests outside src
├── pyproject.toml      # Config center (deps, tools)
└── uv.lock             # Lockfile
```

## 3. Code Quality & Style

Use **Ruff** for linting/formatting and **Mypy** for type checking.

**See [references/style.md](references/style.md) for the detailed Style Guide.**
**See [references/advanced_types.md](references/advanced_types.md) for Type Hinting patterns.**

### Command Checklist

```bash
# Format code
uv run ruff format .

# Lint code (fix auto-fixable issues)
uv run ruff check --fix .

# Type check
uv run mypy src
```

## 4. Testing

Use **pytest** for all testing.

**See [references/testing.md](references/testing.md) for best practices and fixtures.**

### Core Pattern

```python
# tests/test_core.py
from mypackage.core import add

def test_add_positive():
    assert add(1, 2) == 3
```

Run tests: `uv run pytest`

## Additional Resources

-   **Workflow**: [references/uv.md](references/uv.md) - Deep dive into `uv` CLI.
-   **Style**: [references/style.md](references/style.md) - Google-style docstrings, naming, patterns.
-   **Structure**: [references/structure.md](references/structure.md) - Project layouts and config.
-   **Testing**: [references/testing.md](references/testing.md) - Pytest patterns.
-   **Types**: [references/advanced_types.md](references/advanced_types.md) - Generics, Protocols, TypedDict.
-   **Anti-patterns**: [references/antipatterns.md](references/antipatterns.md) - Common mistakes to avoid.
