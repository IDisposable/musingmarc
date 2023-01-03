+++
title = "Fixing forums.asp.net RSS feed issues."
date = 2006-07-28T12:14:00.000-05:00
updated = 2007-11-18T23:58:06.591-06:00
draft = false
url = '/2006/07/fixing-forumsaspnet-rss-feed-issues.html'
tags = ["Annoyances","RSS"]
+++

I use [RSSBandit](http://www.rssbandit.org/) as my feed reader, and it is awesome. I noticed, however, that it was having issues with the forum feeds from ASP.Net. A typical feed from there looks like this [http://forums.asp.net/rss.aspx?ForumID=1022&Mode=0](http://forums.asp.net/rss.aspx?ForumID=1022&Mode=0) (not sure what that Mode=0 does... anyone care to document that). This works fine the first time I hit the page, but subsequent to that [CommunityServer](http://communityserver.org/) (which is what [forums.asp.net](http://forums.asp.net) runs on) will always return a _304 Not Modified_ HTTP status code. This is clearly a lie when the forum has new messages. For a while, I would just tweek the feed url changing the Mode parameter to 1 (no idea what this does), do a refresh, and then change it back. This would cause a URL mismatch, so we would get the feed from the server anew.

This got a little old after a while, so I fired up Fiddler and verified what I needed to tweek to insure the fetch succeeds. If I zap the `If-Modified-Since` and `If-None-Match` header elements from the request headers, then I'll always get back the feed. This is somewhat bad manners, of course, but until the folks at [Telligent](http://telligent.com/) fix the problem in CommunityServer and the folks at Microsoft update the software at [asp.net](http://www.asp.net), I'll just have to keep poking them.

The code for Fiddler's script looks like this:

```
// up in the top of the class Handlers
public static RulesOption("Fix &RSS forums.asp.net")
var m_RSSForum: boolean = true;
 
// in OnBeforeRequest
if (m_RSSForum
    && oSession.host.indexOf("forums.asp.net")>-1
    && oSession.url.toLowerCase().indexOf("/rss.aspx")>-1)
{
    oSession.oRequest.headers.Remove("If-Modified-Since");
    oSession.oRequest.headers.Remove("If-None-Match");
}
```

Now that I run Fiddler all the time, this just works (which I do to strip ads from everything I surf). I hope this helps you.
