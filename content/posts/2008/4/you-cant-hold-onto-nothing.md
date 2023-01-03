+++
title = "You can't hold onto nothing"
date = 2008-04-10T17:20:00.002-05:00
updated = 2008-04-10T17:21:31.302-05:00
draft = false
url = '/2008/04/you-cant-hold-onto-nothing.html'
tags = ["SQL","Insert","Locks"]
+++

A while back, we were looking for an easy way to count "hits" against content in a CMS-like system. For the sake of discussion, pretend we have a table called `ContentEntry` that represents the content. We decided we wanted to track the hits by-hour against a particular content entry, so that's the `ContentEntryPlusPlus` table on the right. The foreign-key is from `ContentEntryPlusPlus.ContentEntryID` to `ContentEntry.ID`.

[![ContentEntry](http://lh3.ggpht.com/idisposable/R_6SqNmUbNI/AAAAAAAABIc/RcA2TAlN5do/ContentEntry_thumb.png?imgmax=800){width=0px height=228}](http://lh5.ggpht.com/idisposable/R_6SptmUbMI/AAAAAAAABIU/YFa6TJbd4-8/s1600-h/ContentEntry.png)

Now the trick is to insert the row if needed for a particular entry and time-slot then increment the `Hits` column. The simplest thing to do is to is check to see if the row exists, insert it if not, then do the update.  Something like this to find the row's `ID`:

```
SELECT TOP 1 ID
FROM dbo.ContentEntryPlusPlus
WHERE ContentEntryID = @ContentEntryID
AND TimeSlot = DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0))
```

Then we have to insert the row if missing:

```
INSERT INTO dbo.ContentEntryPlusPlus(ContentEntryID, TimeSlot)
VALUES (@ContentEntryID, DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0))
SELECT Scope_Identity() AS ID
```

Then we do the update like this:

```
UPDATE dbo.ContentEntryPlusPlus
SET Hits = Hits + 1
WHERE ID = @ID -- from above SELECT or INSERT's Scope_Identity()
```

Obviously we have to do this inside a transaction or we could have issues and I **hate** multiple round-trips, so we crafted this cute statement pair to insert the row if needed and then update. Note the use of `INSERT FROM` coupled with a fake table whose row count is controlled by an `EXISTS` clause checking for the desired row. This gets executed as a single SQL command.

```
INSERT INTO dbo.ContentEntryPlusPlus(ContentEntryID, TimeSlot)
SELECT TOP 1
 @ContentEntryID AS ContentEntryID
 ,DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0) AS TimeSlot
 FROM (SELECT 1 AS FakeColumn) AS FakeTable
 WHERE NOT EXISTS (SELECT * FROM dbo.ContentEntryPlusPlus
                   WHERE ContentEntryID = @ContentEntryID
                   AND TimeSlot = DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0))

UPDATE dbo.ContentEntryPlusPlus
SET Hits = Hits + 1
WHERE ContentEntryID = @ContentEntryID
AND TimeSlot = DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0)
```

This got tested and deployed, working as expected. The only problem is that every once in a while, for some particularly popular content, we would get a violation of the clustered-key's uniqueness check on the `ContentEntryPlusPlus` table. This was quite surprising, honestly as the code obviously worked when we tested it.

The only thing that could cause this is if the two calls executed the inner existence-check simultaneously and both decided an `INSERT` was warranted. I had assumed that locks would be acquired, and they are, for the inner `SELECT`, but since there are no rows to when this is executed, there are no rows locked, so both statements will plow on through. So, I just had to add a quick `WITH (HOLDLOCK)` hint to the inner `SELECT` and poof it works.

So, the moral of the story? **You can't hold onto nothing...**

The final version is:

```
INSERT INTO dbo.ContentEntryPlusPlus(ContentEntryID, TimeSlot)
SELECT TOP 1
 @ContentEntryID AS ContentEntryID
 ,DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0) AS TimeSlot
 FROM (SELECT 1 AS FakeColumn) AS FakeTable
 WHERE NOT EXISTS (SELECT * FROM dbo.ContentEntryPlusPlus WITH (HOLDLOCK)
                   WHERE ContentEntryID = @ContentEntryID
                   AND TimeSlot = DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0))

UPDATE dbo.ContentEntryPlusPlus
SET Hits = Hits + 1
WHERE ContentEntryID = @ContentEntryID
AND TimeSlot = DateAdd(hh, DateDiff(hh, 0, GetUtcDate()), 0)
```

---
### Comments:
#### So - rather than having a dual update / insert sta...
[digiguru]( "noreply@blogger.com") - <time datetime="2009-10-22T03:39:55.000-05:00">Oct 4, 2009</time>

So - rather than having a dual update / insert statement, why not have a single table that stores each view as a row.  
  
ContentEntryView  
ID  
ContentID  
IPAddress (for tracking purposes)  
DateTime  
  
Then you can make ContentEntryPlusPlus a view on the previous table.  
  
Select  
Count(ID)  
ContentID,  
CAST(CONVERT(varchar, GetDate(), 101) AS DateTime) as DateTime,  
FRoM ContentEntryView  
Group By ContentID, CAST(CONVERT(varchar, GetDate(), 101) AS DateTime)  
  
  
Plus you have the ability to graph views over time. I am assuming that you want fast insert, slow select, because this is an adminstrators view?
<hr />
#### Your change would be good if I A) could store th...
[Marc]( "noreply@blogger.com") - <time datetime="2009-10-22T14:34:38.000-05:00">Oct 4, 2009</time>

Your change would be good if I  
  
A) could store the hits \[too many\]  
  
B) wasn't storing them elsewhere \[have a track server\]  
  
C) didn't care about performance \[I display this hit count EVERY time I display the entry, so it has to be very fast on query\]  
  
So, given the schema is what I have to have, this is a very safe way to do UPSERT-like behavior for SQL Server 2000+. I wonder if there is a better technique, but this is very fast and known to work.
<hr />
