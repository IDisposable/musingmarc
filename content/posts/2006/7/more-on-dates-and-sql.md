+++
title = "More on DATEs and SQL"
date = 2006-07-18T13:10:00.000-05:00
updated = 2007-11-19T00:04:08.786-06:00
draft = false
url = '/2006/07/more-on-dates-and-sql.html'
tags = ["DATEADD","SQL","DATEDIFF","DateTime","end of month"]
+++

Building on [this First/Last day of month post](http://musingmarc.blogspot.com/2006/02/sql-date-processing-firstlast-day-of.html) (oddly, one of the main ways people seem to land on my blog), here are some other commonly needed SQL queries:

Beginning of period
-------------------

### Midnight of any day (i.e. truncate the time from a date)

```
SELECT DATEADD(dd, DATEDIFF(dd, 0, TheDate), 0)
```

### Midnight of today (i.e. what day is today)

```
SELECT DATEADD(dd, DATEDIFF(dd, 0, GetDate()), 0)
```

### Monday of any week

```
SELECT DATEADD(wk, DATEDIFF(wk, 0, TheDate), 0)
```

### First Day of the Month

```
SELECT DATEADD(mm, DATEDIFF(mm, 0, TheDate), 0)
```

### First Day of the Quarter

```
SELECT DATEADD(qq, DATEDIFF(qq, 0, TheDate), 0)
```

### First Day of the Year

```
SELECT DATEADD(yy, DATEDIFF(yy, 0, TheDate), 0)
```

End of period
-------------

Okay, so you need the _end_ of the month, quarter, etc. First, remember that if you are not dealing with "known to be date without time" data, you need to be very careful when doing comparisons against a date. For example, comparing a `DATETIME` column against a user-entered date is almost guaranteed to be wrong if the column has any time component. This is one of the reasons I always prefer to use a `BETWEEN` clause, as it forces me to think about the date-as-continuum issues. So, almost always, the best thing to do is compare for `<`. Now that I've justified my reasoning, I'll tell you that it is much easier to get the _next_ "week", "month", "quarter" or "year" and compare for less-than, instead of getting the _last_ value of the current "whatever". Here's the rest:

### Midnight of the next day (i.e. truncate the time from date, then get the next)

```
SELECT DATEADD(dd, DATEDIFF(dd, 0, TheDate) + 1, 0)
```

### Monday of the next week

```
SELECT DATEADD(wk, DATEDIFF(wk, 0, TheDate) + 1, 0)
```

### First Day of the next Month

```
SELECT DATEADD(mm, DATEDIFF(mm, 0, TheDate) + 1, 0)
```

### First Day of the next Quarter

```
SELECT DATEADD(qq, DATEDIFF(qq, 0, TheDate) + 1, 0)
```

### First Day of the next Year

```
SELECT DATEADD(yy, DATEDIFF(yy, 0, TheDate) + 1, 0)
```

Putting it to use
-----------------

This yields queries like this for orders due this month:

```
SELECT [ID]
FROM [dbo].[Orders]
WHERE [ShipDue] >= DATEADD(mm, DATEDIFF(mm, 0, GetUTCDate()), 0)
AND [ShipDue] < DATEADD(mm, DATEDIFF(mm, 0, GetUTCDate()) + 1, 0)
```

But wait, Marc... you said you like to use `BETWEEN`, but that query doesn't have one... that's because `BETWEEN` [is inclusive](http://www.techonthenet.com/sql/between.php), meaning it includes the end-points. If I had an Order that was due at midnight of the first day of the next month it would be included. So how do you get the appropriate value for an end-of-period? It's most certainly **NOT** by using date-parts to assemble one (but is you must, please remember that it's 23:59:59.997 as a maximum time... don't forget the milliseconds). To do it right, we use the incestuous knowledge that Microsoft SQL Server `DATETIME` columns have _at most_ a 3 millisecond resolution (something that is not going to change). So all we do is subtract 3 milliseconds from any of those end-of-period formulas given above. For example, the last possible instant of yesterday (local time) is:
```
SELECT DATEADD(ms, -3, DATEADD(dd, DATEDIFF(dd, 0, GetDate()), 0))
```

So to do the orders due this month as a `BETWEEN` query, you can use this:
```
SELECT [ID]
FROM [dbo].[Orders]
WHERE [ShipDue] BETWEEN DATEADD(mm, DATEDIFF(mm, 0, GetUTCDate()), 0)
AND DATEADD(ms, -3, DATEADD(mm, DATEDIFF(mm, 0, GetUTCDate()) + 1, 0))
```

Remember, always make sure that you do math against input parameters, **NOT** columns, or you will kill the [**SARG**](http://www.databasejournal.com/features/mssql/article.php/1436301)\-ability of the query, which means indexes that might have been used aren't.

Here's the complete pastable list:

```
SELECT
DATEADD(dd, DATEDIFF(dd, 0, GetDate()), 0) As Today
, DATEADD(wk, DATEDIFF(wk, 0, GetDate()), 0) As ThisWeekStart
, DATEADD(mm, DATEDIFF(mm, 0, GetDate()), 0) As ThisMonthStart
, DATEADD(qq, DATEDIFF(qq, 0, GetDate()), 0) As ThisQuarterStart
, DATEADD(yy, DATEDIFF(yy, 0, GetDate()), 0) As ThisYearStart
, DATEADD(dd, DATEDIFF(dd, 0, GetDate()) + 1, 0) As Tomorrow
, DATEADD(wk, DATEDIFF(wk, 0, GetDate()) + 1, 0) As NextWeekStart
, DATEADD(mm, DATEDIFF(mm, 0, GetDate()) + 1, 0) As NextMonthStart
, DATEADD(qq, DATEDIFF(qq, 0, GetDate()) + 1, 0) As NextQuarterStart
, DATEADD(yy, DATEDIFF(yy, 0, GetDate()) + 1, 0) As NextYearStart
, DATEADD(ms, -3, DATEADD(dd, DATEDIFF(dd, 0, GetDate()) + 1, 0)) As TodayEnd
, DATEADD(ms, -3, DATEADD(wk, DATEDIFF(wk, 0, GetDate()) + 1, 0)) As ThisWeekEnd
, DATEADD(ms, -3, DATEADD(mm, DATEDIFF(mm, 0, GetDate()) + 1, 0)) As ThisMonthEnd
, DATEADD(ms, -3, DATEADD(qq, DATEDIFF(qq, 0, GetDate()) + 1, 0)) As ThisQuarterEnd
, DATEADD(ms, -3, DATEADD(yy, DATEDIFF(yy, 0, GetDate()) + 1, 0)) As ThisYearEnd
```

For general reading about dates and time, might I suggest [this post's](http://musingmarc.blogspot.com/2006/04/oooo-cool.html) links.

---

### Comments

#### Thank you! Very Helpfu…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-05-03T15:33:00.000-05:00">May 4, 2007</time>

Thank you! Very Helpful!!
---

#### Why the bias against math on columns? For smaller …

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2008-01-16T15:03:00.000-06:00">Jan 3, 2008</time>

Why the bias against math on columns? For smaller databases losing an index isn't an issue, and results in much simpler queries, just my humble opinion.
---

#### Sweeet tips on the dates...I've just used your adv…

[Mark Abraham](https://www.blogger.com/profile/00940934754906803851 "noreply@blogger.com") - <time datetime="2008-02-26T09:41:00.000-06:00">Feb 2, 2008</time>

Sweeet tips on the dates...I've just used your advice for truncating the time. The customer wanted only to display the top of the hour.
---

#### Is it possible that I can store date/time in "date…

[wsindhu](https://www.blogger.com/profile/15703570835943740801 "noreply@blogger.com") - <time datetime="2009-02-06T00:47:00.000-06:00">Feb 5, 2009</time>

Is it possible that I can store date/time in "datetime" column like this...
  
`"2009-02-06 11:39:25"`
  
Do I have to use something in "formula" field?
  
Please do help me!
---

#### SQL Server (and all the others) store DateTime val…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2009-02-06T01:20:00.000-06:00">Feb 5, 2009</time>

SQL Server (and all the others) store DateTime values in an internal binary representation that has NOTHING to do with how it is displayed for you when you make a query. What you need to do is specify the formatting you require in you client layer (where it belongs) or (at worst) in the SELECT statement. That said, your format is perfectly acceptable for an INSERT or UPDATE statement if single-quoted.
---

#### You are a life saver :) many thanks…

Rocky
[Anonymous](mailto:noreply@blogger.com) - <time datetime="2009-02-17T09:15:00.000-06:00">Feb 2, 2009</time>

You are a life saver :) many thanks.
  
Rocky
---

#### If you use the suggested…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2009-04-05T22:55:00.000-05:00">Apr 0, 2009</time>

If you use the suggested;  
select dateadd(dd, datediff (dd, 0, '20080327 23:59:59.999'), 0)  
It actually returns 20080328 00:00:00.000  
It is obviously a rounding issue but is there a way around it?
---

#### .999 is beyond the precision I stated. You cannot …

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2009-04-06T00:54:00.000-05:00">Apr 1, 2009</time>

.999 is beyond the precision I stated. You cannot use anything higher than .997 because the resolution of SQL Server is 3 milliseconds.
---

#### How about the first weekday of the last month? ;) …

[Guest]( "noreply@blogger.com") - <time datetime="2009-07-22T23:40:14.000-05:00">Jul 3, 2009</time>

How about the first weekday of the last month? ;) Anyone?
---

#### Step by step First day of LAST month: SELECT DA…


[IDIsposable]( "noreply@blogger.com") - <time datetime="2009-07-24T17:30:27.000-05:00">Jul 5, 2009</time>

Step by step  
First day of LAST month:  
SELECT DATEADD(mm, DATEDIFF(mm, 0, GetDate()) - 1, 0)  
First Monday of LAST month OR last Monday of prior month:  
SELECT DATEADD(wk, DATEDIFF(wk, 0, DATEADD(mm, DATEDIFF(mm, 0, GetDate()) - 1, 0)), 0)  
  
So all we have to do is deal with when the prior Monday is in the wrong month (and add 7 days)  
  
DECLARE @today DATETIME  
SET @today = '2009-08-23 11:56:23' --GetDate()  
SELECT CASE WHEN RecentMonday < StartOfLastMonth  
THEN  
DATEADD(wk, DATEDIFF(wk, 0, RecentMonday), 7) -- adjust up a week  
ELSE  
RecentMonday  
END AS FirstMondayOfTheMonth  
FROM (  
SELECT DATEADD(mm, DATEDIFF(mm, 0, @today) - 1 , 0) AS StartOfLastMonth  
, DATEADD(wk, DATEDIFF(wk, 0, DATEADD(mm, DATEDIFF(mm, 0, @today) - 1, 0)), 0) AS RecentMonday  
) AS Fake
---

#### Thanks for the inf…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2010-11-02T02:43:18.000-05:00">Nov 2, 2010</time>

Thanks for the info !
---

#### TheDate is not working in s…


[San]( "noreply@blogger.com") - <time datetime="2010-11-11T02:02:31.000-06:00">Nov 4, 2010</time>

TheDate is not working in sql?
---

#### TheDate is meant to be where \_YOU\_ insert whatever…


[Marc Brooks]( "noreply@blogger.com") - <time datetime="2010-11-11T02:38:36.000-06:00">Nov 4, 2010</time>

TheDate is meant to be where \_YOU\_ insert whatever date you want to manipulate... a column name, GETDATE(), etc...
---

#### Regarding the "Monday of any week", I ne…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2011-10-10T02:04:23.573-05:00">Oct 1, 2011</time>

Regarding the "Monday of any week", I needed to get the monday of any week, regardless of @@DATEFIRST settings. This the formula I came up with:  
**  
DATEADD(DAY,-(DATEPART(weekday, GETDATE()) + @@DATEFIRST +5)%7, GETDATE())**  
  
What it does is for instance given tuesday, and datefirst =7 (sunday)  
3+7+5=15%7=1, and substracts 1, from the date, which gives the monday.  
  
For datefirst = 1(monday) it would be:  
2+1+5=8%7=1 and substract 1, from the date, which gives monday
---
