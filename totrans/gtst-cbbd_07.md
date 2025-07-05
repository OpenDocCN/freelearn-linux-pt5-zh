# Chapter 7. Compiling the Bootloader and Kernel Using a BSP

Sometimes, a special feature in the kernel is required that is not included in the precompiled binaries, or maybe some new piece of Allwinner-based hardware was obtained that is not yet supported in the existing list of precompiled files. To solve issues like these, you need to compile the bootloader or kernel from source. While it is perfectly possible to download and compile the bootloader and the kernel by itself, the linux-sunxi community developed a **board-support-package** (**BSP**) that allows you to compile all these components together.

This chapter will cover the following topics:

*   Installing a toolchain
*   Obtaining and using the BSP
*   Compiling the bootloader
*   Compiling the kernel
*   Creating a hardware pack

# Prerequisites

Compiling things requires a compiler toolchain. Here, there are two options. Either compile on Cubieboard itself or cross compile on a regular PC. Getting a functional toolchain working on anything but Linux is up the reader to solve. The options are thus to use the installation created in the previous chapters and compile directly on Cubieboard or to have a (virtual) Linux machine available where a cross-compiler can be installed and used. In this chapter, both the methods will be described. Additionally, a working Internet connection on the device that is being used to compile on is initially required; this is to obtain the source code.

# Installing a toolchain

A toolchain is a collection of tools, including a compiler, required to compile the source code into binary forms. In theory, just the compiler is enough, but various other bits and pieces are often used to help with the compilation. One such example might be well known, the `make` command. A combination of all the tools required to compile things is known as a toolchain. Installing a toolchain varies from distribution to distribution; we will only cover a few examples here.

## Debian or Ubuntu

For Debian, the toolchain is called `build-essential`, and when cross-compiling, the arm compiler needs to be installed on top of that; this package is called `gcc-arm-none-eabi`. Unfortunately, at the time of writing this book, the `gcc-arm-none-eabi` package did not exist in Debian wheezy. The version to be released after wheezy is jessie, which contains this cross compiler. One way to get it is to set up a virtual machine and install the testing version of Debian there.

## Fedora

On Fedora, the installation is slightly different. Here, the `groupinstall` variant needs to be used to install the `Development Tools` and `Development Libraries` packages. To cross compile, install the `gcc-arm-linux-gnu` package in addition to that. It is necessary to use double quotes ("") due to the spaces in the meta packages.

## Other distributions

Fedora and Debian are of course only two distributions out of the potential hundreds. These two distributions, however, give you a very good indication of how most other distributions handle the installation of the toolchain. These two significant distributions do it differently. For the Gentoo distribution, there is a crossdev toolset, which will compile and install a cross-compiler toolchain. After installing crossdev, use `crossdev --target arm-pc-linux-gnueabi` to install the arm toolchain. The arch distribution has the `gcc-arm-linux-gnueabihf-bin` package in the **Arch User Repository** (**AUR**).

### Note

There are also vendor-supplied toolchains, such as the ones supplied by Linaro and CodeSourcery, for example. These are to be manually downloaded from the vendor site. Often, the toolchain comes in a tarball or a ZIP package and needs to be manually extracted. These toolchains are commonly used when no native arm-toolchain is available. Linaro even offers their cross-compiler for OSX and Windows; however, these two require an immense amount of work before you can start compiling.

# Other required tools and packages

Having obtained a complete toolchain, a few other packages are still required. Git is a tool used for source-code management; the package is named `git` on almost all distributions.

Additionally, `u-boot-tools` is required. The package is also named `u-boot-tools` or `uboot-mkimage` on some distributions.

On some distributions, when installing the toolchain, as described earlier, the **pkg-config** package sometimes doesn't get installed and needs to be installed on the distributions that lack it. The package is nearly always called pkg-config. To compile some of the tools, the `libusb` header files are required. The package name can vary between distributions. On Fedora, it is called **libusb-devel**. On Debian and Ubuntu, it is called **libusb-1.0-0-dev**. Please note that `libusb` is often available under several versions in many distributions. The version required is `1.0`, and other versions may cause the compilation to fail.

Finally, the *ncurses* header files and libraries are required; the package is called either **ncurses-dev** or **ncurses-devel**.

# Obtaining and maintaining the BSP

Whether the BSP is going to get cross-compiled or natively compiled, obtaining and using it is identical, and thus the instructions are common. All the code are stored on a git-server, and GitHub or Gitorious can and should be used as the main mirrors to obtain them from. Using Git, the repository can be cloned from one of the mirrors, as shown in the following screenshot:

