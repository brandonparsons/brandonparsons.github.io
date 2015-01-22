---
layout: post
title: Setting up an R Environment On Mac Mountain Lion
shortname: An R Environment on Mac Mountain Lion
tags:
  - finance
  - tools 
---

R is an excellent language for finance, statistics and general science.  Here's a quick setup guide to get you going.

### Setup

1.  [Install R from the CRAN website][R]
2.  Get [RStudio][] (if IDE's are your thing).  This will provide syntax support, graphical representations of your data and R Objects as well as other handy features.
2.  Get a package `install.packages('quantmod')`
3.  Load the library `library(quantmod)`
4.  Load some stocks `loadSymbols(c('^XAU', 'GLD', 'IAU'))`
5.  Chart! `chartSeries(GLD)`

[R]: http://cran.r-project.org/bin/macosx/
[RStudio]: http://rstudio.org

### Packages

Get some other useful packages:

* `reshape`
* `ggplot2`
* `plyr`
* `RGoogleDocs` (see http://stackoverflow.com/questions/1295955/what-is-the-most-useful-r-trick for an example)

Note that [R][] has powerful [CSV import/export tools][tools].  Plays nicely with Excel!

[tools]: http://cran.r-project.org/doc/manuals/R-data.html

### Console vs. IDE

Like many other programming languages, whether you prefer "text editor" type interaction or an IDE is often a personal preference.  I've been on the former side with Ruby/Rails, but for R I've been leaning towards the IDE (RStudio).  So far this has been mostly for visualization of the data objects saved, as well as some syntax support.

#### R Console   ####
  
![Console Environment](/assets/article_images/2012/r-console.png)

#### [RStudio][]  
![IDE Environment](/assets/article_images/2012/r-studio.png)

__________
  
  
  
More to follow on R....
