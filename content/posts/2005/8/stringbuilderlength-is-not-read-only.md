+++
title = "StringBuilder.Length is not read-only!"
date = 2005-08-23T12:00:00.000-05:00
updated = 2005-08-23T12:02:01.856-05:00
draft = false
url = '/2005/08/stringbuilderlength-is-not-read-only.html'
tags = []
+++

When you wander the blogs, sometimes the comments are more interesting or useful than the posting. Today's case-in-point is [James Curran](http://www.honestillusion.com/)'s observation that StringBuilder.Length is not a read-only value... how many times have you built a comma (or any other delimiter) seperated string by optionally prepending the delimiter in the loop, or trimming off the leading (or trailing) delimiter after the loop. James says just truncate the final delimiter by adjusting Length! [góða nótt. My Ajax.NET Library](http://jason.diamond.name/weblog/2005/07/06/my-ajax-dot-net-library#comment-400)
