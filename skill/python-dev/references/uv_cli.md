# uv CLI Reference

Complete command-line interface reference for uv.

## Global Flags

```bash
--help, -h              Display help
--verbose, -v           Verbose logging
--quiet, -q             Suppress output
--no-cache              Disable caching
--offline               Disable network access
--python <VERSION>      Python interpreter/version
```

## Project Commands

### uv init

Create new project or script.

```bash
uv init [OPTIONS] [PATH]

--lib                   Create library (src/ layout)
--app                   Create application (default)
--script                Create PEP 723 script
--python <VERSION>      Python version requirement
```

### uv add

Add dependencies.

```bash
uv add [OPTIONS] <PACKAGES>...

--dev                   Add to dev dependencies
--group <GROUP>         Add to dependency group
--raw-sources           Raw source specifiers
```

### uv remove

Remove dependencies.

```bash
uv remove [OPTIONS] <PACKAGES>...

--dev                   Remove from dev dependencies
```

### uv sync

Synchronize environment with lockfile.

```bash
uv sync [OPTIONS]

--frozen                Use existing lockfile (CI mode)
--all-extras            Include all optional dependencies
```

### uv lock

Update lockfile.

```bash
uv lock [OPTIONS]

--upgrade               Allow package upgrades
```

### uv run

Execute commands in project environment.

```bash
uv run [OPTIONS] <COMMAND> [ARGS]...

--with <PKG>            Include temporary packages
--no-project            Run standalone
```

## Tool Management

### uvx (uv tool run)

Execute tool without persistent installation.

```bash
uvx ruff check
uvx --from <PKG> <CMD>
```

### uv tool install

Install tool persistently.

```bash
uv tool install ruff
```

## Python Management

### uv python install

Install Python versions.

```bash
uv python install 3.12
```

### uv python pin

Pin Python version for project.

```bash
uv python pin 3.12
```

## Pip Interface

`uv pip` commands drop-in replace `pip`.

```bash
uv pip install -r requirements.txt
uv pip compile pyproject.toml -o requirements.txt
```
