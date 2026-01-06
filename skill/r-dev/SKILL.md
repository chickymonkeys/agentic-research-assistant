---
name: r-development
description: Comprehensive guide for modern R development (Tidyverse, data.table, Spatial). Use for writing R code, creating packages, or optimizing performance. Focuses on vectorization, modern dplyr (1.1+), and strict project management.
---

# R Development

This skill provides comprehensive guidance for modern R development, emphasizing current best practices with tidyverse, performance optimization, and professional package development.

## Core Principles

1. **Prioritize data.table for performance** - Use `data.table` for large datasets (>1GB) and performance-critical paths
2. **Use modern tidyverse patterns** - For analysis, readability, and smaller data (dplyr 1.1+)
3. **Profile before optimizing** - Use profvis and bench to identify real bottlenecks
4. **Write readable code first** - Optimize only when necessary and after profiling
5. **Follow style guides** - Consistent naming, spacing, and structure. **Use comments (`# Header ----`) and 80-char limit.**

## Project Management & Best Practices

### Code Style & Documentation
- **Line Length**: Limit lines to **80 characters** for readability.
- **Section Headers**: Use `# Header ----` to organize scripts.
- **Functions**: Document with `roxygen2` comments (`#' @param`).
- **See [references/documentation.md](references/documentation.md) for full style guide.**

### Environment Management (`renv`)
Use `renv` to manage project-local dependencies and ensure reproducibility.
**See [references/renv.md](references/renv.md) for workflow details.**

```r
# Initialize/Activate
renv::init()

# Save state
renv::snapshot()

# Restore state
renv::restore()
```

### Data Import/Export (`rio`)
Use `rio` for format-agnostic import/export. It handles file extensions automatically.

```r
library(rio)

# Import (auto-detects .csv, .xlsx, .json, etc.)
data <- import("data/raw/dataset.xlsx")

# Export
export(data, "data/processed/clean_data.csv")
export(list(Sheet1 = df1, Sheet2 = df2), "data/processed/multisheet.xlsx")
```

### Path Management (`here`)
Use `here` for robust, project-relative paths. **Never** use `setwd()`.

```r
library(here)

# Robust path construction
raw_path <- here("data", "raw", "input.csv")
```

### String Interpolation (`glue`)
Use `glue` for readable string construction over `paste()`/`sprintf()`.

```r
library(glue)

date <- Sys.Date()
filename <- glue("report_{date}.pdf")
path <- here("outputs", filename)
```

### Dependency Management (`pacman`)
In **scripts**, use `pacman::p_load()` to load (and install if missing) packages.
In **packages**, list dependencies in `DESCRIPTION` and use `library()` or `::`.

```r
# Analysis Script
if (!require("pacman")) install.packages("pacman")
pacman::p_load(
  rio,        # I/O
  here,       # Paths
  glue,       # Strings
  janitor,    # Cleaning
  tidyverse   # Core tools
)
```

### Namespace Usage
- **Scripts**: Load packages at top with `library()` or `p_load()`.
- **Functions**: Use explicit namespacing `pkg::fun()` to avoid conflicts and clarify dependencies.

```r
# Inside a function: be explicit
clean_data <- function(df) {
  df |>
    janitor::clean_names() |>
    dplyr::filter(!is.na(id))
}
```

## High-Performance Data Manipulation (data.table)

For performance-critical code and large datasets, use `data.table`.

**See [references/data-table.md](references/data-table.md) for the full guide and syntax.**

### When to Use
- Datasets > 1GB
- Complex grouping/aggregation
- Reference semantics (`:=`) needed for memory efficiency
- Speed is the primary constraint

## Modern Spatial Analysis

Use the modern spatial stack: `sf` (vector), `terra` (raster), and `stars` (spatiotemporal).
**Avoid** legacy packages: `sp`, `raster`, `rgdal`, `rgeos`.

**See [references/r-spatial.md](references/r-spatial.md) for the complete guide.**

### Quick Rules
1. **Vector**: Use `sf` and standard dplyr verbs.
2. **Raster**: Use `terra` for processing. Use `exactextractr` for heavy zonal stats.
3. **Cubes**: Use `stars` for time-series rasters or NetCDF.
4. **Coordinate Reference Systems**: Always check `st_crs()`/`crs()` matches before operations.

## Modern Tidyverse Essentials

### Native Pipe (`|>` not `%>%`)

Always use native pipe `|>` instead of magrittr `%>%` (R 4.1+):

```r
# Modern
data |>
  filter(year >= 2020) |>
  summarise(mean_value = mean(value))

# Avoid legacy pipe
data %>% filter(year >= 2020)
```

### Join Syntax (dplyr 1.1+)

Use `join_by()` for all joins:

