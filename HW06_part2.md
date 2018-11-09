STAT 547M Homework 6
================
Tian Gao
Oct 30, 2018

# Bring rectangular data in

``` r
## load tidyverse
suppressPackageStartupMessages(library(tidyverse))
## load leaflet
suppressPackageStartupMessages(library(leaflet))
# install.packages("devtools")
#devtools::install_github("JoeyBernhardt/singer")
suppressPackageStartupMessages(library(singer))
## install ggmap
# devtools::install_github("dkahle/ggmap", ref = "tidyup")
## load ggmap
suppressPackageStartupMessages(library(ggmap))
## register API key (please do not use this key for any other purpose, thank you)
register_google(key = "AIzaSyDJnU2gOUgrDiIOSdujwanq9yX2yqRVlu4")
```

## Work with the`singer`data

# Task 4: Work with the `singer` data

  - Use `purrr` to map latitude and longitude into human readable
    information on the band’s origin places.

<!-- end list -->

``` r
# clean the data to delete the locations that have a value of NA
# here I only use the first 20 records to make the result clean
# The idea here can be used on the whole frame
cleaned_data <- singer_locations %>% 
  filter( !is.na(longitude)|!is.na(latitude)) %>%
  slice(1:20)
# convert the data to the readable version 
locations <- map2_chr(cleaned_data$longitude, cleaned_data$latitude, ~ revgeocode(as.numeric(c(.x, .y))))
```

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=41.88415,-87.63241&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.71455,-74.00712&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=42.33168,-83.04792&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.99471,-77.60454&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=34.20034,-119.18044&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=50.7323,7.10169&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=19.59009,-155.43414&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=34.05349,-118.24532&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.5725,-74.154&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=45.51179,-122.67563&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=51.50632,-0.12714&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=42.50172,12.88512&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.43831,-79.99745&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.71455,-74.00712&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=37.77916,-122.42005&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=40.71455,-74.00712&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=34.05349,-118.24532&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=37.27188,-119.27023&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=8.4177,-80.11278&key=xxx

    ## Source : https://maps.googleapis.com/maps/api/geocode/json?latlng=47.38028,-122.23742&key=xxx

``` r
## show the result
cbind(cleaned_data$city, locations) %>% 
  knitr::kable(col.names = c("city", "readable location"))
```

| city                         | readable location                                        |
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

  - Try to check wether the place in `city` corresponds to the
    information you retrieved.

<!-- end list -->

``` r
## first we need to find the city names
newcity<-cleaned_data %>% 
 select(city) %>% 
 separate(city,into = c("1","2","3"),sep=" ") 
```

    ## Warning: Expected 3 pieces. Additional pieces discarded in 2 rows [11, 12].

    ## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 11 rows [1,
    ## 3, 4, 5, 6, 7, 10, 13, 18, 19, 20].

``` r
#delete punctuation
newcity <- mapply(str_replace_all,newcity,",|-","") 
knitr::kable(newcity)
```

| 1            | 2          | 3       |
| :----------- | :--------- | :------ |
| Chicago      | IL         | NA      |
| New          | York       | NY      |
| Detroit      | MI         | NA      |
| Pennsylvania | NA         | NA      |
| Oxnard       | CA         | NA      |
| Bonn         | NA         | NA      |
| Hawaii       | NA         | NA      |
| Los          | Angeles    | CA      |
| Staten       | Island     | NY      |
| Portland     | OR         | NA      |
| UK           |            | England |
| Poggio       | Bustone    | Rieti   |
| Pittsburgh   | PA         | NA      |
| New          | York       | NY      |
| New          | York       | NY      |
| New          | York       | NY      |
| Los          | Angeles    | CA      |
| California   | NA         | NA      |
| Panama       | NA         | NA      |
| KENT         | WASHINGTON | NA      |

``` r
## second we need to extract the usel information from the city names
stringlocation <- as.character(locations)
newlocation <- str_extract_all(stringlocation[str_detect(stringlocation,"\\, [A-Z0-9].*[a-z]\\,|\\b[A-Z]{2,}\\b")],"\\, [A-Z0-9].*[a-z]\\,|\\b[A-Z]{2,}\\b",simplify = TRUE)
City <- str_replace_all(newlocation[,1],", |,", "")
newlocation[,1] = City
knitr::kable(newlocation)
```

|                |             |     |     |
| :------------- | :---------- | :-- | --- |
| Chicago        | IL          | USA |     |
| New York       | NY          | USA |     |
| Detroit        | MI          | USA |     |
| Howard         | PA          | USA |     |
| Oxnard         | CA          | USA |     |
| 53113 Bonn     |             |     |     |
| Hawaii         | USA         |     |     |
| Los Angeles    | CA          | USA |     |
| Staten Island  | NY          | USA |     |
| SW             | , Portland, | OR  | USA |
| Westminster    | UK          |     |     |
| Poggio Bustone | RI          |     |     |
| Pittsburgh     | PA          | USA |     |
| New York       | NY          | USA |     |
| San Francisco  | CA          | USA |     |
| New York       | NY          | USA |     |
| Los Angeles    | CA          | USA |     |
| CA             | USA         |     |     |
| Río Hato       |             |     |     |
| Kent           | WA          | USA |     |

``` r
result <- ((newlocation[,1]==newcity[,1])|(newlocation[,1]==newcity[,2])|(newlocation[,1]==newcity[,3])|(newlocation[,2]==newcity[,1])|(newlocation[,2]==newcity[,2])|(newlocation[,2]==newcity[,3])|(newlocation[,3]==newcity[,1])|(newlocation[,3]==newcity[,2])|(newlocation[,3]==newcity[,3]))
result
```

    ##  [1]  TRUE  TRUE  TRUE    NA  TRUE    NA  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [12] FALSE  TRUE  TRUE FALSE  TRUE  TRUE    NA    NA    NA

  - We can see that there are some false negative results like 53113
    Bonn, Germany“,”Bonn".
    
* If you still have time, you can go visual: give a look to the library leaflet and plot some information about the bands. A snippet of code is provided below.

```{r}
map <- leaflet()  %>%   
addCircles( lat=filter_NA_locations$latitude,
            lng=filter_NA_locations$longitude,
            popup = filter_NA_locations$title) %>% 
  addProviderTiles(providers$OpenStreetMap)

#saveWidget(map, "map.html", selfcontained = FALSE)
#webshot("map.html", file = "map.png",
#        cliprect = "viewport")

#mapshot(map, file = "./map.png")
```

