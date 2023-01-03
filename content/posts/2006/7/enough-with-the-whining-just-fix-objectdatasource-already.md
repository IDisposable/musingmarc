+++
title = "Enough with the whining, just fix ObjectDataSource already..."
date = 2006-07-20T20:15:00.000-05:00
updated = 2007-11-19T00:01:44.512-06:00
draft = false
url = '/2006/07/enough-with-whining-just-fix.html'
tags = [".Net","LCG","DynamicMethod","Dynamic","C#","DataSource","Asp.Net","lightweight code generation","ObjectDataSource"]
+++

### What's the deal?

If you have ever tried to use `ObjectDataSource` with any O/R Mapper or just business entities that have "persistent state", you no-doubt know the issues I've whined about. It really boils down, at the simplest level to the absence of "acquire" semantics for the instance of the `DataObjectType` specified. In other words, you can just get the "old" instance of the business entity object from your persistence-store / cache / web-service / etc. and then let `GridView` / `DetailsView` / `FormView` update **just** the properties it has exposed on the user-interface. This is impossible because ObjectDataSourceView simply calls `Activator.CreateInstance` to create an object ex-nilo. If there was a method you could override on `ObjectDataSourceView` to acquire the object before all the properties are fiddled-with, then all would be good, but there isn't. I'm going to spend the next few posts talking about how to accomplish that goal. If I'm lucky, this will shame Microsoft into fixing this for the next framework release.

### The first hurdle is ~stupidity~ design issues.

First, we have to deal with the fact that some things are simply not as they should be in the `ObjectDataSource` class itself. Firstly, there are a few key fields, properties and types that are `private`, notably the `_view` field, the `Cache` property, and the `Enabled` property of the `SqlDataSourceCache` that the `Cache` stores (not to mention the fact that the `SqlDataSourceCache` is also private). Secondly, there's a bit of extreme silliness in the implementation of the ,code>GetView call-chain. Like all good `DataSourceControl`s, ODS has an `override` for the `GetView(string viewName)` virtual method. This method's body is pretty simple, just verifying you are being a good citizen and passing only the view names that ODS knows about, namely `null`, `String.Empty` or `"DefaultView"`, then it just calls the `GetView()` method to create the actual `ObjectDataSourceView` instance.

```
protected override DataSourceView GetView(string viewName)
{
      if ((viewName == null) || ((viewName.Length != 0) && !string.Equals(viewName, "DefaultView", StringComparison.OrdinalIgnoreCase)))
      {
            throw new ArgumentException(SR.GetString("DataSource_InvalidViewName", new object[] { this.ID, "DefaultView" }), "viewName");
      }
      return this.GetView();
}
```

The `GetView()` method just does a lazy instantiation against the `private ObjectDataSourceView _view` field.
```
private ObjectDataSourceView GetView()
{
      if (this._view == null)
      {
            this._view = new ObjectDataSourceView(this, "DefaultView", this.Context);
            if (base.IsTrackingViewState)
            {
                  ((IStateManager) this._view).TrackViewState();
            }
      }
      return this._view;
}
```

Here's where things really start to go wrong... the `GetView()` method is private and non-virtual, so you can't simply override it to return a subclass of ODSV that does what we want. Ah, but that's okay, we can override the `GetView(string viewName)` virtual method to do what we need, right? Nope, because nowhere in the code of ODS does that method get called. All the standard action methods (like `Select()`, `Delete()`, `Insert()` and `Update()`) directly call the non-replacable `GetView()` private method and then directly delegate the action. **Ick.** Okay, no problem, we just override the action methods, right? Nope, again... they are also non-virtual. Also, there are tons of properties on ODS that do the same direct call to `GetView()` and delegate the call down, this is especially bad for things like `EnablePaging` and similar properties, since they get called while we're still constructing the ODS in the cool new declarative coding model of ASP.Net 2.0.

What this basically means is that we're going to have to code up a derived class replacement for ODS and then poke in our replacement for the `_view` in the private bits. Without further adieu, let me introduce **`AcquiringObjectDataSource`** (I've eliminated all the XML documentation, comments and attributes for this post):

