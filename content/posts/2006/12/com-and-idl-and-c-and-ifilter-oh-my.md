+++
title = "COM and IDL and C++ and IFilter, oh my!"
date = 2006-12-12T14:21:00.000-06:00
updated = 2006-12-12T14:53:53.946-06:00
draft = false
url = '/2006/12/com-and-idl-and-c-and-ifilter-oh-my.html'
tags = ["C++","IDL","IFilter","COM","Source"]
+++

It's a blast from the past! On the [Developmentor Advanced .Net mailing list](http://discuss.develop.com/advanced-dotnet.html) recently, there was a bit of discussion about doing text extraction from Word documents. This is something I did [in a past life](http://www.sendouts.net) and let me tell you it wasn't any fun.

First we tried using WinWord as an automation server. We would open the file, then save as text. This leaked resources horribly and was slow. It eventually would just lockup and the DCOM would go to sleep... not good for a product (at that time) based on a MTS/COM+ server running RDO components under IIS 5.

Next we tried about twenty or thirty "conversion" programs that were supposed to allow batch or individual conversion of file. This was even worse than using WinWord as the fidelity of the conversion was horrible at best.

Then we tried a couple of commercial libraries for data conversion. They were better at getting the text, but honestly less stable than WinWord.

Then it hit me, in the shower as all good ideas do, doesn't Microsoft's Index Server do a ripping good job sucking out the text of documents to index them? Obviously that can't be unstable or Index Server's crawller would die all the time. So I delved a bit into how Index Server works. Turns out that it [loads a COM component](http://msdn2.microsoft.com/en-us/library/ms692488.aspx) and uses the [IFilter interface](http://msdn2.microsoft.com/en-us/library/ms691105.aspx) to ask a pluggable filter to extract the text. All I had to do was find an example of an IFilter consumer. Trouble was, there weren't any... sure you could find a ton of [IFilter providers](http://msdn2.microsoft.com/en-us/library/ms692571.aspx) to install and enable Index Server to crawl the files, but good luck finding an example of calling one.

Eventually I worked it all out and wrote a COM component of my own in C++ that loads the right IFilter driver for the file, then sucks all the text out. It was COM because the client program was a VB6 service.

This really rocked because all I had to do was find the right IFilter drivers for a document format and away it went... no configuration, just joy. And lest you think this is all old news and worthless, Microsoft's Desktop Search **still** uses IFilter magic. Google Desktop Search also can use IFilters with the appropriate plug-in. These days you can find [plug-ins for everything](http://www.ifilter.org/) from AutoCAD drawings to MP3 files (not to mention all the big-boys, like Word, WordPerfect, OpenOffice, PDF, etc).

So with the recent discussion on the ADVANCED-DOTNET list, I offered and was asked to dust off the component and publish it for your use. I had to do a tiny bit of cleanup and conversion to be usable with Visual Studio, but it's up for your pleasure. It can be used from .Net programs, of course, though I might eventually get around to rewriting the tiny amount of real code as a .Net library. I haven't looked into the quality of this, but there is [something on CodeProject](http://www.codeproject.com/csharp/IFilter.asp).

[Download here.](http://idisposable.googlepages.com/downloads)

---
### Comments:
#### I've hosted this project on CodePlex. See[](...</x-turndown)
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-12-29T02:41:00.000-06:00">Dec 5, 2006</time>

I've hosted this project on CodePlex. See [this](http://musingmarc.blogspot.com/2006/12/ifilter-text-extracter-now-hosted-on.html).
<hr />
#### It might be possible, but I haven't played wit...
[Marc Brooks]( "noreply@blogger.com") - <time datetime="2010-10-07T21:41:08.000-05:00">Oct 4, 2010</time>

It might be possible, but I haven't played with this stuff in a long time...
<hr />
#### Hi Marc, I've tried your ifilter implementat...
[wei min](https://www.blogger.com/profile/00522527175294915943 "noreply@blogger.com") - <time datetime="2011-11-16T21:39:37.228-06:00">Nov 3, 2011</time>

Hi Marc,  
  
I've tried your ifilter implementation for c++ and it works great for most cases.  
  
However, i've encountered a problem with the latest pdf 10.1 ifilter. It seems to fail at loadIFilter. The strange thing is that this ifilter works fine for windows search. Btw, I tested it on Win7, not sure if this would affect anything.  
  
Any ideas?
<hr />
#### Hi Marc, I've tried your ifilter implementat...
[wei min](https://www.blogger.com/profile/00522527175294915943 "noreply@blogger.com") - <time datetime="2011-11-16T21:40:37.738-06:00">Nov 3, 2011</time>

Hi Marc,  
  
I've tried your ifilter implementation for c++ and it works great for most cases.  
  
However, i've encountered a problem with the latest pdf 10.1 ifilter. It seems to fail at loadIFilter. The strange thing is that this ifilter works fine for windows search. Btw, I tested it on Win7, not sure if this would affect anything.  
  
Any ideas?
<hr />
#### Adobe Reader 10 (X) does not come with an IFilter ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2012-02-07T15:59:20.881-06:00">Feb 2, 2012</time>

Adobe Reader 10 (X) does not come with an IFilter anymore. Only the 9.5 version does.
<hr />
