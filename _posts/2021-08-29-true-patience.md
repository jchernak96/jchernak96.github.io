---
layout: post
title: Evaluating Batter Effective Patience
excerpt: "Using pybaseball and xRV to measure which batters swing at the best pitches"
modified: 2/29/2016, 9:00:24
tags: [Python, Patience]
comments: true
category: bball
---

*Just started this project, more coming soon.

The topic of batter patience and swing selection has more or less focused on outside zone swing % or chase rate. While this provides a general overview of how strong a batters eye is, it ignores some valuable aspects of swing selection such as:

- What pitches is a batter swinging at? Is he being too patient and not swinging at pitches in his 'blast zone'?
- Is a batter being to aggressive early in the count on fringe pitches that have a low expected run value?
- Was swinging a good decision based on count, pitch location, pitch type, and his historical performance against that pitch in that location?

These aspects of swinging are important and can inform us about the true effectivness of a batters swing decisions. In addition, this venture can provide a method of evaluation for players to potentially change their plate approach under certain circumstances. In this post I find the xRun Value of events in 2021 (only interested in strikeouts and walks for this analysis though), find the xRun Value of each count, and find the xRun Value of pitch locations on a league level and individual level. With this, we can recommend a swing decision on each pitch for each batter and evaluate how closely they follow what the model says is the "optimal strategy".

### Data Prep 

The first step in this project was acquiring pitch level data from statcast. Fortuantly, the pyBaseball savant scraper works very well and allowed me to quickly grab all data from 2021. Once the data was loaded I needed to find the run expectancy values for each game state situation. This is pretty simple as we just have to create a few columns that inform us of the game state (outs, base situations), the max runs scored by the batting team in each inning, and the number of runs scored from the time of an at bat to the end of the inning.

```{r}
#Load packages
from itertools import groupby
import numpy as np
import re
import pandas as pd
from baseball_scraper import statcast
from pandas.core.algorithms import mode

#Data
data = statcast(start_dt='2021-08-05', end_dt='2021-09-24')

#create function for columns prepare for run expectancy calcs
def base_out_game_state(df):
    df['final_pitch_at_bat']   = df.groupby(["game_pk","at_bat_number","inning_topbot"])['pitch_number'].transform('max')
    df['bat_score_end_inning'] = df.groupby(["game_pk", "inning", "inning_topbot"])['bat_score'].transform('max')
    df['runs_to_end_inning']   = df['bat_score_end_inning'] - df['bat_score']
    df['outs_situation']       = df['outs_when_up'].astype(str) + ' outs'
    df['1_b_situation']        = np.where(df[['on_1b']].isnull(), '_', '1b')
    df['2_b_situation']        = np.where(df[['on_2b']].isnull(), '_', '2b')
    df['3_b_situation']        = np.where(df[['on_3b']].isnull(), '_', '3b')
    df['base_out_state']       = df['outs_situation'] + df['1_b_situation'] + df['2_b_situation'] + df['3_b_situation']
    return(df)
```

The first step in this project was acquiring pitch level data from statcast. Fortuantly, the pyBaseball savant scraper works very well and allowed me to quickly grab all data from 2021. 

Another data cleaning part of this model is grouping a few events into more general event types. This primarily involved very specific out types (such as a sac fly) and moving those into a general field out designation. This was pretty straightforward and once completed I was able to select the columns from my data frame needed for determining the 2020 run value of events. 

### Determining Event Level xRV

When a batter is faced with a 3-0, 0-2, or full count, there exists two events that can happen: strikeout in same cases or walk in others. This mattters in our model because if a batter is presented with one of these counts, we need to know the increase in xRV of taking a pitch (in some cases resulting in a walk) or the decrease in xRV from striking out. Thus, a necessary step in this analysis is finding the xRV of all events (which includes strikeouts and walks). 



