[TOC]

# Chapter 1. The Big Picture

## Levels and Layers of Abstraction in a Linux System 

A Linux system has three main levels: ***hardware***, ***kernel***, ***user space/user process***

***User mode*** restricts access to a (usually quite small) subset of memory called ***user space*** and safe CPU operations 
***Kernel mode*** has unrestricted access to the processor and main memory 

## The Kernel 

kernel is in charge of managing tasks in four general system areas: 

1.  **processes** (**context switching**), determining which processes are allowed to use the CPU. <font color=red>Kernel
   runs between process time slices during a **context switch** </font>
2. **memory**: kernel needs to keep track of all memory. Modern CPUs include a **memory management unit (MMU)** that translates the **virtual memory** location from the ***process*** point of view into an actual **physical memory** location in the machine.
3. **device drivers**: kernel acts as an interface between hardware and processes. A device is typically accessible only in ***kernel mode***. Device drivers rarely have the same programming interface and have traditionally been part of the kernel, to present a **uniform interface**. **Pseudodevices** <font color=red>(e.g. /dev/random)</font> implemented purely in software.
4. **system calls and support**: Processes normally use system calls to communicate with the kernel. Other than ***init***, all new user processes on a Linux system start as a result of **fork()** and then **exec(cmd)** to load and start `cmd`, replacing the current process 

## Users 

As powerful as the ***root*** user is, it still runs in the operating system’s ***user mode***, not ***kernel mode*** 

# Chapter 2. Basic Command and Directory Hierarchy
## The Bourne Shell: /bin/sh 

A shell is a program that runs commands, also serves as a small programming environment for shell scripts  

***Bourne shell*** (`/bin/sh`) and ***“Bourne-again” shell*** (`/bin/bash`) 

## Basic Commands 

`ls -F` to display file type information 

`echo` command is very useful for finding expansions of **shell globs** (“wildcards” such as `*`) and variables (such as `$HOME`) 

`cd` command is a **shell built-in** and wouldn’t work as a **separate process** because if it were to run as a subprocess, it could not (normally) change its parent’s current working directory 

### Shell builtin

use `compgen -b` or `man bash` to find all  **shell built-in**

Run `kill -l` to get a mapping of signal numbers to names. 

## Environment and Shell Variables 

The shell can store temporary variables, called **shell variables** 

An **environment variable** is like a **shell variable**, but not specific to the shell 
Child processes inherit **environment variables** from their parent 

## Shell Input and Output 

```bash
# The number 2 specifies the stream ID that the shell modifies. Stream ID 1 is standard output (the default), and 2 is standard error.
$ ls /fffffffff > f 2> e
$ ls /fffffffff > f 2>&1
```

## Job Control 

send **TSTP** with `CTRL-Z` and **CONT** signals 

