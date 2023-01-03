+++
title = "The power of generics compels you, the power of generics compels you..."
date = 2006-07-01T03:21:00.000-05:00
updated = 2007-11-19T00:05:29.724-06:00
draft = false
url = '/2006/07/power-of-generics-compels-you-power-of.html'
tags = ["Emit","LCG","DynamicMethod","IL","Dynamic","lightweight code generation","MethodInfo","Reflection","Nullable"]
+++

Once again I've devoted some time to the exorcism of the daemons of `Reflection`. This time, I'm chasing the cost of calling `MethodInfo.Invoke` against an arbitrary (chosen at run-time) method.

In previous efforts, I've tried to give you [generic sorting](http://musingmarc.blogspot.com/2006/02/dynamic-sorting-of-objects-using.html) with as little cost as possible, added support for [`Enum` properties](http://musingmarc.blogspot.com/2006/03/extending-dynamic-sorting-of-objects.html), improved upon it and [added support](http://musingmarc.blogspot.com/2006/02/dynamic-sorting-of-objects-using.html#115151889470600720) for access to `struct`s (i.e. `ValueType`s) and `Nullable<>`. Along the way, I learned a **lot** about IL and the rules for using `Reflection.Emit` and `DynamicMethod` for LCG.

Now the cost of calling any arbitrary method of a class isn't quite as apparent as when you're trying to do dynamic comparisons, but it's still significant. Using my new `DynamicFunction` and `DynamicProcedure` classes is really noticeable. Running against the same test-jig Person class and Animal struct used in the `DynamicComparer` sample program, I get the following performance chart executing three methods against 500000 objects:

> | Method | Elapsed time (seconds) | Explanation |
> | --- | --- | --- |
> | Compile-time | :00.305 | The test method simply executes the method calls directly with no dynamic choices. This is the baseline for comparison, as good as it can get. |
> | Dynamic Strong | :00.591 | This is called using the strong-typed delegate form, where all the argument types are known and correctly specified. The specific methods called are specified by a `string` method name. |
> | Dynamic Weak | :00.776 | This is called using the weak-typed delegate form, where all the arguments are passed as a `params object[]` object, but are of the correct type. The specific methods called are specified by a `string` method name. |
> | Reflection | :18.243 | This is called using the a standard `MethodInfo.Invoke`, where all the arguments are passed as a `new object[]` object, but are of the correct type. The specific methods called are specified by a `string` method name. _This is **31 times slower** than the strong-type form!_ |

While building this, I've extensively refactored the logic for the `Reflection.Emit` into another class. The new `DynamicEmit` encapsulates the `ILGenerator` used during the synthesis of the `DynamicMethod` in both the old `DynamicComparer` and the new classes.

### DynamicFunction

This class allows you to call any method (instance or static) of a class with any return type. There are two supported forms of the delegate.

*   The first form is a weak-typed delegate that takes an `params object[]` for any arguments needed. This version is much faster than plan old `MethodInfo.Invoke`, but about 30% slower than the strong-typed delegate form. You instantiate and call the method like this, given a target method of `bool YourClass.MethodToCall(int, string)`:
    ```
    YourClass target = new YourClass();
    Func<T, bool> method = DynamicFunction<YourClass, bool>.Initialize('MethodToCall");
    bool result = method(target, 1, "Hi");
    ```
    
    The two arguments are automatically wrapped up by C# in a new `object[]` and then unwrapped and coerced into the target method's types (if needed). The wrapping and unwrapping is the source of most of the performance difference between this and the strong-type form.
*   The second form is a strong-typed delegate that takes an `params object[]` for any arguments needed. This version is much faster than plan old `MethodInfo.Invoke`, but about 50% slower than directly coded calls, but gives you the obvious advantage of variability. You instantiate and call the method like this, given the same target method of `bool YourClass.MethodToCall(int, string)`:
    ```
    YourClass target = new YourClass();
    Func<T, bool, int, string> method = DynamicFunction<YourClass, bool, int, string>.Initialize('MethodToCall");
    bool result = method(target, 1, "Hi");
    ```
    
    The two arguments are now passed directly to the delegate, which will be coerced into the target method's types (if needed, not usually).

Note that any type-coercision is only that which can be done by a implicit, so in general you should code the argument types "correctly", plus when the types are the same the code is shorter and faster.

### DynamicProcedure

This class allows you to call any method (instance or static) of a class that doesn't return a value (e.g. it's a `void` method). There are two supported forms of the delegate.

*   The first form is a weak-typed delegate that takes an `params object[]` for any arguments needed. This version is much faster than plan old `MethodInfo.Invoke`, but about 30% slower than the strong-typed delegate form. You instantiate and call the method like this, given a target method of `void YourClass.VoidMethodToCall(int, string)`:
    ```
    YourClass target = new YourClass();
    Proc<T> method = DynamicProcedure<YourClass>.Initialize('VoidMethodToCall");
    method(target, 1, "Hi");
    ```
    
    The two arguments are automatically wrapped up by C# in a new `object[]` and then unwrapped and coerced into the target method's types (if needed). The wrapping and unwrapping is the source of most of the performance difference between this and the strong-type form.
