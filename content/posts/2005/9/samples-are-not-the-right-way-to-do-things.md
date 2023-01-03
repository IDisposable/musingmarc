+++
title = "Samples are not the -right way- to do things."
date = 2005-09-30T13:41:00.000-05:00
updated = 2006-01-19T09:25:32.156-06:00
draft = false
url = '/2005/09/samples-are-not-right-way-to-do-things.html'
tags = []
+++

Today, once again, I've seen the corruption of code caused by reading code examples and mindlessly parroting the style and not the substance. Every sample you see these days has code that using `String.Format` for simple concatenation. That's **not** what it's for, folks! `String.Format` is to allow formatting (not just conversion to `String`) and parameter rearrangement. Code like this is wasteful and silly:

```csharp
private string CreateSelect()
{
   return String.Format("SELECT TOP {0} {1} FROM {2}", this.rowLimit, this.selectFields, this.fromTables);
}
```

This will needlessly have to scan the format string looking for the replacement parameters `{x}` and substituting the parameters positionally. This code should be:

```csharp
private string CreateSelect()
{
   return "SELECT TOP " + this.rowLimit + " " + this.selectFields + " FROM " + this.fromTables;
}
```

Which C# will actually turn into this behind the scenes:

```csharp
private string CreateSelect()
{
   String[] temp = new String[6] { "SELECT TOP ", this.rowLimit.ToString(), " ", this.selectFields, " FROM ", this.fromTables };
   return String.Concat(temp);
}
```

Which is tons faster. This sort of thing is important to get right in the lowest-level of your code, in things like a DAL or UI framework library.

_Not that I'm whining..._

---

### Comments

#### I know what you mean, but I do think the first ver…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-06T07:00:00.000-05:00">Oct 4, 2005</time>

I know what you mean, but I do think the first version is very readable. My eyes can see what string is being build and pick out the {0} placeholders easily.

---

#### I see the first form most often thanks to the need…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-06T11:25:00.000-05:00">Oct 4, 2005</time>

I see the first form most often thanks to the need to localize the strings. So people drop the {0} form into a string table and do String.Format.
  
Building a string in English sentence structure is asking for a bug later on...
  
jps

---

#### Using it for localized sentences is exact what Str…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-10-25T14:41:00.000-05:00">Oct 2, 2005</time>

Using it for localized sentences is exact what `String.Format` is for. The example was building SQL statements, which are not sentences in any form, nor are they subject to localization (unless you're doing the funky parallel tables for localize lookups trick).

---

#### It depends. :) In this particular case this code…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-01-19T09:25:00.000-06:00">Jan 4, 2006</time>

It depends. :) In this particular case this code is only executed once, during initialization, and then the results are cached -- so I valued readability. But I'm still not sure how big the difference really is anyhow. The numbers I've seen from multiple independent sources is that string concatenation is faster when there are 4 or fewer strings, but once you get 5 or more strings that is not likely to be true. If your option is StringBuilder then it seems to just about always be faster once you get to 5 or more strings, while its not so clear-cut with string.Format, but even then its also not clearly better to use concatenation either. If you really need the performance, and in this case I just don't think it was necessary due to the one-time initialization, then I would switch to a StringBuilder approach at this point. Anyhow, my point isn't to say you're wrong and I'm right -- I just found the post and wanted to add another line of thought to put it into perspective.
  
Paul Wilson

---
