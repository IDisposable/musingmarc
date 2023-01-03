+++
title = "New release of UriTemplate / UriPattern library"
date = 2007-10-26T19:51:00.001-05:00
updated = 2011-07-21T16:42:47.835-05:00
draft = false
url = '/2007/10/new-release-of-uritemplate-uripattern.html'
tags = ["CodePlex","UriTemplate","UriPattern","Source","URI"]
+++

Today I released a new version of the UriPattern and UriTemplate library on CodePlex ([previously announced here](http://musingmarc.blogspot.com/2007/06/uritemplate-project-on-codeplex.html)). There are two changes in this release:

1.  A bug reported by Darrel Miller where the meta character that have special meaning in Regex expressions are not properly escaped. I inherited this bug in the original implementation I based the library on, but no excuses, this was stupid. Sorry to anyone bit by this.
2.  I've added the ability to specify that a UriPattern should be compiled. This should speed up patterns that are used very frequently.

Pick up [Release 1.1 on CodePlex](http://www.codeplex.com/UriTemplate)
