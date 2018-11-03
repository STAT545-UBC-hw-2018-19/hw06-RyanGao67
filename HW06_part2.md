Work with the singer data
================
Jiahui Tang
2017/11/7

``` r
knitr::opts_chunk$set(warning = F)
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(singer))
suppressPackageStartupMessages(library(stringr))
suppressPackageStartupMessages(library(forcats))
suppressPackageStartupMessages(library(purrr))
suppressPackageStartupMessages(library(repurrrsive))
suppressPackageStartupMessages(library(ggmap))
suppressPackageStartupMessages(library(leaflet))
suppressPackageStartupMessages(library(mapview))
suppressPackageStartupMessages(library(htmlwidgets))
suppressPackageStartupMessages(library(webshot))
knitr::opts_chunk$set(fig.width=10, fig.height=5)
```

## Work with the`singer`data

  - The singer\_location dataframe in the singer package contains
    geographical information stored in two different formats: 1. as a
    (dirty\!) variable named city; 2. as a latitude / longitude pair
    (stored in latitude, longitude respectively). The function
    revgeocode from the ggmap library allows you to retrieve some
    information for a pair (vector) of longitude, latitude (warning:
    notice the order in which you need to pass lat and long). Read its
    manual page.

  - Use purrr to map latitude and longitude into human readable
    information on the band’s origin places. Notice that revgeocode(… ,
    output = “more”) outputs a dataframe, while revgeocode(… , output =
    “address”) returns a string: you have the option of dealing with
    nested dataframes. You will need to pay attention to two things:
    
      - Not all of the track have a latitude and longitude: what can we
        do with the missing information? (filtering, …)
      - Not all of the time we make a research through revgeocode() we
        get a result. What can we do to avoid those errors to bite us?
        (look at possibly() in purrr…)

*As for the track without a latitude and longitude, we need to filter
all rows with latitude and longitude are NAs. Then we apply map2() to
call revgeocode() for all pairs of longitude and latitude.*

*When revgeocode() goes wrong, there will be a error like
this:*

``` r
reverse geocode failed - bad location? location = "-87.63241"reverse geocode failed - bad location? location = "41.88415"
```

*we can use`possibly(revgeocode, NA)`to avoid this.*

``` r
#View(singer_locations)
#First we need to filter all rows with latitude and longitude are NAs.
filter_NA_locations <- singer_locations %>% 
  filter(!is.na(latitude) | !is.na(longitude))
filter_NA_locations <- head(filter_NA_locations,20)
#glimpse(filter_NA_locations)
#Then we use map2 to apply revgeocode
possibly(revgeocode, NA)
```

    ## function (...) 
    ## {
    ##     tryCatch(.f(...), error = function(e) {
    ##         if (!quiet) 
    ##             message("Error: ", e$message)
    ##         otherwise
    ##     }, interrupt = function(e) {
    ##         stop("Terminated by user", call. = FALSE)
    ##     })
    ## }
    ## <bytecode: 0x00000000222b06b0>
    ## <environment: 0x00000000222b3328>

``` r
loc <- map2(filter_NA_locations$longitude,
                                filter_NA_locations$latitude, 
                                ~ revgeocode(c(.x, .y)),possibly(revgeocode, NA))

locations <- cbind(location = loc,city = filter_NA_locations$city)

knitr::kable(locations)
```

| location           | city                         |
| :----------------- | :--------------------------- |
| list(address = NA) | Chicago, IL                  |
| list(address = NA) | New York, NY                 |
| list(address = NA) | Detroit, MI                  |
| list(address = NA) | Pennsylvania                 |
| list(address = NA) | Oxnard, CA                   |
| list(address = NA) | Bonn                         |
| list(address = NA) | Hawaii                       |
| list(address = NA) | Los Angeles, CA              |
| list(address = NA) | Staten Island, NY            |
| list(address = NA) | Portland, OR                 |
| list(address = NA) | UK - England - London        |
| list(address = NA) | Poggio Bustone, Rieti, Italy |
| list(address = NA) | Pittsburgh, PA               |
| list(address = NA) | New York, NY                 |
| list(address = NA) | New York, NY                 |
| list(address = NA) | New York, NY                 |
| list(address = NA) | Los Angeles, CA              |
| list(address = NA) | California                   |
| list(address = NA) | Panama                       |
| list(address = NA) | KENT, WASHINGTON             |
