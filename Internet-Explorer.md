# Internet Explorer

## Install IE8

*Don't wait another moment!*

Go and download an Offline Installer for IE8

- [superlogical.net](http://pc.superlogical.net/windows/internet-explorer/2011/04/internet-explorer-8-offline-installer/)
- [itechtics.com](http://www.itechtics.com/download-internet-explorer-all-versions/)
- [webisee.com](http://www.webisee.com/2009/03/20/download-ie8-standalone-installer/)

You can install it either before *or* after you [update to SP3](Techniques.md#windows-updates).

If you feel like it, there's an IE7 installer.


## Don't Install IE7

When I was strugging to get beyond an
["The version of iE you have installed does not match the update you are trying to install."](http://help.lockergnome.com/windows2/KB986688-update-install-wrong-version-ideas--ftopict433845.html)
error from a (what turned out to be bogus) installer for IE8,
I felt I might just have to *sigh* install IE7 before IE8.

Oh, and just to make it more fun, I was using an [Online Installer](https://www.microsoft.com/en-us/download/details.aspx?id=32072) for IE7.

First, I let it apply the Latest Updates and the Malicious Software tool as it suggested ... you know, *just because*.
Then I ultimately found is that no matter what you do, you end up with

> "Setup was unable to install the latest Windows Internet Explorer updates.
>  After you restart your computer ... click Windows Update"

Of course, IE7 doesn't properly install, and you end up [borked](Internet-Explorer.md#borked)
without the ability to run the [Windows Update Agent](Techniques.md#windows-updates) that it so kindly recommends.

So yeah, IE7 is a waste of your time.
This may be a controversial stance to take, but I'm taking it.


## Borked

This document used to lead with:

> *TL;DR: You'll probably be stuck with IE6, and it'll probably be permanently borked*

because that's just what happens once you [upgrade to SP3](Techniques.md#windows-updates).

And by "borked", I mean that it won't load pages.
It just dispatches URL requests off to your default non-IE browser.

This is mostly annoying when you try to

- use Windows Update Agent after you've installed SP3
- use some third-party app which implemented an embedded browser instance for rendering a web page.
  And yep, that'd be a failing instance of the IE6 engine.

So, until that fine day when I found a proper [IE8 installer](Internet-Explorer.md#install-ie8),
my re-built XP instances were weak tea in the Browser department.


## TL;DR

**The rest of this document** describes the terrible pain of trying to install IE6 without an [Offline Installer](Internet-Explorer.md#install-ie8).
It's a link-rich treatise that took too damn long to research &amp; write -- and is too goddamn full of *'character'* -- for me to just shitcan it.

So I present you this rant, in the spirit of what it's like to beat your head against a fucking wall of merciless technology.
Because, you know, commercial computing is such a solid, perfect, reliable science ...


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
