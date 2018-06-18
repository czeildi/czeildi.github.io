---
title: Why shouldn't you use external data within R package
author: Ildi Czeller
date: '2018-06-16'
slug: why-shouldn-t-you-use-external-data-within-r-package
categories: ["R"]
tags: ["pkg"]
---

*Author: Ildi Czeller, @czeildi on Twitter and Github*

## TL;DR

Save all datasets you refer to from your functions within an R package as internal data.

## what I learned

In an R package you might have a dataset that you want to use in your functions and want to publicize as well as external dataset. In this case you should save this dataset as **both** internal and external data with

```r
devtools::use_data(external_and_internal)
devtools::use_data(external_and_internal, internal = TRUE)
```

(More details in Hadley's [book](http://r-pkgs.had.co.nz/data.html) on R packages.)

If you only save your data as external data:

You get a `R CMD check` note about `undefined global variables: 'external_only'`. This you can eliminate with 
```r 
utils::globalVariables("external_only")
``` 
but then you only conceal the problem from yourself.

If your package is loaded, everything looks fine but if someone wants to use your function without loading your package: 
```r 
datasetsexperiment::print_external_in_body()
```
it will fail with `object 'external_only' not found`. This error will be probably frustrating as your user can see the data just fine with 
```r 
datasetsexperiment::external_only
```

I came across this scenario last week and I am really lucky that somehow I suspected that the dataset should be saved as internal data. Otherwise I honestly do not know how long would the debugging have been.

Previously I always loaded the package so it was strange to suddenly see it not working. Also running your testthat tests and `devtools::load_all()` both load external datasets so you won't notice the problem during development.

## detailed results

The exact scenario after `devtools::install()`:

``` r
datasetsexperiment::print_internal_in_body()
#> [1] "piece of data saved as both internal and external"
datasetsexperiment::print_external_in_body()
#> Error in datasetsexperiment::print_external_in_body(): object 'external_only' not found
datasetsexperiment::external_only
#> [1] "piece of external only data"

library("datasetsexperiment")

datasetsexperiment::print_internal_in_body()
#> [1] "piece of data saved as both internal and external"
datasetsexperiment::print_external_in_body()
#> [1] "piece of external only data"
datasetsexperiment::external_only
#> [1] "piece of external only data"
```


## remark

In the repository you can see an experiment where the used data is in a function argument or in the body. This is because I remember seeing one of them work and the other not but I could not reproduce this scenario.

## material

Code (R code + Dockerfile) is available on [GitHub](https://github.com/czeildi/datasetsexperiment).

