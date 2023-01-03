+++
title = "More about exception handling and new coolness from Joshua Flanagan"
date = 2006-06-23T01:09:00.001-05:00
updated = 2007-11-19T00:08:52.758-06:00
draft = false
url = '/2006/06/more-about-exception-handling-and-new_23.html'
tags = ["CLR",".Net","exception","Asp.Net","best practice"]
+++

Regular readers of my blog (all 14 of you), will know that I'm a stickler for handling exceptions correctly in applications. I strongly believe that you should only catch exceptions you can **handle**, never catch something you can't **correct** for, unless you have **signficant** contextual information you can add, then use the **InnerException** property, yada, yada...zzzzzzz...

### SLAP!

Sorry about that, you were dozing off.

Anyway, as I was typing... always log exceptions **in detail** using a top level handler, and only stay running if it **makes sense**; like in ASP.Net applications.

Most of that is easy, but what about when you have a background thread in an ASP.Net application? In CLR 1.1, an unhandled exception in a background thread just killed the thread, which is bad because you don't _know_ that it is safe for other threads to continue, and a deadlock or static-data corruption is a very real possibility. Of course, you always wrapped your outer-most function for that background thread in a `try {} catch (Exception ex) { /* log it */ throw; }` block, so you at least logged it. In CLR 2.0, your (proper and correct) re-throw of the `Exception` will result in the `Application` being terminated. Bummer, but we'll come back to that.

[Joshua Flanagan](http://flimflan.com/blog/) has put together a nice `WaitCallback` derived class that does all the by-rote work of wrapping a _thread_\-top-level `try {} catch (Exception ex) { /* log it */ throw; }` block. If that was _all_ it did, I wouldn't be wasting your time, nice as it is to StayDRY™.

Remember what I said about CLR 2.0 dumping your `Application` if the background thread had an unhandled exception? Remember the pattern of rethrowing at the top-level-catch? What if we could somehow marshal that thread's exception details over to the foreground thread(s) of the ASP.Net `Application` where it could be handled by the handy-dandy infrastructure setup in Global.asax's `HttpApplication.Error` or `Application_Error`.

That's where Joshua's cleverness shows. He has done just that, building a nice generic framework for turning the _thread_\-top-level exception into an inline request to an `HttpHandler`, passing the serialized `Exception` details along. Once in the handler, it's simply deserialized and thrown anew, whereupon it is caught and logged as a foreground-thread exception.

Head over to [Safely running background threads in ASP.NET 2.0](http://flimflan.com/blog/SafelyRunningBackgroundThreadsInASPNET20.aspx) and get the goods!
