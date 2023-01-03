+++
title = "MSDN Product Feedback Center"
date = 2006-05-24T18:01:00.000-05:00
updated = 2007-11-19T00:13:09.142-06:00
draft = false
url = '/2006/05/msdn-product-feedback-center.html'
tags = ["Microsoft","C#","bug"]
+++

From [Eric Lippert](http://blogs.msdn.com/ericlippert/archive/2006/05/24/606278.aspx)

> Now, since this is an error case, surely we can simply fix the bug. We'd be turning a case which is presently an error into something which works. Turning broken code into working code is by definition not a breaking change -- a breaking change is turning working code into broken code, or working code with different semantics.

Someone should tell that to whoever closed this (still lurking) bug: [TypeConverter for value types (such as int, bool, short, etc.) return true to IsValid, even if the value is not valid](http://lab.msdn.microsoft.com/productfeedback/viewfeedback.aspx?feedbackid=9810731d-021a-4de1-90aa-24fdc0d259b4)
