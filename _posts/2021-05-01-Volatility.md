---
layout: post
title: Determining Quarterback Volatility with Gini Coefficients and GAM
excerpt: "Quantifying QB volatility and developing a volatility over expected GAM model"
modified: 8/22/2021, 9:00:24
tags: [GAM, Volatility]
comments: true
category: blog
---

## Intro: 

A few years ago, Bill Petti posted a [great article on FanGraphs](https://tht.fangraphs.com/corrvol-updating-how-we-can-measure-hitter-volatility/) describing hitter volatility and how it can be quantified via Gini coefficients. Petti wrote that volatility is how a player distributes their overall season performance (as measured by wRC in his case) throughout the season on a game by game basis. This is opposed to streakiness which is related to clustering of good and bad performances over the course of a season. To quantify a hitters volatility, he proposed the usage of a Gini coefficient. Typically Gini coefficients are used to evaluate the wealth distribution in countries (0-1 scale where higher values indicate less equal distribution of wealth). In the case of football, a Gini coefficient can tell us how each individual quarterback distributes their cumulative season EPA (passes, rushes, penalties) on a game by game basis (includes playoffs). 

In this post, I follow a similar methodology to Petti’s original work to calculate a quarterback’s volatility by season. In addition, I create a “Volatility Over Expected (VOLoe)” GAM model to evaluate a QB’s volatility within the context of their season EPA and total opportunities (passes, rushes, penalties).

## Data Prep and Packages 

This post uses NFLfastR quarterback data from 1999 to 2020. The full data prep process can be found on my GitHub (just a long loop that loads the data, adds rosters so we can filter for only QB’s, and adding team colors). Some of the notable assumptions in the data cleaning process are:
1.  Games where QB x is involved in less than 5 plays (passes, rushes, penalties) are removed from that QB’s history. This is under the assumption that if QB x was involved in so few plays, they were injured or rested. Since the gini coefficient doesn’t consider the amount of plays in a game, this method protects the gini score from being inflated by very low opportunity, low EPA games. 
2.  Filtered for QB’s who threw greater than 250 passes in a season.

```{r}
#packages
library(dplyr)
library(ggcorrplot)
library(ggplot2)
library(gt)
library(mgcv)
library(nflfastR)
library(readr)
library(tidyverse)
#load data
DF <- read_rds("VOL_DF.rds") %>%
  filter(
    plays >= 10,#remove any games where QB was involved in less than 10 plays
    season_plays >= 250) #filter for greater than 250 attempts
```

## Gini Coefficient Calculation & Analysis

To calculate each QB’s gini coefficient, we can use the same function created in Petti’s article. A typical gini calculation can’t be used with EPA because of negative EPA values, thus the modified gini method has to be used. 

```{r}
#Gini coefficient function to handle negative EPA
Gini_Function <- function(Y) {
  
  Y <- sort(Y)
  
  N <- length(Y)
  
  u_Y <- mean(Y)
  
  top <- 2/N^2 * sum(seq(1,N)*Y) - (1/N)*sum(Y) - (1/N^2) * sum(Y)
  
  min_T <- function(x) {
    
    return(min(0,x))
    
  }
  
  max_T <- function(x) {
    
    return(max(0,x))
    
  }
  
  T_all <- sum(Y)
  
  T_min <- abs(sum(sapply(Y, FUN = min_T)))
  
  T_max <- sum(sapply(Y, FUN = max_T))
  
  u_P <- (N-1)/N^2*(T_max + T_min)
  
  return(top/u_P)
  
}
#calculate gini scores for our QBs
pbp_QBs_gini <- DF %>% 
  select(id, season, Total_EPA) %>% 
  na.omit() %>% 
  aggregate(Total_EPA ~ id + season, data = ., FUN = "Gini_Function") %>%
  rename("VOL" = "Total_EPA")%>%
  left_join(DF, by = c("id", "season")) %>%
  filter(VOL != "NaN")
#remove extra data
rm(DF)
rm(Gini_Function)
```

Applying the function to our data frame yields a gini score for each quarterback season since 1999 (minimum 250 throws in given season). We can then plot a QB’s volatility and EPA/Play in the 2020 season. Lower volatility is better, meaning up and to the right is best.

![placeholder](https://www.opensourcefootball.com/posts/2021-08-15-QB-volatilty-gini-coefficients/QB_Volatility_files/figure-html5/unnamed-chunk-3-1.png)

This isn’t too surprising, good quarterbacks (as measured by EPA/Play) are consistent while middle of the pack QB’s tend to be the most volatile. Once we reach the worst QB’s (Wentz, Haskins, Darnold) those players tend to be consistently bad. Essentially, the relationship between EPA/Play and volatility is a parabola shape. 

This plot also provides perspective on QB’s who have a similar EPA/Play but have very different levels of volatility. For example, Wentz (-0.06 EPA/Play) and Foles (-0.08 EPA/Play). If a decision maker was in the unfortunate position of deciding between these two below average QB’s, they could possibly prefer the volatile QB (Foles, who has a chance of a great game) over a low volatility QB (Wentz, who is consistently mediocre). 

## Volatility Over Expected 

VOL tells a simple story but it doesn’t tell us how much more volatile a QB was given their EPA. E.g. Ryan Tannehill had a 0.31 EPA/Play and a volatility of 0.45, was this more volatile than expected given his performance? We can built a Generalized Additive Model (GAM) with MGCV that includes variables that correlate with VOL. The two variables that most correlate with VOL are EPA/Play & Total Plays. 

![placeholder](https://www.opensourcefootball.com/posts/2021-08-15-QB-volatilty-gini-coefficients/QB_Volatility_files/figure-html5/unnamed-chunk-4-1.png)

And then we can built the model, I tried a few variations but the model with a 9 knot cubic smoothing spline on EPA/Play performed best. 

```{r}
#Build expected VOL model that controls for EPA & Plays
Modeling_Data <- QBs_Grouped %>%
  select(
    VOL,
    EPA_play,
    Plays
  )
#training test split
set.seed(2018)
df    <- sample(nrow(Modeling_Data), nrow(Modeling_Data) * .7)
train <- Modeling_Data[df,]
test  <- Modeling_Data[-df,]

#tried a few models, GAM performed best
#fit model after trying a few different combos
model1 <- mgcv::gam(VOL ~ s(EPA_play, k = 9, bs="cr") + Plays, data = train)

#check again
mgcv::gam.check(model1, k.rep=1000) #K & EDF are not close, p is no longer significant
summary(model1) #0.75 adjusted r squared

#check rmse
fit_1 <- predict(model1, test)
model_1_res_act <- data.frame(actuals = test$VOL, predicted = fit_1) %>%
  mutate(errors = predicted - actuals)
rmse <- function(df) {
  rmse <- sqrt(mean((df$actuals-df$predicted)^2))
  rmse
}
rmse(model_1_res_act) #.05 rmse
```
 
The model has an adjusted R squared of .75, an RMSE of .05, and the GAM check diagnostics pass the smell test. After fitting the model to our data, we can re-plot 2020 QB EPA/Play and VOL over expected. VOLoe values slightly off 0 (Allen, Mahomes, Brady) shouldn't be cause for concern, instead they should be interpreted as about right where expected. The primary usage of VOLoe should be in detecting significantly higher or lower VOLoe values. Higher VOLoe is worse and represents a more volatile than expected season.

```{r}
#fit model to actual data
fit_all <- data_frame(predicted = predict(model1, QBs_Grouped))
fit_values <- cbind(QBs_Grouped, fit_all) %>%
  mutate(VOLoe = VOL - predicted)
```

![placeholder](https://www.opensourcefootball.com/posts/2021-08-15-QB-volatilty-gini-coefficients/QB_Volatility_files/figure-html5/unnamed-chunk-6-1.png)

Most quarterbacks were close to their predicted level of volatility. Two QB’s that particularly stick out are Fitzpatrick and Tannehill, both performed well but had high volatility over expected scores. A quick investigation of QB’s who had a similar high level of EPA/Play and VOL over expected could perhaps provide insight into if we should be skeptical of this profile. 

```{r}
#find QBs with 85th percentile or greater EPA & VOL
quantile(fit_values$EPA_play, probs = c(0.85)) #.18 
quantile(fit_values$VOLoe, probs = c(0.85)) #.05 or more
#filter for QBs that fit those conditions
outlier_qbs <- fit_values %>%
  filter(EPA_play >= .178, VOLoe >= .05) %>%
  mutate(outlier = 1) %>%
  select(id,
         season,
         outlier)
#join data and create table
fit_values %>%
  left_join(outlier_qbs, by = c("id", "season")) %>%
  group_by(id) %>%
  #create indicator that will allow us to filter out QBs without season of interest
  mutate(outlier = mean(outlier, na.rm = TRUE)) %>%
  ungroup() %>%
  filter(outlier != "NaN") %>%
  #create summary columns
  mutate(
    outlier = ifelse(EPA_play >= .178 & VOLoe >= .05, 1, 0), #mark outliers
    VOLoe_prior = lag(VOLoe),
    follower = lag(outlier),
    prior_season = lag(season),
    EPA_prior = round(lag(EPA_play), digits = 3),
    EPA_diff = round(EPA_play - lag(EPA_play), digits = 3),
    EPA_play = round(EPA_play, digits = 3)) %>%
  filter(follower == 1) %>%
  select(
    name,
    season,
    prior_season,
    VOLoe_prior,
    EPA_prior,
    EPA_play,
    EPA_diff
  ) %>%
  rename("VOLoe Prior Season" = "VOLoe_prior") %>%
  rename("EPA/Play Prior Season" = "EPA_prior") %>%
  rename("EPA/Play Next Season" = "EPA_play") %>%
  rename("Change in EPA" = "EPA_diff") %>%
  arrange(desc(season)) %>%
  filter(prior_season + 1 == season) %>%
  arrange(desc(season)) %>%
  gt()
```

Very small sample size disclaimer but most QB’s with this profile experienced a decrease in their EPA/Play the following season (-.08 average decrease in EPA/Play). It’s difficult to know if this is because of a high VOLoe in the prior season or because it is difficult to maintain an elite level of EPA. This does align with some of skepticism people have about Tannehill and Fitzpatrick though. 

Other Questions to Answer

There are a lot of questions related to volatility that come to mind, but for the sake of brevity I will touch upon one that immediately comes to mind: does volatility of VOLoe change with a QB’s age? We can examine both metrics by age. 

```{r}
#first, merge data with rosters and calculate age
calc_age <- function(birthDate, refDate = Sys.Date(), unit = "year") {
  
  require(lubridate)
  
  if(grepl(x = unit, pattern = "year")) {
    as.period(interval(birthDate, refDate), unit = 'year')$year
  } else if(grepl(x = unit, pattern = "month")) {
    as.period(interval(birthDate, refDate), unit = 'month')$month
  } else if(grepl(x = unit, pattern = "week")) {
    floor(as.period(interval(birthDate, refDate), unit = 'day')$day / 7)
  } else if(grepl(x = unit, pattern = "day")) {
    as.period(interval(birthDate, refDate), unit = 'day')$day
  } else {
    print("Argument 'unit' must be one of 'year', 'month', 'week', or 'day'")
    NA
  }
  
}
#loop to get 1999 to 2020 rosters
rosters <- data.frame()
for (x in 1999:2020) {
  df <- nflfastR::fast_scraper_roster(x) %>%
    mutate(birth_date = as.Date(birth_date)) %>%
    select(
      position,
      birth_date,
      gsis_id,
      season,
      full_name,
      team
    ) %>%
    mutate(Age_Sep_1 = calc_age(birth_date, paste0(x,"-09-01")))
  rosters <- rbind(df, rosters)
}
```

![placeholder](https://www.opensourcefootball.com/posts/2021-08-15-QB-volatilty-gini-coefficients/QB_Volatility_files/figure-html5/unnamed-chunk-8-1.png)

There doesn’t appear to be very much of a relationship. This is also a difficult question to answer because of survivorship bias. Good QB’s play longer, good QB’s also tend to have low volatility. Thus, volatile QB’s tend to be average to below average QB’s and don’t play as long. 

## Wrapping Up

This post demonstrated that on a macro scale, good QB’s have low volatility while middle of the pack QB’s tend to have the highest level of volatility. Raw volatility could be helpful for deciding between backup QB’s with similar EPA/Play (prefer the volatile bad QB due to the chance of a good game). Volatility over expected (VOLoe) can be used to detect high level EPA/Play seasons that also have a high VOLoe (possible indication of future regression). 

