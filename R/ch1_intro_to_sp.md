Intro to spatial object
================

``` r
library(sp)
```

    ## Warning: package 'sp' was built under R version 4.2.1

``` r
library(sf)
```

    ## Linking to GEOS 3.9.1, GDAL 3.3.2, PROJ 7.2.1; sf_use_s2() is TRUE

## Conceptualizing spa)tial vector objects in R

The foundational structure for any spatial object in {sp} is the
`Spatial` class with two slots

-   a bounding box
-   a CRS class object to define CRS (Coordinate Reference System)

To manually build up a spatial object, the steps are:

1.  Create geometric objects (Points, Lines, Polygons)

-   Points: generated out of either a single coordinate or set of
    coordinates, like two column matrix or dataframe with columns for
    latitude and longitude.

-   Lines: generated out of `Line` objects and `Line` object is a
    collection of 2D coordinates and generated out of a two column
    matrix or dataframe with one column for latitude and one column for
    longitude. so a `Lines` object is a list of one or more `Line`
    objects, for example all the contours at a single elevation.

-   Polygons: generated out of `Polygon` objects which is a collection
    of 2D coordinates with equal first and last coordinates and is
    generated out of a two column matrix and dataframe with columns for
    latitude and longitude. So a `Polygons` object is a list of one or
    more `Polygon` objects, say, islands belonging to the same country

2.  Create spatial objects (SpatialPoints, SpatialLines,
    SpatialPolygons)

This step adds the bounding box (automatically) and slot for CRS (which
needs to be filled manually)

3.  Add attributes (Optional)

If we add a dataframe with attribute data, then SpatialPoints would be a
SpatialPointsDataFrame object, similarly we would have
SpatialLinesDataFrame, SpatialPolygonsDataFrame objects

## Example (creating a sp object that contains highways)

``` r
# First we would create a `Line` object that holds one highway.
ln1 <- Line(matrix(runif(6), ncol = 2)) # 1st highway
ln2 <- Line(matrix(runif(6), ncol = 2)) # 2nd highway

# then create the Lines object for the two highway
lns1 <- Lines(list(ln1), ID = c("hw1"))
lns2 <- Lines(list(ln2), ID = c("hw2"))

# turn lines into a geospatial object
sp_lns <- SpatialLines(list(lns1, lns2))

dfr <- data.frame(id = c("hw1", "hw2"),
                  cars_par_hour = c(78, 22))

sp_lns_dfr <- SpatialLinesDataFrame(sp_lns, dfr, match.ID = "id")
```

## The {sf} package

`sf` implements a formal standard called “Simple features” that
specifies the storage and access model of Spatial Geometries (point,
line, polygon)

In `sf` spatial objects are stored as a simple data frame with a special
column that contains the information for the geometry coordinates. That
special column is a list with the same length as the number of rows in
the data frame and each of the individual elements of that column can by
of any length needed to hold the coordinates that corresponds to an
individual feature.

To create a `sf` object manually, the steps are:

1.  Create geometric objects (point, line, polygon)

-   Geometric object (simple features) can be created from a numeric
    vector, matrix or a list with the coordinates. They are called `sfg`
    (simple feature geometry). we can use `st_point()`,
    `st_linestring()`, `st_polygon()` to create these simple features.

2.  Combine all the individual single feature objects for the special
    column

-   feature geometries are combined into a simple feature collection
    (sfc) using `st_sfc()`. The sfc object also holds the bbox and
    projection information.

3.  Add attributes with st_sf()

-   st_sf() extends the well known R data.frame with a column that hold
    sfc

## Example (creating a sf object that contains highways)

``` r
lnstr_sfg1 <- st_linestring(matrix(runif(6), ncol = 2))
lnstr_sfg2 <- st_linestring(matrix(runif(6), ncol = 2))

lnstr_sfc <- st_sfc(lnstr_sfg1, lnstr_sfg2)
lnstr_sfc
```

    ## Geometry set for 2 features 
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: 0.0773848 ymin: 0.08403943 xmax: 0.8615646 ymax: 0.8214873
    ## CRS:           NA

    ## LINESTRING (0.1387417 0.08403943, 0.8615646 0.2...

    ## LINESTRING (0.4475611 0.6394715, 0.7272956 0.61...

