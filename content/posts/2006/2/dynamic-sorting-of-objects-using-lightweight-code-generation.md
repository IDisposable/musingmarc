+++
title = "Dynamic sorting of objects using lightweight code generation."
date = 2006-02-15T12:58:00.000-06:00
updated = 2008-01-14T13:43:52.532-06:00
draft = false
url = '/2006/02/dynamic-sorting-of-objects-using.html'
tags = ["Emit","LCG","DynamicMethod","IL","Dynamic","lightweight code generation"]
+++

**UPDATE:** This project is hosted on [CodePlex](http://www.codeplex.com/) as the [Dynamic Reflection Library](http://www.codeplex.com/Dynamic) for all further updates.  

I looked into the cool [Dynamic List Sorting](http://www.codeproject.com/dotnet/dynamiclistsorting.asp) package on CodeProject. It's pretty good, but I wanted to get the absolute best performance for my framework, so I optimized the generated IL some \[download [here](http://idisposable.googlepages.com/Downloads)\]. I've eliminated the local variable used to track the comparison- thus-far as it is always zero until we've got a mismatch. I've also added the ability to reference fields (as well as properties), and added flags to the DynamicMethod constructor to bypass accessiablity checks (allows access to private members).

This new version is very fast. In debug builds, property access is around 5-25% slower than field access, the cost of delegate invocation is about 15% and the cost of element-by-element versus using the object's ICompare method is 4%.

Here's the chart for sorting 500,000 Person objects (in seconds):

| Sort By | Field | Property | Cost of property | Cost of dynamic |
| --- | --- | --- | --- | --- |
| Age (double) | 2.17 | 2.70 | 24.42% |
| LastName (string) | 2.60 | 2.76 | 6.15% |
| LastName,FirstName,Age | 7.17 | 7.99 | 11.44% | 3.21% |
| Dynamic whole object | 7.40 |  |  | 15.09% |
| Built-in whole object | 6.43 |

Sorting by "FirstName, lastName, Age" \[property,field,property respectively\] results in this generated IL (through the cool DebuggerVisualizer mentioned [here](http://musingmarc.blogspot.com/2006/02/lightweight-code-generation-is-fun.html))

```
IL_0000: ldarg.0  
IL_0001: callvirt   System.String get_FirstName()/DynamicComparerSample.Person
IL_0006: ldarg.1  
IL_0007: callvirt   System.String get_FirstName()/DynamicComparerSample.Person
IL_000c: callvirt   Int32 CompareTo(System.String)/System.String
IL_0011: dup      
IL_0012: brtrue.s   IL_0046
IL_0014: pop      
IL_0015: ldarg.0  
IL_0016: ldfld      System.String lastName/DynamicComparerSample.Person
IL_001b: ldarg.1  
IL_001c: ldfld      System.String lastName/DynamicComparerSample.Person
IL_0021: callvirt   Int32 CompareTo(System.String)/System.String
IL_0026: dup      
IL_0027: brtrue.s   IL_0046
IL_0029: pop      
IL_002a: ldarg.0  
IL_002b: callvirt   Double get_Age()/DynamicComparerSample.Person
IL_0030: stloc      V_0
IL_0034: nop      
IL_0035: nop      
IL_0036: ldloca.s   V_0
IL_0038: nop      
IL_0039: nop      
IL_003a: nop      
IL_003b: ldarg.1  
IL_003c: callvirt   Double get_Age()/DynamicComparerSample.Person
IL_0041: callvirt   Int32 CompareTo(Double)/System.Double
IL_0046: ret
```

**Edit:** I forgot the call to .Compare in the benchmark, which saves a LOT of time because then the object isn't wrapped in another delegate deep in the guts of the FCL. Numbers above have been updated.

**UPDATE:** Revised extensively to handle nulls and faster performance. See [this](http://musingmarc.blogspot.com/2006/03/extending-dynamic-sorting-of-objects.html).

**UPDATE:** Revised links to point to GooglePages so I'm not serving the files from my DSL.

---
### Comments:
#### Rich reported a bug (oops, deleted the comment) wh...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-03-01T11:45:00.000-06:00">Mar 3, 2006</time>

Rich reported a bug (oops, deleted the comment) when comparing Enum values. I've updated the DynamicComparer and Utilities zips to have the fix. Seems you need to box Enum(s) before calling System.Enum.CompareTo() -- odd...
<hr />
#### Hi Marc, I'm trying to sort strings where some of ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-03-09T11:11:00.000-06:00">Mar 4, 2006</time>

Hi Marc, I'm trying to sort strings where some of them are null and I am getting NullReferenceException. Is it by design? I don't have any experience in IL. How should I change IL code to enable sorting of null strings too? Thanks!
<hr />
#### I've got it! I implemented there my custom Compare...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-03-09T12:01:00.000-06:00">Mar 4, 2006</time>

I've got it! I implemented there my custom Compare method instead of CompareTo method and changed Callvirt to Call. Thanks anyway for great DynamicComparer!
<hr />
#### Updated the code (extensively) to handle reference...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-03-09T18:43:00.000-06:00">Mar 4, 2006</time>

Updated the code (extensively) to handle reference-types that are nullable. This adds a little to the overhead, but prevents you from having to special-case the CompareTo methods. See [this](http://musingmarc.blogspot.com/2006/03/marcs-musings-dynamic-sorting-of.html) post.
<hr />
#### Revisions are
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-03-17T16:46:00.000-06:00">Mar 5, 2006</time>

Revisions are [here](http://musingmarc.blogspot.com/2006/03/extending-dynamic-sorting-of-objects.html)
<hr />
#### Any idea how to make this work with a list of stru...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-05-16T19:27:00.000-05:00">May 2, 2006</time>

Any idea how to make this work with a list of structs instead of a list of classes? It throws an AccessViolation ever time I try.
<hr />
#### If you've got a class (or list) that shows this er...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-05-16T21:38:00.000-05:00">May 2, 2006</time>

If you've got a class (or list) that shows this error, I'll extend it to handle that situation. BTW, I can't contact you if you post anonymously!  
  
If desired, mail me directly at IDisposable@gmail.com
<hr />
#### The class now handles structs just fine, see downl...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-06-28T13:21:00.000-05:00">Jun 3, 2006</time>

The class now handles structs just fine, see download link at top of the post.
<hr />
#### Are you still working on this? I am using nhiberna...
[Anonymous]( "noreply@blogger.com") - <time datetime="2008-01-14T13:01:00.000-06:00">Jan 1, 2008</time>

Are you still working on this? I am using nhibernate to populate my objects and works fine as long as the object is completely populated. I modified your code to accept nested objects and that works. but now if an object has a nested object that has a value of null but is not a nullable object it blows up when tring to compair the two. Any suggestions?
<hr />
#### I've put this project up on CodePlex for further d...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-01-14T13:38:00.000-06:00">Jan 1, 2008</time>

I've put this project up on CodePlex for further development, so it would be best to open an issue there. If you can give me a simple (not full NHibernate) example that I can include in the test suite, I will **happily** fix it.
<hr />
