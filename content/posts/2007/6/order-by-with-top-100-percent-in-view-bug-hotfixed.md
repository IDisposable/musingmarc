+++
title = "ORDER BY with TOP 100 PERCENT in VIEW bug hotfixed."
date = 2007-06-14T02:57:00.000-05:00
updated = 2007-06-14T03:02:34.769-05:00
draft = false
url = '/2007/06/order-by-with-top-100-percent-in-view.html'
tags = ["SQL","Microsoft","SQL Server 2005","HotFix","bug"]
+++

Anyone who has been bitten when going from SQL Server 2000 to SQL Server 2005 due to the (intentional) decision by Microsoft to ignore the ORDER BY in a VIEW that returns the entire result set can now get a HotFix to enable the legacy behavior. After installing the [HotFix](http://support.microsoft.com/default.aspx?scid=kb;en-us;926292&sd=rss&spid=2855), you will also have to turn on traceflag 168.

[![](http://imagegen.last.fm/DarkSeas/recenttracks/1/IDisposable.gif)](http://www.last.fm/user/IDisposable/?chartstyle=DarkSeas)
