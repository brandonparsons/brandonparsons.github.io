---
layout: post
title: "Careful... Nokogiri escapes HTML!"
tags:
  - web development 
description: "If you are using Nokogiri to render XML (e.g. RSS feeds) - be careful! It will automatically escape HTML, leaving you with some funny-looking RSS feeds!"
---

Hopefully this will save you hours of pain... This might be common knowledge, but I was unaware that Nokogiri escapes some text characters when rendering XML!

I was trying to use Nokogiri XML builder to put together an ATOM feed, and spent hours trying to bug fix what I thought was Markdown escaping HTML characters.  It turns out that Nokogiri escapes these as well....

### The solution

Let's cut to the chase.  Here's what you need to do to force Nokogiri to render raw XML (no escaping).

```
xml.content(:type => "html") do
  xml << html # Have to do it this way to prevent autoescaping
end
```
