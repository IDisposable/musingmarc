+++
title = "A much better way to handle CSS hacks"
date = 2005-08-24T14:17:00.000-05:00
updated = 2005-08-24T14:18:01.663-05:00
draft = false
url = '/2005/08/much-better-way-to-handle-css-hacks.html'
tags = []
+++

When you're doing CSS in the real world, you have to handle the CSS bugs in various browsers. But don't embed them in your real style-sheets and clutter everything up. Rather, have a single CSS that has all the work-arounds, and decorate your `<html>` tag with multiple classes that pull in all the work-around rules.

Next, use javascript to handle the injection of those classes automatically at page load! [When bugs become patterns - A look at CSS Hacks](http://spaces.msn.com/members/siteexperts/Blog/cns!1pNcL8JwTfkkjv4gg6LkVCpw!1805.entry)
