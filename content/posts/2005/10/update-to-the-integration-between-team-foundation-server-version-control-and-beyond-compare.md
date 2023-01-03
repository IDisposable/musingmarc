+++
title = "Update to the integration between Team Foundation Server Version Control and Beyond Compare"
date = 2005-10-26T11:50:00.000-05:00
updated = 2006-02-20T14:08:55.783-06:00
draft = false
url = '/2005/10/update-to-integration-between-team.html'
tags = []
+++

Looks like I was [overly pessimistic](/2005/10/microsoft-team-foundation-server-and.html) about Microsoft.

James Manning at Microsoft (a Team Foundation Version Control guy) dropped in with a few comments to help things along. Adding the **`/title1=%6 /title2=%7`** to the command line arguments in Visual Studio helps the screen display. Thanks James.

Craig at Scooter Software has also addressed some of these issues. The just-released 2.4 version of the ImageViewer from Scooter Software does understand the image file format without needing the extension adjustment (thanks Craig).

I still would love to see Scooter Software update the engine to ignore all the stuff following the _`;`_ so we can not have the overly aggressive patterns in the Rules setup in Beyond Compare, or have Microsoft use subdirectories or something instead of the odd file names, but I'm certainly happy for now.

**UPDATE**: James Manning from Microsoft informs me that they've fixed this in the post-beta-3 bits. Thanks, gang!

**UPDATE #2**: The RC has this issue fixed, so you can change the filters back as needed. Also, James has a nice post on configuring for various tools [here](http://blogs.msdn.com/jmanning/articles/535573.aspx).

---

### Comments

#### Just FYI, while I'm not sure what the final form i…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-27T20:48:00.000-05:00">Oct 4, 2005</time>

Just FYI, while I'm not sure what the final form is, we talked about it and approved this as a fix for V1 (assigned to me :) so this should be better by the time we release.

---

#### Looks like the Team Foundation gang at Microsoft r…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-11-16T22:39:00.000-06:00">Nov 3, 2005</time>

Looks like the Team Foundation gang at Microsoft really is Agile, they've addressed the file-extension issue by putting the version number at the front of the filename.

---
