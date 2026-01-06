# Modern Spatial Analysis in R

Modern spatial R development relies on three core packages: `sf` (vector), `terra` (raster), and `stars` (spatiotemporal). Avoid legacy packages (`sp`, `raster`, `rgdal`, `rgeos`) as they are retired or deprecated.

## 1. Vector Data: `sf` (Simple Features)

The `sf` package encodes spatial geometry as a list-column in a data frame, allowing full compatibility with the `tidyverse`.

### Core Syntax

| Operation | Function | Example |
|-----------|----------|---------|
| **Read** | `st_read()` | `nc <- st_read("shapefile.shp")` |
| **Write** | `st_write()` | `st_write(nc, "output.gpkg")` |
| **Transform CRS** | `st_transform()` | `nc_proj <- st_transform(nc, 3857)` |
| **Get CRS** | `st_crs()` | `st_crs(nc)` |
| **Bounding Box** | `st_bbox()` | `st_bbox(nc)` |
| **To SF** | `st_as_sf()` | `st_as_sf(df, coords = c("lon", "lat"), crs = 4326)` |
| **Drop Geometry** | `st_drop_geometry()` | `df <- st_drop_geometry(nc)` |

### Tidyverse Integration

`sf` objects are data.frames. Use `dplyr` verbs directly:

```r
library(sf)
library(dplyr)

# Filter spatial objects just like data frames
nc_filtered <- nc |>
  filter(AREA > 0.1) |>
  select(NAME, geometry)

# Grouped operations (union geometries by group)
regions <- nc |>
  summarise(geometry = st_union(geometry), .by = REGION)
```

### Geometric Operations

```r
# Buffer (units depend on CRS)
buffered <- st_buffer(nc, dist = 100)

# Intersection (clip)
clipped <- st_intersection(nc, roi_polygon)

# Spatial Join (left join based on location)
# Join points to polygons containing them
joined <- st_join(points, polygons, join = st_intersects)
```

### Best Practices
- **S2 Geometry**: `sf` uses S2 (spherical) geometry by default for geographic coords (lon/lat). If you encounter errors, check validity or temporarily switch to planar: `sf_use_s2(FALSE)`.
- **CRS**: Always ensure CRS matches before operations (`st_crs(x) == st_crs(y)`).

## 2. Raster Data: `terra`

`terra` replaces the `raster` package. It is faster (C++) and pointer-based.

### Core Classes
- **`SpatRaster`**: Multi-layer raster data.
- **`SpatVector`**: `terra`'s internal vector format (lighter than `sf`).

### Core Syntax

```r
library(terra)

# Read
r <- rast("elevation.tif")

# Create from scratch
r <- rast(nrows=10, ncols=10, xmin=0, xmax=1, ymin=0, ymax=1)
values(r) <- 1:ncell(r)

# Manipulate
r_crop <- crop(r, vector_extent)    # Cut to extent
r_mask <- mask(r_crop, vector_mask) # Set values outside polygon to NA

# Project (Reproject)
r_proj <- project(r, "EPSG:3857")

# Extract values to points/polygons
# For simple extraction:
vals <- extract(r, vect(sf_object))
```

### Raster-Vector Interaction
- Convert `sf` to `SpatVector` with `vect(sf_obj)`.
- Convert `SpatVector` to `sf` with `st_as_sf(spat_vect)`.
- **Optimization**: For extracting raster values to *many large polygons*, use the **`exactextractr`** package instead of `terra::extract`.

```r
library(exactextractr)
# Much faster zonal statistics
stats <- exact_extract(r, sf_polygons, fun = c("mean", "max"))
```

## 3. Spatiotemporal Arrays: `stars`

Use `stars` for data cubes (x, y, time, bands), such as satellite time series or climate models (NetCDF).

### Core Syntax

```r
library(stars)

# Read (automatically detects dimensions)
s <- read_stars("climate_model.nc")

# Convert raster to stars
s <- st_as_stars(terra_raster)

# Filter dimensions (e.g., time)
s_slice <- filter(s, time > as.Date("2020-01-01"))

# Aggregate (reduce dimension)
# Mean over time
s_mean <- aggregate(s, by = "time", FUN = mean)
```

## Summary Comparison

| Task | Package | Key Object | Legacy Equivalent |
|------|---------|------------|-------------------|
| Vector Data | `sf` | `sf` / `sfc` | `sp` (`SpatialPolygonsDataFrame`) |
| Raster Analysis | `terra` | `SpatRaster` | `raster` |
| Data Cubes | `stars` | `stars` | `raster` (stacks) |
| Large Zonal Stats | `exactextractr` | - | `raster::extract` |
