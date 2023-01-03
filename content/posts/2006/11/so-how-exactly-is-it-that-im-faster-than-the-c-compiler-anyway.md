+++
title = "So how exactly is it that I'm faster than the C# compiler anyway?"
date = 2006-11-07T17:44:00.000-06:00
updated = 2006-12-11T15:52:21.402-06:00
draft = false
url = '/2006/11/so-how-exactly-is-it-that-im-faster.html'
tags = [".Net","LCG","CodePlex","DynamicMethod","IL","Dynamic","C#","lightweight code generation","Reflection"]
+++

I've had this one in the hopper for a while, but recently people have been asking, so...

I was curious about why my [Dynamic Reflection Library](http://www.codeplex.com/Dynamic "Dynamic Reflection Library home on CodePlex") code was benchmarking as 6% faster than native code (e.g. that produced bu the C# compiler) for reference types, so I did a little disassembly and grokking. Note, I **did** remember to make sure that the _Supress JIT optimization on module load_ option was not checked and this is a release build with compile-time optimizations enabled.

Here's the native-call code with for a single compile-time call `T child = firstEntry.Breed(secondEntry, (Gender)neighbor);`:

```
0000037a mov eax,dword ptr [ebp-38h] 0000037d mov dword ptr [ebp-70h],eax 00000380 push 0 00000382 push 0 00000384 push 0 00000386 mov ecx,esi 00000388 mov edx,0CA000057h 0000038d call 7919286A 00000392 push edi 00000393 mov edx,dword ptr [ebp-70h] 00000396 mov ecx,dword ptr [ebp-30h] 00000399 nop 0000039a nop 0000039b nop 0000039c call dword ptr [eax]
```

Now for the explicit generics call for the matching Explicit call `T child = firstEntry.Breed(secondEntry, (Gender)neighbor);`:

```
0000046a push esi 0000046b push ebx 0000046c mov edx,edi 0000046e mov ecx,dword ptr [ebp-60h] 00000471 mov eax,dword ptr [ecx+0Ch] 00000474 mov ecx,dword ptr [ecx+4] 00000477 call eax
```

That's **tons** simpler, how come? The simple answer is that since the native call code is dealing with an interface, the code has to go through an indirect call. Meanwhile, the generated delgate code (what my Dynamic class creates) gets JITted out of existence and replaced with a direct call to the actual method on the destination class (Person.Breed in this case). Similar savings all through the test method body leave even more opportunity to keep the locals in registers and the lack of indirect calls saves a bunch of memory/cache-line access per-loop pass.

For my benchmark program (itself an interesting read) the `BenchCallDynamicExplict<Person>(Adjuster<Person>, Random rnd)` method loop code (not counting setup and teardown) is 0x1F3 bytes in JIT form, the `BenchCallCompileTime<Person>(Adjuster<Person>, Random rnd)` loop code is 0x2E5 bytes, which is 252 bytes longer.

Looking further at this code, if I constrain the `T` to be a class (e.g. a reference-type not a value-type/`struct`) then I get exactly the same code, so there's no point in that, I modified the benchmark code to remove the "hack" that required a `new()` constraint and passed an progenitor instance to eliminate the disadvantage for the compile-time (which needs to go through an interface). Even that doesn't change the code significantly.

The moral of this exercise is that sometimes the generated code, by giving much clearer information to the JIT process about exactly what methods are being called will allow it to generate much faster code. I'm not saying that you should do this kind of optimization everywhere, or indeed almost anywhere, but I **am** saying that when you write low level code to simplify your life and make things faster, you sometimes get much more than you can expect as a benefit.

Take that C# compiler, **I win!** Oh, and you guys on the JIT team... **you rock!**

Technorati tags: [DynamicMethod](http://technorati.com/tags/DynamicMethod), [LCG](http://technorati.com/tags/LCG), [Lightweight Code Generation](http://technorati.com/tags/Lightweight%20Code%20Generation), [.Net 2.0](http://technorati.com/tags/.Net%202.0), [Generic](http://technorati.com/tags/Generic), [CodePlex](http://technorati.com/tags/CodePlex)

---

### Comments

#### Really cool! So then... when is your dynamic langu…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-11-07T20:53:00.000-06:00">Nov 2, 2006</time>

Really cool! So then... when is your dynamic language coming out? Joel P. might had better take a look at your numbers :)
---

#### Nice job on this (I think) - Although I have to sa…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-12-02T19:56:00.000-06:00">Dec 6, 2006</time>

Nice job on this (I think) - Although I have to say that I don't quite understand exactly what your code is doing or how it works. I will definitely peruse your source code to help me understand LCG as I am very interested in this topic.  
  
Can you provide a brief example of how your code could be used to replace Reflection based OR/M mapping that relies heavily on Metadata/tags?
---
