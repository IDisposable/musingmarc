+++
title = "EntityFramework 6 breaks Backup/Restore of LocalDB."
date = 2013-12-03T20:08:00.001-06:00
updated = 2013-12-03T20:09:57.751-06:00
draft = false
url = '/2013/12/entityframework-6-breaks-backuprestore.html'
tags = ["Restore","EntityFramwork","Backup","LocalDB"]
+++

[EntityFramework 6](http://entityframework.codeplex.com/releases/view/87028) has lots of [Resiliency enhancements](http://entityframework.codeplex.com/wikipage?title=Connection%20Resiliency%20Spec), but one of the side effects is that doing a backup or restore of a LocalDB database [will need a tweak](http://entityframework.codeplex.com/discussions/454994) to keep EF from spinning up a transaction around the SQL statement.  
  
Essentially, when you call ExecuteSqlCommand, you have to request that it does NOT ensure the a transaction exists with the first parameter of **TransactionalBehavior.DoNotEnsureTransaction**.  
  
I have updated the appropriate posts:  
  
[How to backup a LocalDB database under an MVC 4 sytem](http://musingmarc.blogspot.com/2012/11/how-to-backup-localdb-database-under.html)  
[Restoring a LocalDB within an MVC web application](http://musingmarc.blogspot.com/2012/11/how-to-backup-localdb-database-under.html)
