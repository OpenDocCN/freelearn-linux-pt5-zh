# Chapter 6. Updating the Bootloader and Kernel

The previous chapters taught you how to get started and put the hardware and software to good use. As described in the previous chapters, software packages are updated automatically via the new packages; the bootloader and kernel, however, are not. This chapter will describe the various bootloaders available as well as the various kernels and which one to choose.

In this chapter, we will cover the following topics:

*   The difference between the various bootloaders and kernels
*   Obtaining a new bootloader or kernel
*   Installing a new bootloader or kernel

# Prerequisites for this chapter

In this chapter, you will need the destination medium used in the previous chapters, such as an SSD or a hard disk, a USB drive, and a microSD card, to boot from. Optionally, a low-capacity microSD card can also be used for this purpose instead. I have used a small 128 MB microSD card for this book, but even something on the lower side would work. All that needs to fit in the microSD card is the bootloader, some configuration files, and a kernel. About 4 MB of space would be required for that, though it will be really hard to find a microSD card with that capacity.

# The bootloader overview

The bootloader is the first thing that gets loaded that is actually user modifiable. The name of the bootloader that is used by the community and throughout this book is **u-boot**. It comes in two separate flavors. First, there is the *lichee* variant but it is not actively being developed. The reason why it is still around is that it is the only bootloader that currently supports booting of the onboard NAND flash. This bootloader is nearly always preinstalled and generally there isn't any reason for it to be replaced. More interesting is the *sunxi* variant of u-boot, which is being actively developed by the community and which we will check out in the following section.

## U-boot-sunxi