```
public class AcquiringObjectDataSource : ObjectDataSource
{
    const BindingFlags NeedlesslyPrivate = BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance;
 
    static MethodInfo s_InvalidateCacheEntry;
    static MethodInfo s_GetCache;
    static MethodInfo s_GetCacheEnabled;
    static FieldInfo s_View;
 
    static AcquiringObjectDataSource()
    {
        s_InvalidateCacheEntry = typeof(ObjectDataSource).GetMethod("InvalidateCacheEntry", NeedlesslyPrivate, null, new Type[0], null);
        PropertyInfo cache = typeof(ObjectDataSource).GetProperty("Cache", NeedlesslyPrivate);
        s_GetCache = cache.GetGetMethod(true);
        PropertyInfo cacheEnabled = cache.PropertyType.GetProperty("Enabled", NeedlesslyPrivate);
        s_GetCacheEnabled = cacheEnabled.GetGetMethod(true);
        s_View = typeof(ObjectDataSource).GetField("_view", NeedlesslyPrivate);
    }
 
    public AcquiringObjectDataSource()
    {
        // force creation!
        this.GetView();
    }
 
    public AcquiringObjectDataSource(string typeName, string selectMethod)
        : base(typeName, selectMethod)
    {
        // force creation!
        this.GetView();
    }
 
    protected virtual ObjectDataSourceView GetView()
    {
        ObjectDataSourceView view = (ObjectDataSourceView)s_View.GetValue(this);
 
        if (view == null)
        {
            view = new AcquiringObjectDataSourceView(this, "DefaultView", this.Context);
            s_View.SetValue(this, view);
 
            if (base.IsTrackingViewState)
            {
                ((IStateManager)view).TrackViewState();
            }
        }
 
        return view;
    }
 
    protected override DataSourceView GetView(string viewName)
    {
        if (viewName == null || (viewName.Length != 0 && !string.Equals(viewName, "DefaultView", StringComparison.OrdinalIgnoreCase)))
        {
            throw new ArgumentException(ExposedSR.GetString(ExposedSR.InvalidViewName, new object[] { this.ID, "DefaultView" }), "viewName");
        }
 
        return this.GetView();
    }
 
    protected void InvalidateCache()
    {
        object cache = s_GetCache.Invoke(this, null);
        object cacheEnabled = s_GetCacheEnabled.Invoke(cache, null);
 
        if ((bool)cacheEnabled)
        {
            s_InvalidateCacheEntry.Invoke(this, null);
        }
    }
}
```

Some notes are in order to explain that code:

1.  We force creation of the view during our constructor to insure our `GetView()` gets a chance to create an instance of our `AcquiringObjectDataSourceView` before something in ODS causes one to be created. This isn't as lazy as possible, but it's safe.
2.  We have `static` constructor to do the one-time `Reflection` to get all the private bits we want to play with. In a later post, I'll show you how to use the _DynamicCall_ stuff in my Utlities library to handle this is a much type-safer and faster way.
3.  We override `GetView(string viewName)` merely to insure that if someone _does_ call it, we will do the right thing and call our `GetView()` and not the one in ODS
4.  That last method is there to cure a little class-envy on the part of ODSV, which makes a few calls against the `internal SqlDataSourceCache Cache` property. They typically look like this:
    ```
    if (this._owner.Cache.Enabled)
    {
       this._owner.InvalidateCacheEntry();
    }
    ```
    
5.  The [`ExposedSR`](http://musingmarc.blogspot.com/2006/07/recycling-messages-or-how-not-to-be.html) class is just something letting me reuse the localized exception messages embedded in the System.Web.dll assembly, we'll see it in ~part 3~ [part 2](http://musingmarc.blogspot.com/2006/07/recycling-messages-or-how-not-to-be.html).

### Next time, right here...

~Next time~ In part 3, I'll talk about the issues in `ObjectDataSourceView` and present the replacement class [`AcquiringObjectDataSourceView`](http://musingmarc.blogspot.com/2006/07/i-can-see-clearly-now-pain-is-gone-or.html)

---
### Comments:
#### I'm not sure if it would solve all the issues you ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2007-02-15T12:56:00.000-06:00">Feb 4, 2007</time>

I'm not sure if it would solve all the issues you need to address, but a possible alternative approach to what you've done might be to hook into the ObjectCreating event of the ObjectDataSource. That should let you assign the data access component that you want as the ObjectInstance field of the ObjectDataSourceEventArgs. There's a decent explanation of doing that in the book "Pro ASP.NET 2.0 in C# 2005" by MacDonald and Szpuszta.
<hr />
#### This is exactly what I'm looking for!!! I need...
[Tim K.]( "noreply@blogger.com") - <time datetime="2010-10-15T17:50:29.000-05:00">Oct 5, 2010</time>

This is exactly what I'm looking for!!! I need to execute an Update method in a formview that has more than one parameter. This little ditty you wrote about here helped me get a start to hook into the ObjectDataSource deep enough to add those parameters. Thank you!
<hr />
