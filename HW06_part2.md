STAT 547M Homework 6
================
Junbin ZHANG
Oct 30, 2018

# Bring rectangular data in

``` r
## load gapminder
suppressPackageStartupMessages(library(gapminder))
## load tidyverse
suppressPackageStartupMessages(library(tidyverse))
## load testthat
suppressPackageStartupMessages(library(testthat))
## load stringi
suppressPackageStartupMessages(library(stringi))
## load leaflet
suppressPackageStartupMessages(library(leaflet))
```

# Install and load `singer` package

``` r
## install singer
# install.packages("devtools")
#devtools::install_github("JoeyBernhardt/singer")
## load singer
suppressPackageStartupMessages(library(singer))
```

# Install and load `ggmap` package

``` r
## install ggmap
# install.packages("devtools")
# devtools::install_github("dkahle/ggmap", ref = "tidyup")
## load ggmap
suppressPackageStartupMessages(library(ggmap))
## register API key (please do not use this key for any other purpose, thank you)
register_google(key = "AIzaSyDJnU2gOUgrDiIOSdujwanq9yX2yqRVlu4")
```

## Work with the`singer`data

Let’s first have a preview of `single_locations` dataframe, in order to
know why it is “dirty”.

``` r
# show previews of single_locations
head(singer_locations) %>% 
  # select columns we care about
  select(track_id, latitude, longitude, name, city) %>% 
  knitr::kable(col.names = c("Track ID", "Latitude", "Longitude", "Name", "City"))
```

| Track ID           | Latitude |  Longitude | Name          | City         |
| :----------------- | -------: | ---------: | :------------ | :----------- |
| TRWICRA128F42368DB |       NA |         NA | NA            | NA           |
| TRXJANY128F42246FC | 41.88415 | \-87.63241 | Gene Chandler | Chicago, IL  |
| TRIKPCA128F424A553 | 40.71455 | \-74.00712 | Paul Horn     | New York, NY |
| TRYEATD128F92F87C9 |       NA |         NA | NA            | NA           |
| TRBYYXH128F4264585 | 42.33168 | \-83.04792 | Dorothy Ashby | Detroit, MI  |
| TRKFFKR128F9303AE3 | 40.99471 | \-77.60454 | Barleyjuice   | Pennsylvania |

``` r
tail(singer_locations) %>% 
  # select columns we care about
  select(track_id, latitude, longitude, name, city) %>% 
  knitr::kable(col.names = c("Track ID", "Latitude", "Longitude", "Name", "City"))
```

| Track ID           | Latitude |  Longitude | Name          | City            |
| :----------------- | -------: | ---------: | :------------ | :-------------- |
| TRNHZEE128F425DC25 | 32.67828 | \-83.22295 | T-Rock        | Georgia         |
| TRNBOMT128F92FF4E0 | 30.76753 | \-92.11789 | Lonnie Brooks | Dubuisson, LA   |
| TRCVRSU128EF34A53C | 53.34376 |  \-6.24953 | The Bachelors | Dublin, Ireland |
| TRHSWOW128F4223266 |       NA |         NA | NA            | NA              |
| TRCEMNR128F92E0379 |       NA |         NA | NA            | NA              |
| TRLCVCR128F9349A49 |       NA |         NA | NA            | NA              |

So the dataframe is “dirty” because it contains `NA` values. Therefore,
the first thing we need to do is to filter out rows with `NA` latitude
or longitude.

``` r
# filter out latitude and longitude with NA
singer_locations_no_na <- singer_locations %>% 
  filter(!is.na(latitude) | !is.na(longitude))
# show previews of filtered dataframe
head(singer_locations_no_na) %>% 
  # select columns we care about
  select(track_id, latitude, longitude, name, city) %>% 
  knitr::kable(col.names = c("Track ID", "Latitude", "Longitude", "Name", "City"))
```

| Track ID           | Latitude |   Longitude | Name                     | City         |
| :----------------- | -------: | ----------: | :----------------------- | :----------- |
| TRXJANY128F42246FC | 41.88415 |  \-87.63241 | Gene Chandler            | Chicago, IL  |
| TRIKPCA128F424A553 | 40.71455 |  \-74.00712 | Paul Horn                | New York, NY |
| TRBYYXH128F4264585 | 42.33168 |  \-83.04792 | Dorothy Ashby            | Detroit, MI  |
| TRKFFKR128F9303AE3 | 40.99471 |  \-77.60454 | Barleyjuice              | Pennsylvania |
| TRWKTVW12903CE5ACF | 34.20034 | \-119.18044 | Madlib                   | Oxnard, CA   |
| TRUWFXF128E0795D22 | 50.73230 |     7.10169 | Seeed feat. Elephant Man | Bonn         |

