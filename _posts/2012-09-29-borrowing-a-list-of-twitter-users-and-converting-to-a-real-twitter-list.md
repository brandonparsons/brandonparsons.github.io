---
layout: post
title: Borrowing a List of Twitter Users and Converting to a Real Twitter List!
shortname: Programmatically Creating a Twitter List
tags:
  - tools
description: Using JQuery & the Twitter gem to programmatically create a list
---

Startup North [recently][list] came up with a **solid** resource for the Canadian startup scene.  I thought I'd do my part and put in a bit of elbow grease to help out.  They put together a list of "Entrepreneurs, Thought Leaders, Gadflys and Others", which I thought would be great as an actual Twitter list.  Hopefully StartupNorth is ok with this!

## The How

### Grab the List of Twitter Users

I grabbed some of the raw HTML from their website, and took just the relevant `<ul>` to make things simpler.  JQuery to the rescue in order to extract the Twitter user names:

```coffeescript
entries = $("#thought-leaders li a.twitter-follow-button")

entries.each( (index,element) ->
    link = element.href
    match = /.*twitter.com\/(.*)/i.exec(link)
    console.log match[1]
    )
```

I then used [the Twitter gem][twitter-gem] to automatically insert all of these users into a list (you'll need to generate your own app at [Twitter Developers][twit-dev]):

```ruby
t = Twitter.configure do |config|
  config.consumer_key = 'foo-bar'
  config.consumer_secret = 'foo-bar'
  config.oauth_token = 'foo-bar'
  config.oauth_token_secret = 'foo-bar'
end
```
Then it was a simple matter of using the gem to add to the list (created manually):

```ruby
  t.list_add_members("bkparso", "sunorth-thought-leaders", ["aprildunford", "johnphilipgreen", "zakhomuth", .....])
```

[list]:http://startupnorth.ca/2012/09/13/dont-panic
[twitter-gem]:https://github.com/sferik/twitter
[twit-dev]:http://dev.twitter.com

## The Where

Follow the list [here][twit-list] for some startup goodness!

[twit-list]:https://twitter.com/i/#!/bkparso/sunorth-thought-leaders