``` r
lnstr_sf <- st_sf(dfr, lnstr_sfc)
lnstr_sf
```

    ## Simple feature collection with 2 features and 2 fields
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: 0.0773848 ymin: 0.08403943 xmax: 0.8615646 ymax: 0.8214873
    ## CRS:           NA
    ##    id cars_par_hour                      lnstr_sfc
    ## 1 hw1            78 LINESTRING (0.1387417 0.084...
    ## 2 hw2            22 LINESTRING (0.4475611 0.639...

## Challange

Similarly to the example above generate a Point object in R. Use both,
the sp and the sf “approach”.

1.  Create a matrix pts of random numbers with two columns and as many
    rows as you like. These are your points.
2.  Create a dataframe attrib_df with the same number of rows as your
    pts matrix and a column that holds an attribute. You can make up any
    attribute.
3.  Use the appropriate commands and pts to create

-   a SpatialPointsDataFrame and
-   an sf object with a gemoetry column of class sfc_POINT.

4.  Try to subset your spatial object using the attribute you have added
    and the way you are used to from regular data frames.
5.  How do you determine the bounding box of your spatial object?

``` r
pts <- matrix(runif(10), ncol = 2)
attrib_df <- data.frame(popn = sample(50:90, size = nrow(pts)))

# using sp
sp_pts_dfr <- SpatialPointsDataFrame(pts, attrib_df)
sp_pts_dfr
```

    ##              coordinates popn
    ## 1  (0.102645, 0.2166235)   80
    ## 2 (0.3076727, 0.5299168)   75
    ## 3 (0.3786552, 0.3168698)   86
    ## 4 (0.1414986, 0.9055569)   52
    ## 5 (0.4715545, 0.8866251)   74

``` r
# using sf
pts_sf <- cbind(attrib_df, data.frame(pts)) |> st_as_sf(coords = c(2, 3))
```

## Creating a Spatial object from a lat/lon table

### Using {Sf}

An `sf` object can be created from a data frame using `st_as_sf()`

``` r
philly_homicides_df <- readr::read_csv("data/philly_homicides.csv")
```

    ## Rows: 3883 Columns: 10
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): SECTOR, LOCATION_BLOCK, TEXT_GENERAL_CODE
    ## dbl  (5): DC_DIST, UCR_GENERAL, OBJ_ID, POINT_X, POINT_Y
    ## date (1): DISPATCH_DATE
    ## time (1): DISPATCH_TIME
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
philly_homicides_sf <- st_as_sf(philly_homicides_df,
                                coords = c("POINT_X", "POINT_Y"))
```

To have a complete geo object we need to set a CRS. we assign the WGS84
projection which has the EPSG code 4326.

On a side note: full form of EPSG is “European Petrolium Survey Group”

``` r
st_crs(philly_homicides_sf)
```

    ## Coordinate Reference System: NA

``` r
st_crs(philly_homicides_sf) <- 4326
```

To save the sf object as a shapefile we use `st_write()`

``` r
st_write(philly_homicides_sf, "data/PhillyHomicides", driver = "ESRI Shapefile", delete_layer = TRUE)
```

    ## Warning in abbreviate_shapefile_names(obj): Field names abbreviated for ESRI
    ## Shapefile driver

    ## Warning in clean_columns(as.data.frame(obj), factorsAsCharacter): Dropping
    ## column(s) DISPATCH_T of class(es) hms;difftime

    ## Deleting layer `PhillyHomicides' using driver `ESRI Shapefile'
    ## Writing layer `PhillyHomicides' to data source 
    ##   `data/PhillyHomicides' using driver `ESRI Shapefile'
    ## Writing 3883 features with 7 fields and geometry type Point.

### Using {sp})

A SpatialPointsDataFrame object can be create directly from a dataframe
by specifing which columns contain the coordinates and this is can be
done with `coordinates()` function and *this approach replaces the
original data frame* so be warned.

``` r
philly_homicides_df <- subset(philly_homicides_df, select = -c(DISPATCH_TIME))

coordinates(philly_homicides_df) <- c("POINT_X", "POINT_Y")

class(philly_homicides_df)
```

    ## [1] "SpatialPointsDataFrame"
    ## attr(,"package")
    ## [1] "sp"

``` r
# assigning the projection

is.projected(philly_homicides_df)
```

    ## [1] NA

``` r
proj4string(philly_homicides_df) <- CRS("+init=epsg:4326")
```

To save this sp object

