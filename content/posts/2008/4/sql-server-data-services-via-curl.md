+++
title = "SQL Server Data Services via cURL"
date = 2008-04-15T02:30:00.001-05:00
updated = 2008-04-15T02:52:42.210-05:00
draft = false
url = '/2008/04/sql-server-data-services-via-curl.html'
tags = ["SSDS","REST","SQLServer","cURL SQL Server Data Services","SQL Server"]
+++

I just noticed a really [cool article](http://blogs.msdn.com/jcurrier/archive/2008/04/13/curl-ing-up-with-sql-server-data-services.aspx "Jeff Currier") on using Microsoft's new [SQL Server Data Services](http://www.microsoft.com/sql/dataservices/default.mspx "Your Data, Any Place, Any Time ") which explains how to use [cURL](http://curl.haxx.se/ "command-line URL") at the command line to talk to the SSDS RESTful interface.

### What's cURL?

If you've never heard of cURL, it is similar to wget in that it allows you to make HTTP requests of any web service. It can handle all the standard verbs (GET,PUT,POST,DELETE) and also supports all those lovely redirections, security and all that other nonsense. It's great for crufting up any batch/command file that could do all sorts of things as well as to ping-test REST services.

### What's SSDS

If you've never heard of SSDS, it is similar to the Amazon SimpleDB. It offers the ability to push a database out to the public cloud and allow access from web applications, thick clients or whatever. What differentiates it from Google's APE-based access to BigTable or Amazon's S3/SimpleDB setup is that both of those systems are tuple-based (or name-value-based) non-relational databases. The SSDS stuff, on the other hand, is a Linq-based. This makes querying MUCH simpler to do.

The **killer** feature to me is that SSDS doesn't make you (the developer) worry about consistency. With SimpleDB or BigTable, the provider only guarantees "eventual consistency". This means that the changes you make will eventually be propogated through the Amazon/Google cloud. During the time the change was post, but not yet propogate your clients **may see stale data** which makes these services useable mostly for rarely-changed data.

SSDS doesn't have this restriction. Once your call is complete **any access** will result in the commited data being returned. This is a much simpler model to program against and it puts the replication issues squarely on the database server/service where it belongs. What remains to be seen is if Microsoft will really be able to scale this out reasonably.
