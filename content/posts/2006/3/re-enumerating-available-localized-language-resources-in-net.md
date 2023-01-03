+++
title = "RE: Enumerating available localized language resources in .NET"
date = 2006-03-27T18:02:00.000-06:00
updated = 2006-04-12T14:59:59.170-05:00
draft = false
url = '/2006/03/re-enumerating-available-localized.html'
tags = []
+++

The coolest thing about the blog culture is people that respond to comments (and suggested topics). One person that **consistently goes above the call** is [Michael Kaplan](http://blogs.msdn.com/michkap/ "Sorting it all out"). On Friday, I asked a question via his _contact me_ link, and by 9am Saturday he had grokked my question, found a solution, and [blogged about](http://blogs.msdn.com/michkap/archive/2006/03/25/560838.aspx "Enumerating available localized language resources in .NET") it for the universe to share. This is awesome! Microsoft is lucky to have someone of Michael's caliber, and we are lucky to have him exposed and working _on weekends_ for us.

Unfortunately, it's not all peaches and cream in .Net land. Frankly the solution he gave is suboptimal for my use (through no fault of Michael's). Let me explain my situation, the initial solution, the problems, and the eventual resolution

### My requirements

#### Hard requirements

*   I'm coding an ASP.Net 2.0 application
*   The application has a select for the user to choose the language they want the application to be rendered in
*   The selection is maintained in the ASP.Net Profile both for anonymous and logged-in users
*   The selected language is used to force the servicing thread's `CurrentCulture` **and** `CurrentUICulture`

#### Soft requirements

*   We want to allow the setup user to define the locales they want in the list
*   We want to default the list to all _installed_ translations available to based on the installed .Net runtime language packs.

