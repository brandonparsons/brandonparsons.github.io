---
layout: post
description: A brief primer on Monte Carlo Simulation
category: finance
title: What is Monte Carlo Simulation?
tags: 
  - finance
  - statistics
---

What do nuclear chain reactions, infrastructure projects, oil asset valuations, and retirement simulations have in common? You can probably guess - **Monte Carlo simulations**.

Monte Carlo Simulation is a powerful tool that is used in science, business and finance.  Originally used to model nuclear chain reactions during the Manhattan Project (creation of the atomic bomb), it has since grown into widespread use as a computational tool:

- Physical Sciences: Modeling complex natural pheonomena (e.g. molecular dynamics) where closed-form solutions are impractical or impossible
- Engineering & Business: Used to model uncertain business outcomes, ranging from predicting possible product revenues to risk quantification in oil & gas
- Finance: Commonly used to model the stochastic nature of the stock market, economic conditions and unknown outcomes of people's lives

## General Process ##

Say you knew for certain precisely how many widgets your company would sell this year.  How would you model profits?  Well, you'd simply determine your net margin on each individual widget, multiply by the number of widgets sold, and voila - profit!

However as well know, life is not that simple.  The popularity of products rise and fall, people's demands for certain items change throughout the season.  Say that the number of products you'd sell is completely random, bounded between two values.  How would you model your profits then?

If you were strong at math, perhaps you could create a closed-form profit equation which takes the random unit sales and boundaries into account to generate the possible range of profit outcomes.  However, if you were quicker at Excel, you might simply randomly specify the number of unit sales, calculate profit, and by doing this many, many times determine the expected outcome.

This is the heart of Monte Carlo Simulation.  Any number of inputs to a process can be modeled as randomly distributed variables ([uniform]('http://en.wikipedia.org/wiki/Uniform_distribution_(continuous)'), [normal](http://en.wikipedia.org/wiki/Normal_distribution), [lognormal](http://en.wikipedia.org/wiki/Log-normal_distribution), [skewed](http://en.wikipedia.org/wiki/Skewness)....), and with enough trials the expected range of values will start to emerge.

The more trials that are performed, the more certain of the result you can be.

## How it is used at RetirementPlan.io ##

From years of observing the stock market, it is standard practice in the finance world to model asset returns as normal distributions (or equivalently, asset prices following a lognormal distribution).  

Under this assumption (and that the historical values or a function thereof might predict the range of future values), it becomes clear that Monte Carlo Simulation can be used to generate potential outcomes of your portfolio over a number of years.

For a 3-asset portfolio, you would collect the historical mean return and volatility for each asset, make an assumption about future statistics, and generate portfolio paths based on the random outcome of asset price for each asset, through a period of time you select.

At RetirementPlan.io, we extend this to include possible outcomes for real estate values and inflation, in order to try to model the true range of outcomes that are possible throughout your lifetime.

## Why is this Important to You? ##

Have you ever been to a financial adviser where they stated that you "should be able to expect a return of 6-8% in the stock market", and proceed to plot a straight line up and to the right in order to determine how much you need to retire?

When has the stock market ever gone straight up, at a consistent rate, through a number of years? (Or any time period for that matter?) **Never**.  It's important that you take into account the potential volatility of your portfolio when planning for retirement.

Think about all the people who had saved up just enough for their retirement in 2008/2009.  That certainly was not predicted by straight-line models, and was obviously more impactful to the potential retirees than a market crash early in their careers (when they had room to grow it back) or very late in their life when they had less use for their money.

Not only do you need to consider the ups and downs of the stock markets, but also the timing.  *This* is the power of Monte Carlo Simulation.
