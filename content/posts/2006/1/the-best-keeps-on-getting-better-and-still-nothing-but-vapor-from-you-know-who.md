+++
title = "The best keeps on getting better, and still nothing but vapor from you-know-who"
date = 2006-01-24T23:50:00.000-06:00
updated = 2006-02-20T13:02:37.786-06:00
draft = false
url = '/2006/01/best-keeps-on-getting-better-and-still_24.html'
tags = []
+++

The best value in source code has been updated **once** again. Today Paul Wilson released 4.2.1.0 of his **_most-excellent_** Wilson ORMapper. At $50 USD, you will not find a better deal in all of the .Net kingdom. It's good code, and the functionality is _just right_ without the serious overkill that plagues other alternatives. Sure, it's not nHibernate, and nHibernate is free; but honestly the complexity of nHibernate is much more of a cost versus the simplicity and obviousness of WORM (and the attendant cost savings). I know it has saved me an hour or two... what is that in today's consulting rates? In other news, the vaporware specification from Microsoft (ObjectSpaces) has still not shipped... [Wilson ORMapper Latest News v4.2.0.1 Released](http://www.ormapper.net/News/)

---

### Comments

#### Have you tried Active Record…

It takes away muc...
[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-02-20T13:02:00.000-06:00">Feb 1, 2006</time>

Have you tried Active Record?
It takes away much of the complexity in NHibernate, and it's very powerful.
I've a series of posts about it in my blog.

---

#### I've played I with ActiveRecord a little, and I li…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-02-20T14:04:00.000-06:00">Feb 1, 2006</time>

I've played I with ActiveRecord a little, and I like what I've see, but honestly it's not that much simpler than WilsonORMapper and POCO objects.
  
I am NOT willing to use class-level attributes to setup my mappings because my current project needs end-user extensibility, so I'm much happier with XML configurations outside my business Entity objects.
  
Your VERY cool EntitySet and EntityRef classes, on the other hand are very, are something I'm in the process of adapting for use with WilsonORMapper.
  
As for WORM vs. nHibernate, I like WORM better because it's a lot simpler to use and grok for my fellow workers.

---