So, first step is to enumerate the `CultureInfo` objects like this (using a custom business object called Locale and [WilsonORMapper](http://www.ormapper.net "The easiest way to map POCOs to a SQL database, Wilson's O/R Mapper"))

```
public static void Fill()
{
    ObjectSpace space = Manager.ObjectSpace.IsolatedContext;
 
    using (Transaction transaction = space.BeginTransaction(IsolationLevel.ReadCommitted))
    {
        Dictionary<string, Locale> locales = new Dictionary<string, Locale>();
 
        foreach (Locale existingLocale in space.GetCollection<Locale>(string.Empty))
        {
            locales.Add(existingLocale.LocaleCode, existingLocale);
        }
 
        // Grab a type that we know is in mscorlib
         Assembly mscorlib = Assembly.GetAssembly(typeof(System.Object));
 
        // Enumerate through all the languages .NET may be localized into
        foreach (CultureInfo ci in CultureInfo.GetCultures(CultureTypes.SpecificCultures))
        {
            string name = ci.Name;

            if ( ! locales.ContainsKey(name))
            {
               Locale newLocale = new Locale();
               newLocale.LocaleCode = name;
               newLocale.Description = ci.NativeName;
               newLocale.InvariantDescription = ci.DisplayName;
               space.StartTracking(newLocale, InitialState.Inserted);
               locales.Add(name, newLocale);
            }
        }
 
        transaction.PersistChanges(locales.Values, PersistDepth.ObjectGraph);
        transaction.Commit();
    }
}
```

This is pretty simple. The problem is that I end up with more than 159 specific cultures (and 69 neutral cultures) available, only a few of which have localized .Net runtimes installed. This is where I searched and eventually punted to Micheal. He suggested that I can use the `Assembly.GetSatelliteAssembly(CultureInfo ci)` method to feel out the correct culture. He also changed his call to `CultureInfo.GetCultures()` to also include the `CultureTypes.NeutralCultures` enumeration flag.

Several problems with this approach pop up. Firstly, since we are calling `GetSatelliteAssembly`, it has to have a way to communicate when the assembly cannot be located. It does this by throwing a `FileNotFoundException`. That sucks because I'm going to get hundreds of exceptions for the 24 or so available localizations, and exceptions are a terrible performance hit (not to mention bad API design). This could be easily cured if Microsoft had exposed the ([Reflectored](http://www.aisto.com/roeder/dotnet/ "Better than Rotor, see the real deal!")) `InternalGetSatelliteAssembly` method, which takes a boolean flag to determine if it should `throwOnFileNotFound`. Alas, that isn't exposed and I certainly am not going to start twiddling the internal members of `mscorlib`. Another possibility is the tantalizing `ResourceManager.TryLookingForSatellite()`, but it's private as well.

The other problem with this is the fact that I am getting back neutral cultures. These are great when doing localizations because you can create an English translation without worrying about the differences between British English and United States English. This sucks because you cannot set the `Thread.CurrentUICulture` to any neutral culture (it wants you to be specific, and it'll thunk down to the neutral culture if needed). Thus I've got a tiny list of cultures of which only a couple are actually useable as-is.

Sigh... what to do? I take my hint from the path Michael has started down and do some more reflectoring. So after **much** digging, I find that the best I can do is catch is basically what Michael suggests, so I tried to minimize the number of exceptions thrown _and solve the neutral/specific culture issue_, so what I did was track the list of known installed neutral localizations and test the parents of each specific culture. The final code loos likes this:

```
// Grab a type that we know is in mscorlib
private static readonly Assembly s_MSCorLib = Assembly.GetAssembly(typeof(System.Object));
 
public static void Fill()
{
    ObjectSpace space = Manager.ObjectSpace.IsolatedContext;
 
    using (Transaction transaction = space.BeginTransaction(IsolationLevel.ReadCommitted))
    {
        Dictionary<string, Locale> locales = new Dictionary<string, Locale>();
 
        foreach (Locale existingLocale in space.GetCollection<Locale>(string.Empty))
        {
            locales.Add(existingLocale.LocaleCode, existingLocale);
        }
 
        Dictionary<string, CultureInfo> neutralCultures = new Dictionary<string, CultureInfo>();
 
        // Enumerate through all the neutral cultures first (this saves time in checking all the others...)
        foreach (CultureInfo ci in CultureInfo.GetCultures(CultureTypes.NeutralCultures))
        {
            if (AddIt(locales, neutralCultures, ci))
            {
                neutralCultures.Add(ci.Name, ci);
            }
        }
 
        // Enumerate through all the specific languages .NET may be localized into
        foreach (CultureInfo ci in CultureInfo.GetCultures(CultureTypes.SpecificCultures))
        {
            if (AddIt(locales, neutralCultures, ci))
            {
                // if we got here, the culture localizations are installed.
                Locale newLocale = new Locale();
                newLocale.LocaleCode = ci.Name;
                newLocale.Description = ci.NativeName;
                newLocale.InvariantDescription = ci.DisplayName;
                space.StartTracking(newLocale, InitialState.Inserted);
                locales.Add(ci.Name, newLocale);
            }
        }
 
        transaction.PersistChanges(locales.Values, PersistDepth.ObjectGraph);
        transaction.Commit();
    }
}
 
private static bool AddIt(Dictionary<string, Locale> locales
    , Dictionary<string, CultureInfo> neutralCultures
    , CultureInfo ci)
{
    bool addIt = false;
 
    if (!locales.ContainsKey(ci.Name))
    {
        // if it exists in neutral form, we can use it, special-case English since that is
        // always installed..
        if (neutralCultures.ContainsKey(ci.Name) || ci.TwoLetterISOLanguageName == "en")
        {
            addIt = true;
        }
        else
        {
            try
            {
                Assembly satellite = s_MSCorLib.GetSatelliteAssembly(ci);
                addIt = true;   // if we got here, the localization was found...
            }
            catch (FileNotFoundException)
            {
                if (ci.Parent != null && ci.Parent != CultureInfo.InvariantCulture)
                    addIt = AddIt(ref locales, ref neutralCultures, ci.Parent);
            }
        }
    }
 
    return addIt;
}
```

---
### Comments:
#### Hi Marc,  
  
I actually explained the solution ...
[Anonymous]( "noreply@blogger.com") - <time datetime="2006-04-12T14:59:00.000-05:00">Apr 3, 2006</time>

Hi Marc,  
  
I actually explained the solution in my post -- rather than getting the Assembly by using a .NET Framework intrinsic type, get it from your own application!
<hr />
#### It's not that I don't want to handle the translati...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2006-04-12T16:14:00.000-05:00">Apr 3, 2006</time>

It's not that I don't want to handle the translations I don't have... it's that I don't want the exceptions generated when the the appropriate Framework localization doesn't exist.  
  
I precisely DO want the list of localizations available FOR the Framework, not mine... I'm the slave to what is installed on the client's server. They control the language packs installed, and I want my list to be driven by what the Framework can offer \[in this install\].
<hr />