```r
# Modern join syntax with equality
transactions |> 
  inner_join(companies, by = join_by(company == id))

# Inequality joins
transactions |>
  inner_join(companies, join_by(company == id, year >= since))

# Rolling joins (closest match)
transactions |>
  inner_join(companies, join_by(company == id, closest(year >= since)))
```

Control match behavior:

```r
# Expect 1:1 matches
inner_join(x, y, by = join_by(id), multiple = "error")

# Ensure all rows match
inner_join(x, y, by = join_by(id), unmatched = "error")
```

### Per-Operation Grouping with `.by`

Use `.by` instead of `group_by() |> ... |> ungroup()`:

```r
# Modern approach (always returns ungrouped)
data |>
  summarise(mean_value = mean(value), .by = category)

# Multiple grouping variables
data |>
  summarise(total = sum(revenue), .by = c(company, year))
```

### Column Operations

Use modern column selection and transformation functions:

```r
# pick() for column selection in data-masking contexts
data |>
  summarise(
    n_x_cols = ncol(pick(starts_with("x"))),
    n_y_cols = ncol(pick(starts_with("y")))
  )

# across() for applying functions to multiple columns
data |>
  summarise(across(where(is.numeric), mean, .names = "mean_{.col}"), .by = group)

# reframe() for multi-row results per group
data |>
  reframe(quantiles = quantile(x, c(0.25, 0.5, 0.75)), .by = group)
```

## rlang Metaprogramming

For comprehensive rlang patterns, see [references/rlang-patterns.md](references/rlang-patterns.md).

### Quick Reference

- **`{{}}`** - Forward function arguments to data-masking functions
- **`!!`** - Inject single expressions or values
- **`!!!`** - Inject multiple arguments from a list
- **`.data[[]]`** - Access columns by name (character vectors)
- **`pick()`** - Select columns inside data-masking functions

Example function with embracing:

```r
my_summary <- function(data, group_var, summary_var) {
  data |>
    summarise(mean_val = mean({{ summary_var }}), .by = {{ group_var }})
}
```

## Performance Optimization

**See [references/performance.md](references/performance.md) for profiling, benchmarking, and optimization strategies.**

### Key Strategies

1. **Profile first**: Use `profvis::profvis()` and `bench::mark()`
2. **Vectorize operations**: Avoid loops when vectorized alternatives exist
3. **Use dtplyr**: For large data operations (lazy evaluation with data.table backend)
4. **Parallel processing**: Use `furrr::future_map()` for parallelizable work
5. **Memory efficiency**: Pre-allocate, use appropriate data types

## Package Development

**See [references/package-development.md](references/package-development.md) for complete guidance.**
**See [references/documentation.md](references/documentation.md) for style guide and roxygen2 rules.**

### Quick Guidelines

**API Design:**
- Use `.by` parameter for per-operation grouping
- Use `{{}}` for column arguments
- Return tibbles consistently
- Validate user-facing function inputs thoroughly

**Dependencies:**
- Add dependencies for significant functionality gains
- Core tidyverse packages usually worth including: dplyr, purrr, stringr, tidyr
- Minimize dependencies for widely-used packages

**Testing:**
- Unit tests for individual functions
- Integration tests for workflows
- Test edge cases and error conditions

**Documentation:**
- Document all exported functions
- Provide usage examples
- Explain non-obvious parameter interactions

## Common Migration Patterns

### Base R → Tidyverse

```r
# Data manipulation
subset(data, condition)         → filter(data, condition)
data[order(data$x), ]          → arrange(data, x)
aggregate(x ~ y, data, mean)   → summarise(data, mean(x), .by = y)

# Functional programming
sapply(x, f)                   → map(x, f)  # type-stable
lapply(x, f)                   → map(x, f)

# Strings
grepl("pattern", text)         → str_detect(text, "pattern")
gsub("old", "new", text)       → str_replace_all(text, "old", "new")
```

### Old → New Tidyverse

```r
# Pipes
%>%                            → |>

# Grouping
group_by() |> ... |> ungroup() → summarise(..., .by = x)

# Joins
by = c("a" = "b")             → by = join_by(a == b)

# Reshaping
gather()/spread()              → pivot_longer()/pivot_wider()
```

## Additional Resources

- **High-Performance Data Table**: See [references/data-table.md](references/data-table.md) for the complete data.table guide
- **Spatial Analysis**: See [references/r-spatial.md](references/r-spatial.md) for sf, terra, and stars guidance
- **rlang patterns**: See [references/rlang-patterns.md](references/rlang-patterns.md) for comprehensive data-masking and metaprogramming guidance
- **Performance optimization**: See [references/performance.md](references/performance.md) for profiling, benchmarking, and optimization strategies
- **Package development**: See [references/package-development.md](references/package-development.md) for complete package creation guidance
- **Object systems**: See [references/object-systems.md](references/object-systems.md) for S3, S4, S7, R6, and vctrs guidance
