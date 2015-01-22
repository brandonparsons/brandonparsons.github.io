---
title: An automated Ruby/Rake task for pinging search engines with your sitemap
shortname: "Ruby/Rake task for pinging search engines"
tags:
  - web development
  - tools
description: "Quit submitting your sitemap to Google.  This Rake task will automatically push your sitemap for indexing, and can be extended to other search engines."
---

Looking for a way to automate pinging Google and Bing with your updated Sitemap from the command line?  Useful if you are running a Jekyll or Ruhoh blog....

### The Code

```ruby
require 'open-uri'

namespace :website do

  desc "Pings search engines with sitemap"
  task :ping_google do
    http = open("http://www.bing.com/webmaster/ping.aspx?siteMap=http://brandonparsons.me/sitemap.xml")
    http = open("http://www.google.com/webmasters/sitemaps/ping?sitemap=http://brandonparsons.me/sitemap.xml")
  end
end
```
