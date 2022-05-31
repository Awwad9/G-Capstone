---
title: "Bike Share Analysis"
author: "Awwad Albalawi"
date: "6/05/2022"
output: html_document
---

## Introduction

This is my data analysis for the Cyclistic bike-share analysis case study. In this R project, you will find the steps I took to answer the key business question, posed by the Case Study. I have followed the steps of the data analysis process: Ask, Prepare, Process, Analyze, Share, and Act. Along the way, I determine the answers to the Business tasks put before me.
 
By the end of this file, I will have presented a portfolio-ready case study that answers the question:

> In what ways do members and casual riders use Cyclistic bikes differently?

## Setup

First, I need to import the following libraries for use in my analysis:

* tidyr - Helps with Data Import & Wrangling
* dplyr - Helps with Data Wrangling
* here - Simplifies File Referencing
* lubridate - For working with Dates
* geosphere - To calculate Geographic Distances from longitudes & latitudes
* ggplot2 - For Visualizing Data

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)

## Set CRAN REPO
options(repos=structure(c(CRAN="http://cran.us.r-project.org")))

##Import Necessary Libraries
install.packages("librarian")
librarian::shelf(tidyr, dplyr, here, lubridate, geosphere, ggplot2)

```

## Verifying the Data

Data for this Analysis was secured from an online bucket / datasource of the last several months of a Bike Sharing Company's bike trips. This resource was provided by a 2nd party source i.e Google, for the purpose of analysis. They in turn got it from Motivate Inc who operate the Divvy Bike Share Service for the City of Chicago.

I verified this information by visiting the respective websites, and ascertaining that Divvy indeed exists, and is providing their data under a license. I reviewed the license and found everything to be in order. So in fact, the data did "ROCCC." That is, it was Reliable, Original, Current (I used data from the last 12 months), Cited, and Comprehensive.

## Exploring the Data

To get a sense of the data, I picked the most recent file in my dataset, and browsed it with a couple of R Commands from the Tidyverse and Dplyr

```{r data exploration}

aug_2021_biketrips <- read.csv(here("data/202108-divvy-tripdata.csv"))
head(aug_2021_biketrips)
glimpse(aug_2021_biketrips)

```

Now, having looked at the first few rows, I notice that the dates for **started_at** and **ended_at** are stored as characters instead of datetimes, so I want to change that using the *lubridate* library. I also notice that the latitude and longitude of the startpoint and endpoint are recorded. With these, I should be able to calculate the distance travelled using the *geosphere* library. 

```{r transforming the date columns and calculating distance}

aug_2021_biketrips <- aug_2021_biketrips %>% 
  mutate(started_at = ymd_hms(started_at)) %>% 
  mutate(ended_at = ymd_hms(ended_at)) %>% 
  mutate(distance = distHaversine(cbind(start_lng, start_lat), cbind(end_lng, end_lat)))

```

With that done, we can now add a calculated column of the ride length on each ride. I have a theory that the people who spend the most time cycling, are either members or casual cyclists more likely to become members.

```{r calculate and sort by ride length }

aug_2021_biketrips <- aug_2021_biketrips %>% 
  mutate(ride_length = ended_at - started_at) %>% 
  mutate(time_spent = seconds_to_period(ride_length)) %>% 
  mutate(day_of_week = weekdays(started_at)) %>% 
  arrange(-ride_length)

head(aug_2021_biketrips) 

```
Wow! Contrary to my expectation, casual cyclists topped the list of time spent biking. Let me see if that holds true across the board.

```{r plot rideable type against ride length and member type}

ggplot(aug_2021_biketrips) +
  geom_jitter(aes(x = rideable_type, y = ride_length, color = member_casual))

```

Yes it does. Not only do casual riders spend more time cycling, but they also seem to have a preference for the docked bike (over electric, and classic). But, this was simply the case in August 2021. Let's look back over the course of 12 months and see if the data still reflects the same way.

## Creating The Full Dataset

```{r looking over a 12 month period}

biketrips <- list.files(path = here("data"), full.names = TRUE) %>% 
  lapply(read.csv) # %>% 
# bind_rows 

# Bind rows doesn't work because for some reason the start_station_id and end_station_id column doesn't maintain the same data type across files. So we're just going to have to fix it up..

merged_biketrips <- bind_rows(biketrips[1], biketrips[2], biketrips[3])

merged_biketrips <- merged_biketrips %>% 
  mutate(start_station_id = as.character(start_station_id)) %>% 
  mutate(end_station_id = as.character(end_station_id))

merged_biketrips_2 <- bind_rows(biketrips[4], biketrips[5], biketrips[6],
                                biketrips[7], biketrips[8], biketrips[9],
                                biketrips[10], biketrips[11], biketrips[12])

merged_biketrips <- bind_rows(merged_biketrips, merged_biketrips_2)

```

Now we have all the data, we can see if the august sample was reflective of the whole population. But first we need to rebuild the calculated columns in our full dataset.

```{r calculating time spent & distance travelled over full dataset}

all_trips <- merged_biketrips %>% 
  mutate(started_at = ymd_hms(started_at)) %>% 
  mutate(ended_at = ymd_hms(ended_at)) %>%
  mutate(ride_length = ended_at - started_at) %>% 
  mutate(day_of_week = weekdays(started_at)) %>% 
  mutate(distance = distHaversine(cbind(start_lng, start_lat), cbind(end_lng, end_lat)))

head(all_trips)
```

Finally, we can reconsider the plot of ride length by casual and member riders per bike type.

```{r Plotting the full dataset with ride length}

ggplot(all_trips) +
  geom_jitter(aes(x = rideable_type, y = ride_length, color = member_casual))

```

As shown on the plot above, there is some dirty data: rides with negative ride lengths for starters. So we need to clean our data. Lets start by keeping only rides with positive distances or ride lengths. So at least we know that they are valid trips.

```{r cleaning trip data}

# Keep the fields we want for bike trips that have valid ride length & distance
all_trips <- all_trips %>% 
  select(ride_id,
         rideable_type,
         started_at,
         ended_at,
         distance,
         member_casual,
         ride_length,
         day_of_week) %>% 
  filter(distance > 0) %>% # Only bikes 
  filter(ride_length > 0)  # that have been used

# Plot the cleaned dataset to crosscheck

ggplot(all_trips) +
  geom_jitter(aes(x = rideable_type, y = ride_length, color = member_casual))

```

Let's also check out what the most popular day for riding was by the different member types.

```{r most popular riding day by member type}

ggplot(all_trips) +
  geom_bar(aes(x = day_of_week, fill = member_casual)) +
  theme(axis.text.x = element_text(angle = 25))

```

And then lets check out the relationship between bikes and travel distances by member type

```{r plot rideable type against distances travelled by member_casual }

ggplot(all_trips) +
  geom_jitter(aes(x = rideable_type, y = distance, color = member_casual))

```

## Conclusions

As we can see from the last few plots, casual riders are the most numerous riders, and they prefer docked bikes. Whereas members prefer electric and classic bikes. Also as is shown in the second plot, the most popular days for casual riders riding are Saturday & Sunday while members do similar amounts of riding throughout the week. Finally as far as distance goes, we can see that both members and casual riders ride about the same distances across bikes.

So, given this information, what are my top 3 recommendations?

1. Perhaps create a special offer for members using docked bikes. To encourage docked bike users (mostly casual) to consider taking up memberships.
2. Perhaps offer discounts on the weekends for members. Again to encourage weekend riders (mostly casual), to become members.
3. Finally, a public leader board might be an interesting idea. To recognize riders who spend the most time biking (currently casual riders), and encourage people to ride more.
