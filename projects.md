---
title : Projects
description: A brief overview of some of my personal projects.
permalink: /projects/
---

For various reasons, I'm leaving all details about my engineering "projects" off of this website.  *(Can't see why it would be particularly interesting for anyone to read in any case!)*

### [RetirementPlan.io](http://www.retirementplan.io) ###

The first version of this software was my first real web application.  It started as a means to learn [Ruby on Rails][Rails] and scratch a personal itch related to finance, retirement planning and statistics.  It grew from there into a full-fledged customer-facing application, incorporating individual user input, modern financial research and portfolio tracking.

I launched this application as a true startup corporation, and was even featured as "Startup of the Week" in the Calgary Herald. Unfortunately the Canadian Financial Regulations are not quite ready for 100% algorithmic financial advice (unlike in the U.S.) so I had to shelve this project.  Good learning experience in any case!

[Rails]: http://rubyonrails.org

The math side of the application was fairly complex... I ended up using all of the following in some fashion:

- Inverse Normal & Student's T distributions
- Cholesky decompositions
- Confidence Intervals
- Monte Carlo statistical simulations
- Regression and analysis of historical ETF returns
- Modern Portfolio Theory (portfolio variance & expected return)
- Reverse portfolio optimization

The web application itself was also challenging (especially for a beginner to the scene), and has morphed into a complex machine as my experience with Rails and web development grew.

I have obviously also tied some of the marketing pieces together for this application - namely Google Analytics, Mailchimp, transactional e-mail and some other minor integrations.

### [Automated Campaign/UTM Parameter Tracking][post] ###

This was a quick weekend-day project I used to learn Google Apps Scripts and solve an issue I'd been having around that time.  I had been shortening a lot of links where I was adding specific UTM parameters in order to track incoming traffic in Google Analytics.  I created this script/spreadsheet to allow you to keep track of what UTM parameters you have been using (to ensure consistency) and quickly shorten new links as they come up.

The spreadsheet/script are freely shared on Google Apps.  Check out my instructions for [how to set up the UTM parameter tracking spreadsheet][post].

[post]: /2012/external-campaign-link-tracking/
