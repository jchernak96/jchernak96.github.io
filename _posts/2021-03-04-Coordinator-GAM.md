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

This process was very similar to my previous psot regarding injury scraping on PFR. I followed the same process of constructing URLs for each page




