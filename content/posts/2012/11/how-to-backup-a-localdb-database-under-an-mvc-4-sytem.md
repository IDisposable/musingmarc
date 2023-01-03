+++
title = "How to backup a LocalDB database under an MVC 4 sytem"
date = 2012-11-27T23:53:00.001-06:00
updated = 2013-12-03T20:10:17.204-06:00
draft = false
url = '/2012/11/how-to-backup-localdb-database-under.html'
tags = ["SQL","MVC",".Net","Backup","LocalDB"]
+++

_UPDATE: [EntityFramework 6](http://entityframework.codeplex.com/releases/view/87028) has lots of [Resiliency enhancements](http://entityframework.codeplex.com/wikipage?title=Connection%20Resiliency%20Spec), but one of the side effects is that this needs a tweak to keep EF from spinning up a transaction around the SQL statement.  Essentially, you have to call ExecuteSqlCommand and request that it does NOT ensure the a transaction exists with the first parameter of **TransactionalBehavior.DoNotEnsureTransaction**.  If still running EF 5 or below, omit that argument._   
  
I have a smallish MVC 4 site with a database. In this case it isn't worthy of a dedicated SQL Server, so I decided to try out the new LocalDb database feature.  
  
While the database isn't particularly mission critical, I would like to be able to easily back it up on demand to allow some level of disaster recovery.  
  
So, without further adéu, I give you:  
  
```
namespace SomeSimpleProject.Controllers
{
    [Authorize(Roles="Admin")]
    public class BackupController : Controller
    {
        public ActionResult BackupDatabase()
        {
            var dbPath = Server.MapPath("~/App_Data/DBBackup.bak");
            using (var db = new DbContext())
            {
                var cmd = String.Format("BACKUP DATABASE {0} TO DISK='{1}' WITH FORMAT, MEDIANAME='DbBackups', MEDIADESCRIPTION='Media set for {0} database';"
                    , "YourDB", dbPath);
                db.Database.ExecuteSqlCommand(TransactionalBehavior.DoNotEnsureTransaction, cmd);
```

```
            }
       
            return new FilePathResult(dbPath, "application/octet-stream");
        }
    }
}

```

---

### Comments

#### Just what I need but you didn't include the re…

[Unknown](https://www.blogger.com/profile/04795306849055145440 "noreply@blogger.com") - <time datetime="2012-12-16T15:58:02.398-06:00">Dec 0, 2012</time>

Just what I need but you didn't include the required references. Thanks!
---

#### This is what i was looking for, plus it has the en…


[Unknown](https://www.blogger.com/profile/03861533854630998450 "noreply@blogger.com") - <time datetime="2013-01-18T09:43:35.119-06:00">Jan 5, 2013</time>

This is what i was looking for, plus it has the entity framework base.  
Thanks a lot!!
---

#### This is exactly what i need even in the entity fra…


[Unknown](https://www.blogger.com/profile/03861533854630998450 "noreply@blogger.com") - <time datetime="2013-01-18T09:45:48.877-06:00">Jan 5, 2013</time>

This is exactly what i need even in the entity framework base.  
Thanks a lot!
---
