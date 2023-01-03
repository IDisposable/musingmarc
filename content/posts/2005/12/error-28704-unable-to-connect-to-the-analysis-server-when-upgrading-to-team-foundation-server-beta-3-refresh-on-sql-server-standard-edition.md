+++
title = "Error 28704. Unable to connect to the Analysis server when upgrading to Team Foundation Server Beta 3 Refresh on SQL Server Standard Edition."
date = 2005-12-22T13:59:00.000-06:00
updated = 2005-12-22T14:17:57.793-06:00
draft = false
url = '/2005/12/error-28704-unable-to-connect-to.html'
tags = []
+++

[Error 28704. Unable to connect to the Analysis server when upgrading to Team Foundation Server Beta 3 Refresh on SQL Server **Standard** Edition.](http://forums.microsoft.com/MSDN/ShowPost.aspx?PostID=178427&SiteID=1) If you are upgrading a Team Foundation Server to Beta 3 Refresh on a Data Tier that has SQL Server **Standard** Edition then you will get this error:

> Errors in the metadata manager. An error occurred when loading the Code Churn perspective, from the file, '\\\\?\\C:\\Program Files\\Microsoft SQL Server\\MSSQL.2\\OLAP\\Data\\TFSWarehouse.0.db\\Team System.1.cub\\Code Churn.1.persp.xml'.

due to the fact that nothing cleans up the old Analysis Server's TFSWarehouse database. You can't do it inside any of the GUI managers because the Analysis Services service will not start up correctly (nice one!).

BUT THERE IS A WORK-AROUND.
===========================

When this error appears on-screen do this:

1.  Control Panel/Administrative Tools/Services.
2.  **Stop** the SQL Server Analysis Services service.
3.  Navigate in Explorer to the database directory where the **Analysis Services databases** are stored (usually _C:\\Program Files\\Microsoft SQL Server\\MSSQL.2\\OLAP\\Data_).
4.  Delete the **file** _TFSWarehouse.0.db.xml_ and the **directory** _TFSWarehouse.0.db_
5.  **Start** the SQL Server Analysis Services service.
6.  See if you can now connect to the Analysis Services in SQL Server Management Studio (Object Explorer/Connect/Analysis Services/your server name). Typically there will be no Databases listed at this point.
7.  Click the **Retry** button on the setup screen.

This essentially eliminates the database that Team Foundation Server is going to setup anyway... so it should work fine.