As precompiled versions of a bootloader are already available, this chapter is not about compiling a bootloader. They are compiled every time a new addition is done to the codebase. Each u-boot binary is unique for each board, and an entire list of bootloaders can be found at [http://dl.linux-sunxi.org/nightly/u-boot-sunxi/u-boot-sunxi/u-boot-sunxi-latest/](http://dl.linux-sunxi.org/nightly/u-boot-sunxi/u-boot-sunxi/u-boot-sunxi-latest/). There are several files, however, for each board, as follows:

*   A build logfile ending with `.build.txt`
*   A `sha1` hash file for the generated file ending with `.sha1`
*   A file list of generated files ending within the archive ending with `.txt`
*   An archive with the compiled binary files ending with `.tar.xz`

Once the correct binary file for the development board at hand is in the downloading stage, it becomes easy with the Linux utility **wget**. To use this file for Cubieboard, the following command can be used:

```
packt@PacktPublishing:~$ wget http://dl.linux-sunxi.org/nightly/u-boot-sunxi/u-boot-sunxi/u-boot-sunxi-latest/u-boot-sunxi-cubieboard.tar.xz

```

This file will then need to be extracted; tar is a common tool to do this, where the `x` parameter stands for extract, capital `J` stands for filter through `xz`, `v` asks tar to be verbose, and `f` tells tar that the next argument is a filename.

### Note

The directory name inside the archive has a complex structure appended at the end, and most likely will not be identical to the one in this example.

Let us look at the archive in more detail. In the beginning, after the u-boot identifier, there is the architecture, sunxi. Following this is the name of the board, in this case, Cubieboard. The following command line is a timestamp of when the binary was compiled. Finally, in the end, there is a hash for the specific Git commit:

```
packt@PacktPublishing:~$ tar xJvf u-boot-sunxi-cubieboard.tar.xz
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/u-boot-sunxi-with-spl.bin
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/u-boot.bin
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/sunxi-spl.bin

```

Some explanation is required about these three files. When the Allwinner SoC chip boots for the first time, it cannot access the main system memory, which is also known as RAM. There is a little bit of memory, the SRAM, available inside the SoC chip. The SRAM, however, is too small to hold the entire u-boot, so u-boot is split into two: the **Secondary Program Loader** (**SPL**) and the actual u-boot. The SPL is just enough to initialize the memory and load the new, bigger u-boot into the memory. These files both need to be written to very specific regions so that the SoC understands where to boot from. To make it a little bit easier, there is a third file, called `u-boot-sunxi-with-spl.bin`. This file consists of both components combined with the appropriate spacing and is the only thing required at this moment.

## Installing the bootloader

The bootloader can only be installed at one location even though the SoC can boot from three different locations. SPI-flash, at the time of writing this book, is unsupported. However, some early experiments suggest that it should be relatively easy. While u-boot could be written onto the NAND flash ROM, the community-developed version of u-boot is currently unable to read or write from the NAND flash. The only option that remains is the microSD card, which as been used until now and will be used again.

The bootloader can be installed in the following ways:

*   Using a USB to microSD adapter on either a PC or on the Cubieboard
*   Using an SD card reader in a PC or on the board

    ### Note

    Be very careful when writing the bootloader. A single mistake can ruin the data on the SD card. For example, writing the bootloader at the beginning of the SD card destroys the partition table. Writing the bootloader a little too far can destroy the filesystem on the SD card. In both cases, there will be data loss because the bootloader is not in the right position, and thus the SD card is unbootable.

One requirement is that the microSD card has a partition table. It is quite common for microSD cards to have no partition table. Using the knowledge acquired in [Chapter 3](ch03.html "Chapter 3. Installing an Operating System"), *Installing an Operating System*, at least one partition should be created on the microSD card with at least 4 MB of free space. The partition has to be formatted either in `fat`, `ext2`, `ext3`, or `ext4`.

### Tip

If no partitions from the microSD card are mounted, such as the boot partition, it is safe to remove the used microSD card and insert a different one. Depending on which device is being used, make sure you replace `sdb` with `mmcblk0` or whichever device node the SD card is assigned to. Nothing to do with bits. Copying certain files, however, might be trickier, and might have to be stored elsewhere meanwhile.

Using the `dd` tool, a new bootloader is written to `/dev/sdb`. Please make sure that this really is the device node that holds the destination SD card. Using the wrong device node can destroy important data, even rendering the system unbootable. The important parameters are `blocksize` (`bs`) and `seek`, where `blocksize`, when multiplied with `seek`, indicates the exact location on the microSD card—in this case, at exactly 8 KB. The reason is simple; this is the exact location at which the SoC chip will search the SD card for the bootloader, as illustrated by the following command:

```
packt@PacktPublishing:~$ sudo dd if=u-boot-sunxi-a10-olinuxino-lime-20140307T10232-b5bd4c9/u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1024 seek=8

```

## Completing the bootloader

The bootloader does need some standard configuration files, such as the kernel that needs to be booted. A few things that cross one's thoughts are the arguments that need to be passed to the kernel and the address of the kernel to be loaded into the memory. There are three helper files that can be safely copied from the previous boot medium. They are as follows:

*   `boot.scr`: This consists of a compiled list of commands that u-boot is expected to execute
*   `uEnv.txt`: This consists of a list of variables that will be passed onto the kernel
*   `boot.cmd`: This consists of the source file used to generate `boot.scr`

    ### Tip

    The `boot.cmd` command is not used or needed; it is just very helpful to have around. To create `boot.scr` from `boot.cmd`, the following command can be used:

    ```
    mkimage -C none -A arm -T script -d boot.cmd boot.scr

    ```

Having copied these three files, the Cubieboard is almost bootable, and only the kernel remains.

# Exploring the kernel

For x86-based systems, there is usually only one kernel; the one supplied by the distribution in most cases. This kernel is always based on the kernel from [http://www.kernel.org](http://www.kernel.org). This kernel is often called the mainline kernel, vanilla kernel or upstream kernel. For Allwinner-based SoC chips, there is very limited support in the kernel developed by the community since 2012 with Version 3.8\. There are older versions of the kernel where all the features have been added by Allwinner itself. While these changes are not of the quality that the mainline kernel accepts, it is all that there is for now. It, however, is near feature-complete and thus often the only option available to date. All these various kernels are available in various branches on a Git repository. There are nightly precompiled images available just as there were for u-boot, and they can be used in this book. Before continuing to download one of these precompiled archives, a little needs to be explained about the various versions available.

## Variants of the SoC

There are various generations of Allwinner SoC chips. There is the A10, for example. The A10 series is internally known as the **sun4i**—the fourth generation of the sun series. Similarly, the **sun5i** is the internal name for the A10S, and A13\. A31 and A31S are called **sun6i**. The A20 is called **sun7i,** and **sun8i** is the name for the new A23\. The combined name for all these is called **sunxi**. For almost all the kernel versions and each of these machine types, there is either a Git branch or a nightly pre-compiled binary version.

### Overview of the various kernels

Newcomers to the sunxi community are often confused by the various kernel versions and are not sure what kernel to use. The next few subsections explain a little about the various available versions.

#### Kernel Version 2.6.36

Kernel Version 2.6.36 was originally held hostage by Allwinner, which refused to release the GPL licensed code. Once it was liberated by a device maker of Allwinner-based tablets, Allwinner officially released the sources for the kernel. A community then started to form around this source release. It has been obsolete for a long time.

#### Kernel Versions 3.0 and stage 3.0

The Version 3.0 was the first actively developed version of the sunxi kernels. The basis for the 3.0 kernel was also initially liberated by a third-party tablet manufacturer. The community did start from scratch again and put in some heavy work into this kernel's release. Later, Allwinner also released their kernel Version 3.0, but it had already diverged compared to the community-built kernel. One major thing that was done by the community, for example, is the unification of the various machine types. Allwinner intended to release the sun4i and sun5i kernels completely separated, where the community brought it under a generic sunxi kernel and drivers. It has seen many other improvements from what was supplied by Allwinner, but not all patches and fixes were immediately put into the 3.0 series. Patches were always first put into a so-called stage tree of 3.0, and after a few weeks of testing, those patches were merged back into the 3.0 tree. The 3.0 tree is technically obsolete as well but does see some active use, as it is still in use by many tablets.

#### Kernel Version 3.3

Allwinner has released a new version of their kernel, but this was after the community had already started and ported everything to the mainline 3.4 kernel. The 3.3 kernel is never used by the community, as none of the bug fixes and improvements were picked up by Allwinner. It is sometimes also known as the lichee kernel.

#### Kernel Versions 3.4 and stage 3.4

When the 3.4 kernel was released, it was marked as the **Long Term Support** (**LTS**) release, meaning it would receive support and security patches backported to it for an extended period. The drivers for sunxi hardware were forwardported from 3.0 to 3.4 to have the Allwinner hardware support, on a well-supported kernel release. Just as with 3.0, the stage variant of 3.4 holds all the new patches and is merged to 3.4, as it is considered stable. The 3.4 series is currently being actively developed and recommended for day-to-day use.

#### Kernel version experimental-3.14

The highly experimental branch for Version 3.14 is not usable as of now. The idea is to take the 3.14 kernel, which is an LTS release, and apply the 3.4 sunxi drivers to it. The big difference with 3.4 is that 3.14 was a release of the kernel, and there was some actual support for sunxi SoCs. As long as these kernels are marked experimental, they should not be used except to experiment with.

#### The devel branch of the kernel

Patches that are written against the mainline kernel end up on various mailing lists while they are being reviewed. The devel branch, which stands for development, tries to capture all these under-review patches and merges them into the mainline following tree. In other words, this tree holds support for most of the hardware to this date, but it may very well change as the review process continues. This kernel should only be used when working on drivers, doing reviews, or wanting to help testing new things. This branch, however, may be removed in future as things start to settle.

#### Next branch of the kernel

Very similar to the devel branch, it tracks the mainline kernel and holds accepted patches. It could be considered as the stable variant of the devel kernel. This kernel should only be used when there is a need for the next and upcoming kernel release.

## Choosing a kernel

With so many kernels available, there really is only one kernel of interest at this moment—the 3.4 kernel. While this may change when 3.14 leave the experimental state and start to replace 3.4—for now, they are not an option. With that choice set, there's the machine type to pick from, depending on the type of Cubieboard being used. The variants section discussed earlier will explain which machine type to go for.

## Installing the kernel

Using `wget`, an A20-based kernel will be downloaded and extracted in the following example:

```
packt@PacktPublishing:~$ wget http://dl.linux-sunxi.org/nightly/linux-sunxi/linux-sunxi-3.4-sun7i/linux-sunxi-3.4-sun7i-latest.tar.xz
linux-sunxi-3.4-sun7i-20140215T115414-8ea347b/lib/
linux-sunxi-3.4-sun7i-20140215T115414-8ea347b/boot/

```

Also here, a new directory containing a `datestamped` and `githash` named directory will be created. Within this directory, there will be two subdirectories. The boot directory holds the actual kernel, a file called **uImage**. It is to be placed on the first partition of the microSD card.

## Installing the kernel modules

In a kernel, drivers can be compiled into the kernel and are always loaded, or they are separated from the kernel and can be loaded as required. These separated drivers are called modules and on most distributions live at `/lib/modules/`. Copying over this directory copies all the files needed for this new kernel.

### Tip

The following command assumes that this is done natively on the Cubieboard. When using a different system to perform the copy, the destination path will be different:

```
packt@PacktPublishing:~$ sudo cp -ar linux-sunxi-3.4-sun7i-20140215T115414-8ea347b/lib /

```

With both the kernel and modules in place, the system can be rebooted safely using the new kernel and its modules.

# Summary

This chapter covered the various bootloaders and kernels and briefly explained the differences between them. It then showed you how to install a precompiled kernel onto a microSD card to be used as a boot device.

The next chapter will go over the steps of actually compiling a bootloader or kernel from scratch using sunxi-bsp or the sunxi board.