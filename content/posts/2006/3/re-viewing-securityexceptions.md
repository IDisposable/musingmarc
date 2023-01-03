+++
title = "RE: Viewing SecurityExceptions"
date = 2006-03-23T14:39:00.000-06:00
updated = 2006-04-16T01:17:41.926-05:00
draft = false
url = '/2006/03/re-viewing-securityexceptions.html'
tags = []
+++

[Shawn Farkas](http://blogs.msdn.com/shawnfa) notes [here](http://blogs.msdn.com/shawnfa/archive/2006/03/23/557062.aspx) that you can't display some of the information available in `SecurityException` when not running in fully trusted code (like an ASP.Net host). Then along comes [Dominick Baier](http://www.leastprivilege.com/) with the [fix](http://www.leastprivilege.com/PermaLink.aspx?guid=3d536597-877a-4576-ab04-f3d67314b291). I smell a setup.

In any case, the trick is to put a fully-trusted assembly (in the GAC, of course!) to handle the extraction of this information, and pass the unmangled `SecurityException` in. If the trusted assembly's method asserts the needed `ControlEvidence` and `ControlPolicy` then you can get the extra goodies in`ToString()`.
Cool!

```csharp
using System;
using System.Security.Permissions;
using System.Security;

[assembly: AllowPartiallyTrustedCallers]
namespace LeastPrivilege
{
  [
   SecurityPermission(SecurityAction.Assert, 
   ControlEvidence=true, ControlPolicy=true)
  ]

  public class SecurityExceptionViewer
  {
    public static string[] ViewException(SecurityException ex)
    {
      return new string[] {
        ex.ToString(),
        ex.Demanded.ToString(),
        ex.GrantedSet.ToString() 
      };
    }
  }
}
```

_\[Via [www.leastprivilege.com](http://www.leastprivilege.com/PermaLink.aspx?guid=3d536597-877a-4576-ab04-f3d67314b291)\]_

---

### Comments

#### it does not necessarily need to be in the GAC - yo…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-04-16T01:17:00.000-05:00">Apr 0, 2006</time>

it does not necessarily need to be in the GAC - you could also use ~/bin and write a policy file that grants the assembly the necessary permission..
  
GAC is just the easiest...

---
