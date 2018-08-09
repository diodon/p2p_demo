Join tables
================
Eduardo Klein. <eklein@usb.ve>
created 2018-08-09

-   [Join matched records](#join-matched-records)

Last run 2018-08-08 20:43:32 UTM

Join matched records
--------------------

Now you have the matched records with WoRMS, you need to join the results to your existing table. This script will add the WoRMS information for each taxa to your original table using the scientific name as index.

We will use `dplyr::join_left` to perform the join

``` r
require(readr)
```

    ## Loading required package: readr

``` r
require(dplyr)
```

    ## Loading required package: dplyr

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
## I'll use Venezuela as case study here
## read the data
filename = "Venezuela"
SARCE.clean <- read_csv(file=paste0("data/SARCE/", filename, "_clean.csv"))
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   Year = col_integer(),
    ##   Months = col_integer(),
    ##   Replicate = col_integer(),
    ##   BioR = col_integer(),
    ##   Latitude = col_double(),
    ##   occurrence = col_integer()
    ## )

    ## See spec(...) for full column specifications.

``` r
## read the matched data. WoRMS returns a table with variables separated by tabs
## 
taxaWoRMS = read.csv(file="data/SARCE/SARCE_taxa_matched.csv", sep="\t")
```

So in both tables we have the variable `scientificName` but in WoRMS is returned with capital S and capital N (`ScientificName`) and we will use this for the join

``` r
SARCE.matched = left_join(SARCE.clean, taxaWoRMS, by=c("scientificName" = "ScientificName"))
```

    ## Warning: Column `scientificName`/`ScientificName` joining character vector
    ## and factor, coercing into character vector
