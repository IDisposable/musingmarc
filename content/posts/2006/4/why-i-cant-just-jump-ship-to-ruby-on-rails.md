+++
title = "Why I can't just jump ship to Ruby on Rails..."
date = 2006-04-05T17:18:00.000-05:00
updated = 2006-05-02T00:17:13.716-05:00
draft = false
url = '/2006/04/why-i-cant-just-jump-ship-to-ruby-on.html'
tags = []
+++

> [412 Precondition Failed](http://www.rubyonrails.org/): "Precondition Failed The precondition on the request for the URL / evaluated to false."

I what way is this comforting? Does it just not like my browser? **Don't ask _me_**.

**Update:** Okay, I tried it on another machine, and it works there. So it doesn't like my browser!. I then used [Rex Swain's cool HTTP View tool](http://www.rexswain.com/httpview.html) to see what was going on. I tried it from the working machine and saw a very different UA string. So I tried removing elements one-by-one to see what made it work. Eventually, I had removed most of the non-standard stuff (.Net CLR versions, etc.) and got it to work. Once I got it to work, I added the missing bits back one-by-one. It's not _which_ strings are in there, it's how long the UA string is.

### This is pathetic!

You don't let people see your site because their UA string is too long?

---
### Comments:
#### That's Apache's mod\_security, not at all related t...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-04-05T18:29:00.000-05:00">Apr 3, 2006</time>

That's Apache's mod\_security, not at all related to Rails... Just an FYI.
<hr />
#### Apache may be _causing_ the error, but I...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-04-05T19:50:00.000-05:00">Apr 3, 2006</time>

Apache may be _causing_ the error, but I'm having trouble taking seriously any project where >50% of the proponents sites are not configured correctly.
<hr />
