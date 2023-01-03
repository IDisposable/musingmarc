+++
title = "Speeding access to properties by caching accessors."
date = 2005-10-31T11:10:00.000-06:00
updated = 2005-10-31T11:42:47.513-06:00
draft = false
url = '/2005/10/speeding-access-to-properties-by.html'
tags = []
+++

When writing generic frameworks, such as [O/R Mappers](http://weblogs.asp.net/yreynhout/archive/2003/10/07/30798.aspx) or [UI Mapping](http://weblogs.asp.net/pwilson/archive/2004/07/10/179588.aspx) Frameworks, you inevitably run into the need to access the members (fields or properties) of another class to bind data to the database call parameters or user interface controls. This process is essentially trivial in .Net and implementation exampled are everywhere. All of them rely on some use of the Reflection classes in .Net. I've heard too many complaints about the speed of systems leveled against the use of [Reflection](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpref/html/frlrfsystemreflection.asp), so I spent a little effort getting things zippy on the two frameworks I use on a daily basis. I'm a huge proponent of not doing premature optimization, but when coding framework-level classes, it **is** good idea to practice good design principles and make the things that stand out during timing runs as quick as possible (while still being maintainable). As a sidebar, the frameworks I am using for O/R Mapping and generic UI are [Paul Wilson](http://www.wilsondotnet.com)'s excellent [ORMapper](http://www.ormapper.net) and [UIMapper](http://www.uimapper.net). ORMapper is very mature, and does most of what I need in an O/R Mapper. UI Mapper is still a 1.00 release, and I've got tons of changes for Paul once he gets time to play with it again, but he's been very open to changes in the past. I can stress how good a deal this software is, everything that Paul's got on the [site](http://www.wilsondotnet.com/Code/) in good C# code for [$50](http://www.wilsondotnet.com/Login/?ReturnUrl=%2fCode%2fSubscription.aspx)! Anywho, the short of this is that I've got a MemberAccessor cache class that makes sure the reflection is done once, and all the [MemberInfo](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpref/html/frlrfsystemreflectionmemberinfoclasstopic.asp) and/or [FieldInfo](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpref/html/frlrfsystemreflectionfieldinfoclasstopic.asp) is cached so everything runs very quickly each subsequent use. This stuff works in service classes, WinForms and ASP.Net so don't worry about application or dependancies on things like [HttpContext.Cache](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpref/html/frlrfsystemwebhttpcontextclasscachetopic.asp). Still to do is adding a [Lightweight Code Generation](http://blogs.msdn.com/joelpob/archive/2004/03/31/105282.aspx) to emit the calling stubs, which is a facinating new feature of .Net 2.0 based on the [DynamicMethod](http://msdn2.microsoft.com/en-us/library/80h6baz2(en-US,VS.80).aspx) class. Much of this is based on the ideas give in Joel Pobar's blog [entry](http://blogs.msdn.com/joelpob/archive/2004/04/01/105862.aspx) and his [article on MSDN](http://msdn.microsoft.com/msdnmag/issues/05/07/Reflection/default.aspx) Here's the class, hit me up with any questions:
```
using System;
using System.Collections.Generic;
using System.Reflection;

namespace Phydeaux.Mapping.Utility
{
    public class MemberAccessor
    {
        internal RuntimeMethodHandle Get;
        internal RuntimeMethodHandle Set;
        internal RuntimeFieldHandle Field;

        internal bool HasGet
        {
            get { return (this.Get.Value != IntPtr.Zero); }
        }

        internal bool HasSet
        {
            get { return (this.Set.Value != IntPtr.Zero); }
        }

        internal bool HasField
        {
            get { return (this.Field.Value != IntPtr.Zero); }
        }

        internal bool Settable
        {
            get { return this.HasSet || this.HasField; }
        }

        internal bool Gettable
        {
            get { return this.HasGet || this.HasField; }
        }

        internal bool AnyDefined
        {
            get { return this.HasGet || this.HasSet || this.HasField; }
        }

        internal bool FullyDefined
        {
            get { return this.HasGet && this.HasSet && this.HasField; }
        }
    }

    public class MemberCacheKey : IEquatable
    {
        internal RuntimeTypeHandle TypeHandle;
        internal string Member;

        internal MemberCacheKey(Type type, string member)
        {
            this.TypeHandle = type.TypeHandle;
            this.Member = member;
        }

        public override bool Equals(object other)
        {
            // covers both null and same reference check...
            if (System.Object.ReferenceEquals(this, other))
                return true;

            return this.Equals(other as MemberCacheKey);
        }

        public override int GetHashCode()
        {
            return (TypeHandle.Value.GetHashCode() << 5) ^ Member.GetHashCode();
        }

        #region IEquatable Members
        public bool Equals(MemberCacheKey other)
        {
            // covers both null and same reference check...
            if (System.Object.ReferenceEquals(this, other))
                return true;

            if (other == null)
                return false;

            return TypeHandle.Equals(other.TypeHandle)
                && Member.Equals(other.Member);
        }
        #endregion

        public class Comparer :  IEqualityComparer
        {
            #region IEqualityComparer Members
            public bool Equals(MemberCacheKey x, MemberCacheKey y)
            {
                // covers both null and same reference check...
                if (System.Object.ReferenceEquals(x, y))
                    return true;

                if (x == null)
                    return false;

                return x.Equals(y);
            }

            public int GetHashCode(MemberCacheKey obj)
            {
                if (obj == null)
                    return 0;

                return obj.GetHashCode();
            }
            #endregion
        }
    }

    public class AccessorCache : Dictionary
    {
        const MemberTypes WhatMembers = MemberTypes.Field | MemberTypes.Property;
        const BindingFlags WhatBindings = BindingFlags.SetProperty | BindingFlags.SetField 
                                            | BindingFlags.GetProperty | BindingFlags.GetField 
                                            | BindingFlags.Public | BindingFlags.NonPublic
                                            | BindingFlags.Instance | BindingFlags.DeclaredOnly;

        public AccessorCache()
            : base(new MemberCacheKey.Comparer())
        {
        }

        public MemberAccessor GetAccessor(Type entityType, string member)
        {
            MemberCacheKey key = new MemberCacheKey(entityType, member);
            MemberAccessor accessor;

            if (!this.TryGetValue(key, out accessor))
            {
                if (BuildAccessor(entityType, member, out accessor))
                {
                    this.Add(key, accessor);
                }
                else
                {
                    throw ArgumentValidation.Decorate(
                        new UIMapperException("cannot build accessor")
                        , MethodBase.GetCurrentMethod(), entityType, member);
                }
            }

            return accessor;
        }

        private bool BuildAccessor(Type entityType, string member, out MemberAccessor accessor)
        {
            accessor = new MemberAccessor();
            return BuildAccessorRecursive(entityType, member, accessor);
        }

        private bool BuildAccessorRecursive(Type entityType, string member, MemberAccessor accessor)
        {
/// TODO build a LCG delegate like http://msdn.microsoft.com/msdnmag/issues/05/07/Reflection/default.aspx
            if (entityType == null || entityType == typeof(Object))
                return accessor.AnyDefined;

            MemberInfo[] members = entityType.GetMember(member, WhatMembers, WhatBindings);

            // look for a property
            foreach (MemberInfo someMember in members)
            {
                if (someMember.MemberType == MemberTypes.Property)
                {
                    PropertyInfo property = (PropertyInfo) someMember;

                    if (property.CanRead && ! accessor.HasGet)
                    {
                        accessor.Get = property.GetGetMethod(true).MethodHandle;
                    }

                    if (property.CanWrite && !accessor.HasSet)
                    {
                        accessor.Set = property.GetSetMethod(true).MethodHandle;
                    }
                }

                if (someMember.MemberType == MemberTypes.Field && !accessor.HasField)
                {
                    FieldInfo field = ((FieldInfo) someMember);
                    accessor.Field = field.FieldHandle;
                }
            }

            return accessor.FullyDefined
                || BuildAccessorRecursive(entityType.BaseType, member, accessor);
        }
    }        
}
```

---
### Comments:
#### That's not a musing... that's a mega-thought.. in ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2005-11-08T13:41:00.000-06:00">Nov 2, 2005</time>

That's not a musing... that's a mega-thought.. in fact, I can't even think of a major M word that would mean mega-thought...  
  
I feel so worthless... I'll go back to pushing papers on my desk now. Sorry to interrupt.
<hr />
#### Thanks Jodster, credit the incredible article from...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-11-08T14:38:00.000-06:00">Nov 2, 2005</time>

Thanks Jodster, credit the incredible article from Joel for the bulk of this idea.
<hr />
