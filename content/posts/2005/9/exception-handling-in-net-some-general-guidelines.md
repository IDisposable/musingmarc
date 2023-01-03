+++
title = "Exception handling in .Net (some general guidelines)"
date = 2005-09-16T14:26:00.001-05:00
updated = 2008-05-02T14:53:01.494-05:00
draft = false
url = '/2005/09/exception-handling-in-net-some-general.html'
tags = [".Net","exception","best practice"]
+++

Based on a generic query on the Advanced .Net mailing list I dumped this:

Regarding Exceptions themselves
-------------------------------

1. Do **NOT** catch exceptions you don't know how to handle - this means correct for the problem or do some unwind work.
2. If you _know_ what to do when an Exception happens, `catch`. If not, **DON'T**.
3. Do **NOT** catch exceptions to merely translate the to another type of exception (even if you do set the `InnerException` property). If you don't have something to add, don't catch. This is wrong:

    ```csharp
    catch (Exception ex)
    {
       // do something interesting (see 1)
       throw new MyCoolException("nothing of value here", ex);
    }
    ```

4. Do **NOT** catch an exception then throw it again, rather do a "rethrow". This is wrong:

    ```csharp
    catch (Exception ex)
    {
       // do something interesting (see 1)
       throw ex; // StackTrace now loses everything below this level
    }
    ```

    Rather **do** this:

    ```csharp
    catch (Exception ex)
    {
       // do something interesting (see 1)
       throw; // notice there's no ex! The original StackTrace is left intact.
    }
    ```

5. Do [**NOT**](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpgenref/html/cpconerrorraisinghandlingguidelines.asp) derive all of your exceptions from `System.ApplicationException` or `System.SystemException` (or some other base level exception of your creation), as you are not adding value by doing so. If you want to standardize your exception creation, write builder methods to create specific exceptions in regular "forms".
  
