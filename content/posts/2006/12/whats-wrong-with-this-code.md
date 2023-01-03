+++
title = "What's wrong with this code..."
date = 2006-12-11T15:33:00.000-06:00
updated = 2006-12-11T15:46:37.003-06:00
draft = false
url = '/2006/12/whats-wrong-with-this-code.html'
tags = ["Microsoft","thread-safe","bug"]
+++

```
private void EnsurePropsOwned()
{
    if (!this.propsOwned)
    {
        this.propsOwned = true;
        if (this.properties != null)
        {
            PropertyDescriptor[] descriptorArray1 = new PropertyDescriptor[this.Count];
            Array.Copy(this.properties, 0, descriptorArray1, 0, this.Count);
            this.properties = descriptorArray1;
        }
    }
   
    if (this.needSort)
    {
        this.needSort = false;
        this.InternalSort(this.namedSort);
    }
}
```

1.  If you're going to take ownership of something, wouldn't it be nice to grab a copy BEFORE you claim to own it?
2.  If you are going to sort something, don't you think you should actually sort it before clearing needs-sort flag?

The scary part is that this is code from `System.ComponentModel.PropertyDescriptorCollection` which is something deep inside the framework and completely impossible to debug for threading issues.

I would just report this as a bug to Microsoft, but since they have a history of just ignoring bugs (or worse yet, just closing them without reason), I'm not going to waste my time. Not that I could today anyway, since [Microsoft Connect](http://connect.microsoft.com "The blackhole where bugs disappear... look, it's Jimmy Hoffa") doesn't (irony!)

---

### Comments

#### Perhaps I just don't get the real issue here, but …

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-12-13T14:54:00.000-06:00">Dec 3, 2006</time>

Perhaps I just don't get the real issue here, but I don't see how re-ordering the statements would make any difference. Surely it wouldn't be any 'more' threadsafe? (I know there is no such thing as 'more' threadsafe, eiterh it is or it isn't). Also the docs say quite explicitly that the class is not supposed to be threadsafe anyways, i.e. that the caller is responsible for any and all synchronisation needed.  
ben
---

#### Hah, sure don't you remember back in Computer …


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-01-17T11:39:00.000-06:00">Jan 3, 2007</time>

Hah, sure don't you remember back in Computer Science when we studied the "Commutative Property of Code ?" It's just like in math where a + b = b + a.  
  
ldarg\_1  
call System.Console.Out.WriteLine  
  
therefore equals:  
  
call System.Conosle.Out.WriteLine ldarg\_1  
  
</sarcasm>  
  
I'm not really taking sides on this issue, but it seemed like a fun thing to say. While I agree with marc I'd order them his way, ben does indeed have a point. The class comes with a disclaimer.
---