![Obtaining and maintaining the BSP](img/1572OS_07_01.jpg)

After entering sunxi-bsp, the following list of files and directories will be visible:

![Obtaining and maintaining the BSP](img/1572OS_07_02.jpg)

Let us take a minute to quickly go over this list of file directories, of which some are actually separate Git repositories:

*   `allwinner-tools`: This is a collection of files, drivers, and tools when working with the Allwinner-supplied material, such as livesuit. It is not of importance when working with the community tools.
*   `rootfs`: These are the files to be placed into the generated root filesystem, commonly named `rootfs`.
*   `sunxi-boards`: These are the FEX files of the community-supported boards. Refer to [Appendix C](apc.html "Appendix C. The FEX Configuration File"), *The FEX Configuration File*, for more information about FEX.
*   `u-boot-sunxi`: This is the community-developed bootloader.
*   `cedarx-libs`: These are proprietary libraries for the **Video Processing Unit** (**VPU**) supplied by Allwinner.
*   `linux-sunxi`: This is the community-developed Linux kernel.
*   `scripts`: These are various scripts used by BSP.
*   `sunxi-tools`: These are community-developed tools to work with Allwinner hardware, including the FEX to a binary `script.bin` compiler.
*   `Makefile`: This is a file to control how to compile various repositories.
*   `Configure`: This is a script to choose and configure the entire BSP.
*   `README.md`: This is a simple text file with some basic usage instructions.

## Updating the repositories

Some of the directories within the BSP are not yet populated. The BSP is smart enough to populate the required repositories by itself as and when it needs them. If, however, one of the repositories is to be manually populated or more importantly, updated, Git can be used to do so. Adding the `–init` parameter after the update is required when updating the repository for the first time, as follows:

```
packt@PacktPublishing:~/sunxi-bsp$ git submodule update --init sunxi-tools

```

Omitting the last parameter, in this case, `sunxi-tools`, will update and populate all the Git subrepositories. However, this will not always yield the latest version of the repository. The BSP controls the version of each repository to use. It can be said that it locks each of the subrepositories to a certain version. The BSP itself can be updated using Git, as follows:

```
packt@PacktPublishing:~/sunxi-bsp$ git pull
Already up-to-date.

```

If, however, the BSP is not updated or does not include the latest updates to subrepositories, the subrepositories can be manually updated. To update one of the repositories, enter it, and use the regular Git commands to update or check out a different branch as follows:

```
packt@PacktPublishing:~/sunxi-bsp$ cd sunxi-tools/
packt@PacktPublishing:~/sunxi-bsp/sunxi-tools$ git pull

```

Do note that this could potentially break the version control of the BSP itself. Or rather, the local BSP will no longer match the official BSP. To delete all changes made to the local subrepository and bring the BSP in sync with the upstream version, the following command can be used:

```
packt@PacktPublishing:~/sunxi-bsp$ git checkout – sunxi-tools

```

Updating and modifying the submodules in Git is perfectly safe and is done frequently by developers. Do be careful when getting started and even more so when unfamiliar with Git.

### Note

Some experience with version control systems, especially Git, is strongly recommended before tinkering with the repositories. In case all else fails, feel assured that it is always possible to remove the BSP and start again.

# Choosing a kernel

As discussed in [Chapter 6](ch06.html "Chapter 6. Updating the Bootloader and Kernel"), *Updating the Bootloader and Kernel*, there are a few different kernels available. These kernels are built from the various branches available in the Git repository. Listing these branches is done using the `git branch` command, with the addition of the `-a` parameter telling Git to show all the available branches. In the following screenshot, the kernels discussed in [Chapter 6](ch06.html "Chapter 6. Updating the Bootloader and Kernel"), *Updating the Bootloader and Kernel*, should be recognizable:

![Choosing a kernel](img/1572OS_07_03.jpg)

The detached branch, in this case, is the kernel version that is linked to the BSP at the time of writing this book. Using `git checkout`, it is easy to switch to an alternative branch and eventually to a kernel. This can be seen in the following screenshot:

![Choosing a kernel](img/1572OS_07_04.jpg)

# Compiling for a Cubieboard

Before compiling for a Cubieboard, the BSP has to be configured first. Whenever building for a different development board, the BSP will have to be reconfigured. This however, is an easy task using the configure script. Running configure without a parameter will populate the `sunxi-boards` repository, as that repository contains a list of supported boards and prints a list of available boards, as shown in the following code. Take note of the prefix `./`, which is used to the configure the script. The output generated by configure is reduced here for clarity and convenience:

