---
layout: post
title: Scraping Pro Football Reference (PFR) Injury Data
excerpt: "Developing a web scraper with Rvest to collect and organize PFR injury data"
modified: 2/29/2016, 9:00:24
tags: [Rvest, R, Scraping, Injuries]
comments: true
category: blog
---

  Every week, NFL players get injured with varying degrees of severity. Not much data is available to the public related to injuries, weeks missed due to injury, or the type of injury sustained. The best source of public injury info can be found on [PFR's individual team injury page](https://www.pro-football-reference.com/teams/nwe/2020_injuries.htm). While informative, the layout is not great for analysis and it is difficult to make any conclusions about team injuries. This data goes back to 2009 for every NFL team, in this post I develop a function to scrape injury data along with the underlying injury descriptions that are only visible when hovered over.

## Creating a List of URLs to Scrape

If you opened the linked page above, you would notice that each URL only contains a specific team and specific season. This means that to collect all of the injury data we will need a list of URLs for our future scraping function to run through. The key points in the URL are the <strong>PFR team abbreviation</strong> and the <strong>season of interest</strong> (data on PFR only goes back to 2009). First, we can start with assembling the list of team abbreviations. This is a little different than NFL Fast R abbreviations so if you don't recognize an abbreviation don't worry (Titans are oti which very confusing). 

<script src="https://gist.github.com/jchernak96/c8db3fb82a1255f3a64e4a9967ef0b71.js"></script>

Great! Now we can just create a list of years (2009 - 2020) and then write a loop that modifies the base PFR URL with all possible combinations of seasons and team abbreviations. One the loop finishes I perform some basic cleaning of the URLs for the web scraping function.

<script src="https://gist.github.com/jchernak96/9c6660319ca8361dbe2f85c780b1df75.js"></script>

And voila, we have all of the URLs we need to scrape PFR. Now comes the hard part, the table provided by PFR could be scraped by the html_table function in Rvest but that results in the scraper missing what the specific injury was for each player. Notice that on the [team injury page](https://www.pro-football-reference.com/teams/nwe/2020_injuries.htm), one has to hover over the player's name and week to see what injury the player had. This is very important information, so the easiest approach of html_table can't be used. Thus, I had to employ a more complex approach that required html_nodes. 

## Developing the Injury Scraper

There are a few distinct features of the scraper that were created to accomodate inconsistencies between each team page. 






- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.



### Gists via GitHub Pages

Vestibulum id ligula porta felis euismod semper. Nullam quis risus eget urna mollis ornare vel eu leo. Donec sed odio dui.

{% gist 5555251 gist.md %}

Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Nullam quis risus eget urna mollis ornare vel eu leo. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Donec sed odio dui. Vestibulum id ligula porta felis euismod semper.

### Analysis of 2020 Injuries

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](https://pbs.twimg.com/media/Eu8fXWXXYAMXfb3?format=png&name=medium)

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/poole/poole/issues/new">Open an issue.</a>
