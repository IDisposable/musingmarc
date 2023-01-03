+++
title = "Extending dynamic sorting of objects using lightweight code generation."
date = 2006-03-09T18:42:00.000-06:00
updated = 2008-01-14T13:45:22.127-06:00
draft = false
url = '/2006/03/extending-dynamic-sorting-of-objects.html'
tags = ["Emit","LCG","CodePlex","DynamicMethod","IL","Dynamic","lightweight code generation"]
+++

**UPDATE:** This project is hosted on [CodePlex](http://www.codeplex.com/) as the [Dynamic Reflection Library](http://www.codeplex.com/Dynamic) for all further updates.
RE: [Dynamic sorting of objects using lightweight code generation.](/2006/02/dynamic-sorting-of-objects-using.html)

I got a report of issues sorting objects that have null (not `Nullable<T>`) values. In the previous implementation, I blindly call down to the CompareTo method associated with the property's `Type`. This obviously doesn't work with virtual methods. For some types (like `String`) it was enough to simply make the emitted code a `Call` instead of a `CallVirt`, but I didn't want to be half-right. So the new version does the right thing and handles the null-checks for the left and right filed/property values.

The emitted code is slightly more complex, but still about as fast as I now check for `IsFinal` methods and use `Call` instead of `CallVirt` when possible. This optimization is in place for property get calls, and the call the CompareTo, so it is a bit faster.

I also fixed an issue if you compared a whole bunch of attributes and blew the generate IL so large that the loop-break label wasn't reachable by a short branch (nobody reported this, just noticed in code review).

I've updated both the [DynamicSorter.zip](http://idisposable.googlepages.com/DynamicComparer.zip) file and the [Utilites.zip](http://idisposable.googlepages.com/Utilities.zip) file (the latter has other updates to improve fxCop compliance).

The emitted code for "FirstName, lastName, Gender" \[`String` property, `double` field, `Enum` property, respectively\] now looks like this:

```il
IL_0000:  ldarg.0 
IL_0001:  callvirt   System.String get_FirstName()/DynamicComparerSample.Person
IL_0006:  dup     
IL_0007:  brtrue.s   IL_0018
IL_0009:  pop     
IL_000a:  ldarg.1 
IL_000b:  callvirt   System.String get_FirstName()/DynamicComparerSample.Person
IL_0010:  brtrue.s   IL_0015
IL_0012:  ldc.i4.0
IL_0013:  br.s       IL_0023
IL_0015:  ldc.i4.m1
IL_0016:  br.s       IL_0023
IL_0018:  ldarg.1 
IL_0019:  callvirt   System.String get_FirstName()/DynamicComparerSample.Person
IL_001e:  call       Int32 CompareTo(System.String)/System.String
IL_0023:  dup     
IL_0024:  brtrue     IL_0060
IL_0029:  pop     
IL_002a:  ldarg.0 
IL_002b:  ldfld      Double age/DynamicComparerSample.Person
IL_0030:  stloc.0 
IL_0031:  ldloca.s   V_0
IL_0033:  ldarg.1 
IL_0034:  ldfld      Double age/DynamicComparerSample.Person
IL_0039:  call       Int32 CompareTo(Double)/System.Double
IL_003e:  dup     
IL_003f:  brtrue     IL_0060
IL_0044:  pop     
IL_0045:  ldarg.0 
IL_0046:  callvirt   DynamicComparerSample.Gender get_Gender()/DynamicComparerSample.Person
IL_004b:  box        DynamicComparerSample.Gender
IL_0050:  ldarg.1 
IL_0051:  callvirt   DynamicComparerSample.Gender get_Gender()/DynamicComparerSample.Person
IL_0056:  box        DynamicComparerSample.Gender
IL_005b:  call       Int32 CompareTo(System.Object)/System.Enum
IL_0060:  ret
```

---

### Comments

#### Hi Marc…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-03-30T22:03:00.000-06:00">Mar 4, 2006</time>

Hi Marc,
  
I received your question from my blog. Emitting an ordinary 'call' for final methods, and 'callvirt' for others, is OK. However, you could actually just use 'callvirt's for all methods, and the CLR's JIT compiler will be intelligent enough to use a static dispatch when it knows it's safe to do so. This is what the C# 2.0 compiler does.
  
Cheers,
joe

---

#### Thanks for the input. I realized that the callvirt…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-03-30T23:25:00.000-06:00">Mar 4, 2006</time>

Thanks for the input. I realized that the `callvirt` was safe in either case, but my benchmarks actually showed a bit better performance for call. Odd. Is checking `IsFinal` the best way to know it's safe, since I **know** the type of the variable, it seems safe.

---

#### We have a list objects with a property that we are…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-03-06T06:41:00.000-06:00">Mar 2, 2007</time>

We have a list objects with a property that we are trying to sort by. The propery in question is another list of objects.
  
The first list made of an object with a property:

```vb
Property Offices as _List(of MarketingOffice_)  
...  
  
**MarketingOffice**  
Property Branch as _Office_  
Property IsPrimary as Boolean  
...  
**Office**  
Property Code as Char  
...  
```

We need to be able to sort by offices. The original list has two objects in.
  
Object 1:

```vb
Offices=
{MarketingOffice, Primary=True, Branch = Office, Code = H}
{MarketingOffice, Primary=False, Branch = Office, Code = M}
```

Object 2:

```vb
Offices=
{MarketingOffice, Primary=True, Branch = Office, Code = M}
```

We need to able to do a sort by Offices and have the results come out:
  
`H,M
M`
  
When use the dynamic comparer we get the following:
  
`GenericArguments[0], 'Business.Entities.MarketingOffice', on 'System.Nullable``1[\T]' violates the constraint of type parameter 'T'.`

It is throwing an exception at:

```csharp
public static bool IsComparable(Type valueType, out bool isNullable)  
{  
    isNullable = valueType.IsGenericType  
        && !valueType.IsGenericTypeDefinition  
        && valueType.IsAssignableFrom(typeof(Nullable<>).MakeGenericType(valueType.GetGenericArguments()[0]));  
    
    return (typeof(IComparable).IsAssignableFrom(valueType)  
            || typeof(IComparable<>).MakeGenericType(valueType).IsAssignableFrom(valueType)  
            || isNullable);
}
```
  
Please help...

---

#### If you can send me a simple class that has the pro…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2007-03-06T13:02:00.000-06:00">Mar 2, 2007</time>

If you can send me a simple class that has the properties you are trying to use, I'll happily fix it. You can attach it up on CodePlex.

---
