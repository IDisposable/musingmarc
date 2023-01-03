+++
title = "Version 2 of ASP.Net Rss Toolkit released!"
date = 2007-06-16T00:46:00.001-05:00
updated = 2007-06-16T00:48:22.935-05:00
draft = false
url = '/2007/06/version-2-of-aspnet-rss-toolkit.html'
tags = ["Microsoft","RDF","CodePlex","Atom","RssToolkit","Asp.Net","RSS"]
+++

Thanks to some amazing work by [Piyush Shah](http://blogs.msdn.com/shahpiyush/default.aspx "Piyush Shah's blog") of Microsoft, the ASP.Net RssToolkit originally authored by [Dmitry Robsman](http://blogs.msdn.com/dmitryr/ "Dimitry Robsman of Microsoft's blog") has grown up big and strong!

The new release adds some awesome features that many users have be asking for, some considerable tightening of the code base was done by myself and [Jon Gallant](http://blogs.msdn.com/jongallant/ "Jon Gallant of Microsoft's blog"), and I got off my lazy butt and update the Wiki using some documentation that Piyush wrote as a basis.

This release adds support for some **huge** features that I'll summarize here, but you should really head to the [project home](http://www.codeplex.com/ASPNETRSSToolkit "ASP.Net RssToolkit on CodePlex") page to read the Wiki documentation.

New features:

*   Atom, RDF and OPML support! Available both for consuming and publishing, this includes full support for the required elements of the various RSS, Atom and RDF schemas. For OMPL feeds, supports aggregation of the referenced feeds (even if mixed format) into a single feed.
*   Strong-typed code generation of classes that fully understand the feed schema, including any custom extensions like the [Yahoo Media RSS extensions](http://search.yahoo.com/mrss "Description of the Yahoo extension to Rss to encode media attributes for searching").
*   RSS/Atom/RDF schema validation during aggregation of OPML
*   Ability to reflect or generate any feed in any of the formats supported (including pulling Atom feeds in and morph to RSS out).
*   DownloadManager can be used to cache any feed format and supports app-relative paths under ASP.Net applications
*   Added support for enclosures and qualified namespaces for feed elements
*   Now packaged as a Visual Studio solution with proper projects for all the sub-projects
*   Sporting a new complete set of Visual Studio Team unit tests (sorry, nUnit guys... haven't created parallel ones yet)

See this post for details on the earlier [version 1.0 releases](http://musingmarc.blogspot.com/2006/12/aspnet-rss-toolkit-published-on.html "ASP.Net RssToolkit version 1.0.0.1 released to CodePlex")
