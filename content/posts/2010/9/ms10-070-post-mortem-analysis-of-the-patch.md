+++
title = "MS10 - 070 Post Mortem analysis of the patch"
date = 2010-09-28T17:57:00.000-05:00
updated = 2010-09-28T17:57:02.644-05:00
draft = false
url = '/2010/09/ms10-070-post-mortem-analysis-of-patch.html'
tags = ["Microsoft","code review","bug","patch","best practice"]
+++

Now that Microsoft [has patched](http://weblogs.asp.net/scottgu/archive/2010/09/28/asp-net-security-update-now-available.aspx) the [POET vulnerability](http://weblogs.asp.net/scottgu/archive/2010/09/24/update-on-asp-net-vulnerability.aspx), I thought it would be instructive to see what they changed. Now I'm not masochistic enough to disassemble aspnet\_wp.exe or webengine.dll so there are things changed there that I don't know... but I did do a [Reflector](http://www.red-gate.com/products/reflector/) Export of the System.Web and System.Web.Extensions assemblies and then used [BeyondCompare](http://www.scootersoftware.com/moreinfo.php?zz=moreinfo_compare) to do a diff against the files.  

The analysis is simple:
-----------------------

1.  Don't [leak exception](http://www.owasp.org/index.php/Top_10_2007-Information_Leakage_and_Improper_Error_Handling) information - This prevents exploits from seeing what is broken.
2.  Don't short-circuit on padding checks (take the same amount of time for padding correct verses padding broken) - This prevents exploits from seeing [timing difference](http://www.owasp.org/index.php/Covert_timing_channel) for incorrect padding.
3.  Don't be too picky about exception catching in IHttpHandler.ProcessRequest - This prevents exploits from seeing that you caught one kind of exception (CryptographicException) instead of all exceptions.
4.  Switch from [Hash](http://en.wikipedia.org/wiki/Cryptographic_hash_function)\-based [initialization vectors](http://en.wikipedia.org/wiki/Initialization_vector) to [Random](http://en.wikipedia.org/wiki/Random) IVs - This prevents exploits from using the relationship between the data and the hash to decrypt faster.
5.  Allow for backward compatibility - In case this breaks something, allow the new behavior to be [reverted](http://www.mskbarticles.com/index.php?kb=2425938) in-part.
6.  When doing a code review pass, change to make it clear you've considered the new options.

### Here's the blow-by-blow review

#### Changes in System.…


*   **AssemblyInfo.cs**  
    (v2.0/v3.5) Bump AssemblyFileVersion from 2.0.50272.5014 to 2.0.50727.5053  
    (v4.0) Bump AssemblyFileVersion from 4.0.30319.1 to 4.0.30319.206
*   **ThisAssembly.cs**  
    (v2.0/v3.5) Bump InformationalVersion from 2.0.50272.5014 to 2.0.50727.5053  
    (v4.0) Bump the BuildRevisionStr from 1 to 206  
    (v4.) Bump InformationalVersion from 4.0.30319.1 to 4.0.30319.206
*   **HttpCapabilitiesEvaluator.cs**  
    When caching the browser capabilities, sets the cache expiration to sliding time based on the the setting specified (instead of no sliding, no absolute)
*   **MachineKeySection.cs**  
    Calls to EncryptOrDecryptData without specifying an initialization vector type now defaults to _IVType.Random_ instead of _IVType.Hash_.  
    Now uses AppSettings.UseLegacyEncryption flag to determine if the data should be signed.  
    If decrypting and signing, now checks to ensure that the data has unhashed content (GetUnHashedData) and throws if there is no data other than the hash block. If there is no data after the signature, throws new Exception()!  
    VerifyHashedData no longer fast-aborts when the hashed data mismatches. This makes the check take the same amount of time whether a match is found or not.
*   **Handlers\\AssemblyResourceLoader.cs**  
    ProcessRequest now catches any exception and morphs to an undistinguished InvalidRequest instead of leaking it out.  
    (v4.0) Also no longer passes the assembly name to the InvalidRequest exception formatting.
*   **Security\\MachineKey.cs** _(v4.0)_  
    Decode respects the new AppSettings.UseLegacyMachineKeyEncryption setting. \[note, not AppSettings.UseLegacyEncryption\]
*   **Security\\MembershipAdapter.cs** _(v4.0)_  
    EncryptOrDecryptData now explicitly passes false for signData to MachineKeySection.EncryptOrDecryptData to remain compatible with prior calls, whilst still showing this code has been checked.
*   **Security\\MembershipProvider.cs**  
    DecryptPassword and EncryptPassword now explicitly passes false for useValidationSymAlgo and signData to MachineKeySection.EncryptOrDecryptData to remain compatible with prior calls, whilst still showing this code has been checked.
*   **UI\\WebControls CheckBoxField.cs CheckBoxList.cs MailDefinition.cs ObjectDataSource.cs** and **WebControl.cs**  
    (v4.0) Various property attributes have been reordered \[unrelated, artifact of a new compiler version?\]
*   **UI\\ObjectStateFormatter.cs**  
    Deserialize now ignores the caught exception and now throws the _MacValidationError_ without leaking out the exception details.
*   **UI\\Page.cs**  
    Now flags the page response as in-error for **any** exception, not just a _CryptographicException_, thus not leaking out the exception kind.
*   **Util\\AppSettings.cs**  
    Now exposes a _UseLegacyEncryption_ setting to allow reverting to old behavior for encryption. \[defaults to false\]  
    (v4.0) Now exposes a _UseLegacyMachineKeyEncryption_ setting to allow reverting to old behavior for MachineKey encryption. \[defaults to false\]  
    (v4.0) Now exposes a _ScriptResourceAllowNonJsFiles_ setting to allow reverting to old behavior for ScriptResource encryption. \[defaults to false\]
*   **Util\\VersionInfo.cs** _(v4.0)_  
    Bumped the serialized SystemWebVersion property from 4.0.30319.1 to 4.0.30319.206

#### System.Web.Extensions _(v4.…


*   **ScriptResourceHandler.cs**  
    ProcessRequest now catches any exception and throws an undistinguished 404 error.  
    ProcessRequest now uses the new AppSettings.ScriptResourceAllowNonJsFiles setting to control behavior when a resource is requested that doesn't end in ".js" If enabled (**NOT DEFAULT**), then resource is allowed, otherwise now will throw an undistinguished 404 error.
*   **VersionInfo.cs**  
    Bumped the serialized SystemWebVersion property from 4.0.30319.1 to 4.0.30319.206
*   **ThisAssembly.cs**  
    Bumped the build revision from 1 to 206

---

### Comments

#### Very interesting! Than…

[Greg Bray]( "noreply@blogger.com") - <time datetime="2010-09-28T20:52:06.000-05:00">Sep 2, 2010</time>

Very interesting! Thanks!
---

#### awesome! than…


[Shiva]( "noreply@blogger.com") - <time datetime="2010-09-29T05:23:00.000-05:00">Sep 3, 2010</time>

awesome! thanks!
---

#### Fantastic, thanks for taking the time to look this…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2010-09-29T12:26:08.000-05:00">Sep 3, 2010</time>

Fantastic, thanks for taking the time to look this over. You're referenced on StackOverflow.com: http://stackoverflow.com/questions/3824125/how-was-the-oracle-padding-attack-on-asp-net-fixed/3824238
---

#### Very nice detailed info, saved me the trouble from…


[eglasius]( "noreply@blogger.com") - <time datetime="2010-09-29T13:12:34.000-05:00">Sep 3, 2010</time>

Very nice detailed info, saved me the trouble from opening it :)  
  
As I mentioned in the stackoverflow question @Guest linked to, imho the main fix was to sign any encrypted data that is sent to the browser.  
  
Others are there for compatibility, further try to prevent side channels and take out the unnecessary risk of non js files exposure. Not saying these aren't important, specially since what the impact the researches claimed was only possible because of the web.config exposure and having the machine keys in it: http://eglasius.blogspot.com/2010/09/aspnet-padding-oracle-how-it-relates-to.html. Note that was confirmed in a public tweet response by one of the researches, they didn't break features that had validation until they got the keys from the web.config (which was the default in DotNetNuke, which is supposed to have 600k+ installs out there)
---

#### Couple of clarifications: 1) Although calls to Enc…


[pitz]( "noreply@blogger.com") - <time datetime="2010-09-30T13:09:15.000-05:00">Sep 4, 2010</time>

Couple of clarifications:  
1) Although calls to EncryptOrDecryptData without specifying an initialization vector type now defaults to IVType.Random, the calls coming from Page and the resource handlers (going through Page) still specify Hash as the IV type.  
2) VerifyHashedData is new and doesn't fast-abort on a mismatch. This is not the original padding check that's within the crypto services, so the crypto services are still "susceptible" to timing attacks. However, the HMAC hash verification is performed before the padding check and the attacks would fail there even before reaching the crypto services.
---

#### WRT preventing leaks. They may have improved it bu…


[Yaron Naveh]( "noreply@blogger.com") - <time datetime="2010-10-01T12:30:31.000-05:00">Oct 5, 2010</time>

WRT preventing leaks. They may have improved it but I do not see how it is possible in 100%. If someone tampers with data (let's say it is not signed) and it is not a valid data for the algorithm to decrypt then this is one type of error. If the data is valid and now the application needs to decide it had decrypted nonsense this is another type. There will always be a side channel (e.g. time) that leaks. Unless they use random sleep?  
  
Also why does using a random IV help? AFAIK the attacker guesses the IV and does not assume anything about it anyway.
---

#### Great post! Tha…


[Waleed Eissa]( "noreply@blogger.com") - <time datetime="2010-10-07T03:54:11.000-05:00">Oct 4, 2010</time>

Great post! Thanks
---

#### Thank you for this post. We are running something …


[Min]( "noreply@blogger.com") - <time datetime="2011-02-18T17:19:32.243-06:00">Feb 5, 2011</time>

Thank you for this post. We are running something that uses aspnet Membership provider passwords directly, and this was helpful in knowing how to call the changed EncryptOrDecryptData call.
---
