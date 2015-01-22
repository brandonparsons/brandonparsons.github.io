---
title: Using CoffeeScript with Google Apps
tags:
  - javascript
  - google docs
description: A way to use CoffeeScript to simplify Google Apps
---

[CoffeeScript]: http://coffeescript.org
[Ruby]: http://www.ruby-lang.org/en/
[JavaScript]: http://en.wikipedia.org/wiki/JavaScript
[StackOverflow]: http://stackoverflow.com/questions/9059475/how-to-generate-global-named-javascript-functions-in-coffeescript-for-google-a

To date, I've been using [CoffeeScript][] to develop the [JavaScript][] portion of my apps.  I find that CoffeeScript keeps me from having to make as much of a mental leap between Ruby & JS every time I switch over.  Also, coming from a [Ruby][] background (and being too lazy at the moment to fully learn the intricacies of [JavaScript][]) the syntax feels comfortable.

So naturally when I took a few first leaps into writing Google Apps Scripts (for some spreadsheets I was working on) I was looking for a way to use CoffeeScript to write functions for Google Apps.  Turns out that this issue has been asked and answered over at [StackOverflow][].

The following CoffeeScript (compiled using the 'bare' option) will yield a function directly triggerable in Google Apps Scripts.

```coffeescript
myFunction = ->
  Logger.log("I do something fancy in a Google Spreadsheet!")

`function thisFunctionIsTriggerable() { myFunction(); }`
```

Compiles to:

```javascript
var myFunction;
myFunction = function() {
  return Logger.log("I do something fancy in a Google Spreadsheet!");
};
function thisFunctionIsTriggerable() { myFunction(); };
```

You should then be able to grab the ```thisFunctionIsTriggerable()``` function in the Scripts manager.
