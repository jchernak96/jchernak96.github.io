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

The topic of pitcher patience and swing selection has more or less focused on outside zone swing % or chase rate. While this provides a general overview of how strong a batters eye is, it ignores some valuable aspects of swing selection such as:

- What pitches is a batter swinging at? Is he being too patient and not swinging at pitches in his 'blast zone'?
- Is a batter being to aggressive early in the count on fringe pitches that have a low expected run value?
- Was swinging a good decision based on count, pitch location, pitch type, and his historical performance against that pitch in that location?

These aspects of swinging are important and can inform us about the true effectivness of a batters swing decisions. In addition, this venture can provide a method of evaluation for players to potentially change their plate approach under certain circumstances. 
