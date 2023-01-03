+++
title = "IIS Compression the easy way."
date = 2006-06-19T21:38:00.001-05:00
updated = 2007-11-19T00:09:13.875-06:00
draft = false
url = '/2006/06/iis-compression-easy-way_19.html'
tags = ["performance","IIS"]
+++

With server-based IIS 7.0 you get easy compression of outgoing content, but for earlier versions compression is unavailable (workstation IIS doesn't even support it). For IIS versions above 5.0 you can get very good compression support from Ripcord Software. They've just released IISxpress 2.0, and it's really slick. The configuration is well integrated into the IIS pipeline and works in an opt-out paradigm so you can easily configure specific things not to compress (sub-sites, directories, file extensions, content-type, URI, client IP, etc.) You also can watch live statistics of the files and compression ratios as they are served.

The best part? It's free for non-commercial use and only $50 for commercial use (per server). That's almost as good a deal as Paul Wilson's [Wilson O/R Mapper](http://www.ormapper.net).

Download (or buy) [IISxpress here](http://www.ripcordsoftware.com/IISxpress/default.aspx).

_No remuneration was or will be paid for this recommendation, I just like this stuff_
