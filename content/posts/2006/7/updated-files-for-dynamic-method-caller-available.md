+++
title = "Updated files for Dynamic method caller available..."
date = 2006-07-27T03:13:00.000-05:00
updated = 2007-11-18T23:59:05.561-06:00
draft = false
url = '/2006/07/updated-files-for-dynamic-method.html'
tags = [".Net","LCG","CodePlex","DynamicMethod","Dynamic","lightweight code generation"]
+++

I've [updated the Utilities.zip and DynamicComparer.zip files](http://idisposable.googlepages.com/downloads) with my latest cleanup of the DynamicCall stuff. This version fully supports static methods is much simpler, uses less working set and is faster. A full post will follow on all the changes soon.

### BREAKING CHANGE

This version has an entirely new syntax that I think is clearer, plus it leads to much better generic-specialization / JIT memory usage. The new syntax looks like this:

```
Func<Person, bool, Person> compatible = Dynamic<Person>.Instance.Function<bool>.Explicit<Person>.CreateDelegate("Compatible");
Func<Person, Person, Person, Gender> breed = Dynamic<Person>.Instance.Function<Person>.Explicit<Person, Gender>.CreateDelegate("Breed");
Proc<Person> mutate = Dynamic<Person>.Instance.Procedure.Explicit.CreateDelegate("Mutate");
StaticFunc<Person, int, Person, Person> ageDifference = Dynamic<Person>.Static.Function<int>.Explicit<Person, Child>.CreateDelegate("AgeDifference");
```

Or, if you use the new C# 3.0 `var` syntax:
```
var compatible = Dynamic<Person>.Instance.Function<bool>.Explicit<Person>.CreateDelegate("Compatible");
var breed = Dynamic<Person>.Instance.Function<Person>.Explicit<Person, Gender>.CreateDelegate("Breed");
var mutate = Dynamic<Person>.Instance.Procedure.Explicit.CreateDelegate("Mutate");
var ageDifference = Dynamic<Person>.Static.Function<int>.Explicit<Person, Child>.CreateDelegate("AgeDifference");
```

I'm sorry for any trouble this might cause, but I think it is worth it. _If you have any comments, I'm listening..._

**UPDATE** (27-July-2007 15:32): I changed the `Build` to skip access-checks when creating the `DynamicMethod`. This allows you to bind against private methods of classes even when the bound-to class is not on the inheritance path. Also fixed a bug with instance methods that are functions requiring no parameters not always binding correctly. If you downloaded before 15:30 CDT, please download again.

---
### Comments:
#### Ok, I've been employing the library to build an OR...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-08-18T12:40:00.000-05:00">Aug 5, 2006</time>

Ok, I've been employing the library to build an ORMapper that gets its direction from attributes (before you say that I'm an idiot because there are a million out there, it's for a non-relational database, and it doesn't use a query language, so that's why I'm building yet another ORMapper -- probably better termed an O-Associative mapper)  
  
I'm using this to build strongly typed arguments for returning properties, and also to generate delegates that insert the bits of data one typed piece at a time. What I was wondering, is if there is a way to use this to parametrically define a type on the fly using IL generation. For example, each of my 'write it to the base' delegates takes a different type of value, and the base allows only a certain number of types. So the way I've currently been doing it is by implementing  
  
interface IDelegateFactory  
{  
Delegate String();  
Delegate Integer();  
Delegate Boolean();  
Delegate Float();  
Delegate ...  
.  
.  
.  
}  
  
I'm returning a Delegate for each like Dynamic<PutMethods>.Static.Explicit<IIdentifiable, string, \[thischanges\]>.CreateDelegate("Add");  
  
which of course produces delegates that only differ by one parameter type value.  
  
How, and I feel there most likely is a way here somewhere, can I avoid having to do that with generics?  
  
I realize that I can put in a generic there and specify the <T> in there, but then I'm still left doing a bunch of IF blocks somewhere else to figure out what goes in there... Am I missing something? Even if I have to generate some IL I'm up to the task, but do you have a way already with your library that I'm just seeing too flatly to identify?
<hr />
#### You can do partial generics and lazily instantiate...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-08-18T14:24:00.000-05:00">Aug 5, 2006</time>

You can do partial generics and lazily instantiate based on the attributes or reflected types. I'm working on a post right now.
<hr />
#### I've written this quick post, hope this is what yo...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-08-18T16:24:00.000-05:00">Aug 5, 2006</time>

I've written this quick post, hope this is what you are looking for: [Late delegate creation](http://musingmarc.blogspot.com/2006/08/how-to-do-late-dynamic-method-creation.html)
<hr />
#### Hi Marc,  
  
I downloaded your version of the D...
[Anonymous]( "noreply@blogger.com") - <time datetime="2007-08-18T10:43:00.000-05:00">Aug 6, 2007</time>

Hi Marc,  
  
I downloaded your version of the DynamicComparer yesterday after being tipped by [Johannes Hansen](http://www.codeproject.com/dotnet/dynamiclistsorting.asp?select=2187973&df=100&forumid=246789#xx2187973xx).  
After checking it out, I saw that the comparer wasn't able to sort objects on properties of associations.  
  
Tonight I made some modifications to the code to support this.  
  
Now, I can call the DynamicComparer like this:  
DynamicComparer<Person> comparer = new DynamicComparer<Person> ("Address.City, Address.Street");  
list.Sort(comparer.Compare);  
  
If you're interested I can send you the code.  
  
Greets,  
Nils Gruson
<hr />
#### I would love to integrate your changes into the co...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2007-08-18T12:22:00.000-05:00">Aug 6, 2007</time>

I would love to integrate your changes into the code... I'm hosting the code out at CodePlex now and there's already an outstanding request for this there.
<hr />
