+++
title = "The .Net Framework 2.0 language pack install fails on Vista"
date = 2006-12-28T15:34:00.000-06:00
updated = 2006-12-30T03:48:27.215-06:00
draft = false
url = '/2006/12/net-framework-20-language-pack-install.html'
tags = ["Microsoft",".Net","Ladybug","Vista","Connect","Localization","bug"]
+++

I've discovered that you can't install the [.Net Framework Language Packs](http://www.microsoft.com/downloads/details.aspx?FamilyID=39C8B63B-F64B-4B68-A774-B64ED0C32AE7) on Vista. They all claim to have already been installed, yet the localized resources directories are not present.

Looks like the language pack checks if the key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v2.0.50727\_LCID_` exists (where the _LCID_ is that of the pack). If the key exists, the pack declares that it has already been installed (it hasn't) and thus all the localized resources are not installed.

I have found an ugly workaround (note the values, delete the appropriate key, install the language pack, restore any missing values, set any that are now wrong). Reported, validated and workaround entered on Microsoft's Connect site, feel free to [vote here](https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=248617).

Also, the Arabic language pack is built wrong. The internal INI file doesn't even provide a LCID in the key-to-check so it balks outright. I fixed that by extracting the pack manually, fixing the INI and following the above workaround. This issue is reported on Connect [here](https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=248621).

The language packs I'm talking about install the localized resources for the various provided languages for all the user-display strings in the .Net Framework. This includes localized versions of exception messages, the messages shown by validators, ASP.Net wizards, etc. You install these to support end-users running in thier own language, especially for ASP.Net applications.

### Let the whining commence

Don't get me started about how stupid it is that:

1.  Picking what language pack you want to download changes the language of the page itself.
2.  The file you download is served up as langpack.exe (not named uniquely by the locale).
3.  The language of the installer for the language pack doesn't respect the current language of the person running the installer, so you better watch carefully to figure out where the buttons move to when doing a RTL language or what the error messages that display mean if you aren't a native speaker.
4.  There isn't a single file that you can use to install all the language packs.

Can you imagine being a vertical-market developer that needs to show his customers how do this hackery?
