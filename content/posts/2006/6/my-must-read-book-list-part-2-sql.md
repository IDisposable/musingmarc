+++
title = "My must-read book list (part 2, SQL)"
date = 2006-06-14T15:30:00.000-05:00
updated = 2007-11-19T00:11:31.308-06:00
draft = false
url = '/2006/06/my-must-read-book-list-part-2-sql.html'
tags = ["SQL","books"]
+++

Continuing the recommended book list from [part one](http://musingmarc.blogspot.com/2006/06/my-must-read-book-list-part-1-basics.html), here's my short list of SQL books:

SQL
---

*   Joel Celko's [SQL For Smarties](http://www.amazon.com/exec/obidos/ASIN/0123693799/marcsmusing0a-20) is a fine distillation of SQL syntax and strategies. He covers all of SQL in a detailed way, including views, partitioning, grouping, aggregates. From there he builts difficult queries advanced statistics, regions, runs, gaps, sequences and series. Next he takes on emulating array, sets (working on all relevant rows at once), subsetting, trees, hierarchies. Next he dives into a topic near and dear to me--temporal (time-based) queries, graphs, and OLAP. Wrapping it up, he talks about transactions, concurrency and optimizing. Simply put, it's like have a SQL ninja in your cube.
*   Joel Celko's [Trees and Hierarchies in SQL for Smarties](http://www.amazon.com/exec/obidos/ASIN/1558609202/marcsmusing0a-20) splits out the trees and hierarchies chapter from SQL for Smarties and expounds on it in a **much** more detailed treatment. Simply amazing detail on the issues and strategies that work.
*   Richard T. Snodgrass's [Developing Time-Oriented Database Applications in SQL](http://www.amazon.com/exec/obidos/ASIN/1558604367/marcsmusing0a-20) takes on a very difficult subject, temporal (time-based) data and explains the issues in creating efficient designs for SQL databases. It addresses the concepts of effective-time, as-entered time, bitemporal data and issues that arise when you start giving data a "when valid" or "when entered" dimension. Coverage is for several databases so you can see a solution expressed in multiple ways. Unfortunately, the book is out of print, but it is well worth the search. If nothing else, the example code for triggers that handles set-logic correctly is 100% gold.
*   Ales Spetic and Jonathan Gennick's [Transact-SQL Cookbook](http://www.amazon.com/exec/obidos/ASIN/1565927567/marcsmusing0a-20) gives great sample solutions specific to SQL Server. Some overlap with Joel's books, but a nice T-SQL perspective. He also covers audit logging and importing data, topics which are lacking in the other books given here.

That's about it for this cut, I'll update when I think of others. Stay tuned for posts about language-specific and platform-specific books. Don't miss my [first part](http://musingmarc.blogspot.com/2006/06/my-must-read-book-list-part-1-basics.html) with the basics.

---

### Comments

#### Since Snodgrass' book is out of print, he has made…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-11-20T13:48:00.000-06:00">Nov 1, 2006</time>

Since Snodgrass' book is out of print, he has made a [PDF available for **free**](http://www.cs.arizona.edu/~rts/publications.html)
---
