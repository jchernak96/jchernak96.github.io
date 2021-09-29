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

The first step in this project was acquiring pitch level data from statcast. Fortuantly, the pyBaseball savant scraper works very well and allowed me to quickly grab all data from 2021. Once I had this data, I did some basic re-coding of pitches to move less common pitches (such as a knuckle curve or splitter) into more common designations (such as curveball or changup) under the assumption that these pitches have similar shapes and areas of effectivness. 

Another data cleaning part of this model is grouping a few events into more general event types. This primarily involved very specific out types (such as a sac fly) and moving those into a general field out designation. This was pretty straightforward and once completed I was able to select the columns from my data frame needed for determining the 2020 run value of events. 