***fg*** (bring to foreground) or ***bg*** (move to background; see the next section 

**background** process which tries to read from the **standard input** will **freeze or terminate** 

The best way to make sure that a background process doesn’t bother you is to **redirect its output (and possibly input)  **

## File Modes and Permissions 

Some executable files such as ***passwd*** have an `s` in the owner permissions listing instead of an x. This indicates that the executable is `setuid`, meaning that the program runs as if the file owner is the owner instead of you.

## Running Commands as the Superuser 

Use the **visudo** command to edit `/etc/sudoers`. This command checks for file syntax errors after you save the file. 

# Chapter 3. Device

## Device Files 

the kernel presents many of the **device I/O interfaces** to **user processes** as files called **device nodes**.  

The **mode of device files** contains **b (block)**, **c (character)**, **p (pipe)**, or **s (socket)**

Not all devices have **device files** such as **network interfaces** and there is a limit to what you can do with a file interface

The kernel uses **major and minor device numbers** to identify the device. Similar devices usually have the **same major** 

## The sysfs Device Path 

While the traditional Unix `/dev` directory is a convenient and very simplistic scheme, **linux kernel** offers the **sysfs interface** through a system of files and directories in `/sys/devices` to view information and manage the device. 

Use the **udevadm** command as follows to show the path and several other interesting attributes: 

## dd and Devices 

**dd** is extremely useful when working with **block and character devices** to read from an input file or stream and write to an output file or stream 

```bash
# an old IBM Job Control Language (JCL) style (name=value)
dd if=/dev/zero of=new_file bs=1024 count=1 ibs=size, obs=size skip=num
```

## Device Name 

It can sometimes be difficult to find the name of a device and the kernel might not have a device file for
your hardware 

1. Most reliable: Query ***udevd*** using **udevadm** 
2. Look for the device in the ***/sys*** directory. 
3. Guess the name from the output of the `journalctl -k` command 
4. For a disk device, use **mount** command 
5. Run `cat /proc/devices` to see the **block and character devices** for which system currently has drivers 

The most common Linux devices and their naming conventions is as follows

### Hard Disks: /dev/sd* 

***sd*** stands for **Small Computer System Interface (SCSI) disk**.

Linux assigns devices to device files in the order in which its drivers encounter the devices 

many Linux systems use the **Universally Unique Identifier (UUID**) and/or the **Logical Volume Manager (LVM) stable disk device mapping** to identify devices

### Virtual Disks: /dev/xvd\*, /dev/vd\* 

disk devices for virtual machines in **Xen virtualization system** 

### Non-Volatile Memory Devices: /dev/nvme* 

**Non-Volatile Memory Express (NVMe) interface **

### Device Mapper: /dev/dm-\*, /dev/mapper/\* 

***LVM***, which uses a **kernel system** called the **device mappe**, is a level up from disks and other direct block storage 

### CD and DVD Drives: /dev/sr* 

Linux recognizes most **optical storage drives** as the **SCSI devices** ***/dev/sr0*** which is read only, For the write and rewrite capabilities of optical devices, use the **“generic” SCSI devices** such as ***/dev/sg0***. 

### PATA Hard Disks: /dev/hd* 

an older type of storage bus **PATA (Parallel ATA)** or **SATA drive** running in a compatibility mode 

### Terminals: /dev/tty\*, /dev/pts/\*, and /dev/tty 

Most terminals are **pseudo-terminal devices**, **emulated terminals** such as the **shell terminal window**  

Two common terminal devices are `/dev/tty1` (the first **virtual console**) and `/dev/pts/0` (the first **pseudo-terminal device**). 
The `/dev/pts` directory itself is a dedicated filesystem.
The `/dev/tty` device is the controlling terminal of the current process 

```bash
# try to force the system to change consoles to tty1
chvt 1
```

### Serial Ports: /dev/ttyS\*, /dev/ttyUSB\*, /dev/ttyACM\* 

**true terminal devices** such as older **RS-232** type and similar serial ports which you can’t do much on the command line, but can use the `screen` command to connect to

The port known as **COM1** on Windows is ***/dev/ttyS0***;
**Plug-in USB serial adapters** show up with USB and ACM with the names ***/dev/ttyUSB0***, ***/dev/ttyACM0***

### Parallel Ports: /dev/lp0 and /dev/lp1 

**unidirectional parallel port devices** replaced by USB and networks
***/dev/lp0*** and ***/dev/lp1*** correspond to **LPT1:** and **LPT2:** in Windows

The **bidirectional parallel ports** are ***/dev/parport0*** and ***/dev/parport1***.

### Audio Devices: /dev/snd/*, /dev/dsp, /dev/audio, and More 

Linux has two sets of audio devices.
**Advanced Linux Sound Architecture (ALSA) system interface** and the older **Open Sound System (OSS)**. The **ALSA devices** are in the ***/dev/snd*** directory 

## udev 

device files are created by **devtmpfs(mounted at /dev)** and **udev** 

On a rare occasion, you might need to use **mknod** command creates own device.  

```bash
# specifies a block device with a major number 8 and a minor number 1
mknod /dev/sda1 b 8 1
```

In older versions of Unix and Linux, the `/dev` directory is **static system** that must run **MAKEDEV** program in order to create new devices every time system upgraded 

This is stupid and the first attempt to fix it was **devfs**, a **kernel-space implementation** of `/dev` that contained all of the devices that the current kernel supported with a number of limitations, which led to the development of **udev** and **devtmpfs**. 

Unnecessary complexity in the kernel is dangerous because you can too easily introduce system instability  so **udevd** is **user-space process** which **Linux kernel** can send notifications to once detecting a new device 

But there is a problem with this approach—**device files** are necessary early in the **boot procedure**, so **udevd** must also start early and very quickly so that the rest of the system doesn’t get held up waiting
for it. and cannot depend on any devices that it is supposed to create.

The **udevd daemon** operates as follows:
1. The kernel sends ***udevd*** a notification event, called a **uevent**, through an **internal network link**.
2. ***udevd*** loads all of the attributes in the **uevent**.
3. ***udevd*** parses its rule files, in the `/lib/udev/rules.d`(**defaults**) and `/etc/udev/rules.d` directories, filters and updates the **uevent** based on those rules, and takes actions or sets more attributes accordingly. 

### devtmpfs 

The **devtmpfs filesystem**, similar to the older **devfs** support, but simplified, was developed in response to the problem of **device availability during boost**

### udevadm 

The **udevadm** program is an administration tool for ***udevd*** with the ability to search for and explore system devices and monitor **uevents** as ***udevd*** receives them from the kernel and reload ***udevd*** rules

```bash
$ udevadm info --query=all --name=/dev/sda

# P: sysfs device path
P: /devices/pci0000:00/0000:00:1f.2/host0/target0:0:0/0:0:0:0/block/sda
# N: device node (that is, the name given to the file in /dev)
N: sda
# S: symbolic link to the device node that udevd placed in /dev according to rules
S: disk/by-id/ata-WDC_WD3200AAJS-22L7A0_WD-WMAV2FU80671
S: disk/by-id/scsi-SATA_WDC_WD3200AAJS-_WD-WMAV2FU80671
S: disk/by-id/wwn-0x50014ee057faef84
S: disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0
# E: additional device information extracted in the udevd rules
E: DEVLINKS=/dev/disk/by-id/ata-WDC_WD3200AAJS-22L7A0_WD-WMAV2FU80671 /dev/disk/by-id/scsi-SATA_WDC_WD3200AAJS-_WD-WMAV2FU80671 /dev/disk/by-id/wwn-0x50014ee057faef84 /dev/disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0
E: DEVNAME=/dev/sda
E: DEVPATH=/devices/pci0000:00/0000:00:1f.2/host0/target0:0:0/0:0:0:0/block/sda
E: DEVTYPE=disk
E: ID_ATA=1
E: ID_ATA_DOWNLOAD_MICROCODE=1
E: ID_ATA_FEATURE_SET_AAM=1
--snip--
```

## In-Depth: SCSI and the Linux Kernel 

**SATA disks** also appear on your system as **SCSI devices**, through a **translation layer** in the **libata library**

Some SATA controllers (especially **high-performance RAID controllers**) perform this translation in hardware. 

```bash
$ lsscsi
# [SCSI host adapter number : SCSI bus number : device SCSI ID : LUN (logical unit number, a further subdivision of a device)]
[0:0:0:0] disk ATA WDC WD3200AAJS-2 01.0 /dev/sda
[1:0:0:0] cd/dvd Slimtype DVD A DS8A5SH XA15 /dev/sr0
[2:0:0:0] disk USB2.0 CardReader CF 0100 /dev/sdb
[2:0:0:1] disk USB2.0 CardReader SM XD 0100 /dev/sdc
[2:0:0:2] disk USB2.0 CardReader MS 0100 /dev/sdd
[2:0:0:3] disk USB2.0 CardReader SD 0100 /dev/sde
[3:0:0:0] disk FLASH Drive UT_USB20 0.00 /dev/sdf
```

the driver and interface hierarchy of **SCSI subsystem inside the kernel** include three layers 

1. The top layer handles operations for a **class of device** (***sd***, ***sr***). For example, translate between requests from the kernel device interface and class-specific commands in the **SCSI protocol**
2. The middle layer moderates and routes the **SCSI messages** between the top and bottom layers, and keeps track of all of the **SCSI buses and devices** attached to the system  
3. The bottom layer handles hardware-specific actions, **send/extract SCSI protocol messages to/from specific host adapters or hardware** because <font color=red>different kinds of **host adapters** have varying procedures for sending the same messages although **SCSI messages** are uniform for a device class (such as the disk class),</font>

for any given **device file**, the kernel (nearly always) uses one top-layer driver and one lower-layer driver. 

### USB Storage and SCSI 

USB is quite similar to SCSI—it has **device classes, buses, and host controllers**. Therefore, Linux kernel includes a **three-layer USB subsystem** that closely resembles the **SCSI subsystem** 

the top level of USB storage driver translates **SCSI commands** from **lower-layer SCSI bridge driver** into **USB messages**

### SCSI and ATA 

The Linux kernel uses part of a **libata library** to reconcile **SATA (and ATA) drives** with the **SCSI subsystem** 

### Generic SCSI Devices 

A **user-space process** normally communicates with the **SCSI subsystem** through **kernel service** that sits on **top layer of an SCSI device class driver** (like ***sd*** or ***sr***), in which case user processes never need to know anything about SCSI devices or commands. 

However, user processes can **bypass device class drivers** and directly give **SCSI protocol commands** to devices through **generic SCSI devices**. 

But do user processes really read data this way? Normally, the answer is no, not directly.
There are more layers on top of the block devices and even more points of access for hard disks,  

# Chapter 4. Disks and Filesystems

**Partitions** are subdivisions of the whole disk, defined on a small area of the disk called a **partition table** (also called a **disk label**). 

The kernel presents each partition as a block device, just as it would an entire disk.

The **Linux Logical Volume Manager (LVM)** adds more flexibility to traditional disk devices and partitions, 

## Partitioning Disk Devices 

The traditional **partition table**, **Master Boot Record (MBR)**, has many limitations. Most newer systems
use the **Globally Unique Identifier Partition Table (GPT)**. 

<font color=red>Linux partitioning tools</font>:

1. **parted (“partition editor”)** A text-based tool that supports both ***MBR*** and ***GPT***. Partitions are created, modified, and removed as you issue the commands 
2. **gparted** A graphical version of **parted**.
3. **fdisk** The traditional text-based **Linux disk partitioning tool**. Recent versions support the ***MBR***, ***GPT***, and many other kinds of **partition tables**, but older versions were limited to ***MBR*** support. Partition table changes only when you exit the program 

view partition table with

1.  `fdisk -l` show exact **MBR system ID**, a number identifying the partition type; 
2.  `parted -l` attempts to be more informative and show **filesystem type** 

There are a few ways to see the partition changes:

1. Use command `udevadm monitor -kernel` to watch the kernel event changes. 
2. Check **/proc/partitions** for full partition information.
3. Check **/sys/block/device/** for altered partition system interfaces or **/dev** for altered partition devices. 

A hard disk geometry is called **CHS, for cylinder-head-sector **

While **Logical Block Addressing (LBA)** to address a location on the disk by a **block number**  is a much more straightforward interface, but remnants of ***CHS*** remain. For example, the **MBR partition table** contains CHS information as well as LBA equivalents 

One of the most significant factors affecting the performance of SSDs is partition alignment where data must be read in chunks (called ***pages***, not to be confused with **virtual memory pages**) 

## Filesystems 

The filesystem, a layer up from the partition, is a form of **database** of files and directories interacting with user space. 

Filesystems are traditionally implemented in the kernel, but **File System in User Space (FUSE)** feature allows **user-space filesystems** in Linux 

Much as the **SCSI subsystem** standardizes communication between different device types and kernel control commands, **Virtual File System (VFS) abstraction layer** ensures that all filesystem implementations support a standard interface

### Filesystem Types  

**The Fourth Extended filesystem (ext4)**: native to Linux. **The Third Extended filesystem (ext3)** added a journal feature (a small cache outside the normal filesystem data structure) to enhance data integrity and hasten booting. The **ext4 filesystem** is an incremental improvement and supports larger files than **ext2** or **ext3**  

**Btrfs, or B-tree filesystem (btrfs)**, is a newer filesystem native to Linux designed to **scale beyond** the capabilities of ext4  

**FAT filesystems (msdos, vfat, exfat)** pertain to Microsoft systems  

**XFS** is a high-performance filesystem used by default by some distributions  

**HFS+ (hfsplus)** is an **Apple standard** used on most Macintosh systems.

**ISO 9660 (iso9660)** is a **CD-ROM standard**  

### Creating a Filesystem  

**mkfs** is **only a frontend** for a **series of filesystem creation programs**, **mkfs.fs**, where fs is a filesystem type So ` mkfs -t ext4` in turn runs **mkfs.ext4**  

### Mounting a Filesystem  

```bash
$ mount
/dev/sda1 on / type ext4 (rw,errors=remount-ro) # mount option
proc on /proc type proc (rw,noexec,nosuid,nodev)
sysfs on /sys type sysfs (rw,noexec,nosuid,nodev)
fusectl on /sys/fs/fuse/connections type fusectl (rw)
debugfs on /sys/kernel/debug type debugfs (rw)
securityfs on /sys/kernel/security type securityfs (rw)
udev on /dev type devtmpfs (rw,mode=0755)
devpts on /dev/pts type devpts (rw,noexec,nosuid,gid=5,mode=0620)
tmpfs on /run type tmpfs (rw,noexec,nosuid,size=10%,mode=0755)
--snip--
```



```bash
mount -t type device mountpoint
# don’t need to supply the -t type option except to distinguish between two similar types, such as the various FAT-style filesystem
mount -t ext4 /dev/sdf2 /home/extra
```

The Linux native partitions all have standard **universally unique identifier (UUID)**, but the FAT partition doesn’t. You can reference the **FAT partition** with its **FAT volume serial number**

<font color=red>**UUID** is part of **filesystem** and stored in **superblock**</font>

```bash
# blkid
/dev/sdf2: UUID="b600fe63-d2e9-461c-a5cd-d3b373a5e1d2" TYPE="ext4"
/dev/sda1: UUID="17f12d53-c3d7-4ab3-943e-a0a72366c9fa" TYPE="ext4"
PARTUUID="c9a5ebb0-01"
/dev/sda5: UUID="b600fe63-d2e9-461c-a5cd-d3b373a5e1d2" TYPE="swap"
PARTUUID="c9a5ebb0-05"
# 4859-EFEA is FAT volume serial number
/dev/sde1: UUID="4859-EFEA" TYPE="vfat"
```

You can change the UUID of a filesystem by **tune2fs** if necessary   

### Disk Buffering, Caching, and Filesystems  

When you unmount a filesystem with ***umount***, the kernel automatically synchronizes with the disk, writing the **cache in RAM** to the disk. If for some reason you can’t unmount a filesystem before you turn off the system, be sure to run ***sync*** to force the kernel to sync cache first.  

### The /etc/fstab Filesystem Table  

To mount filesystems at boot time  

### Checking and Repairing Filesystems: fsck

As with the **mkfs**, there’s a different version of **fsck** for each filesystem type that Linux supports, such as **e2fsck** for **Extended filesystem series (ext2/ext3/ext4)**

<font color=red>Never use fsck on a mounted filesystem </font>

When reconnecting **loose inodes**(files don’t appear to have a name), **fsck** places the file in the **lost+found directory** in the filesystem, with a number as the filename  

### Special-Purpose Filesystems  

**proc**: Mounted on `/proc`. Abbreviation for **process**, Each numbered directory inside **/proc** refers to the ID of a current process on the system. The **Linux proc filesystem** includes a great deal of additional **kernel and hardware information** in files like `/proc/cpuinfo`. <font color=red>But the **kernel design guidelines** recommend moving information unrelated to processes out of `/proc` and into `/sys`, so system information in `/proc` might **not be the most current interface**.  </font>

**sysfs** Mounted on `/sys`  

**tmpfs** Mounted on `/run` and other locations. With **tmpfs**, you can use your **physical memory and swap space** as **temporary storage**  

**squashfs** A type of **read-only filesystem** where content is stored in a compressed format and extracted on demand through a **loopback device**.  

**overlay** A filesystem that merges directories into a composite. Containers often use overlay filesystems  

## Swap Space  

To remove a **swap partition or file** from the **kernel’s active pool**, use the **swapoff** command  

High-performance servers should never dip into swap space and should avoid disk access if at all possible.  

### Using a Disk Partition as Swap Space  

1. Make sure the partition is empty.

2. Run `mkswap dev`, where `dev` is the partition’s device. 

3. Execute `swapon dev` to register the space with the kernel  

### Using a File as Swap Space  

```bash
dd if=/dev/zero of=swap_file bs=1024k count=num_mb
mkswap swap_file
swapon swap_file
```

## The Logical Volume Manager  

The **LVM** adds another layer——**device mapper** by which kernel handles the work of routing a request for a location on a logical volume’s block device to the true location on an actual device ——  between the **physical block devices** and the **filesystem**. Many features, such as **software RAID and encrypted disks**, are built on the **device mapper**.  

Before it does anything, an **LVM utility** must first scan the available **block devices** to look for **PVs** as follows:

1. Find all of the **PVs** on the system.
2. Find all of the **volume groups** that the PVs belong to by UUID (this information is contained in the PVs).

3. Verify that everything is complete (that is, all necessary PVs that belong to the **volume group** are present).

4. Find all of the **logical volumes** in the **volume groups**.
5. Figure out the scheme for mapping data from the **PVs** to the **logical volumes**.  
6. Communicate with the **kernel’s device mapper driver** in order to initialize the block devices for the **logical volumes** and load their **mapping tables** with the **ioctl system call** on the `/dev/mapper/control` device file  

LVM has a number of user-space tools for managing volumes and **volume groups**. Most of these are based around the **lvm** command, an interactive general-purpose tool. There are individual commands (which are just **symbolic links to lvm**) to perform specific tasks.   

**vgs** and **vgdisplay** shows the **volume groups (virtual disk)** currently configured on the system  

**lvs** and **lvdisplay** shows **logical volumes (virtual block device)**, you can create **partitions and filesystems** on **logical volumes**

**pvs** and **pvdisplay** shows **physical volume (physical block device)**. A **physical extent (abbreviated as PE)** is a piece of a **physical volume**, much like a block, but on a much larger scale. **PVs** are just **block devices**.   Each **PV** contains **physical volume metadata**, extensive information about its **volume group** and its **logical volumes**.  



**vgcreate** can designate one of the partitions as a PV and assign it to a new **volume group**. **PVs** can also be created in a separate step with the **pvcreate** command on **any block device**, even entire-disk device. **vgextend** command add PVs to the **volume group**

**lvcreate** command allocates a new **logical volume** in a **volume group**. **lvremove** removes it

To resize **logical volume**, you need to resize both it with **lvresize** and the filesystem inside it with **fsadm** command. **fsadm** is just a script that knows how to transform its arguments into the ones used by **filesystem-specific tools** like **resize2fs**.   

## Disks and User Space  

The **kernel** handles **raw block I/O** from the devices

In normal use, **user space** uses only the **filesystem support** that the kernel provides on **top of the block I/O**. **User-space tools** can also use the block I/O through **device files**, but, typically **only for initializing operations**, such as **partitioning, filesystem creation, and swap space creation**  

## Inside a Traditional Filesystem  

A traditional Unix filesystem has two primary components: 

1. a pool of **data blocks**
2. a **database system** that manages the data pool. The database contains inode data table, a **block bitmap** corresponding to whether block in the data pool is in use and so on. 

An **inode** is a set of data that describes a particular file,   

There is one small exception in **link counts**. The **root inode** has a link in the **filesystem’s superblock**

Most **fsck** programs make new files in the **filesystem’s lost+found directory**  when the **inode table** data doesn’t match the **block allocation data in bitmap**  

**VFAT filesystem** designed for Windows doesn't support **inode numbers** or **link counts**, therefore, nor **hard link**.

# Chapter 5. How the Linux Kernel Boots

## Overview

A simplified view of the boot process looks like this:

1. The machine’s **BIOS or boot firmware** loads and runs a **boot loader**.
2. The **boot loader** finds the **kernel image on disk**, loads it into memory, and starts it.

3. The kernel initializes the **devices** and its **drivers**.
4. The kernel mounts the **root filesystem**.
5. The kernel starts a program called ***init*** with a process ID of 1. This point is the **user space** start  
6. ***init*** sets the rest of the system processes in motion.
7. At some point, ***init*** starts a process allowing you to log in, usually at the end or near the end of the boot sequence.  

### Startup Messages  

Traditional Unix systems produce many **diagnostic messages** upon boot that come first from the **kernel** and then from **processes and initialization procedures** that ***init*** starts. The **diagnostic messages** from **user-space startup procedure** will be captured by ***systemd*** and normally go to the console.  

To view the kernel’s boot and runtime diagnostic messages, use **journalctl** command. If you don’t have **systemd**, check for a logfile such as `/var/log/kern.log` or run the **dmesg** command to view the messages in the **kernel ring buffer**.  

### Kernel Initialization

Upon startup, the **Linux kernel initializes** in this general order:

1. CPU inspection
2. Memory inspection
3. Device bus discovery
4. Device discovery (may have problem of dependencies for **loadable kernel modules** but solved by **initial RAM filesystem (initrd)**)
5. Auxiliary kernel subsystem setup (networking and the like)
6. Root filesystem mount
7. User space start  

### Kernel Parameters  

When the **Linux kernel** starts, it receives a set of **text-based kernel parameters** which can be   looked at the `/proc/cmdline` file

```bash
$ cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.15.0-43-generic root=UUID=17f12d53-c3d7-4ab3-943e
-a0a72366c9fa ro quiet splash vt.handoff=1
```

One that is critical is the **root** parameter specifying location of the **root filesystem**;

The **ro parameter** instructs the kernel to mount the **root filesystem** in **read-only mode** upon **user space** start so that **fsck** can check the **root filesystem** safely before trying to do anything serious. After the check, the **bootup process** remounts the **root filesystem** in **read-write mode**.  

Upon encountering a parameter that it doesn’t understand, the Linux kernel saves that parameter and  later passes the parameter to ***init*** when performing the user space start.

***bootparam(7) manual page*** gives an overview of **basic boot parameters** and **kernel-params.txt** gives very specific parameters

## Boot Loaders  

**boot loader** program loads the kernel into memory from somewhere on a disk and then starts the kernel with a set of **kernel parameters**.   

To find **kernel** and **kernel parameters** in **filesystem**, a **boot loader** does need a driver to access the disk, but it’s not the same one that the kernel uses. On PCs, **boot loaders** use the traditional **Basic Input/ Output System (BIOS)** or the newer **Unified Extensible Firmware Interface (UEFI)** to access disks  

Contemporary **disk hardware** includes **firmware** allowing the **BIOS or UEFI** to access attached storage hardware via **Logical Block Addressing (LBA)**. **LBA** is a universal, simple way to access data from any disk, but its performance is poor. This isn’t a problem because **boot loaders** are often the only programs that must use this mode for disk access

To determine if your system uses a **BIOS** or **UEFI**

1. run **efibootmgr**
2. check to see that `/sys/firmware/efi` exists; if so, your system uses **UEFI**.    

Once access to the **disk’s raw data** has been resolved, the **boot loader** must do the work of locating the desired data on the **filesystem**. Most common **boot loaders** can read **partition tables** and have **built-in support** for **read-only access to filesystems**.

### Boot Loader Tasks  

A **Linux boot loader’s core functionality** includes the ability to do the following:

1. Select from **multiple kernels**.
2. Switch between sets of **kernel parameters**.
3. Allow the user to manually override and edit **kernel image names and parameters** (for example, to enter single-user mode).
4. Provide support for **booting other operating systems**  

### Boot Loader Overview
Here are the main boot loaders that you may encounter:

1. **GRUB** A near-universal standard on Linux systems, with **BIOS/MBR** and **UEFI** versions.
2. **LILO** One of the first Linux boot loaders. **ELILO** is a **UEFI** version.
3. **SYSLINUX** Can be configured to run from many different kinds of filesystems.
4. **LOADLIN Boots** a kernel from **MS-DOS**.
5. **systemd-boot** A simple **UEFI boot manager**.
6. **coreboot (formerly LinuxBIOS)** A high-performance replacement for the PC BIOS that can include a kernel.
7. **Linux Kernel EFISTUB** A kernel plug-in for loading the kernel directly from a **EFI/UEFI System Partition (ESP)**.
8. **efilinux** A **UEFI boot loader** intended to serve as a model and reference for other UEFI boot loaders.  

You can learn a lot about a boot loader by getting to a **boot prompt** where you can enter a kernel name and parameters. Unfortunately, **Linux distributions** heavily customize boot loader behavior and appearance  

## GRUB Introduction  

One of most important capabilities of **GRUB(Grand Unified Boot Loader)** is filesystem navigation that allows for easy kernel image and configuration selection  

To access the **GRUB menu**, press and hold `SHIFT` for **BIOS**, or`ESC` if your system has **UEFI**

**GRUB** borrows terminology from many sources including **Linux kernel** and **Unix shell commands**. GRUB has its own **insmod command** to dynamically load GRUB modules, and even **ls command** to list files.

There’s a **GRUB module** for **LVM** that is required to boot systems where the kernel resides on a **logical volume**.

**GRUB** has its own device-addressing scheme. For example, the first hard disk found is named `hd0`, followed by `hd1`  

The **GRUB configuration directory** `/boot/grub` or `/boot/grub2` contains the central configuration file, `grub.cfg`, an architecture-specific directory such as `i386-pc` containing **loadable modules** with a `.mod` suffix, and a few other items such as fonts and localization information. 

We won’t modify `grub.cfg` directly; instead, we’ll use the **grub-mkconfig command**   

## UEFI Secure Boot Problems  

When **secure boot feature** is active, this UEFI mechanism requires any boot loader to be digitally signed by a trusted authority in order to run.
Microsoft has required hardware vendors shipping Windows 8 and later with their systems to use secure boot  

This is disadvantages to **secure boot systems**, especially for someone experimenting with building their own **boot loaders**.   

## Chainloading Other Operating Systems
**UEFI** makes it relatively easy to support loading other operating systems because you can install **multiple boot loaders** in the **EFI partition**. The older **MBR style** doesn’t support this functionality  

**GRUB** can load and run a different **boot loader** on a specific partition on your disk; this is called **chainloading**.  

## Boot Loader Details  

**Boot loading schemes** of **PC boot mechanisms** have two main variations **MBR** and **UEFI**.  

### MBR Boot  

PC **BIOS** loads and executes a small area of 441 bytes of **MBR** after its **Power-On Self-Test (POST)**. Unfortunately, this space is inadequate to house almost any boot loader, the initial piece of code in the **MBR** does nothing other than load the rest of the **boot loader code** which are usually stuffed into the space between the **MBR** and the **first partition** on the disk, resulting in what is sometimes called a **multistage boot loader**.   

**BIOS** doesn’t work with a **GPT-partitioned disk** because the **GPT information** resides in the area after the **MBR**. (**GPT** leaves the traditional **MBR** alone for backward compatibility.)   

The solution of **GRUB** for **BIOS** to work with **GPT** is to create a small partition called a **BIOS boot partition** with a special **UUID (21686148-6449-6E6F-744E-656564454649)** to give the full boot loader code a place to reside.   

### UEFI Boot  

Due to the limitation of **traditional PC BIOS**, PC manufacturers and software companies develop a replacement called **Extensible Firmware Interface (EFI)**. The current standard is **Unified EFI (UEFI)**, which includes features such as a **built-in shell** and the ability to read **partition tables** and navigate **filesystems**. The **GPT partitioning scheme** is part of the **UEFI standard**.  

In **UEFI**, rather than **executable boot code** residing outside of a **filesystem** for **BIOS**, there’s always a **special VFAT filesystem** called the **EFI System Partition (ESP)**, which contains a directory named **EFI**. 

The **ESP** is usually mounted on Linux system at`/boot/efi`,
Each boot loader has its own identifier and a corresponding subdirectory, such as `efi/microsoft`, `efi/apple`, `efi/ubuntu`, or `efi/grub`. A boot loader file has a `.efi` extension and resides in one of these subdirectories, along with other supporting files  

However, you can’t just put old **boot loader code** written for the **BIOS interface** into the **ESP**. Instead, you must provide a **boot loader** written for **UEFI**.   

# Chapter 6. How User Space Starts

**User space** starts in roughly this order:

1. ***init***
2. Essential **low-level services**, such as ***udevd*** and ***syslogd***
3. **Network configuration**
4. **Mid- and high-level services** (***cron***, printing, and so on)
5. **Login prompts**, **GUIs**, and **high-level applications**, such as **web servers**  

## Introduction to init  

***init*** is a **user-space program**, and you’ll find it in `/sbin` along with many of the other **system binaries** whose main purpose is to start and stop the **essential service processes** on the system  

varieties of ***init***:  

1. **systemd**: the standard implementation of ***init***, has `/usr/lib/systemd` and `/etc/systemd` directories  
2. **System V init**: traditional **sequenced init** found on **Red Hat Enterprise Linux (RHEL)** prior to version 7.0, have an `/etc/inittab` file,  
3. **Debian 8. Upstart**: on **Ubuntu** distributions prior to version 15.04, have an `/etc/init` directory that contains several `.conf` files  
4. Other versions of ***init*** exist, especially on **embedded platforms**. For example, **Android** has its own ***init***, and a version called ***runit*** is popular on **lightweight systems**  

<font color=red>The **init(1) manual page** normally tells your system’s version of ***init***.</font>

**System V init** is basically a series of scripts that ***init*** runs, in sequence, one at a time. Each script usually starts one service or configures an individual piece of the system. This scheme suffers from some significant limitations which can be grouped into “**performance problems**” and “**system management
hassles**” as follows:

1. **Performance suffers** because two parts of the boot sequence cannot normally run at once.
2. **Managing a running system can be difficult**. Startup scripts are expected to start **service daemons**. To find the PID of a service daemon, you need to use `ps`, or a **semi-standardized system** of recording the PID, such as `/var/run/myservice.pid`. 
3. **Startup scripts** tend to include a lot of standard **“boilerplate” code** difficult to read and understand what they do.
4. There is little notion of **on-demand services and configuration**. Most services start at **boot time**; **system configuration** is largely set at that time as well. At one time, the traditional ***inetd*** daemon was able to handle **on-demand network services**, but it has largely fallen out of use

Different implementations of ***init*** have been developed to address several shortcomings by changing how services start, how they are supervised, and how the dependencies are configured 

## systemd 

The ***systemd init*** is one of the newest **init implementations** on Linux, inspiration from ***Apple’s launch***. 

**systemd** is **goal-oriented**. At the top level, you define a goal, called a ***unit***, for some system task, which can contain instructions and dependencies. When starting (or activating) a unit, ***systemd*** attempts to activate its dependencies

### Units and Unit Types 

**systemd** doesn’t just operate processes and services; but also manage filesystem mounts, monitor network connection requests, run timers, and more. Each capability is called a ***unit type*** 

1. **Service units** Control the service daemons found on a Unix system.
2. **Target units** Control other units, usually by **grouping** them.
3. **Socket units** Represent incoming network connection request locations.
4. **Mount units** Represent the attachment of filesystems to the system. 
5. see the **systemd(1) manual page** for more **unit types** 

### Booting and Unit Dependency Graphs 

When you boot a system, you’re activating a **default unit**, normally a target unit called ***default.target*** that groups together a number of **service and mount units** as dependencies.  

unit dependencies form a graph rather than a tree, You can even create a **dependency graph** with the **systemd-analyze dot** command 

### systemd Configuration 

The **systemd configuration files** are spread among many directories across the system, two main directories are

1. **system unit directory** (**global configuration**; your distribution maintains and not change yourself, `pkg-config systemd --variable=systemdsystemunitdir` usually `/lib/systemd/system` or `/usr/lib/systemd/system`
2. **system configuration directory** (**local definitions**; `pkg-config systemd --variable=systemdsystemconfdir` usually `/etc/systemd/system`) 

### Unit Files 

The format for **unit files** is derived from the **XDG Desktop Entry specification** (used for `.desktop` files,  similar to `.ini` files on **Microsoft systems**) 

In **unit files**, a variable starting with a dollar sign (`$`) is usually defined in the file specified by the **EnvironmentFile **setting 

### systemd Operation 

```bash
$ systemctl list-units
$ systemctl start
$ systemctl stop
$ systemctl restart
$ systemctl enable/disable unit # do [Install] section in unit file
$ systemctl reload unit # Reloads just the configuration for unit.
$ systemctl daemon-reload # Reloads all unit configurations
```

### systemd Dependencies and Ordering 

**systemd** uses **cgroups**, a Linux kernel feature that allows for finer tracking of a process hierarchy, to manage activated units easily 

Use the `Type` option in your **service unit file** to indicate **startup behavior**

1. **Type=simple** The service process doesn’t fork and terminate; it remains the main service process.
2. **Type=forking** The service forks, and **systemd** expects the original service process to terminate. Upon this termination, **systemd** assumes the service is ready 
3. **Type=notify** When ready, the service sends a notification specific to **systemd** with a special function call.
4. **Type=dbus** When ready, the service registers itself on the **D-Bus (Desktop Bus)**
5. **Type=oneshot** the service process terminates completely with no child processes after starting 
6. **Type=idle** This works like the ***simple style***, but it instructs **systemd** not to start the service until all active jobs finish 

**systemd** offers several **dependency types and style**

1. **Requires** Strict dependencies. If the dependency unit fails, **systemd** also deactivates the dependent unit 
2. **Wants** Dependencies for activation only. It doesn’t care if those dependencies fail 
3. **Requisite** Units that must already be active. Otherwise **systemd** fails on activation of the unit with the dependency
4. **Conflicts** Negative dependencies. systemd automatically deactivates the opposing
dependency if it’s active

When you use **ordering**, **systemd** waits until a unit has an active status before activating its dependent units. 
1. **Before** The current unit will activate before the listed unit(s).
2. **After** The current unit activates after the listed unit(s).

**systemd** internally adds an **After modifier** alongside any unit listed as a **Wants** dependency. These additional dependencies are called **default dependency**, calculated at boot time, and not stored in configuration files.
The **default dependency** is different among **unit types**, see **DEFAULT DEPENDENCIES** sections of the unit configuration manual pages, such as `systemd.service(5)` and `systemd.target(5)`.

disable **default dependency** in a unit by adding `DefaultDependencies=no` to its configuration file.  

**conditional dependency parameters** applies only to the unit in which it appears and not affect **unit dependencies **

```bash
ConditionPathExists=p # True if the (file) path p exists in the system.
ConditionPathIsDirectory=p # True if p is a directory.
ConditionFileNotEmpty=p # True if p is a file and it’s not zero-length.
```

### The [Install] Section

To define dependencies without modifying original configuration files, add a **WantedBy** or **RequiredBy** parameter in the `[Install]` section 

Enabling a unit survives reboots 

the effect of enabling a unit is to create a **symbolic link** in a `.wants` and `.requires` subdirectory corresponding to the **dependent unit** 

### systemd On-Demand and Resource-Parallelized Startup 

One of systemd’s features is the ability to delay a unit startup until it is absolutely needed 

**Resource units** such as **socket units** provide a way to do **on-demand startup**, so can be used as **auxiliary units** to simplify **dependency order** and speed up **boot time**

### systemd Auxiliary Components 

numerous programs prefixed with `systemd-` in `/lib/systemd`;

1.  **systemd-udevd**: part of **systemd**.
2. **systemd-journald** A logging service that handles a few different logging mechanisms, including the traditional Unix syslog service. 
3. **systemd-resolved** A name service caching daemon for DNS;

## System V Runlevels 

In **System V init**, this state of the machine is called its **runlevel**, from 0 through 6. 

A system spends most of its time in a single runlevel but when you shut down the machine, ***init*** switches to a different runlevel in order to terminate the system services

Runlevels serve various purposes, but the most common one is to **distinguish between system startup, shutdown, single-user mode, and console mode states**. For example, most systems traditionally used runlevels 2 through 4 for the text console; a runlevel of 5 means that the system starts a GUI login. 

```bash
# check your system’s runlevel and the datetime that the runlevel was established
$ who -r
run-level 5 2019-01-27 16:43
```

To **systemd**, **runlevels** are obsolete and exist primarily to start services that support only the **System V init scripts**. 

## System V init 

Core idea of **System V init** is to support an orderly bootup to different runlevels with a carefully constructed startup sequence. 

A typical System V init installation has two components: 

1. a central configuration file `/etc/inittab` where it all starts 
2. a large set of **boot scripts** augmented by a symbolic link farm. 

All lines in `inittab` has four fields separated by colons in this order: 

```bash
id:runlevels:action:process
```

1. A unique identifier
2. The applicable runlevel number(s).
3. The action that init should take
4. A command to execute (optional). 

System V init startup command sequence by the following with a utility, called **run-parts** to run a bunch of executable programs in a given directory, 

```bash
l5:5:wait:/etc/rc.d/rc 5
# rc stands for run commands 
```

When switching runlevels by **telinit**, init tries to kill off any processes not in the `inittab` file for the new runlevel 

### systemd System V Compatibility 

1. First, **systemd** activates `runlevel<N>.target`, where N is the runlevel.

2. For each symbolic link in `/etc/rc<N>.d`, **systemd** identifies the script in `/etc/init.d`.
3. **systemd** associates the script name with a **service unit**
4. **systemd** activates the **service unit** and runs the script with either a **start** or **stop** argument, based on its name in `rc<N>.d`.
5. **systemd** attempts to associate any processes from the script with the **service unit** 

## Shutting Down Your System 

On most machines and versions of Linux, a `halt` cuts the power to the machine.  

On Linux, `shutdown` notifies anyone logged on that the machine is going down, but it does little real work. If you specify a time other than now, the `shutdown` command creates a file called `/etc/nologin`. When this file is present, the system **prohibits logins** by anyone except the **superuser**.
When the system shutdown time finally arrives, shutdown tells **init** to begin the shutdown process. On **systemd**, this means activating the **shutdown units**, and on **System V init**, it means changing the **runlevel to 0 (halt) or 6 (reboot)**.
Regardless of the **init implementation or configuration**, the procedure generally goes like this: 

1. ***init*** asks every process to shut down cleanly.
2. If a process doesn’t respond after a while, ***init*** kills it, first with a **TERM signal**.
3. If the **TERM signal** doesn’t work, ***init*** uses the **KILL signal**
4. The system **locks system files** into place and makes other preparations for shutdown.
5. The system unmounts all filesystems other than the root.
6. The system remounts the root filesystem read-only.
7. The system writes all buffered data out to the filesystem with the ***sync*** program 
8. The final step is to tell the **kernel to reboot or stop** with the `reboot(2) system call`. This can be done by ***init*** or an auxiliary program, such as `reboot`, `halt`, or `poweroff` 

## The Initial RAM Filesystem 

Unlike **boot loader** talk to the **PC BIOS interface or EFI** to get data from disks, **Linux kernel** needs **driver support** for the underlying storage mechanism but many drivers are shipped as **loadable modules** are files. So **boot loader** loads a **archive** gathering a small collection of **kernel driver modules** along with a few other utilities into memory before running the kernel. Upon start, the kernel reads the contents of the archive into a **temporary RAM filesystem (the initramfs)**, mounts it at `/`, and performs the user-mode handoff to the ***init*** on the **initramfs**. Then, the utilities included in the **initramfs** allow the kernel to load the necessary **driver modules** for the **real root filesystem** 

If your kernel has all the drivers it needs to mount your root filesystem, you can <font color=red>omit</font> the **initial RAM filesystem** in your **boot loader configuration**.  

Most systems now use archives created by **mkinitramfs** that you can unpack with **unmkinitramfs**. Others might be older compressed **cpio archives**  

## Emergency Booting and Single-User Mode 

In the **System V init**, **single-user mode** is usually **runlevel 1**. In **systemd**, it’s represented by **rescue.target**. 

**single-user mode** doesn’t offer network, GUI, and terminal may not even work correctly. For this reason, **live images** which is simply a **Linux system** that can **boot and run without an installation process**, are nearly always considered preferable. 

# Chapter 7.  System Configuration: Logging, System Time, Batch Job And User

## System Logging 

Most system programs write their diagnostic output as messages to the **syslog service** <font color=red>(`/var/log` contains many logfiles )</font>. The traditional ***syslogd*** daemon performs this service by listening and waiting for messages on **Unix domain socket `/dev/log`** or **network socket**, and, upon receiving one, sending it to an appropriate channel, such as a file or a database or across **network to a logging server**. On most contemporary systems, ***journald*** (which comes with ***systemd***) does most of the work. 

**Syslog** has a classic **client-server architecture**, including its own protocol (currently defined in **RFC 5424**) 

A log message typically contains the **process name**, **process ID**, and **timestamp**, **facility** (a general category) and **severity** (how urgent the message is). 

use `tail -f` or `less` follow mode (`less +F`) on a logfile to monitor messages as they arrive from the system logger. For ***journald*** which doesn’t use plaintext files, use `journalctl -f` instead

### Logfile Rotation 

When logrotate decides that it’s time to delete some old data, it “rotates” the files like this:
1. Removes the oldest file, `auth.log.3`.
2. Renames `auth.log.2` to `auth.log.3`.
3. Renames `auth.log.1` to `auth.log.2`.
4. Renames `auth.log` to `auth.log.1`. 

The journals stored in `/var/log/journal` don’t need rotation, because ***journald*** itself can identify and remove old messages. 

## User Management Files 

At the kernel level, users are simply numbers (**user IDs**),  **Usernames** exist only in **user space**, so any
program that works with a username needs to find its corresponding **user ID** when talking to the kernel. 

### The /etc/passwd File 

`/etc/passwd` file syntax is fairly strict, allowing for **no comments or blank lines**.

```bash
# login name:password:user id:group id:real name (GECOS):home directory:shell
root:x:0:0:Superuser:/root:/bin/sh
```

1. ***password field***: An `x` indicates that the encrypted password is stored in the **shadow file `(/etc/shadow)`** An `asterisk (*)` indicates that the user cannot log in. **Blank field** indicates no password is required to log in and should be avoided
2. ***group ID (GID)***: which should be one of the numbered entries in the `/etc/group` file. Groups determine **file permissions** and little else. 
3. ***real name (often called the GECOS field)***. You’ll sometimes find commas in this field, denoting room and telephone numbers. 
4. The user’s shell: the shell must be listed in `/etc/shells` 

Run `passwd user` as the **superuser** to set a user’s password. Use `adduser` and `userdel` to add
and remove users, use ***chfn*** and ***chsh*** to change the **real name** and **shell**, respectively These are all **suid-root executables**, because only the superuser can change the `/etc/passwd` file.

Directly editing **passwd** is too easy to make a mistake, and has a concurrency problem  if really needed, use the ***vipw*** program to edit `/etc/passwd` and  `vipw -s`  to edit `/etc/shadow`, which backs up and locks while editing.

### Special Users 

The **superuser (root)** always has **UID 0** and **GID 0**. 

Users that have **no login privileges**, such as ***daemon*** and ***nobody***, are called **pseudo-users**, created for security reasons Although they can’t log in, the system can start processes with their user IDs. 

The ***nobody*** user is an underprivileged user, some processes run as ***nobody*** because it cannot (normally) write to anything on the system. 

Above are all **user-space conventions**. These users have no special meaning to the kernel; the only exception is **superuser’s user ID 0**. 

### The /etc/group file 

The `/etc/group` file defines the **group IDs**

To see the groups you belong to, run ***groups*** 

```bash
# group name:group passwd:group ID:An optional list of users that belong to the group
root:*:0:juser
daemon:*:1:
bin:*:2:
sys:*:3:
adm:*:4:
disk:*:6:juser,beazley
nogroup:*:65534:
user:*:1000:
```

The **Unix group passwords** are hardly ever used, nor should you use them (a good alternative in most cases is `sudo`).

## getty and login 

The system uses ***getty*** only for logins on **virtual terminals**. After you enter your login name, ***getty*** replaces itself using `exec()` with the login program. Much of the login program’s **real authentication work** is handled by **PAM (Pluggable Authentication Modules)**

## Setting the Time 

PC hardware has a **battery-backed real-time clock (RTC)**

set the **RTC** to your kernel’s UTC clock using this command: 

```bash
$ hwclock --systohc --utc
```

The **local time zone** is controlled by the file `/etc/localtime`, usually a **symbolic link** to **system-available time zone files**  in `/usr/share/zoneinfo` directory  

To set **system’s time zone**, either make a symbol link for `/etc/localtime` manually or use command-line program ***tzselect***

To use a **time zone** for just **one shell session**, set the **TZ environment variable** to the name of a file in `/usr/share/zoneinfo`, like this: 

```bash
$ export TZ=US/Central
$ date
```

set the time zone **TZ environment variable** for the **duration of a single command** like this: 

```bash
$ TZ=US/Central date
```

Network Time Protocol (NTP) was once handled by the **ntpd daemon**, but replaced by ***systemd*** package, ***timesyncd*** 

## Scheduling Recurring Tasks with cron and Timer Units 

The **cron service** has long been the **de facto standard** for doing this. However, **systemd’s timer units** are an alternative with advantages in certain cases 

### User Crontab Files 

Each user can have their own **crontab file** usually found in `/var/spool/cron/crontabs`. Normal users can’t write to this directory; use **crontab command** to install, list, edit, and remove a user’s **crontab file**. To install a **cron job**, run the **crontab** command to create an entry line in your **crontab file**

```bash
# Minute Hour Day-of-month Month Day-of-week Command
15 09 5,14 * * /home/juser/bin/spmake
```

### System Crontab Files 

Many common **cron-activated system tasks** are run as the **superuser**. However, rather than editing and maintaining a **superuser’s crontab** to schedule these tasks, Linux distributions normally have an `/etc/crontab` file for the entire system. You won’t use **crontab** to edit this file, and there’s an additional field specifying the user that should run the job.  

```bash
42 6 * * * root /usr/local/bin/cleansystem > /dev/null 2>&1
```

Some distributions store additional **system crontab files** in the `/etc/cron.d` directory. 

### Timer Units 

For an entirely new task, you must create two units: a **timer unit** and a **service unit**.

```bash 
# loggertest.timer
[Unit]
Description=Example timer unit
[Timer]
# OnCalendar time format is year-month-day hour:minute:second. The field for seconds is optional
# The periodic / syntax is also valid
OnCalendar=*-*-* *:00,20,40 # OnCalendar=*-*-* *:00/20 (every 20 minutes)
Unit=loggertest.service
[Install]
WantedBy=timers.target
```
```bash
# loggertest.service
[Unit]
Description=Example Test Service
[Service]
Type=oneshot
ExecStart=/usr/bin/logger -p local3.debug I\'m a logger
```

a utility called ***systemd-run*** does allow for creating timer units and associated services without creating unit files

## Scheduling One-Time Tasks

To run a job once in the future without using ***cron***, use the **at service**. To check that the job has been scheduled, use ***atq***. To remove it, use ***atrm*** 

As a substitute for ***at***, use ***systemd-run*** command to create a **transient timer** unit that you can view with the usual **systemctl list-timers** command 

All of the **systemd timer units** run as **root**. It’s also possible to create a timer unit as a regular user by adding the `--user` option to **systemd-run**. However, if the user log out before the unit runs, the unit won’t start; and if the user log out before the unit completes, the unit terminates because ***systemd*** has a **user manager** associated with a logged-in user, and this is necessary to run timer units. You can tell ***systemd*** to keep the user manager around after log out with this command: 

```bash
$ loginctl enable-linger
# loginctl enable-linger user
```

## User Access Topics 

### User IDs and User Switching 

**User switching** has nothing to do with passwords or usernames. Those are strictly **user-space concepts** 

All **user switching** really does is changing **user ID**. There are two ways to do this 

1. with a **setuid executable** 
2. through the **`setuid()` family** of system calls 

The **kernel** has **basic rules** about **setuid executables** and `setuid()`:

1. A process can run a **setuid executable** as long as it has adequate **file permissions**.
2. A process running as **root (user ID 0)** can use `setuid()` to become any other user.
3. A process not running as root has severe restrictions on how it may use `setuid()`; in most cases, it cannot. 

In reality, every process has more than one **user ID**

1.  **effective user ID (effective UID, or euid)**, which defines the **access rights for a process** (most significantly, file permissions) 
2. **real user ID (real UID, or ruid)**, indicates who initiated a process 
3. a **saved user ID** (which is usually not abbreviated). A process can switch its **euid** to the **ruid** or **saved user ID** during execution 
4. **file system user ID**, or **fsuid**, which defines the user accessing the filesystem but is rarely used.

To view both user IDs on your system 

```bash
$ ps -eo pid,euser,ruser,comm
```

Normally, **euid** and **ruid** are identical, but for a **setuid** program, Linux sets the **euid** to the program’s **owner** during execution, but it keeps **original user ID** in the **ruid**. However, ***sudo*** and many other **setuid programs** explicitly change both **euid** and **ruid** with one of the **`setuid()` system calls**. To prevent ***sudo*** from changing the **ruid**, add this line to `/etc/sudoers` file 

```bash
Defaults stay_setuid
```

### User Identification, Authentication, and Authorization 

A **multiuser system** must provide basic support for user security in three areas: 

1. ***identification***: answers the question of who users are 
2. ***authentication***: asks users to prove that they are who they say they are 
3. ***authorization***: define and limit what users are allowed to do 

**Linux kernel** knows only the numeric user IDs for process and file ownership and calls a function like `getpwuid()` and `geteuid()` in the standard library to get more user information.

However, the **kernel** doesn’t know anything about **authentication**: usernames, passwords, and so on. Practically everything related to authentication happens in user space 

## Pluggable Authentication Modules 

To accommodate flexibility in **user authentication**, a new standard called ***Pluggable Authentication Modules (PAM)*** is proposed , a **system of shared libraries for authentication**

Because there are many kinds of authentication scenarios, ***PAM*** employs a number of **dynamically loadable authentication modules**. To find out available **PAM modules** on your system, try `man -k pam_` and try the `locate pam_unix.so` command  to track down the location of **PAM modules**.

### PAM Configuration 

**PAM’s application configuration files** in the `/etc/pam.d` directory (older systems may use a single `/etc/pam.conf` file). Many distributions automatically generate certain PAM configuration files. The `/etc/pam.d/other` configuration file defines the default configuration for any application that lacks its own configuration file. 

**PAM configuration** is detailed on the `pam.conf(5)` manual page 

```bash
# This line says that the user’s shell must be listed in /etc/shells 
auth requisite pam_shells.so
```

Each configuration line has three fields: 

1. **Function type** The function that a user application asks PAM to perform.
2. **Control argument** This setting controls what PAM does after success or failure of its action for the current line
3. **Module** The **authentication module** that runs for this line,

For any configuration line, the **module** and **function** together determine **PAM’s action**

#### Function Types

A user application can ask PAM to perform one of the following four functions: 

1. **auth** Authenticate a user (see if the user is who they say they are).
2. **account** Check user account status (whether the user is authorized to do something, for example).
3. **session** Perform something only for the user’s current session (such as displaying a message of the day).
4. **password** Change a user’s password or other credentials.  

#### Control Arguments and Stacked Rules 

There are two kinds of control arguments: 

1. the simple syntax
   1. ***sufficient*** If this rule succeeds, the authentication is successful, and PAM doesn’t need to look at any more rules. If the rule fails, PAM proceeds to additional rules.
   2. ***requisite*** If this rule succeeds, PAM proceeds to additional rules. If the rule fails, the authentication is unsuccessful, and PAM doesn’t need to look at any more rules.
   3. ***required*** If this rule succeeds, PAM proceeds to additional rules. If the rule fails, PAM proceeds to additional rules but will always return an unsuccessful authentication regardless of the end result of the additional rules. 
2. a more advanced syntax: denoted inside **square brackets (`[]`)**, allows you to manually control a reaction based on the specific return value of the module 

#### Module Arguments 

**PAM modules** can take arguments after the module name 

```bash
auth sufficient pam_unix.so nullok # nullok argument
```

### PAM and Passwords 

The configuration file for the original **shadow password suite** `/etc/login.defs` contains information
about the **encryption algorithm** used for the `/etc/shadow` **password file**, but it’s
rarely used on a system with ***PAM*** installed.

To see which encrypt algorithm PAM uses when a user sets a password 

```bash
$ grep password.*unix /etc/pam.d/*
```

The **configuration** and **manual pages** won’t tell you anything about which algorithm to use when
authenticating. It turns out that **pam_unix.so** simply tries to guess the algorithm, usually by asking the **libcrypt library** to do the dirty work 

# Chapter 8. A Closing Look At Processes And Resource Utilization

There are three basic kinds of **hardware resources**: **CPU**, **memory**, and **I/O**. The kernel’s job
is to allocate resources fairly for processes.

## Tracking Processes 

use **top**, **atop** and **htop** 

## Finding Open Files with lsof 

The **lsof** command lists open files including **regular files, network resources, dynamic libraries, pipes**, and more and the processes using them 

run **lsof** as **root** will get more information 

The output list of **lsof** 

1. ***FD*** field can contain two kinds of elements, the purpose of the file or file descriptor of the open file 
2. ***SIZE/OFF*** field: The file’s size. 

## Tracing Program Execution and System Calls 

The **strace (system call trace)** command begins working on the new process (the copy of the original process) just after the **`fork()` call** 

The **ltrace (library trace)** command tracks **shared library calls**, it doesn’t track anything at the **kernel level**

The **ltrace** command doesn’t work on **statically linked binaries**. 

## Threads 

**top** doesn’t show threads by default; press `H` to turn it on 

To display the thread information in **ps**, add the `m` option. 

```bash
$ ps m -o pid,tid,command
```

**TID**s of the single-threaded processes are identical to the **PID**s; this is the **main thread**. 

## Resource Monitoring 

### Measuring CPU Time 

```bash
$ top -p pid1 [-p pid2 ...]
$ time cmd # shell builtin command
real 0m0.442s # real time (real) (also called elapsed time) is the total time it took to run the process from start to finish, including the time that the CPU spent doing other tasks.
user 0m0.052s # User time (user) is the number of seconds that the CPU has spent running the program’s own code.
sys 0m0.091s #  system time (sys or system) is how much time the kernel spends doing the process’s work
$ /usr/bin/time cmd # provide extensive statistics than shell builtin time
```

### Measuring CPU Performance with Load Averages 

The **load average** is the average number of processes currently <font color=red>**ready to run**</font> 

Processes waiting for input (from the keyboard, mouse, or network, for example), are **not ready to run** and shouldn’t contribute anything to the **load average** 

The **uptime command** prints how long the kernel has been running and **load averages** for the past 1 minute, 5 minutes, and 15 minutes 

```bash
$ uptime
... up 91 days, ... load average: 0.08, 0.03, 0.01
```

### Adjusting Process Priorities 

The kernel runs each process according to its **scheduling priority**, which is a number between `–20` and `20`, with `–20` being the **foremost priority**. 

By default, the ***nice*** value is 0. The kernel adds the **nice** value to the current **priority** to determine the next time slot for the process 

Only for **superuser**, **renice** command can set the **nice value** to a **negative number**, but doing so is almost always a bad idea because system processes may not get enough CPU time 

### Monitoring Memory Status 

run the **free command** or view `/proc/meminfo` to see how much real memory is being used for caches and buffers. 

#### How Memory Works 

The kernel assists the MMU by breaking down the memory used by processes into smaller chunks called ***pages***. The kernel maintains a data structure, called a **page table**, that maps a process’s **virtual page addresses** to **real page addresses** in memory. MMU translates the virtual addresses used by the process into real addresses based on the **kernel’s page table** 

The kernel does not arbitrarily map pages of real memory to virtual addresses. Real memory has many divisions that depend on hardware limitations, kernel optimization of contiguous pages, and other factors.  

#### Page Faults 

In the event of a **page fault**, the kernel takes control of the CPU from the process in order to get the page ready. There are two kinds of **page faults**: ***minor*** and ***major***

1.  **Minor page faults**: occurs when the **desired page** is actually in **main memory**, but the MMU doesn’t know where it is. This can happen when the process requests more memory or when the MMU doesn’t
   have enough space to store all of the page locations for a process (the **MMU’s internal mapping table** is usually quite small).  
2. **Major page faults**: occurs when the **desired memory page** isn’t in **main memory** at all, which means that the kernel must load it from the **disk or some other slow storage mechanism**. This can happen when load the code from disk when running a program for the first time or swapping pages of working memory out to the disk when running out of memory 

page faults for individual processes with the **ps**, **top**, and `/usr/bin/time` commands. 

### Monitoring CPU and Memory Performance with vmstat 

**vmstat command** is one of the oldest, with minimal overhead, to monitor system performance 

```bash
$ vmstat 2 # every two seconds
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
r b swpd free buff cache si so bi bo in cs us sy id wa
2 0 320416 3027696 198636 1072568 0 0 1 1 2 0 15 2 83 0
2 0 320416 3027288 198636 1072564 0 0 0 1182 407 636 1 0 99 0
```

- si (swap-in) and so (swap-out) columns 
- us, sy, id, and wa columns, list the percentage of time the CPU is spending on user tasks, system (kernel) tasks, idle time, and waiting for I/O 
- b column that a few processes are blocked (prevented from running) while waiting for memory pages 

The **pidstat** utility allows you to see the resource consumption of a process over time in the style of **vmstat** 

### I/O Monitoring 

By default, **vmstat** provides some **general I/O statistics** and detailed **per-partition resource usage** with **`vmstat -d`** 

**iostat** like **vmstat** and **iotop** like **top**

The **PRIO (priority)** column of **iotop** indicates the **I/O priority** of the process, such as `be/4`, the `be` is the **scheduling class**, and the number is the **priority level** 

The kernel uses the **scheduling class** to add more control for **I/O scheduling** 

1. `be` Best effort. The kernel does its best to **schedule I/O fairly** for this class. Most processes run under this I/O scheduling class.
2. `rt` Real time. The kernel schedules any real-time I/O before any other class of I/O, no matter what.
3. `idle` Idle. The kernel performs I/O for this class only when there is no other I/O to be done. The **idle scheduling class has no priority level.** 

check and change the **I/O priority** for a process with the **ionice** utility; see the `ionice(1)` manual page for detail 

## Control Groups (cgroups) 

There are several traditional systems, such as the **POSIX rlimit interface**, to limit resource what processes can consume,  but the most flexible option on Linux systems is now the **cgroup (control group) kernel feature**.

The basic idea is that you place several processes into a **cgroup**, which allows you to manage the resources that they consume on a **group-wide basis**.  

Although ***systemd*** makes extensive use of the **cgroup feature** and most (if not all) of the **cgroups** may be managed by ***systemd***, **cgroups** are in **kernel space** and do not depend on ***systemd***. 

There are two versions of **cgroups, 1 and 2**, and unfortunately, both are currently in use and can be configured simultaneously on a system

1. In **cgroups v1**, each type of **controller (cpu, memory, and so on)** has its own set of **cgroups**. <font color=red>A process can belong to one cgroup per controller</font>
2. In **cgroups v2**, a process can belong to **only one cgroup**. You can set up different types of controllers for each cgroup 
3. **cgroups v1** has flexibility in one respect over **v2** because you can assign different combinations of **cgroups** to processes. However, no one actually used them this way, and this approach was **more complicated to set up and implement** than simply having one cgroup per process. 
4. if a **controller** is being used in **cgroups v1**, the controller cannot be used in **v2** at the same time
   due to potential conflicts 

### Viewing cgroups 

list the **v1 and v2 cgroups** for any process by looking at file `/proc/<pid>/cgroup `.  

```bash
$ cat /proc/self/cgroup
# cgroups number:controller:name
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice
5:pids:/user.slice/user-1000.slice/session-2.scope
4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup 1
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```

1. Numbers 2–12 are for **cgroups v1**.
2. Number 1 is **cgroup without controller for version 1**. This cgroup is for **management purposes** only (in this case, ***systemd*** configured it).
3. Number 0, is for **cgroups v2**. No controllers are visible here. On a system that doesn’t have cgroups v1, this will be the only line of output 
4. Names are hierarchical and look like parts of file paths 

Unlike the **traditional Unix system call interface** for interacting with **the kernel**, **cgroups** are accessed entirely through the **filesystem**, which is usually mounted as a **cgroup2 filesystem** under `/sys/fs/cgroup`. (`/sys/fs/cgroup/unified` for **cgroups v1**) 

```bash
$ cat /proc/self/cgroup
0::/user.slice/user-1000.slice/session-2.scope
# cd /sys/fs/cgroup/unified/user.slice/user-1000.slice/session-2.scope/
$ cd /sys/fs/cgroup/user.slice/user-1000.slice/session-2.scope/
$ ls
```

the primary **cgroup interface files** begin with `cgroup`

1. `cgroup.procs` lists the processes in the cgroup.
2. `cgroup.threads` includes threads in the cgroup 
3. `cgroup.controllers`: controllers currently in use for the cgroup 
4. for **controller** such as ***pids***, look for the **files** that match the **controller prefix** such as ***pids.current***
5. files such as `cpu.stat`, `memory.current` show current **resource utilization** of all processes across their **cgroups** 

### Manipulating and Creating cgroups 

To manipulate **cgroup**, just interact with **cgroup interface files** as **root**: 

```bash
# put a process into a cgroup
echo pid > cgroup.procs
# limit the maximum number of PIDs of a cgroup
echo 3000 > pids.max
```

To **create a cgroup**, creating a subdirectory somewhere in the **cgroup tree**. Then **the kernel automatically creates the interface files**. If a **cgroup** has no processes, you can remove the cgroup with `rmdir` even with the **interface files** present 

## Further Topics 

Further topics in **resource monitoring** and **performance analysis** include:

1. ***sar*** (**System Activity Reporter**) The ***sar*** package has many of the continuous monitoring capabilities of ***vmstat***, but it also records resource utilization over time. With ***sar***, you can look back at a particular time to see what your system was doing.
2. ***acct*** (**process accounting**) The ***acct*** package can record the processes and their resource utilization.
3. ***Quotas*** You can limit the amount of **disk space** that a user can use with the **quota system**. 

# Chapter 9. Understanding your network and its configuration

The **Linux kernel** handles networking in a similar way to the **SCSI subsystem**  

## Network Layers  

**Application layer** processing occurs in **user space**.  

In Linux, the **transport layer and all layers below** are primarily handled by **the kernel**, but there are some exceptions where packets are sent into **user space** for processing.  

## The Internet Layer  

The internet currently is based on **internet protocol versions 4 (IPv4) and 6 (IPv6)**  

The **internet layer** is a **software network**, meaning no particular requirements on **hardware** or **operating systems**  

The **internet’s topology** is 

1. **decentralized**; it’s made up of smaller networks called ***subnets***.  
2. A host attached to more than one ***subnet*** is called ***gateway*** or ***router***. It can transmit data from one ***subnet*** to another

### IP Addresses  

Technically, an **IP address** consists of **4 bytes (or 32 bits)**, `abcd`. Bytes `a` and `d` are numbers from 1 to 254, and `b` and `c` are numbers from 0 to 255.

***Classless Inter-Domain Routing (CIDR) notation*** is a **subnet representation** that identifies the **subnet mask** by the number of leading 1s in the subnet mask such as `10.23.2.0/255.255.255.0` is written as `10.23.2.0/24`

The **ip command** is the current **standard network configuration tool** replacing the older command **ifconfig**, **route** and **arp**.  

```bash
$ ip address show
$ ip route show
$ route -n # The -n option tells route to show IP addresses instead of attempting to show hosts and networks by name
```

### Localhost  

The **lo loopback interface** is often the only place you might see **static network configuration** in boot-time scripts  

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group
default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
valid_lft forever preferred_lft forever
```

The **lo interface** is a **virtual network interface** called the ***loopback*** because **the kernel** just repackages outgoing data to **localhost** and sends it back through ***lo***  

### Routes and the Kernel Routing Table  

The **Linux kernel** sends data between **internet subnets** by using a ***routing table***   

```bash
$ ip route show
default via 10.23.2.1 dev enp0s31f6 proto static metric 100
10.23.2.0/24 dev enp0s31f6 proto kernel scope link src 10.23.2.4 metric 100
```

How the kernel chooses a route; **the kernel** chooses the **longest destination prefix** that matches. `10 23 2 0/24` and `0 0 0 0/0` matches, but 24 bits is longer than 0 bits, so the rule for `10 23 2 0/24` takes priority  

On most networks with a **netmask** of `/24 (255.255.255.0)`, the **router** is usually at address 1 of the **subnet** (for example, `10.23.2.1` in `10.23.2.0/24`). This is simply a **convention**, and there can be exceptions.  

### IPv6 Addresses and Networks  

**An IPv6 address has 128 bits**  typical divided into **a subnet (the first 64 bits)** and **a interface ID (the last 64 bits)**

The representation rules, `2001:0db8:0a0b:12f0:0000:0000:0000:8b6e` as `2001:db8:a0b:12f0::8b6e  `

1. leave out any leading zeros
2. set of contiguous zero groups can become `::`   

**IPv4 and IPv6 protocols** are independent of each other and can run simultaneously.   

`::/128` unspecified address corresponding to `0.0.0.0/32` in **IPv4**

`::1/128` **loopback address** and **unicast localhost address** corresponding to `127.0.0.1/8` in **IPv4**

`fe80::/10` addresses with **link-local prefix** are only valid and unique on the **local subnet**, comparable to the **auto-configuration addresses** `169.254.0.0/16` of **IPv4**

`fc00::/7` comparable to **IPv4 private addresses** `10.0.0.0/8` `172.16.0.0/12` `192.168.0.0/16`

`2000::/3` **global unicast addresses**

## Basic ICMP and DNS Tools  

**Internet Control Message Protocol (ICMP)** is a **transport layer protocol** that doesn’t carry any true user data, and thus no **application layer** above. It's used to configure and diagnose internet network  

use **IPv4** or **IPv6** to `ping` with the `-4` and `-6` options  

For **security** reasons, some hosts on the internet **disable response to ICMP echo request packets**, so you might find that you can connect to a website on a host but not get a ping response.  

To find the **IP address** behind a **domain name**, use the **host command**:  

You can also use **host** to try to discover the hostname behind the IP address but it's not reliable because administrator for that host needs to manually set up the reverse lookup, and often don’t do so  

### Resolving Hostnames  

***DNS*** is in the **application layer**, entirely in **user space**. Practically all network applications on a Linux system perform **DNS lookups**  

1. The application calls a **system’s shared library function** to look up the IP address behind a hostname.

2. When the function in the shared library runs, it acts according to a set of rules (found in **`/etc/nsswitch.conf`** to determine a plan of action on lookups. For example, the rules usually say that even before going to DNS, check for a manual override in the **`/etc/hosts`** file.

3. When the function decides to use DNS for the **name lookup**, it consults an additional configuration file (traditionally **`/etc/resolv.conf`**) to find a **DNS name server**. The name server is given as an IP address.

4. The function sends a DNS lookup request (over the network) to the name server.

5. The name server replies with the IP address for the hostname, and the function returns this IP address to the application  

Many machines run an **intermediate daemon** such as ***systemd-resolved***, ***dnsmasq***, ***nscd*** or ***BIND (the standard Unix name server daemon)*** to intercept **name server requests** and cache the reply, and then use the cached answers if possible.

You can check the current **DNS settings** with the `resolvectl status` command (note that this might be called `systemd-resolve` on older systems).  

**zero-configuration name service systems** such as ***Multicast DNS (mDNS)*** and ***Link-Local Multicast Name Resolution (LLMNR)*** allow to look up names on **local network** without messing around with a lot of configuration by **broadcasts** and reply of target host

## The Physical Layer and Ethernet  

The internet is a **software network** and works on almost any kind of computer, **operating system**, and **physical network**  

A **network layer** is on top of some kind of **hardware interface** called **physical layer**. 

The most common kind of **physical layer** is an **Ethernet network**. The ***IEEE 802 family of standards documentatio***n defines many different kinds of **Ethernet networks**, from wired to wireless, but they all  in common:  

1. All devices on an **Ethernet network** have a **Media Access Control (MAC) address**, sometimes called a **hardware address**. This address is unique to the host’s **Ethernet network**  
2. Devices on an **Ethernet network** send messages in ***frames***, which are wrappers around the data sent. A ***frame*** contains the origin and destination **MAC addresses**.  

### Ethernet, IP, ARP, and NDP 

A host using ***Ethernet*** as its **physical layer** and ***IP*** as the **network layer** maintains a small table called
an ***ARP cache*** that maps **IP addresses** to **MAC addresses**. 

In Linux, the ***ARP cache*** is in the kernel. To view your machine’s ***ARP cache***, use the `ip neigh` command. (The “neigh” part will make sense when you see the **IPv6 equivalent**. The old command for working with the ***ARP cache*** is `arp`.) 

When a machine boots, its ***ARP cache*** is empty. When the machine wants to send a packet to another host. If a **target IP address** is not in an ***ARP cache***, it uses ***Address Resolution Protocol (ARP)***  to look up **MAC address** corresponds to the **target IP address**:

1. The origin host creates a special **Ethernet frame** containing an **ARP request packet** for the MAC address corresponds to the target IP address.
2. The origin host ***broadcasts*** this frame to the **entire physical network** for the **target’s subnet**. (**ARP** applies only to machines on **local subnets**)
3. If one of the other hosts on the subnet knows the correct MAC address, it creates a r**eply packet and frame** containing the address and sends it back to the origin. Often, the host that replies is the target host and is simply replying with its own MAC address.
4. The origin host adds the IP-MAC address pair to the ***ARP cache***. 

***ARP cache*** can get out of date and **Unix systems** invalidate ***ARP cache*** entries if there’s no activity after a while 

In IPv6, there’s a new mechanism called ***Neighbor Discovery Protocol (NDP)*** used on the **link-local network** The **ip command** unifies ***ARP*** from **IPv4** and ***NDP*** from **IPv6**. ***NDP*** includes these kinds of messages:

1. **Neighbor solicitation** Used to obtain information about a **link-local host**, including the **hardware address** of the host
2. **Neighbor advertisement** Used to respond to a **neighbor solicitation message**
3. **Router advertisement (RA) message** includes the **global network prefix**, the **router IP address**, and possibly **DNS information**

### Wireless Ethernet 

***Wireless Ethernet (“Wi-Fi”)*** networks aren’t much different from **wired networks**, they also have **MAC**
**addresses** and use **Ethernet frames** to transmit and receive data. The main differences are additional components in the **physical layer**, such as frequencies, network IDs, and security features. 

1. **Transmission details** These are physical characteristics, such as the **radio frequency**.
2. **Network identification** Because more than one wireless network can share the same **basic medium**, you have to be able to distinguish between them with the **wireless network identifier**: **Service Set Identifier** (***SSID***, also known as the “network name”).
3. **Management** Although it’s possible to configure wireless networking to have hosts talk directly to each other, most wireless networks are managed by one or more **access points** that all traffic goes through.
   **Access points** often bridge a **wireless network** with a **wired network**, making both appear as one single network 
4. **Authentication** To restrict access to a wireless network, you can configure **access points** to require a password or other authentication key
5. **Encryption** Encrypt all traffic that goes out across radio waves 

The **Linux configuration and utilities** that handle these components are spread out over a number of areas. Some are in the kernel. You can view and change **kernel space device** and **network configuration**
with a utility called `iw`. 

#### Wireless Security 

For most **wireless security setups**, Linux relies on the **wpa_supplicant daemon** to manage both **authentication** and **encryption** for a **wireless network interface**.

This daemon can handle the ***WPA2*** and ***WPA3*** (***WiFi Protected Access***; don’t use the older, insecure ***WPA***) schemes of **authentication**, as well as nearly any kind of encryption technique used on wireless networks. 

When the daemon first starts, it reads a configuration file (by default, `/etc/wpa_supplicant.conf`)
and attempts to identify itself to an **access point** and establish communication based on a given network name. Mange the daemon with **automatic network configuration managers** such as ***NetworkManager*** can reduce tedious work with configuration file

## Understanding Kernel Network Interfaces  

The Linux kernel provides a **communication standard** called a ***(kernel) network interface*** for linking the **physical and the internet layers** to retain **hardware-independent** flexibility of the **internet layer**

When you configure a ***network interface***, you link the **IP address settings** from the **internet side** with the **hardware identification** on the **physical device side**.

At **boot time**, ***network interfaces*** have traditional names such as `eth0` (the first **Ethernet card** in the computer) and `wlan0` (a wireless interface), but ***systemd*** quickly rename them as ***predictable network interface device name*** that indicate the kind of **hardware underneath**, such as `enp0s31f6` (an interface in a PCI slot). 

use something like the **ethtool command** to display or change the settings on **Ethernet cards**  

## Network Configuration  

1. Connect the **network hardware** and ensure that the kernel has a driver for it. 

2. Perform any additional **physical layer setup**, such as choosing a network name or password.

3. Assign **IP address(es) and subnets** to the **kernel network interface** so that the kernel’s device drivers (physical layer) and internet subsystems (internet layer) can talk to each other.

4. Add any additional necessary routes, including the **default gateway**.  

the kernel did `step 1` and users do `step 3` with the old **ifconfig command** and `step 4` with the old **route command**  

### Network Configuration Managers  

Traditional way to ensure the correctness of a machine’s network configuration was to have ***init*** run a script to run the **manual configuration** at **boot time**. However there is usually more than simple **boot scripts** can handle, and a **system service** is used to monitor physical networks and choose (and automatically configure) the kernel network interfaces based on a set of rules

**network configuration management systems**:

1. ***NetworkManager***: The most widely used
2. ***systemd-networkd***: useful for machines that don’t need much flexibility (such as servers), **configuration directory** is `/etc/systemd/network`.  
3. for **embedded systems** such as OpenWRT’s ***netifd***, **Android**’s **ConnectivityManager** service, ***ConnMan***, and ***Wicd***  

#### NetworkManager  

**NetworkManager** is a daemon that the system starts upon boot and maintains two basic levels of configuration:

1. a collection of information about available **hardware devices**, which it normally collects from the kernel and maintains by monitoring ***udev*** over the **Desktop Bus (D-Bus)**.   
2. a more specific list of ***connections***: hardware devices and additional **physical and network layer configuration parameters**  

To activate a connection, ***NetworkManager*** uses **plug-ins** to delegate the tasks to other **distribution-dependent network tools and daemons**  

Here’s how ***NetworkManager*** makes decision for **Ethernet interfaces**:

1. If a **wired connection** is available, try to connect using it. Otherwise, try the **wireless connections**.

2. Scan the list of available **wireless networks**. If a network is available that you’ve previously connected to, ***NetworkManager*** will try it again.

3. If more than one previously connected **wireless network** is available, select the most recently connected  

use **nmcli command** to interact with ***NetworkManager*** 

***NetworkManager***’s **general configuration directory** is usually `/etc/NetworkManager`. The **general configuration file** is `NetworkManager.conf`. The format is similar to **the XDG-style `.desktop`** and **Microsoft `.ini` files**, with key-value parameters falling into different sections  

When a system’s **network interface status** changes, ***NetworkManager*** runs everything in `/etc/NetworkManager/dispatcher.d` with an argument such as `up` or `down` to specify additional system actions for when a network interface goes up or down. But many distributions have their own **network control scripts** so they don’t place the individual **dispatcher scripts** in this directory. **Ubuntu**, for example, has just one script named `01ifupdown` that runs everything in an appropriate subdirectory of `/etc/network`, such as `/etc/network/if-up.d`  

### Netplan

Different distributions have completely different implementations of **network managers** and different **configuration files**.  A tool called ***Netplan*** is a **unified network configuration standard** and a tool to transform **Netplan configuration files** (**YAML format**, reside in`/etc/netplan`)   into the files used by existing ***network managers*** such as ***NetworkManager*** and ***systemd-networkd***,   

## The Transport Layer: TCP, UDP, and Services  

**Transport layer protocols** bridge the gap between the raw packets of the **internet layer** and the refined needs of applications  

How do you know if a port is well known?  In addition to `/etc/services`, an online registry for ports at http://www.iana.org/ is governed by the RFC 6335 network standards document.  

On Linux, only processes running as the **superuser** can use ports 1 through 1023, also known as **system, well-known, or privileged ports**  

Although **UDP** has ports, it doesn’t even have connections!  

## Understanding DHCP  

***Dynamic Host Configuration Protocol (DHCP)***: get an **IP address**, **subnet mask**, **default gateway**, and **DNS servers**  

each physical network should have its own **DHCP server**, the **router** usually acts as the **DHCP server**.  

When making an initial **DHCP request**, a host doesn’t even know the address of a **DHCP server**, so it **broadcasts** the request to **all hosts** (usually all hosts on its **physical network**).  

When a machine asks a **DHCP server** to assign it an IP address, it’s really asking for a ***lease*** on an address for a certain amount of time  

Although there are many different kinds of **network manager systems**, there are only two **DHCP clients** that do the actual work of obtaining leases. 

1. The **traditional standard client** is the ***Internet Software Consortium (ISC) dhclient*** program. `dhclient` stores its **process ID** in `/var/run/dhclient.pid` and its **lease information** in `/var/lib/dhcp/dhclient.leases`. 
2. ***systemd-networkd*** now includes a **built-in DHCP client**.  

## Automatic IPv6 Network Configuration  

The IETF took advantage of the large **IPv6 address space** to devise a new way of network configuration that does not require a **central server**, called **stateless configuration**.

**Stateless IPv6 network configuration** starts with the **link-local network **prefixed `fe80::/64`.

1. Because there are so many available addresses on the link-local network, a host can generate an address with the **link-local network **prefixed `fe80::/64` that is unlikely to be duplicated anywhere on the network
2. The host **broadcasts** to the network, asking if any other host on the network is using the address 
3. Once the host has a **link-local address**, it can determine a **global address** by listening for a **router advertisement (RA) message** (includes the **global network prefix**, the **router IP address**, and possibly **DNS information**) that routers occasionally send on the **link-local network**.
4. With **global network prefix** in **RA message**, the host can attempt to fill in the **interface ID** part of the **global address**, similar to what it did with the **link-local address**. 

## Configuring Linux as a Router 

**Routers** are just computers with more than one **physical network interface**. 

However, in some basic configurations, the **Linux kernel** does not automatically move packets from one subnet to another. Enable ***IP forwarding*** in the router’s kernel with following command and add it to `/etc/sysctl.conf` file (or files in `/etc/sysctl.d`) to make this change permanent upon reboot

```bash
$ sysctl -w net.ipv4.ip_forward=1
```

***BusyBox*** is a single executable program that offers limited functionality for many **Unix commands** such as the shell, `ls`, `grep`, `cat`, and `more`. (This saves a significant amount of memory.) 

### Network Address Translation (IP Masquerading) 

In Linux, the variant of ***NAT (Network Address Translation)*** that most people use is known as **IP masquerading**. The system works roughly like this:

1. A host on the **internal private network** wants to make a connection to the outside world, so it sends its connection request packets through the **router**.
2. The **router** intercepts the **connection request packet** rather than passing it out to the **internet** (where it would get lost because the public internet knows nothing about private networks).
3. The **router** determines the **destination of the connection request packet** and opens its own connection to the destination, and then it fakes a “connection established” message back to the original internal host.
4. The **router** is now the **middleman** between the **internal host** and the **destination**. The destination knows nothing about the **internal host**; the connection on the remote host looks like it came from the router. 

In order to set up a Linux machine to perform as a **NAT router**, you must activate all of the following inside the kernel configuration: **network packet filtering (“firewall support”)**, **connection tracking**, **iptables support**, **full NAT**, and **MASQUERADE target support**. Most distribution kernels come with this support. 

use **iptables command** to make the router perform ***NAT*** for its **private subnet** 

***NAT*** is essentially a hack that extends the lifetime of the **IPv4 address space**. **IPv6** does not need ***NAT*** 

### Firewalls 

A ***firewall*** (called ***IP filtering***) is a software and/or hardware configuration that usually sits on a **router** between the internet and a smaller network, putting **checkpoints** at the **packet level.** The **checkpoints** drop, reject, or accept packets, usually based on some of these **criteria**:

1. The source or destination IP address or subnet
2. The source or destination port (in the transport layer information)
3. The firewall’s network interface 

#### Linux Firewall Basics 

Firewalls provide an opportunity to work with the subsystem of the **Linux kernel** that processes **IP packets**. The whole system is called ***iptables***, with an **iptables user-space command** to create and
manipulate the rules and there is a newer system called ***nftables*** with a **nft command**, and there’s an **iptables-to-nftables translator** called ***iptables-translate*** for the **iptables command** 

In Linux, firewall rules are known as a ***chain***. A set of ***chains*** makes up a ***table***. All of these data structures are maintained by **the kernel**. 

The kernel reads the ***chain*** from top to bottom, using the **first rule that matches**. 

There can be many ***tables*** but normally only a single ***table*** named ***filter*** controls basic **packet flow.** There are three basic ***chains*** in the **filter table**, you can see in the output of `iptables -L`: 

1. ***INPUT*** for **incoming packets**
2. ***OUTPUT*** for **outgoing packets**
3. ***FORWARD*** for **routed packets** 
4. more ***chains*** such as ***PREROUTING*** and ***POSTROUTING***

#### Firewall Strategies 

There are two basic kinds of **firewall scenarios**: one for protecting individual machines (where you set rules in each machine’s ***INPUT chain***) and one for protecting a network of machines (where you set rules in the router’s ***FORWARD chain***) 

# Chapter 10. Network Applications And Services

A server program can listen to a port on its own or through a secondary server 

Network clients use the operating system’s **transport layer protocols and interfaces** 

## telnet

directly to an unencrypted web server on ***TCP*** port 80 to get an idea 

```bash
$ telnet example.org 80
```

***telnet*** was originally meant to enable logins to remote hosts. Although the **telnet remote login server** is completely **insecure**, the **telnet client** can be useful for debugging remote services. ***telnet*** does
not work with any **transport layer** other than **TCP**. If you’re looking for a general-purpose network client, consider ***netcat*** 

```bash
$ curl --trace-ascii trace_file http://www.example.org/
```

## Network Servers 

- ***httpd***, ***apache***, ***apache2***, ***nginx*** Web servers
- ***sshd*** Secure shell daemon
- ***postfix***, ***qmail***, ***sendmail*** Mail servers 
- ***cupsd*** Print server
- ***nfsd***, mountd Network filesystem (file-sharing) daemons
- ***smbd***, ***nmbd*** Windows file-sharing daemons
- ***rpcbind*** **Remote procedure call (RPC)** portmap service daemon 

Most **network servers** usually operate as multiple processes. At least one process listens on a network port, and upon receiving a new incoming connection, the listening process uses `fork()` to create a child process, called a ***worker process***, which terminates when the connection is closed. 

To avoid system overhead of calling `fork()`, **high-performance TCP servers** such as the **Apache web server** may create a number of **worker processes** upon startup so they’re available to handle connections as needed. Servers that accept **UDP packets** don’t need to fork at all, as they don’t have connections to listen for; they simply receive data and react to it. 

### SSH 

**Standalone secure shell (SSH)** is the de facto standard for **remote access to a Unix machine**. ***SSH*** is designed to allow **secure shell logins**, **remote program execution**, simple **file sharing**, and more—replacing the old, insecure ***telnet*** and ***rlogin*** remote-access systems with public-key cryptography for authentication and simpler ciphers for session data 

`/etc/ssh` **configuration directory** contains **SSH server configuration file** `sshd_config` and global **SSH client configuration file** `ssh_config` 

***OpenSSH*** (http://www.openssh.com/) is a popular free SSH implementation for Unix, 

The OpenSSH client program is ***ssh***, and the server is ***sshd***. 

There are two main **SSH protocol versions: 1 and 2**. ***OpenSSH*** supports only version 2, having dropped version 1 support due to vulnerabilities and lack of use 

**OpenSSH** includes the **file transfer programs** `scp` and `sftp`, which are intended as replacements for the older, insecure programs `rcp` and `ftp`. It works like the cp command 

```bash
$ scp user1@host1:file user2@host2:dir
```

**SSH** does the following:

1. Encrypts your password and all other session data, protecting you from snoopers.
2. ***Tunnels*** (Tunneling is the process of packaging and transporting one **network connection**
   within another) other network connections, including those from X Window System clients.
3. Offers clients for nearly any operating system.
4. Uses keys for **host authentication**. 

### fail2ban 

To block repeated login attempts  ***fail2ban*** uses **iptables** to create a rule to deny traffic from that host. After a specified period, during which the host has probably given up trying to connect, ***fail2ban*** removes the rule. 

## Pre-systemd Network Connection Servers: inetd/xinetd 

Before the widespread use of ***systemd*** and the **socket units**, the ***inetd*** daemon, a kind of **superserver** designed to standardize network port access and interfaces between server programs and network ports. It reads its configuration file and then listens on the network ports defined in that file. As new network connections come in, ***inetd*** attaches a newly started process to the connection.
A newer version of ***inetd*** called ***xinetd*** offers easier configuration and better access control, but ***xinetd*** has almost entirely been replaced by ***systemd*** 

Before **lower-level firewalls** such as ***iptables*** became popular, many administrators used the **TCP wrapper library and daemon** to control access to network services In these implementations, ***inetd*** runs the ***tcpd*** program, which first looks at the incoming connection as well as the access control lists in the `/etc/hosts.allow` and `/etc/hosts.deny` files.

## Diagnostic Tools 

```bash
$ netstat -n
$ lsof -iprotocol@host:p
$ lsof -iTCP -sTCP:LISTEN
```

**tcpdump** puts your network interface card into ***promiscuous mode*** and reports on every packet that comes across. **tcpdump** can filter based on source and destination hosts, networks, **Ethernet addresses**, **protocols** at many different layers in the network model.

If you need to do a lot of **packet sniffing**, consider using a GUI alternative to ***tcpdump*** such as **Wireshark**. 

***netcat*** (or ***nc***) can connect to remote TCP/UDP ports, specify a local port, listen on ports, scan ports,
redirect standard I/O to and from network connections, and more. 

***netcat*** terminates when the other side ends the connection 

The ***Network Mapper (Nmap)*** program scans all ports on a machine or network of machines looking for open ports 

***RPC*** (**remote procedure call**, a system residing in the lower parts of the **application layer**) implementations use **transport protocols** such as ***TCP*** and ***UDP***, and they require a special intermediary service called ***rpcbind*** to map **program numbers** to TCP and UDP ports. 

most **FTP servers** such as ***ftpd*** use cleartext passwords. ***telnetd***, ***rlogind***, ***rexecd*** pass remote session data
(including passwords) in cleartext form. 

## Network Sockets 

***Sockets*** are the **interface** that processes use to **access the network** through **the kernel**; they represent the boundary between **user space** and **kernel space** and often also used for ***interprocess communication (IPC)*** 

Applications that use network facilities don’t have to involve two separate hosts Many applications are built as **client-server** or **peer-to-peer** mechanisms, where processes running on the same machine use **interprocess communicatio**n to negotiate what work needs to be done and who does it. For example daemons such as ***systemd*** and ***NetworkManager*** use **D-Bus** to monitor and react to **system events** 

Processes are capable of using regular **IP networking** over **localhost** (`127.0.0.1` or `::1`) to communicate with each other, but they typically use a special kind of **socket** called a **Unix domain socket** as an alternative 

Compared with **localhost**, **Unix domain sockets for IPC** allow the option to use special **socket files** in the filesystem to **control access**, and have better performance because **Linux kernel** doesn’t have to go through the many layers of its **networking subsystem** when working with **Unix domain sockets**,


# Chapter 11. Introduction to Shell Scripts

**\# character** at the beginning of a line indicates a comment with the exception of the ***shebang*** **(start with `\#!`)** at the top of a script

```bash
# Make sure that there’s no whitespace at the beginning
# env looks for the command to run in the current command path
#!/usr/bin/env python 
```

Running a script with a ***shebang*** **(`./script.sh` requires execution and readable permission bits)** is almost the same as running a command with shell **(`bash script.sh` only requires readable bit)**

<font color=red>Keep your shell scripts short  </font>

Remember what shell scripts do best: manipulate simple files and commands  

## Quoting and Literals  

what happens when the shell runs a command:

1. Before running the command, the shell looks for **variables, globs, and other substitutions** and performs the substitutions if they appear.

2. The shell passes the results of the **substitutions** to the command  

Shell won’t try any substitutions in single quotes  Double quotes work just like single quotes, except that the shell expands any variables within double quotes  

### Special Variables  

`$1`, `$2`, and all variables named as positive nonzero integers contain the values of the script parameters  

`$0` variable holds the name of the script and is useful for generating diagnostic messages  

`$#` variable holds the number of arguments passed to a script  

`$@` variable represents all of a script’s arguments and is very useful for passing them to a command inside the script  

`$$` variable holds the **process ID** of the shell.  

`$?` variable holds the **exit code of the last command** that the shell executed.   

The **built-in shell command** `shift` can be used with argument variables to remove the first argument (`$1`)  

```bash
#!/bin/sh
echo Argument: $1
shift
echo Argument: $1
shift
echo Argument: $1

$ ./shiftex one two three
Argument: one
Argument: two
Argument: three
```

## Conditionals  

All Unix systems have a command called `[`, also known as ***test***, that performs tests for shell script conditionals.

```bash
if [ $1 = hi ]
then # same as if [ $1 = hi ]; then
echo 'The first argument was "hi"'
fi
```

### Testing Conditions  

all of test operations fall into three general categories:

1. file tests,
2. string tests, 
3. arithmetic tests  

### case

```bash
#!/bin/sh
case $1 in
    bye)
        echo Fine, bye.
        ;; # just like break in C language
    hi|hello)
        echo Nice to see you.
        ;;
    what*)
        echo Whatever.
        ;;
    *)
        echo 'Huh?'
        ;;
esac
```

## Command Substitution with old backticks (\`\`) or newer `$()`

## Temporary File Management  

When creating a **temporary file**, make sure that the filename is distinct enough that no other programs will accidentally write to it using something as simple as the **shell’s PID ($$)** in a filename works, but **mktemp** is often a better option.  

A common problem with scripts that employ **temporary files** is that if the script is aborted, the temporary files could be left behind. Use the **trap command** to create a **signal handler** to catch the signal   

```bash
#!/bin/sh
TMPFILE1=$(mktemp /tmp/im1.XXXXXX)
TMPFILE2=$(mktemp /tmp/im2.XXXXXX)
# must use exit in the handler to explicitly end script execution
trap "rm -f $TMPFILE1 $TMPFILE2; exit 1" INT
```

## Here Documents  

**here document** `<<MARKER` tells the shell to redirect all subsequent lines to the standard input of the command that precedes `<<MARKER`. The redirection stops as soon as the **MARKER** **marker** occurs **on a line by itself**  

```bash
#!/bin/sh
DATE=$(date)
cat <<MARKER 
Date: $DATE
The output above is from the Unix date command.
It's not a very interesting command.
MARKER 
```

## Subshells  

**Subshell** is a **child process** launched by a shell used to alter the environment without affecting original shell   

To use a **subshell**, put the commands to be executed by the subshell in **parentheses**. 

**Pipes** and **background processes** work with **subshells**, too  

```bash
$ (PATH=/usr/confusing:$PATH; uglyprogram)
$ tar cf - orig | (cd target; tar xvf -)
```

## Including Other Files in Scripts  

include code from another file in your shell script, use the **dot (.) operator**, called ***sourcing a file***  

```bash
. config.sh
```

<font color=red>`.` is the [POSIX standard](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#dot) while **source** is a **bash builtin synonym** for `.` and is not portable.. **Bash** named it `source` because that's what it was called in **C shell** </font>

# Chapter 12. Network File Transfer And Sharing

Large-scale sharing between multiple locations with many users.  is not within the scope of this book  

## SimpleHTTPServer 

This starts a **basic web server** that makes the current directory available to any browser on the network 

```bash
$ python -m SimpleHTTPServer
```

By default, it runs on port **8000**, so if the machine you run this on is at address `10.1.2.4`, point your browser on the destination system to `http://10.1.2.4:8000` and get the files

## rsync 

```bash
$ scp -r directory user@remote_host[:dest_dir]
```

This **scp** can copy more than just a file or two,  but is not very flexible. 

On Linux, **rsync** is the **standard synchronizer**, offering good performance and many useful ways to perform transfers and useful for making **backups** 

On the surface, the **rsync** command is not much different from **scp** 

## File Sharing 

In traditional **Unix-based networks,** there were two major reasons for **file sharing**: 

1. convenience
2. lack of local storage 

**network storage performance and latency** is often poor compared to local storage 

Traditionally, **security** in **file sharing protocols** has not been treated as a high priority. Tools such as ***stunnel***, ***IPSec***, and ***VPNs*** can secure the layers below **basic file sharing protocols**. 

### Samba 

**Windows** machines using the **standard Windows network protoco**l, ***Server Message Block (SMB).*** The **standard file sharing software suite** for **Unix** is called ***Samba*** 

**macOS** supports **SMB file sharing** too, but you can also use ***SSHFS*** 

***Samba*** includes ***nmbd*** daemon as a **NetBIOS name server**, and ***smbd*** does the actual work of handling share requests 

***Samba*** allow your network’s **Windows** computers to get to your **Linux** system, but it also allows **Linux** machine to print and access files on **Windows servers** via its **Samba client software** 

To set up a **Samba server**, do the following:
1. Create an `smb.conf` file (in an directory, such as `/etc/samba` or `/usr/local/samba/lib`).
2. Add file sharing sections to `smb.conf`.
3. Add printer sharing sections to `smb.conf`.
4. Start the Samba daemons ***nmbd*** and ***smbd*** 

Runtime **diagnostic messages** of **Samba server** go to the `log.nmbd` and `log.smbd` logfiles, usually in a `/var/log` directory, such as /`var/log/samba.` 

#### The Samba Client 

The ***Samba*** **client** program ***smbclient*** can print to and access **remote Windows shares**

```bash
# get a list of shares from a remote server named SERVER
$ smbclient -L -U username SERVER
```

```bash
$ smbclient -U username '\\SERVER\sharename'
smb: \> # In this file transfer mode, smbclient is similar to the Unix ftp 
```

You can attach a **share on a Windows server** directly to **Linux system** with ***mount*** as ***Common Internet File System (CIFS)*** filesystem  

```bash
$ mount -t cifs SERVER:sharename mountpoint -o user=username,pass=password
```

### SSHFS 

A convenient option for **file sharing** between **Linux system** is ***SSHFS***, a **user-space filesystem** that opens an **SSH connection** and presents the files on the other side at a mount point on your machine 

***SSHFS*** has these advantages:

1. minimal setup. The only requirement on the remote host is that ***SFTP*** is enabled, and most SSH servers enable it by default.
2. not dependent on any kind of specific **network configuration**. ***SSHFS*** works as long as you can open an **SSH connection**

disadvantages of ***SSHFS*** are:

1. Performance suffers. There is a lot of overhead in **encryption**, **translation**, and **transport** (but it may not be as bad as you expect).
2. **Multiuser** setups are limited. 

### NFS 

One of the most commonly used traditional systems for **file sharing** among Unix systems is ***NFS (Network File System)***.

To mount a remote directory on a server with ***NFS***, use the same basic syntax as for mounting a ***CIFS*** directory: 

```bash
$ mount -t nfs server:directory mountpoint
```

***NFS*** can be a big topic because there are many different versions of NFS for different scenarios such as serving ***NFS*** over **TCP** and **UDP**, with a large number of **authentication and encryption options** (very few of which are enabled by default, unfortunately)  

The traditional **automounting tool** was called ***automount***, and there’s a newer version called ***amd***, but much of this functionality has now been supplanted by the **automount unit type** in ***systemd*** 

It’s often much more convenient to share storage over a network by simply purchasing an **NAS device** to handle it for you  

### Cloud Storage 

Another network file storage option is **cloud storage,** such as ***AWS S3*** or ***Google Cloud Storage*** 

Aside from the **web and programmatic interfaces** that all cloud storage providers offer, there are ways to **mount** most kinds of **cloud storage** on a **Linux system** as ***FUSE (File System in User Space)*** interfaces. 

# Chapter 13. User Environment 

The two main shell instance types are **interactive** and **noninteractive**, but we’re interested only in **interactive shells**, because **noninteractive shells** (such as those that run **shell scripts**) usually don’t read any **startup file**.

**Interactive shells** can be classified as **login shell** (using a program such as `/bin/login` ) or **non-login shell** (an additional shell that you run after you log in)

## Startup File Order 

### The bash Shell 

When ***bash*** runs as a **login shell**, it runs 

1. `/etc/profile.`
2.  the first one that it sees of user’s `.bash_profile`, `.bash_login,` and `.profile` files 

Upon starting up as a **non-login shell**, ***bash*** runs 

1. `/etc/bash.bashrc`
2. the user’s `.bashrc` 

### The tcsh Shell 

The standard ***csh*** on virtually all Linux systems is ***tcsh***, an **enhanced C shell**  

Upon startup as **login shells** or **non-login shells**, 

1. ***tcsh*** looks for a `.tcshrc` file. 
2. if fail, it looks for the **csh shell**’s `.cshrc` startup file 

## Shell Startup File Elements 

a **shell startup file** contains the **command path**, **prompt** (`PS1='\u\$ ' `), **aliases** and **default permissions mask** ( using **umask command**)

# Chapter 14. A Brief Survey of The Linux Desktop And Printing

## Desktop Components 

From the beginning until recently, **Linux desktops** used ***X*** (**X Window System**, also known as ***Xorg***, after its maintaining organization). However, this is now changing; many distributions have transitioned to a software set based on the ***Wayland protocol*** to build a windowing system 

***Wayland*** can run an **X application** through a **compatibility layer** with an entire ***X server*** running as a **Wayland client** called ***Xwayland*** and it’s also possible to start a **Wayland compositor** inside ***X*** 

For the task of channeling input to the correct application, most ***Wayland setups*** and many ***X servers*** use a library called `libinput` to **standardize events to clients** which is not required but is nearly **universal** on **desktop systems**.  The `libinput` library supports to collect and massage the **input** from the various `/dev/input` **kernel devices**.

***framebuffer*** is a chunk of memory that the graphics hardware reads and transmits to the screen for display.  On any contemporary system, windows belong to individual processes, doing all of their graphics
updates independently. But how does an application know where to draw its graphics?

### The X Window System
The approach that the **X Window System** takes is to have a server (called the ***X server***) that acts as a sort of “kernel” of the desktop to **manage everything** from **rendering windows** to **configuring displays** to **handling input from devices,** such as keyboards and mice, through a system of **X events**. **X events** work like other **asynchronous interprocess communication events**, such as **udev events** and **D-Bus events**, and can be experiment with the **xev command**  

**X client** programs handle the **user interface**. **Basic X client applications**, such as **terminal windows** and
**web browsers**, make connections to the ***X server*** and ask to draw windows 

The **X server** can be a significant bottleneck because it manages everything and includes a lot of **obsolete functionality**, dating back to the 1980 

The ***X server*** uses the **X Input Extension** which creates a ***virtual core device*** that funnels **device input** to the ***X server***, to manage input from many different devices. Most **X clients** listen for input from the **core devices**  

The most common way to start an ***X server*** is with a ***display manager*** (such as **gdm** for **GNOME**
and **kdm** for **KDE**), a program that starts the server and puts a **login box** on the screen. When you log in, the ***display manager*** starts a set of clients, such as a window **manager** and **file manager**

### Wayland 

Unlike ***X***, ***Wayland*** is significantly **decentralized** by design. Each client gets its own **memory buffer** for its own window, and a piece of software called a ***compositor*** combines all of the clients’ buffers into the necessary form for copying to the screen’s **framebuffer**. The ***compositor*** can be quite efficient because of **hardware support**

***X*** has a **compositing extension** that has been in use for several years now where most ***X clients*** render all of their own data as a bitmap and then send the bitmap to the **X server** without any assistance from the ***X server***

The name ***Wayland*** refers to a **communications protocol** between a c**ompositing window manager** and **graphical client** program

### Window Managers 

***window manager*** is software that determines how to arrange windows on the screen and is central to the user experience. 

In ***X***, the **window manager** is a **client** that acts as a helper to the **server** that handles input events and draw for windows’ **decorations** (such as **title bars** and c**lose buttons**),  

in ***Wayland***, the **window manager** is the **server**, responsible for compositing all of the **client window buffers** into the **display framebuffer**, and it handles the channeling of **input device events** 

Most of the popular ***window managers***, such as ***Mutter*** (in ***GNOME***) and ***Kwin*** (from ***KDE***) have
also been extended to include **Wayland compositing support**. 

### Toolkits 

To speed up development and provide a common look, programmers use **graphical toolkits** to provide ***widgets***. 

On operating systems like ***Windows*** or ***macOS***, the **vendor** provides a common **toolkit**.
On ***Linux***, the ***GTK+*** **toolkit** and **Qt framework**

### Desktop Environments 

To provide for a desktop that require a degree of **cooperation** between different applications such as one application may wish to update a common notification bar on a desktop, **toolkits** and other libraries are bundled into larger packages called ***desktop environments***. ***GNOME***, ***KDE***, and ***Xfce*** are some common **Linux desktop environments**. **Toolkits** are at the core of most ***desktop environment*** 

## D-Bus 

One of the most important developments to come out of the **Linux desktop** is the ***Desktop Bus (D-Bus)***, a **message-passing system**. ***D-Bus*** serves as an **interprocess communication mechanism,** and most **Linux systems** use it to notify processes of **system events**, such as inserting a USB drive.

***D-Bus*** itself consists of a library that standardizes **interprocess communication** with a **protocol** and supporting **functions** for any two processes to talk to each other 

Processes that need to react to events can connect to `dbus-daemon` and register to receive certain
kinds of events 

### System and Session Instances 

***D-Bus*** has become an integral part of the **Linux system**, and it now reaches beyond the desktop. For example, both ***systemd*** and ***Upstart*** have ***D-Bus channels*** of communication. 

However, adding dependencies to **desktop tools** inside the **core system** goes against a ***design rule of Linux***. So there are actually two kinds of `dbus-daemon` **instances (processes)** that can run 

1. ***system instance***, started by ***init*** at **boot time** with the `--system` option. The ***system instance***
   usually runs as a **D-Bus user**, its configuration file is `/etc/dbus-1/system.conf`. The ***system instance*** can be connected to through the `/var/run/dbus/system_bus_socket` **Unix domain socket**. 
2. ***session instance***, an **optional** instance that runs only when you start a **desktop session**. 

### D-Bus Message Monitoring 

Use the **dbus-monitor** utility to monitor the **events** that go over the ***system*** and ***session*** **dbus-daemon instances** 

```bash
# dbus-monitor --system
$ dbus-monitor --session
```

## Printing 

***PostScript*** is actually a **programming language**, so when you print a file using it, you’re sending a program to the printer. ***PostScript*** serves as a standard for printing in Unix-like system 

Printing a document on Linux is a **multistage process**:
1. The program doing the printing usually converts the document into ***PostScript*** form. This step is optional.
2. The program sends the document to a **print server**.
3. The **print server** receives the document and places it on a ***print queue***. 
4. When the document’s turn in the queue arrives, the **print server** sends the document to a **print filter**.
5. If the document is not in ***PostScript*** form, a **print filter** might perform a conversion.
6. If the destination printer does not understand ***PostScript***, a **printer driver** converts the document to a **printer-compatible format**.
7. The **printer driver** adds optional instructions to the document, such as paper tray and duplexing options.
8. The **print server** uses a **backend** to send the document to the printer. 

### CUPS 

The **standard printing system** in Linux is ***CUPS*** (http://www.cups.org/), the same system used on ***macOS***. The **CUPS server daemon** is called `cupsd` and **lpr command** can send files as a simple client to the daemon. 

One significant feature of ***CUPS*** is that it implements the ***Internet Print Protocol (IPP),*** a system that allows for **HTTP-like transactions** among clients and servers on **TCP port 631**. You can connect to `http://localhost:631/` to see your current configuration and check on any printer jobs.  

It’s usually best to use a **graphical settings interface** to add and modify **printers configuration files**, normally in `/etc/cups`.

### Format Conversion and Print Filters 

Many printers, including nearly all **low-end** models, do not understand ***PostScript*** or ***PDF***. 

To support these printers, a **Linux printing system** must convert documents to a format specific to the printer. ***CUPS*** uses the **printer drivers** to consult the ***PostScript Printer Definition (PPD)* file** for the **printer-specific settings** such as resolution and paper size and sends the document to a ***Raster Image Processor (RIP)*** to produce a bitmap. 

The ***RIP*** almost always uses the ***Ghostscript (gs)*** program to do most of the real work

# Chapter 15. Development Tools

## How ld.so Finds Shared Libraries  

A small program named `ld.so` (the **runtime dynamic linker/loader**) finds and loads shared libraries for a program at runtime.  

1. an executable’s **preconfigured runtime library search path** (***rpath***, with `-Wl,-rpath`),  (use the ***patchelf*** program to change the **runtime library search path** of an existing binary , but it’s generally better to do this at compile time.)
2. looks in a **system cache**, `/etc/ld.so.cache`  (**cache configuration file** is `/etc/ld.so.conf`)
3. environment variable ***LD_LIBRARY_PATH***.   

The `ld.so(8)` **manual page** is worth a read  

## make  

Most Linux packages are built using an additional layer around ***make*** or a similar tool.  

Running ***make*** without a **Makefile** is actually most useful when you’re dealing with something like ***Fortran***, ***Lex***, or ***Yacc***  

## Lex and Yacc  

**Lex** is a ***tokenizer*** that transforms text into numbered **tags** with **labels**. The **GNU/Linux** version is named ***flex***. You may need a `-ll` or `-lfl` **linker flag** in conjunction with **Lex**.
**Yacc** is a ***parser*** that attempts to read **tokens** according to a **grammar**. The **GNU parser** is ***bison***; to get **Yacc compatibility**, run `bison -y` and need the `-ly` **linker flag**  

## Java  

There are two kinds of Java compilers: 

1. **native compilers** for producing **machine code** for your system (like a **C compiler**) 
2. **bytecode compilers** for use by a **bytecode interpreter** (sometimes called a **virtual machine**) 

**Java bytecode files** end in `.class`. **bytecode files** that end in `.jar` are collections of archived `.class` files.   

The **Java Runtime Environment (JRE)** contains all of the programs you need to run **Java bytecode**.   

To compile a `.java` file into **bytecode**, you need run the ***javac*** **compiler** from **JDK (Java Development Kit)**

# Chapter 16. Introduction to Compile Software from Source Code

Most nonproprietary third-party Unix software packages come as **source code** that you can build and install for the reasons

1. Unix (and Linux itself) has so many different flavors and architectures  
2. widespread source code distribution throughout the Unix community encourages users to contribute bug fixes and new features to software, called ***open source***.  

## Software Build Systems  

Many **programming environments** exist on **Linux**, from traditional C to interpreted scripting languages such as Python. Each typically has at least one distinct system for **building and installing packages** in addition to the tools that a **Linux distribution** provides.  

The default **prefix** in ***GNU autoconf*** and many other packages is `/usr/local`, the traditional directory for locally installed software. **Operating system upgrades** ignore `/usr/local`  

## GNU Autoconf  

Early solutions for different platform were to provide individual ***Makefiles*** for every **operating system** or to provide a ***Makefile*** that was easy to modify. This evolved into scripts that generate Makefiles based on an analysis of the system used to build the package.  

GNU autoconf is a popular system for **automatic Makefile generation**, with files named `configure`, `Makefile.in`, and `config.h.in`. The .in files are templates; the idea is to run the ***configure*** script in order to discover the characteristics of your system, and then make substitutions in the `.in` **template files** to create the real build files including one or more ***Makefiles*** and a `config.h` file, as well as a cache file (`config.cache`), the log file is `config.log`

### configure Script Options  

Most versions of ***configure*** have a `--help` option  

By default, the **install target** from an **autoconf-generated Makefile** uses a prefix of `/usr/local`—that is, binary programs go in `/usr/local/bin`, libraries go in `/usr/local/lib`, and so on, use `--prefix` option to specify the installation directory

### Autoconf Targets  

`make distclean` This is similar to `make clean` except it removes all automatically generated files, including ***Makefiles***, `config.h`, `config.log`, and so on 

`make install-strip` This is like `make install` except it strips the **symbol table** and other **debugging information** from executables and libraries when installing.  

## Installation  

On most distributions, it’s possible to install new software as a package that you can maintain later with **distribution’s packaging tools**. 

For **Debian-based distributions**, such as ***Ubuntu***, rather than running a plain `make install`, install the package with the **checkinstall** utility  

```bash
$ checkinstall make install
```

**checkinstall** keeps track of all of the files to be installed on the system and puts them into a `.deb` file. You can then use **dpkg** to install (and remove) the new package.  

### pkg-config  

Many libraries now use the ***pkg-config*** program not only to advertise the locations of their **include files and libraries** but also to specify the **exact flags** you need to compile and link a program.  

***pkg-config*** finds package information by reading **configuration files** that end with `.pc` in the `lib/pkgconfig` directory of **installation prefix** of ***pkg-config***. Set ***PKG_CONFIG_PATH*** **environment variable** to include any extra **pkgconfig directories**.  

# Chapter 17. Visualization

## Virtual Machines  

**Virtual machines** are based on the same concept as **virtual memory**, except with all of the machine’s hardware instead of just memory. 

In this model, you create an entirely new machine (**processor, memory, I/O interfaces**, and so on) with the help of software, and run a whole **operating system** in it. This type of **virtual machine** is more specifically called a ***system virtual machine***  

A **virtual machine** can be constructed entirely in software (usually called an ***emulator***) or by utilizing the underlying hardware as much as possible, as is done in **virtual memory**

### Hypervisors  

A software called a ***hypervisor*** or ***virtual machine monitor (VMM)*** manages one or more **virtual machines** on a computer, which works similarly to how an **operating system** manages **processes**  

There are two types of ***hypervisors***

1. **Type-1** or **native hypervisors**: run directly on the host's hardware to control the hardware and manage the guest operating systems. All **cloud computing services** run as **virtual machines** under **type 1 hypervisors** such as ***Xen***  
2. **Type-2** or **hosted hypervisors**: run on a conventional OS as a process on the host

**Network interfaces** and **block devices** are among the most likely to allow guests to access **host resources** more directly to be more efficient, known as ***paravirtualization***.

### Virtual Machine CPU Modes  

When **the kernel** running inside the **virtual machine** wants to be in **kernel mode**, the ***hypervisor*** can detect and react to (“trap”) any restricted instructions coming from a virtual machine. With a little work, the ***hypervisor*** emulates the restricted instructions, enabling virtual machines to run in kernel mode on an architecture not designed for it

**Processor manufacturers** ***Intel*** and ***AMD*** released feature sets as ***VT-x*** and ***AMD-V***, respectively, to assist the ***hypervisor*** by eliminating the need for the **instruction trap** and **emulation**

## Containers  

**Virtual machines** are for insulating an **entire operating system** and its set of running applications.

**Container technology** is a lighter-weight alternative that uses a number of **kernel features** (such as ***resource limit (rlimit) feature*** and ***chroot jail*** that uses the **`chroot()` system call** to change the **root directory** to something other than the actual system root, and the process is no longer able to access anything outside that directory) to give **restricted runtime environment** for a set of processes  

Some of the important aspects of processes running in a ***container*** are:

1. They have their own **cgroups**.
2. They have their own **devices** and **filesystem**.
3. They cannot see or interact with any other processes on the system.
4. They have their own **network interfaces**  

Two of the most popular tools that perform the necessary subtasks of creating and managing effective containers are ***Docker*** and ***LXC***  

### Docker

There is an alternative to ***Docker*** called ***Podman***. The primary difference between the two tools is that ***Docker*** requires a server to be running when using **containers**, while ***Podman*** does not. This affects the way the two systems set up containers. Most **Docker configurations** require **superuser privileges** for the ***dockerd*** **daemon** to access the **kernel features** used by its **containers**. In contrast, you can run ***Podman*** as a normal user, called **rootless operation**, it uses different techniques to achieve **isolation**.  

Fortunately, ***Podman*** is **command line–compatible** with ***Docker***  

**Docker containers** run with an ***image***, which comprises the **filesystem** and a few other defining features for a container to run with.

As soon as all of container's processes terminate, Docker puts them in an **exit state**, but it still keeps the **containers** (unless you start with the `--rm` option) and `docker ps` doesn’t show e**xited containers** by default; you have to use the `-a` option to see everything.  

#### process ID   

In the **container**, the **process ID** starts at `PID 1`, whch is **kernel features** used for **containers**: ***Linux kernel namespaces*** specifically for **process IDs**. A process can create a whole new set of **process IDs** for itself and its children, starting at `PID 1`, and then they are able to see only those  

#### Overlay Filesystems  

**overlay filesystem** is a **kernel feature** that allows you to create a filesystem by combining existing directories as layers, with changes stored in a single spot  

In **rootless mode**, ***Podman*** uses the **FUSE** version of the **overlay filesystem**  

#### Networking  

The default (and most common) way to achieve **isolation** in the **network stack**, is called a **bridge network**, using the **network namespace (netns)**.   

***Docker*** creates a new **network interface** (usually `docker0`) on the host system, for communication between the **host machine** and its **containers**, typically assigned to a **private network** such as `172.17.0.0/16`.

The **Docker network** on the host configures ***NAT*** so the **containers** can reach outside hosts.

***Podman*** also uses a new **network namespace**, a ***TAP interface*** (usually at `tap0`), an interface that can be set up to operate in user space, and in conjunction with a **forwarding daemon** called ***slirp4netns***. This is less capable; for example, **container** can reach the outside world but cannot connect to one another.  

#### Docker Service Process Models  

In **Docker containers** before a process can completely **terminate**, its parent is supposed to collect (“reap”) its exit code with the `wait()` **system call**.  

However, there are some situations in which **dead processes** can remain because their parents don’t know being the parent of a child. To remedy this, 

1. for a simple single-minded service, add the `--init` option to docker run. This creates a very simple ***init*** **process** to run as `PID 1` in the **container** and act as a parent that knows what to do when a child process terminates.  
2. for multiple services or tasks inside a container, instead of starting them with
   a script, consider using a **process management daemon** such as ***Supervisor (supervisord)*** to start and monitor them  

### LXC  

The term ***LXC (Linux Containers)*** is referred specifically to a library and package containing a number of utilities for creating and manipulating **Linux containers**. Unlike Docker, ***LXC*** involves a fair amount
of manual setup.   

Originally, ***LXC*** was intended to be as much of an **entire Linux system** as possible inside the **container**—***init*** and all.  

An accompanying **management package** called ***LXD*** can help you work through some of LXC’s finer, manual points (such as network creation and image management) and offers a **REST API** that you can use to access **LXC** instead of the **C API**.  

### Kubernetes  

Google’s ***Kubernetes*** has become dominant for its ability to run **Docker container image**s.  

***Kubernetes*** has two basic sides, much like any **client-server** application. The **server** involves the machine(s) available to run **containers**, and the **client** is primarily a set of **command-line utilities** that launch and manipulate sets of **containers**.   

use the ***Minikube*** tool to install a **virtual machine** running a **Kubernetes cluster** on your own machine.  

## Runtime-Based Virtualization  

There are types of environment that differs from the **system virtual machines** and **containers**. The type of **virtual environment** is used to develop an application (such as **Python**’s **virtual environment feature `venv`** ) 
