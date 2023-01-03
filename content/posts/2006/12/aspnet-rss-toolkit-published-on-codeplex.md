+++
title = "ASP.Net RSS Toolkit published on CodePlex."
date = 2006-12-29T00:43:00.000-06:00
updated = 2007-11-18T23:54:28.548-06:00
draft = false
url = '/2006/12/aspnet-rss-toolkit-published-on.html'
tags = [".Net","CodePlex","C#","RssToolkit","Asp.Net","RSS"]
+++

In February of 2006, [Dmitry Robsman](http://blogs.msdn.com/dmitryr/) released a very cool [RSS Toolkit](http://blogs.msdn.com/dmitryr/archive/2006/02/21/536552.aspx) that eases both consuming and publishing RSS feeds in an ASP.Net application.

> The RSS toolkit includes support for consuming as well as publishing RSS feeds in ASP.NET applications. Features include:
> 
> *   RSS Data Source control to consume feeds in ASP.NET applications
>     *   Works with ASP.NET data bound controls
>     *   Implements schema to generate columns at design time
>     *   Supports auto-generation of columns at runtime (via ICustomTypeDescriptor implementation)
> *   Caching of downloaded feeds both in-memory and on-disk (persisted across process restarts)
> *   Generation of strongly typed classes for RSS feeds (including strongly typed channel, items, image, handler) based on a RSS URL (the toolkit recognizes RSS and RDF feeds) or a file containing RSS definition. Allows programmatically download (and create) RSS channels using strongly-typed classes. The toolkit includes:
>     *   Stand-alone command line RSS compiler
>     *   Build provider for .rssdl file (containing the list of feed URLs)
>     *   Build provider for .rss file (containing RSS XML)
> *   Support for generation of RSS feeds in ASP.NET application including:
>     *   RSS HTTP handler (strongly typed HTTP handlers are generated automatically by the build providers) to generate the feed.
>     *   RSS Hyper Link control (that can point to RSS HTTP handler) to create RSS links.
>     *   Optional secure encoding of user name into query string to allow generation of personalized feeds
> *   Set of classes for programmatic consumption and generation of RSS feed in a late-bound way, without using strongly typed generated classes
> 
> The toolkit is packaged as an assembly (DLL) that can be either placed in GAC or in ‘bin’ directory of a web application. It is also usable from client (including WinForms) applications.
> 
> RSS Toolkit works in Medium Trust (RssToolkit.dll Assembly either in GAC or in ‘bin’) with the following caveats:
> 
> *   If the ASP.NET application consumes RSS feeds, the trust level must be configured to allow outbound HTTP requests.
> *   To take advantage of disk caching, there must be a directory (configurable via AppSettings\["rssTempDir"\]) where the trust level policy would allow write access. However, disk caching is optional.

In March, he [updated it to release 1.0.0.1](http://blogs.msdn.com/dmitryr/archive/2006/03/26/561200.aspx) adding a couple much-needed features:

> *   Added MaxItems property to RssDataSource to limit the number of items returned.
> *   Added automatic generation of <link> tags from RssHyperLink control, to light up the RSS toolbar icon in IE7. For more information please see [http://blogs.msdn.com/rssteam/articles/PublishersGuide.aspx](http://blogs.msdn.com/rssteam/articles/PublishersGuide.aspx)
> *   Added protected Context property (of type HttpContext) to RssHttpHandlerBase class, to allow access to the HTTP request while generating a feed.
> *   Added generation of LoadChannel(string url) method in RssCodeGenerator so that one strongly typed channel class can be used to consume different channels.
> *   Fixed problem expanding app relative (~/…) links containing query string when generating RSS feeds.

Then in November, he decided that the [project needed to get hosted](http://blogs.msdn.com/dmitryr/archive/2006/11/03/asp-net-rss-toolkit-on-codeplex-coordinator-s-needed.aspx) somewhere and chose [CodePlex](http://www.codeplex.com) and solicited someone to shepherd the project. I got the nod, and have re-released his 1.0.0.1 release and posted the source in the new [CodePlex project ASP.NET RSS Toolkit](http://www.codeplex.com/ASPNETRSSToolkit). I want to make absolutely clear that up to and including this release, I haven't done anything to what Dmitry released. All the good work is his! In the upcoming couple of weeks, I'll be putting his tools to good use and will be growing the project, if you have any ideas what you want added, please use the [project forums](http://www.codeplex.com/ASPNETRSSToolkit/Project/ListForums.aspx) out on CodePlex. First on my list of things to support is full ETag and Last-Modified header support to enable bandwidth reduction. I also plan on adding event handlers to deal with redirects for feeds that move to a new home.

---
### Comments:
#### Good, the toolkit is in good hands. I've used it i...
[Unknown](https://www.blogger.com/profile/02260143150268254080 "noreply@blogger.com") - <time datetime="2006-12-29T03:29:00.000-06:00">Dec 5, 2006</time>

Good, the toolkit is in good hands.  
I've used it in the past and I'm going to use it in an incoming proyect.  
See you in the codeplex site !
<hr />
