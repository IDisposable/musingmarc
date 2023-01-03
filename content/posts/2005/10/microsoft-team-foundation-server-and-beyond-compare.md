+++
title = "Microsoft Team Foundation Server and Beyond Compare"
date = 2005-10-25T13:13:00.000-05:00
updated = 2006-02-20T14:13:51.983-06:00
draft = false
url = '/2005/10/microsoft-team-foundation-server-and.html'
tags = []
+++

Simply put, I **can't stand** any other comparison tool than [Beyond Compare](http://www.scootersoftware.com/). This is a product that **definitely** earns the name!. I would pay more than Abraxis Merge for Beyond Compare, but I don't have to, as they are also the most [reasonably priced](http://www.scootersoftware.com/shop.php?c=products) tool I've ever seen.

I use Beyond Compare to do file differences, directory comparisons, heck I use it to upload the latest version of [XBox Media Center](http://www.xboxmediacenter.com/) to my [hacked](http://www.smartxx.com/specs/specs_01.php) XBox (I use the [pimped](http://bitspace.dyndns.org:6560/index.php) edition).

I'm also using [Microsoft Team Foundation Server](http://lab.msdn.microsoft.com/teamsystem/teamcenters/team/default.aspx)'s source control.

Here's how to set Beyond Compare up as the comparison tool:

* In Visual Studio, click menu `File / Options...`
* Expand `Source Control` in the treeview.
* Click `Visual Studio Team Foundation Server` in treeview.
* Click button `Configure User Tools...`
* Click button `Add...`
* Enter `.\*` in the Extension textbox
* Choose `Compare` in Operation combobox
* Click button `...` next to Command textbox
* Browse to `BC2.EXE` in file chooser and click button `Open`
* Enter `%1 %2 /title1=%6 /title2=%7` into the Arguments textbox.
* `Ok` on out.

Microsoft's version of these instructions is [here](http://msdn2.microsoft.com/en-us/library/ms181446).

**UPDATE**: James Manning has comprehensive instructions for other tools [here](http://blogs.msdn.com/jmanning/articles/535573.aspx).

Now one issue remains, and hopefully either Microsoft or Scooter Software will do something about it. When using an external comparison tool Visual Studio checks both files out to a temporary directory. It appends a tag to indicate the file version. The suffix looks something like `;C19` where the C indicates this is a changeset, and the 19 is the changeset number. It's also possible to see an `;Lxxx` which indicates a label _xxx_. For more information on the suffixes, see [this](http://msdn2.microsoft.com/en-us/library/56f7w6be.aspx) at the Versionspecs section.

**UPDATE #2 and #3**: Both Microsoft and Scooter have addressed this, the RC of VSTS now preserves the file extension; using the suggestions below will improve the display in BeyondCompare until then. This means the external diff tool command line is something like `"BC2.EXE Foo.cs;C19 Foo.cs;C22"`. In Beyond Compare, the funky changeset tag interferes with the file-extension sniffing in Beyond Compare. The actual quick "files are different" compare thinks these are "any other file", so no [rules](http://www.scootersoftware.com/download.php?c=kb_morerules) are run.

Secondly, when the viewer for different files is run, it also thinks these are "any other file", so no language-specific or file-extension specific [plug-ins](http://www.scootersoftware.com/moreinfo.php?c=v2plugins) run.

What is needed is to have Beyond Compare have the comparison and the viewer strip off everything after the last `;` from each filename before determining which rules to run and which viewer to launch. The other option is for Microsoft to not create these _oddly_ named files and instead create a subdirectory for each file that reflects the version/changeset/label, etc. Personally, while I think that the filenames are odd, I'm hoping for a fix from Scooter Software. Why?

1. Having the suffix on the filenames clearly indicates which one is which in the view header.
2. Scooter Software moves like an ice-cube on a hot griddle.
3. Microsoft, especially this close to release, moves like a glacier.

Edit 2005 Oct 25th 11:44CDT: Changed the command line arguments above and posted [this](/2005/10/update-to-integration-between.html) note.

**UPDATE**: James Manning from Microsoft has informed me that they've fixed the file name issue in the post-beta 3 bits. Thanks, gang.

**UPDATE #3**: Confirmed fix in the RC

---

### Comments

#### As predicted, the gang at Scooter Softw…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-10-25T14:37:00.000-05:00">Oct 2, 2005</time>

As predicted, the gang at Scooter Software [moved quickly](http://www.scootersoftware.com/ubbthreads/showflat.php?Cat=&Number=4540&page=0&view=expanded&sb=5&o=&fpart=1) in a temporary work-around. I can add an **\*** to the Rule patterns. This is not a complete solution in that viewers (like the Image Viewer) that sniff the file extension are still not getting the right information, and the rules will match too agressively depending on the order in the rules (e.g. .aspx\* will match Foo.aspx.cs unless the .cs\* rule comes first), but it is a good start. **Thanks, Craig!**

---

#### Well, we are pretty close to release (having shipp…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-25T19:16:00.000-05:00">Oct 2, 2005</time>

Well, we are pretty close to release (having shipped Beta3), but this is not the first tool to have problems with how we're choosing our temporary filenames. I'll bring it up tomorrow and we'll see if it's something we can change for V1.
  
Thanks for the feedback, Marc!

---

#### By the way, we construct labels that should be hel…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-26T07:44:00.000-05:00">Oct 3, 2005</time>

By the way, we construct labels that should be helpful for understanding the 2 files you're editing better. In Beyond Compare, you would use the /title1 and /title2 parameters, so I would recommend setting your Arguments to:
  
`%1 %2 /title1=%6 /title2=%7`
  
Hopefully that will work a little better for you (it works here, it puts the labels in the drop-down initial text and in the window title).

---

#### Thanks for the hints and response James. You've ma…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-10-26T11:58:00.000-05:00">Oct 3, 2005</time>

Thanks for the hints and response James. You've made me eat my pessimistic words. The `**/title1=%6 /title2=%7**` does indeed improve the experience.
Any clues on what the `%5` actually passes for various compare scenarios?

---

#### %5 passes any command-line options you decide to s…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-26T16:46:00.000-05:00">Oct 3, 2005</time>

`%5` passes any command-line options you decide to specify with `/options`. For instance, if you decided "for this particular diff, but not in general, I want to use bc2.exe's `/readonly` option and rule foo" then you could do: `tf diff /options:"/readonly /rules=foo" somefile.cs`
  
When we run the command (and do substitutions on the variables, %1, %2, etc.) the %5 variable, if it's in your Arguments entry, will get substituted with the string you passed in for `/options`.
  
FWIW, this is how Visual SourceSafe's interface worked (%1 through %9, %5 for options) for 3rd-party tools, so we kept it the same to ease migration for those users that were used to configuring tools there.

---

#### That makes sense for the command-line arguments, b…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-10-26T19:05:00.000-05:00">Oct 3, 2005</time>

That makes sense for the command-line arguments, but I'm not sure where in the VisualStudio context it comes in (I suspect it doesn't). In other words, where is the options set when doing a Right-Click Compare?  
  
One other thing I dearly miss versus the VSS interface is the ability in the Source Control Explorer to compare **all** files in a workspace. As it stands now, I have to do a get-latest into a seperate workspace and then use Beyond Compare to compare the two directories... it would be COOL to be able to compare entire directory trees like VSS let me do..

---

#### Directory compare is V2 at this point, but it's a …

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-10-27T13:35:00.000-05:00">Oct 4, 2005</time>

Directory compare is V2 at this point, but it's a common request.  
  
One side note, the current version of this post says:  
  
Leave the default in Arguments textbox (which is %1 %2 /title1=%6 /title2=%7).  
  
That implies the default value includes the /title1 and /title2 arguments, which isn't true.

---

#### Fixed, than…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-10-27T14:34:00.000-05:00">Oct 4, 2005</time>

Fixed, thanks.

---

#### You'll be happy to that the extension handling has…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-11-16T21:28:00.000-06:00">Nov 3, 2005</time>

You'll be happy to that the extension handling has been fixed for RTM. The change got approved and James recently checked in the fix (the version now appears before the extension instead of after it).

---

#### Thanks tons guys! I like the new Agile Microsoft…

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2005-11-16T22:37:00.000-06:00">Nov 3, 2005</time>

Thanks tons guys! I like the new Agile Microsoft :)

---

#### The blog post still makes it sound like we're goin…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-11-29T13:21:00.000-06:00">Nov 2, 2005</time>

The blog post still makes it sound like we're going to be using the temp files with the busted extensions. If you get a chance, it'd be great to update it with a quick note that we fixed the issue for the post-beta3 bits in case others run across this and aren't patient enough to read the comments :)

---

#### Hi Marc…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-02-20T12:41:00.000-06:00">Feb 1, 2006</time>

Hi Marc!
  
Hopefully by now you're running the RC bits and you've been able to get rid of the `*` you added to your rules as our extensions are back to what they should be.
  
I just wanted to mention that I added these Beyond Compare values to a new blog post where I'm going to track command/argument values for various diff/merge tools.
  
[JManning cite](http://blogs.msdn.com/jmanning/articles/535573.aspx)

---

#### i am a fan of beyond compare like you. just config…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2007-09-07T07:11:00.000-05:00">Sep 5, 2007</time>

i am a fan of beyond compare like you. just configured it, took me 2 minutes, and well, it works perfect. coooooolicoooooooooooo !
  
cheers

thomas

---
