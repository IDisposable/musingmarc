+++
title = "Maintaining thread safety is hard"
date = 2005-08-12T10:54:00.000-05:00
updated = 2005-09-19T13:32:02.773-05:00
draft = false
url = '/2005/08/maintaining-thread-safety-is-hard.html'
tags = []
+++

Even Microsoft's best have issues.. First they mess it up in .Net 1.1, then they rewrite it wrong in .Net 2.0. I wonder if they need another [code reviewer](/2005/01/my-development-and-deployment-strategies). The 2.0 version has three bugs now! First, it will reset the `_inTrim` flag on any exception, even if it didn't set the flag. Second, it will reset the `_inTrim` flag right before the second `NeedsTrim()` check, even if someone else has already set it (and presumably is using that flag). Lastly, it will now silently eat exceptions when trimming. How many times have you seen a `catch` when a `finally` was the right thing. Go [vote on Ladybug](http://lab.msdn.microsoft.com/productfeedback/viewfeedback.aspx?feedbackid=64a8cd76-0d1b-4c50-9cbc-1894fde44a4f) to get this fixed.
