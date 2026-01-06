# Environment Management with renv

`renv` provides project-local R dependency management, ensuring reproducibility by isolating project libraries.

## Core Workflow

| Command | Description |
|---------|-------------|
| `renv::init()` | Initialize a new project-local environment. |
| `renv::snapshot()` | Save the current state of libraries to `renv.lock`. |
| `renv::restore()` | Install packages defined in `renv.lock`. |
| `renv::status()` | Check for inconsistencies between library and lockfile. |
| `renv::update()` | Update packages. |

### Initialization

```r
# Initialize project
renv::init()
```

### Dependency Management

1.  **Install** packages normally (`install.packages` or `pacman::p_load`).
2.  **Snapshot** to save state.

```r
install.packages("dplyr")
renv::snapshot()  # updates renv.lock
```

3.  **Restore** on a new machine or after cloning.

```r
renv::restore()   # installs exact versions from lockfile
```

## Best Practices

### Git Configuration
**Commit** these files:
- `renv.lock` (The source of truth)
- `.Rprofile` (Activates renv on startup)
- `renv/activate.R` (Activation script)
- `renv/settings.json` (Project settings)

**Ignore** these files (`.gitignore`):
- `renv/library/` (Local library)
- `renv/python/` (If using reticulate)
- `renv/staging/`

### Static Analysis & `pacman` Integration

`renv` uses static analysis to detect dependencies. It scans `.R`, `.Rmd`, and `.qmd` files for `library()`, `require()`, and `::` usage.

**Caveat**: `renv` **does not** automatically detect packages loaded via `pacman::p_load()` because it is non-standard evaluation.

**Workaround**:
If you use `pacman::p_load(dplyr, data.table)`, you must explicitly explicitly signal dependencies to `renv` in a dummy file (e.g., `_dependencies.R`) or ensure at least one `library(pkg)` call exists in the project.

```r
# _dependencies.R
# Explicitly list packages for renv detection if using dynamic loading
library(dplyr)
library(data.table)
library(rio)
```

Alternatively, configure `renv` to scan strictly:
```r
renv::settings$snapshot.type("explicit") # Only snapshot what is explicitly listed in DESCRIPTION
```

## CI/CD Integration (GitHub Actions)

Use the `setup-renv` action to restore environments efficiently with caching.

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: r-lib/actions/setup-r@v2
  - uses: r-lib/actions/setup-renv@v2
    with:
      cache-version: 1
```

This action automatically:
1. Installs `renv`.
2. Restores packages from `renv.lock`.
3. Caches the library for future runs.

## Troubleshooting

- **Desynchronized**: If `renv::status()` reports issues, run `renv::restore()` to revert to the lockfile state or `renv::snapshot()` to update the lockfile to the current state.
- **System Libraries**: By default, `renv` isolates the project. To use system packages, configure `renv::settings$external.libraries()`.
