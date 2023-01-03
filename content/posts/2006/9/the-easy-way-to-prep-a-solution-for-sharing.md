+++
title = "The easy way to prep a solution for sharing"
date = 2006-09-05T13:04:00.000-05:00
updated = 2006-12-11T15:59:56.387-06:00
draft = false
url = '/2006/09/easy-way-to-prep-solution-for-sharing.html'
tags = [".Net","Visual Studio","tools"]
+++

How many times have you downloaded a .Net project from somewhere and unzipped it to find all kinds of kruft that should NEVER have been shipped to anyone? Things like .SUO files, .USER files, the obj and bin directories? Of course we sometimes forget to clean the solution out before zipping, or forget to prune the zip file (my personal strategy). Wouldn't it be cool to just automate that process in the fine tradition of ~lazy~ smart programmers everywhere?

Head over to [The Code Project](http://www.codeproject.com/) and check out [SolutionZipper](http://www.codeproject.com/useritems/SolutionZipper.asp), a Visual Studio 2005 Addin that cleans and zips a Solution in one step.

Thanks Robert Nadler!
