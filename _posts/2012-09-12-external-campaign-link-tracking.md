---
title: External Campaign Link Tracking via UTM Parameters
tags:
  - tools
  - google docs
  - javascript
  - web development 
---

I created a Google Apps Script (for a spreadsheet) that helps you keep track of external campaign UTM parameters, and your shortened links.  For your sharing pleasure!

## Campaign Tracking via UTM Parameters

If you are using Google Analytics, you are already tracking your incoming web traffic.  Without lifting a finger, you can tell generally where traffic is originating by looking at your referrers (for example, you can tell how Facebook traffic differs from Twitter traffic).

However, if you want more detailed information on where people are coming from (e.g. my profile link in Twitter, or my status updates?) you should be using campaign tracking parameters.

There is a great overview of using Google UTM parameters over [at KissMetrics' blog][1] so I won't rehash that here.  An important point to note is that you should not be using these URL parameters for internal campaigns (i.e. is my sidebar link or footer link leading to more conversions?) - you should use the methods [shown here][2].

[1]: http://blog.kissmetrics.com/how-to-use-utm-parameters/
[2]: http://cutroni.com/blog/2010/03/30/tracking-internal-campaigns-with-google-analytics/

## My Tool

In using UTM parameters for some of my web projects, it was becoming a bit of a pain maintaining specific campaign parameters for different uses, tracking click analytics and using a URL shortening service.  As such, I built a Google Apps Spreadsheet which automatically keeps track of your campaign parameters, shortens URLs using Google's URL shortener [goo.gl][goo.gl], and can even pull click data for a quick check while you are generating your URLs.

[goo.gl]: http://goo.gl

## Usage

Feel free to use, modify, change this spreadsheet/script in any way that you need.  All I ask is that you link to this page somewhere on your site in order to help spread the word!

## Instructions

1.  [Get the campaign tracking spreadsheet][3] and copy a version to your Google Drive  
2.  Input the URL at your site that you intend to link to.  
![Update Link Information](/assets/article_images/2012/campaign-doc-1.png)
3.  Go through the source, medium and campaign lists.  
![Source, medium and campaign lists](/assets/article_images/2012/campaign-doc-2.png)  
Feel free to add whatever you plan to use in here.  These values will show up in dropdown boxes on the main page.  
4.  Click on the "Create Link!" button.  Hit this button whenever you want to generate a new shortened link.  
5.  You will likely need to authorize the script to run.  
![Authorize the script](/assets/article_images/2012/campaign-doc-3.png)  
Click "Grant Access" in the following screen  
6.  You can now automatically create shortened links at will by clicking the "Create Link!" button!  

[3]: https://docs.google.com/spreadsheet/ccc?key=0Ap94IzrZKBS7dEZXcUJTc21xWUJleEJXeC1IaU9haGc#gid=0
