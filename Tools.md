# Tools

Here's a few things you'll want to have in your toolkit.


## VirtualBox

[VirtualBox](https://www.virtualbox.org/) -- v4.3.18, running on Mac OSX -- was my container of choice.
Creating an 32-bit Windows XP VM takes just a few clicks, so I'll skip the details.

At various points, you may need to [download](https://www.virtualbox.org/wiki/Downloads) and install

- the Oracle VM VirtualBox **Extension Pack**,

  onto your host workstation

- the Oracle VM VirtualBox **Guest Additions**,

  via the 'Devices > Insert Guest Additions CD ...' menu option for your VM

You can reach the VirtualBox BIOS by pressing the `F12` key while your VM is booting.
If you have keyboard mapping issues in Mac OS X, [this article](http://www.micromux.com/2009/08/25/virtual-box-virtual-keys/) may be of help.


## Windows XP

> Because sticking with an [end-of-lifed](http://windows.microsoft.com/en-us/windows/lifecycle) OS is just *that much fun*.

Rip yourself an ISO of your old-school Windows XP Setup CD and you can access it in VirtualBox.
It's best to start with an OS install that already includes SP2.
Apparently, there's [a virus going around](http://blog.chron.com/techblog/2008/07/average-time-to-infection-4-minutes/) ...

You'll need an XP license key for when you run the [Repair Installation](Techniques.md#xp-repair-installation) process.
The [Recovery Console](Techniques.md#xp-recovery-console) provides some useful features as well.

After you've made a repaired or fresh installation of XP, be sure to [install all the updates](Techniques.md#windows-updates).


## Virtual Disks

Along the way, you'll be creating a lot of virtual disks.
I chose the VMDK format for all of mine, so I just use the term 'VMDK' in these documents.

I tend to used the Fixed Size option, vs. Dynamically allocated.
Either way, it has a fixed maximum size.
There are [steps you can take](https://blogs.oracle.com/virtualbox/entry/how_to_compact_your_virtual) to compact your VMDK at a later point in time.

For this VM, I chose to split the file into 2Gb parts.
Any `rsync`-ish backup strategy would have to replicate the *entire VMDK* if the virtual disk even got looked at funny.
In this case, slicing it up made [Time Machine](https://support.apple.com/en-us/HT201250) happier.


## XP Helper

> *TL;DR* - It's a VMDK of a bootable baseline instance of [Windows XP](Tools.md#windows-xp).

I recommend that the first VM you set up be just a basic from-scratch Windows XP install with [all updates applied](Techniques.md#windows-updates).
At any time you need to format or partition something in an XP-specific way,
you can just attach your 'target' [VMDK](Tools.md#virtual-disks) via IDE controller to your XP Helper, boot it up,
and pop into [Disk Management](Techniques.md#disk-management)
or Notepad for editing your [`boot.ini` file](Techniques.md#bootini).


## Ghost

My full-machine backup tool of choice was Norton Ghost.
I ripped an ISO of **Norton Ghost 12** --
which has been [end-of-lifed](http://www.symantec.com/business/support/index?page=content&id=DOC7209) --
to use as the Recovery OS under [VirtualBox](Tools.md#virtualbox).
It seems fully backward-compatible with V2I files from prior versions of Ghost.

You'll be using several tools that it provides

- [Norton Support Utilities](Techniques.md#norton-support-utilities),
  to configure your [partitions](Techniques.md#partitioning) and [`boot.ini` file](Techniques.md#bootini)
- The standard **'Recover My Computer' Wizard**, once your VMDK is partitioned.

I tried the Recover OS on Norton Ghost 10 -- similarly end-of-lifed -- but I'm blocked by its DRM requirements even though I have multiple licenses.
[Lots](http://community.norton.com/en/forums/norton-ghost-100-activation-error)
[of](http://community.norton.com/en/forums/norton-ghost-10-activation-fails)
[folks](https://community.norton.com/en/forums/cannot-reinstall-ghost-10-error-msg-says-too-many-installations)
[have](http://community.norton.com/en/forums/norton-ghost-10-activation-fails-my-newer-machine)
experienced the dreaded message

> "You have exceeded the number of allowable installations for this product key."

To overcome this issue, I tried to

- [reclaim a Norton product license](https://support.norton.com/sp/en/us/norton-renewal-purchase/current/solutions/kb20100527223228EN_EndUserProfile_en_us),
- [un-install Ghost cleanly](http://www.symantec.com/business/support/index?page=content&id=TECH110583) and re-install,
- plus any number of other approaches,

*none of which* solved the problem.

Norton Ghost 12 has a little bit of DRM, but it's not nearly as nasty.
Just make sure you've only activated it [on one machine](https://community.norton.com/en/forums/reinstalling-ghost-12-another-pc).
