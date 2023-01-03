+++
title = "System.Enum.CompareTo needs boxed values..."
date = 2006-03-01T11:49:00.002-06:00
updated = 2008-01-14T13:49:02.697-06:00
draft = false
url = '/2006/03/systemenumcompareto-needs-_114123536776382301.html'
tags = [".Net","LCG","CodePlex","CompareTo","Enum"]
+++

I got a report of a bug in the DynamicComparer sources when comparing Enum values. Turns out that you need to box the values before you call `System.Enum.CompareTo()`.

I've made the necessary changes in both the [DynamicComparer.zip](http://idisposable.googlepages.com/DynamicComparer.zip) and [Utilities.zip](http://idisposable.googlepages.com/Utilities.zip) files.

RE: [Dynamic sorting of objects using lightweight code generation.](http://musingmarc.blogspot.com/2006/02/dynamic-sorting-of-objects-using.html)
