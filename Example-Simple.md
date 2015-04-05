# Example:  Simple Basic

My simplest restoration was a workstation had three volumes partitioned on a single physical disk.
When virtualizing it, I chose to create a separate [VMDK](Tools.md#virtual-disks) for each volume.
I could also have created one massive [partitioned](Techniques.md#partitioning) VMDK file.

The [initial VM](Techniques.md#initial-vm-setup) should have an identical CPU and RAM allocation as the hardware.

Then all you need to do is

1. [Restore each volume](Techniques.md#restoring-with-ghost) using Ghost
2. [Repair your installation](Techniques.md#xp-repair-installation) of Windows XP,
   since most of your device drivers are invalid.

And yeah, it's *that simple*.

Until you read deeper into the hyperlinks, of course ...
