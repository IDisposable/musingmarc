+++
title = "DateTime is NOT UTC."
date = 2005-07-08T12:20:00.000-05:00
updated = 2006-06-14T13:16:27.020-05:00
draft = false
url = '/2005/07/datetime-is-not-utc.html'
tags = []
+++

It is funny the things you learn from searches. Today [I learned that System.DateTime is not based in UTC](http://www.angrycoder.com/article.aspx?cid=5&y=2004&amp;m=5&d=13). In fact it is based in TAI, which is the real "atomic" time without leap-seconds. Now I realize most people don't know or care about the difference, but I have an obsession with dates, times and all things chronological. Here are a few of my favorite tidbits:*   [Developing Time-Oriented Database Applications in SQL](http://www.amazon.com/exec/obidos/ASIN/1558604367/marcsmusing0a-20) by Richard T. Snodgrass is a wonderful book about doing proper SQL database designs and queries when the chronology of data points, the posting of them, or the subsequent view "in time" of data is important. This area is extremely difficult to get right consistently and this book walks you through the evolution of several **real world** systems to get it right as the needs are evolved. It is out of print, but you can [download the book in PDF form here.](http://www.cs.arizona.edu/people/rts/)
*   Why Daylight Savings Time is Non-intuitive [Raymond Chen blog](http://weblogs.asp.net/oldnewthing/archive/2003/10/24/55413.aspx)
*   Coding Best Practices Using DateTime in the .NET Framework [Dan Rogers MSDN article](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dndotnet/html/datetimecode.asp). This is a great place to start, as it addresses most of the issues you will likely run into.
*   New DateTime Best Practices Article [BCL Team Article](http://www.gotdotnet.com/team/clr/bcl/TechArticles/techarticles/datetimeguidelines.doc). This is a very good summary of the issues you can run into and how to deal with them.
*   Representing Null DateTime values in code [Scott Munro blog](http://dotnetjunkies.com/WebLog/scottmunro/archive/2004/03/15/9153.aspx) talks about how nulls in dates can be handled, lots of excellent links. I personally use the [Null Object](http://www.refactoring.com/catalog/introduceNullObject.html) pattern with DateTime.MinValue representing _start_ dates and DateTime.MaxValue representing _end_ dates.
*   What are the New DateTime Features in Whidbey [BCL Team blog](http://blogs.msdn.com/bclteam/archive/2005/03/15/396440.aspx) gives the changes in .Net 2.0 for DateTime. I recommend [all of Anthony Moore's postings](http://blogs.msdn.com/search.aspx?q=Anthony+Moore&p=1) as they are very informative and he owns System.DataTime.
More links to follow once CodeProject is back online.
