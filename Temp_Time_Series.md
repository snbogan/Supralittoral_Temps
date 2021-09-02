Temp\_Time\_Series
================
Sam Bogan
6/23/2021

This is an R Markdown written by Sam Bogan that reads in, wrangles, summarizes, and analyzes time series temperatures collected from n = 3 per site supralittoral splash pools occupied by *Tigriopus californicus* across 4 sites in coastal California (see README). Data were recorded by TidBit MX5000 temperature loggers made by Onset Computer Corp between 2019 - 2021.

# Read and Wrangle Raw Data

``` r
# Load packages
library( tidyverse )
```

    ## Warning: package 'tidyverse' was built under R version 3.6.2

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.2     ✓ dplyr   1.0.6
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## Warning: package 'ggplot2' was built under R version 3.6.2

    ## Warning: package 'tibble' was built under R version 3.6.2

    ## Warning: package 'tidyr' was built under R version 3.6.2

    ## Warning: package 'readr' was built under R version 3.6.2

    ## Warning: package 'purrr' was built under R version 3.6.2

    ## Warning: package 'dplyr' was built under R version 3.6.2

    ## Warning: package 'forcats' was built under R version 3.6.2

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library( Rmisc )
```

    ## Loading required package: lattice

    ## Warning: package 'lattice' was built under R version 3.6.2

    ## Loading required package: plyr

    ## ------------------------------------------------------------------------------

    ## You have loaded plyr after dplyr - this is likely to cause problems.
    ## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    ## library(plyr); library(dplyr)

    ## ------------------------------------------------------------------------------

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

    ## The following object is masked from 'package:purrr':
    ## 
    ##     compact

``` r
library( lubridate )
```

    ## Warning: package 'lubridate' was built under R version 3.6.2

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library( data.table )
```

    ## Warning: package 'data.table' was built under R version 3.6.2

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:lubridate':
    ## 
    ##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
    ##     yday, year

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

``` r
# Move to input data directory for tempm time series
setwd( "~/Documents/GitHub/HotOnes_Tigriopus/Temp_Time_Series/Input_files/" )

# Get the files names
temps <- list.files( pattern = "*0.csv" )

# Read in .csv files listed in temps
mytemps <- lapply( temps, read.csv )

# Correct df names from mytemps
names( mytemps ) <- gsub( " .*",
                          "",
                          temps )

# Remove unecessary columns and standardize column number
mytemps <- lapply( mytemps, function ( y ) { y <- select( y, c( 1, 2 ) ) } )

# Combine all temp dfs
all_temps <- rbindlist( mytemps, 
                        idcol = "Logger" )
```

    ## Column 1 ['Date.Time..GMT..0700'] of item 4 is missing in item 1. Use fill=TRUE to fill with NA (NULL for list columns), or use.names=FALSE to ignore column names. use.names='check' (default from v1.12.2) emits this message and proceeds as if use.names=FALSE for  backwards compatibility. See news item 5 in v1.12.2 for options to control this message.

``` r
# Remove rows where temp is empty
all_temps <- all_temps[ !(is.na( all_temps$Temp....F. ) | 
                            all_temps$Temp....F.  == "" ), ]

# Create site variable
all_temps$Site <- gsub( "[1-9]+", 
                        "", 
                        all_temps$Logger )

# Fix column names
names( all_temps )[ names( all_temps ) == "Date.Time..GMT..0800"] <- "Date_Time"
names( all_temps )[ names( all_temps ) == "Temp....F."] <- "Temp"

# Convert temp to celcius
all_temps$Temp <- round( ( ( all_temps$Temp - 32 ) * ( 5 / 9 ) ),
                         digits = 2 )
```
