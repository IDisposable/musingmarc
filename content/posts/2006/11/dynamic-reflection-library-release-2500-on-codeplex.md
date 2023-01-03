+++
title = "Dynamic Reflection Library release 2.5.0.0 on CodePlex"
date = 2006-11-03T05:08:00.000-06:00
updated = 2006-12-11T15:57:50.922-06:00
draft = false
url = '/2006/11/dynamic-reflection-library-release.html'
tags = [".Net","LCG","CodePlex","DynamicMethod","Dynamic","C#","lightweight code generation","MethodInfo","Reflection"]
+++

I've updated [my](http://musingmarc.blogspot.com/2006/08/how-to-do-late-dynamic-method-creation.html) [Dynamic](http://musingmarc.blogspot.com/2006/07/power-of-generics-compels-you-power-of.html) LCG\-[based](http://musingmarc.blogspot.com/2006/07/things-to-remember-when-creating.html) Reflection [wrapper](http://musingmarc.blogspot.com/2006/07/updated-files-for-dynamic-method.html) tonight. This new version finally has extensive XML documentation in place. I also refactored a little bit of the code to eliminate duplicate calls and sped up the code a tiny bit.

As of this release, I'm actually 6% **faster** than native calls (e.g. the code emitted by the C# compiler is actually slower) against reference-types when doing explicit binding to methods. This means that there is realistically little to no cost doing name-based binding to arbitrary methods at runtime.

Head on over to the [CodePlex project](http://www.codeplex.com/Wiki/View.aspx?ProjectName=Dynamic) for more information and the downloads.

Technorati tags: [LCG](http://technorati.com/tags/LCG), [DynamicMethod](http://technorati.com/tags/DynamicMethod), [Lightweight Code Generation](http://technorati.com/tags/Lightweight%20Code%20Generation), [Reflection.Emit](http://technorati.com/tags/Reflection.Emit), [CodePlex](http://technorati.com/tags/CodePlex), [MethodInfo](http://technorati.com/tags/MethodInfo), [FieldInfo](http://technorati.com/tags/FieldInfo), [PropertyInfo](http://technorati.com/tags/PropertyInfo), [ConstructorInfo](http://technorati.com/tags/ConstructorInfo), [.Net 2.0](http://technorati.com/tags/.Net%202.0)
