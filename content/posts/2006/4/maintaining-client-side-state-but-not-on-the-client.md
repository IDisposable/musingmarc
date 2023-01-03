+++
title = "Maintaining client-side state but NOT on the client."
date = 2006-04-24T13:44:00.000-05:00
updated = 2006-04-24T13:44:31.273-05:00
draft = false
url = '/2006/04/maintaining-client-side-state-but-not.html'
tags = []
+++

So you are writing distributed browser-based application in JavaScript. So you need or want to store some things client-side. Up to now, the best thing to do was to embed a Flash object and use Local Shared Obects to store that information. There are a couple things that aren't optimal about that:

* Some version of Flash 6+ must be on the client box
* You're subverting Flash via APIs that will change
* You're limited to the 100K store-size of Flash (which still beats a 4K cookie)
* **The state is on the client machine**

Along comes [Les Orchard](http://decafbad.com/blog/) with a JavaScript wrapper around Amazon's _newish_ [S3](http://www.amazon.com/gp/browse.html/ref=sc_fe_c_1_3435361_1/002-6025358-1136059?node=16427261) information store. Check out [S3Ajax](http://decafbad.com/blog/2006/04/03/javascript-apps-with-readwrite-access-to-s3).

Think about these cool things:

* You can store anything in S3, including _your_ images and JavaScript
* The client pays for the bandwidth and storage for _their_ data
* Someone else is responsible for backup and uptime
* **The client's data is available _everywhere_**, not just on their machine
