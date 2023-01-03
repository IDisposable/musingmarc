+++
title = "Statistics aren't easy, good design is just as hard. Want to see both?"
date = 2005-09-27T09:17:00.000-05:00
updated = 2006-06-14T13:20:30.480-05:00
draft = false
url = '/2005/09/statistics-arent-easy-good-design-is.html'
tags = []
+++

I've been a fan of Jeffrey Sax's work in the mathematical area for .Net for quite a while (since he and I helped out a developer on CodeProject with a [Fractions library](http://www.codeproject.com/csharp/Fractiion.asp) for .Net). Today his company, Extreme Optimization, released the Statistics Library for .Net. This is a **major** boon for those doing statistical work in .Net.  
The [Extreme Optimization Statistics Library for .NET](http://www.extremeoptimization.com/Statistics/Default.aspx) contains classes for probability distributions, random number generation, hypothesis tests, and statistical models, including linear regression and analysis of variance. The library is easy to use without compromising performance or reliability.  

*   Supports 25 probability distributions
*   Uses a General Linear Model (GLM) approach for unified treatment of regression and ANOVA models
*   Integrates inference and validation tests with statistical models
*   Provides 4 robust pseudo-random number generators
*   Is fully compliant with Microsoft's Design Guidelines for Class Library Developers.

  
That last point is something near and dear to my heart. All **libraries** should be written to comply with those guidelines. It could be argued that some of the guidelines are not perfect, but adhering to them leads to a library that _feels_ like the FCL, making it easy to learn.  
Brad Abrams, the lead program manager on the Microsoft .NET Framework team and co-author of [Framework Design Guidelines : Conventions, Idioms, and Patterns for Reusable .NET Libraries](http://www.amazon.com/exec/obidos/ASIN/0321246756/marcsmusing0a-20) has this to say about Extreme Optimization and its products:

> I have made it my mission to institutionalize the value of good API design. I strongly believe that this is key to making developers more productive and happy on our platform. It is clear that Extreme Optimization values good API design in their work, and takes to heart developer productivity and synergy with the .NET framework.

In short, I **strongly** recommend this product I also strongly recommend Brad Abrams' new book [Framework Design Guidelines](http://www.amazon.com/exec/obidos/ASIN/0321246756/marcsmusing0a-20).
