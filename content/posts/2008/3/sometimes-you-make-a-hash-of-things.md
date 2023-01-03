+++
title = "Sometimes you make a hash of things."
date = 2008-03-09T03:23:00.004-05:00
updated = 2008-03-09T04:16:42.269-05:00
draft = false
url = '/2008/03/sometimes-you-make-hash-of-things.html'
tags = ["Microsoft",".Net","GetHashCode","C#","Utilities"]
+++

So, hash has been on my mind lately.  No, not [that kind](http://en.wikipedia.org/wiki/Hashish "hashish") of hash, or [that kind](http://en.wikipedia.org/wiki/Hash_%28food%29 "mmmm, now that I'm thinking of it, sounds tasty") either.  First, there was last week, when I installed Internet Explorer 8 beta 1.  I was reading the release notes and was amazed to find that # (you know, octothorpe, pound sign) was not considered part of the URL by this version.  Thus you can't link directly to a `name`d element on a page. Eeew!

Then today, [Hugh Brown](http://www.iwebthereforeiam.com/ "I Web, Therefore I Am") dropped a comment on my [diatribe post about value-types, reference-types, Equals and GetHashCode](http://musingmarc.blogspot.com/2007/08/vtos-rtos-and-gethashcode-oh-my.html). The post has been live for many months now, and has quite a bit of Google juice. Until now, nobody has ever quibbled with the stuff I wrote, but Hugh had some interesting observations.

### First the little stuff

In a minor part of his comment, he was surprised by the many overloads of GetHashCode that I suggest, wondering why I didn't just always expect callers to use the params int\[\] version. Quite simply, this is because by providing several overloads for a small number of arguments (5 in my example), I avoid paying the cost of allocating the array of integers and copying the values for each call to the `CombineHashCodes`. While this may seem like a trivial savings, remember that `GetHashCode` is called many times when dealing with `HashTable` collections and thus it is worth it to provide expedited code paths for the more common usages. Additional savings inside the `CombineHashCodes` method are garnered by avoiding the loop setup/iteration overhead. Finally, in optimized builds, these simpler method calls will be inlined by the compiler and/or JIT, where methods having loops in the body are never inlined (in CLR releases thus far). It is worth noting that the .Net runtime implementation does the same thing for `System.Web.Util.HashCodeCombiner` and `System.String.Format`.

### To the meat of the comment

The main body of his comment was that my code actually didn't return useful values. That concerned me a lot. Given his use of Python and inlined implementation, I had to write my own test jig. Unfortunately it confirmed his complaint. On the one hand, the values he was using to test were not normal values you would expect from `GetHashCode`. Normally `GetHashCode` values are well-distributed across the entire 32-bit range of an `Int32`. He was using sequential, smallish, numbers which was skewing the result oddly. That said, the values SHOULD have been different for very-similar inputs. I delved a little into the code I originally wrote and found that what's on the web page does NOT match what is **now** in use in the BCL's internal code to combine hash codes (which is where I got the idea of left-shifting by 5 bits before XORing). I think that my code was originally based on the 1.1 BCL but I'm not really sure.

In the .Net 2.0 version, there's a class called `System.Web.Util.HashCodeCombiner` that actually reflects essentially the same technique as my code, with one huge and very significant difference. Where I simply left-shift the running hash code by 5 bits and then XOR in the next value, they are doing the left-shift **and also adding in the running hash**, then doing the XOR.

### Why so shifty, anyway?

You might be wondering why do the left shift in the first place. The simple answer is that by doing a left-shift by some number of bits, we preserve the low order bits of the running hash somewhat. This prevents the incoming value from XORing away all the significance of the bits thus far, and also insures that low-byte-only intermediate hash codes don't simply cancel each other out. By shifting left 5 digits, we're simply multiplying by 32 (and thus preserving the lowest 5 digits). Then the original running hash value is added in on more time, making the effective multiplier 33. This isn't far off from Hugh's suggestion of multiplying by 37, while being significantly faster in the binary world of computers. Once the shift and add (e.g. multiplication by 33) is completed, the XOR of the new values results in much better distribution of the final value.

I've updated my code in the [Utilities library](http://idisposable.googlepages.com/downloads "My downloads"), and I'm going back to the original post to point to this post and the new code. So, I owe you one, Hugh...and **maybe Microsoft does too** because while I was reviewing their code in the newly released BCL source code, I found a very unexpected implementation. This is the snippet in question:

```
    internal static int CombineHashCodes(int h1, int h2) {
        return ((h1 << 5) + h1) ^ h2; 
    }
 
    internal static int CombineHashCodes(int h1, int h2, int h3) { 
        return CombineHashCodes(CombineHashCodes(h1, h2), h3);
    } 

    internal static int CombineHashCodes(int h1, int h2, int h3, int h4) {
        return CombineHashCodes(CombineHashCodes(h1, h2), CombineHashCodes(h3, h4));
    } 

    internal static int CombineHashCodes(int h1, int h2, int h3, int h4, int h5) { 
        return CombineHashCodes(CombineHashCodes(h1, h2, h3, h4), h5); 
    }
```

Did you see the oddity? That implementation taking 4 values does its work by calling the two-value one three times. Once to combine the first pair (h1 and h2) of arguments, once to combine the second pair (h3 and h4), then finally to combine the two intermediate values. That's a bit different than doing what the 3-value and 5-value overloads use. I personally think it should have called the 2-value against output of the 3-value to combine the 4th value (h4). That would be more like what the 3-value and 5-value overload do. In other words, the method should be:
```
    internal static int CombineHashCodes(int h1, int h2, int h3, int h4) {
        return CombineHashCodes(CombineHashCodes(h1, h2, h3), h4);
    }
```

Perhaps they don't care that the values are inconsistent, especially since they don't provide a combiner that takes a `params int[]` overload, but imagine if I had blindly copied that code and you got two different values from this:

```
   Console.WriteLine("Testing gotcha:");   Console.WriteLine(String.Format("1,2: {0:x}", Utilities.CombineHashCodes(1, 2)));   Console.WriteLine(String.Format("1,2,3: {0:x}", Utilities.CombineHashCodes(1, 2, 3)));   Console.WriteLine(String.Format("1,2,3,4: {0:x}", Utilities.CombineHashCodes(1, 2, 3, 4)));   Console.WriteLine(String.Format("1,2,3,4,5: {0:x}", Utilities.CombineHashCodes(1, 2, 3, 4, 5)));
   Console.WriteLine(String.Format("[1,2]: {0:x}", Utilities.CombineHashCodes(new int[] { 1, 2 })));   Console.WriteLine(String.Format("[1,2,3]: {0:x}", Utilities.CombineHashCodes(new int[] { 1, 2, 3 })));   Console.WriteLine(String.Format("[1,2,3,4]: {0:x}", Utilities.CombineHashCodes(new int[] { 1, 2, 3, 4 })));   Console.WriteLine(String.Format("[1,2,3,4,5]: {0:x}", Utilities.CombineHashCodes(new int[] { 1, 2, 3, 4, 5 })));
```

### Where we are at now

Here is the revised version of the `CombineHashCodes` methods from my [Utilities library](http://idisposable.googlepages.com/downloads "My downloads")

```
    public static partial class Utilities
    {
        public static int CombineHashCodes(params int[] hashes)
        {
            int hash = 0;

            for (int index = 0; index < hashes.Length; index++)
            {
                hash = (hash << 5) + hash;
                hash ^= hashes[index];
            }

            return hash;
        }

        private static int GetEntryHash(object entry)
        {
            int entryHash = 0x61E04917; // slurped from .Net runtime internals...

            if (entry != null)
            {
                object[] subObjects = entry as object[];

                if (subObjects != null)
                {
                    entryHash = Utilities.CombineHashCodes(subObjects);
                }
                else
                {
                    entryHash = entry.GetHashCode();
                }
            }

            return entryHash;
        }

        public static int CombineHashCodes(params object[] objects)
        {
            int hash = 0;

            for (int index = 0; index < objects.Length; index++)
            {
                hash = (hash << 5) + hash;
                hash ^= GetEntryHash(objects[index]);
            }

            return hash;
        }

        public static int CombineHashCodes(int hash1, int hash2)
        {
            return ((hash1 << 5) + hash1)
                   ^ hash2;
        }

        public static int CombineHashCodes(int hash1, int hash2, int hash3)
        {
            int hash = CombineHashCodes(hash1, hash2);
            return ((hash << 5) + hash)
                   ^ hash3;
        }

        public static int CombineHashCodes(int hash1, int hash2, int hash3, int hash4)
        {
            int hash = CombineHashCodes(hash1, hash2, hash3);
            return ((hash << 5) + hash)
                   ^ hash4;
        }

        public static int CombineHashCodes(int hash1, int hash2, int hash3, int hash4, int hash5)
        {
            int hash = CombineHashCodes(hash1, hash2, hash3, hash4);
            return ((hash << 5) + hash)
                   ^ hash5;
        }

        public static int CombineHashCodes(object obj1, object obj2)
        {
            return CombineHashCodes(obj1.GetHashCode()
                , obj2.GetHashCode());
        }

        public static int CombineHashCodes(object obj1, object obj2, object obj3)
        {
            return CombineHashCodes(obj1.GetHashCode()
                , obj2.GetHashCode()
                , obj3.GetHashCode());
        }

        public static int CombineHashCodes(object obj1, object obj2, object obj3, object obj4)
        {
            return CombineHashCodes(obj1.GetHashCode()
                , obj2.GetHashCode()
                , obj3.GetHashCode()
                , obj4.GetHashCode());
        }

        public static int CombineHashCodes(object obj1, object obj2, object obj3, object obj4, object obj5)
        {
            return CombineHashCodes(obj1.GetHashCode()
                , obj2.GetHashCode()
                , obj3.GetHashCode()
                , obj4.GetHashCode()
                , obj5.GetHashCode());
        }
     }

```

---
### Comments:
#### Hi. Very interesting post. Thanks.  
  
I have t...
[Blair Conrad](https://www.blogger.com/profile/17093978353640865561 "noreply@blogger.com") - <time datetime="2008-03-24T09:18:00.000-05:00">Mar 1, 2008</time>

Hi. Very interesting post. Thanks.  
  
I have two questions, though.  
First, why is  
"return ((h1 << 5) + h1) ^ h2;" safe? For sufficiently large h1 or h2, there's a danger of arithmetic overflow on the "+". Am I missing something here?  
  
Second, I noticed the versions of CombineHashCode that you define that take specific object parameters (as opposed to a params array) don't care about null-checking their arguments, but the more generic version does. Is there a particular reason for this?  
  
Thanks,  
Blair
<hr />
#### A1) Indeed you do lose bits for high-value hash co...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-03-24T13:58:00.000-05:00">Mar 1, 2008</time>

A1) Indeed you do lose bits for high-value hash code elements, but that is expected behavior. When building a hash code, you are looking to capture the _essence_ of the values as a single 32-bit number. This means information will be lost.  
  
A2) You are right about the null-checks being missing. For the int versions, that's not a problem, but for the overloads taking an object I am being a bit sloppy. This is mostly because most of my code never has null references... instead I point things at null-object stand-in objects. When deployed in a public library, it would be best to validate them. That said, it's not a big stretch to feel that walking on method up on a call-stack isn't too much to deal with when you attempt a call to GetHashCode() against a null parameter.
<hr />
#### Thanks for the response, @IDisposable - it clears ...
[Blair Conrad](https://www.blogger.com/profile/17093978353640865561 "noreply@blogger.com") - <time datetime="2008-03-24T14:51:00.000-05:00">Mar 1, 2008</time>

Thanks for the response, @IDisposable - it clears up some things, but not others. Let me try again.  
  
A1: I wasn't perfectly clear here. It's not the lost bits (from the << or the + or the ^) that I'm worried about - it's the "System.OverflowException: Arithmetic operation resulted in an overflow" that I fear will cause problems when certain arguments (like hash1=67108863, for example) are passed into CombineHash, or when the running hash becomes equal to one of those values before being combined with another value.  
  
A2: I thought that maybe you were being careful about nulls, but thought I'd ask anyhow. And since I work primarily in healthcare, I get a little more worried about having to walk a method up a callstack to identify the source of my null reference after the fact. Still, you've cleared things up - thanks for the info.
<hr />
#### A1.1) The System.OverflowException would only be t...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-03-24T15:13:00.000-05:00">Mar 1, 2008</time>

A1.1) The System.OverflowException would only be thrown if that block of code was surrounded with a checked block... by default the arithmatic operations on integers is NOT checked. See: [the checked keywork in C#](http://msdn2.microsoft.com/en-us/library/74b4xzyw(vs.71).aspx "checked (C#)")  
  
A2.2) I understand... an it's an FxCop violation to not check reference parameters for public methods. I do that in production code with my ArgumentValidation suite of methods in the utility class.
<hr />
#### A1.1: Ah. I had overlooked that. And I whipped out...
[Blair Conrad](https://www.blogger.com/profile/17093978353640865561 "noreply@blogger.com") - <time datetime="2008-03-24T15:53:00.000-05:00">Mar 1, 2008</time>

A1.1: Ah. I had overlooked that. And I whipped out my sample app using SharpDevelop as I don't have Visual Studio at home, and after reading the information you thoughtfully provided and digging a bit more in SharpDevelop, I see that for some reason it ships with overflow checking on by default - hence the exception that I saw.  
  
A2.2: Regrettably my team doesn't run checkers like FxCop or anything like that on our code as part of the automated build. Developers are free to do that on their own, and there are code inspections, but it's easy for lax checking to sneak in. Perhaps it's time for me to start proselytizing again...  
  
Thanks again for spending time with me on this. I've learned stuff and enjoyed the exchange.
<hr />
#### Wow! I thought you were mistaken about integer ari...
[Jonathan Gilbert]( "noreply@blogger.com") - <time datetime="2009-11-09T08:54:00.000-06:00">Nov 1, 2009</time>

Wow! I thought you were mistaken about integer arithmetic and checked mode. For years, I've been under the impression that it was enabled by default -- and it turns out I wasn't \_completely\_ wrong:  
  
\[C:\\test\]copy con test.cs  
con => C:\\test\\test.cs  
using System;  
  
class Test  
{  
static void Main()  
{  
Console.WriteLine(int.MaxValue + 1);  
}  
}  
^Z  
1 file copied  
  
\[C:\\test\]csc test.cs  
Microsoft (R) Visual C# 2008 Compiler version 3.5.30729.1  
for Microsoft (R) .NET Framework version 3.5  
Copyright (C) Microsoft Corporation. All rights reserved.  
  
test.cs(7,23): error CS0220: The operation overflows at compile time in checked  
mode  
  
\[C:\\test\]  
  
However, if it doesn't detect the overflow at compile time, and you don't explicitly ask it to, then it doesn't detect it at all:  
  
\[C:\\test\]copy con test.cs  
con => C:\\test\\test.cs  
using System;  
  
class Test  
{  
static void TestMethod(int x)  
{  
Console.WriteLine(x + 1);  
}  
  
static void Main()  
{  
TestMethod(int.MaxValue);  
}  
}  
^Z  
1 file copied  
  
\[C:\\test\]csc test.cs  
Microsoft (R) Visual C# 2008 Compiler version 3.5.30729.1  
for Microsoft (R) .NET Framework version 3.5  
Copyright (C) Microsoft Corporation. All rights reserved.  
  
  
\[C:\\test\]test  
\-2147483648  
  
\[C:\\test\]  
  
This is a surprise to me. I guess you learn something new every day :-)  
  
Jonathan Gilbert
<hr />
#### Wow, finally an easy to understand, well researche...
[Shaun]( "noreply@blogger.com") - <time datetime="2010-02-19T04:24:46.000-06:00">Feb 5, 2010</time>

Wow, finally an easy to understand, well researched, peer reviewed and verified method for decent hash code generation! Thank you very kindly :)
<hr />
