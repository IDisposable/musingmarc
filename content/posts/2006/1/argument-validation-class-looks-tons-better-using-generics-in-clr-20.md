+++
title = "Argument validation class looks tons better using generics in CLR 2.0"
date = 2006-01-17T12:08:00.000-06:00
updated = 2007-11-19T00:20:59.706-06:00
draft = false
url = '/2006/01/argument-validation-class-looks-tons.html'
tags = [".Net","generic","C#","validation"]
+++

I've had this class around for a long time, finally decided to update it to use generic types instead of tons of overloads. It's tons nicer this way. Contact me if you want a version with XML documentation (which just looks ugly in the blog).

```csharp
using System;
using System.ComponentModel;
using System.Diagnostics;

namespace IDisposable.Utilities
{
 public static class ArgumentValidation
 {
  #region Helper
  private static string PadWord(string text)
  {
   if (String.IsNullOrEmpty(text))
    return string.Empty;
   else
    return text + " ";
  }
  #endregion

  public static void ArgumentDifferentTypeCheck(object value, string name, Type type)
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentNullCheck(type, "type");

   if (value.GetType() != type)
   {
    throw new ArgumentException(
     string.Format("Argument {0}must be of type {1}.",
      ArgumentValidation.PadWord(name), type.Name), name);
   }
  }

  public static void ArgumentIncompatibleTypeCheck(object value, string name, Type type)
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentNullCheck(type, "type");

   if (!type.IsAssignableFrom(value.GetType()))
   {
    throw new ArgumentException(
     string.Format("Argument {0}must be compatible with type {1}.",
      ArgumentValidation.PadWord(name), type.Name), name);
   }
  }

  public static void ArgumentNullCheck(object value, string name)
  {
   if (null == value)
   {
    throw new ArgumentNullException(name,
     string.Format("Argument {0}cannot be null.",
      ArgumentValidation.PadWord(name)));
   }
  }

  public static void ArrayArgumentDifferentRank(Array value, string name, int rank)
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");

   if (value.Rank != rank)
   {
    throw new RankException(string.Format("Argument {0}must be "
     + "an array with {1} dimension.", ArgumentValidation.PadWord(name),
     rank));
   }
  }

  public static void ArgumentInvalidEnumValueCheck(object value, string name, Type enumType)
  {
   Debug.Assert(value.GetType().IsPrimitive || (value.GetType() == typeof(string))
    , "Expected a primitive type or a String!");

   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentNullCheck(enumType, "enumType");

   if (enumType.GetCustomAttributes(typeof(FlagsAttribute), true).Length > 0)
   {
    throw new ArgumentException(string.Format(
     "Cannot validate against the type {0} because it has the custom attribute FlagsAttribute."
      , enumType), "enumType");
   }

   if (!Enum.IsDefined(enumType, value))
   {
    if (value.GetType() == typeof(string))
    {
     // Let the value argument be int.MaxValue if a string was passed in.
     throw new InvalidEnumArgumentException(name, int.MaxValue, enumType);
    }
    else
    {
     int valueInt32 = 0;
     TypeConverter typeConverter = TypeDescriptor.GetConverter(
             Enum.GetUnderlyingType(enumType));

     try
     {
      valueInt32 = (int)typeConverter.ConvertTo(value, typeof(int));
     }
     catch (OverflowException)
     {
      // Let the value argument be 0 if it overflows Int32.
     }

     throw new InvalidEnumArgumentException(name, valueInt32, enumType);
    }
   }
  }

  public static void ArgumentOutOfRangeCheck(T value, string name, T minValue, T maxValue) where T : IComparable
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentIncompatibleTypeCheck(minValue, "minValue", typeof(T));
   ArgumentValidation.ArgumentIncompatibleTypeCheck(maxValue, "maxValue", typeof(T));

   if (value.CompareTo(minValue) < 0 || 0 < value.CompareTo(maxValue))
   {
    throw new ArgumentOutOfRangeException(name, value
     , string.Format("Argument {0}must be greater than or equal to {1} and less than or equal to {2}."
     , ArgumentValidation.PadWord(name), minValue, maxValue));
   }
  }

  public static void ArgumentOutOfRangeExclusiveCheck(T value, string name, T lowerBound, T upperBound) where T : IComparable
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentDifferentTypeCheck(lowerBound, "lowerBound", typeof(T));
   ArgumentValidation.ArgumentDifferentTypeCheck(upperBound, "upperBound", typeof(T));

   if (lowerBound.CompareTo(upperBound) == 0)
   {
    throw new ArgumentException("Arguments lowerBound and upperBound cannot be equal.");
   }

   if (value.CompareTo(lowerBound) <= 0 || 0 <= value.CompareTo(upperBound))
   {
    throw new ArgumentOutOfRangeException(name, value,
     string.Format("Argument {0}must be greater than {1} and less than {2}."
      , ArgumentValidation.PadWord(name), lowerBound, upperBound));
   }
  }

  public static void ArgumentOutOfRangeIncludeMaxCheck(T value, string name, T lowerBound, T maxValue) where T : IComparable
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentIncompatibleTypeCheck(lowerBound, "lowerBound", typeof(T));
   ArgumentValidation.ArgumentIncompatibleTypeCheck(maxValue, "maxValue", typeof(T));

   if (value.CompareTo(lowerBound) <= 0 || 0 < value.CompareTo(maxValue))
   {
    throw new ArgumentOutOfRangeException(name, value,
     string.Format("Argument {0}must be greater than {1} and less than or equal to {2}."
     , ArgumentValidation.PadWord(name), lowerBound, maxValue));
   }
  }

  public static void ArgumentOutOfRangeIncludeMinCheck(T value, string name, T minValue, T upperBound) where T : IComparable
  {
   ArgumentValidation.ArgumentNullCheck(value, "value");
   ArgumentValidation.ArgumentIncompatibleTypeCheck(minValue, "minValue", typeof(T));
   ArgumentValidation.ArgumentIncompatibleTypeCheck(upperBound, "upperBound", typeof(T));

   if (value.CompareTo(minValue) < 0 || 0 <= value.CompareTo(upperBound))
   {
    throw new ArgumentOutOfRangeException(name, value,
     string.Format("Argument {0}must be greater than or equal to {1} and less than {2}."
     , ArgumentValidation.PadWord(name), minValue, upperBound));
   }
  }
 }
}

```

---

### Comments

#### Can you throw out a .zip link with the .cs and .xm…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-02-16T12:28:00.000-06:00">Feb 4, 2006</time>

Can you throw out a .zip link with the .cs and .xml doc? This thing is a goldmine!

---

#### Done, …

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-02-16T19:28:00.000-06:00">Feb 4, 2006</time>

Done, see [here.](/2006/02/some-utilities-for-net-development.html)

---

#### Updated to fix E…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-03-01T13:41:00.000-06:00">Mar 3, 2006</time>

Updated to fix Enum [bug](/2006/03/systemenumcompareto-needs-_114123536776382301.html)

---
