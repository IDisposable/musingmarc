+++
title = "When failing to understand the constructor gets you.."
date = 2007-04-06T12:42:00.000-05:00
updated = 2007-04-06T13:40:30.466-05:00
draft = false
url = '/2007/04/when-failing-to-understand-constructor.html'
tags = ["funny","Microsoft",".Net","code review","WTF","C#","patterns and practice","Enterprise Library"]
+++

So I was persuing the April release 3.0 of the Microsoft Enterprise Library code and noticed this amusing little bit of code in the CachingStoreProvider.cs file:

```
private ICacheItemExpiration[] GetCacheExpirations()
{
   ICacheItemExpiration[] cachingExpirations = new ICacheItemExpiration[2];
   cachingExpirations[0] = new AbsoluteTime(new TimeSpan(0, 0, ConvertExpirationTimeToSeconds(absoluteExpiration)));
   cachingExpirations[1] = new SlidingTime(new TimeSpan(0, 0, ConvertExpirationTimeToSeconds(slidingExpiration)));
   return cachingExpirations;
}

private int ConvertExpirationTimeToSeconds(int expirationInMinutes)
{
&nbps;&nbps;&nbps;return expirationInMinutes * 60;
}
```

What amuses **you** about that code? For me, it's the way that someone completely blanked on the constructor arguments for `TimeSpan`. Let's look at the specification:

<table><tbody><tr><td>Name</td><td>Description</td></tr><tr><td><a href="http://msdn2.microsoft.com/en-us/library/zz841zbz.aspx">TimeSpan (Int64)</a></td><td>Initializes a new TimeSpan to the specified number of ticks. Supported by the .NET Compact Framework.</td></tr><tr><td><a href="http://msdn2.microsoft.com/en-us/library/bk8a3558.aspx">TimeSpan (Int32, Int32, Int32)</a></td><td>Initializes a new TimeSpan to a specified number of hours, minutes, and seconds.</td></tr><tr><td><a href="http://msdn2.microsoft.com/en-us/library/85ydwftb.aspx">TimeSpan (Int32, Int32, Int32, Int32)</a></td><td>Initializes a new TimeSpan to a specified number of days, hours, minutes, and seconds.</td></tr><tr><td><a href="http://msdn2.microsoft.com/en-us/library/6c7z43tw.aspx">TimeSpan (Int32, Int32, Int32, Int32, Int32)</a></td><td>Initializes a new TimeSpan to a specified number of days, hours, minutes, seconds, and milliseconds.</td></tr></tbody></table>

So, we're using the "hours, minutes, seconds" version of the constructor in that code, right? So why in the world do we need that `ConvertExpirationTimeToSeconds` function? That function can be completely deleted, and the two `TimeSpan` constructors should pass the configurable number of minutes as the minutes (second)parameter and zero as the seconds (third) parameter.

Yes, I realize this is pedanticism, but it clearly points out something that has been bothering me lately... namely that having many overloads, especially for constructors, is probably not a great idea. Nobody is really sure which is the best to use and nobody can keep an entire API in their head. In fact, for `TimeSpan`, the best choice is not even a constructor, but a static factory method `FromMinutes`. So, the final version of that code should be:

```
private ICacheItemExpiration[] GetCacheExpirations()
{
   ICacheItemExpiration[] cachingExpirations = new ICacheItemExpiration[2];
   cachingExpirations[0] = new AbsoluteTime(TimeSpan.FromMinutes(absoluteExpiration));
   cachingExpirations[1] = new SlidingTime(TimeSpan.FromMinutes(slidingExpiration));
   return cachingExpirations;
}
```

[![](http://imagegen.last.fm/DarkSeas/recenttracks/1/IDisposable.gif)](http://www.last.fm/user/IDisposable/?chartstyle=DarkSeas)

---
### Comments:
#### I couldn't agree more.  
This ambiguous construct...
[Anonymous]( "noreply@blogger.com") - <time datetime="2007-04-11T19:40:00.000-05:00">Apr 3, 2007</time>

I couldn't agree more.  
This ambiguous constructor problem is explained well in "Refactoring to Patterns":  
http://industriallogic.com/xp/refactoring/constructorCreation.html
<hr />
