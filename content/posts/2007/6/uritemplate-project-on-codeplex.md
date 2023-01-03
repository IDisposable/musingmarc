+++
title = "UriTemplate project on CodePlex"
date = 2007-06-20T18:11:00.001-05:00
updated = 2007-06-20T18:13:18.479-05:00
draft = false
url = '/2007/06/uritemplate-project-on-codeplex.html'
tags = [".Net","CodePlex","UriTemplate","REST","C#","URI","Asp.Net"]
+++

I've just created a new project on CodePlex, and it's got the first (and hopefully only) release available. Enjoy [UriTemplate](http://www.codeplex.com/UriTemplate "UriTemplate library on CodePlex").

### Some background whining...

I admit that sometimes I get a little jealous of other developers, who are not as limited in the things they can adopt. In some cases its a cool new idea like doing RESTful applications. In other cases its a bit of nice functionality living in another platform like java. In still more cases I'm lusting after the cool new stuff in various Microsoft .Net CTPs, betas and such.

The real-life world I find myself in, though, often has me coding against legacy systems running on ASP.Net 1.1, on servers I cannot control or upgrade with inherited systems that barely grok the idea that `WebControl`s can have properties. Woe is me, and probably many others of you out there.

Today, however, I'm taking back the future for slobs like me, and I'm doing it one class at a time.

### Enter the idea of fire...

A while back, I was reading [Steve Maine](http://hyperthink.net/blog/ "Brain.Save()")'s excellent blog and found the interesting post [UriTemplate 101](http://hyperthink.net/blog/2007/05/15/UriTemplate+101.aspx "Introducing another cool class I can't really use yet."), which talks all about a new class available in _an upcoming_ release of .Net. The basic idea of this class is to let you specify a pattern of replaceable tokens to use when constructing or parsing URIs. The class looks to be quite nice, but being a future released, I just filed it away for later cogitation.

### A log in the fireplace...

Dare and everyone else have been talking about this wonderful [REST](http://rest.blueoxen.net/cgi-bin/wiki.pl?RestArchitecturalStyle)ful world [forever](http://www.25hoursaday.com/weblog/2005/03/22/SOAAJAXAndRESTTheSoftwareIndustryDevolvesIntoTheFashionIndustry.aspx), where everything is about URIs that mean something and state transitions occur by following those meaningful paths. Couple that with the long standing best-practice of building Web systems with "[hackable URLs](http://www.useit.com/alertbox/990321.html)" . This resonates with me, and I start thinking about UriTemplate as the application. Of course I don't have any new stuff I'm building that would let me play that way... until last week.

### A match could start something...

Suddenly, a new project appears on the near horizon... a chance to retrofit an cool new set of functionality to an existing ASP.Net 1.1 site. This new stuff would really benefit from hackable URLs and thus needs a good URL Rewriter and Virtual path handler. Sure, those exist, but almost all force me to map URLs to pages via some lovely RegEx matching. This project, however, is all content-driven and just cries out REST. I want a more general solution and UriTemplate sounds like a match.

### Fanning the flames...

So, off I go, looking for the DLL for UriTemplate that Steve's talked about for "[investigation](http://www.aisto.com/roeder/dotnet/ "Reflector is the only reason I can use .Net, thank you Lutz")". I spin and whirl and Google and Live Search (not a verbable word!) till I'm blue in the face, but I can't figure out where this wonderful class has even been sneaked out for peek.  In fact, none the searches turn up much more than  Joe Gregorio's original [idea posting](http://bitworking.org/news/URI_Templates) as a follow up to the application to [RESTful development](http://www.xml.com/pub/a/2005/04/06/restful.html) of templated URIs.

### Sputter, sputter...

Eventually, I stumble across Jeff Newsom's [curiously titled posting](http://integralpath.blogs.com/thinkingoutloud/rest_architecture/index.html "What's an ASK?") about some upcoming WCF features that shows using a UriTemplate in a WebInvokeAttribute. He also mentions in another post that some functionality was "folded into the  BizTalk Services SDK", which brings me right back to the start with Steve Maine's blog. So, I now know where to look.  A quick download of the [BizTalk Services SDK](http://labs.biztalk.net/downloads.aspx) and I've got some code to look at. Fire up Reflector and ugh...way to complex for me to use since I can't deploy the SDK to my production environment. I guess I'm going to have to write my own, but I'm sure not going to back port the one in the SDK.

### I breathe some life into it...

So, last Friday, I finally got around to deciding it was time to just write the code myself, I did another search based on the links that I previously found and stumbled across [James Snells posting](http://www.snellspace.com/wp/?p=467) about [draft specification for URI Templates](http://www.mnot.net/blog/2006/10/04/uri_templating), which included a Java implementation. This code is simple and clean... very likeable. Too bad it's in the wrong language and built for the wrong platform. But I know Java, I know C# and I know how to make one look like the other.  A short time later and I've got a fully functional .Net 2.0 version of the package that James wrote.

This week, my coworker [Ryan Stephenson](http://nopaper.net/) did a quick back-port to the .Net 1.1 framework (I told you we had deployment restrictions!) and today I bundled it all up, created a project on [CodePlex](http://www.codeplex.com/ "CodePlex") and made a quick home page for it.

### Bask in the glow...

So, like I said way back up there... there's now a perfectly serviceable UriTemplate implemention available for schmucks like me. If you are interested, the [goods are here](http://www.codeplex.com/UriTemplate).
