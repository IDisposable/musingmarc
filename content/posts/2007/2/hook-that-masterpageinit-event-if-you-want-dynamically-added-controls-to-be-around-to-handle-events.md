+++
title = "Hook that MasterPage.Init event if you want dynamically added controls to be around to handle events."
date = 2007-02-07T23:58:00.001-06:00
updated = 2011-07-05T13:09:47.862-05:00
draft = false
url = '/2007/02/hook-that-masterpageinit-event-if-you.html'
tags = [".Net","code review","Asp.Net"]
+++

MasterPages sometimes interfere with your pages in the oddest of ways. [Oren Ellenbogen](http://www.lnbogen.com/) shows a clever strategy for handling the creation of dynamically added controls (which you normally would add in `OnPreInit`.

You want to hook the `base.Master.Init` event. See [this great post](http://lnbogen.com/2007/02/07/override-onpreinit-on-a-page-with-master-page/).

[![](http://imagegen.last.fm/DarkSeas/recenttracks/1/IDisposable.gif)](http://www.last.fm/user/IDisposable/?chartstyle=DarkSeas)

---
### Comments:
#### refering to a dead link...
[bjornoyvind]( "noreply@blogger.com") - <time datetime="2011-07-05T13:07:56.040-05:00">Jul 2, 2011</time>

refering to a dead link...
<hr />
#### Thanks, fixed the link-rot.
[Marc Brooks]( "noreply@blogger.com") - <time datetime="2011-07-05T13:10:25.845-05:00">Jul 2, 2011</time>

Thanks, fixed the link-rot.
<hr />
