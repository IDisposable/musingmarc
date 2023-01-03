+++
title = "How to do late Dynamic method creation"
date = 2006-08-18T16:23:00.000-05:00
updated = 2006-12-11T16:00:40.546-06:00
draft = false
url = '/2006/08/how-to-do-late-dynamic-method-creation.html'
tags = [".Net","LCG","DynamicMethod","IL","Dynamic","C#","lightweight code generation","MethodInfo"]
+++

In [this post comment](http://musingmarc.blogspot.com/2006/07/updated-files-for-dynamic-method.html#115592285105438051), I was asked about how to do creation of method delegates whose types can vary somewhat when using the `Dynamic` class. It's really a matter of using the unspecified form of a generic class and the `Type.MakeGenericType` method to synthesize up the desired fully specified type.  This code is going to look ugly, simply because playing with generics this way is more akin to writing a compiler than writing normal code.  Much of this code is boiler-plate and could easily be extracted into a set of helper methods... given some time, I'll do so in my current `Dynamic` class. Until then, here's an example of what things can look like:

```
// needed by both versions
BindingFlags creatorFlags = BindingFlags.FlattenHierarchy | BindingFlags.Public | BindingFlags.Static;
Type staticProc = typeof(Dynamic<>.Static.Procedure.Explicit<,,>);
 
// this version "knows" the final argument type of the method, and could have been done explicitly
// I show this to make obvious the way you use MakeGenericType.
Type doubleProc = staticProc.MakeGenericType(typeof(PutMethods), typeof(IIdentifiable), typeof(string), typeof(double));
MethodInfo doubleProcCreateDelegate = doubleProc.GetMethod("CreateDelegate", creatorFlags, null, new Type[] { typeof(string) }, null); 
StaticProc<PutMethods, IIdentifiable, string, double> proc =
   doubleProcCreateDelegate.Invoke(null, new object[] { "Add" }) as StaticProc<PutMethods, IIdentifiable, string, double>
proc(null, "Age", 1.0);
 
// this is more like a normal use, where the final argument type is not known at delegate generation time...
Type lazyProc = staticProc.MakeGenericType(typeof(PutMethods), typeof(IIdentifiable), typeof(string), typeof(object));
MethodInfo lazyProcCreateDelegate = lazyProc.GetMethod("CreateDelegate", creatorFlags, null, new Type[] { typeof(string) }, null);
StaticProc<PutMethods, IIdentifiable, string, object> lazy =
   lazyProcCreateDelegate.Invoke(null, new object[] { "Add" }) as StaticProc<PutMethods, IIdentifiable, string, object>
lazy(null, "Balance", -10.0);

  
public interface IIdentifiable
{
   void SetProperty(string propertyName, object value);
} 
 
public class PutMethods
{
   public static void Add(IIdentifiable dr, string propertyName, double value)
   {
      if (dr != null)
         dr.SetProperty(propertyName, value);
   }
}

```

Note that you can easily build up the `Type[]` passed to `Type.MakeGenericType` based on things like an attribute class or configuration/XML markup.

---

### Comments

#### :D Yes! Thanks! It's the missing piece to my puz…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-08-18T20:20:00.000-05:00">Aug 5, 2006</time>

:D Yes! Thanks! It's the missing piece to my puzzle! I owe you one!
---
