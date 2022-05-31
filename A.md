---
title: "Cyclistic Bike-Share-Data-Analysis"
author: "Awwad Albalawi"
date: "06/01/2022"
output: html_document
---

# The Cyclistic Bike-Share Company Data Analysis case study.

## Main Objectives:

* The director of marketing believes the company’s future success depends on maximizing the number of annual memberships.

* To understand how casual riders and annual members use Cyclistic bikes differently is very important to the director.

* Insights of the data would inform the design of a new marketing strategy to convert casual riders into annual members.

## Main question:
How do annual members and casual riders use Cyclistic bikes differently?

### Sub Questions:
1) What age groups are more likely to use Cyclistic Bike-Share services?
2) Does usage of Cyclistic Bike-Share services differ among males and females?
3) How has the Cyclistic Bike-Share user type changed from 2013 to 2019?
4) What is the average trip duration among subscribers, compared to non-subscribers?
5) What age groups are the non-subscribers? 
6) Does the gender of the non-subscriber matter in Bike-Share services usage?
7) What are the top "From stations" used by non-subscribers?
8) What is the popularity of different bike types among subscribers and non-subscribers?
9) What are the high demand periods for bike-share services?

### Data Sources:
The analysis in this case study relies on the Cyclistic Bike-Share Company Data extracted from the following link:

<https://divvy-tripdata.s3.amazonaws.com/index.html>

Part of the datasets (from 2013 to 2019) are grouped by quarter and can be downloaded as separate csv files. In the quarterly datasets, the column names differ in some of the files. The data types also differ in some of the datasets. However, the individual datasets have the same number of columns.Individual datasets have hundreds of thousands of rows. These quarterly datasets have information on the gender and birth year of the user.
The newer data (from april 2020 to april 2022) are vailable as monthly csv datasets. These newer datasets do not have any information neither on the gender nor the birth year of the user. 

The first seven questions are addressed using the data from 2013 to 2019 while the last two questions were answered using the newer data (2021 and 2022 data) 

### Data Cleaning phase:
In order not to make this report too long, I will exclude some of the data cleaning steps since I had to go through more than 50 datasets in order to align them. For each of the new and the old data, the data cleaning steps included in this report start at the point where all the individual datasets have been put together into one dataset. I kept only columns that I intended to use for
the analysis in this case study. 

#### Get the R libraries needed for this analysis:
