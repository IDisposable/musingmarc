+++
title = "Things to remember when creating delegates and doing LCG..."
date = 2006-07-25T12:24:00.000-05:00
updated = 2007-11-18T23:59:58.454-06:00
draft = false
url = '/2006/07/things-to-remember-when-creating.html'
tags = [".Net","LCG","DynamicMethod","Dynamic","C#","delegates","lightweight code generation"]
+++

In my `AcquiringObjectDataSource` articles [part three](http://musingmarc.blogspot.com/2006/07/i-can-see-clearly-now-pain-is-gone-or.html "Building AcquiringObjectDataSourceView...") I promised to show what things look like using the `DynamicCall` library I've [previously posted](http://musingmarc.blogspot.com/2006/07/power-of-generics-compels-you-power-of.html "Generic methods to ease lightweight code generation of delegates."). In doing that, I've had to make the `DynamicCall` stuff work with `static` methods. Previously I had inserted some code to start handling `statics` deep inside the `Build` methods, but I've certainly never gotten to the point of using them. Sorry to anyone that tried, I should have documented that code as not-yet working.

Anyway, last night I worked into the wee hours of the morning getting everything hooked up for `static` methods. In the process I learned some things that I thought I would dump now. In an upcoming post, I'll talk about the changes to the `DynamicCall` library, but for now:

*   When you create a delegate for a `static` method, the delegate signature should not include a "this" reference. Duh. This manifests itself in the error message below when calling `CreateDelegate`. This happens because the delegate definition expects a "this" pointer as the first method argument, but the generated `DynamicMethod` doesn't have one. This spawned a whole new set of `StaticProc` and `StaticFunc` delegate definitions.
    
    > ArgumentException: Error binding to target method
    
*   If you get the delegate definition correct, but don't generate the right code, it'll hurl at JIT with the message below. Note that the generation of the LCG `DynamicMethod` is fine, the delegate binding is fine. The JIT of the first caller to that delegate is where things will pop. This is often very in the runtime, which is bad.
    
    > System.InvalidProgramException: Common Language Runtime detected an invalid program.
    
*   Not strictly related to `static` method supprt, but I rediscovered that you can't access private members of a class unless the `DynamicMethod` was properly bound to that class. This resulted in me still having to know the class of the `static` method, even though there's no "this" pointer to deal with.
*   Ruthless refactoring makes things much easier. Doing this work on the `DynamicCall` library has really cleaned things up, I'll give more detail in a later post.

As it stands now binding to a `static` method now looks like this:

```
StaticProc<ObjectDataSourceView, ParameterCollection, IDictionary, IDictionary> s_MergeDictionaries =
    DynamicProcedure<ObjectDataSourceView, ParameterCollection, IDictionary, IDictionary>.Static.Initialize("MergeDictionaries", CreateParameterList.Auto);
    Dynamic<ObjectDataSourceView>.Static.Procedure.Explicit<ParameterCollection, IDictionary, IDictionary>.CreateDelegate("MergeDictionaries", CreateParameterList.Auto);
```

While this is really clear (I think), it is very verbose. The new C# 3.0 "var" feature will will make this much more readable:

```
var s_MergeDictionaries =
    DynamicProcedure<ObjectDataSourceView, ParameterCollection, IDictionary, IDictionary>.Static.Initialize("MergeDictionaries", CreateParameterList.Auto);
    Dynamic<ObjectDataSourceView>.Static.Procedure.Explicit<ParameterCollection, IDictionary, IDictionary>.CreateDelegate("MergeDictionaries", CreateParameterList.Auto);
```

**UPDATE** (27-July-2007 15:32): New syntax based on the [latest round of refactorings on the ~DynamicCall~Dynamic<> class.](http://musingmarc.blogspot.com/2006/07/updated-files-for-dynamic-method.html)
