# Example:  The Inspiron

This was an old Dell that wasn't a Dell anymore.
In its later life, it has become an OEM box running a standard install of XP SP2 which booted from a custom partition.
*Brilliant,* I say -- a lovely [parasitic relationship](http://www.iflscience.com/plants-and-animals/parasites-turn-their-hosts-zombies).
Yes, this are the crazy tricks we used to play in the old days.

Due to this trickery, it's safest to *exactly match* the original partitioning --
[much like](Techniques.md#initial-vm-setup) the CPU and RAM allocations.


## Partitions

The original hardware partitions were as follows

```
| Partition | Type | Boot | Start Cyl | Head | Sector | End Cyl | Head | Sector | Sectors Before | Total Sectors |
| 1         | DE   | 00   | 0         | 1    | 1      | 6       | 254  | 63     | 63             | 112392        |
| 2         | 07   | 80   | 7         | 2    | 60     | 1023    | 109  | 13     | 112640         | 20971520      |
| 3         | 0F   | 00   | 1023      | 0    | 1      | 1023    | 254  | 63     | 21093345       | 467170200     |
```

1. A hidden **Dell OEM partition** -- very small (55MB in this case, vs. the minimum 8MB) &nbsp;&dagger;
2. `H:` **the Brain Transplant** -- a bootable Primary partition with XP, and no data
3. `C:` **Storage** -- a Data-only Extended partition with one Logical drive

> &dagger;&nbsp; It is possible that I could have done away with the OEM partition entirely,
> but once I'd figured out a way to make it work as-was, I kept it.

The best strategy here was to create *one huge VMDK*.
The partitions were re-constructed using [diskpart](Techniques.md#diskpart) from the [Ghost](Tools.md#ghost) disk.

**First**, emulate the Dell OEM partition.

```
select disk 0
create partition extended size=55
```

While `diskpart` was still running, launch [Partition Table Operations](Techniques.md#norton-support-utilities)
and change the first partition's type to 'DE'.
The 'Set Type' list describes this as a "Dell Corporation diagnostic partition".

Make this change *right now*, because otherwise `diskpart` will eventually complain that "The maximum number of partitions has already been reached".
One Primary and one Extended partition [is your limit](Techniques.md#primary-extended-and-logical) --
and this edit makes the first partition an "OEM" which doesn't count towards that maximum.

**Then**, inform `diskpart` of those saved changes

```
rescan
```

**Finally**, allocate the rest of the partitions.

```
create partition primary size=10240
create partition extended
create partition logical
select partition 2
active
assign letter=H
select partition 3
assign letter=C
```

The third partition doesn't specify a 'size', so it consumes all available space.

**The results are**

```
list partition

Partition ###  Type              Size     Offset
-------------  ----------------  -------  -------
Partition 1    OEM                 55 MB    32 KB
Partition 2    Primary             10 GB    56 MB
Partition 0    Extended           130 GB    10 GB
Partition 3    Logical            130 GB    10 GB

list volume

Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
----------  ---  -----------  -----  ----------  -------  ---------  --------
Volume 0     H   RECOVERY     RAW    Partition     10 GB  Healthy
Volume 1     C                RAW    Partition    130 GB  Healthy
Volume 2                      RAW    Partition     55 MB  Healthy    Hidden
```

Schweet.

## Restore with Ghost

Both the boot and Data-only volumes were backed up as their own set of V2Is.
Simply [restore each volume](Techniques.md#restoring-with-ghost) to its respective partition.

After restoration, check the VOLUMEs using [diskpart](Techniques.md#diskpart)

```
select disk 0
list volumes

Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
----------  ---  -----------  -----  ----------  -------  ---------  --------
Volume 1     H   RECOVERY     NTFS   Partition     10 GB  Healthy    System
Volume 2     C                NTFS   Partition    223 GB  Healthy    Boot
```

Looks about right.

But there was still *one more issue* ...

## Repair Detection

Before the XP install can be [repaired](Techniques.md#xp-repair-installation),
the Setup process must detect the installation and *believe that it it can repair it*.
Trust me, when you can't get it to detect your installation, it can feel like a pretty arbitrary belief system.

Some common issues are

- The "version of Windows on your computer is newer than the version on the CD" [scenario](http://support.microsoft.com/en-us/kb/898594).

  This shouldn't be a problem if your Setup disk contains the SP2 release of XP.

- The "OEM installation (but not an OEM CD)" [scenario](http://superuser.com/questions/35290/xp-cd-doesnt-offer-repair-option).

  If you're virtualizing an old OEM workstation, such as Dell or Compaq,
  the vendor may have had a custom version of XP built for their own nefarious needs.
  You'll need to repair using the *OEM version* of the Setup disk.
  Hopefully it's in an envelope somewhere -- or you could [fake one](http://www.geekstogo.com/forum/topic/296146-no-repair-option-when-repairing-windows-xp/).
  I've heard rumour that some vendors were kind enough to bake in an XP program that could burn just such a disk for you.
  For the Inspiron, the XP license key was printed on a silver sticker stuck to the computer case.

Plus stranger scenarios -- perhanps indistinguishable from conspiracy theories -- such as

- ["corrupt or missing registry hives"](http://forums.techguy.org/windows-xp/1030320-windows-xp-install-repair-option.html), or
- ["executing PnP (Plug and Play) and ACPI routines"](http://www.geekstogo.com/forum/topic/296146-no-repair-option-when-repairing-windows-xp/).

Ultimately, if you can't get the Repair Installation process to work,
check out Microsoft's [thorough article](http://support.microsoft.com/en-us/kb/978788) on re-installing XP from scratch,
including some manual backup steps that could serve as your fallback option.

Fortunately, *none of those* were the issue in this case ...

## Confirm the Boot Disk

In order to repair the version of XP that the old hardware had been booting into,
the active partiton's [`boot.ini` file](Techniques.md#bootini) must reference actually it.
Somehow, even after [restoration](#restore-with-ghost) of up-to-date V2I files, that didn't happen.

Everything finally worked once I had

- Established the **active partition** using [diskpart](Techniques.md#diskpart)

  ```
  select disk 0
  select partition 2
  active
  ```

  Which we already did [above](#partitions).

- Referenced the bootable XP installation within the **boot.ini file**

  Your restored Windows XP install will already have a [`boot.ini` file](Techniques.md#bootini).
  Use the **Edit boot.ini** tool to ensure that it points at the active disk + partition combination.
  You may want to set aside a backup copy.

  In a typical installation, XP is installed on the boot parition,
  so `boot.ini` references the active disk + partition indexes as you used in `diskpart`,
  In this case, that would be 'multi(0)disk(0)rdisk(0)partition(2)\Windows'.
  And in fact, that's why my restored `boot.ini` contained.

  However, in inspecting the original computer, the *actual* configuration was to run Windows XP off of the *Data partition*.
  In my case, that was 'multi(0)disk(0)rdisk(0)partition(3)\WINDOWS', capitalization and all.
  ```
  [boot loader]
  timeout=1
  default=multi(0)disk(0)rdisk(0)partition(3)\WINDOWS
  [operating systems]
  multi(0)disk(0)rdisk(0)partition(3)\WINDOWS="Microsoft Windows XP Professional" /noexecute=optin /fastdetect
  ```

The XP Setup disk can now detect it for a [Repair Installation](Techniques.md#xp-recovery-installation) process.
