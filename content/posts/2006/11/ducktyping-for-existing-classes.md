+++
title = "DuckTyping for existing classes"
date = 2006-11-02T14:33:00.000-06:00
updated = 2006-12-11T15:54:56.764-06:00
draft = false
url = '/2006/11/ducktyping-for-existing-classes.html'
tags = [".Net","Emit","Dynamic","Reflection"]
+++

First off, if you don't lurk on [CodeProject](http://www.CodeProject.com "Free Source Code and Tutorials"), I mock you... but assuming you either don't, or just missed this one, Günter Prossliner posted a great bit of code that lets you [dynamically "implement" any interface](http://www.codeproject.com/cs/library/nduck.asp "NDuck") against a class instance by _at runtime_ building a shim class that forwards all the calls through to matching methods in the target.

This is what is commonly called DuckTyping, and allows you to program "as if" an object implements any arbitrary interface, even if it really doesn't inherit from that class. In [my Dynamic library](http://musingmarc.blogspot.com/2006/07/updated-files-for-dynamic-method.html "DynamicMethod, LCG and Reflection.Emit, oh my"), I let you do that for a single method at a time. Günter's code lets you do it an interface at a time, which is obviously cooler if you have multiple methods that you expect a class to implement (and for which you can define an interface).

I think my next weekend project will be to port his code, which uses dynamic assemblies, to use my Dynamic library instead (and thus benefit from LCG).

Technorati tags: [LCG](http://technorati.com/tags/LCG), [Lightweight Code Generation](http://technorati.com/tags/Lightweight%20Code%20Generation), [Duck Type](http://technorati.com/tags/Duck%20Type), [DuckType](http://technorati.com/tags/DuckType), [DynamicMethod](http://technorati.com/tags/DynamicMethod), [Reflection](http://technorati.com/tags/Reflection), [Emit](http://technorati.com/tags/Emit)

[Lightweight Code Generation](http://technorati.com/tag/Lightweight+Code+Generation)

---
### Comments:
#### Man the ASP.NET team needs to look into something ...
[Rick Strahl](https://www.blogger.com/profile/07420160265480672656 "noreply@blogger.com") - <time datetime="2006-11-05T23:47:00.000-06:00">Nov 0, 2006</time>

Man the ASP.NET team needs to look into something like this to stop breaking the framework with duplicated objects in ATLAS :-}... Like that whole ScriptManager vs. ClientScriptManager nightmare.
<hr />
#### Implementing duckTyping via your library would be ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-11-19T06:40:00.000-06:00">Nov 0, 2006</time>

Implementing duckTyping via your library would be really cool.. I would just wondering if it would be possible to like customize the post/pre access code.. something like dynamic interceptors (see codeproject).. ??
<hr />
