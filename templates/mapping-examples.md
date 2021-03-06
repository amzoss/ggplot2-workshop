Mapping in R
================
Angela Zoss
3/23/2021

## Setup your environment

``` r
# Load required libraries

library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.6     ✓ dplyr   1.0.3
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
#install.packages("maps")
library(maps)
```

    ## 
    ## Attaching package: 'maps'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     map

``` r
#install.packages("mapproj")
library(mapproj)

#install.packages("sf")
library(sf)
```

    ## Linking to GEOS 3.8.1, GDAL 3.1.4, PROJ 6.3.1

## Using sf

``` r
# example from https://www.r-spatial.org/book/

# load NC data included in sf package, then transform to appropriate projection
# (North Carolina State Plane, with EPSG code 32119)

nc <- system.file("gpkg/nc.gpkg", package="sf") %>% read_sf() %>% st_transform(32119)
```

``` r
ggplot(nc) +
  geom_sf(aes(fill=BIR74)) +
    scale_fill_gradientn(colors = sf.colors())
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## California counties, random data

``` r
# load in the data for each county in California; 
# this function comes from the "maps" package

# ?map_data for more information about how to get basic polygons

county_map <- map_data("county","california")

# add a new column providing a random value for each county

county_map <- county_map %>% group_by(subregion) %>% mutate(rand = runif(1))

# With polygon data (data where you have a series of lat/lon points defining a polygon),
# you can use geom_polygon() and coord_map(). Make sure to assign the id of each polygon to 
# "group" to have each polygon appear as separate shapes.

ggplot(county_map) + 
  geom_polygon(aes(x = long,
                   y = lat,
                   group=group,
                   fill=rand)) +
  coord_map()
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## Buildings in Calaveras County, CA

``` r
# load in building data from a csv file
point_locations <- read_csv("../data/Calaveras-County-Government-Offices.csv")
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   Longitude = col_double(),
    ##   Latitude = col_double(),
    ##   Place = col_character(),
    ##   Category = col_character(),
    ##   Address = col_character()
    ## )

``` r
# color definitions; change these to change the look of the map
default_county_color <- "white"
default_county_border <- "gray80"
cal_county_color <- "white"
cal_county_border <- "red"
dot_color <- "blue"
dot_size <- 2 #can change this if the dots are too small or large
dot_transparency <- 0.5 #can change this if the dots are too light or dark

# three different ggplot pieces; one for all the counties, one for calaveras, and one for the lat/lon points
map_counties <- geom_polygon(data = county_map, aes(x = long, y = lat, group=group), fill=default_county_color, color=default_county_border)

map_calaveras <- geom_polygon(data = county_map[county_map$subregion=="calaveras",], aes(x = long, y = lat, group=group), fill=cal_county_color, color = cal_county_border, size=1)

map_points <- geom_point(data=point_locations, aes(x=Longitude, y=Latitude), color=dot_color, size=dot_size, alpha=dot_transparency)

# add everything into a single map object, then display the map
mp <- ggplot()
mp <- mp + map_counties 
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
mp <- mp + map_calaveras
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

``` r
mp <- mp + map_points
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-5-3.png)<!-- -->

``` r
# so far, the map is just displaying as if it's normal x/y data; we can add a
# coordinate system to project the map into a more normal ratio
# default projection is mercator(); other options are listed at ?mapproject

mp <- mp + coord_map()
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-5-4.png)<!-- -->

``` r
# optional: can zoom into map, using these long/lat as the new boundaries
zoom_long_left <- -122.0
zoom_long_right <- -119.0
zoom_lat_top <- 39.5
zoom_lat_bottom <- 37.0

# add this line (below) to zoom in 
# (that is, change the minimum and maximum values shown)
mp <- mp + coord_map(xlim=c(zoom_long_left,zoom_long_right),
                     ylim=c(zoom_lat_bottom,zoom_lat_top))
```

    ## Coordinate system already present. Adding new coordinate system, which will replace the existing one.

``` r
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-5-5.png)<!-- -->

## El Niño measurements

``` r
elnino <- read_csv("../data/elnino.csv", na=c(".",NA), 
                   col_types = cols(Date = col_date(format="%y%m%d"),
                                    Humidity = col_double())) %>% 
  type_convert()
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols()

``` r
elnino$Longitude <- ifelse(elnino$Longitude < 0, 360+elnino$Longitude,elnino$Longitude)

# load in the data for each county in California; 
# this function comes from the "maps" package, which gets loaded automatically

world_map <- map_data("world2")

# ?map_data for more information about how to get basic polygons

# color definitions; change these to change the look of the map
default_county_color <- "white"
default_county_border <- "gray80"

dot_color <- "blue"
dot_size <- 2 #can change this if the dots are too small or large
dot_transparency <- 0.5 #can change this if the dots are too light or dark

# three different ggplot pieces; one for all the counties, one for calaveras, and one for the lat/lon points
map_countries <- geom_polygon(data = world_map, aes(x = long, y = lat, group=group), fill=default_county_color, color=default_county_border)

map_buoys <- geom_point(aes(x=elnino$Longitude, y=elnino$Latitude), color=dot_color, size=dot_size, alpha=dot_transparency)

map_bin2d <- geom_bin2d(aes(x=elnino$Longitude, y=elnino$Latitude))

# add everything into a single map object, then display the map
mp <- ggplot()
mp <- mp + map_countries 
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
# add coord_map to show polygons in Mercator projection
mp <- mp + coord_map()
mp
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-6-2.png)<!-- -->

``` r
# try alternative data overlays
mp + map_buoys
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-6-3.png)<!-- -->

``` r
mp + map_bin2d
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-6-4.png)<!-- -->

``` r
# example from ggplot2 cheatsheet

# create a small data frame of state name and murder rate
data <- data.frame(murder = USArrests$Murder, state = tolower(rownames(USArrests)))

# grab state polygons from map_data
map <- map_data("state")

# In this example, still getting polygon data from map_data, but the numerical
# data that is used for fill is not included inside the polygon data.
# The fill is mapped from the USArrests dataset, but then the polygon
# data (from "map") is passed to the geom_map layer via the map attribute.

# The expand_limits command seems to be necessary because the plot doesn't
# have access to lat and long as the y and x variables in the normal way.


ggplot(data, aes(fill = murder)) + 
  geom_map(aes(map_id = state), map = map) + 
  expand_limits(x = map$long, y = map$lat) +
  coord_map(projection = "azequalarea")
```

![](mapping-examples_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->
