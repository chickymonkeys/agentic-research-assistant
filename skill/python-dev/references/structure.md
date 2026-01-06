# Python Project Structure

Standard directory layouts and configuration patterns for modern Python projects.

## The `src` Layout

We strongly recommend the `src` layout for all Python projects. It prevents import errors (like importing the local folder instead of the installed package) and forces you to test against the installed version of your code.

### Structure

```text
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── core.py
│       └── utils.py
├── tests/
│   ├── conftest.py
│   └── test_core.py
├── pyproject.toml      # Project config
├── README.md
├── .gitignore
└── uv.lock             # Dependency lockfile
```

## `pyproject.toml` Configuration

The single source of truth for build, dependencies, and tool configuration.

### Project Metadata & Dependencies (PEP 621)

```toml
[project]
name = "mypackage"
version = "0.1.0"
description = "A short description"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "httpx>=0.27.0",
    "pydantic>=2.7.0",
]

[project.scripts]
myapp = "mypackage.cli:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

### Tool Configuration

**Ruff (Linting & Formatting)**

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP"] # Pyflakes, pycodestyle, isort, bugbear, pyupgrade
ignore = []
```

**Pytest**

```toml
[tool.pytest.ini_options]
pythonpath = ["src"]
addopts = "-ra -q"
testpaths = ["tests"]
```

**Mypy (Type Checking)**

```toml
[tool.mypy]
python_version = "3.11"
strict = true
ignore_missing_imports = true
```

## Module Organization

### `__init__.py`

Keep it minimal. Expose the public API.

```python
from .core import process_data
from .utils import helper

__all__ = ["process_data", "helper"]
```

### `cli.py` or `__main__.py`

Entry points for command-line execution.

```python
# src/mypackage/cli.py
import sys

def main():
    print("Running CLI")
    sys.exit(0)

if __name__ == "__main__":
    main()
```
