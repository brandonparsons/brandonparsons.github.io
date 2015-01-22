---
layout: post
title: Setting up social tracking in Google Analytics
tags:
  - tools
  - javascript
  - analytics
---

I recently implemented social tracking on my blog - thought I would share the code.  You can obviously find this elsewhere on the internet, but thought it would be handy for me to have in one place for other projects.

## What I track

I track the following events:

- Visits to social profiles ([Twitter][twitter]/[Google Plus][googleplus]) as an event
- Social shares (tweets, facebook 'likes' and Google +1's)
- Twitter follows (via a twitter button)

The last two items are tracked using Google's "new" social tracking framework.  Instead of doing it all via events, you can now take advantage of advanced integration in Google Analytics to try to measure the impact of social networks on your site.
 
[twitter]: https://twitter.com/bkparso
[googleplus]: https://plus.google.com/u/0/114865027828933883024

## The Code

### Social Profile Link Events

Here is the code I use to push profile visits to Google Analytics as [Events][events]:

[events]: https://developers.google.com/analytics/devguides/collection/gajs/eventTrackerGuide

```javascript
  $("#twitter-link").click(function() {
    _gaq.push(['_trackEvent', 'social', 'twitter-profile-visit']);
  });

  $("#googleplus-link").click(function() {
    _gaq.push(['_trackEvent', 'social', 'gplus-profile-visit']);
  });

  $("#linkedin-link").click(function() {
    _gaq.push(['_trackEvent', 'social', 'linkedin-profile-visit']);
  });

  $("#github-link").click(function() {
    _gaq.push(['_trackEvent', 'social', 'github-profile-visit']);
  });

  $("#feed-link").click(function() {
    _gaq.push(['_trackEvent', 'social', 'rss-subscribe']);
  });
```

Simply attach CSS ID's to your social profile buttons and you are off to the races (e.g. `<a href="https://twitter.com/intent/user?screen_name=bkparso" id='twitter-link'><i class='icon-twitter-sign'></i></a>`).

Next up is the code for integrating social shares with Google Analytics.

### Social Share Integration

This is a bit more complicated than the event tracking as a result of needing to interact with the IFrames these buttons drop onto your site.

First you'll need to put some asynchronously executed javascript near the top of your page:

```html
<!-- For Twitter Widgets -->
<script type="text/javascript" charset="utf-8">
  window.twttr = (function (d,s,id) {
    var t, js, fjs = d.getElementsByTagName(s)[0];
    if (d.getElementById(id)) return; js=d.createElement(s); js.id=id;
    js.src="//platform.twitter.com/widgets.js"; fjs.parentNode.insertBefore(js, fjs);
    return window.twttr || (t = { _e: [], ready: function(f){ t._e.push(f) } });
  }(document, "script", "twitter-wjs"));
</script>

<!-- For Google +1 Button -->
<script type="text/javascript">
  window.___gcfg = {
    lang: 'en-US'
  };

  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>

<!-- For Facebook 'Like' button -->
<div id="fb-root"></div>
<script>(function(d, s, id) {
  var js, fjs = d.getElementsByTagName(s)[0];
  if (d.getElementById(id)) return;
  js = d.createElement(s); js.id = id;
  js.src = "//connect.facebook.net/en_US/all.js#xfbml=1&appId=330932300336907";
  fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>
```

You then drop in your standard social buttons HTML (you will want to use the XFBML version of the Facebook like button):

```html
<table>
  <tbody>
    <tr>
      <td>
        <script src="http://connect.facebook.net/en_US/all.js#xfbml=1"></script>
        <fb:like send="false" layout="button_count" width="450" show_faces="false" width="100"></fb:like>
      </td>
      <td>
        <a href="https://twitter.com/share" class="twitter-share-button" data-lang="en" data-via="bkparso" data-count="none">Tweet</a>
      </td>
      <td>
        <g:plusone href='http://brandonparsons.me/about' annotation='none'></g:plusone>
      </td>
    </tr>
  </tbody>
</table>
</div>
<br />
<p style='padding-left:73px;'><a href="https://twitter.com/bkparso" class="twitter-follow-button" data-show-count="false">Follow @bkparso</a></p>
```

And now the complicated JavaScript (thanks Google!):

```javascript
  FB.Event.subscribe('edge.create', function(targetUrl) {
    _gaq.push(['_trackSocial', 'facebook', 'like', targetUrl]);
  });

  FB.Event.subscribe('edge.remove', function(targetUrl) {
    _gaq.push(['_trackSocial', 'facebook', 'unlike', targetUrl]);
  });

  FB.Event.subscribe('message.send', function(targetUrl) {
    _gaq.push(['_trackSocial', 'facebook', 'send', targetUrl]);
  });

  function extractParamFromUri(uri, paramName) {
    if (!uri) {
      return;
    }
    var regex = new RegExp('[\\?&#]' + paramName + '=([^&#]*)');
    var params = regex.exec(uri);
    if (params != null) {
      return unescape(params[1]);
    }
    return;
  }

  function trackTwitterShare(intent_event) {
    if (intent_event) {
      var opt_pagePath;
      if (intent_event.target && intent_event.target.nodeName == 'IFRAME') {
            opt_target = extractParamFromUri(intent_event.target.src, 'url');
      }
      _gaq.push(['_trackSocial', 'twitter', 'tweet', opt_pagePath]);
    }
  }

  function trackTwitterFollow(intent_event) {
    if (intent_event) {
      var label = intent_event.data.user_id + " (" + intent_event.data.screen_name + ")";
      //pageTracker._trackEvent('twitter_web_intents', intent_event.type, label);
      _gaq.push(['_trackSocial', 'twitter', 'follow']);
    };
  }

  //Wrap event bindings - Wait for async js to load
  twttr.ready(function (twttr) {
    //event bindings
    twttr.events.bind('tweet', trackTwitterShare);
    twttr.events.bind('follow', trackTwitterFollow);
  });
```

As a final gotcha - since you are using the XFBML version of the Facebook like button, you'll need the following in your HTML tag:

```html
<html lang="en" xmlns="http://www.w3.org/1999/xhtml" xmlns:fb="http://ogp.me/ns/fb#">
```

(Note the `xmlns:fb` entitiy).

## And we're done!

I've tested this out on my website and it all appears to work.  You'll obviously need to adjust some of the code *(e.g. to make sure your Facebook App ID is yours, not mine!)*.

Let me know if you see any issues with this.
