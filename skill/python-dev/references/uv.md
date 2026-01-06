# Python Development with uv

**uv** is Astral's extremely fast Python package and project manager written in Rust. It replaces `pip`, `pip-tools`, `poetry`, `pyenv`, and `virtualenv`.

## Core Commands

### Project Initialization

```bash
# Application (default)
uv init myapp --app

# Library (src/ layout)
uv init mylib --lib

# Script (PEP 723)
uv init --script analyze.py
```

### Dependency Management

```bash
# Add dependencies
uv add requests
uv add --dev pytest ruff

# Sync environment
uv sync
uv sync --frozen  # CI/CD mode
```

### Running Code

```bash
# Run script
uv run main.py

# Run tool
uvx ruff check
```

## Python Management

uv automatically manages Python versions.

```bash
# Install version
uv python install 3.12

# Pin version
uv python pin 3.11
```

## Migration

### From pip

```bash
uv init --bare
uv add -r requirements.txt
```

### From Poetry

```bash
uvx migrate-to-uv
```

## CI/CD Integration

Use `uv sync --frozen` in CI to ensure lockfile consistency.

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@v6
- run: uv sync --frozen
```

For detailed reference, see `references/uv_cli.md`.
