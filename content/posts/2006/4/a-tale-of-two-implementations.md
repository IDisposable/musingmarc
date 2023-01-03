+++
title = "A tale of two implementations"
date = 2006-04-17T16:56:00.000-05:00
updated = 2006-04-20T12:13:40.313-05:00
draft = false
url = '/2006/04/tale-of-two-implementations.html'
tags = []
+++

Doing dynamic programming is all the rage these days. Some of the biggest proponents seem to be those that practice TDD. To that style of development, everything is fair game and dependency injection is used to solve many issues. Those injections are usually based on some reference to a provider of some sort. Others, caught in the declarative world of modern databinding are using named properties of objects.

Both worlds have need of the ability to link up the desired behavior, and they usually accomplish it through string constants that point at the desired provider or property. This is where one of my oldest code-smells kicks in; you can't dependency-check a string constant very well. The TDDers will tell you, "No problem, Mon! The tests will catch that." Old hackers like me don't want to wait that long (and that's one of the reasons type-less languages have so little appeal to me). We want something that will catch the errors for us immediately. Using things like Visual Assist's red-squiggle, the compiler's error messages, or even the IDE's cool automatic scoped rename ability keeps me from having to even **run** the test to catch the stupids.

So back to the string constants. How do we get the compiler/IDE to understand the connection? By making it a **code** link, of course. We want to replace those string literals with actual references, in some form, to the actual property or provider. This isn't a new problem, so it should come as no surprise that it's been tackled a couple of times recently for .Net development.

Let us study a couple examples _and by **us** I mean you, I've finished my homework_.

The **wrong** way: [Get rid of nasty "string"](http://jroller.com/page/viveksingh123?entry=get_rid_of_nasty_strings)

The **right** way: [Static Reflection](http://www.ayende.com/Blog/2005/10/29/StaticReflection.aspx)

Okay, are you back now? I'll wait, really...

What I don't like about the first idea is that it reads HORRIBLY. The code's intent is completely non-obvious, it creates all kinds of proxies where none are needed. It's ugly. Ayende's solution works without making any significant objects at runtime. The whole thing is just compiled away into a single delegate which will likely get optimized out of existance anyway.

What's the moral? If the code you are replacing is much easier to understand and maintain than the code you are using instead... you are not done refactoring.

**Update**: Of course this breaks down at pointing to properties because C# is overprotective, read the comments for the best-until-they-lighten-up-at-C#-land.

---

### Comments

#### Actually, I can see his point, you can't use my ap…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-04-17T17:15:00.000-05:00">Apr 1, 2006</time>

Actually, I can see his point, you can't use my approach for this stuff.
The compiler just won't let you do it.
  
I gave it a try, and you can't get a delegate instance from a property, since the compiler thinks that you are trying to get the \_value\_ of the property, and not the getter method.

---

#### Yes, delegates to the property accessors cannot be…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-04-17T17:57:00.000-05:00">Apr 1, 2006</time>

Yes, delegates to the property accessors cannot be generated in C# 2.0 (I hope it is something they fix for C# 3.0). In the meantime, I use another wrapper function like GetAge() which merely returns this.Age; To get the name of the method, you need something like this in the StaticReflection.cs file:

```csharp
  public static String MethodName<TRet>(Func<TRet> func0)  
  {  
    return func0.Method.Name;  
  }  
   
  public static String MethodName<TRet, A0>(Func<TRet, A0> func1)  
  {  
    return func1.Method.Name;  
  }
```

---

#### This is the best I can do…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-04-17T19:03:00.000-05:00">Apr 1, 2006</time>

This is the best I can do:

```csharp
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Reflection;  
   
namespace Whee  
{  
    class Person  
    {  
        public int _age;  
   
        public int Age  
        {  
            get { return _age; }  
            set { _age = value; }  
        }  
   
        // this method only exists to wrap the property get  
        public int GetAge()  
        {  
            return _age;  
        }  
   
        public Person()  
        {  
            _age = 42;  
        }  
    }  
   
    class Program  
    {  
        public delegate TRet Func<TRet>();  
   
        public static MethodInfo MethodInfo<TRet>(Func<TRet> func0)  
        {  
            return func0.Method;  
        }  
   
        public static string MethodName<TRet>(Func<TRet> func0)  
        {  
            return func0.Method.Name;  
        }  
   
        delegate int GetAge();  
   
        static void Main(string[] args)  
        {  
            Person me = new Person();  
            //GetAge getter = me.Age;     // cannot implicitly converty 'int' to 'GetAge'  
            //GetAge getter = me.get\_Age; // cannot explicitly call operator or accessor  
            GetAge getter = me.GetAge;  
            Console.WriteLine(getter.ToString() + " returns: " + getter.Invoke());  
   
            MethodInfo getterInfo = MethodInfo<int>(me.GetAge);  
            Console.WriteLine(getterInfo.ToString() + " returns: " + getterInfo.Invoke(me, null));  
   
            String name = MethodName<int>(me.GetAge);  
            Console.WriteLine(name);  
        }  
    }  
}
```

---

#### If you want to pass properties by reference then y…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-04-20T12:13:00.000-05:00">Apr 4, 2006</time>

If you want to pass properties by reference then you should switch to Visual Basic. There are some VB features which C# does still lack like passing properties by reference:

```vb
Module Module1  
  
Public Class PropertyByRef  
Public Delegate Function RetValDelegate() As Integer  
  
Public Sub New()  
Value = 10  
Func(Value) ' pass property by reference print it and change the value to 500  
Func(Value) ' pass it againt print the changed value  
  
End Sub  
  
Public Sub Func(ByRef value As Integer)  
System.Console.WriteLine("Got value " & value)  
value = 500  
End Sub  
  
Dim myValue As Integer  
Property Value() As Integer  
Get  
Return myValue  
End Get  
Set(ByVal value As Integer)  
myValue = value  
End Set  
End Property  
  
End Class  

Sub Main()  
  
Dim inst = New PropertyByRef()  
End Sub  
  
End Module  
```

Yours,
  
Alois Kraus

---

#### Ahhhhh, my eyes! VB code!!…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-04-20T12:17:00.000-05:00">Apr 4, 2006</time>

Ahhhhh, my eyes! VB code!!!
  
:) Seriously, this is just a stupid C# restriction that should be removed ASAP.

---
