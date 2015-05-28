# Internet Explorer

> *TL;DR: You'll probably be stuck with IE6, and it'll probably be permanently borked*


### Borked

By this, I mean that it won't load pages.
It just dispatches URL requests off to your default non-IE browser.

I could find no effective way to *force* IE to be the default browser at this point.


## The Quest for IE8

I spent a considerable amount of time attempting to get IE8 to install
as a [Windows Updates](Techniques.md#windows-updates), as it did back in the day.
However, in the end, my VM always ended up with IE6 as the native browser,
and IE6 is effectively broken by the installation of SP3.

Although you may not shed a tear at the loss of Internet Explorer,
you may run into issues with Applications that use an embedded browser for their UI.
You can install [Google Chrome](http://google.com/chrome/) or [Firefox](http://getfirefox.com/) all you want,
but the native embeddable browser for Windows XP will continue to be IE6.

Here's an outline of some attempts I made to cajole Windows Update Agent into installing it "naturally".
There may be useful techniques here for solving other issues that you may run into.


### Revert to IE6

Microsoft [Warns](http://support.microsoft.com/en-us/kb/917964):

> "Before you perform a repair installation of Microsoft Windows XP,
> you must uninstall Windows Internet Explorer 7 or Windows Internet Explorer 8 from the Windows XP-based computer."

So you may need to take these step prior to starting the [Windows Updates](Techniques.md#windows-updates) process.
Using the [Recovery Console](Techniques.md#xp-recovery-console), uninstall IE8 first, then IE7.

NOTE:  `spuninst.txt` -- the uninstaller batch file -- is hard-coded with a drive path of 'C:\'.
That will suck for folks whose Windows installation is not on the boot partition.
Set aside a backup copy, then use Notepad to do a global replace.

<!--
If you get "Access is Denied", `DEL` the files from within XP (vs. the Recovery Console).
Perform whatever `COPY` operations you can from within Recovery Console.
The rest do from within XP, using the `/Y` flag (eg. "Yes, overwrite").
Once you get "No matching files were found", they're gone!
-->

Ultimately, these steps didn't help me.
*I ended up with IE6 anyway.*


### Find an IE8 Installer

This is not an easy task on the public web.
Regardless of what the [this page](http://support.microsoft.com/en-us/kb/318378) suggests,
the Download links will ultimately tell you

> "You are running Windows XP 32-bit. Although Internet Explorer 7 will not run on your system"

I never found an installer binary myself.
Perhaps you could find one somewhere in a Torrent.

> Practical suggestions for finding an IE8 installer are welcomed.


### "Internet Explorer 8 Delivery Through Automatic Updates"

That's the title of [this technical post](https://technet.microsoft.com/en-us/ie/dd365125.aspx).
Sounds like exactly what we want, right? ...

It's not very helpful.
It just describes the magic.
*The magic that never happens.*


### Windows Update Won't Start

Or, according to this [technical article](https://support.microsoft.com/en-us/kb/2497281),

> When you try to start Windows Update from the Start menu,
> the Windows Update window is blank or does not open.

Why, that's exactly what IE6 looks like when you try to manually launch the [Windows Updates](Techniques.md#windows-updates) process!
Perhaps this is the clue we're looking for ...

There are two paths of manual cleanup suggested in the article

- re-registering the Windows Update Service
- forcing re-creation of the `%systemroot%\SoftwareDistribution` folder

However, no amount of re-installation fixes the problem.
*IE6 is simply a husk of its former self.*


### Disabling Anti-Virus Software

It's possible that your Anti-Virus Software could be "protecting" you by silently blocking the installation of some Windows component.
You know, such as IE8 maybe ...

If you get an "Service Pack 3 setup error" message, you can check the Service Pack update log

```
type %windir%\Svcpack.log
```

I found a "725.583: Update.exe failed 1603." message in mine.

This [technical post](https://support.microsoft.com/en-us/kb/949377) suggests that if the log contains
a "DoRegistryUpdates failed" message that you

> "Restart the computer,
> and then close or disable any antivirus or antispyware program that may be running"

You can disable virus protection from the Windows Task Bar.
You could even un-install the software, if you still have the original installer & license key.

If you've been taking [snapshots](Techniques.md#snapshot-your-vm) of your VM,
you can revert back to just before the last Windows Update was run
so that you can ensure nothing's clogging up the pipes.

*I still didn't get IE8 though.*


### "Parameter Is Incorrect"

While watching all the command prompts scripts fly by when your instance reboots and completes an upgrade,
you may notice a "Parameter Is Incorrect" failure message.

I never fully resolved this issue, but these articles were my source of information:

- This [Microsoft Answers page](https://answers.microsoft.com/en-us/windows/forum/windows_xp-windows_update/error-parameter-is-incorrect-after-installinf/e2cf8584-f48b-41a1-95e0-edc08f8a5c24)
  which suggests it's either [Anti-Virus Software](Internet-Explorer.md#disabling-anti-virus-software) interference or
  a need to [revert to IE6](Internet-Explorer.md#revert-to-ie6).

- This [WinWiki page](http://winwiki.org/xp-service-pack-3-parameter-is-incorrect/),
  which suggests using System Restore (if you'd set that up)
  or some downloadable EXE repair script which I wouldn't trust without [snapshotting](Techniques.md#snapshot-your-vm) your VM first.

Sorry, not a lot of reliable knowledge here.
*Again, it's complicated.*


### Faster Updates, Please

In theory, you can [trigger the Automatic Updates process](http://superuser.com/questions/351937/how-do-i-force-windows-to-check-for-updates)
from a command prompt

```
%windir%\system32\wuauclt.exe /detectnow
```

You can even restart the Automatic Updates Service

```
net stop wuauserv
net start wuauserv
```

But ultimately, Windows Update notifications don't seem to arrive any faster.
They just arrive from Microsoft *whenever XP feels like it*.
