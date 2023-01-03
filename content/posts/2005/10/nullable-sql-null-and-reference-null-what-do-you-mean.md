+++
title = "Nullable<>, SQL NULL, and reference null  (what do you mean?)"
date = 2005-10-11T10:49:00.000-05:00
updated = 2005-10-11T10:53:59.846-05:00
draft = false
url = '/2005/10/nullable-sql-null-and-reference-null.html'
tags = []
+++

An interesting post on Wesner Moise's blog shows that C# 3.0 and VB.Net 9.0 don't take the same view of Nullable<>. C# takes the idea that Nullable<> is to bridge between value and reference types, while VB takes the idea that Nullable<> is to mirror SQL's NULL. This is a huge difference and it will be a source of REAL problems, mark my words. [Null Comparison](http://wesnerm.blogs.com/net_undocumented/2005/10/null_comparison.html)
