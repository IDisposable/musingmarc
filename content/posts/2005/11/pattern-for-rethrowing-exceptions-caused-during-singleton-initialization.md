+++
title = "Pattern for rethrowing exceptions caused during singleton initialization"
date = 2005-11-11T16:17:00.001-06:00
updated = 2008-05-02T14:54:12.320-05:00
draft = false
url = '/2005/11/pattern-for-rethrowing-exceptions.html'
tags = ["provider","exception","best practice"]
+++

If you are doing [provider](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnaspnet/html/asp02182004.asp) or other [similar](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnaspnet/html/asp04212004.asp) constructs that have late-constructed [singleton](http://) behavior, you need to be sure that anyone using the singleton will not proceed with an erroneously constructed object. But **how** do you let them know something went wrong originally? Simply rethrow the original exception again! I found this pattern in the [System.Web.Profile.ProviderBase](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnaspnet/html/asp02182004.asp) class thanks to [Reflector](http://www.aisto.com/roeder/dotnet/)

```csharp
public class Default : Singleton
{
    public Default()
    {
        // anything else you really want to happen in the instance constructor...
    }
}

public class Singleton
{
    private static bool s_Initialized;
    private static object s_InitializeLock;
    private static Exception s_InitializeException;
    private static Singleton s_Instance;

    static Singleton()
    {
        s_InitializeLock = new object();
        s_InitializeException = null;
        s_Initialized = false;
        s_Instance = null;
    }

    protected Singleton()
    {
        if (!s_Initialized)
        {
            InitializeStatic();
        }

        // anything else you really want to happen in the base constructor...
    }

    public void Create()
    {
        InitializeStatic();

        if (s_Instance != null)
        {
            return s_Instance;
        }

        lock (s_InitializeLock)
        {
            if (s_Instance == null)
            {
                s_Instance = new Default();
            }
            
            return s_Instance;
        }
    }
    
    private static void InitializeStatic()
    {
        if (s_Initialized)
        {
            if (s_InitializeException != null)
            {
                throw s_InitializeException;
            }
        }
        else
        {
            lock (s_InitializeLock)
            {
                if (s_Initialized)
                {
                    if (s_InitializeException != null)
                    {
                        throw s_InitializeException;
                    }
                    
                    return;
                }
                try
                {
                    // do real singleton initialization
                }
                catch (Exception ex)
                {
                    if (s_InitializeException == null)
                    {
                        s_InitializeException = ex;
                    }
                }
                
                s_Initialized = true;
            }
            
            if (s_InitializeException != null)
            {
                throw s_InitializeException;
            }
        }
    }
    
    internal static Singleton Instance
    {
        get
        {
            InitializeStatic();
            return s_Instance;
        }
    }
}
```
