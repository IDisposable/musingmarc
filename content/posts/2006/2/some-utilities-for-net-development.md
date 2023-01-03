+++
title = "Some utilities for .Net development"
date = 2006-02-16T19:21:00.001-06:00
updated = 2008-03-09T03:54:58.220-05:00
draft = false
url = '/2006/02/some-utilities-for-net-development.html'
tags = [".Net","DisposableAction","Double","C#","UndoableAction","DateTime","GetHashCode","HTML","Dynamic","Serialization","DynamicComparer","ArgumentValidation"]
+++

I've had several requests for a package of my various code snippets. I'm releasing these for anyone to use \[download [here](http://idisposable.googlepages.com/Utilities.zip)\].

Some are based on prior postings, some are things I've gathered from across the net for other projects, some are even things I wrote myself. Everything is packaged as a single Assembly. This code requires at least .Net 2.0 and is in C#.

*   There is one class `Utilites` which is a partial class containing a ton of `String`, `Double`, `DateTime`, HTML, and serialization helper methods. This class is marked partial, so you can add members anytime.
*   There is the `ArgumentValidation` class mentioned [here](http://musingmarc.blogspot.com/2006/01/argument-validation-class-looks-tons.html).
*   There is the `DynamicComparer` and `SortProperty` class mentioned [here](http://musingmarc.blogspot.com/2006/02/dynamic-sorting-of-objects-using.html).
*   Finally there are `UndoableAction` and `DisposableAction` inspired by a cool posting by Oren Eini [here](http://www.ayende.com/Blog/2005/12/07/TheUltimateDisposable.aspx)

If you make any improvements or have any comments, please let me know.

**UPDATED:** On 9 March, 2008 I updated the Utilities library to include fixes in my CombineHashCodes methods.

---
### Comments:
#### Some useful stuff in there.  
  
Off hand, if I ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-02-17T09:41:00.000-06:00">Feb 5, 2006</time>

Some useful stuff in there.  
  
Off hand, if I were to use some of it I would have to change some things to globalize them.  
  
IsAlpha() and IsDecimal(), for example assume a North American locale. I would probably make use of Char.IsLetter(), Char.IsDigit(), NumberFormatInfo.NumberDecimalSeparator, NumberGroupSeparator, etc.  
  
And, just for my curiousity: was DateTime.IsLeapYear() avoided for a specific reason?
<hr />
#### Yeah, the context I used those tools in was US onl...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-02-17T10:29:00.000-06:00">Feb 5, 2006</time>

Yeah, the context I used those tools in was US only... definately can put in much better RegEx values.  
  
DateTime.IsLeapYear I thinkI needed static method for databinding use. I don't remember :)
<hr />
