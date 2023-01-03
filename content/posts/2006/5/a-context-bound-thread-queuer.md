+++
title = "A Context-Bound Thread Queuer"
date = 2006-05-19T12:33:00.000-05:00
updated = 2007-11-19T00:15:41.410-06:00
draft = false
url = '/2006/05/context-bound-thread-queuer.html'
tags = ["OperationContext","C#","cool","thread"]
+++

[Avner Kashtan](http://weblogs.asp.net/avnerk) has posited a wonderful class that lets you bind operational contexts of any kind to the another thread via the `ThreadPool.QueueUserWorkItem`. This makes it really easy to store away contextual information (like a WCF `OperationContext`) that is then used during the pooled-thread's execution of the work item. Read on _\[[here](http://weblogs.asp.net/avnerk/archive/2006/05/19/447089.aspx)\]_