*   The second form is a strong-typed delegate that takes an `params object[]` for any arguments needed. This version is much faster than plan old `MethodInfo.Invoke`, but about 50% slower than directly coded calls, but gives you the obvious advantage of variability. You instantiate and call the method like this, given the same target method of `void YourClass.VoidMethodToCall(int, string)`:
    ```
    YourClass target = new YourClass();
    Proc<T, bool, int, string> method = DynamicProcedure<YourClass, int, string>.Initialize('MethodToCall");
    bool result = method(target, 1, "Hi");
    ```
    
    The two arguments are now passed directly to the delegate, which will be coerced into the target method's types (if needed, not usually).

Note that any type-coercision is only that which can be done by a implicit, so in general you should code the argument types "correctly", plus when the types are the same the code is shorter and faster.

### What does the code look like?

```
class Person
{
    public Person(string name) { ... }
    public bool Compatible(Person potentialMate) { ... }
    public Person Breed(Person mate, Gender childGender) { ... }
    public void Mutate() { ... }
}
 
Func<Person, bool, Person> compatible = DynamicFunction<Person, bool, Person>.Initialize("Compatible");
Func<Person, Person, Person, Gender> breed = DynamicFunction<Person, Person, Person,  Gender>.Initialize("Breed");
Proc<Person> mutate = DynamicProcedure<Person>.Initialize("Mutate");
 
Person child;
 
if (compatible(new Person("Marc"), new Person("Beth"))
    child = breed(you, me, Gender.Female);
else
    mutate(me);
```

### What does the emitted IL code look like?

A sample of the emitted code, in weak-typed form call for the above bool Compatible(Person) looks like this:

```
IL_0000: /* 03  |          */ ldarg.1    
IL_0001: /* 16  |          */ ldc.i4.0   
IL_0002: /* 9a  |          */ ldelem.ref 
IL_0003: /* 74  | 02000002 */ castclass  DynamicComparerSample.Person
IL_0008: /* 0a  |          */ stloc.0    
IL_0009: /* 02  |          */ ldarg.0    
IL_000a: /* 06  |          */ ldloc.0    
IL_000b: /* 28  | 0A000003 */ call       Boolean Compatible(DynamicComparerSample.Person)/DynamicComparerSample.Person
IL_0010: /* 2a  |          */ ret
```

A sample of the (much simpler) emitted code, in strong-typed form call for the above bool Compatible(Person) looks like this:

```
IL_0000: /* 02  |          */ ldarg.0    
IL_0001: /* 03  |          */ ldarg.1    
IL_0002: /* 28  | 0A000002 */ call       Boolean Compatible(DynamicComparerSample.Person)/DynamicComparerSample.Person
IL_0007: /* 2a  |          */ ret
```

As always, download [here](http://idisposable.googlepages.com/downloads)

---
### Comments:
#### Pedantry compels me to tell you that you spelled "...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-07-10T15:16:00.000-05:00">Jul 1, 2006</time>

Pedantry compels me to tell you that you spelled "compels" wrong. FYI.
<hr />
#### I'm a moron and KNOW I can't spell. Spell checking...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-07-10T15:25:00.000-05:00">Jul 1, 2006</time>

I'm a moron and KNOW I can't spell. Spell checking the body is automatic, but the subject line? Thanks.
<hr />
#### wow. I'm speechless. here I am trying to write mys...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-08-17T00:13:00.000-05:00">Aug 4, 2006</time>

wow. I'm speechless. here I am trying to write myself some IL by hand to do what you've virtually automated via generics. I can see that I have a WHOLE lot to learn, but in the meantime, I think I'll have a look at Dynamic.cs and drink another diet mt. dew. Mahn! Very cool stuff, you win man.
<hr />
#### Do you still have the test case for this project.....
[Unknown](https://www.blogger.com/profile/16068263371859826097 "noreply@blogger.com") - <time datetime="2008-05-26T20:45:00.000-05:00">May 1, 2008</time>

Do you still have the test case for this project... I am running my own test and I couldn't replicate the result. I am testing the ToString() method for the Person class.
<hr />
#### The test project is part of the Dynamic library. H...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-05-26T21:42:00.000-05:00">May 1, 2008</time>

The test project is part of the Dynamic library. Head over to CodePlex and download [the Dynamic library](http://www.codeplex.com/Dynam<br/>ic). By the way, ToString() is not necessarily a very quick method dues to all the string manipulation, so its execution time might be swamping the runtime of all frameworks. This is a hazard of any micro-benchmarks...
<hr />
#### What about generic procedures and functions? I.e. ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2008-07-15T02:40:00.000-05:00">Jul 2, 2008</time>

What about generic procedures and functions? I.e. T foo>T<( v1, v2, v3 ) or void bar>T<( T, T, T )?
<hr />
#### Doing binding to generics isn't really much differ...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-07-15T14:38:00.000-05:00">Jul 2, 2008</time>

Doing binding to generics isn't really much different, except you have to "fill in the blanks". I talk a little about that in [this post about late-bound dynamic](http://musingmarc.blogspot.com/2006/08/how-to-do-late-dynamic-method-creation.html), but if you have some ideas of how I can make it more transparent (like with a fluent interface) please give me an idea.
<hr />
