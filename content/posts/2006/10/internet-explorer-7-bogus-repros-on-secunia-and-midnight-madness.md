+++
title = "Internet Explorer 7, bogus repros on Secunia and Midnight Madness"
date = 2006-10-19T17:52:00.000-05:00
updated = 2006-12-11T15:57:05.095-06:00
draft = false
url = '/2006/10/internet-explorer-7-bogus-repros-on.html'
tags = ["Microsoft","IE","bug","Internet Explorer","Secunia"]
+++

By now you all realize that [IE7 has been released](http://www.microsoft.com/windows/ie/default.mspx "Download Internet Explorer 7 here") on the 18th. And now we have a report of the first vulnerability. This one is purported to be a problem with the way that IE handles redirections that specify the `mhtml:` URI handler. The problem may exist or not, but I can tell you that the "test" for it over on Secunia is flawed at best. I watched the "test" of IE7 RTM under Fiddler, what it does is use an XMLHttpRequest to do a request from their site, which returns a `[302 Found](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html "HTTP status codes defined")` redirect requesting `mhtml:http//...` (again from their site), which then gets processed (not sure what that indicates, and what the point of the mhtml URI handler is, other than the way that we're supposed to lose track of the hosting domain, but whatever). That request returns another `302 Found` redirect to http://news.google.com, then they test the response to see if it contains "news.google" in it. **However**, my browser never followed that other redirect, so I can't fathom how their test is remotely valuable. Fiddler clearly shows **only** two requests. Here's the complete session after clicking the "test it" link:

```
------------------------------------------------------------------
GET /ie_redir_test_1/?0.9495259804243511 HTTP/1.1
Accept: */*
Accept-Language: en-us
Referer: http://secunia.com/Internet_Explorer_Arbitrary_Content_Disclosure_Vulnerability_Test/
UA-CPU: x86
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)
Host: secunia.com
Proxy-Connection: Keep-Alive
Cookie: SECUNIA=XRdqbl8qUUgk9kpJInxPBmogDpbYp0RNS5E3rMtL;
__utma=25207103.253302053.1161293305.1161293305.1161293305.1;
__utmb=25207103; __utmc=25207103;
__utmz=25207103.1161293305.1.1.utmccn=(direct)|utmcsr=(direct)|utmcmd=(none)


HTTP/1.1 302 Found
Date: Thu, 19 Oct 2006 21:41:11 GMT
Server: Apache
Location: mhtml:http://secunia.com/ie_redir_test_2
Transfer-Encoding: chunked
Content-Type: text/html

0

------------------------------------------------------------------
GET /ie_redir_test_2 HTTP/1.1
Accept: */*
UA-CPU: x86
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)
Host: secunia.com
Proxy-Connection: Keep-Alive
Cookie: SECUNIA=XRdqbl8qUUgk9kpJInxPBmogDpbYp0RNS5E3rMtL;
__utma=25207103.253302053.1161293305.1161293305.1161293305.1;
__utmb=25207103; __utmc=25207103;
__utmz=25207103.1161293305.1.1.utmccn=(direct)|utmcsr=(direct)|utmcmd=(none)


HTTP/1.1 302 Found
Date: Thu, 19 Oct 2006 21:41:11 GMT
Server: Apache
Location: http://news.google.com/
Transfer-Encoding: chunked
Content-Type: text/html

0

--------------------------------------------------------------------
```

I've been running the recent release candidates for a while now, and I really like it. It troubles me that this sort of "reporting" is already happening. I still remember "staying up till midnight" to download it way back when [IE 3.0](http://en.wikipedia.org/wiki/Internet_Explorer#History "History of Internet Explorer") was released on August 13, 1996 (bringing CSS support, ActiveX, Java applets, inline multimedia t to a mainstream browser for the first time!), and I still have [the t-shirt](http://www.richardgiles.net/blog/archives/000204.html "Midnight Madness").

In fact a few years ago that shirt almost got me in some trouble. I had several years earlier bought a whole bunch of dumb terminals ([Televideo 950](http://www.cs.utk.edu/~shuford/terminal/televideo.html "More information about Televideo terminals than you really need.")'s to be precise) and modems (U.S. Robotics 1200 baud!) that I had been bundling with appropriate cables to use as [BBS](http://www.bbsdocumentary.com/ "The history of computer bulletin board systems.") machines. I made a killing, buying nearly a hundred terminals and modems and tons of other hardware for a total of $56. On average I got $20 per "setup", and sold a bunch of the other stuff too. The problem was that I bought far more than I could ever sell, and the Internet was starting to catch on an BBS's were going out of fashion. While you could use a dumb-terminal for [PINE](http://en.wikipedia.org/wiki/Pine_%28e-mail_client%29 "Program for Internet News & Email") and [LYNX](http://en.wikipedia.org/wiki/Lynx_%28web_browser%29 "the text-only web browser and gopher client") on a shell account, my market was drying up.

After a few years of moldering in the basement, my wife handed down the law and I was tasked with ridding myself of the excess. The problem, until then unforeseen, was that dumb terminals are fancy televisions, and televisions (in those days) contain CRT tubes. CRT tubes contain [phosphor](http://en.wikipedia.org/wiki/Phosphor "It glows!") and phosphor is considered hazardous waste. No wonder they wanted to get rid of all those dumb terminals!

So, left with the the prospect of paying large amounts of money to get them disposed of properly, and not having all the cool resources we have these days, I did what any young and stupid guy would do. I threw them away. No in **my** garbage, of course... no, I went from strip mall to strip mall, tossing a couple terminals in random dumpsters. Knowing that this would look a tad suspicious, I (being young and stupid) decided to wear black jeans, black shirt, and do this at 3 in the morning. That's where the I Downloaded t-shirt comes into the story...

I chose that shirt because it was black... new moon black... shadow black... what I **didn't realize** was that the cool lettering and logos printed on the shirt were glow-in-the-dark ink. So, there I was, in the back of a strip mall, with my car idling but the lights off, under a new moon wearing black pants and a black shirt that was **glowing,** declaring **brightly** that "I downloaded Midnight August 13, 1996" on the back and "Midnight Madness" on the front.

Madness, indeed.
