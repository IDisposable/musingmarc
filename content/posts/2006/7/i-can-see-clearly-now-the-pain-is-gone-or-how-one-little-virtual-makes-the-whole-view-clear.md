+++
title = "I can see clearly now, the pain is gone (or how one little virtual makes the whole view clear)"
date = 2006-07-24T16:57:00.000-05:00
updated = 2007-11-19T00:01:00.888-06:00
draft = false
url = '/2006/07/i-can-see-clearly-now-pain-is-gone-or.html'
tags = [".Net","LCG","DynamicMethod","Dynamic","C#","DataSource","Asp.Net","lightweight code generation","ObjectDataSource"]
+++

In [part one](http://musingmarc.blogspot.com/2006/07/enough-with-whining-just-fix.html "Building AcquiringObjectDataSource") of this little series, I introduced the first player in our little play, `AcquringObjectDataSource`. Quite literally, the only reason that class was necessary was to insure that we create a slightly smarter `ObjectDataSourceView`. This part of the series is all about adding the "acquire" behavior. What I'm after is a simple way of getting the choice as to whether the `DataObjectType` instance is created ex-nilo via `Activator.CreateInstance` (the standard behavior of `ObjectDataSourceView`, or created by a factory class, or recalled from some cache or datastore. In my case, I want to reload the object instances from the database via [Wilson's O/R Mapper](http://www.ormapper.net "Wilson's Object/Relational Mapper - the best value in .Net"), after first having checked the ASP.Net `Session` to see if we've got partial changes to carry through.

### The View does the work

The first thing that becomes painfully obvious when building on ODS is that the ODSV object does the heavy lifting. Almost everything in ODS that isn't related to the creation of the view, or the maintenance of the cache is merely a call-through proxy to the ODSV. The main operations are obviously the CRUD operations. These operations are carried out but the `ExecuteInsert`, `ExecuteSelect`, `ExecuteUpdate` and `ExecuteDelete`, respectively. The read method (`ExecuteSelect`) already acts as expected, so we'll ignore that method. The others mutating methods are handed an `IDictionary` of keys, old values and new values (each method gets some of these, as appropriate to the use). The dictionaries are used to fill in values for a new object (in case of insert), an old object (in case of delete), or both (in case of update). This is, as stated, where my pain begins. I want to insure that we hit whatever the original data source is if needed, not create the objects out of the ether. In each of the mutating methods it builds up a list of parameter values by merging in any hard-coded parameters values, the keys `IDictionary` and any values `IDictionary`(s). This is done by a `private` method of ODSV called `BuildDataObject`. This simple method looks like this in [Reflector](http://www.lutzroeder.com/ "The single-most important tool in the .Net arsenal") (I've highlighted the line that grieves my soul):

```
private object BuildDataObject(Type dataObjectType, IDictionary inputParameters)
{
      object obj1 = Activator.CreateInstance(dataObjectType);
      PropertyDescriptorCollection collection1 = TypeDescriptor.GetProperties(obj1);
      foreach (DictionaryEntry entry1 in inputParameters)
      {
            string text1 = (entry1.Key == null) ? string.Empty : entry1.Key.ToString();
            PropertyDescriptor descriptor1 = collection1.Find(text1, true);
            if (descriptor1 == null)
            {
                  throw new InvalidOperationException(SR.GetString("ObjectDataSourceView_DataObjectPropertyNotFound", new object[] { text1, this._owner.ID }));
            }
            if (descriptor1.IsReadOnly)
            {
                  throw new InvalidOperationException(SR.GetString("ObjectDataSourceView_DataObjectPropertyReadOnly", new object[] { text1, this._owner.ID }));
            }
            object obj2 = ObjectDataSourceView.BuildObjectValue(entry1.Value, descriptor1.PropertyType, text1);
            descriptor1.SetValue(obj1, obj2);
      }
      return obj1;
}
```

If it weren't for that simple little `Activator.CreateInstance`, my life would have been easy... but, alas, I must toil against the oppressors. I could easily replace that method, but for several things. First, it is not virtual, so my version would not get called. Second, it is private, so I can't very well call down to reuse the behavior. Third, it uses `ObjectDataSourceView.BuildObjectValue`, which is _of course_ `private`.

### Virtual is where the action is

What I need to do is make sure that I have an integration-point where I can _embrace and extend_ the Microsoft implementation. I want to make sure that some event or virtual method is called to create / rehydrate / lookup the actual data object. How about something blindingly obvious like:

```
public virtual object AcquireDataObject(Type dataObjectType, IDictionary inputParameters)
{
    return Activator.CreateInstance(dataObjectType);
}
```

This has the advantage of being hookable, while still letting the original naive implementation to be used. _\[Note: If Microsoft had simply done this **and ONLY this** to the original ODSV class, I wouldn't have needed **any of this**, not that I'm bitter\]_. We pass the `IDictionary` of input parameters to the creation method to give it a place to extract the key values, if needed.

Of course, it can't be that easy for me. Rather, I get bit by the fact that since BuildDataObject is non-`virtual`, `private` and pretty much incestuously coupled with everything else in ODSV, I'm going to have to do much more work. Specifically, I need to insure that my version of `BuildDataObject` gets called, so that I can replace that first line with a call to the new virtual `AcquireDataObject` method. This means that I need to override the mutating methods `ExecuteInsert`, `ExecuteUpdate` and `ExecuteDelete`. Since I only care about the case where a `DataObjectType` has been specified, I can down-call if that value isn't set, and have a vastly simplified body for each of these that only handles the DOT case. Of course, to simply reimplement those three methods I have to call a boat-load of other methods in ODSV, which are (you guessed it) `private`. Namely I need access to the `Owner` field, the `GetType` method, the `TryGetDataObjectType` method, the `MergeDictionaries` method, the `GetResolvedMethodData` method, the previously mentioned `BuildObjectValue` method, the `InvokeMethod` method, and ickly-enough the private `ObjectDataSourceMethod` nested class; whose `Parameters` field we also need to fiddle with. Lastly, we need to be able to extract the `AffectedRows` of the private `ObjectDataSourceResult` nested class. That's a lot of tendrils stuck in to ODSV, but I for the want of a single `virtual`, that is the path I must tread.

### Where we are now

So, without any further delay, may I introduce to you the fair and fleeting `AcquiringObjectDataSourceView`, whose merest existence I groan at:

```
public class AcquiringObjectDataSourceView : ObjectDataSourceView
{
    const BindingFlags NeedlesslyPrivate = BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.DeclaredOnly;
    const BindingFlags NeedlesslyPrivateStatic = BindingFlags.NonPublic | BindingFlags.Static | BindingFlags.DeclaredOnly;
 
    static readonly FieldInfo s_Owner;
    static readonly MethodInfo s_GetType;
    static readonly MethodInfo s_TryGetDataObjectType;
    static readonly MethodInfo s_MergeDictionaries;
    static readonly MethodInfo s_GetResolvedMethodData;
    static readonly MethodInfo s_InvokeMethod;
    static readonly MethodInfo s_BuildObjectValue;
    static readonly FieldInfo s_AffectedRows;
    static readonly FieldInfo s_Parameters;
 
    static AcquiringObjectDataSourceView()
    {
        s_Owner = typeof(ObjectDataSourceView).GetField("_owner", NeedlesslyPrivate);
 
        Type[] getType = new Type[] { typeof(string) };
        s_GetType = typeof(ObjectDataSourceView).GetMethod("GetType", NeedlesslyPrivate, null, getType, null);
 
        Type[] tryGetDataObjectType = new Type[] { };
        s_TryGetDataObjectType = typeof(ObjectDataSourceView).GetMethod("TryGetDataObjectType", NeedlesslyPrivate, null, tryGetDataObjectType, null);
 
        Type[] mergeDictionaries = new Type[] { typeof(ParameterCollection), typeof(IDictionary), typeof(IDictionary) };
        s_MergeDictionaries = typeof(ObjectDataSourceView).GetMethod("MergeDictionaries", NeedlesslyPrivateStatic, null, mergeDictionaries, null);
 
        Type[] getResolvedMethodData = new Type[] { typeof(Type), typeof(string), typeof(Type), typeof(object), typeof(object), typeof(DataSourceOperation) };
        s_GetResolvedMethodData = typeof(ObjectDataSourceView).GetMethod("GetResolvedMethodData", NeedlesslyPrivate, null, getResolvedMethodData, null);
 
        Type[] buildObjectValue = new Type[] { typeof(object), typeof(Type), typeof(string)};
        s_BuildObjectValue = typeof(ObjectDataSourceView).GetMethod("BuildObjectValue", NeedlesslyPrivateStatic, null, buildObjectValue, null);
 
        Type objectDataSourceMethod = typeof(ObjectDataSourceView).GetNestedType("ObjectDataSourceMethod", BindingFlags.NonPublic);
        s_Parameters = objectDataSourceMethod.GetField("Parameters", NeedlesslyPrivate);
 
        Type[] invokeMethod = new Type[] { objectDataSourceMethod };
        s_InvokeMethod = typeof(ObjectDataSourceView).GetMethod("InvokeMethod", NeedlesslyPrivate, null, invokeMethod, null);
 
        Type objectDataSourceResult = typeof(ObjectDataSourceView).GetNestedType("ObjectDataSourceResult", BindingFlags.NonPublic);
        s_AffectedRows = objectDataSourceResult.GetField("AffectedRows", NeedlesslyPrivate);
    }
 
    public AcquiringObjectDataSourceView(ObjectDataSource owner, string name, HttpContext context)
        : base(owner, name, context)
    {
    }
 
    protected override int ExecuteDelete(IDictionary keys, IDictionary oldValues)
    {
        if (!this.CanDelete)
        {
            throw new NotSupportedException(ExposedSR.GetString(ExposedSR.DeleteNotSupported, this.Owner.ID));
        }
 
        // we only change the behavior of sources that provide a DataObjectTypeName
        if (String.IsNullOrEmpty(this.DataObjectTypeName))
            return base.ExecuteDelete(keys, oldValues);
 
        Type sourceType = this.GetType(this.TypeName);
        Type dataObjectType = this.TryGetDataObjectType();
        IDictionary deleteParameters = new OrderedDictionary(StringComparer.OrdinalIgnoreCase);
 
        if (this.ConflictDetection == ConflictOptions.CompareAllValues)
        {
            if (oldValues == null || oldValues.Count == 0)
            {
                throw new InvalidOperationException(ExposedSR.GetString(ExposedSR.Pessimistic, ExposedSR.GetString(ExposedSR.Delete), this.Owner.ID, "oldValues"));
            }
 
            MergeDictionaries(this.DeleteParameters, oldValues, deleteParameters);
        }
 
        MergeDictionaries(this.DeleteParameters, keys, deleteParameters);
        object dataObject = this.BuildDataObject(dataObjectType, deleteParameters);
 
        object deleteMethod = this.GetResolvedMethodData(sourceType, this.DeleteMethod, dataObjectType, dataObject, null, DataSourceOperation.Delete);
        IOrderedDictionary parameters = ExtractMethodParameters(deleteMethod);
        ObjectDataSourceMethodEventArgs args = new ObjectDataSourceMethodEventArgs(parameters);
        this.OnDeleting(args);
 
        if (args.Cancel)
        {
            return 0;
        }
 
        object result = this.InvokeMethod(deleteMethod);
 
        this.Owner.InvalidateCache();
        this.OnDataSourceViewChanged(EventArgs.Empty);
        return ExtractAffectedRows(result);
    }
 
    protected override int ExecuteInsert(IDictionary values)
    {
        if (!this.CanInsert)
        {
            throw new NotSupportedException(ExposedSR.GetString(ExposedSR.InsertNotSupported, this.Owner.ID));
        }
 
        // we only change the behavior of sources that provide a DataObjectTypeName
        if (String.IsNullOrEmpty(this.DataObjectTypeName))
            return base.ExecuteInsert(values);
 
        Type sourceType = this.GetType(this.TypeName);
        Type dataObjectType = this.TryGetDataObjectType();
 
        if (values == null || values.Count == 0)
        {
            throw new InvalidOperationException(ExposedSR.GetString(ExposedSR.InsertRequiresValues, this.Owner.ID));
        }
 
        IDictionary insertParameters = new OrderedDictionary(StringComparer.OrdinalIgnoreCase);
        MergeDictionaries(this.InsertParameters, values, insertParameters);
        object dataObject = this.BuildDataObject(dataObjectType, insertParameters);
 
        object insertMethod = this.GetResolvedMethodData(sourceType, this.InsertMethod, dataObjectType, null, dataObject, DataSourceOperation.Insert);
        IOrderedDictionary parameters = ExtractMethodParameters(insertMethod);
        ObjectDataSourceMethodEventArgs args = new ObjectDataSourceMethodEventArgs(parameters);
        this.OnInserting(args);
 
        if (args.Cancel)
        {
            return 0;
        }
 
        object result = this.InvokeMethod(insertMethod);
 
        this.Owner.InvalidateCache();
        this.OnDataSourceViewChanged(EventArgs.Empty);
        return ExtractAffectedRows(result);
    }
 
    protected override int ExecuteUpdate(IDictionary keys, IDictionary values, IDictionary oldValues)
    {
        if (!this.CanUpdate)
        {
            throw new NotSupportedException(ExposedSR.GetString(ExposedSR.UpdateNotSupported, this.Owner.ID));
        }
 
        // we only change the behavior of sources that provide a DataObjectTypeName
        if (String.IsNullOrEmpty(this.DataObjectTypeName))
            return base.ExecuteUpdate(keys, values, oldValues);
 
        object updateMethod;
        Type sourceType = this.GetType(this.TypeName);
        Type dataObjectType = this.TryGetDataObjectType();
 
        if (this.ConflictDetection == ConflictOptions.CompareAllValues)
        {
            if (oldValues == null || oldValues.Count == 0)
            {
                throw new InvalidOperationException(ExposedSR.GetString(ExposedSR.Pessimistic, ExposedSR.GetString(ExposedSR.Update), this.Owner.ID, "oldValues"));
            }
 
            IDictionary oldParameters = new OrderedDictionary(StringComparer.OrdinalIgnoreCase);
            MergeDictionaries(this.UpdateParameters, oldValues, oldParameters);
            MergeDictionaries(this.UpdateParameters, keys, oldParameters);
            object oldDataObject = this.BuildDataObject(dataObjectType, oldParameters);
 
            IDictionary newParameters = new OrderedDictionary(StringComparer.OrdinalIgnoreCase);
            MergeDictionaries(this.UpdateParameters, oldValues, newParameters);
            MergeDictionaries(this.UpdateParameters, keys, newParameters);
            MergeDictionaries(this.UpdateParameters, values, newParameters);
            object newDataObject = this.BuildDataObject(dataObjectType, newParameters);
 
            // if oldDataObject and newDataObject are the same object, this is a bit odd... but since we
            // built them old-then-new, the resulting object has the correct values. In general we aren't
            // going to be running that way.
 
            updateMethod = this.GetResolvedMethodData(sourceType, this.UpdateMethod, dataObjectType, oldDataObject, newDataObject, DataSourceOperation.Update);
        }
        else
        {
            IDictionary updateParameters = new OrderedDictionary(StringComparer.OrdinalIgnoreCase);
            MergeDictionaries(this.UpdateParameters, oldValues, updateParameters);
            MergeDictionaries(this.UpdateParameters, keys, updateParameters);
            MergeDictionaries(this.UpdateParameters, values, updateParameters);
            object dataObject = this.BuildDataObject(dataObjectType, updateParameters);
 
            updateMethod = this.GetResolvedMethodData(sourceType, this.UpdateMethod, dataObjectType, null, dataObject, DataSourceOperation.Update);
        }
 
        IOrderedDictionary parameters = ExtractMethodParameters(updateMethod);
        ObjectDataSourceMethodEventArgs args = new ObjectDataSourceMethodEventArgs(parameters);
        this.OnUpdating(args);
 
        if (args.Cancel)
        {
            return 0;
        }
 
        object result = this.InvokeMethod(updateMethod);
 
        this.Owner.InvalidateCache();
        this.OnDataSourceViewChanged(EventArgs.Empty);
        return ExtractAffectedRows(result);
    }
 
    public virtual object AcquireDataObject(Type dataObjectType, IDictionary inputParameters)
    {
        return Activator.CreateInstance(dataObjectType);
    }
 
    // Oh, if only this was a virtual on ObjectDataSourceView!
    private object BuildDataObject(Type dataObjectType, IDictionary inputParameters)
    {
        object dataObject = AcquireDataObject(dataObjectType, inputParameters);
        PropertyDescriptorCollection properties = TypeDescriptor.GetProperties(dataObjectType);
 
        foreach (DictionaryEntry entry in inputParameters)
        {
            string propertyName = (entry.Key == null) ? string.Empty : entry.Key.ToString();
            PropertyDescriptor descriptor = properties.Find(propertyName, true);
 
            if (descriptor == null)
            {
                throw new InvalidOperationException(ExposedSR.GetString(ExposedSR.DataObjectPropertyNotFound, new object[] { propertyName, this.Owner.ID }));
            }
 
            if (descriptor.IsReadOnly)
            {
                throw new InvalidOperationException(ExposedSR.GetString(ExposedSR.DataObjectPropertyReadOnly, new object[] { propertyName, this.Owner.ID }));
            }
 
            object propertyValue = BuildObjectValue(entry.Value, descriptor.PropertyType, propertyName);
            descriptor.SetValue(dataObject, propertyValue);
        }
 
        return dataObject;
    }
 
    #region Proxies for private methods
    private AcquiringObjectDataSource Owner
    {
        get { return (AcquiringObjectDataSource)s_Owner.GetValue(this); }
    }
 
    private Type GetType(string typeName)
    {
        return (Type)s_GetType.Invoke(this, new object[] { typeName });
    }
 
    private Type TryGetDataObjectType()
    {
        return (Type)s_TryGetDataObjectType.Invoke(this, null);
    }
 
    private static void MergeDictionaries(ParameterCollection reference, IDictionary source, IDictionary destination)
    {
        s_MergeDictionaries.Invoke(null, new object[] { reference, source, destination });
    }
 
    private static object BuildObjectValue(object value, Type destinationType, string paramName)
    {
        return s_BuildObjectValue.Invoke(null, new object[] { value, destinationType, paramName });
    }
 
    private object GetResolvedMethodData(Type type, string methodName, Type dataObjectType
                                         , object oldDataObject, object newDataObject, DataSourceOperation operation)
    {
        return s_GetResolvedMethodData.Invoke(this, new object[] { type, methodName, dataObjectType, oldDataObject, newDataObject, operation });
    }
 
    private IOrderedDictionary ExtractMethodParameters(object method)
    {
        return (IOrderedDictionary)s_Parameters.GetValue(method);
    }
 
    private object InvokeMethod(object method)
    {
        return s_InvokeMethod.Invoke(this, new object[] { method });
    }
 
    private int ExtractAffectedRows(object result)
    {
        return (int)s_AffectedRows.GetValue(result);
    }
    #endregion
}
```

Once again, I'm doing the reflection work once in a static constructor. In my next (and final?) post on this mess, I'll show you how my DynamicCall stuff makes this code look much nicer and makes it tons type-safer (hmmm... is that something I want to have forever Google-able?). There are a couple of notes.

1.  In all but the insert case, I have a list of keys provided by my caller in a seperate collection. It would be nice to be able to pass that along to the `AcquireDataObject` method, but it turns out that this isn't necessarily possible or sufficient. Since the `ObjectDataSource` forwards any explicitly-defined parameters through to the `ObjectDataSourceView` the key collection given to the update and delete methods may or may not contain the actual key. Rather, the key could be coming from the `InsertParameters`, `DeleteParameters` or `UpdateParameters` collections (which get merged-in in the respective mutation methods).
2.  It would be nice for the `AcquireDataObject` method to be told why we were trying to acquire the object (e.g. are we doing an insert, update or delete). I didn't make that change in this verison simply to make it easier the correlation to the Microsoft verison, but in the final post, I've added that argument. This allows calling the correct factory method or any other operation-specific behavior.
3.  The calls to `this.Owner.InvalidateCache()` replaces the bit of class-envy in the original Microsoft version that I talked about in [part one](http://musingmarc.blogspot.com/2006/07/enough-with-whining-just-fix.html "Fixing ObjectDataSource").

---

### Comments

#### Hi Marc…

this is indeed a major pain you are tr...
[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-08-22T10:15:00.000-05:00">Aug 2, 2006</time>

Hi Marc,  
this is indeed a major pain you are trying to resolve here... I think it will be of great use for us.  
However, I could not find the source code for AcquringObjectDataSource class in your download section.  
Will it be possible to make it available, as well as a working sample if you got one.  
  
Thank you in advance, Eyal.
---

#### I had the same issues; we went down a different pa…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-04-12T03:15:00.000-05:00">Apr 4, 2007</time>

I had the same issues; we went down a different path using a proxy and generics.  
  
[Posted Here](http://adammills.wordpress.com/2007/04/12/the-evil-of-objectdatasource-overcome/)
---

#### It's of a great use for us using BLToolkit for ORM…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2008-02-04T03:14:00.000-06:00">Feb 1, 2008</time>

It's of a great use for us using BLToolkit for ORM! Thank you!  
A single problem - schema isn't transferred correctly to visual controls of ASP.NET when using your wonderful AODS (refresh schema doesn't work).  
Could you help with some directions on where I should look to fix it? It's not a big issue, but still a bit inconvinient...  
  
I'd appreciate any help to kyrel\[at\]list\[dot\]ru  
  
BR,  
Mikhail Kirillov
---

#### Extremely useful post by the way..…


  
By sett...
[Andrew](https://www.blogger.com/profile/01174630370937086286 "noreply@blogger.com") - <time datetime="2008-06-05T15:36:00.000-05:00">Jun 4, 2008</time>

Extremely useful post by the way...  
  
By setting the update method on your ODS to one that does NOT take a business object as an input parameter, you can prevent the ODSV from calling Activator.CreateInstance. Because the ODS's DataObjectTypeName will be null if you use an update method that takes a boatload of object properties and keys as input parameters, this line of code will prevent the Activator.CreateInstance call:  
  
if (dataObjectType != null)  
  
This is the 8th line of code in the ExecuteUpdate method of the ODSV
---

#### This is a fantastic article. Using your data sourc…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2008-07-03T04:08:00.000-05:00">Jul 4, 2008</time>

This is a fantastic article. Using your data source it is easy to plug your own custom business objects into ObjectDataSource. As you can now update objects properly, and not have to bind to every single property using hidden controls.  
  
For our business objects we did the following:  
  
public virtual object AcquireDataObject(Type dataObjectType, IDictionary inputParameters)  
{  
#region Generic Controller  
  
Type controllerObjectType = this.GetType(this.TypeName);  
  
MethodInfo methodGetPrimaryKeyValue = controllerObjectType.GetMethod("GetPrimaryKeyValue", BindingFlags.Static | BindingFlags.Public | BindingFlags.FlattenHierarchy);  
MethodInfo methodGetEntity = controllerObjectType.GetMethod("GetEntity", BindingFlags.Static | BindingFlags.Public | BindingFlags.FlattenHierarchy, null, new Type\[\] { typeof(object) }, null);  
PropertyInfo propertyPrimaryKey = controllerObjectType.GetProperty("PrimaryKeyDBColumnName", BindingFlags.Static | BindingFlags.Public | BindingFlags.FlattenHierarchy);  
  
string primaryKeyValue = propertyPrimaryKey.GetValue(controllerObjectType, null) as string;  
  
object entityPrimaryKey = inputParameters\[primaryKeyValue\];  
object loadedEntity = methodGetEntity.Invoke(null, new object\[\] { entityPrimaryKey });  
  
return loadedEntity;  
#endregion  
  
//return Activator.CreateInstance(dataObjectType);  
}
---

#### Obviously Microsoft made internal changes in .NET …


[Unknown](https://www.blogger.com/profile/14213138785266454410 "noreply@blogger.com") - <time datetime="2012-09-12T07:45:52.120-05:00">Sep 3, 2012</time>

Obviously Microsoft made internal changes in .NET 4.5 and this means some code of the AcquiringObjectDataSourceView is broken.  
  
Solution for those who want to use .NET 4.5 (or lower), here you go!  
  
**Add class IsNet45.cs with following code:**  
  
public class IsNet45  
{  
public static bool IsNet45OrNewer()  
{  
// Class "ReflectionContext" exists from .NET 4.5 onwards.  
return Type.GetType("System.Reflection.ReflectionContext", false) != null;  
}  
}  
  
**Second, add variable t\_ParsingCulture in the AcquiringObjectDataSourceView class:**  
  
private static readonly Type t\_ParsingCulture =  
Type.GetType(  
"System.Web.UI.WebControls.ParsingCulture, System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a");  
  
**In the static constructor of AcquiringObjectDataSourceView, find 'Type\[\] buildObjectValue' and replace with:**  
  
Type\[\] buildObjectValue = null;  
  
//Version specific code  
if(IsNet45.IsNet45OrNewer())  
{  
buildObjectValue = new Type\[\] { typeof(object), typeof(Type), typeof(string), t\_ParsingCulture };  
  
} else  
{  
buildObjectValue = new Type\[\] { typeof(object), typeof(Type), typeof(string) };  
}  
  
**Search the 'BuildObjectValue' subroutine and replace with:**  
  
private static object BuildObjectValue(object value, Type destinationType, string paramName)  
{  
//Version specific code  
if (IsNet45.IsNet45OrNewer())  
return s\_BuildObjectValue.Invoke(null, new object\[\] { value, destinationType, paramName, Activator.CreateInstance(t\_ParsingCulture) });  
  
return s\_BuildObjectValue.Invoke(null, new object\[\] { value, destinationType, paramName });  
  
}  
  
This should work!
---