6. If you do have contextual information you can add to an exception, **DO SO**. Use the `Exception.Data` [collection](http://msdn2.microsoft.com/en-us/library/2wyfbc48(en-us,vs.80).aspx), that's what it is there for! You can add the values of interesting parameters to you methods. This is especially useful in the context of a database layer or other low-level library. You can squirrel-away the SQL and all the parameters. Only do this if you think that these details will be useful for post-mortem diagnosis. If the information you log is transitory it will NOT help tracking down errors from logs. This is (mostly) good:

    ```csharp
    catch (Exception ex)
    {
       ex.Data.Add("SQL", command.Text);
       ex.Data.Add("key", myKey); // or enumerate command Parameters collection
       throw; // (see #3)
    }
    ```

7. If you add things to the `Exception.Data` collection, make sure that you don't conflict with what is already there as this is a `HashTable`. I use the catching-class's name to scope the values. This is much better than #5:

    ```csharp
    catch (Exception ex)
    {
       ex.Data.Add(String.Format("{0}.{1}.SQL", System.Reflection.MethodBase.GetCurrentMethod().DeclaringType.FullName, System.Reflection.MethodBase.GetCurrentMethod().Name), command.Text);
       throw; // (see #3)
    }
    ```

8. If you are catching exceptions to do some undo-work (like rolling back transactions, closing handles, etc.) Strongly consider creating a _very small_ wrapper for that resource and implementing `IDisposable` on it. Then you can have a `using` clause to hide the `try { } finally { }` block.
  
9. Never catch `System.Exception`, `System.SystemException` or `System.ApplicationException` without doing a rethrow except at the top level logging handler. This also applies to the the catching of non-CLS exceptions (don't throw them in the first place). In CLR 2.0, those will be [wrapped](http://blogs.msdn.com/jmanning/archive/2005/09/16/469091.aspx) in RuntimeWrappedException anyway.

On logging exceptions
---------------------

1. Always put a top-level exception catch that logs the exception and don't make (or allow) anyone else do it
2. If you are going to _handle_ the exception (see #1 above), then you can log the one you caught with an "informational" level, then handle it and **DO NOT** throw/rethrow unless the handling is only undo/release (see #3, #7 above).
3. Always log at the highest level possible, anything not handled as in #B above is an "error" level.
4. When logging a message, you should assign a correlation ID to the log entry and make sure that any user-displayed message contains that correlation ID so they can report an error and you can look it up in the logs.
5. Don't log ThreadAbortExceptions unless you really care! In ASP.Net applications, those happen on `Response.Redirect("xxx", true)` calls. You can do:

    ```csharp
    if (exception as ThreadAbortException == null)
    {
       // log it.
    }
    ```

Where to log exceptions
-----------------------

1. Event logs are easliy monitored by WMI et al.
2. Event logs can "fill up" and truncating them or setting the attributes so that they automatically truncate requires administrative rights, so you could get errors that you cannot log.
3. You should really create your OWN event log and not use the generic Application event log, so you can set it up correctly from the get-go.
4. Text files need to be placed somewhere the user has rights (don't you DARE put it on `C:\\`)!
5. If you use text files, disks can fill up so you can get errors that you cannot log.
6. To monitor a text file, you need access to the file (net share, FTP, etc..) and you have to parse them to filter.
7. If you care to do backups of the logs, you have that option and responsibility
8. XML files == text files for the purposes of this discussion. They might be easire to parse for monitoring, but all other comments are the same.
9. For database logging the rules are similar to XML files, excepting you can leverage the backup and space management policies in place for the database itself.
10. Databases can be easily monitored remotely by doing simple SELECTs which makes the parsing and processing easier.
11. Databases can fail... what do you do if the logging database is down? You'll need a backup plan to log THAT error (typically event), and possibly want to redirect the original log entry elsewhere (either file/event).
12. If you want centralized reporting of log messages, you should also consider using a store-and-forward repository like SMTP or MSMQ to send the messages to somewhere else. If course you need to be able to log transport errors somewhere.
13. If you do store-and-forward, make sure you build in heartbeat log entries so you can tell if messages are getting through (from the receipient's view).

Final thoughts
--------------

1. There is no one-best solution... _your_ requirements and restrictions govern to much to make blanket statements. If you can't require administrative rights, event logs are bad. If you can't do file shares, text files are bad, etc.
2. Defensive-in-depth is important. If you want to catch all errors, you need to make an attempt to log errors in you error logging somewhere.
3. It doesn't matter where you log messages if you are not monitoring the log.
4. If it is possible to keep the `AppDomain` alive then, go ahead. For ASP.Net applications, that's usually done by forwarding the user to an error page.

\[Edit: 16 Sept, 2005 based on comments from [Peter Ritchie](http://www.PeterRitchie.com) and [Paul Mehner](http://www.ssdotnet.org), added links to relevent sources and clarifications to `System.ApplicationException` and `System.SystemException`\]

---

### Comments

#### Excellent post. Exception handling in my developme…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-09-17T04:08:00.000-05:00">Sep 6, 2005</time>

Excellent post. Exception handling in my development has always been a crux. I'm gonna follow your guidelines, it's the best I've seen so far.

---

#### Note that the Exception.Data is a framework 2.0 fe…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-12-20T15:44:00.000-06:00">Dec 2, 2005</time>

Note that the `Exception.Data` is a framework 2.0 feature.

---

#### Enjoyed your article - full of useful ti…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-04-23T11:10:00.000-05:00">Apr 1, 2007</time>

Enjoyed your article - full of useful tips.
  
Reached it from a post (by you I assume) [at](http://codebetter.com/blogs/karlseguin/archive/2006/04/05/142355.aspx?CommentPosted=true#commentmessage)
  
Can I suggest, as an ammedment to 6.
  
`ex.Data.Add(String.Format("{0}.{1}.SQL", System.Reflection.MethodBase.GetCurrentMethod().DeclaringType.FullName, System.Reflection.MethodBase.GetCurrentMethod().FullName),command.Text);`
  
This will work in static methods where the 'this' keyword is unavailable and will also provide the method name dynamically.
  
I'd be interested to know if you felt this had drawbacks.

---

#### Last submitted comment should b…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-04-23T11:14:00.000-05:00">Apr 1, 2007</time>

Last submitted comment should be
**`System.Reflection.MethodBase.GetCurrentMethod().Name`** not ~`System.Reflection.MethodBase.GetCurrentMethod().FullName`~

---

#### Neil…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2007-04-23T12:06:00.000-05:00">Apr 1, 2007</time>

Neil:  
  
Excellent point. Updating.

---

#### Regarding #2 above. Wouldn't you want to translat…

[Bill Arnette](https://www.blogger.com/profile/12950174681799205729 "noreply@blogger.com") - <time datetime="2007-04-23T13:25:00.000-05:00">Apr 1, 2007</time>

Regarding #2 above. Wouldn't you want to translate from one exception type to another to reduce coupling? For example, would you want your data access layer to allow a SQLException out to the business layer, or would you want it catch and throw a DAL-defined exception?

---

#### Bill…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2007-04-23T13:55:00.000-05:00">Apr 1, 2007</time>

Bill:
  
It depends, are you going to add value by doing this translation, and this hiding the nature of the real exception?
  
Your example is the classic question about whether the DAL should let provider-specific exceptions leak out. To make a reasonable decision, ask yourself if you would react or catch the exception differently. Would you even catch (other than to log) any of these exceptions and how would you deal with them?
  
Perhaps you are writing a ASP.Net application, in which case you need to generally log the errors and direct to a general error page. In that case wrapping the exception in another gains you nothing.
  
If, on the other hand, you are writing web service and you **want** to obscure the actual database-specific detail when shuffling it out to the caller... in that case you would either wrap low, like you are suggesting, or catch high and issue a generic exception for the WSDL wrapper to pass through. Personally, I would opt for the latter as I can then **verify** at my interface methods that I'm always obscuring the details.  
  
So, long and short of it... if you have a defined interface for the DAL that is **specified** to obscure the details, then you should do so as you are treating that as an black-box service boundary. In that case, the inner exception should **NOT** be set, and it should be logged withing the DAL as needed. This means that you need a cross-cutting concern to catch all exceptions at the service boundary and translate ALL of them... something that should probably be done with an aspect injection system (and thus, once again not code YOU should be writing).

---

#### This is the best post about exception handling i h…

[Sakthi](https://www.blogger.com/profile/13894629780054143558 "noreply@blogger.com") - <time datetime="2007-04-24T03:02:00.000-05:00">Apr 2, 2007</time>

This is the best post about exception handling i have seen so far.

---

#### As I see the matter the rule #1 never applies as y…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-06-07T15:07:00.000-05:00">Jun 4, 2007</time>

As I see the matter the rule #1 never applies as you ALWAYS know what to do when you have an unknow exception:
  
\- Catch it, log it, explain to the user that something wrong has happened and most times end your application (usually you don't know the current state of it) or if the exception concerts only a subsystem of the application stop that subsistem.
  
As with this approach you handle everything rule C changes to "always log at the lowest level". To signal the exception all the methods returns always a boolean with true if the method ended ok or false if it don't.
  
This approach works wonders for me. Just a example:
  
You have a batch processing of x files, if one throws an exception somewhere, the exception gets logged, the false value returned goes up thru the chain of calls and that file gets marked as faulty.

---

#### issoft…

Logging an exception is NOT handlin...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2007-06-07T17:53:00.000-05:00">Jun 4, 2007</time>

issoft:
  
Logging an exception is NOT handling it. That's just noting that you had a problem for later diagnosis and does NOTHING to insure that your code can continue or recover.
  
Handling an exception is something like trying to open a read-only file, which throws an exception that you catch, realize that the file is read-only, change it's attributes and retry.
  
With your strategy of bubbling up a return code indicating success, nothing FORCES a coder to properly check and react that that failure, so they sometimes won't.
  
Let's say that you want to read a file's contents and emit to some service. Once the file is fully processed, you want to move it to a archive directory. During the processing of the file, something throws an exception.
  
With your return code strategy, if the caller of the "process the file" forgot to check the return code, they would move the file into the archive directory EVEN though it wasn't fully processed.
  
With an exception-based strategy, you would fast-fail all the way out of the entire process the file and move the file to a top level handler (where it's logged and possibly moved to a bad directory) without the smallest chance that someone ignored the failure.

---
