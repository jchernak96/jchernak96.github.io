---
layout: post
title: Mixed Models for Analyzing NFL Offensive Coordinator Tendencies
excerpt: "Usage of a generalized additive model with mixed effects to analyze coordinator pass heaviness."
modified: 2/29/2016, 9:00:24
tags: [Rvest, Scraping, GAM, Mixed Effects]
comments: true
category: blog
---

We are all aware of team tendencies in the NFL and how certain ones are more pass heavy than others. What is not well researched is how this may vary on the coordinator level and if certain OC's have tendencies that can be picked up on. In this post, I scrape PFR for coordinator data (OC & DC), clean the data, and develop a simple model with the OC as a random effect. I then analyze how each OC increases the probability of passing on 1st & 10. 

## Scraping PFR

This process was very similar to my previous post regarding injury scraping on PFR. I followed the same process of constructing URLs for each page and combining them with years to get a list of URLs to scrape through. I won't delve into that again and instead will move onto the scraping function. The function is interested in the coaches info you can see at the [top of the team page](https://www.pro-football-reference.com/teams/nwe/2020.htm). My process can be found below but the basic idea was that all of the coordinator info is located in the same node. I scrape that node, use regex to search for defensive or offensive coordinator, and then add that to a dataframe. I also scrape important id info such as season and team so that the data can be added to play by play data. Once this is done, I clean the data so that it is easily understood. 

```{r}
coordinator_scraper <- function(urls) {
  
url   <- read_html(urls)
  
A <- url %>%
  html_nodes("p") %>%
        html_text() %>%
  as.data.frame() %>%
  rename("O_Coord" = ".") %>%
  filter(across(O_Coord, ~ grepl('Offensive Coordinator', .))) 
A <- ifelse(length(A$O_Coord) < 1, 
       url %>%
       html_nodes("p") %>%
       html_text() %>%
       as.data.frame() %>%
       rename("O_Coord" = ".") %>%
       filter(across(O_Coord, ~ grepl('Coach:', .))) ,
       A) %>%
  as.data.frame()
colnames(A)[1] <- "O_Coord"
B <- url %>%
  html_nodes("p") %>%
        html_text() %>%
  as.data.frame() %>%
  rename("D_Coord" = ".") %>%
  filter(across(D_Coord, ~ grepl('Defensive Coordinator', .))) 
B <- ifelse(length(B$D_Coord) < 1, 
       url %>%
       html_nodes("p") %>%
       html_text() %>%
       as.data.frame() %>%
       rename("D_Coord" = ".") %>%
       filter(across(D_Coord, ~ grepl('Coach:', .))) ,
       B) %>%
  as.data.frame()
colnames(B)[1] <- "D_Coord"
C <-url %>%
  html_nodes("p") %>%
        html_text() %>%
  as.data.frame() %>%
  rename("Team" = ".") %>%
  filter(across(Team, ~ grepl('Franchise', .))) 
D <-url %>%
  html_nodes("p") %>%
        html_text() %>%
  as.data.frame() %>%
  rename("Season" = ".") %>%
  filter(across(Season, ~ grepl('Statistics', .))) 
data <- cbind(A,B,C, D)
return(data)
}
###Create empty data frame to store data
empty <- data.frame()
###Create loop that will go through our urls and return injuryies
for (x in url_list) {
output <- coordinator_scraper(x)
empty <- rbind(empty, output)
}

Cleaned_Data <- empty 

#remove uneeded data
Cleaned_Data$O_Coord <- gsub("Offensive Coordinator: ","", Cleaned_Data$O_Coord)
Cleaned_Data$O_Coord <- gsub("Coach: \n ","", Cleaned_Data$O_Coord)
Cleaned_Data$D_Coord <- gsub("Defensive Coordinator: ","", Cleaned_Data$D_Coord)
Cleaned_Data$D_Coord <- gsub("Coach: \n ","", Cleaned_Data$D_Coord)
Cleaned_Data$Team    <- gsub("Franchise Pages","", Cleaned_Data$Team)

#clean season column
Cleaned_Data$Season <- str_sub(Cleaned_Data$Season, 1, 4) 
Cleaned_Data$Season <- as.numeric(Cleaned_Data$Season)

#clean coordinator columns that have records because HC is the coordinator
Cleaned_Data$O_Coord <- gsub("[[:digit:]]+","",Cleaned_Data$O_Coord)
Cleaned_Data$O_Coord <- gsub("(--)","",Cleaned_Data$O_Coord)
Cleaned_Data$O_Coord <- gsub(paste(c("[(]", "[)]"), collapse = "|"), "", Cleaned_Data$O_Coord)
Cleaned_Data$D_Coord <- gsub("[[:digit:]]+","",Cleaned_Data$D_Coord)
Cleaned_Data$D_Coord <- gsub("--","",Cleaned_Data$D_Coord)
Cleaned_Data$D_Coord <- gsub(paste(c("[(]", "[)]"), collapse = "|"), "", Cleaned_Data$D_Coord)
Cleaned_Data <- Cleaned_Data %>%
  rename("Coord_O" = O_Coord) %>%
  rename("Coord_D" = D_Coord)
  
#Make data long for merging with PBP
Cleaned_Data <- Cleaned_Data %>%
 pivot_longer(
   cols = starts_with("Coord"),
   names_to = "Coord",
   names_prefix = "Season",
   values_to = "Coordinator",
   values_drop_na = TRUE
 ) %>%
  rename("Side_of_Ball" = "Coord") %>%
  mutate(Side_of_Ball = ifelse(Side_of_Ball == "Coord_O", "Offense", "Defense"))
```






