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

## Conceptualizing spatial vector objects in R

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
    ## Bounding box:  xmin: 0.008177101 ymin: 0.014693 xmax: 0.8891981 ymax: 0.9410715
    ## CRS:           NA

    ## LINESTRING (0.313836 0.6738433, 0.8568102 0.857...

    ## LINESTRING (0.5459799 0.886321, 0.008177101 0.4...

``` r
lnstr_sf <- st_sf(dfr, lnstr_sfc)
lnstr_sf
```

    ## Simple feature collection with 2 features and 2 fields
    ## Geometry type: LINESTRING
    ## Dimension:     XY
    ## Bounding box:  xmin: 0.008177101 ymin: 0.014693 xmax: 0.8891981 ymax: 0.9410715
    ## CRS:           NA
    ##    id cars_par_hour                      lnstr_sfc
    ## 1 hw1            78 LINESTRING (0.313836 0.6738...
    ## 2 hw2            22 LINESTRING (0.5459799 0.886...

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
    ## 1 (0.9054533, 0.4205953)   66
    ## 2  (0.356249, 0.3693913)   74
    ## 3 (0.3130491, 0.1541322)   86
    ## 4 (0.8524506, 0.4514962)   65
    ## 5 (0.5663895, 0.4704432)   62

``` r
# using sf
pts_sf <- cbind(attrib_df, data.frame(pts)) |> st_as_sf(coords = c(2, 3))
```
