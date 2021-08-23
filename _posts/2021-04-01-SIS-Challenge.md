---
layout: post
title: SIS Analytics Competition Submission
excerpt: "Evaluating route combos and coverages with mixed models."
modified: 2/29/2016, 9:00:24
tags: [Mixed Effects, Competition, Presentation]
comments: true
category: blog
---

Sports Info Solutions (SIS) posed the following prompt as a part of their 2021 analytics competition: Which route combinations were most popular in the NFL in 2020? Of these route combinations, which perform best against each coverage type?

Not a terribly complicated question, but developing a statistical solution without tracking data is a complicated problem. For transparency, each row in the provided data set looks something like this for one play:

<table>
  <thead>
    <tr>
      <th>Play ID</th>
      <th>Receiver Name</th>
      <th>Route</th>
      <th>Side of Center</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>012854</td>
      <td>Julian Edelman</td>
      <td>Dig</td>
      <td>L</td>
    </tr>
    <tr>
      <td>012854</td>
      <td>Jacobi Meyers</td>
      <td>Drag</td>
      <td>L</td>
    </tr>
    <tr>
      <td>012854</td>
      <td>Devin Asiasi</td>
      <td>Dig</td>
      <td>R</td>
    </tr>
  </tbody>
</table>

This layout is extended to the rest of the 2020 season to create a database of plays. My first step in addressing the promt was laying out assumpstions and cleaning the data.

## Assumptions and Cleaning

The first assumption made in this analysis is what defines a route combo. I developed the following description: <strong> 2 or more routes that are designed within the structure of the play to interact with each other. This includes patterns that begin on opposite sides of the center (for example mesh) </strong>. This is a relatively simple definition that provides enough guidance when manipulating the data when cleaning but still provides direction when answering the prompt. Once this workign definition was established, I moved onto cleaning the route data. This began with reclassifying routes into more general bins. The idea behind this assumption was that I sought to capture the general route concept given we can't see the play. Therefore, we are not capturing very subtle differences in the data and having a sample size issue when we later group route combos together. 

Additionally, routes that were classified as “Flat – Left”, “Flat – Right”, “Swing – eft”, or “Swing – Right” and did not involve fast motion were reclassified to just 
“Flat” or “Swing”. Those routes that did involve a fast motion were also reclassified to “Flat” or “Swing”, but their side of center was changed if their side of center was opposite from the described “left” or “right” action. (E.g., a player who: 1.) ran a “Flat – Left”, 2.) was initially marked as a “R” side  of center & 3.) was marked as involved in fast motion, was reclassified as a “Flat” with an “L” side of center and their order from out to in was set as the innermost route runner. 

In theory, we could just split each play into routes run onto the left side of the center and right side of the center and analyze both sides as separate route combos. But this ignores route combos that involve crossing patterns. To address this, I classified the following routes as “crossing routes” if run by a slot WR or tight end: "Drag", "Dig", "Deep Cross", "Post", "Sit Over Middle”. If a play had a crossing route from a slot WR or TE on both sides of the field, then I considered this play a crosser and extracted only the crossing routes from the play. For example: Left SOC WR #1 has a “Go” and is aligned at “WR”, WR #2 has a “Drag” is aligned at “SWR” , WR #3 has a “Dig” and is aligned at “SWR”. Right SOC WR # 1 has a “Drag” is aligned at “SWR”. We would mark this play as a crosser and extract the two drags and the dig while ignoring the “Go”. The process of extracting crossing routes along with cleaning the data was very tedious and long so I omitted it from the post here but it is available [on my GitHub](https://github.com/jchernak96/AnalyticsChallenge2021/blob/main/Submissions/jtchernak%40comcast.net/R/Analytics_Challenge_Cleaning.rmd). 

## What Route Combo's Were Most Popular?

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Gists via GitHub Pages

Vestibulum id ligula porta felis euismod semper. Nullam quis risus eget urna mollis ornare vel eu leo. Donec sed odio dui.

{% gist 5555251 gist.md %}

Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Nullam quis risus eget urna mollis ornare vel eu leo. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Donec sed odio dui. Vestibulum id ligula porta felis euismod semper.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](http://placehold.it/800x400 "Large example image")
![placeholder](http://placehold.it/400x200 "Medium example image")
![placeholder](http://placehold.it/200x200 "Small example image")

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
