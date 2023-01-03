+++
title = "Browser-specific CSS the clearest way possible"
date = 2006-07-21T18:17:00.000-05:00
updated = 2007-11-19T00:02:08.503-06:00
draft = false
url = '/2006/07/browser-specific-css-clearest-way.html'
tags = ["CSS","HTML"]
+++

How would you like to have all your browser-specific adjustments be trivial to apply? What if you could have a `<style>` block like this?

```
<style type="text/css">
.ie .example {
  background-color: yellow
}
.gecko .example {
  background-color: gray
}
.opera .example {
  background-color: green
}
.konqueror .example {
  background-color: blue
}
.safari .example {
  background-color: black
}
.example {
  width: 100px;
  height: 100px;
}
</style>

```

Now all you need is some JavaScript to apply the correct top-level CSS class to the `<html>` element. We'll here you go, the [CSS Browser Selector](http://rafael.adm.br/css_browser_selector/) from [Rafael Lima](http://rafael.adm.br/). Awesome simplicity!
