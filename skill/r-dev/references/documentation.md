# Documentation & Code Style

This guide defines the standards for documenting R code, emphasizing `roxygen2` for functions/packages and structured comments for scripts, adhering to Tidyverse style conventions and an 80-character line limit.

## Script Structure (Section Headers)

Use comment lines to break code into logical sections. Headers should use trailing hyphens `-` to fill the line up to **80 characters**.

- **Level 1**: `# Header -------------------------------------------------------------------`
- **Level 2**: `## Sub-header --------------------------------------------------------------`
- **Level 3**: `### Sub-sub-header ---------------------------------------------------------`
- [...]

```r
# Load Libraries ---------------------------------------------------------------
library(tidyverse)
library(here)

# Data Import ------------------------------------------------------------------
raw_data <- read_csv(here("data/input.csv"))

# Preprocessing ----------------------------------------------------------------
## Clean Names -----------------------------------------------------------------
clean_df <- raw_data |>
  janitor::clean_names()

## Handle Missing Values -------------------------------------------------------
clean_df <- clean_df |>
  drop_na(id)
```

## Function Documentation (`roxygen2`)

Use `roxygen2` comments (`#'`) for all functions, whether in packages or scripts. This allows automatic generation of manual pages (man/*.Rd) and namespace directives.

### Standard Tags
- `@title`: (First line) Short one-line summary.
- `@description`: (Second paragraph) Detailed explanation of what the function does.
- `@param`: Describe each argument and its type.
- `@return`: Describe the output object and its type.
- `@importFrom`: Explicitly declare external functions (`pkg fun`).
- `@export`: (Package only) Make function available to users.
- `@examples`: Executable code demonstrating usage.

### Example

```r
#' Calculate Summary Statistics by Group
#'
#' Computes the mean and standard deviation of a numeric variable for each
#' level of a grouping variable. Handles missing values by default.
#'
#' @param data A data frame or tibble containing the variables.
#' @param group_col The column name to group by (unquoted).
#' @param value_col The numeric column name to summarize (unquoted).
#' @param na.rm Logical. Should NA values be removed? Default is TRUE.
#'
#' @return A tibble with columns: `group_col`, `mean`, and `sd`.
#'
#' @importFrom dplyr group_by summarise
#' @importFrom rlang enquo
#' @export
#'
#' @examples
#' calculate_summary(mtcars, cyl, mpg)
#' calculate_summary(iris, Species, Sepal.Length)
calculate_summary <- function(data, group_col, value_col, na.rm = TRUE) {
  data |>
    summarise(
      mean = mean({{ value_col }}, na.rm = na.rm),
      sd = sd({{ value_col }}, na.rm = na.rm),
      .by = {{ group_col }}
    )
}
```

## Inline Comments

- **Explain WHY, not HOW**: Code tells you how it works; comments tell you why you wrote it that way.
- **Start with a space**: `# comment` not `#comment`.
- **Indent properly**: Align comments with the code they describe.

```r
# Bad
x <- x + 1 # add one to x

# Good
# Increment counter to include the boundary value
x <- x + 1
```

## R6 Class Documentation

Document R6 classes using `roxygen2` tags above the class definition. Use `@section` to document fields and methods.

```r
#' @title Person Class
#' @description Represents a person with a name and age.
#' @export
Person <- R6::R6Class("Person",
  public = list(
    #' @field name The person's full name.
    name = NULL,

    #' @description Create a new Person object.
    #' @param name Character string. Name of the person.
    #' @param age Integer. Age of the person.
    initialize = function(name, age) {
      self$name <- name
      private$age <- age
    }
  )
)
```
