+++
title = "Google Sitemaps for ASP.NET 2.0"
date = 2005-12-03T11:23:00.000-06:00
updated = 2005-12-03T11:23:36.476-06:00
draft = false
url = '/2005/12/google-sitemaps-for-aspnet-20.html'
tags = []
+++

This is really cool! Google has long let you expose your site's "logical sitemap" structure so the crawler can better understand the site, details [here](https://www.google.com/webmasters/sitemaps/docs/en/protocol.html#sitemapXMLFormat). With ASP.Net 2.0, you can create an XML sitemap file that acts as a DataSource for the various Menu and MenuPath controls. Betrand Le Roy has done the cool work of mapping the ASP.Net sitemap file to an HTTP handler that will expose it in a format for Google to consume. Details [here](http://weblogs.asp.net/bleroy/archive/2005/12/02/432188.aspx).
