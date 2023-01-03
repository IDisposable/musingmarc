+++
title = "Things to consider when using SQL 2005 Query Notifications"
date = 2006-06-28T15:30:00.000-05:00
updated = 2007-11-19T00:06:50.834-06:00
draft = false
url = '/2006/06/things-to-consider-when-using-sql-2005.html'
tags = ["SQL","query notifications","SQL Server 2005","Asp.Net"]
+++

When looking into why I was getting immediate notifications of a SQLDependancy being changed, I discovered a really nice MSDN2 page that documents the things to watch out for, [Special Considerations When Using Query Notifications](http://msdn2.microsoft.com/en-us/library/aewzkxxh.aspx)

Major things to note:

*   If you use a query that doesn't comply, you'll get notified it is out-of-date immediately, so make sure you save the query results somewhere other than Cache\[\] when you do have to go to SQL Server.
*   You **must** use two-part table names like `dbo.Address`, but you should be doing that anyway or you don't get pre-parsed queries. This means that you are _obviously?_ only allowed to talk to one database.
*   Don't use any SQL that requires aggregations or row-counting. This means aggregates like `AVG, MIN, MAX, DISTINCT, COUNT(*)`. You can use `SUM` on non-nullable columns in the presence of a `GROUP BY`, but no `ROLLUP` or `CUBE`.

For full details, consult the article.
