+++
title = "Need a date range in SQL without filling a table?"
date = 2006-07-19T13:14:00.000-05:00
updated = 2007-11-19T00:02:30.715-06:00
draft = false
url = '/2006/07/need-date-range-in-sql-without-filling.html'
tags = ["SQL","DateTime"]
+++

It's evidentally SQL week here at Chez Brooks. Today I needed a really high performance query to deliver a date range table between two dates. Simple, and there seem to be tons of variants out there, but I didn't like the query plans of any of them. The resulting query below works for any start date and end date pair, and will return a date between two given dates.

```
DECLARE @LowDate DATETIME
SET @LowDate = '01-01-2006'

DECLARE @HighDate DATETIME
SET @HighDate = '12-31-2016'

SELECT DISTINCT DATEADD(dd, Days.Row, DATEADD(mm, Months.Row, DATEADD(yy, Years.Row, @LowDate))) AS Date
FROM
(SELECT 0 AS Row UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4
 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9
 UNION ALL SELECT 10 UNION ALL SELECT 11 UNION ALL SELECT 12 UNION ALL SELECT 13 UNION ALL SELECT 14
 UNION ALL SELECT 15 UNION ALL SELECT 16 UNION ALL SELECT 17 UNION ALL SELECT 18 UNION ALL SELECT 19
 UNION ALL SELECT 20 -- add more years here...
) AS Years
INNER JOIN
(SELECT 0 AS Row UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4
 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9
 UNION ALL SELECT 10 UNION ALL SELECT 11
) AS Months
ON DATEADD(mm, Months.Row,  DATEADD(yy, Years.Row, @LowDate)) <= @HighDate 
INNER JOIN
(SELECT 0 AS Row UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4
 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9
 UNION ALL SELECT 10 UNION ALL SELECT 11 UNION ALL SELECT 12 UNION ALL SELECT 13 UNION ALL SELECT 14
 UNION ALL SELECT 15 UNION ALL SELECT 16 UNION ALL SELECT 17 UNION ALL SELECT 18 UNION ALL SELECT 19
 UNION ALL SELECT 20 UNION ALL SELECT 21 UNION ALL SELECT 22 UNION ALL SELECT 23 UNION ALL SELECT 24
 UNION ALL SELECT 25 UNION ALL SELECT 26 UNION ALL SELECT 27 UNION ALL SELECT 28 UNION ALL SELECT 29
 UNION ALL SELECT 30
) AS Days
ON DATEADD(dd, Days.Row, DATEADD(mm, Months.Row,  DATEADD(yy, Years.Row, @LowDate))) <= @HighDate
WHERE DATEADD(yy, Years.Row, @LowDate) <= @HighDate
ORDER BY 1
```

Some notes on this:

1.  If we assume a start date of January 1st, we need to **add** _at most_ 11 months and _at most_ 30 days to bump to then end of the year, so that's where the 0-11 months and 0-30 days come from.
2.  Due to the fact that some months have less than 31 days, we can conceivably generate the same date twice. By adding one month and 28 days to January 31st we get the same date as adding zero days and two months; either way the result is March 1st. Thus we need the `DISTINCT` operator.
3.  If you don't care about the dates being in order, delete the `ORDER BY` clause.
4.  I've currently limited it to a 20 year range, but you can change that in the subquery that generates the years quite easily.
5.  Doing the `DATEADD` in the order of year, then month, then days is **very** important as it insure that the correct leap-day rule is followed.

---
### Comments:
#### Very useful. Cheers!
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-08-03T10:44:00.000-05:00">Aug 4, 2006</time>

Very useful. Cheers!
<hr />
#### I think my method is easier to read ...  
  
DEC...
[Tom Øyvind Hogstad](https://www.blogger.com/profile/14958448086518497717 "noreply@blogger.com") - <time datetime="2006-09-08T06:40:00.000-05:00">Sep 5, 2006</time>

I think my method is easier to read ...  
  
DECLARE @LowDate DATETIME  
SET @LowDate = '2006-01-01' -- Use iso 8601 for "datestrings"  
  
DECLARE @HighDate DATETIME  
SET @HighDate = '2016-12-31'  
;  
With Dates(MyDate)  
AS  
(  
Select @LowDate MyDate  
UNION ALL  
SELECT (MyDate+1) MyDate  
FROM Dates  
WHERE  
MyDate < @HighDate  
)  
SELECT MyDate FROM Dates  
OPTION(MAXRECURSION 0)  
  
And theres really not much of a speed difference :-)
<hr />
#### However your version only works with SQL Server 20...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-09-08T11:59:00.000-05:00">Sep 5, 2006</time>

However your version only works with SQL Server 2005 (because of the Common Table Expressions), whereas mine works with just about any SQL engine (including Oracle, DB2, Informix, MySQL, and SQL Server 2000).
<hr />
#### Thanks Mark and Tom,  
  
With just a little bit...
[Phillip](https://www.blogger.com/profile/01263659264930752471 "noreply@blogger.com") - <time datetime="2008-05-01T10:53:00.000-05:00">May 4, 2008</time>

Thanks Mark and Tom,  
  
With just a little bit of tinkering, I got a min and max date out of my dataset, created a temp table through these dates, and then inner joined to it on mydate between the startdate and enddate of each record in the dataset.  
  
Really made it simple to turn a large dataset with date ranges into an even larger dataset with a record for each date.
<hr />
#### Nice Post, very helpfull. Karl
[Karl](http://www.directplanning.com "noreply@blogger.com") - <time datetime="2009-05-19T04:22:00.000-05:00">May 2, 2009</time>

Nice Post, very helpfull.  
  
Karl
<hr />
#### Nice post, very helpful.
[not Karl]( "noreply@blogger.com") - <time datetime="2009-10-31T17:07:19.000-05:00">Oct 6, 2009</time>

Nice post, very helpful.
<hr />
