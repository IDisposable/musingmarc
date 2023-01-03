+++
title = "Did you know that TempDB's collation isn't necessarily the same as your database's collation?  Do you care?"
date = 2006-05-30T12:07:00.000-05:00
updated = 2007-11-19T00:09:46.776-06:00
draft = false
url = '/2006/05/did-you-know-that-tempdbs-collation.html'
tags = ["SQL","SQL Server 2005"]
+++

Why, yes, you do.

*   We all know that you can set the default collation for a SQL Server instance when you install it, right?
*   We all know that you can set the default collation for a database when you create it, right? (and it defaults to the instance's setting)
*   We mostly all know that you can set the collation for a table when you create it, right? (and it defaults to the database's setting)
*   Some of us even know that you can set the collation for an individual column, right? (and it defaults to the table's setting)

Now, quick, what happens to temporary tables? They're created in `TempDB`, of course. What collation is applied to those tables? The default collation for `TempDB`, of course.

Oh... I see, so if my instance is case insensitive (most are), and my `TempDB` hasn't been `ALTER`ed to a specific collation (most haven't), then when I create a temporary table from my case-sensitive database I am about to be surprised in a very subtle way... It's as easy as `CREATE TABLE #temp COLLATE database_default` to prevent this.

[Kimberly L. Tripp](http://www.sqlskills.com/blogs/kimberly/) has the details, it is a must-read: [Changing Database Collation and dealing with TempDB Objects](http://www.sqlskills.com/blogs/kimberly/PermaLink.aspx?guid=7b4c9796-66d0-4ed2-b19d-bef6bb1e3e1d).
