---
title: "Google Data Analyst Case Study (JR)"
author: "Eray Nurtekin"
date: "19 10 2021"
output: html_document
---
## ASK

## Scenario

You are a junior data analyst working on the marketing analyst team at [Bellabeat](https://bellabeat.com/), a high-tech manufacturer of health-focused products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat executive team along with your high-level recommendations for Bellabeat’s marketing strategy.

## Business Task
Analyze Fitbit data to gain insight and help guide marketing strategy for Bellabeat to grow as a global player.

The dataset used for this analysis is the [FitBit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit) public data, made available through [Möbius](https://www.kaggle.com/arashnic)

## Stakeholders
Primary Stakeholders: 

Urška Sršen: Cofounder and Chief Creative Officer
Sando Mur: Cofounder and Mathematician


Secondary Stakeholders: 


Bellabeat marketing analytics team

## PREPARE

In the Prepare phase, we identify the data being used and its limitations.

Information on Data Source:

1-Data is publicly available on Kaggle: [FitBit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit) and stored in 18 csv files.
2-Generated by respondents from a survey via Amazon Mechanical Turk between 12 March 2016 to 12 May 2016.
3-30 FitBit users consented to the submission of personal tracker data.
4-Data collected includes physical activity recorded in minutes, heart rate, sleep monitoring, daily activity and steps.

Limitations of Data Set:
1-Data is collected 5 years ago in 2016. Users’ daily activity, fitness and sleeping habits, diet and food consumption may have changed since then. Data may not be timely or relevant.
2-Sample size of 30 FitBit users is not representative of the entire fitness population.
3-As data is collected in a survey, we are unable to ascertain its integrity or accuracy.

Information of ROCCC?

Reliable — LOW — Not reliable because it is only has 30 answering

Original  — LOW — Third party provider

Comprehensive — MED — Parameters match most of Bellabeat products’ parameters

Current — LOW — Data is 5 years old and may not be relevant

Cited — LOW — Data collected from third party, hence unknown


---The datasets consist of 18 files, but I will study mainly on dailyActivity_merged.csv file as supporting data. The reasons are:
I will not dive deep into the details in datasets based on hourly, minutes, and seconds to limit the scope of analysis
I will not use weightlog since only 8 IDs registered weight, while there are 33 IDs in total
dailyActivity_merged.csv is already merged versions of several other datasets


#Loading Packages and libraries
```{r}
#install.packages('tidyverse')
#install.packages('skimr')
#install.packages('cowplot')
#install.packages("plotly")
library(tidyverse) #wrangle data
library(dplyr) #clean data
library(lubridate)  #wrangle date attributes
library(skimr) #get summary data
library(ggplot2) #visualize data
library(cowplot) #grid the plot
library(readr) #save csv 
library(plotly) #pie chart
library(janitor)#examining and cleaning dirty data
library(scales)#Scales to better state the numbers shown in visualization
```


#Reading in the selected file.
```{r}
setwd("C:/Users/PC/Desktop/Work/Fitabase Data 4.12.16-5.12.16")

daily_activity <- read.csv("dailyActivity_merged.csv")
head(daily_activity)
```

#Used Skimr package to get the general sense of the data
```{r}
skim_without_charts(daily_activity)
```

#check for NA and Duplicates
```{r}
sum(is.na(daily_activity))
sum(duplicated(daily_activity))
```
#Looked into the how many Ids tracked the data in daily_activity
```{r}
n_distinct(daily_activity$Id)
```
#convert activity_date from chr to date
```{r}
daily_activity <- daily_activity %>% mutate( Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))
```
#For using english weekday names
```{r}

daily_activity<- daily_activity %>% 
  mutate(daily_activity, week_eng =case_when(
    Weekday == "Pazartesi" ~ "Monday",
    Weekday == "Salı" ~ "Tuesday",
    Weekday == "Çarşamba" ~ "Wednesday",
    Weekday == "Perşembe" ~ "Thursday",
    Weekday == "Cuma" ~ "Friday",
    Weekday == "Cumartesi" ~ "Saturday",
    Weekday == "Pazar" ~ "Sunday"))

View(daily_activity)
```

Helped to create step rank column by finding min and max values 

Interpreting statistical finding:

-On average, users logged 7,637 steps or 5.4km which is not adequate. As recommended by CDC, an adult female has to aim at least 10,000 steps or 8km per day to benefit from general health, weight loss and fitness improvement. Source: Medical News Today article

```{r}
daily_activity <- daily_activity %>%
  mutate(daily_activity, step_rank = case_when(
    TotalSteps < 100 ~ "N/A",
    TotalSteps >= 100 & TotalSteps < 6000 ~ "sedentary",
    TotalSteps >= 6000 & TotalSteps < 8500 ~ "low active",
    TotalSteps >= 8500 & TotalSteps < 11000 ~ "somewhat active",
    TotalSteps >= 11000 & TotalSteps < 13500 ~ "active",
    TotalSteps >= 13500 ~ "highly active"))
```

#filtering out the "N/A" values in the step_rank colum
```{r}
daily_activity <- daily_activity %>%
  filter(step_rank != "N/A")
daily_activity$step_rank <- ordered(daily_activity$step_rank, levels=c("sedentary", "low active", "somewhat active", "active", "highly active"))
View(daily_activity)
```
# Analyzing and Visualization

From our scatter plot, we can see there is a positive correlation between total steps and calories, this could be a good point for marketing the Bellabeat's Time for active users.
```{r}
daily_activity %>%
  ggplot(aes(x=TotalSteps,y=Calories)) + geom_point() +
  geom_smooth(method = "lm", formula=y~x) + 
  labs(title="TotalSteps vs Calories")
```
![smooth1](https://user-images.githubusercontent.com/79145084/137901802-a8d2d1ca-1352-4085-bd86-a5b6b8cd83b2.png)

From our plot, we see that again there is a positive correlation between total distance and calories, this can be used for Bellabeat strategies. 
```{r}
daily_activity %>%
  ggplot(aes(x=TotalDistance,y=Calories)) + geom_point() +
  geom_smooth(method = "lm", formula=y~x) + 
  labs(title="TotalDistance vs Calories")
```

![smooth2](https://user-images.githubusercontent.com/79145084/137901811-13a888f7-c602-49b5-9e5f-be99a909c6f4.png)


Noted a few outliers :

1 observation of > 35,000 steps with < 3,000 calories burned.
Deduced that outliers could be due to natural variation of data, change in user’s usage or errors in data collection (ie. miscalculations, data contamination or human error).

#Pie charts
```{r}
active_users <- daily_activity %>%
    filter(FairlyActiveMinutes >= 20.5 | VeryActiveMinutes>=10.5) %>% 
    group_by(Id) %>% 
    count(Id) 
  total_minutes <- sum(daily_activity$SedentaryMinutes, daily_activity$VeryActiveMinutes, daily_activity$FairlyActiveMinutes, daily_activity$LightlyActiveMinutes)
  sedentary_percentage <- sum(daily_activity$SedentaryMinutes)/total_minutes*100
  lightly_percentage <- sum(daily_activity$LightlyActiveMinutes)/total_minutes*100
  fairly_percentage <- sum(daily_activity$FairlyActiveMinutes)/total_minutes*100
  active_percentage <- sum(daily_activity$VeryActiveMinutes)/total_minutes*100
  
 

  percentage <- data.frame(
    level=c("Sedentary", "Lightly", "Fairly", "Very Active"),
    minutes=c(sedentary_percentage,lightly_percentage,fairly_percentage,active_percentage))
  plot_ly(percentage, labels = ~level, values = ~minutes, type = 'pie',textposition = 'outside',textinfo = 'label+percent') %>%
    layout(title = 'Percentage of Activity in Minutes',
           xaxis = list(showgrid = TRUE, zeroline = FALSE, showticklabels = TRUE),
           yaxis = list(showgrid = TRUE, zeroline = FALSE, showticklabels = TRUE))
           
```
![pie](https://user-images.githubusercontent.com/79145084/137901527-6fbf8773-d645-45b7-ad25-49a9b29343c8.png)

This indicates that users are using the FitBit app to log daily activities such as daily commute, inactive movements (moving from one spot to another) or running errands.
App is rarely being used to track fitness (ie. running) as per the minor percentage of fairly active activity (1.24%) and very active activity (1.93%). This is highly discouraging as FitBit app was developed to encourage fitness.
           
## ACT
## Recommendations and Conclusion

What are the trends identified?
-Trends that Majority of users (79.2%) are using the FitBit app to track sedentary activities and not using it for tracking their health habits.

How could these trends help influence Bellabeat marketing strategy?
-Product should be tailored to each users profile to find what works for the and how best to utilize the product.

In conclusion, Bellabeat marketing team can encourage users by educating and equipping them with knowledge about fitness benefits, suggest different types of exercise (ie. simple 10 minutes exercise on weekday and a more intense exercise on weekends) and calories intake and burnt rate information on the Bellabeat app.
