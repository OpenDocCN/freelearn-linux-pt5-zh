# 3

# Linux Software Management

**Software management** is an important aspect of Linux system administration because, at some level, you will have to work with software packages as a system administrator. Knowing how to work with software packages is an asset that you will master after finishing this chapter.

In this chapter, you will learn how to use specific software management commands, as well as learn how software packages work, depending on your distribution of choice. You will learn about the latest **Snap** and **Flatpak** package types and how to use them on modern Linux distributions.

In this chapter, we’re going to cover the following main topics:

*   Linux software package types
*   Managing software packages
*   Installing new desktop environments in Linux

# Technical requirements

No special technical requirements are needed for this chapter, just a working installation of Linux on your system. **Ubuntu**, **Fedora** (or **AlmaLinux**), or **openSUSE** are equally suitable for this chapter’s exercises as we will cover all types of package managers.

# Linux software package types

As you’ve already learned by now, a Linux distribution comes packed with a kernel and applications on top of it. Although plenty of applications are already installed by default, there will certainly be occasions when you will need to install some new ones or remove ones that you don’t need.

In Linux, applications come bundled into **repositories**. A repository is a centrally managed location that consists of software packages maintained by developers. These could contain individual applications or operating system-related files. Each Linux distribution comes with several official repositories, but on top of those, you can add some new ones. The way to add them is specific to each distribution, and we will get into more details later in this chapter.

Linux has several types of packages available. Ubuntu uses `deb` packages, as it is based on Debian, while Fedora (or Rocky Linux and AlmaLinux) uses `rpm` packages, as it is based on RHEL. There is also openSUSE, which uses `rpm` packages too, but it was based on Slackware at its inception. Besides those, two new package types have been recently introduced – the snap packages developed by Canonical, the company behind Ubuntu, and the flatpak packages, developed by a large community of developers and organizations, including GNOME, Red Hat, and Endless.

## The DEB and RPM package types

DEB and RPM are the oldest types of packages and are used by Ubuntu and Fedora, respectively. They are still widely used, even though the two new types mentioned earlier (snaps and flatpaks) are starting to gain ground on Linux on desktops.

