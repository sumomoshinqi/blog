---
layout: post
title: "New partition set up on Linux"
date: 2015-08-09 15:54:16 +0800
comments: true
categories: ['Linux']
---
This article demonstrates how to set up new disk partition and make it auto-mounted on boot via Linux CLI. Instance here is a Ubuntu 12.04 remote server which just has a new disk attached to. 
<!--more--> 

###1. Find all disks on the computer

```sh
    sudo fdisk -l
```

For me, it looks like 

```text
    administrator@626bdbc5:~$ sudo fdisk -l
    [sudo] password for administrator: 
    
    Disk /dev/sda: 32.2 GB, 32212254720 bytes
    255 heads, 63 sectors/track, 3916 cylinders, total 62914560 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00028d90
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048      499711      248832   83  Linux
    /dev/sda2          501758    41940991    20719617    5  Extended
    /dev/sda5          501760    41940991    20719616   8e  Linux LVM
    
    Disk /dev/mapper/ubuntu-var: 3997 MB, 3997171712 bytes
    255 heads, 63 sectors/track, 485 cylinders, total 7806976 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/mapper/ubuntu-var doesn't contain a valid partition table
    
    Disk /dev/mapper/ubuntu-swap: 1996 MB, 1996488704 bytes
    255 heads, 63 sectors/track, 242 cylinders, total 3899392 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/mapper/ubuntu-swap doesn't contain a valid partition table
    
    Disk /dev/mapper/ubuntu-root: 15.2 GB, 15221129216 bytes
    255 heads, 63 sectors/track, 1850 cylinders, total 29728768 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/mapper/ubuntu-root doesn't contain a valid partition table
    
    Disk /dev/sdb: 107.4 GB, 107374182400 bytes
    255 heads, 63 sectors/track, 13054 cylinders, total 209715200 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/sdb doesn't contain a valid partition table
```

###2. Use fdisk to create the partition
Here, for example, I'm creating a new partition using /dev/sdb

```sh
    sudo fdisk /dev/sdb
```

Because sdb is a new disk, you'll see

```text
    Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
    Building a new DOS disklabel with disk identifier 0x5f40a261.
    Changes will remain in memory only, until you decide to write them.
    After that, of course, the previous content won't be recoverable.

    Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)
```

Create a new partition following instructions.

```text
Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): n     # Here we type n to create a new partition
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p       # Choose primary
Partition number (1-4, default 1):      # Use default
Using default value 1
First sector (2048-209715199, default 2048):    # Use default
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199):  # Use default
Using default value 209715199

Command (m for help): w     # Write back
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

###3. Format the partition

```sh
    sudo mkfs.ext4 /dev/sdb1 # Format new partition with ext4 file system
```

You'll see output like

```text
mke2fs 1.42.9 (4-Feb-2014)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26214144 blocks
1310707 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done  
```

###4. Let the new partition mounted automatically on boot
Edit system configuration.

```sh
    sudo vim /etc/fstab
```

Add a new line for the partition just created.

```text
# /etc/fstab: static file system information.
#
# Use 'blkid -o value -s UUID' to print the universally unique identifier
# for a device; this may be used with UUID= as a more robust way to name
# devices that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
proc            /proc           proc    nodev,noexec,nosuid 0       0
/dev/mapper/ubuntu-root /               ext4    errors=remount-ro 0       1
# /boot was on /dev/sda1 during installation
UUID=4936e7c7-515b-4bbd-b9e4-7b6dbbd5239a /boot           ext2    defaults        0       2
/dev/mapper/ubuntu-var /var            ext4    defaults        0       2
/dev/mapper/ubuntu-swap none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
# Add new entry for sdb1
/dev/sdb1       /data   ext4    defaults        0       0
```

###5. Reboot and Check result

```sh
    sudo reboot
```

Check result
```sh
    df
```

Here sdb1 is successfully auto-mounted.

```
Filesystem              1K-blocks    Used Available Use% Mounted on
/dev/mapper/ubuntu-root  14499760 1824804  11915356  14% /
none                            4       0         4   0% /sys/fs/cgroup
udev                      4071712       4   4071708   1% /dev
tmpfs                      817024     812    816212   1% /run
none                         5120       4      5116   1% /run/lock
none                      4085116       0   4085116   0% /run/shm
none                       102400       0    102400   0% /run/user
/dev/mapper/ubuntu-var    3776568  866076   2698936  25% /var
/dev/sda1                  233191   41801    178949  19% /boot
/dev/sdb1               103080224   61044  97759968   1% /data
```