+++
title = "Restoring a LocalDB within an MVC web application"
date = 2012-12-10T21:08:00.001-06:00
updated = 2013-12-03T20:10:09.282-06:00
draft = false
url = '/2012/12/restoring-localdb-within-mvc-web.html'
tags = ["Database","SQL","MVC","LocalDB"]
+++

_UPDATE: [EntityFramework 6](http://entityframework.codeplex.com/releases/view/87028) has lots of [Resiliency enhancements](http://entityframework.codeplex.com/wikipage?title=Connection%20Resiliency%20Spec), but one of the side effects is that this needs a tweak to keep EF from spinning up a transaction around the SQL statement.  Essentially, you have to call ExecuteSqlCommand and request that it does NOT ensure the a transaction exists with the first parameter of **TransactionalBehavior.DoNotEnsureTransaction**.  If still running EF 5 or below, omit that argument._   
  
So, as a follow-up to [backing up a LocalDB database](http://musingmarc.blogspot.com/2012/11/how-to-backup-localdb-database-under.html), I guess I should show the simplest path to restoring one.  
  
So, without further adéu, I give you:  
```
public class RestoreDatabaseModel
    {
        public HttpPostedFileBase File { get; set; }
    }

        //
        // GET: /Admin/RestoreDatabase
        [Authorize(Roles = "Admin")]
        public ActionResult RestoreDatabase()
        {
            return View(new RestoreDatabaseModel());
        }

        //
        // POST: /Admin/RestoreDatabase
        [Authorize(Roles = "Admin")]
        [HttpPost]
        public ActionResult RestoreDatabase(RestoreDatabaseModel model)
        {
            const string YOURAPPNAME = "YourAppName";
            var dbPath = Server.MapPath(String.Format("~/App_Data/Restore_{0}_DB_{1:yyyy-MM-dd-HH-mm-ss}.bak", YOURAPPNAME, DateTime.UtcNow));

            try
            {
                model.File.SaveAs(dbPath);

                using (var db = new DBContext())
                {
                    var cmd = String.Format(@"
USE [Master]; 
ALTER DATABASE {0} SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
RESTORE DATABASE {0} FROM DISK='{1}' WITH REPLACE;
ALTER DATABASE {0} SET MULTI_USER;"
                        , YOURAPPNAME, dbPath);
                    db.Database.ExecuteSqlCommand(TransactionalBehavior.DoNotEnsureTransaction, cmd);
                }

                 ModelState.AddModelError("", "Restored!");
            }
            catch (Exception ex)
            {
                ModelState.AddModelError("", ex);
            }

            return View(model);
        }
```

This iteration saves the posted file based on the current date-time and the supplies the correct commands to restore the database. I leave the flushing of HttpCache to you...  
  
Also, if you haven't extended the upload limits of a POST, you'll need this (or similar) in your web.config  
```
<system.web>
    <httpRuntime maxRequestLength="40960" targetFramework="4.5">
    </httpRuntime>
</system.web>
```

This enables larger files to be uploaded (in this case, 40MB... if your database backup is bigger than that, you have no business hacking around with a LocalDB... get a real SQL Server instance to point at.
