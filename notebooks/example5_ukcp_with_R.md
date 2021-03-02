Met Office Climate Data Hackathon
================
R programming examples
2/3/2021

# Met Office Climate Data Hackathon

## Using R with UK Climate Projections data

This notebook shows you how to work with the UK Climate Projections
available on the CEDA Archive using R. It demonstrates:

  - how to read in a dataset,
  - perform a few statistical tasks,
  - calculate decadal means,
  - calculate a simple spatial-average,
  - calculate monthly anomalies",
  - how to create a simple and more advanced plots

This code requires the following:

  - UKCP data files
  - monthly mean monthly mean surface air temperature (available from
    CEDA Archive
    [here](http://dap.ceda.ac.uk/badc/ukcp18/data/land-rcm/uk/12km/rcp85/01/tas/mon/latest/tas_rcp85_land-rcm_uk_12km_01_mon_198012-208011.nc))
  - land mask (available from here)
  - daily maximum surface air temperatures (available from CEDA Archive
    [here](http://data.ceda.ac.uk/badc/ukcp18/data/land-rcm/uk/12km/rcp85/01/tasmax/day/latest))

Within R they are many ways of producing similar plots or analysis. The
examples shown below illustrate one approach. I make no claim that this
is the best or most efficient approach. I’m always interested in
learning new or different approaches to writing code in R. If you have
some comments or code that you would like to share, I can be contacted
at <kate.brown@metoffice.gov.uk>. I hope you enjoy the Hackathon.

Kate Brown 1/3/2021

## Preparatory actions

Load in relevant R libraries

    ## Loading required package: sp

    ## Checking rgeos availability: TRUE

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

    ## 
    ## Attaching package: 'raster'

    ## The following object is masked from 'package:nlme':
    ## 
    ##     getData

    ## Loading required package: viridisLite

Specify directories and
files

``` r
ukcp_file <- 'Insert directory for downloaded file here/tas_rcp85_land-rcm_uk_12km_01_mon_198012-208011.nc'
ukcp_file_land_mask <-'Insert directory for downloaded file here/lsm_land-rcm_uk_12km.nc'
ukcp_file_directory_tasmax <- 'Insert directory for downloaded file here'

```

Load a netCDF data files into R and print information about its
contents.

``` r
ncin <- nc_open(ukcp_file)

print(ncin)
```

    ## File /project/ukcp/land-rcm/uk/12km/rcp85/01/tas/mon/v20190731/tas_rcp85_land-rcm_uk_12km_01_mon_198012-208011.nc (NC_FORMAT_NETCDF4_CLASSIC):
    ## 
    ##      11 variables (excluding dimension variables):
    ##         float tas[projection_x_coordinate,projection_y_coordinate,time,ensemble_member]   
    ##             _FillValue: 1.00000002004088e+20
    ##             standard_name: air_temperature
    ##             long_name: Mean air temperature
    ##             units: degC
    ##             description: Mean air temperature
    ##             label_units: °C
    ##             plot_label: Mean air temperature at 1.5m (°C)
    ##             cell_methods: time: mean
    ##             grid_mapping: transverse_mercator
    ##             coordinates: ensemble_member_id grid_latitude grid_longitude month_number year yyyymm
    ##         int transverse_mercator[]   
    ##             grid_mapping_name: transverse_mercator
    ##             longitude_of_prime_meridian: 0
    ##             semi_major_axis: 6377563.396
    ##             semi_minor_axis: 6356256.909
    ##             longitude_of_central_meridian: -2
    ##             latitude_of_projection_origin: 49
    ##             false_easting: 4e+05
    ##             false_northing: -1e+05
    ##             scale_factor_at_central_meridian: 0.9996012717
    ##         double time_bnds[bnds,time]   
    ##         double projection_y_coordinate_bnds[bnds,projection_y_coordinate]   
    ##         double projection_x_coordinate_bnds[bnds,projection_x_coordinate]   
    ##         char ensemble_member_id[string27,ensemble_member]   
    ##             units: 1
    ##             long_name: ensemble_member_id
    ##         double grid_latitude[projection_x_coordinate,projection_y_coordinate]   
    ##             units: degrees_north
    ##             standard_name: grid_latitude
    ##         double grid_longitude[projection_x_coordinate,projection_y_coordinate]   
    ##             units: degrees_east
    ##             standard_name: grid_longitude
    ##         int month_number[time]   
    ##             units: 1
    ##             long_name: month_number
    ##         int year[time]   
    ##             units: 1
    ##             long_name: year
    ##         char yyyymm[string64,time]   
    ##             units: 1
    ##             long_name: yyyymm
    ## 
    ##      7 dimensions:
    ##         ensemble_member  Size:1
    ##             units: 1
    ##             long_name: ensemble_member
    ##         time  Size:1200   *** is unlimited ***
    ##             axis: T
    ##             bounds: time_bnds
    ##             units: hours since 1970-01-01 00:00:00
    ##             standard_name: time
    ##             calendar: 360_day
    ##         projection_y_coordinate  Size:112
    ##             axis: Y
    ##             bounds: projection_y_coordinate_bnds
    ##             units: m
    ##             standard_name: projection_y_coordinate
    ##         projection_x_coordinate  Size:82
    ##             axis: X
    ##             bounds: projection_x_coordinate_bnds
    ##             units: m
    ##             standard_name: projection_x_coordinate
    ##         bnds  Size:2
    ##         string27  Size:27
    ##         string64  Size:64
    ## 
    ##     15 global attributes:
    ##         collection: land-rcm
    ##         contact: ukcpproject@metoffice.gov.uk
    ##         creation_date: 2019-07-31T00:00
    ##         domain: uk
    ##         frequency: mon
    ##         institution: Met Office Hadley Centre (MOHC), FitzRoy Road, Exeter, Devon, EX1 3PB, UK.
    ##         institution_id: MOHC
    ##         project: UKCP18
    ##         references: https://ukclimateprojections.metoffice.gov.uk
    ##         resolution: 12km
    ##         scenario: rcp85
    ##         source: UKCP18 regional realisation from a set of 12 limited-area Met Office Unified Model Global Atmosphere GA7 models at 12km resolution driven by perturbed variants of the global HadGEM3-GC3.05
    ##         title: UKCP18 land projections - 12km regional climate model, mean air temperature at 1.5m (°c) over the UK for the RCP 8.5 scenario
    ##         version: v20190731
    ##         Conventions: CF-1.5

print the objects in ncin (the netCDF file that has been read in).

``` r
var_names <- unlist(attributes(ncin$var))

print(unname(var_names))  # uname removes the name attributes in R, so just the names of the objects are printed
```

    ##  [1] "tas"                          "transverse_mercator"         
    ##  [3] "time_bnds"                    "projection_y_coordinate_bnds"
    ##  [5] "projection_x_coordinate_bnds" "ensemble_member_id"          
    ##  [7] "grid_latitude"                "grid_longitude"              
    ##  [9] "month_number"                 "year"                        
    ## [11] "yyyymm"

Read temperature data in from the netCDF file

``` r
temp_data <- ncvar_get(ncin, varid = "tas")

print("Dimension of the temperature array")
```

    ## [1] "Dimension of the temperature array"

``` r
dim(temp_data)
```

    ## [1]   82  112 1200

``` r
cat("\n First 10 observations in temp_data \n")
```

    ## 
    ##  First 10 observations in temp_data

``` r
head(temp_data)
```

    ## [1] 10.57284 10.57409 10.57399 10.56402 10.54636 10.52990

Check what type of calendar the climate model is using

``` r
cat("Number of days in model year is",ncin$dim$time$calendar)
```

    ## Number of days in model year is 360_day

As this is a non-standard calendar, our years only having 360 days, we
need to use specialised functions to read in the calendar data. Such
functions are available in the library ncdf4.helpers package, you will
also need the PCICt library to use this package

``` r
temp_time <- nc.get.time.series(ncin, v = "tas")

str(temp_time)
```

    ##  'PCICt' num [1:1200(1d)] 1980-12-16 1981-01-16 1981-02-16 1981-03-16 ...
    ##  - attr(*, "cal")= chr "360"
    ##  - attr(*, "months")= num [1:12] 30 30 30 30 30 30 30 30 30 30 ...
    ##  - attr(*, "dpy")= num 360
    ##  - attr(*, "tzone")= chr "GMT"
    ##  - attr(*, "units")= chr "secs"

The mid-point of the month is printed out, and each month has 30 days.
There are 1200 months in total.

Print out the first 6 and last 6
    times

``` r
head(temp_time)
```

    ## [1] "1980-12-16" "1981-01-16" "1981-02-16" "1981-03-16" "1981-04-16"
    ## [6] "1981-05-16"

``` r
tail(temp_time)
```

    ## [1] "2080-06-16" "2080-07-16" "2080-08-16" "2080-09-16" "2080-10-16"
    ## [6] "2080-11-16"

Climate years start in Dec, to coincide with the start of meteorological
winter (Dec, Jan , Feb) and end in Nov, the end of meteorological autumn
(Sep, Oct, Nov). The climate year 1981 consists of Dec 1980 and all the
months in 1981 except Dec, which will start the next year. The data
starts in Dec 1980 and ends in Nov 2080, so we have 1200 months in total

## Calculate decadal means

Split the temp data into decades - each decade will contain 10 x 12
months, starting in Dec 2080 and ending Nov 2080.

``` r
# Calculate the month mean for the whole tegion
year_mean <- apply(temp_data, 3, mean)

# Define which month belongs to which decade
decade <- rep(1:10, each = 120)

#Calculate mean for each decade
decade_mean <- tapply(X = year_mean, INDEX = decade, FUN = mean)
cat("Mean for each decade \n", decade_mean)
```

    ## Mean for each decade 
    ##  9.502062 9.658127 9.977906 10.4065 11.0167 11.32755 11.75401 12.27232 12.45431 12.76532

``` r
mid_decade <- seq(from = 1985, to = 2075, by = 10)
plot(mid_decade, decade_mean, type = "l")
```

![](example_2_ukcp_with_R_files/figure-gfm/Calculate%20decadal%20means-1.png)<!-- -->

## Plot a smoothed time series of monthly means

Plot a smoothed time series using a 12-month running mean over a plot of
the monthly temperatures

``` r
month_mean <- apply(temp_data, 3, mean)

roll_year_mean <- rollapply(month_mean, 12, mean)

plot(month_mean, col = "blue", type = "l")
lines(roll_year_mean, type = "l", col = "black", lwd = 2)
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## Plotting maps of the data

Make a simple spatial plot of the first month in the data. (There are
probably many different ways in R that this can be done. The method
shown below uses rasters, I expect these plots could also be produced
using ggplot or other graphical libraries in R)

Read netCDF file into R as a raster brick - a multi-layer raster object.
Here the layers are the months. plot the first month

``` r
temp1 <- brick(ukcp_file, varname = "tas")

str(temp1)
```

    ## Formal class 'RasterBrick' [package "raster"] with 12 slots
    ##   ..@ file    :Formal class '.RasterFile' [package "raster"] with 13 slots
    ##   .. .. ..@ name        : chr "/net/spice/project/ukcp/land-rcm/uk/12km/rcp85/01/tas/mon/v20190731/tas_rcp85_land-rcm_uk_12km_01_mon_198012-208011.nc"
    ##   .. .. ..@ datanotation: chr "FLT4S"
    ##   .. .. ..@ byteorder   : chr "little"
    ##   .. .. ..@ nodatavalue : num 1e+20
    ##   .. .. ..@ NAchanged   : logi FALSE
    ##   .. .. ..@ nbands      : int 1200
    ##   .. .. ..@ bandorder   : chr "BIL"
    ##   .. .. ..@ offset      : int 0
    ##   .. .. ..@ toptobottom : logi TRUE
    ##   .. .. ..@ blockrows   : int 0
    ##   .. .. ..@ blockcols   : int 0
    ##   .. .. ..@ driver      : chr "netcdf"
    ##   .. .. ..@ open        : logi FALSE
    ##   ..@ data    :Formal class '.MultipleRasterData' [package "raster"] with 14 slots
    ##   .. .. ..@ values    : logi[0 , 0 ] 
    ##   .. .. ..@ offset    : num 0
    ##   .. .. ..@ gain      : num 1
    ##   .. .. ..@ inmemory  : logi FALSE
    ##   .. .. ..@ fromdisk  : logi TRUE
    ##   .. .. ..@ nlayers   : int 1200
    ##   .. .. ..@ dropped   : NULL
    ##   .. .. ..@ isfactor  : logi FALSE
    ##   .. .. ..@ attributes: list()
    ##   .. .. ..@ haveminmax: logi FALSE
    ##   .. .. ..@ min       : num [1:1200] Inf Inf Inf Inf Inf ...
    ##   .. .. ..@ max       : num [1:1200] -Inf -Inf -Inf -Inf -Inf ...
    ##   .. .. ..@ unit      : chr "degC"
    ##   .. .. ..@ names     : chr [1:1200] "X1980.10.20.00.00.00" "X1980.11.18.23.00.00" "X1980.12.18.23.00.00" "X1981.01.17.23.00.00" ...
    ##   ..@ legend  :Formal class '.RasterLegend' [package "raster"] with 5 slots
    ##   .. .. ..@ type      : chr(0) 
    ##   .. .. ..@ values    : logi(0) 
    ##   .. .. ..@ color     : logi(0) 
    ##   .. .. ..@ names     : logi(0) 
    ##   .. .. ..@ colortable: logi(0) 
    ##   ..@ title   : chr "Mean air temperature"
    ##   ..@ extent  :Formal class 'Extent' [package "raster"] with 4 slots
    ##   .. .. ..@ xmin: num -216000
    ##   .. .. ..@ xmax: num 768000
    ##   .. .. ..@ ymin: num -108000
    ##   .. .. ..@ ymax: num 1236000
    ##   ..@ rotated : logi FALSE
    ##   ..@ rotation:Formal class '.Rotation' [package "raster"] with 2 slots
    ##   .. .. ..@ geotrans: num(0) 
    ##   .. .. ..@ transfun:function ()  
    ##   ..@ ncols   : int 82
    ##   ..@ nrows   : int 112
    ##   ..@ crs     :Formal class 'CRS' [package "sp"] with 1 slot
    ##   .. .. ..@ projargs: chr "+proj=tmerc +pm=0 +a=6377563.396 +b=6356256.909 +lon_0=-2 +lat_0=49 +x_0=4e+05 +y_0=-1e+05 +k_0=0.9996012717"
    ##   ..@ history : list()
    ##   ..@ z       :List of 1
    ##   .. ..$ Date/time: chr [1:1200] "1980-10-20 00:00:00" "1980-11-18 23:00:00" "1980-12-18 23:00:00" "1981-01-17 23:00:00" ...

``` r
plot(temp1,1)
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
Change the colour scheme I like viridis - meant to be good for most
forms of colour blindness and good for printing out in black and
white.

``` r
plot(temp1, 1, col = viridis(256))
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Mask out the sea data

The land sea mask represents the sea as a missing value (NA)

``` r
# Read in the land seas mask as a aster file
mask_lsm <- raster(ukcp_file_land_mask, varname = "lsm")


land <- mask(temp1, mask = is.na(mask_lsm), maskvalue = T)

plot(land, 1, col = viridis(256)) 
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->
Add in a coastline for the UK. The added complication is that the
temperature data uses a transverse Mercator grid One approach is to
project the UK coastline on to the transverse Mercator grid.

``` r
# Select out parts of the world map that we want

uk_coast <- map("world",
                c("UK", "Ireland", "Jersey", "Guernsey", "Isle of Man"),
                xlim = c(-11, 3),
                ylim = c(49, 60.9),
                fill = T,
                col = "transparent",
                plot = F)
  
ids <- sapply(strsplit(uk_coast$names, ":"), function(x) x[1])

# convert the coastlines a spatial polygon
  
uk_cst_sp <- map2SpatialPolygons(uk_coast, IDs = ids, proj4string =
                                     CRS("+proj=longlat +datum=WGS84"))

# Transform the spatial polygon from standard lat/lon projection to
# the projection used for the land sea mask.
uk_utm <- spTransform(uk_cst_sp, projection(land))

# plot the temperatures then add the coast outline

plot(land, 1, col = viridis(256))

plot(uk_utm, add = T)
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Tidy up the appearance of the
graph

``` r
plot(land, 1, col = viridis(256), horizontal = T, main = "Plot title her deg C",
    legend.args = list(text = "Legend title"))

plot(uk_utm, add = T)
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

Working with daily files

1)  load up new daily
data

<!-- end list -->

``` r

temp_ds <- list.files(path = ukcp_file_directory_tasmax, pattern =    '^tasmax_rcp26_land-gcm_uk_60km_04_day')

temp_ds <- temp_ds[9:15]

print(temp_ds)
```

    ## [1] "tasmax_rcp26_land-gcm_uk_60km_04_day_19791201-19891130.nc"
    ## [2] "tasmax_rcp26_land-gcm_uk_60km_04_day_19891201-19991130.nc"
    ## [3] "tasmax_rcp26_land-gcm_uk_60km_04_day_19991201-20091130.nc"
    ## [4] "tasmax_rcp26_land-gcm_uk_60km_04_day_20091201-20191130.nc"
    ## [5] "tasmax_rcp26_land-gcm_uk_60km_04_day_20191201-20291130.nc"
    ## [6] "tasmax_rcp26_land-gcm_uk_60km_04_day_20291201-20391130.nc"
    ## [7] "tasmax_rcp26_land-gcm_uk_60km_04_day_20391201-20491130.nc"

``` r
temp_rcp26 <- NULL


for (itemp in paste(dirname, temp_ds, sep = "/")) {
  nc_rcp26 <- nc_open(itemp)
  
  var_data <- ncvar_get(nc_rcp26, varid = "tasmax")
  temp_rcp26 <- abind(temp_rcp26, var_data, along = 3)
}  

dim(temp_rcp26)
```

    ## [1]    17    23 25200

Subset the data to the first 70 years, 360 days in a year ,from 01 Dec
1979 to 30 Nov 2049 Also subset the data to the baseline period of Dec
1980 to Nov 2000

``` r
temp_rcp26 <- temp_rcp26[, , 1:(70 * 360)]
baseline <- temp_rcp26[, , (720 + 1):(720 + 20 * 360)]
```

Calculate the spatial monthly mean monthly values for the baseline
period

``` r
spatial_daily_mean <- apply(baseline, 3, mean)

mon_ind_bsl <- rep(rep(1:12, each = 30), 20)

basemonth_mean <- tapply(X = spatial_daily_mean, INDEX = mon_ind_bsl, FUN = mean)
```

Calculate the spatial monthly means for the whole period of interest

``` r
spatial_daily_mean_wp <- apply(temp_rcp26, 3, mean)

mon_ind_wp <- rep(rep(1:(12*70), each = 30))

wp_month_mean <- tapply(X = spatial_daily_mean_wp, INDEX = mon_ind_wp, FUN = mean)
```

Calculate the monthly anomalies and the annual anomalies derived from
the monthly anomalies

``` r
anom_mon <- wp_month_mean - rep(basemonth_mean, 70)

anom_year <- tapply(anom_mon, INDEX = rep(c(1:70),each = 12), FUN = mean)
```

Plot the
result

``` r
plot(wp_month_mean, type = "l", col = "#29AF7FFF",lwd = 1.5, ylim = c(-5,25))
lines(anom_mon, type = "l",col = "#95D840FF", lwd = 2)
lines(seq(from = 6, to = 840, by = 12), anom_year, type = "l", col = "#440154FF", lwd = 3)
```

![](example_2_ukcp_with_R_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->
