+++
title = "Profiled performance does not equal real-life performance."
date = 2005-11-08T12:42:00.000-06:00
updated = 2005-11-09T11:25:42.783-06:00
draft = false
url = '/2005/11/profiled-performance-does-not-equal.html'
tags = []
+++

Ian **nails it**\* with [this](http://discuss.develop.com/archives/wa.exe?A2=ind0511b&L=dotnet-cx&T=0&F=&S=&P=1941) post to the [DevelopMentor](http://www.develop.com/) [Dotnet-CX](http://discuss.develop.com/dotnet-cx.html) mailing list. Profiling does not give you a real view of the performance of a segment of code, nor does the performance of a segment of code reflect the performance of that code in real use. Don't optimize the performance of something unless you:

* Have a _reproducible_ test-jig for repeatable performance testing
* Have an idea of the _baseline_ performance
* _Know_ where the bottlenecks really are in the code
* Can tell if the performance of the _system_ gets better or worse with changes.
* Have an idea of what performance is _good enough_

\* apologies to Don Box. Edit: Ian blogged it [here](http://www.interact-sw.co.uk/iangblog/2005/11/09/profiling).
