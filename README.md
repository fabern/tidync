
<!-- README.md is generated from README.Rmd. Please edit that file -->

# tidync

[![Travis-CI Build
Status](http://badges.herokuapp.com/travis/hypertidy/tidync?branch=master&env=BUILD_NAME=trusty_release&label=linux)](https://travis-ci.org/hypertidy/tidync)[![Build
Status](http://badges.herokuapp.com/travis/hypertidy/tidync?branch=master&env=BUILD_NAME=osx_release&label=osx)](https://travis-ci.org/hypertidy/tidync)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/hypertidy/tidync?branch=master&svg=true)](https://ci.appveyor.com/project/mdsumner/tidync-3m5h7)
[![Coverage
status](https://codecov.io/gh/hypertidy/tidync/branch/master/graph/badge.svg)](https://codecov.io/github/hypertidy/tidync?branch=master)[![](https://badges.ropensci.org/174_status.svg)](https://github.com/ropensci/onboarding/issues/174)[![lifecycle](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/tidync)](https://cran.r-project.org/package=tidync)

The goal of tidync is to ease exploring the contents of a NetCDF source
and constructing efficient queries to extract arbitrary hyperslabs. The
data extracted can be used directly as an array, or in long-form data
frame. In contrast to other packages tidync helps reduce the volume of
code required to discover and read the contents of NetCDF, with simple
steps:

  - Connect and summarize `tidync()`.
  - (optionally) Specify source variables `activate()`.
  - (optionally) Specify array sub-setting (slicing) `hyper_filter()`.
  - Read array data in either native `hyper_array()` or long-form
    `hyper_tibble()`.

NetCDF is **Network Common Data Form** a very common, and very general
way to store and work with scientific array-based data. NetCDF is
defined and provided by
[Unidata](https://www.unidata.ucar.edu/software/netcdf/). R has
(independent) support for NetCDF via the
[RNetCDF](https://CRAN.R-project.org/package=RNetCDF),
[ncdf4](https://CRAN.R-project.org/package=ncdf4),
[rhdf5](https://bioconductor.org/packages/release/bioc/html/rhdf5.html),
[stars](https://CRAN.R-project.org/package=stars),
[sf](https://CRAN.R-project.org/package=sf) and
[rgdal](https://CRAN.R-project.org/package=rgdal) packages.

This project uses RNetCDF for the primary access to the NetCDF library.

## Installation

You can install tidync from github with:

``` r
# install.packages("remotes")
remotes::install_github("hypertidy/tidync", dependencies = TRUE)
```

Also required are packages ncdf4 and RNetCDF, so first make sure you can
install and use these.

``` r
install.packages("ncdf4")
install.packages("RNetCDF")
```

If you have problems, please see the [INSTALL instructions for
RNetCDF](https://CRAN.R-project.org/package=RNetCDF/INSTALL), these
should work as well for ncdf4. Below I note specifics for different
operating systems, notably Ubuntu/Debian where I work the most - these
aren’t comprehensive details but might be helpful.

### Windows

On Windows, everything should be easy as ncdf4 and RNetCDF are supported
by CRAN. The RNetCDF package now includes OpenDAP/Thredds for 64-bit
Windows (not 32-bit), and so tidync will work for those sources too.

### MacOS

On MacOS, it should also be easy as there are binaries for ncdf4 and
RNetCDF available on CRAN. As far as I know, these binaries won’t
support OpenDAP/Thredds.

### Ubuntu/Debian

On Linux you will need at least the following installed by an
administrator, here tested on Ubuntu Xenial 16.04. As far as I know
nothing extra above the libnetcdf-dev library is required for
OpenDAP/Thredds.

``` bash
apt update 
apt upgrade --assume-yes

## Install 3rd parties for NetCDF
apt install libnetcdf-dev libudunits2-dev

## install 3rd parties needed for devtools + openssl git2r httr
apt install libssl-dev
```

Then in R

``` r
install.packages("devtools")
devtools::install_github("hypertidy/tidync")
```

At the time of writing the [travis install
configuration](https://github.com/hypertidy/tidync/blob/master/.travis.yml)
was set up for “trusty”, Ubuntu 14.04 so also provides a record.

More general information about system dependencies libnetcdf-dev and
libudunits2-dev is available from [Unidata
NetCDF](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)
and [Unidata
Udunits2](https://www.unidata.ucar.edu/software/udunits/udunits-current/doc/udunits/udunits2.html#Installation).

## Usage

This is a basic example which shows how to connect to a
file.

``` r
file <- system.file("extdata", "oceandata", "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc", package = "tidync")
library(tidync)
tidync(file) 
#> 
#> Data Source (1): S20080012008031.L3m_MO_CHL_chlor_a_9km.nc ...
#> 
#> Grids (4) <dimension family> : <associated variables> 
#> 
#> [1]   D1,D0 : chlor_a    **ACTIVE GRID** ( 9331200  values per variable)
#> [2]   D3,D2 : palette
#> [3]   D0    : lat
#> [4]   D1    : lon
#> 
#> Dimensions 4 (2 active): 
#>   
#>   dim   name  length    min   max start count   dmin  dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>  <dbl> <dbl> <int> <int>  <dbl> <dbl> <lgl> <lgl>     
#> 1 D0    lat     2160 -180.  180.      1  2160 -180.  180.  FALSE TRUE      
#> 2 D1    lon     4320  -90.0  90.0     1  4320  -90.0  90.0 FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1   256 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1     3 FALSE FALSE
```

There are two main ways of using tidync, interactively to explore what
is there, and for extraction. The functions `tidync` and `activate` and
`hyper_filter` allow us to hone in on the part/s of the data we want,
and functions `hyper_array` and `hyper_tibble` give raw-array and data
frames with-full-coordinates forms respectively.

Also
[http://www.matteodefelice.name/research/2018/01/14/tidyverse-and-netcdfs-a-first-exploration/](this%20blog)
post by Matteo De Felice.

### Interactive

Use `tidync()` and `hyper_filter()` to discern what variables and
dimensions are available, and to craft axis-filtering expressions by
value or by index. (Use the name of the variable on the LHS to target
it, use its name to filter by value and the special name `index` to
filter it by its
index).

``` r
## discover the available entities, and the active grid's dimensions and variables
tidync(filename)

## activate a different grid
tidync(filename) %>% activate(grid_identifier)

## get a dimension-focus on the space occupied within a grid
tidync(filename) %>% hyper_filter()

## pass named expressions to subset dimension by value or index (step)
tidync(filename) %>% hyper_filter(lat = lat < -30, time = time == 20)
```

A grid is a “virtual table” in the sense of a database source. It’s
possible to activate a grid via a variable within it, so all variables
are available by default. Grids have identifiers based on which
dimensions they are defined with, so use i.e. “D1,D0” and can otherwise
be activated by their count identifier (starting at 1). The “D0” is an
identifier, it matches the internal 0-based indexing and identity used
by NetCDF itself.

### Extractive

Use what we learned interactively to extract the data, either in data
frame or raw-array (hyper slice)
form.

``` r
## we'll see a column for sst, lat, time, and whatever other dimensions sst has
## and whatever other variable's the grid has
tidync(filename) %>% activate("sst") %>% 
  hyper_filter(lat = lat < -30, time = time == 20) %>% 
  hyper_tibble()


## raw array form, we'll see a (list of) R arrays with a dimension for each seen by tidync(filename) %>% activate("sst"")
tidync(filename) %>% activate("sst") %>% 
  hyper_filter(lat = lat < -30, time = time == 20) %>% 
  hyper_array()
```

It’s important to not actual request the data extraction until the
expressions above would result in an efficient size (don’t try a data
frame version of a 20Gb ROMs variable …). Use the interactive modes to
determine the likely size of the output you will receive.

Functions seamlessly build the actual index values required by the
NetCDF library. This can be used to debug the process or to define your
own tools for the extraction. Currently each `hyper_*` function can take
the filtering expressions, but it’s not obvious if this is a good idea
or not.

See the vignettes for more:

``` r
vignette("tidync-examples")

browseVignettes(package = "tidync")
```

## Limitations

Please get in touch if you have specific workflows that `tidync` is not
providing. There’s a lot of room for improvement\!

  - we can’t do “grouped filters”" (i.e. polygon-overlay extraction),
    but it’s in the works
  - compound types are not supported, though see the “rhdf5” branch on
    Github
  - NetCDF groups are not exposed (groups are like a “files within a
    file”, analogous to a file system directory)

I’m interested in lighter and rawer access to the NetCDF library, I’ve
explored that here and it may or may not be a good idea:

<https://github.com/hypertidy/ncapi>

## Terminology

  - **slab**, **hyperslab** - array variable that may be read from a
    NetCDF
  - **shape**, **grid** - set of dimensions that define variables in
    NetCDF
  - **activation** - choice of a given grid to apply subsetting and read
    operations to

## Code of conduct

Please note that this project is released with a [Contributor Code of
Conduct](CONDUCT.md). By participating in this project you agree to
abide by its terms.
