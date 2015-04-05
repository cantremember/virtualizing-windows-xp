# Techniques

Breakdowns of common steps you'll need to follow.


## Initial VM Setup

For each new VM you intend to create, you'll need

- a shared virtual IDE Controller
- at least one 'target' [VMDK](Tools.md#virtual-disks), for your new OS
- a virtual CD/DVD drive
- an ISO image for [Ghost](Tools.md#ghost)
- an ISO image of [Windows Setup](Tools.md#windows-xp) for your XP version of choice
- the V2I files containing your Ghost backup

At times, you may also need your [XP Helper](Tools.md#xp-helper) VM.

Make sure to provide at least the same number of CPUs and amount of virtual RAM as the original hardware.
It's easy enough to increase them later -- Windows XP doesn't fall over under such conditions
(like it does when you swap out hardware without [repairing](Techniques.md#xp-repair-installation) the device drivers).


## Partitioning

You may be slicing up your 'target' [VMDK](Tools.md#virtual-disks) to host both an OS and Data volume (or more).
I recommend using the [diskpart](Techniques.md#diskpart) utility on from the Ghost Recovery OS.

### Make a Backup

> If you're starting from scratch, this isn't a problem.
> But it's a good way to way to capture a snapshot of the partition tables you're going to replicate.

It's easy to screw up a partition table, for any of the reasons below.
So make a backup via [Partition Table Operations](Techniques.md#norton-support-utilities).
Your best choice is to save it off to a [USB drive](#backup-files-via-usb).

Otherwise, save it to a formatted partition on the disk itself.
Then mount your VMDK to your [XP Helper](Tools.md#xp-helper), boot up and pull the backup file onto your host machine.
You know, before you go and  overwrite everything.

### Sector math is hard

A [partition table](https://technet.microsoft.com/en-us/library/cc976786.aspx) is a delicate thing,
with sector boundary values related to the geometries of the physical disk.
Of course, *your disk is virtualized*, but it emulates the same kind of hardware-like sizing shennanigans.
You can't blindly copy the values from your old workstation onto your VMDK -- the dimensions aren't compatible.
As a general rule, allocate everything through [diskpart](Techniques.md#diskpart)
and only use [Partition Table Operations](Techniques.md#norton-support-utilities) for edits that don't impact the dimensions.

### Primary, Extended and Logical

Windows Vista very nicely explains [the rules of 'how many'](http://windows.microsoft.com/en-us/windows-vista/what-are-partitions-and-logical-drives),
and this article explains how [Extended & Logical work together](http://ntfs.com/logdrives.htm).

Unless you're going to virtualize more than one OS -- *which you totally could* -- you'll either have

- one Primary partition, housing a single disk for your OS and Data.
- two partitions - one bootable Primary with your OS, and the other an Extended with a single Logical partition for your Data.

IMHO, having a separate OS-and-software disk and Data disk is always a good strategy.
A virtualization effort is a great time to finally split from a one-disk soultion to separate OS & Data.
Even if ... *ahem* ... you're ultimately sticking with an [end-of-life](http://windows.microsoft.com/en-us/windows/end-support-help)'d OS.

There are plenty of tools at your disposal for partition management.

### diskpart

You can access `diskpart` from an MSDOS prompt within the Ghost Restore OS.
It's a more [command-driven](http://support.microsoft.com/kb/300415) version of the tool
than the one you'll find in the [XP Recovery Console](Techniques.md#xp-recovery-console).

The units-of-work are `DISK`s, `PARTITION`s and `VOLUME`s.
You `CREATE` new ones, or `SELECT` an existing one and perform operations upon it.
If you modify the partition table using another tool while `diskpart` is still running, you'll need to do a `RESCAN`.
The built-in `HELP` is very ... helpful.

### Norton Support Utilities

Some classic Norton Utilites for disk management are provided in the [Ghost](Tools.md#ghost) Recovery OS under the 'Utilities' menu.

- **'Partition Table Operations'** lets you backup, restore and edit your partition table.
- **'Edit boot.ini'** gives you a low-level tool for backing up, restoring and editing [the file](Techniques.md#bootini).

The 'Analyze' menu provides

- **'Open Command Shell Window'**, to give you an MSDOS prompt
- **'Explore My Computer'**, for a Windows Explorer interface

### XP Recovery Console

The Recovery Console is launched by pressing "R" at the first screen after booting from the [Windows XP](Tools.md#windows-xp) Setup disk.
If your hard drive / VMDK already has a Windows XP install, you'll need to enter the Administrator password.

> There are a lot of good tools here that I did not use in my efforts.
> Type `help` at the MSDOS prompt.

A version of the `diskpart` utility is available here.
It is [more wizard-based](http://en.wikipedia.org/wiki/Diskpart#Recovery_Console)
than the `diskpart` you'll find in [Norton Support Utilities](Techniques.md#norton-support-utilities).
If you create more than one partition, `diskpart` will also generate a 'bonus' 8MB space between the first and second partitions.
It is always listed last, and does not get displayed by the Norton version.

### Disk Management

Windows XP provides the [Disk Management](https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/dm_create_partitions.mspx) tool for partitioning and formatting disks.
It's launched via 'Administrative Tools > Computer Management > Storage > Disk Management'.

You can use this polite little GUI from your [XP Helper](Tools.md#xp-helper), when booted and mounting your target VMDK.
However, it cannot do low-level edits -- such as updation the partition type -- like [diskpart](Techniques.md#diskpart) can.

### Partition Magic

Speaking of dustware, I had an old copy of [Norton Partition Magic](http://www.symantec.com/about/news/release/article.jsp?prid=20040601_02) 8.0
that I installed on my [XP Helper](Tools.md#xp-helper).
It provides resize operations on partitions, etc.

However, it cannot do low-level edits -- such as updation the partition type -- like [diskpart](Techniques.md#diskpart) can.
And *seriously* ... that press release is like from 2004.


## Restoring with Ghost

Frankly, using Ghost is the *easy part*.
Once you have access to your V2I backup files --
via [USB drive](#backup-files-via-usb) or a [source VMDK](#backup-files-via-vmdk) --
and a large-enough target virtual disk, you can restore everything via the [Recover My Computer](Tools.md#ghost) process as normal.

The devil is in all the details leading up to that point.
[Forum posts like this](https://forums.virtualbox.org/viewtopic.php?t=3700) are a good start,
and I wrote this Wiki to be more of a guided-hand tour of the nether regions of this process.

Some things that I ran into.

- I could not see any Storage drives until they [had a partition](Techniques.md#partitioning).
  Nothing too surprising there.
- I could not access VirtualBox Shared Folders, with my VM in either NAT or Bridge mode,
  after starting 'My Networking Services'.
  That's okay -- it's dustware, and it knows nothing about [VirtualBox Guest Additions](Tools.md#virtualbox).

> Honestly, I'm glad this process worked at all.

Of course, make sure to restore the MBR for your boot partition.

Ultimately, you can only mount four drives at once.
That's all the IDE Primary/Secondary + Master/Slave permutations you've got.
Three should be all you need -- one is your Ghost boot CD, another's your V2I source, and the third is your target drive.

### Backup Files via USB

The most sensible way to have [Ghost](Tools.md#ghost) access your V2I files is via an external USB drive.

- Your USB drive will need to be formatted for FAT or NTFS.
- You'll need to install the [VirtualBox Extension Pack](Tools.md#virtualbox) on your host computer.

Giving your VM access to a hosted USB drive can be tricky, at least on Mac OS X.
I found [this article](https://forums.virtualbox.org/viewtopic.php?t=45349), which became the basis for these instructions.
They worked for any VM, whether booted into Windows XP or Ghost Recovery OS.

- Connect the USB drive to your Mac.
  This will auto-mount a volume.
- Eject the USB volume from OS X Finder.
  Your drive will still be attached, but unmounted.
- Click the USB indicator, and select the USB drive from the menu.
  It'll be enabled once OS X has fully released the mount.
- Watch the USB activity indicator in the Status Bar at the bottom of your VirtualBox VM window.
  If it shows a steady red circle for a while, that's *good*; the VM is negotiating with USB.
- Once the indicator is clear -- it might take a while -- your VM should have mouned the USB drive.

You can do all of this steps without having to re-start the Recovery OS.
However, once you unmount the drive -- deselect it from the USB indicator menu -- or restart the VM,
VirtualBox will lose access to the USB drive, and it will be re-mounted by OS X.
To re-mount the drive to your 'target', just eject the volume from Finder and follow the steps again.

The mounting process can take a while.
Which means that it's trying, and that's an indicator that *you got it right*.

If you're still having issues, try informing VirtualBox of the device you'll want to use.

- Connect the USB drive to your Mac
- In your VM Settings, go to 'Ports > USB'
- [x] Enable everything
- add a USB Device Filter, and select your USB Drive

And if you make sure that the USB is mounted before you click 'Accept' on the EULA dialog, you should never have a problem.

### Backup Files via VMDK

Before I got the [USB steps](#backup-files-via-usb) working, I struggled to assess the huge V2I files that Ghost will need for restoration.
The networking capabilities of Ghost's clunky-but-good-enough Recovery OS were kind of useless, and USB was being troublesome.
I learned that *if all else goes wrong*, you can always take the brute force approach:

- Create a 'Holder' [VMDK](Tools.md#virtual-disks)
- Mount it for, and boot into, your[XP Helper](Tools.md#xp-helper) VM
- Partition and quick-format the disk via [Disk Management](Techniques.md#disk-management)
- Copy your V2Is to the Holder disk

If you're having trouble mounting a network drive or accessing a Shared Folder from your XP Helper,
you may need to install [VirtualBox Guest Additions](Tools.md#virtualbox).
Once copied onto your Holder, your V2Is are on mountable VMDK -- eg. a normal drive -- and Ghost will have no trouble accessing it.


## Repairing

Booting up your old XP in a new virtual environment would completely violate all of its driver expectations.
You'll first want to run the 'Repair Installation' process on it.

For a more in-depth coverage of what's mentioned here

- Michael Stevens wrote an **excellent guide** to [XP Repair Installation](http://www.michaelstevenstech.com/XPrepairinstall.htm)
- This [how-to article](http://www.geekstogo.com/forum/topic/138-how-to-repair-windows-xp/) from GeeksToGo has some lovely screen-caps

### boot.ini

The `boot.ini` file on the active [partition](Techniques.md#partitioning) references the Windows XP installation to boot.
NeoSmart provides an [excellent overview](https://neosmart.net/wiki/rebuilding-boot-ini-file/) to this often misunderstood creature.

Microsoft offers a nice [guide to editing](http://support.microsoft.com/en-us/kb/289022) the file,
but most of the instructions assume you already have XP up and running -- as you would in the case of your [XP Helper](Tools.md#xp-helper).

While you're still in the restoration process, you'll be using the [Edit boot.ini](Techniques.md#norton-support-utilities) utility.
Nothing special -- it's just Notepad -- but it helps because the MSDOS shells you have available to you do not provide a text editor.
And all you'll need to do is save minor changes.

You can completely re-build the `boot.ini` from the [Recovery Console](Techniques.md#xp-recovery-console) by using `bootcfg`

```
bootcfg /rebuild
```

You'll definitely want to make a backup of it first.

### XP Repair Installation

Boot from your Windows XP Setup disk

1. At the first Setup prompt, press **ENTER**

   You do *not* want to use the [Recovery Console](Techniques.md#xp-recovery-console) here.

2. Setup will scan for previous XP installations.

   If you've aligned all the stars correctly &dagger;, you will see a screen displaying the previous install and offering:
   "To repair the selected Windows XP Installation, press R".
   And *damn*, you've earned it -- press that **R**

3. Setup will copy over some files, then reboot into guided re-install.

   This is a great time to kick back, have a beverage, and giggle quietly as you read Microsoft's self-promotional interstitials.
   The Future, it's just gonna be so *bright* ...

   It'll take a lot less than the 39 minutes Setup advertises.
   Mid-way through, you'll be prompted to enter an XP license key.
   At the end, you'll be asked about Activation and Registration.

And then you'll be shown your restored Login screen.
It'll be just like old times &Dagger;.

> &dagger;&nbsp; Assuming that the Setup process locates a version of XP that it believes it can repair ...

If it *doesn't* locate one, Setup will take only allow you to install XP from scratch.
Sure, it'll warn you that you've overwriting another operating system, but it won't recognize it as being itself.
Regrettably, self-reflection is not one of XP's stronger points.

If you run into problems, please consult the notes in the [Inspiron Example](Example-Inspiron.md#repair-detection).

> &Dagger;&nbsp; Well, not exactly like old times ...

Hooray, you have a fresh set of drivers that match your virtualized hardware!

But you're also running the way ancient(er) version of XP burned on that ISO.
You'll want to re-install all the [Windows Updates](Techniques.md#windows-updates).



<!--
!!!

within WinXP, give a VMDK a Volume / Drive Letter
  - http://www.theirishpenguin.com/2009/07/22/howto-add-a-secondary-hard-drive-for-windows-via-virtualbox-2-1-4-ose.html
  - 'Administrative Tools > Computer Management > Disk Management' + Create Partition

-->

### Windows Updates

<!--
!!!

Spin it up, and pull down all available patches &amp; updates,
with the possible exception of [Windows Genuine Advantage Notification]().

- http://superuser.com/questions/351937/how-do-i-force-windows-to-check-for-updates
- http://support.microsoft.com/kb/949104 = SP3
- http://support.microsoft.com/fixit/
- http://support.microsoft.com/kb/943144/en + install Windows Update Agent
- http://windowsupdate.microsoft.com/windowsupdate/v6/default.aspx?ln=en-us + install Windows Updater
- and just keep running Windows Updater
- skip
  - Windows Genuine Advantage Notification
-->
