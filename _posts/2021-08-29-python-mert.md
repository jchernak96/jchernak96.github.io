---
layout: post
title: Creating a Coordinator Adjusted Expected Pass Model in Python
excerpt: "Using coordinator data to build a tree boosted mixed effects model"
modified: 2/29/2016, 9:00:24
tags: [Python, Mixed Effects, Coordinators]
comments: true
category: blog
---

As the season gets underway, I've been thinking about the importance of coordinators and how their tendencies can be used against them. Past investigations into this topic have shown that certain coordinators have a high pass rate over expected and that this is mostly sticky throughout the season. But nobody has attempted to incorporate who the coordinator is in their pass probability models. Thus, I decided to fill this gap and attempt to build an xPass model with coordinator mixed effects. 

In this post I;
- Use Python to build a tree boosted mixed effects model that provides a probability of pass for any given scenario
- Examine coordinator pass probabilities under various scenarios

### Data 

As outlined in my previous post, I scraped PFR for their coordinator level data and output it into a format that can be quickly joined to PBB data. I loaded this data into Python and only selected the last 6 years of data (otherwise my computer will run very very slow). An important aspect of my analysis is that I am considering each coordinator year as its own group, meaning the model will view coordinator 2019 as different than coordinator 2020. My reasoning is that tendencies can change between years because of personnel so it is more effective to assign situational pass probabilites year by year and then compare those charts. Ideally, there is a way to weight more recent coordinator observations so that the predictions are not skewed by old seasons. This is something to explore in the future though.

```{r}
#Load in packages
from gpboost.engine import train
import pandas as pd
import numpy as np
import gpboost as gpb
from sklearn.model_selection import train_test_split
import matplotlib
from sklearn.model_selection import GroupShuffleSplit

#change options and set seed
pd.set_option("display.max_rows", None, "display.max_columns", None)
np.random.seed(59)

#load in data & check
pbp_data = pd.read_csv("C:/Users/Joe/Desktop/Coordinator_NFL_Final.csv")
pbp_data.head(5)
```

### Issues and Solutions

Incorporating categorical variables into a random forest model can be approached from multiple angles, one could one hot encode how ever many coordinators into the data or just build the random forest model and insert a mixed effect for coordinators after. Both approaches are suboptimal though and I saught out a solution that combines random forests wtih mixed effects. I found the GPboost package as a result, the package combines tree-boosting and mixed effects to create one model. I've never seen this method used in the sports space to my knowledge so this required a fair amount of trial and error on my part (especially since I am still new to Python). Nevertheless, I powered through and learned a lot about Python along the way. 

With one issue out of the way, I found another new problem. A traditional train test split would not be a good idea for this mixed effect model because we would run the risk of overrepresenting certain coordinators or underrepresenting in the train or test sets. And because we are interested in every coordinator, we have to make sure each one has an equal proportion of data in the train and test sets we will be using. So, I wrote a simple loop that takes 70% of each coordinators season data and adds it to a train dataframe while the remaining proportion is put into a test set. 

```{r}
#Select our target and variables of interest

modeling_data = pbp_data[["yardline_100", 
                          "qtr",
                          "half_seconds_remaining", 
                          "ydstogo",
                          "down",
                          "shotgun",            
                          "score_differential",
                          "posteam_timeouts_remaining",
                          "defteam_timeouts_remaining",
                          "wp", 
                          "O_Coordinator",
                          "pass"]]

# Train test method

train_final  = pd.DataFrame()
test_final   = pd.DataFrame()
Coordinators = modeling_data['O_Coordinator'].unique()
length = len(Coordinators)
for x in range(length):
    Coordinator_X = Coordinators[x]
    Data_Filtered = modeling_data[modeling_data['O_Coordinator'] == Coordinator_X]
    train, test   = train_test_split(Data_Filtered, test_size= .3, random_state=94)
    train_final   = train_final.append(train)
    test_final    = test_final.append(test)
```

