+++
title = "SQL Date Processing - First/Last Day of a month"
date = 2006-02-16T10:59:00.000-06:00
updated = 2007-11-19T00:17:51.681-06:00
draft = false
url = '/2006/02/sql-date-processing-firstlast-day-of.html'
tags = ["DATEADD","SQL","DATEDIFF","DateTime","end of month"]
+++

[Oren Eini](http://www.ayende.com/) posted a nice bit about SQL. It's easy to get the first or last day of a specific month and year without doing all the silly CASTing about. (slightly tweaked in LastDayOfMonth to avoid a redundant `DATEADD`)

```
CREATE FUNCTION FirstDayOfMonth(@year int, @month int)
   RETURNS DATETIME AS
BEGIN
   RETURN DATEADD(MONTH, @month - 1, DATEADD(YEAR, @year - 1900, 0))
END
 
CREATE FUNCTION LastDayOfMonth(@year int, @month int)
   RETURNS DATETIME AS 
BEGIN
   RETURN DATEADD(DAY, -1, DATEADD(MONTH, @month, DATEADD(YEAR, @year - 1900, 0)))
END
```

This works because 0 as a DateTime equals 01/01/1900, so subtracting 1900 from the year and one from the month gives the correct values.

_\[Via [http://www.ayende.com/Blog/PermaLink,guid,23fefdfe-22f1-4649-8e12-f36d06cd54cc.aspx](http://www.ayende.com/Blog/PermaLink,guid,23fefdfe-22f1-4649-8e12-f36d06cd54cc.aspx)\]_

**UPDATE:** I've added a complete list of the ways to get first/last/next values for a date/week/month/quarter/year in SQL [here](http://musingmarc.blogspot.com/2006/07/more-on-dates-and-sql.html). (I also removed the horrid syntax coloring of this post, sorry).

---
### Comments:
#### Why the black background? It makes prints of the s...
[Anonymous]( "noreply@blogger.com") - <time datetime="2008-07-11T13:33:00.000-05:00">Jul 5, 2008</time>

Why the black background? It makes prints of the screen nasty.
<hr />
#### You print things? Tree killer :)  
  
Seriously...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-07-11T16:21:00.000-05:00">Jul 5, 2008</time>

You print things? Tree killer :)  
  
Seriously, though, I could make a print-based CSS.
<hr />
