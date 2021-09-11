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

As outlined in my previous post, I scraped PFR for their coordinator level data and output it into a format that can be quickly joined to PBB data. I loaded this data into Python and only selected the last 6 years of data (otherwise my computer will run very very slow). 

```
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