```
packt@PacktPublishing:~/sunxi-bsp$ ./configure
Usage: ./configure <board>
supported boards:
 * a10s-olinuxino-m a10s-olinuxino-m-android
 * a10-olinuxino-lime a10-olinuxino-lime-android
 * a13-olinuxino a13-olinuxino-android
 * a13-olinuxinom a13-olinuxinom-android
 * a20-olinuxino_micro a20-olinuxino_micro-android
 * cubieboard cubieboard-android
 * cubieboard2 cubieboard2-android
 * cubietruck cubietruck-android

```

There are two variants for each board that are to be passed to the configure script; the Android variant is specifically used to build Android kernels. While Android is Linux, there are some small differences that need to be accounted for. In the following example, the BSP is configured to build for Cubieboard2:

```
packt@PacktPublishing:~/sunxi-bsp$ ./configure cubieboard2
cubieboard2 configured. Now run `make`

```

Now, before running, as suggested by the BSP, one more thing needs to be mentioned. The BSP wants to know what compiler to use, and it knows this from the `CROSS_COMPILE=` parameter. By default, this parameter is forced to `arm-linux-gnueabihf-` via the Makefile, which is the prefix to the installed arm compiler. Thus `gcc` is expected to be named `arm-linux-gnueabihf-gcc`. Things get more interesting when compiling natively on one of these boards. This is because in theory, no cross-compiler is desired, `gcc` should just be called gcc. To remedy this, pass an empty `CROSS_COMPILE=` parameter to make, as follows:

```
packt@PacktPublishing:~/sunxi-bsp$ make CROSS_COMPILE=

```

Otherwise, the installed compiler prefix needs to be added, and yes, the dash at the end is a part of the prefix. If you are compiling on Debian, the following command can be used:

```
packt@PacktPublishing:~/sunxi-bsp$ make CROSS_COMPILE=arm-none-eabi-

```

Every distribution tends to name their cross-compiler differently; there is no right or wrong. Using arm- and double tab completion should yield the prefix for almost all distributions. Also, adding the correct cross-compiler prefix to the Makefile can be very helpful here.

Depending on the amount of memory and which system is used to perform the compilation, this could take from several minutes to an hour or two! If there are strange crashes or problems, before looking at [Appendix A](apa.html "Appendix A. Getting Help and Finding Other Helpful Online Resources"), *Getting Help and Other Helpful Online Resources*, about contacting the community for support, make sure that the chosen board is properly supported and supplied with adequate power. Quite often, overclocked memory or a lack of enough power will show up under the heavy stresses that a kernel compilation encompasses.

While it is nice to be able to compile the standard kernel, often, someone will want to compile a kernel due to customization, for example, with certain drivers or options added or removed. Even a custom patch is something that needs a custom-compiled kernel. Normally, the `menuconfig` command is used in a kernel directory. The BSP also allows this by passing the `linux-config` parameter to the `make` command, as follows:

```
packt@PacktPublishing:~/sunxi-bsp$ make linux-config CROSS_COMPILE=arm-none-eabi-

```

Following this will yield a standard `menuconfig` session.

Some other parameters to the `make` command are Linux or u-boot, which are used to compile only the Linux kernel or only u-boot. The resulting binaries will be located under the `sunxi-bsp/build` directory under their own respective trees.

When the compilation is completed, a so-called hwpack or hardware pack is created in the `sunxi-bsp/output` directory. The hardware pack is an archive containing three subdirectories. The first is called `bootloader` and contains the `u-boot-sunxi-with-spl.bin` bootloader.

In the `kernel` subdirectory, the board-specific kernel, named `uImage`, lives combined with the board specific `script.bin` file.

The final directory is the `rootfs` directory. This directory contains everything specific for the chosen target board. The content can and should be copied to the target root filesystem.

Installing the files in the hwpack, specifically the bootloader, was described well in the [Chapter 6](ch06.html "Chapter 6. Updating the Bootloader and Kernel"), *Updating the Bootloader and Kernel*.

# Summary

In this chapter, the basics of the BSP were covered. Using the BSP in combination with Git, which is a powerful tool to download and manage the various source repositories, you can compile various components and generate an easy-to-use device-specific hardware pack.

The next chapter will cover **general purpose input output pins** (**GPIOs**). This can be useful to do various jobs, including blinking an LED!