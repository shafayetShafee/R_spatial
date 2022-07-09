Spatial Data Manipulation in R
================

## Attribute Join in sf

Its the process of joining data in tabular format to a data in a format
that holds geometries (point, line, polygon)

``` r
library(sf)
```

    ## Linking to GEOS 3.9.1, GDAL 3.3.2, PROJ 7.2.1; sf_use_s2() is TRUE

``` r
ph_edu <- read.csv("data/PhillyEducation.csv")
names(ph_edu)
```

    ##  [1] "GEOID"           "NAME"            "fem_bachelor"    "fem_doctorate"  
    ##  [5] "fem_highschool"  "fem_noschool"    "fem_ovr_25"      "male_bachelor"  
    ##  [9] "male_doctorate"  "male_highschool" "male_noschool"   "male_ovr_25"    
    ## [13] "pop_ovr_25"

``` r
philly_sf <- st_read("data/Philly/")
```

    ## Reading layer `PhillyTotalPopHHinc' from data source 
    ##   `C:\Users\User\Desktop\R-desk\R_spatial\data\Philly' using driver `ESRI Shapefile'
    ## Simple feature collection with 384 features and 17 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 1739497 ymin: 457343.7 xmax: 1764030 ymax: 490544.9
    ## Projected CRS: Albers

``` r
names(philly_sf)
```

    ##  [1] "STATEFP10"  "COUNTYFP10" "TRACTCE10"  "GEOID10"    "NAME10"    
    ##  [6] "NAMELSAD10" "MTFCC10"    "FUNCSTAT10" "ALAND10"    "AWATER10"  
    ## [11] "INTPTLAT10" "INTPTLON10" "GISJOIN"    "Shape_area" "Shape_len" 
    ## [16] "medHHinc"   "totalPop"   "geometry"

So merging by `GEOID10` which is same as `GEOID`

``` r
philly_sf_merged <- merge(philly_sf, ph_edu, by.x = "GEOID10", by.y = "GEOID")

names(philly_sf_merged)
```

    ##  [1] "GEOID10"         "STATEFP10"       "COUNTYFP10"      "TRACTCE10"      
    ##  [5] "NAME10"          "NAMELSAD10"      "MTFCC10"         "FUNCSTAT10"     
    ##  [9] "ALAND10"         "AWATER10"        "INTPTLAT10"      "INTPTLON10"     
    ## [13] "GISJOIN"         "Shape_area"      "Shape_len"       "medHHinc"       
    ## [17] "totalPop"        "NAME"            "fem_bachelor"    "fem_doctorate"  
    ## [21] "fem_highschool"  "fem_noschool"    "fem_ovr_25"      "male_bachelor"  
    ## [25] "male_doctorate"  "male_highschool" "male_noschool"   "male_ovr_25"    
    ## [29] "pop_ovr_25"      "geometry"

## Attribute joins in sp

``` r
philly_sp <- rgdal::readOGR("data/Philly/", "PhillyTotalPopHHinc")
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "C:\Users\User\Desktop\R-desk\R_spatial\data\Philly", layer: "PhillyTotalPopHHinc"
    ## with 384 features
    ## It has 17 fields

``` r
philly_sp_merged <- merge(philly_sp, ph_edu, by.x = "GEOID10", by.y = "GEOID")
names(philly_sp_merged)
```

    ##  [1] "GEOID10"         "STATEFP10"       "COUNTYFP10"      "TRACTCE10"      
    ##  [5] "NAME10"          "NAMELSAD10"      "MTFCC10"         "FUNCSTAT10"     
    ##  [9] "ALAND10"         "AWATER10"        "INTPTLAT10"      "INTPTLON10"     
    ## [13] "GISJOIN"         "Shape_area"      "Shape_len"       "medHHinc"       
    ## [17] "totalPop"        "NAME"            "fem_bachelor"    "fem_doctorate"  
    ## [21] "fem_highschool"  "fem_noschool"    "fem_ovr_25"      "male_bachelor"  
    ## [25] "male_doctorate"  "male_highschool" "male_noschool"   "male_ovr_25"    
    ## [29] "pop_ovr_25"

## Topological subsetting (select polygons by Location)

we want to select all Philadelphia census tracts within a range of 2
kilometers from the city center.

The steps could be

1.  Get the census tract polygons
2.  Find the philadelphia city center coordinates
3.  create a buffer around the city center point
4.  select all the census tract polygons that intersects with the center
    buffer.

### Using sf package