Both package types are compliant with the **Linux Standard Base** (**LSB**) specifications. The last iteration of LSB is version 5.0, released in 2015\. You can find more information about it at [https://refspecs.linuxfoundation.org/lsb.shtml#PACKAGEFMT](https://refspecs.linuxfoundation.org/lsb.shtml#PACKAGEFMT).

### The DEB package’s anatomy

DEB was introduced with the Debian distribution back in 1993 and has been in use ever since on every Debian and Ubuntu derivative. A `deb` package is a binary package. This means that it contains the files of the program itself, as well as its dependencies and meta-information files, all contained inside an archive.

To check the contents of a binary `deb` package, you can use the `ar` command. It is not installed by default in Ubuntu 22.04.2 LTS, so you will have to install it yourself using the following command:

```
$ sudo apt install binutils
```

There you go – you have installed a package in Ubuntu! Now, once `ar` has been installed, you can check the contents of any `deb` package. For this exercise, we’ve downloaded the `deb` package of a password manager called **1password** and checked its contents. To query the package, perform the following steps:

1.  Use the `wget` command; the file will be downloaded inside your current working directory:

    ```
    ar t 1password-latest.deb command to view the contents of the binary package. The t option will display a table of contents for the archive:
    ```

![Figure 3.1 – Using the ar command to view the contents of a deb file](img/Figure_03_01_B19682.jpg)

Figure 3.1 – Using the ar command to view the contents of a deb file

As you can see, the output listed four files, from which two are archives. You can also investigate the package with the `ar` command.

1.  Use the `ar x 1password-latest.deb` command to extract the contents of the package to your present working directory:

    ```
    ls command to list the contents of your directory. You will see that the four files have been extracted and are ready to inspect. The debian-binary file is a text file that contains the version of the package file format, which in our case is 2.0\. You can concatenate the file to verify your package with the help of the following command:

    ```
    control.tar.gz archive contains meta-information packages and scripts to be run during the installation or before and after, depending on the case. The data.tar.xz archive contains the executable files and libraries of the program that are going to be extracted during the installation. You can check the contents with the following command:

    ```
    gpg signature file.
    ```

    ```

    ```

Important note

A `gpg` file is a file that uses the GNU Privacy Guard encryption. It uses an encryption standard known as OpenGPG (defined by the RFC4880 standard). It is usually used to sign package files as it offers a safe way for developers to distribute software. For more information on this matter, you can read the official documentation at [https://www.openpgp.org/](https://www.openpgp.org/).

The following screenshot shows the outputs of these commands:

![Figure 3.2 – The contents of a deb package](img/Figure_03_02_B19682.jpg)

Figure 3.2 – The contents of a deb package

Meta-information for each package is a collection of files that are essential for the programs to run. They contain information about certain package prerequisites and all their dependencies, conflicts, and suggestions. Feel free to explore everything that a package is made of using just the packaging-related commands.

Now that we know what a Debian-based package consists of, let’s look at the components of a Red Hat package.

### The RPM packages anatomy

The **Red Hat Package Manager** (**RPM**) packages were developed by Red Hat and are used in Fedora, CentOS, RHEL, AlmaLinux, Rocky Linux, SUSE, and openSUSE. RPM binary packages are similar to DEB binary packages in that they are also packaged as an archive.

Let’s test the `rpm` package of `1password`, just as we did with the `deb` package in the previous section:

1.  Download the `rpm` package using the following command:

    ```
    # wget https://downloads.1password.com/linux/rpm/stable/x86_64/1password-latest.rpm
    ```

If you want to use the same `ar` command, you will see that in the case of `rpms`, the archiving tool will not recognize the file format. Nevertheless, there are other more powerful tools to use.

1.  We will use the `rpm` command, the designated low-level package manager for rpms. We will use the `-q` (query), `-p` (package name), and `-l` (list) options:

    ```
    # rpm -qpl 1password-latest.rpm
    ```

The output, contrary to the `deb` package, will be a list of all the files related to the application, along with their installation locations for your system.

1.  To see the meta-information for the package, run the `rpm` command with the `-q`, `-p`, and `-i` (install) options. The following is a short excerpt from the command’s output:

![Figure 3.3 – Meta-information of the rpm package](img/Figure_03_03_B19682.jpg)

Figure 3.3 – Meta-information of the rpm package

The output will contain information about the application’s name, version, release, architecture, installation date, group, size, license, signature, source RPM, build date and host, URL, relocation, and summary.

1.  To see which other dependencies the package will require at installation, you can run the same `rpm` command with the `-q`, `-p`, and `–``requires` options:

    ```
    $ rpm -qp --requires 1password-latest.rpm
    ```

The output is shown in the following screenshot:

![Figure 3.4 – Package requirements](img/Figure_03_04_B19682.jpg)

Figure 3.4 – Package requirements

You now know what Debian and Red Hat packages are and what they contain. DEB and RPM packages are not the only types available on Linux. They are perhaps the most widely used and known, but there are also other types, depending on the distribution you choose. Also, as we stated earlier, there are new packages available for cross-platform Linux use. Those newer packages are called flatpaks and snaps, and we will detail them in the following section.

## The snap and flatpak package types

**Snap** and **Flatpak** are relatively new package types, and they are considered to be the future of apps on Linux. They both build and run applications in isolated containers for more security and portability. Both have been created to overcome the need for desktop applications’ ease of installation and portability.

Even though major Linux distributions have large application repositories, distributing software for so many types of Linux distributions, each with its own kind of package types, can become a serious issue for **independent software vendors** (**ISVs**) or community maintainers. This is where both snaps and flatpaks come to the rescue, aiming to reduce the weight of distributing software.

Let’s consider that we are ISVs, aiming to develop our product on Linux. Once a new version of our software is available, we need to create at least two types of packages to be directly downloaded from our website – a `.deb` package for Debian/Ubuntu/Mint and other derivatives, and a `.rpm` package for Fedora/RHEL/SUSE and other derivatives.

But if we want to overcome this and make our app available cross-distribution for most of the existing Linux distributions, we can distribute it as a flatpak or snap. The flatpak package would be available through **Flathub**, the centralized flatpak repository, and the snap package would be available through the Snap Store, the centralized snap repository. Either one is equally suitable for our aim to distribute the app for all major Linux distributions with minimal resource consumption and centralized effort.

Important note

Both package types are trying to overcome the overall fragmentation of the Linux ecosystem when it comes to packages. However, these two packages have different philosophies, even though they want to solve the same problem. Snaps emerged as a new type of package that would be available on the IoT and server versions of Canonical’s Ubuntu, while flatpaks emerged from the need to have a coherent package type for desktop applications in Linux. Thus, flatpaks are not available on server or IoT versions of Linux, only for desktop editions. As both packages evolve, more and more distributions are starting to provide them by default, with flatpak being the winner in terms of the number of distributions that are offering it by default. On the other hand, snaps are mostly available by default on official Ubuntu versions, starting with version 23.04\. Flatpaks are available by default in Fedora, openSUSE, Pop!_OS, Linux Mint, KDE neon, and other distributions.

The takeaway from this situation is that the effort to distribute software for Linux is higher than in the case of the same app packaged for Windows or macOS. Hopefully, in the future, there will be only one universal package for distributing software for Linux, and this will all be for the better for both users and developers alike.

### The snap package’s anatomy

The snap file is a **SquashFS** file. This means that it has its own filesystem encapsulated in an immutable container. It has a very restrictive environment, with specific rules for isolation and confinement. Every snap file has a meta-information directory that stores files that control its behavior.

Snaps, as opposed to flatpaks, are used not only for desktop applications but also for a wider range of server and embedded apps. This is because Snap has its origins in the Ubuntu **Snappy** for IoT and phones, the distribution that emerged as the beacon of convergence effort from Canonical, Ubuntu’s developer.

### The flatpak package’s anatomy

Flatpak is based on a technology called **OSTree**. The technology was started by developers from GNOME and Red Hat, and it is now heavily used in Fedora Silverblue in the form of **rpm-ostree**. It is a new upgrade system for Linux that is meant to work alongside existing package management systems. It was inspired by Git since it operates similarly. Consider it as a version control system at the OS level. It uses a content-addressed object store, allows you to share branches, and offers transactional upgrades, as well as rollback and snapshot options, for the OS.

Currently, the project has changed its name to `btrfs` filesystems.

Flatpak uses libostree, which is similar to rpm-ostree, but it is solely used for desktop application containers, with no bootloader management. Flatpak uses sandboxing based on another project named **Bubblewrap**, which allows unprivileged users to access user namespaces and use container features.

Both snaps and flatpaks have full support for graphical installations but also have commands for easier installations and setup from the shell. In the following sections, we will focus solely on command operations for all package types.

# Managing software packages

Each distribution has its own `rpm` command, while the high-level tools are the `yum` and `dnf` commands. For openSUSE, another major RPM-based distribution, the low-level tool is the same `rpm` command, but in terms of high-level tools, the `zypper` command is used. For DEB-based distributions, the low-level command is `dpkg` and the high-level command is `apt` (or the now deprecated `apt-get`).

What is the difference between low-level and high-level package managers in Linux? The low-level package managers are responsible for the backend of any package manipulation and are capable of unpacking packages, running scripts, and installing apps. The high-end managers are responsible for dependency resolution, installing and downloading packages (and groups of packages), and metadata searching.

## Managing DEB packages

Usually, for any distribution, package management is handled by the administrator or by a user with root privileges (`sudo`). Package management implies any type of package manipulation, such as installation, search, download, and removal. For all these types of operations, there are specific Linux commands, and we will show you how to use them in the following sections.

### The main repositories of Ubuntu and Debian

Ubuntu’s official repositories consist of about 60,000 packages, which take the form of binary `.deb` packages or snap packages. The configuration of the system repositories is stored in one file, the `/etc/apt/sources.list` file. Ubuntu has four main repositories, also called package sources, and you will see them detailed inside the `sources.list` file. These repositories are as follows:

*   `Main`: Contains free and open source software supported by Canonical
*   `Universe`: Contains free and open source software supported by the community
*   `Restricted`: Contains proprietary software
*   `Multiverse`: Contains software restricted by copyright

All the repositories are enabled by default in the `sources.list` file. If you would like to disable some of them, feel free to edit the file accordingly.

In Debian, the repository information is stored in the same `/etc/apt/sources.list` file. The only difference is that it uses different names for the main package sources, as follows:

*   `main`: Contains software that is compliant with Debian’s free software guidelines
*   `contrib`: Software that can, or not, be compliant with free software guidelines but is not part of the main distribution
*   `non-free`: Software that is not open source and is not compliant with free software guidelines

Debian’s and Ubuntu’s source files are very similar as they have the same information structure inside. What is different is the parts that are specific to each distribution’s package source.

Both are based on Debian’s **advanced package tool** (**APT**), so we will detail it in the following section.

### APT-related commands

Until four years ago, packages in any Debian-based distribution were implemented using the `apt-get` command. Since then, a new and improved command called `apt` (derived from the abbreviation `apt-get`, thus offering a more integrated experience.

Before doing any kind of work with the `apt` command, you should update the list of all available packages in your repositories. You can do this with the following command:

```
$ sudo apt update
```

The output of the preceding command will show you if any updates are available. The number of packages that require updates will be shown, together with a command that you could run if you want more details about them.

Before going any further, we encourage you to use the `apt --help` command as this will show you the most commonly used APT-related commands. The output is shown in the following screenshot:

![Figure 3.5 – The most commonly used apt commands](img/Figure_03_05_B19682.jpg)

Figure 3.5 – The most commonly used apt commands

Let’s dive into some of these in more detail.

#### Installing and removing packages

Basic system administration tasks include installing and removing packages. In this section, we will show you how to install and remove packages using the `apt` command.

To install a new package, you can use the `apt install` command. We used this command at the beginning of this chapter when we talked about the DEB package’s anatomy. Remember that we had to install the `ar` command as an alternative to inspect `.deb` packages. Back then, we used the following command:

```
$ sudo apt install binutils
```

This command installed several packages on the system and among them the one that we need to fulfill our action. The `apt` command automatically installs any requisite dependencies too.

To remove a package, you can use the `apt remove` or `apt purge` command. The first one removes the installed packages and all their dependencies installed by the `apt install` command. The latter will uninstall the packages, just like `apt remove`, but also deletes any configuration files created by the applications.

In the following example, we are removing the `binutils` applications we installed previously using the `apt` `remove` command:

```
$ sudo apt remove binutils
```

The output will show you a list of packages that are no longer needed and remove them from the system, asking for your confirmation to continue. This is a very good safety measure as it allows you to review the files that are going to be deleted. If you feel confident about the operation, you can add a `-y` parameter at the end of the command, which tells the shell that the answer to any question provided by the command will automatically be *Yes*.

However, using `apt remove` will not remove all the configuration files related to the removed application. To see which files are still on your system, you can use the `find` command. For example, to see the files related to the `binutils` package that were not removed, we can use the following command:

```
sudo find / -type d -name *binutils 2>/dev/null
```

The output will show the directories (hence the `-type d` option that was used with the command), where binutils-related files remain after the removal of the package.

Another tool that’s used to remove packages and all the configuration files associated with them is `apt purge`. If you want to use the `apt purge` command instead of `apt remove`, you can use it as follows:

```
$ sudo apt purge binutils
```

The output is similar to the `apt remove` command, showing you which packages will be removed and how much space will be freed on the disk, and asking for your confirmation to continue the operation.

Important note

If you plan on using `apt purge` to remove the same package (in our case, `binutils`), you will have to install it again as it was removed using the `apt` `remove` command.

The `apt remove` command has a `purge` option too, which has the same outcome as the `apt purge` command. The syntax is as follows:

```
sudo apt remove --purge [packagename]
```

As stated earlier, when using the `apt remove` command, some configuration files are left behind in case the operation was an accident and the user wants to revert to the previous configuration. The files that are not deleted by the `remove` command are small user configuration files that can easily be restored. If the operation was not an accident and you still want to get rid of all the files, you can still use the `apt purge` command to do that by using the same name as those of the already removed packages.

#### Upgrading the system

Now and then, you will need to perform a system upgrade to ensure that you have all the latest security updates and patches installed. In Ubuntu and Debian, you will always use two different commands to accomplish this. One is `apt update`, which will update the repository list and makes sure it has all the information concerning the updates that are available for the system. The other command is `apt upgrade`, which upgrades the packages. You can use them both in the same command using metacharacters:

```
$ sudo apt update; sudo apt upgrade
```

The `update` command will sometimes show you which packages are no longer required, with a message similar to the following:

```
The following packages were automatically installed and are no longer required:
  libfprint-2-tod1 libllvm9
Use 'sudo apt autoremove' to remove them.
```

You can use the `sudo apt autoremove` command to remove the unneeded packages after you perform the upgrade. The `autoremove` command’s output will show you which packages will be removed and how much space will be freed on the disk and will ask for your approval to continue the operation.

Let’s say that during our work with Ubuntu, a new distribution is released, and we would like to use that as it has newer packages of the software we use. Using the command line, we can make a full distribution upgrade. The command for this action is as follows:

```
$ sudo apt dist-upgrade
```

Similarly, we can also use the following command:

```
$ sudo apt full-upgrade
```

Upgrading to a newer distribution version should be a flawless process, but this is not always a guarantee. It all depends on your custom configurations. No matter the situation, we advise you to do a full system backup before upgrading to a new version.

#### Managing package information

Working with packages sometimes implies the use of information-gathering tools. Simply installing and removing packages is not enough. You will need to search for certain packages to show details about them, create lists based on specific criteria, and so on.

To search for a specific package, you can use the `apt search` command. It will list all the packages that have the searched string in their name, as well as others that use the string in various ways. For example, let’s search for the `nmap` package:

```
$ sudo apt search nmap
```

The output will show a considerably long list of packages that use the `nmap` string in various ways. You will still have to scroll up and down the list to find the package you want. For better results, you can pipe the output to the `grep` command, but you will notice a warning, similar to the one shown in the following screenshot:

![Figure 3.6 – Output of the apt search command](img/Figure_03_06_B19682.jpg)

Figure 3.6 – Output of the apt search command

Following the warning, the output shows a short list of packages that contain the `nmap` string, and among them is the actual package we are looking for, as highlighted in *Figure 3**.5*.

To overcome that warning, you can use a legacy command called `apt-cache search`. By running it, you will get a list of packages as output, but it won’t be as detailed as the output of the `apt` `search` command:

![Figure 3.7 – The output of the apt-cache command](img/Figure_03_07_B19682.jpg)

Figure 3.7 – The output of the apt-cache command

Now that we know that the `nmap` package exists in Ubuntu repositories, we can investigate it further by showing more details using the `apt` `show` command:

```
$ apt show nmap
```

The output will show a detailed description, including the package’s name, version, priority, origin and section, maintainer, size, dependencies, suggested extra packages, download size, APT sources, and description.

`apt` also has a useful `list` command, which can list packages based on certain criteria. For example, if we use the `apt list` command alone, it will list all the packages available. But if we use different options, the output will be personalized.

To show the installed packages, we can use the `--` `installed` option:

```
$ sudo apt list --installed
```

To list all the packages, use the following command:

```
$ sudo apt list
```

For comparative reasons, we will redirect each output to a different file, and then compare the two files. This is an easier task to do to see the differences between the two outputs since the lists are reasonably large. We will now run the specific commands, as follows:

```
$ sudo apt list --installed > list-installed
$ sudo apt list > list
```

You can compare the two resulting files by using the `ls -la` command and observe the difference in size. You will see that the `list` file will be significantly larger than the `list-installed` file.

There are other ways in which to compare the two outputs, and we would like to let you discover them by yourself, as an exercise for this sub-section. Feel free to use any other APT-related commands you would like, and practice with them enough to get familiar with their use. APT is a powerful tool, and every system administrator needs to know how to use it to sustain a usable and well-maintained Linux system. Usability is closely related to the apps that are used and their system-wide optimization.

## Managing RPM packages

RPM packages are the equivalent packages for Linux distributions such as Fedora, AlmaLinux, Rocky Linux, RHEL, and openSUSE/SLES. They have dedicated high-level tools, including `dnf`, `yum`, and `zypper`. The low-level tool is the `rpm` command.

In RHEL, the default package manager is **Yellow Dog Updater, Modified** (**YUM**) and it is based on **Dandified YUM** (**DNF**), the default package manager in Fedora. If you use both Fedora and RHEL, for ease of use, you can use only one of those as they are the same command. For consistency, we will use YUM for all the examples in this chapter.

YUM is the default high-level manager. It can install, remove, update, and package queries, as well as resolve dependencies. YUM can manage packages installed from repositories or local `.``rpm` packages.

### The main repositories in Fedora/RHEL-based distributions

Repositories are all managed from the `/etc/yum.repos.d/` directory, with configuration available inside the `/etc/yum.conf` file. If you do a listing for the `repos` directory, the output will be similar to the following screenshot:

![Figure 3.8 – RHEL derivative repositories](img/Figure_03_08_B19682.jpg)

Figure 3.8 – RHEL derivative repositories

All these files listed contain vital information about the repository, such as its name, mirror list, the `gpg` key’s location, and enabled status. All the ones listed are official repositories.

### YUM-related commands

YUM has many commands and options, but the most commonly used ones are related to package installation, removal, search, information query, system update, and repository listing.

#### Installing and removing packages

To install a package from a repository in AlmaLinux/Rocky Linux (or Fedora), simply run the `yum install` command. In the following example, we will install the GIMP application from the command line:

```
$ sudo yum install gimp
```

If you already have a package downloaded and would like to install it, you can use the `yum localinstall` command. Here, we have downloaded the 1password `.``rpm` package:

```
wget https://downloads.1password.com/linux/rpm/stable/x86_64/1password-latest.rpm
```

Then, we installed it with the following command:

```
sudo yum localinstall 1password-latest.rpm
```

The `localinstall` command automatically resolves the dependencies needed and shows the source (repository) for each of them.

This is a very powerful command that makes using the `rpm` command itself almost redundant in some cases. The main difference between the `yum install` and `yum localinstall` commands is that the latter is capable of solving dependencies for locally downloaded packages. While the former looks for packages inside the active repositories, the latter looks for packages to install in the current working directory.

To remove a package from the system, use the `yum remove` command. We will remove the newly installed 1password package:

```
sudo yum remove 1password.x86_64
```

You will be asked if you want to remove all the packages that the application installed. Choose accordingly and proceed.

Important note

The default action for pressing the *Enter* or *Return* key while inside a command dialogue in Fedora or RHEL derivatives is *N* (for no, or negative), while in Ubuntu, the default action is set to *Y* (for yes). This is a precautionary safety measure, which requires your extra attention and intervention.

The output, very similar to the output of the installation command, will show you which packages and dependencies will be removed if you proceed with the command.

As you can see, all the dependencies installed with the package using the `yum localinstall` command will be removed using the `yum remove` command. If you’re asked to proceed, type `y` and continue with the operation.

#### Updating the system

To upgrade a Fedora/RHEL-based system, we can use the `yum upgrade` command. There is also a `yum update` command, which has the same effect by updating the installed packages:

```
$ sudo yum upgrade
```

You can use the `-y` option to automatically respond to the command’s questions.

There is also an `upgrade-minimal` command, which installs only the newest security updates for packages.

#### Managing package information

Managing files with `yum` is very similar to managing files with `apt`. There are plenty of commands to use, and we will detail some of them – the ones we consider to be the most commonly used. To find out more about those commands and their use, run `yum --help`.

To see an overview of the `yum` command history and which package was managed, use the following command:

```
$ sudo yum history
```

This will give you an output that shows every `yum` command that was run, how many packages were altered, and the time and date when the actions were executed, as in the following example:

![Figure 3.9 – Using the yum history command](img/Figure_03_09_B19682.jpg)

Figure 3.9 – Using the yum history command

To show details about a certain package, we have the `yum info` command. We will query the `nmap` package, similar to what we did in Ubuntu. In CentOS, the command will be as follows:

![Figure 3.10 – Using the yum info command](img/Figure_03_10_B19682.jpg)

Figure 3.10 – Using the yum info command

The output will show you the name, version, release, source, repository, and description, very similar to what we saw with the `.``deb` packages.

To list all the installed packages or all the packages for that matter, we can use the `yum` `list` command:

```
# yum list
```

To see only the installed packages, run the following command:

```
# yum list installed
```

If we redirect the output of each command to specific files and then compare the two files, we will see the differences between them, similar to what we did in Ubuntu. The output shows the name of the packages, followed by the version and release number, and the repository from which it was installed. Here is a short excerpt:

![Figure 3.11 – Excerpt of the yum list installed command](img/Figure_03_11_B19682.jpg)

Figure 3.11 – Excerpt of the yum list installed command

As we have covered the most commonly used commands for both DEB and RPM files, we did not cover a specific package manager for openSUSE and SUSE SLE called **Zypper**. We will quickly show you some commands to get you acquainted with Zypper and let you give openSUSE a try next.

### Working with Zypper

In the case of openSUSE, **Zypper** is the package manager, similar to APT and DNF from Debian/Ubuntu and Fedora/RHEL. The following sections cover some useful commands.

#### Installing and removing packages

Similar to using APT and DNF, the Zypper package manager in openSUSE is used to install and remove packages using almost the same syntax. For example, we will install `nmap` using the `zypper` command. But first, let’s search for the name in the respective repositories to see if it exists. We will use the following command:

```
sudo zypper search nmap
```

The output of this command is a list of packages containing the `nmap` string in their name, together with the type and a summary:

![Figure 3.12 – Using the zypper search command in openSUSE](img/Figure_03_12_B19682.jpg)

Figure 3.12 – Using the zypper search command in openSUSE

You will notice `S` in the first column of the list. It consists of the status of the package, and the output will be different if the package was already installed.

From the search output, we can see that the name of the package for the Nmap application is `nmap` (it could have been a different name, hence why we used the `search` command in the first place), so we will proceed and install it on our system. We will use the `zypper install` command to do so.

Important note

In openSUSE, you can use short versions of the command. For example, instead of using `zypper install`, you can use `zypper in`, followed by the name of the package you want to install. The same goes for `zypper update`, which can be used as `zypper up`, and also for `dist-upgrade`, where you can use `dup`. Alternatively, you can use the `zypper remove` command as `zypper rm`. Check the manual pages for more information.

So, here is the command to install the `nmap` package:

```
sudo zypper install nmap
```

Alternatively, you can use the following command:

```
sudo zypper in nmap
```

You can see the output in the following screenshot:

![Figure 3.13 – Using the zypper in command](img/Figure_03_13_B19682.jpg)

Figure 3.13 – Using the zypper in command

The output shows which packages will be installed. What is important to notice here is that Zypper is automatically dealing with dependencies, just like other package managers. Besides `nmap`, there are two more library packages ready to be installed. Type `y` to continue the installation and the packages will be installed.

Now, let’s use the `zypper search nmap` command one more time to see how the list has changed when it comes to showing the package information about `nmap`:

![Figure 3.14 – Checking the status of nmap with zypper search](img/Figure_03_14_B19682.jpg)

Figure 3.14 – Checking the status of nmap with zypper search

In the output, you will see that the first column of the list has `i+` in front of the `nmap` package we just installed. This means that the package and its dependencies are already installed. So, if you are searching for some package and it is already installed, you will know this by checking the first column of the list, which is the status column.

Now, let’s remove the same package we already installed. We will use the following command:

```
sudo zypper remove nmap
```

Alternatively, we can use the following command:

```
sudo zypper rm nmap
```

The output is shown in the following screenshot:

![Figure 3.15 – Using the zypper remove command](img/Figure_03_15_B19682.jpg)

Figure 3.15 – Using the zypper remove command

The output of this command shows which packages are going to be removed. As you can see, only the `nmap` package will be removed; the other dependencies installed that were alongside won’t be removed. To remove them together with the package, use the `--clean-deps` argument when using the command. Details are shown in the following screenshot:

![Figure 3.16 – Removing dependencies](img/Figure_03_16_B19682.jpg)

Figure 3.16 – Removing dependencies

Now that you’ve learned how to use `zypper` to install and remove packages in openSUSE, let’s learn how to use it to update or upgrade the entire system.

#### Upgrading and updating the system

Before updating a system, you might want to see which updates are available. For this, you can use the following command:

```
zypper list-updates
```

The output of this command will show all the updates available on your system. To install the updates, use the following command:

```
sudo zypper update
```

An alternative is the following command:

```
sudo zypper up
```

If you use these commands with no parameters, as we just showed, all the available updates will be installed. You can also update individual packages by including the package name as a parameter for the `update` command.

Some more useful commands in openSUSE are used for adding and managing locks to a package if we don’t want it to be updated or removed. Let’s learn how to do this using the same `nmap` package. If you removed it as we did in the previous section, please install it again. We will add a lock, check for that lock, and then remove the lock for the package.

To add a lock to a package, use the `add-lock` or `zypper al` command. To see the locked packages on your system, you can use the `zypper ll` command (list locks); to remove a lock from a package, you can use the `zypper rl` command (remove locks):

![Figure 3.17 – Adding and removing locks to and from packages with Zypper](img/Figure_03_17_B19682.jpg)

Figure 3.17 – Adding and removing locks to and from packages with Zypper

Now, let’s lock the `nmap` package again and try to remove it. You will see that the package will not be removed. First, you will be asked what to do to remove it. Details are shown in the following figure:

![Figure 3.18 – Trying to remove a locked package](img/Figure_03_18_B19682.jpg)

Figure 3.18 – Trying to remove a locked package

Updating is straightforward, and you also learned how to use the lock option in Zypper to protect different packages. Now that you know how to update your openSUSE system, we’ll learn how we can find information about certain packages in the following section.

#### Managing package information

As shown when using the APT and DNF package managers in Ubuntu and Fedora, we can use Zypper in openSUSE to obtain information about packages. Let’s use the same `nmap` package as in the previous section and obtain more information about it. To do this, we will use the `zypper` `info` command:

```
sudo zypper info nmap
```

As shown in *Figure 3**.19*, the information provided is similar to that in Ubuntu and RHEL-based distributions. As we uninstalled the `nmap` package, the information shown in the output will state that the package is not installed. There is also a longer description for the package, which we did not include in the following screenshot:

![Figure 3.19 – Using the zypper info command](img/Figure_03_19_B19682.jpg)

Figure 3.19 – Using the zypper info command

Now, let’s learn how to manage flatpaks and snaps on a Linux machine.

## Using the snap and flatpak packages

Snaps and flatpaks are relatively new package types that are used in various Linux distributions. In this section, we will show you how to manage those types of packages. For snaps, we will use Ubuntu as our test distribution, while for flatpaks, we will use Fedora, even though, with a little bit of work, both package types can work on either distribution.

### Managing snap packages on Ubuntu

Snap is installed by default in Ubuntu 22.04.2 LTS. Therefore, you don’t have to do anything to install it. Simply start searching for the package you want and install it on your system. We will use the Slack application to show you how to work with snaps.

#### Searching for snaps

Slack is available in the Snap Store, so you can install it. To make sure, you can search for it using the `snap find` command, as in the following example:

```
$ snap find "slack"
```

In the command’s output, you will see many more packages that contain the `slack` string or are related to the Slack application, but only the first on the list is the one we are looking for.

Important note

In any Linux distribution, two apps originating from different packages and installed with different package managers can coexist. For example, Slack can be installed using the `deb` file provided by the website, as well as the one installed from the Snap Store.

If the output says that the package is available, we can proceed and install it on our system.

#### Installing a snap package

To install the `snap` package for Slack, we can use the `snap` `install` command:

![Figure 3.20 – Installing the Slack snap package](img/Figure_03_20_B19682.jpg)

Figure 3.20 – Installing the Slack snap package

Next, let’s see how we can find out more about the `snap` package we just installed.

#### Snap package information

If you want to find out more about the package, you can use the `snap` `info` command:

```
$ snap info slack
```

The output will show you relevant information about the package, including its name, summary, publisher, description, and ID. The last piece of information that’s displayed will be about the available **channels**, which are as follows in the case of our Slack package:

![Figure 3.21 – Snap channels shown for the Slack app](img/Figure_03_21_B19682.jpg)

Figure 3.21 – Snap channels shown for the Slack app

Each channel contains information about a specific version and it is important to know which one to choose. By default, the stable channel will be chosen by the `install` command, but if you would like a different version, you could use the `--channel` option during installation. In the preceding example, we used the default option.

#### Displaying installed snap packages

If you want to see a list of the installed snaps on your system, you can use the `snap list` command. Even though we only installed Slack on the system, in the output, you will see that many more apps have been installed. Some, such as `core` and `snapd`, are installed by default from the distribution’s installation and are required by the system:

![Figure 3.22 – Output of the snap list command](img/Figure_03_22_B19682.jpg)

Figure 3.22 – Output of the snap list command

Now, we’ll learn how to update a snap package.

#### Updating a snap package

Snaps are automatically updated. Therefore, you won’t have to do anything yourself. The least you can do is check whether an update is available and speed up its installation using the `snap refresh` command, as follows:

```
$ sudo snap refresh slack
```

Following an update, if you want to go back to a previously used version of the app, you can use the `snap revert` command, as in the following example:

```
$ sudo snap revert slack
```

In the next section, we’ll learn how to enable and disable snap packages.

#### Enabling or disabling snap packages

If we decide to not use an application temporarily, we can disable that app using the `snap disable` command. If we decide to reuse the app, we can enable it again using the `snap` `enable` command:

![Figure 3.23 – Enabling and disabling a snap app](img/Figure_03_23_B19682.jpg)

Figure 3.23 – Enabling and disabling a snap app

Remember to use `sudo` to enable and disable a snap application. If disabling is not what you are looking for, you can completely remove the snap.

#### Removing a snap package

When removing a snap application, the associated configuration files, users, and data are also removed. You can use the `snap remove` command to do this, as in the following example:

```
$ sudo snap remove slack
```

After removal, an application’s internal user, configuration, and system data are saved and retained for 31 days. These files are called snapshots, they are archived and saved under `/var/lib/snapd/snapshots`, and they contain the following types of files: a `.json` file containing a description of the snapshot, a `.tgz` file containing system data, and specific `.tgz` files that contain each system’s user details. A short listing of the aforementioned directory will show the automatically created snapshot for Slack:

![Figure 3.24 – Showing the existing snapshots after removal](img/Figure_03_24_B19682.jpg)

Figure 3.24 – Showing the existing snapshots after removal

If you don’t want the snapshots to be created, you can use the `--purge` option for the `snap remove` command. For applications that use a large amount of data, those snapshots could have a significant size and impact the available disk space. To see the snapshots saved on your system, use the `snap` `saved` command:

![Figure 3.25 – Showing the saved snapshots](img/Figure_03_25_B19682.jpg)

Figure 3.25 – Showing the saved snapshots

The output shows that in the list, in our case, just one app has been removed, with the first column indicating the ID of the snapshot (`set`). If you would like to delete a snapshot, you can do so by using the `snap forget` command. In our case, to delete the Slack application’s snapshot, we can use the following command:

![Figure 3.26 – Using the snap forget command to delete a snapshot](img/Figure_03_26_B19682.jpg)

Figure 3.26 – Using the snap forget command to delete a snapshot

To verify that the snapshot was removed, we used the `snap saved` command again, as shown in the preceding figure.

Snaps are versatile packages and easy to use. This package type is the choice of Ubuntu developers, but they are not commonly used on other distributions. If you would like to install snaps on distributions other than Ubuntu, follow the instructions at [https://snapcraft.io/docs/installing-snapd](https://snapcraft.io/docs/installing-snapd) and test its full capabilities.

Now, we will test the other new kid on the block: flatpaks. Our test distribution will be Fedora, but keep in mind that flatpaks are also supported by Ubuntu-based distributions such as Linux Mint and elementary OS, and Debian-based distributions such as PureOS and Endless OS. A list of all the supported Linux distributions can be found at [flatpak.org](http://flatpak.org).

### Managing flatpak packages on Fedora Linux

As flatpaks are available only as desktop applications, we will use Fedora Linux Workstation as our use case. In this scenario, you could use RHEL/AlmaLinux/Rocky Linux on a server, but Fedora for your workstation.

Similar to snaps, flatpaks are isolated applications that run inside sandboxes. Each flatpak contains the needed runtimes and libraries for the application. Flatpaks offer full support for graphical user interface management tools, together with a full set of commands that can be used from the `flatpak`, which has several other built-in commands to use for package management. To see all of them, use the following command:

```
$ flatpak --help
```

In the following sections, we will detail some of the widely used commands for flatpak package management. But before that, let’s say a few words about how flatpak apps are named and how they will appear on the command line so that there will be no confusion in this regard.

Each app has an identifier in a form similar to `com.company.App`. Each part of this is meant to easily identify an app and its developer. The final part identifies the application’s name since the preceding one identifies the entity that developed the app. This is an easy way for developers to publish and deliver multiple apps.

#### Adding flatpak repositories

Repositories must be set up if you wish to install applications. Flatpaks call repositories **remotes**, so this will be the term by which we will refer to them.

On our Fedora 37 machine, flatpak is already installed, but we will need to add the `flathub` repository. We will add it with the `flatpak remote-add` command, as shown in the following example:

```
$ sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Here, we used the `--if-not-exists` argument, which stops the command if the repository already exists, without showing any error. Once the repository has been added, we can start installing packages from it, but not before a mandatory system restart.

In Fedora 37 and previous versions, not all the apps from the Flathub repository are available by default, but starting with version 38, developers are aiming to provide all the apps from Flathub out of the box by default. Let’s learn how to install an application from Flathub on our Fedora Workstation.

#### Installing a flatpak application

To install a package, we need to know its name. We can go to [https://flathub.org/home](https://flathub.org/home) and search for apps there. We will search for a piece of software called **Open Broadcaster Software** (**OBS**) studio on the website and follow the instructions provided. We can either click on the **Install** button in the top right-hand corner or use the commands from the lower half of the web page. We will use the following command:

```
$ sudo flatpak install flathub com.obsproject.Studio
```

On recent versions of flatpak (since version 1.2), installation can be performed with a much simpler command. In this case, you only need the name of the app, as follows:

```
$ sudo flatpak install slack
```

The result is the same as using the first `install` command shown previously.

#### Managing flatpak applications

After installing an application, you can run it using the command line with the following command:

```
$ flatpak run com.obsproject.Studio
```

If you want to update all the applications and runtimes, you can use this command:

```
$ sudo flatpak update
```

To remove a flatpak package, simply run the `flatpak` `uninstall` command:

```
$ sudo flatpak uninstall com.obsproject.Studio
```

To list all the flatpak applications and runtimes installed, you can use the `flatpak` `list` command:

![Figure 3.27 – The flatpak list command’s output](img/Figure_03_27_B19682.jpg)

Figure 3.27 – The flatpak list command’s output

To see only the installed applications, you can use the `--``app` argument:

```
$ flatpak list --app
```

The commands shown here are the most commonly used for flatpak package management. Needless to say, there are many other commands that we will not cover here, but you are free to look them up and test them on your system. For a quick overview of the basic flatpak commands, you can refer to the following link: [https://docs.flatpak.org/en/latest/flatpak-command-reference.html](https://docs.flatpak.org/en/latest/flatpak-command-reference.html).

Flatpaks are versatile and can provide access to newer app versions. Let’s say you want to use a solid base operating system, but the downside of that is that you will get old versions of base applications by default. Using flatpaks can overcome this and give you access to newer app versions. Feel free to browse the apps available on Flathub and test the ones you find interesting and useful.

You now know how to install new applications on your operating system, using either the command line or the graphical user interface. Apart from that, you can also install new desktop environments. We will show you how in the following section.

# Installing new desktop environments in Linux

We will continue to use Fedora as an example, but the commands shown here can also be used for any RHEL-based distribution, such as AlmaLinux or Rocky Linux.

By default, Fedora Workstation uses GNOME as the desktop environment, but what if you would like to use another one, such as KDE? Before showing you how, we would like to give you some information about the graphical desktop environments available for Linux.

Linux is all about choice, and this can’t be more true when it comes to **desktop environments** (**DEs**). There are dozens of DEs available, such as GNOME, KDE, Xfce, LXDE, LXQT, Pantheon, and others. The most widely used DEs on Linux are GNOME, KDE, and Xfce, and the first two have the largest communities. If you want to use the very best and latest of GNOME, for example, you can try distributions such as Fedora, openSUSE Tumbleweed with GNOME, or Arch Linux (or Manjaro). If you want to use the best of KDE, you can try KDE neon, openSUSE Tumbleweed with KDE, or Arch Linux with KDE (or Manjaro). For Xfce, you can try MX Linux (based on Debian), which defaults to Xfce, or openSUSE with Xfce. As a rule, the most widely used Linux distributions offer variants, also called *flavors* (in the case of Ubuntu) or *spins* (in the case of Fedora) with different desktop environments available. The RHEL and SUSE commercial versions come with GNOME only by default. For more information about the DEs described here, refer to the following websites:

*   For KDE, visit [www.kde.org](http://www.kde.org)
*   For GNOME, visit [www.gnome.org](http://www.gnome.org)
*   For Xfce, visit [www.xfce.org](http://www.xfce.org)

Now, let’s learn how to install a different DE on our default Fedora Workstation.

## Installing KDE Plasma on Fedora Linux

In Fedora and derivative distributions (and also in openSUSE), there are application groups available that ease the process of installing larger apps and their dependencies. And this becomes extremely useful when you’re planning to install many apps as part of a larger *group*, just like a DE is.

To install a group, you can use the `dnf install` command and appeal the group using `@` and the name of the group. Alternatively, you can use the `dnf groupinstall` command while using the name of the group within quotes.

To check the groups that are available from the Fedora repositories, you can use the following command:

```
$ dnf group list --all | grep "KDE"
```

The output will be a list of groups from Fedora repos, and somewhere in there, the **KDE Plasma Workspaces** will be available. To install it, you can use the following command:

```
$ sudo dnf groupinstall "KDE Plasma Workspaces"
```

Alternatively, you can use the following command:

```
$ sudo dnf install @kde-desktop-environment
```

This command will install the new KDE Plasma environment, as shown in the following figure:

![Figure 3.28 – Installing the KDE Plasma DE](img/Figure_03_28_B19682.jpg)

Figure 3.28 – Installing the KDE Plasma DE

The installation might take a while, depending on your internet connection. To start using KDE Plasma as your DE, you will need to log out of the active session. On the login screen, select your user, and then, in the bottom-right corner, click on the wheel icon and select **Plasma** when the option becomes available. You will have two options, one for **Wayland** and the other for the **X11** display manager:

![Figure 3.29 – Selecting Plasma (Wayland) on the login screen](img/Figure_03_29_B19682.jpg)

Figure 3.29 – Selecting Plasma (Wayland) on the login screen

Wayland is the newer option and might not have full support in KDE compared to the support it has in GNOME. Choose according to your preference.

Now, you can log into KDE Plasma on Fedora Workstation. The following screenshot shows the **Info Center** application inside KDE Plasma, with details about the installed version and hardware:

![Figure 3.30 – Info Center in KDE Plasma on Fedora 37](img/Figure_03_30_B19682.jpg)

Figure 3.30 – Info Center in KDE Plasma on Fedora 37

With that, you have learned about working with packages in Linux, and even installing a new DE. This is sufficient for you to start fiddling around with your new OS. You can install new applications, configure them, and make your distribution the way you want it to be.

# Summary

In this chapter, you learned how to work with packages in Ubuntu, Fedora/AlmaLinux, and openSUSE, and the skills you’ve learned will help you to manage packages in any Linux distribution. You learned how to work with both `.deb` and `.rpm` packages, and also the newer ones, such as flatpaks and snaps. You will use the skills you’ve learned here in every chapter of this book, as well as in your day-to-day job as a systems administrator – or even in your free time, enjoying your Linux operating system.

In the next chapter, we will show you how to manage user accounts and permissions, where you will be introduced to general concepts and specific tools.

# Questions

Now that you have a clear idea of how to manage software packages, here are some exercises that will contribute further to your learning:

1.  Make a list of all the packages installed on your system.

`apt list --``installed` command.

1.  Add support for flatpaks on your Ubuntu system.

**Hint**: Follow the documentation at [flatpak.org](http://flatpak.org).

1.  Test other distributions and use their package managers. We recommend that you try openSUSE and, if you feel confident, Arch Linux.

# Further reading

For more information about what was covered in this chapter, please refer to the following resources:

*   *Mastering Linux Administration – First Edition*, Alexandru Calcatinge, Julian Balog
*   Snapcraft.io official documentation: [https://snapcraft.io/docs](https://snapcraft.io/docs)
*   Flatpak documentation: [https://docs.flatpak.org/en/latest/](https://docs.flatpak.org/en/latest/)
*   openSUSE official documentation: [https://doc.opensuse.org/](https://doc.opensuse.org/)