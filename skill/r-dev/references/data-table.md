# High-Performance Data Manipulation with data.table

`data.table` enhances R's `data.frame` for **high-performance** data manipulation on large datasets, emphasizing concise syntax, reference semantics, and speed.

## Core Philosophy

- **Concise syntax**: Single `DT[i, j, by]` expression handles filtering, selection, grouping, updates.
- **Reference semantics**: Modify data in-place with `:=` and `set*()` functions to avoid memory copies.
- **High performance**: Binary search joins, parallel operations, optimized I/O; scales to billions of rows.

## General Syntax: `DT[i, j, by]`

**Filter rows** (`i`), **operate on columns** (`j`), **group by** (`by`).

```r
library(data.table)
DT <- data.table(x = 1:10, y = letters[1:10], z = rnorm(10))

# Basic examples
DT[3:5]                          # Rows 3-5
DT[x > 5]                        # Filter x > 5
DT[x %in% 1:3, .(y, z)]          # Filter + select columns as DT
DT[, sum(x)]                     # Sum x across all rows
DT[, .(sum_x = sum(x)), by = y]  # Grouped sum
```

**`.()`** creates `data.table` lists; equivalent to `list()`.

## Cheat Sheet: data.table vs dplyr

| Operation | dplyr | data.table |
|-----------|-------|------------|
| **Filter** | `filter(mpg > 20)` | `DT[mpg > 20]` |
| **Select** | `select(cyl, mpg)` | `DT[, .(cyl, mpg)]` |
| **Arrange** | `arrange(desc(mpg))` | `DT[order(-mpg)]` |
| **Mutate** | `mutate(kpl = mpg/2.35)` | `DT[, kpl := mpg/2.35]` |
| **Summarise** | `summarise(mean_mpg = mean(mpg))` | `DT[, .(mean_mpg = mean(mpg))]` |
| **Group By** | `group_by(cyl) %>% summarise(...)` | `DT[, .(mean_mpg = mean(mpg)), by = cyl]` |
| **Join** | `left_join(dt1, dt2, by = "id")` | `dt1[dt2, on = "id"]` (keys required for speed) |

Chaining: `DT[filter][, mutate, by = group][order_var]`.

## Key Features

### Reference Semantics with `:=`
Modifies **in-place** (no copy), saving memory on large data.

```r
DT[, new_col := x * 2]           # Add column by reference
DT[, x := NULL]                  # Delete column
DT[J(x > 5), x := 0]             # Update filtered rows
```

**`set()`** for fast row/col assignments:
```r
set(DT, i = 1:3, j = "x", value = 999)  # Set rows 1-3, col "x"
```

### `.SD` and `.SDcols`
**`.SD`**: Subset of data excluding `by` vars (Sub-Data.table).
**`.SDcols`**: Select columns for `.SD`.

```r
DT[, lapply(.SD, mean), by = y, .SDcols = c("x", "z")]  # Mean of selected cols by group
DT[, (cols) := lapply(.SD, sum), .SDcols = patterns("num")]  # Dynamic col selection
```

### Fast I/O: `fread` / `fwrite`
**10-100x faster** than `read.csv`/`write.csv`.

```r
DT <- fread("large_file.csv", nrows = 1e6, select = c("col1", "col2"))
fwrite(DT, "output.csv", nThread = 4)  # Parallel write
```

### Keys and Binary Search
**`setkey()`** sorts and indexes columns for **O(log n)** joins/filters.

```r
setkey(DT, x)       # Set key (sorts DT)
DT[J(5)]            # Fast lookup x == 5
dt1[dt2, on = "x"]  # Keyed join (left by default)
```

## Performance Best Practices

### `set*` Functions (In-Place, No Copy)
```r
setnames(DT, "old", "new")       # Rename
setcolorder(DT, c("z", "x", "y"))# Reorder
setkey(DT, "x")                  # Key
```

### Pre-Allocation
```r
DT <- vector("list", 1e6)        # Pre-allocate list
DT <- data.table(matrix(0, 1e6, 10))  # Pre-allocate DT
```

### Conditional Logic: `fcase` / `fifelse`
**Vectorized** alternatives to `ifelse()` (handles NA).

```r
DT[, grade := fcase(
  score >= 90, "A",
  score >= 80, "B",
  default = "F"
)]

DT[, fast_ifelse := fifelse(x > 0 & !is.na(x), "pos", "other")]
```

**`.N`**: Group size; **`.I`**: Row indices.
```r
DT[, .(n_rows = .N), by = y]     # Count per group
```

## Integration with Tidyverse

Pure `data.table` preferred for speed, but interoperable:
```r
library(dtplyr)
dt_lazy <- lazy_dt(DT)           # dplyr syntax on data.table backend
result <- dt_lazy %>%
  filter(x > 5) %>%
  summarise(m = mean(z)) %>%
  collect()                       # Execute
```

**Conversion**: `setDT(df)` (in-place); `setDF(dt)`.

**Pro Tip**: Use `data.table` for production pipelines; `dtplyr` for tidyverse teams transitioning.