``` r
library(rgdal, quietly = TRUE)
```

    ## Warning: package 'rgdal' was built under R version 4.2.1

    ## Please note that rgdal will be retired by the end of 2023,
    ## plan transition to sf/stars/terra functions using GDAL and PROJ
    ## at your earliest convenience.
    ## 
    ## rgdal: version: 1.5-32, (SVN revision 1176)
    ## Geospatial Data Abstraction Library extensions to R successfully loaded
    ## Loaded GDAL runtime: GDAL 3.4.3, released 2022/04/22
    ## Path to GDAL shared files: C:/Users/User/AppData/Local/R/win-library/4.2/rgdal/gdal
    ## GDAL binary built with GEOS: TRUE 
    ## Loaded PROJ runtime: Rel. 7.2.1, January 1st, 2021, [PJ_VERSION: 721]
    ## Path to PROJ shared files: C:/Users/User/AppData/Local/R/win-library/4.2/rgdal/proj
    ## PROJ CDN enabled: FALSE
    ## Linking to sp version:1.5-0
    ## To mute warnings of possible GDAL/OSR exportToProj4() degradation,
    ## use options("rgdal_show_exportToProj4_warnings"="none") before loading sp or rgdal.

``` r
writeOGR(philly_homicides_df, "data/PhillyHomicides", "PhillyHomicides", driver = "ESRI Shapefile", overwrite_layer = TRUE)
```

    ## Warning in writeOGR(philly_homicides_df, "data/PhillyHomicides",
    ## "PhillyHomicides", : Field names abbreviated for ESRI Shapefile driver

> `writeOGR()` cannot convert data of class `hms`. So had to remove
> DISPATCH_TIME column just to avoid digggin in right now. But have to
> check a solution for this later.

## Loading Shape files

### Using sf

``` r
philly_sf <- st_read("data/Philly")
```

    ## Reading layer `PhillyTotalPopHHinc' from data source 
    ##   `C:\Users\User\Desktop\R-desk\R_spatial\data\Philly' using driver `ESRI Shapefile'
    ## Simple feature collection with 384 features and 17 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 1739497 ymin: 457343.7 xmax: 1764030 ymax: 490544.9
    ## Projected CRS: Albers

As a side note: you can turn a sf dataframe back to normal non spatial
dataframe using `st_drop_geometry()`

``` r
plot(philly_sf)
```

    ## Warning: plotting the first 10 out of 17 attributes; use max.plot = 17 to plot
    ## all

![](ch1_intro_to_sp_files/figure-gfm/sf-plot-1.png)<!-- -->

In order to plot only the polygon boundaries we need to specify to
directly use geometry column with `st_geometry`

``` r
plot(st_geometry(philly_sf))
```

![](ch1_intro_to_sp_files/figure-gfm/sf-geometry-1.png)<!-- -->

To get only the polygons for which median income is \> 60000 (actually
we are adding layers on the original boundaries)

``` r
philly_sf_rich <- subset(philly_sf, medHHinc > 60000)

plot(st_geometry(philly_sf))
plot(st_geometry(philly_sf_rich), add = TRUE, col = "red")
```

![](ch1_intro_to_sp_files/figure-gfm/sf-geometry-01-1.png)<!-- -->

### loading shapefile with {sp}

In order to read spatial data into R and to turn them into Spatial\*
family objects we require `rgdal` package which provides bindings to
GDAL.

> GDAL also known as GDAL/OGR, is a library of tools used for
> manipulating geospatial data (for both raster and vector data types)
> and its a open-source geospatial data IO library. GDAL supports over
> 200 raster formats and vector formats. We can use `ogrDrivers()` and
> `gdalDrivers()` to see which vector and raster formats `rgdal`
> supports.

We can read in and write out spatial data using

    readOGR() and writeOGR() for vector format
    readGDAL() and writeGDAL() for raster format

> A shapefile consists of various files of the same name but with
> different extensions. Shapefile consists of a collection of files with
> a common filename prefix, stored in the same directory. The three
> mandatory files have filename extensions `.shp`, `.shx`, `.dbf`. The
> actual shapefile relates specifically to the `.shp` file, but alone is
> incomplete for distribution as the other supporting files are
> required.

*Mandatory files* - `.shp` - shape format: the feature geometry itself -
`.shx` - shape index format: a positional index of the feature geometry
to allow seeking forwards and backwards - `.dbf` - attribute format

*Other files*
