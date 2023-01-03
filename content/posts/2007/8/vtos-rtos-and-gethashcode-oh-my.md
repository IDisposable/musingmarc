+++
title = "VTOs, RTOs and GetHashCode() -- oh, my!"
date = 2007-08-17T17:21:00.002-05:00
updated = 2008-03-09T03:30:16.470-05:00
draft = false
url = '/2007/08/vtos-rtos-and-gethashcode-oh-my.html'
tags = ["CLR",".Net","Equals","GetHashCode","FCL","C#","Utilities"]
+++

**UPDATED**:On 9 March, 2008, I fixed some issues with this posting due to comments from Hugh Brown, make **sure** you use the read the follow-up post [Sometimes you make a hash of things](http://musingmarc.blogspot.com/2008/03/sometimes-you-make-hash-of-things.html "Oops...").

### Introduction

Checking out a new blog today \[[Davy Brion's Blog](http://ralinx.wordpress.com/ "Good stuff from another .Net hacker")\] I stumbled across a very nice entry about [Implementing A Value Object](http://ralinx.wordpress.com/2007/07/08/implementing-a-value-object/ "Immutable objects mean that the values can't be changed without yielding a new object"). Go read that now if you don't know what a value object is, what immutable means or why it's good.

### Identity is who you are

What I want to talk about is GetHashCode() as used with value-type objects (e.g. `struct` in C#) but to do that, I really need to talk about the difference between reference-type objects (RTOs from here out) vs. value-type objects (VTOs from here out). Feel free to skip down if this is old hat to you.

What's important to realize is that if your are a reference-type object, your identity revolved around "where you are". This is expressed, in terms of .Net, by the fact that you have the same reference handle/memory address. The problem with this is that you might have an `Person` object that **currently** represents me and thus has the `FirstName` property == "Marc" and `LastName` property == "Brooks". If I give you a reference to that `Person` object and you change the `FirstName` property to "Charles", you're suddenly talking about my father. What's dangerous about this is that you have changed the underlying object to which I gave you a reference, thus my reference **also** now seems to be my father.

On the other hand, if I gave you a copy of the original `Person` object (perhaps via a `Clone()` operation), then you can change any property you wish and I will never know. This is good, if that's what you intend. Your personal copy of the object is **not** my copy of the object, they have different **physical** identities, even though they might initially share the same **logical** identity. To me, it's much like the difference between giving you a money order, or simply a copy of a money order. In the former case you are free to set the payee name to be whatever you want and cash/spend that money order.  In the latter, you can do whatever you want to your copy, but it doesn't affect mine.

VTOs automatically enforce the making of copies, you simply cannot change the original, no matter what... though you might change the property values on your copy, this does nothing to my original properties. What this means is that comparing value-type objects cannot meaningfully compare the **physical** identity (e.g. the reference handle/memory address) between to value-type objects because they will **always** be different.

So, how do you meaningfully compare VTOs? By their **logical** identity. In the example of a money order, the **logical** identity is actually the money order number, not the physical piece of paper. Some less-sophisticated verifications of the money order's validity might hinge on the appearance of the piece of paper, but a much better authoritative verification comes from calling in the money order number to the issuer and seeing if that number is still valid and for what amount.  Even modern sporting event venues operate similarly, checking not the physical appearance of of a ticket; rather they scan the barcode and match that against a database to insure the ticket is valid and hasn't been used yet.

Thus, a VTO's identity must be defined in terms of one or more of the property values. To check **logical** equality of two VTOs, you compare the equality of the _identifying_ properties. In the case of a money order, the money order number.

### Collections and hash-codes

When you drop an object in collection you expect to be able to later be able to retrieve that object (or, in the case of a VTO, a copy of the object) back out. The simplest way is enumeration, but that's not very quick. More commonly, you stick the object in some sort of dictionary keyed by some value. In the case of an `Array`, the key is simply the integer index of where you stuck the item, but for large numbers of potential objects you really need a _identifying_ property on the object itself. In the event ticket example, it's the barcode of the ticket. That key is used to store and retrieve the ticket information into a collection (perhaps a `Dictionary<TicketNumber, TicketStatus>` collection) is the barcode value. To make the storage and lookup quick, the collection internally stores the key values in "buckets" that are based in some way on the key's value. Each "bucket" contains a list of objects that have the same key-gives-bucket-number collection. Once you find the right bucket, you scan through all the objects in that bucket by doing an identity comparison. This means that:

1.  There must be some way to map the key values into bucket numbers.
2.  The mapping should not change if the identity of the object doesn't change.
3.  When two objects map to the same bucket number, they are disambiguated using the identity comparison.
4.  That once you've placed an reference-type object in a collection, changes to it's _identifying_ properties are going to break the comparison on retrieval.
5.  Changes to a value-type-object are impossible, since the collection is holding a copy.

The standard method used in the .Net Framework Class Library for identity comparison is the the `Equals()` method.  The standard method in the FCL to map object key values into buckets is the `GetHashCode()` method. In practical terms, this means that the `Equals()` method and the `GetHashCode()` method work together for any object you might want to place in a collection. They must agree one what properties of an object are _identifying_. When you design a value-type object, you really have to get it right because they have no meaningful **physical** identity.

### `Equals()` and `GetHashCode()` are free!

In the the .Net runtime, all objects automatically inherit an implementation of both the `Equals()` method and the `GetHashCode()` method. But as with many things in life, not all free things are really worth much. The default implementation of the `Equals()` method for reference-type objects is simply to compare the reference handle for equality. If we're pointing at the same object, we're talking about equal objects. The default implementation of the `GetHashCode()` method similarly bases its answer on the reference handle value. For value-type objects, the .Net runtime treats them _as-if_ they inherit from `ValueType`, so the the `Equals()` method on `ValueType` is what is called. This method compares field-by-field the individual elements of the object and returns if each is equal. Likewise, the the `GetHashCode()` method of `ValueType` is the default and it merely computes and combines the field-by-field hash-codes and combines them in an unspecified way to generate an overall object hash-code.

In summary, this means is that the default treatment of VTOs is to treat **all** fields as identifying. The default treatment of RTOs is to treat **none** of the fields as identifying. Rarely would this be the right thing to do, but that's what you get for the low-low-price of free.

If you have a **logical** identity for a VTO, or an RTO. then you need to supply your own implementation of `Equals()` and `GetHashCode()`. As detailed above, you need to make sure that they are coupled in their understanding of what fields and/or properties are the _identifying_ ones.

### Building your equality

Once you've identified what fields or properties to use when comparing to objects for **logical** identity, you need to implement an `Equals()` method and a `GetHashCode()` method the right way. For the Equals() method, there are [only a few rules](http://msdn2.microsoft.com/en-us/library/7h9bszxx(vs.71).aspx "Guidelines for Implementing Equals and the Equality Operator"):

1.  Thou shalt **not** throw an exception
2.  Thou shalt implement reasonable overrides (at least for your own type and taking `System.Object`)
3.  If you override `operator ==`, you must have a corresponding `Equals()` method.
4.  If you implement the `IComparable` interface, you should override `Equals()`

So, a classic implementation of a value-type object would be something like this (borrowed from Davy's post):

```
public override bool Equals(object obj)
{
   Address address = obj as Address;
   if (address != null)
   {
      return this.Equals(address);
   }
 
   return object.Equals(obj);
}
 
public bool Equals(Address address)
{
   if (address != null)
   {
      return this.Street.Equals(address.Street)
             && this.City.Equals(address.City)
             && this.Region.Equals(address.Region)
             && this.PostalCode.Equals(address.PostalCode)
             && this.Country.Equals(address.Country);
   }
 
   return false;
}

```

Note that it's perfectly fine for the Address object to have many other properties that are not considered _identifying_ and thus not included in the implementation of the `Equals()` method. That's really the whole point of implementing the `Equals()` method on an object. For VTOs you are trying to ignore some fields that the default `ValueType` implementation would have included. For RTOs, you are trying to establish some properties that give **logical** equivalence.

### Computing a useful hash-code

Once you've established a the body for `Equals()` method, you _absolutely must_ define the `GetHashCode()` method. This is where Davy's gets it 99% right.  He correctly states that every field/property value you call `Equals()` against should also be included in the `GetHashCode()` return value. Most people get that right, and Davy avoids the common mistake of adding the `GetHashCode()` sub-values together (which would skew the distribution pattern toward larger absolute values) and does an XOR of the sub-values. This is excellent, but we can get it a tiny bit better by following the pattern of many Microsoft provided classes and shifting the accumulated value before the XOR of the next sub-value. This leads to the low-order bits of the sub-value hash-codes being "distributed" into the final value instead of canceling each other out. Thus, my version of Davy's method is:

```
public override int GetHashCode()
{
      return (((((((this.Street.GetHashCode() << 5)
                   ^ this.City.GetHashCode()) << 5)
                 ^ this.Region.GetHashCode()) << 5)
               ^ this.PostalCode.GetHashCode()) << 5)
             ^ this.Country.GetHashCode();
}
```

Unfortunately, that's kind of ugly and error prone due to all the operator precedence issues. Can we make it better?

### Introducing CombineHashCode

So, a much better approach would be to have a little helper method set that knows how to do the combining according to this rule. For simplicity and ultimate flexibility, we'll have a version that takes an `params` array of objects and calls `GetHashCode()` on each of them in-turn. For better performance (to avoid boxing and unboxing) we'll add a version that takes a `params` array of precomputed hash codes (actually `System.Int32` values). Finally, for ultimate performance, we'll have a few overloads that take a specific number of objects or hash-code values to avoid the allocation of the `params` array.  You can add more as needed, but your really ought to rethink your class if you get more than five _identifying_ fields/properties.

```
public static partial class Utilities
{
    public static int CombineHashCodes(params int[] hashes)
    {
        int hash = 0;
 
        for (int index = 0; index < hashes.Length; index++)
        {
            hash <<= 5;
            hash ^= hashes[index];
        }
 
        return hash;
    }
 
    public static int CombineHashCodes(params object[] objects)
    {
        int hash = 0;
 
        for (int index = 0; index < objects.Length; index++)
        {
            int entryHash = 0x61E04917; // slurped from .Net runtime internals...
            object entry = objects[index];

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
 
            hash <<= 5;
            hash ^= entryHash;
        }
 
        return hash;
    }
 
    public static int CombineHashCodes(int hash1, int hash2)
    {
        return (hash1 << 5)
               ^ hash2;
    }
 
    public static int CombineHashCodes(int hash1, int hash2, int hash3)
    {
        return (((hash1 << 5)
                 ^ hash2) << 5)
               ^ hash3;
    }
 
    public static int CombineHashCodes(int hash1, int hash2, int hash3, int hash4)
    {
        return (((((hash1 << 5)
                   ^ hash2) << 5)
                 ^ hash3) << 5)
               ^ hash4;
    }
 
    public static int CombineHashCodes(int hash1, int hash2, int hash3, int hash4, int hash5)
    {
        return (((((((hash1 << 5)
                     ^ hash2) << 5)
                   ^ hash3) << 5)
                 ^ hash4) << 5)
               ^ hash5;
    }
 
    public static int CombineHashCodes(object object1, object object2)
    {
        return CombineHashCodes(object1.GetHashCode()
            , object2.GetHashCode());
    }
 
    public static int CombineHashCodes(object object1, object object2, object object3)
    {
        return CombineHashCodes(object1.GetHashCode()
            , object2.GetHashCode()
            , object3.GetHashCode());
    }
 
    public static int CombineHashCodes(object object1, object object2, object object3, object object4)
    {
        return CombineHashCodes(object1.GetHashCode()
            , object2.GetHashCode()
            , object3.GetHashCode()
            , object4.GetHashCode());
    }
}
```

This leaves us with the final version of Davy's `GetHashCode()` method looking like this:

```
public override int GetHashCode()
{
      return CombineHashCodes(this.Street, this.City, this.Region, this.PostalCode, this.Country);
}
```

That's pretty clean and easy to understand, right?

**UPDATED**:On 9 March, 2008, I fixed some issues with this posting due to comments from Hugh Brown, make **sure** you use the read the follow-up post [Sometimes you make a hash of things](http://musingmarc.blogspot.com/2008/03/sometimes-you-make-hash-of-things.html "Oops...").

Technorati Tags: [C#](http://technorati.com/tags/C#), [.Net](http://technorati.com/tags/.Net), [CLR](http://technorati.com/tags/CLR), [GetHashCode](http://technorati.com/tags/GetHashCode), [Equals](http://technorati.com/tags/Equals)

---
### Comments:
#### Hi Mark,  
I've posted a follow-up to your post o...
[Anonymous]( "noreply@blogger.com") - <time datetime="2007-08-19T01:50:00.000-05:00">Aug 0, 2007</time>

Hi Mark,  
I've posted a follow-up to your post on my blog, feel free to check it out.  
  
http://blogs.microsoft.co.il/blogs/sasha/archive/2007/08/19/ValueType-virtual-methods-and-why-avoid-them.aspx  
  
My main point is regarding the performance costs of the ValueType-provided methods.  
  
Cheers,  
Sasha
<hr />
#### I think that if you shift and XOR, you end up thro...
[Anonymous]( "noreply@blogger.com") - <time datetime="2008-03-08T11:07:00.000-06:00">Mar 6, 2008</time>

I think that if you shift and XOR, you end up throwing away bits. That is, their effect is removed after a few iterations.  
  
Try this python code:  
  
def hash(i, c) :  
 return ((int(i) << 5) ^ c) & 0xFFFFFFFF  
  
def hashR(low, high) :  
 hashC = 0  
 for i in range(low, high) :  
  hashC = hash(hashC, i)  
 return hashC  
  
**print "%08x" % hashR(24, 32)**  
75be77df  
**print "%08x" % hashR(25, 32)**  
75be77df  
**print "%08x" % hashR(26, 32)**  
35be77df  
  
All of the hashes below 25 are identical -- the first 24 pieces of data provide no value.  
  
I'd recommend multiplying by 37 and adding instead of shifting and XOR:  
  
def hash(i, c) :  
 return ((int(i) \* 37) + c) & 0xFFFFFFFF  
  
for i in range(1,32) :  
 print "%08x" % hashR(i, 32)  
  
All of the results are distinct.  
  
Also, I am amazed at your 2-, 3-, 4-, n-parameter hash calculations when you also have the calculation using _params_. Using that exclusively seems like such a better idea.
<hr />
#### "The default implementation of the GetHashCode() m...
[Anonymous]( "noreply@blogger.com") - <time datetime="2008-03-08T14:55:00.000-06:00">Mar 6, 2008</time>

"The default implementation of the GetHashCode() method similarly bases its answer on the reference handle value."  
  
I don't think this is true. I've heard that the default GetHashCode() implementation is based solely on the hash of the first member of the structure. I haven't disassembled the code, but check this:  
  
public struct X  
{  
 public string a, b, c;  
 public X (string a, string b, string c)  
 {  
  this.a = a;  
  this.b = b;  
  this.c = c;  
 }  
 public override string ToString()  
 {  
  return string.Format("{0}.{1}.{2}", this.a, this.b, this.c);  
 }  
}  
  
public class MyClass  
{  
 public static void Main()  
 {  
  for (int i = 0; i < 10; i++)  
  {  
   //X x = new X ("asdasd", i.ToString(), (2\*i).ToString());  
   X x = new X (i.ToString(), "asdasd", (2\*i).ToString());  
   Console.WriteLine("{0} {1}", x.ToString(), x.GetHashCode());  
  }  
 }  
}  
  
If you run it with this line:  
 X x = new X ("asdasd", i.ToString(), (2\*i).ToString());  
then all the structures hash to the same value. If you run it with this line:  
 X x = new X (i.ToString(), "asdasd", (2\*i).ToString());  
Then all of the structures hash to a different value.  
  
So I think that you should override the GetHashCode() method of structures because the default implementation is so wrong for the most common case. On the other hand, if the first field were a GUID, then the default implementation would be perfectly okay for use in collection classes.
<hr />
#### Your test-case reveals a weakness that I will addr...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-03-08T23:45:00.000-06:00">Mar 6, 2008</time>

Your test-case reveals a weakness that I will address in a follow-up post, but it is in no way a normal test. HashCodes should not be sequential values, nor should only the lowest bits be non-zero. Thus your test is not typical.  
  
The weakness is there, and in 2.0 runtime, the combiner uses: ((combinedHash << 5) + combinedHash) ^ hash;  
I've got a post coming about the change and updating the code.  
  
As to your "amazed" response to the overloads (which I take to be a negative reaction), they are there to eliminate the need to allocate and initialize the int\[\] for the params version when a small number of items exist. This is done in the BCL version as well, both in HashCodeCombiner and in the String.Format.
<hr />
#### A struct is a ValueType, not a reference type. The...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2008-03-09T00:25:00.000-06:00">Mar 0, 2008</time>

A struct is a ValueType, not a reference type. The quote you are using was about reference types.  
   
class ThreeClass  
{  
public int a; public int b; public int c;  
public ThreeClass(bool bogus)  
{ c = 1; b = 2; a = 3; }  
}  
  
struct ThreeStruct  
{  
public int a; public int b; public int c;  
public ThreeStruct(bool bogus)  
{ c = 1; b = 2; a = 3; }  
}  
  
ThreeClass aClass = new ThreeClass(true);  
ThreeClass bClass = new ThreeClass(true);  
Console.WriteLine(String.Format("aClass: {0:x}, bClass: {1:x}", aClass.GetHashCode(), bClass.GetHashCode()));  
aClass.a = 4;  
Console.WriteLine(String.Format("aClass (a=4): {0:x}", aClass.GetHashCode()));  
aClass.c = 5;  
Console.WriteLine(String.Format("aClass (c=5): {0:x}", aClass.GetHashCode()));  
  
ThreeClass cClass = bClass;  
Console.WriteLine(String.Format("bClass: {0:x}, cClass: {1:x}", bClass.GetHashCode(), cClass.GetHashCode()));  
  
ThreeStruct aStruct = new ThreeStruct(true);  
ThreeStruct bStruct = new ThreeStruct(true);  
Console.WriteLine(String.Format("aStruct: {0:x}, bStruct: {1:x}", aStruct.GetHashCode(), bStruct.GetHashCode()));  
aStruct.a = 4;  
Console.WriteLine(String.Format("aStruct (a=4): {0:x}", aStruct.GetHashCode()));  
aStruct.c = 5;  
Console.WriteLine(String.Format("aStruct (c=5): {0:x}", aStruct.GetHashCode()));  
  
ThreeStruct cStruct = bStruct;  
Console.WriteLine(String.Format("bStruct: {0:x}, cStruct: {1:x}", bStruct.GetHashCode(), cStruct.GetHashCode()));  
  
The output is:  
aClass: 2b89eaa, bClass: 273e403  
aClass (a=4): 2b89eaa  
aClass (c=5): 2b89eaa  
bClass: 273e403, cClass: 273e403  
aStruct: f391c0, bStruct: f391c0  
aStruct (a=4): f391c7  
aStruct (c=5): f391c3  
bStruct: f391c0, cStruct: f391c0  
   
This clearly shows that a reference type's default GetHashCode() is dependant on the handle of the object because assigning the class instance to another class gives the same hash code. For the structs (aka ValueTypes), the value is directly related to the values as changing either the first or last element changes the hash code.
<hr />