``` r
tail(singer_locations_no_na) %>% 
  # select columns we care about
  select(track_id, latitude, longitude, name, city) %>% 
  knitr::kable(col.names = c("Track ID", "Latitude", "Longitude", "Name", "City"))
```

| Track ID           |   Latitude |   Longitude | Name          | City            |
| :----------------- | ---------: | ----------: | :------------ | :-------------- |
| TROAIKU128F92F87F3 |   51.50632 |   \-0.12714 | DJ Vix        | London          |
| TRXSDQL128F931DA39 |   31.16890 | \-100.07715 | E.S.G.        | Texas           |
| TRTQBDM128F4263652 | \-14.24292 |  \-54.38783 | Hardrive      | Brazil          |
| TRNHZEE128F425DC25 |   32.67828 |  \-83.22295 | T-Rock        | Georgia         |
| TRNBOMT128F92FF4E0 |   30.76753 |  \-92.11789 | Lonnie Brooks | Dubuisson, LA   |
| TRCVRSU128EF34A53C |   53.34376 |   \-6.24953 | The Bachelors | Dublin, Ireland |

In the next step, we use `map2()` (since we need to input both latitude
and longitude) of `purrr` and `revgeocode()` of `ggmap` to get the
cities. Her we use `revgeocode()` as a **formula** instead of a
**function** inside `map2()`. According to [this
reference](https://www.rdocumentation.org/packages/purrr/versions/0.2.5/topics/map2),
we need to add “\~” before the function name, and use “.x” and “.y” to
represent those two arguments.

In addtion, since there are more than 4,000 rows in
`singer_locations_no_na`, and it requires billing on Google Clouds
Services, we only use the first 30 rows as an example. However, this
solution can be easily extended to handle the whole dataframe.

``` r
# trim singer_locations_no_na for the top 100 rows
trim_locations <- singer_locations_no_na %>% 
  slice(1:20)
# use ggmap to get locations
ggmap_locations <- map2_chr(trim_locations$longitude, trim_locations$latitude, ~ revgeocode(as.numeric(c(.x, .y))))
```

Here we apply `map2_chr()` to make the results as a vector of strings
instead of a list to get rid of unnecessary information.

``` r
# show the final results
cbind(trim_locations$city, ggmap_locations) %>% 
  knitr::kable(col.names = c("City in `singer_locations`", "City Obtained by `ggmap`"))
```

| City in `singer_locations`   | City Obtained by `ggmap`                                 |
| :--------------------------- | :------------------------------------------------------- |
| Chicago, IL                  | 134 N LaSalle St suite 1720, Chicago, IL 60602, USA      |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  |
| Detroit, MI                  | 1001 Woodward Ave, Detroit, MI 48226, USA                |
| Pennsylvania                 | Z. H. Confair Memorial Hwy, Howard, PA 16841, USA        |
| Oxnard, CA                   | 300 W 3rd St, Oxnard, CA 93030, USA                      |
| Bonn                         | Regina-Pacis-Weg 1, 53113 Bonn, Germany                  |
| Hawaii                       | Unnamed Road, Hawaii, USA                                |
| Los Angeles, CA              | 1420 S Oakhurst Dr, Los Angeles, CA 90035, USA           |
| Staten Island, NY            | 215 Arthur Kill Rd, Staten Island, NY 10306, USA         |
| Portland, OR                 | 1500 SW 1st Ave, Portland, OR 97201, USA                 |
| UK - England - London        | 39 Whitehall, Westminster, London SW1A 2BY, UK           |
| Poggio Bustone, Rieti, Italy | Localita’ Pescatore, Poggio Bustone, RI 02018, Italy     |
| Pittsburgh, PA               | 410 Grant St, Pittsburgh, PA 15219, USA                  |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  |
| New York, NY                 | 1 Dr Carlton B Goodlett Pl, San Francisco, CA 94102, USA |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  |
| Los Angeles, CA              | 1420 S Oakhurst Dr, Los Angeles, CA 90035, USA           |
| California                   | Shaver Lake, CA 93634, USA                               |
| Panama                       | Calle Aviacion, Río Hato, Panama                         |
| KENT, WASHINGTON             | 220 4th Ave S, Kent, WA 98032, USA                       |

## Task 2.2

**Try to check wether the place in `city` corresponds to the information
you retrieved.**

Having a closer look at the above table, there is another point that
make `city` dirty: the way to name a city are not the same accoss the
column, for example, “UK - England - London” is a weired expression.

However, we can perform some preprocessing on the data to finish this
task:

  - We first extract words from every string, and convert them into
    lower case for further processing.
  - Based on some knowledge on the states of America, we replace all
    full state names into their abbreviations.

In the end, we try to find the intersection of the original `city` and
the ones obtained from `ggmap`. If there is at least one match for a
row, then we get the right result.

``` r
# create pattern for replacement
patterns <- c("new york" = "ny", "pennsylvania" = "pa", "california" = "ca", "washington" = "wa", "louisiana" = "la")
## preprocessing
ori_words <- trim_locations$city %>%
  # convert to lower case
  str_to_lower() %>% 
  # split into words
  str_split(pattern = boundary("word")) %>%
  # replace state names
  map(str_replace_all, patterns)
ggmap_words <- ggmap_locations %>% 
  # convert to lower case
  str_to_lower() %>% 
  # split into words
  str_split(pattern = boundary("word")) %>%
  # replace state names
  map(str_replace_all, patterns)
## find intersection and check if there are at least one match
correct <- map2(ori_words, ggmap_words, ~intersect(.x, .y)) %>% 
  map(function(l) {
    return(length(l) >= 1)
  })
# show results
cbind(trim_locations$city, ggmap_locations, correct) %>% 
  knitr::kable(col.names = c("City in `singer_locations`", "City Obtained by `ggmap`", "Correct?"))
```

| City in `singer_locations`   | City Obtained by `ggmap`                                 | Correct? |
| :--------------------------- | :------------------------------------------------------- | :------- |
| Chicago, IL                  | 134 N LaSalle St suite 1720, Chicago, IL 60602, USA      | TRUE     |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  | TRUE     |
| Detroit, MI                  | 1001 Woodward Ave, Detroit, MI 48226, USA                | TRUE     |
| Pennsylvania                 | Z. H. Confair Memorial Hwy, Howard, PA 16841, USA        | TRUE     |
| Oxnard, CA                   | 300 W 3rd St, Oxnard, CA 93030, USA                      | TRUE     |
| Bonn                         | Regina-Pacis-Weg 1, 53113 Bonn, Germany                  | TRUE     |
| Hawaii                       | Unnamed Road, Hawaii, USA                                | TRUE     |
| Los Angeles, CA              | 1420 S Oakhurst Dr, Los Angeles, CA 90035, USA           | TRUE     |
| Staten Island, NY            | 215 Arthur Kill Rd, Staten Island, NY 10306, USA         | TRUE     |
| Portland, OR                 | 1500 SW 1st Ave, Portland, OR 97201, USA                 | TRUE     |
| UK - England - London        | 39 Whitehall, Westminster, London SW1A 2BY, UK           | TRUE     |
| Poggio Bustone, Rieti, Italy | Localita’ Pescatore, Poggio Bustone, RI 02018, Italy     | TRUE     |
| Pittsburgh, PA               | 410 Grant St, Pittsburgh, PA 15219, USA                  | TRUE     |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  | TRUE     |
| New York, NY                 | 1 Dr Carlton B Goodlett Pl, San Francisco, CA 94102, USA | FALSE    |
| New York, NY                 | 80 Chambers St, New York, NY 10007, USA                  | TRUE     |
| Los Angeles, CA              | 1420 S Oakhurst Dr, Los Angeles, CA 90035, USA           | TRUE     |
| California                   | Shaver Lake, CA 93634, USA                               | TRUE     |
| Panama                       | Calle Aviacion, Río Hato, Panama                         | TRUE     |
| KENT, WASHINGTON             | 220 4th Ave S, Kent, WA 98032, USA                       | TRUE     |

So there is only one incorrect result at line 15.

``` r
# print incorrect results
cbind(trim_locations$city, ggmap_locations, correct) %>% 
  # convert to tibble
  as_tibble() %>% 
  # filter to have incorrect results
  filter(correct == FALSE) %>% 
  # select out column correct
  select(-correct) %>% 
  # show table
  knitr::kable(caption = "Incorrect Result", col.names = c("City in `singer_locations`", "City Obtained by `ggmap`"))
```

| City in `singer_locations` | City Obtained by `ggmap`                                 |
| :------------------------- | :------------------------------------------------------- |
| New York, NY               | 1 Dr Carlton B Goodlett Pl, San Francisco, CA 94102, USA |

Incorrect Result

## Task 2.3

**If you still have time, you can go visual.**

``` r
# use leaflet to plot information
#singer_locations_no_na %>% 
#  leaflet() %>% 
#  addTiles() %>% 
#  addCircles(
#    lat = singer_locations_no_na$latitude,
#    lng = singer_locations_no_na$longitude,
#    # here the popup contains city, title, artist name and year
#    popup = str_c(singer_locations_no_na$city, ": ", singer_locations_no_na$title, " by ", singer_locations_no_na$artist_name, " in ", #singer_locations_no_na$year)
#  ) %>% 
#  addProviderTiles(providers$OpenStreetMap)
```
