# Chapter 3. Package Management

You've installed a basic system, now it's time to install additional software. Or, if you've selected some tasks during installation, you want to see what's installed and maybe remove some you won't use. Maybe your boss has asked for a report on what's installed. Or what about security updates?

All of these, and more, are the province of the Debian package management system. In this chapter, we'll cover package managers, software selection and maintenance, how to update your system, and how to set up automatic updating.

### Note

**A note for beginners**

This section assumes that you are familiar with using the root account. Information on accessing root account functions is available at [https://wiki.debian.org/Root](https://wiki.debian.org/Root), and in the Debian reference manual at [http://www.debian.org/doc/manuals/debian-reference/](http://www.debian.org/doc/manuals/debian-reference/). Quick help for commands mentioned in this chapter (as well as most Linux commands) can be obtained by executing `man <command name>` or `info <command name>` from the command line, or using the help button available in most graphical applications. The Debian reference manual also contains more detailed information on using the package manager commands in this chapter.

# Package managers

The Debian package manager started out as a simple, command line utility, **dpkg**, with an additional utility called **dselect** that allowed more complex package selection and dependency resolution via a menu-based, curses interface. Eventually, additional utilities were developed to provide a better interface, better automatic dependency resolution, or both. The current standard package manager is Synaptic, a full-blown GUI application that runs in a graphical window manager, and provides extensive selection and reporting features.

We'll start at the beginning.

## dpkg and dselect

These were the first package management tools for Debian. The `dpkg` command still does all of the work, since all the newer tools use it as a backend. As such, it has all the functions required to install, remove, configure, and report on packages. It is a command line tool.

One of the limitations of dpkg is that it does very little in the way of dependency checking, other than to offer an error message when there is a dependency problem. It requires the user to examine the dependency report and include the necessary packages during installation. Another limitation is that dpkg only works on packages that have already been downloaded. However, both of these limitations are addressed by the dselect utility.

The `dselect` command is a menu-driven utility that provides access to information on packages in the Debian repositories, and also checks and helps resolve software dependencies. This greatly simplifies package selection and installation. Once packages are selected and all dependencies satisfied, either automatically or with user assistance, `dpkg` is run automatically to perform the actual installation.

## Advanced Package Tool

The **Advanced Package Tool** (**APT**) was developed to provide a better command line tool, that provides the download and dependency resolution of dselect without requiring a separate utility for installation. Think of it as an all-in-one command line tool that can select and install or remove packages, and automatically resolve dependencies.

APT is actually a set of utilities that include apt-get, the basic package installation tool, plus several other command line tools with an `apt` prefix that provide additional functions, such as reporting on available software, and other basic local repository maintenance functions.

### Note

The `man apt` command is a good starting point, as it references other man pages for the additional commands. The Debian reference manual also has a lot of information on these and other package management commands

This tool is fast and, except in unusual circumstances, handles dependencies without requiring user intervention. It is the basic tool used for automatic, unattended software installation and updating.

## aptitude

aptitude is a frontend for the APT suite of tools, with added functions that make it a little more like dselect, where it offers finer-grained dependency checking, and resolves dependencies with user assistance rather than autonomously. As such, it is sometimes more successful than dselect or apt-get in resolving dependencies in ways that require fewer major software changes. Like dselect, it is menu-driven (using the curses interface), with command line functions as well. Due to the user assistance sometimes required for dependency resolution, it is less suited to unattended or automatic software updating. However, it will frequently find simpler solutions, involving fewer changes, when compared to APT.

## Synaptic

Synaptic is a package manager with a complete GUI interface and no command line capability. It offers most of the capabilities of aptitude along with many of the repository handling features of dselect. Like the menu-based dselect and aptitude utilities, it provides a software list divided into sections of interest, such as databases, development, editors, and many more, which allow an administrator to browse available software more effectively. It also has search functions which allow easy discovery of packages for specific purposes.

### Tip

Best practices are as follows:

*   For general use – Synaptic
*   For automated installation – APT
*   For dependency resolution in difficult cases – aptitude

# Package selection and maintenance

Debian software is grouped together in a release. All of the software in a release is available as a set of purchased or downloaded media (CDs, DVDs, or new with Debian 7, Blue-ray Discs), or as individual packages grouped in an online repository. While dpkg works only on packages already downloaded (or on media mounted locally), the other package management utilities understand offline media, and local and remote repositories, which must be configured.

## Configuring media or repositories

All of the configuration for media or repositories resides in `/etc/apt`, in a file called `sources.list` and any files in `/etc/apt/sources.list.d` with a `.list` extension. These files can be modified manually using your preferred editor, manipulated by various APT utilities such as `apt-add-repository` or `apt-spy`, or via a menu item in the Synaptic GUI. Details on how each method works are available in various man pages, such as those for `sources.list`, `apt-add-repository` and `apt-spy`, and so on, or in the help files for Synaptic. However, since they all depend on the same configuration files and format, the required entries are all similar.

Each line includes an indicator of whether the repository contains binary packages or source packages (from which binary packages can be built), the location of the repository, the identity of the release, and the sections from which software may be selected. Generally, an entry for the media from which you installed Debian has already been made during the installation process, along with an entry for the online repositories if they were used during installation as well.

All package sources are identified by a URI, described in the `sources.list` man page. The release is identified by its release name (such as **squeeze** for Debian 6, or **wheezy** for Debian 7) or by a generic term such as stable, which refers to whatever the current stable release is.

### Note

The current Debian stable release is Debian 7, code named wheezy, released on 4 May, 2013\. At the time of writing, stable is a synonym for wheezy. Debian releases are named in order to make the mirroring of various distributions easier. The code names to date are all taken from the movie *Toy Story*. This tradition apparently began in 1996 when Bruce Perens, who worked for Pixar at that time, took command of the Debian Project.

Taking all these together, a set of repositories as they might appear in `/etc/apt/sources.list` would look like the following:

```
# deb cdrom:[Debian GNU/Linux 7.0.0 "Wheezy" - Official amd64 \ NETINST Binary-1 20130504-14:43]/ stable main
deb http://ftp.us.debian.org/debian/ wheezy main non-free contrib
deb-src http://ftp.us.debian.org/debian/ wheezy main non-free \ contrib

deb http://security.debian.org/ wheezy/updates main contrib \ non-free
deb-src http://security.debian.org/ wheezy/updates main contrib \ non-free

# wheezy-updates, previously known as 'volatile'
deb http://ftp.us.debian.org/debian/ wheezy-updates main contrib \ non-free
deb-src http://ftp.us.debian.org/debian/ wheezy-updates main \ contrib non-free

```

### Note

**Beginner's note**

Some of the lines in the example are too long for the page and are split into two lines, using the common convention of adding a backslash (\) at the end of the first line to indicate it is continued. In reality, these lines should not be split in the APT sources configuration files.

Let's take the lines one at a time.

The first line begins with a `#`, meaning this entry is disabled. This entry was made by a network installation, wherein a minimal CD is mounted, and basic software is installed to allow the remainder of the software to be installed from online repositories as listed in the later lines. Only the main section is required, as shown at the end of the line.

The next two lines are for binaries (`deb`) and source packages (`deb-src`), to be obtained from an HTTP server ([http://ftp.us.debian.org/debian](http://ftp.us.debian.org/debian)). The release is wheezy, and all three sections—main, contrib, and non-free—will be available. Following the main repository lines are two lines for binary and source package updates. This is where security updates to the stable release are available.

### Tip

Even if you prefer to use media for the release, rather than online repositories, you should include the update repositories, as this is the only way to obtain security fixes that are released as necessary.

Finally, there is a comment, and two lines for what used to be called the 'volatile' repository, and is now just referred to by the release code name followed by `-updates`. This repository contains packages that are routinely updated throughout the life of the release, much more often than the security update repository. Packages that include virus definitions are examples of software included in this repository.

While the various methods of configuring the repositories have slight differences, the same basic information will be required, no matter which method you use. Also, since all of the utilities use the same configuration files and format, information entered, deleted, or modified by one method will be immediately visible to all of the utilities.

All of the previous lines were pre-configured by the installation procedure, and did not need to be modified. However, there are often reasons to modify or add repositories. You may want to add repositories for software that isn't available directly from Debian, or modify the URL to use a different, better performing server, or different access method.

### Note

For example, HTTP is more resistant to network delays or error, while FTP is somewhat faster. Also, not all mirrors support both methods, so if you change servers, you may need to change the access method as well.

There are a number of non-Debian repositories that contain software that is not included in the standard Debian release. Usually, this is due to licensing issues, or because development takes place outside of the Debian Project policies and there is no sponsor to integrate it into Debian. Some of the more useful ones are as follows:

*   **Deb Multimedia**: As it says, this is primarily a multimedia package that can't be included in the normal distribution
*   **Webmin**: This is a web-based system administration software
*   **Oracle**: This provides Oracle Express software
*   **Skype**: This provides the Skype software
*   **MongoDB**: This is a software from the NoSQL MongoDB project

As an example, the following is a `/etc/apt/sources.list.d/webmin.list` file for the Webmin archive just mentioned:

```
deb http://download.webmin.com/download/repository sarge contrib
deb http://webmin.mirror.somersettechsolutions.co.uk/repository \ sarge contrib

```

Generally, sites that offer such repositories will include instructions for configuring the sources list file for their repository.

## The significance of the release name

One of the more subtle changes often made, other than adding non-Debian repositories, has very important implications. Note that the release name in most of the lines from the previous Debian release `sources.list` is wheezy. This means that the packages available through the package managers will always be from the Debian 7 release. Some administrators change the release name to stable. This has both advantages and disadvantages.

One advantage is that, when a new major version is released, your package managers will immediately recognize this and update package lists and dependencies accordingly. The disadvantage related is that major releases involve major changes in package dependencies. While package managers can handle this, such major changes usually result in many new packages being installed to satisfy new dependencies, many old ones being deleted due to changing dependencies or obsolescence, and major version changes that often change the way the software behaves. These changes can be quite disruptive to server operation, or to developers' or users' habits.

### Tip

Best practice to ensure stability is to leave the official release name in place until you are ready to upgrade to the next release. Then change the name in the package manager configuration, and perform a manual upgrade.

## Selecting packages

Once you have the repositories that you want configured, you need to retrieve information about what is in the repositories. This includes not only package lists, but package descriptions, contents, and dependencies. This is done by updating your package cache, after which you can browse, select, install, upgrade, and delete packages.

## Updating your package cache

The package information is updated simply by the refresh menu entry in Synaptic, or the aptitude or apt-get `update` command to update the package information cache. This should be done regularly to ensure that the information you have on available packages is current. Once you have the repositories configured and have updated the package information cache, you can select and install software from any or all of them as desired. There are two basic methods for selecting packages. command line and selection lists.

### Command-line selection

This is the simplest and fastest method to install one or a few packages and their dependencies, but it requires that you know what packages you want to install. There are several utilities that can be used to search package names and descriptions, which will provide you with the means to find the proper package names. The most common of these is `apt-cache`. Once you know what package or packages you want to install, you can use apt-get or the command line format of aptitude or dselect to quickly download and install the packages.

### Selection lists

Both aptitude and dselect provide a basic, interactive interface as well. You can navigate through a list of available packages, classified according to sections, or you can search for packages using a number of criteria. The interface is based on the simple curses library, and can seem cumbersome at times, although it is an improvement over the command line utilities mentioned previously. One advantage of these interfaces (as well as the command line utilities) is that they can be used in a terminal environment and do not require a graphic desktop environment be installed. They are frequently used on high-performance servers where a graphic desktop environment is not installed for security or performance reasons.

On the other hand, Synaptic provides a full GUI interface for browsing, searching, and selecting packages, as well as configuring repositories, selecting installation options, and providing information on available and installed packages. It requires a graphic desktop environment, such as GNOME or KDE, in order to operate.

### Note

Synaptic can be run remotely, over a Secure Shell connection, from a system that does have a graphic window manager installed. Some administrators install Synaptic on servers without graphical desktops and use it in this manner to avoid security or performance issues of graphic environments on the server itself.

## Meta packages

One of the most useful package types is called **meta packages**. These are packages that contain no software themselves, but that require other packages to be installed, and thus provide a unified set of software for a particular purpose. This works because, although no actual software is in the meta package, the package manager will select and install all of the required dependent packages, providing a complete set of features in a single, easy installation step.

Frequently, there are multiple packages with slightly different names that install a slightly different set of dependencies. A good example is GNOME. You can install the GNOME meta package, which will provide a complete GNOME installation, including many extras. Or, you can elect to install `gnome-core`, which provides only the basic desktop environment, and select from any of the additional packages that provide additional features and functions, such as:

*   evolution (e-mail software, similar to Outlook)
*   gnome-documents (document management features)
*   gnome-games
*   gnome-media (multimedia applications)
*   libreoffice-gnome (office suite) and others

One minor problem with meta packages is that there is no easy way to search for them. Many do have 'meta' in their descriptions, which can facilitate some searches, but this is not universally true. The GNOME packages mentioned previously do not follow this convention as of Debian 7\. Nevertheless, they are often fairly obvious, and not too difficult to find.

## A word about dependency resolution

dselect, APT, aptitude, and Synaptic, all provide some form of automatic dependency resolution. In rare cases, a dependency can't be resolved automatically, and user intervention is required. aptitude will calculate alternatives and ask the user to select from them. APT and Synaptic generally require the user to add packages to the command line or selection list manually.

Such problems generally only occur in the testing and unstable releases, where software dependencies are constantly updated and some may not completely resolve until all of the software involved has been updated and placed in the release. However, one common source of this problem occurs in the stable release as well, and is due to a dependency on a **virtual package**.

A virtual package is not the name of an actual package, but the name of a library or function that any one of a number of packages can provide. Since there are usually multiple packages that can satisfy the dependency, the user must choose one to manually install, after which the remainder of the original dependencies can be satisfied automatically. This rarely occurs during a standard upgrade, and almost never during a distribution upgrade, where such virtual packages are selected automatically.

### Note

In general, you will only see this problem, and then rarely, when installing single packages manually.

## Removing packages

Removing packages is also handled by any of the package managers. Something to be aware of, however, is that apt-get and Synaptic do not automatically remove dependencies after the package that depends on them is removed. The command `apt-get autoremove` should be used to do this, no matter which package manager was used for installation and removal. aptitude does this automatically.

# Keeping current

After installing the software you need for your system, it is good practice to check for updates at regular intervals. In particular, security updates are released as soon as possible after a security flaw in any Debian software is identified.

It is easy to update a Debian system. After updating the package cache (see the previous section on *Updating your package cache*) so that it holds current information on all software in the repositories, the Synaptic mark all upgrades menu item, or the apt-get or aptitude `upgrade` command will update all software with any newer versions available.

## Automatic updates

It is possible to perform automatic, unattended updates to a system, but there are some potential problems. Setting it up is quite simple. Just install the unattended-upgrades package. If you aren't asked during installation if you want to enable automatic security upgrades, run the command `dpkg-reconfigure -plow unattended-upgrades`.

Generally, only security upgrades will be automatically installed, which will minimize potential problems, which include modified dependencies and changes that modify how the software is configured or how it operates. It is possible to allow other upgrades by modifying the configuration file in `/etc/apt/apt.conf.d/50unattended-upgrades`. The file is commented to help identify the modifications desired, generally just removing the `//` in front of the lines you want to enable. Note that enabling anything other than security updates can result in errors (when dependency issues are encountered) or system disruption (when the upgrade modifies software behavior or configuration). This is especially true if the stable release name is used, which can result in very major changes when a new stable version is released.

### Tip

Best practice is to allow automatic installation of security upgrades only (use the default configuration). The package information cache for all packages will be updated in any case, so you can manually upgrade the rest of the packages periodically, allowing you to address any unusual upgrade issues as they arise.

# Foreign packages

What do you do when a package doesn't exist in Debian? There are several options. One mentioned previously is to add non-Debian repositories to the repository configuration. After a Synaptic refresh, apt-get, or aptitude `update`, all of the package information in the repository will be available to the Debian package managers.

If, however, the software isn't included in any repository, there are a couple of options available.

### Note

There is actually one additional option: just install such software from its original source. This technique is not recommended because it places the software completely outside of the package management system. Future system upgrades that involve required libraries can cause the software to behave strangely or even break completely, and finding the reason can be quite frustrating. This technique should be used as a last resort only.

## Alien

If the package exists in some other Linux distribution, it can often be converted to a Debian package. This is done using a package called **alien**. Alien provides commands to convert between a number of package formats (including Red Hat RPM, Stampede SLP, Slackware TGZ, Solaris PKG, and Debian DEB).

### Tip

Before using alien, note the warnings in the man page.

In general, the conversion itself is fairly straightforward. Although the results cannot be guaranteed, the converted package often will install okay under Debian. Any problems that do occur are most likely to be caused by different library dependency names, or even differences in software level identification.

All is not lost when this happens, however. Alien can perform a partial conversion, essentially stopping at the point where it has created the temporary directory from which it normally builds the Debian package. You can then go into the directory and make the necessary modifications, and then complete the build manually.

## Manual builds

It is also possible to build Debian packages yourself, either from a partial alien conversion (as previously discussed), or from scratch using the original software. The procedure can vary from simple to complex, depending on what the package is to provide, and is well covered in the Debian maintainer's guide and the Debian policy manual, both available online or as installable packages in Debian, and in many other online resources. A good online starting place is the Debian packaging Wiki page at [https://wiki.debian.org/HowToPackageForDebian](https://wiki.debian.org/HowToPackageForDebian).

### Tip

Manual builds, either from scratch or from a partial alien conversion, are the recommended way to handle software that cannot be obtained in standard Debian format.

Details vary greatly, depending on the actual software involved. Generally, a package is built from source code, but it is also possible to build a package from a binary only software release as well. The general procedure for this is as follows:

1.  Obtain the source (or binary files) and place in an appropriate package building directory.
2.  Create the necessary Debian packaging files, which include additional documentation as necessary, optional script files specific to Debian packages, files to control the package building process, and files required by Debian package managers.
3.  Test the build. If necessary, add patches to correct any problems in packaging, or that are required for the software to compile or run properly in a Debian environment.
4.  Repeat steps 2 and 3 until the final product installs and runs on your distribution.

The packages available to Debian developers are included in the distribution for anyone to use. The primary ones used in building your own packages are:

*   build-essential (packages essential for building Debian packages)
*   dpkg-dev (package development tools)
*   fakeroot (allows users to build as if they were the root user)
*   dh-make (tools to create files in the debian package build directory)
*   debhelper (helper programs for the debian/rules file)
*   cdbs (optional, additional helper programs for the debian/rules file)
*   quilt (debian package patch management)

It is all but impossible to give any general example, as every package will differ in all but the first step. However, there are many good examples and tutorials available online, and the full package source of all Debian packages included in the distribution is available for anyone to examine and learn from.

# Upgrading your system

As mentioned previously, it is simple to update your system. The commands (or menu items) for upgrading your system to the next official release are different from the standard updating commands. The apt-get command `dist-upgrade` or the aptitude `full-upgrade` will perform the necessary special calculations to upgrade to the next major distribution release after the package information cache has been updated (either using the normal `update` command if you have configured the release name as stable, or after changing to the new release name and executing the `update` command). The reason for the special commands is that there are major changes in package dependencies between official releases, and the way some software is configured, as well as the removal of obsolete packages, all of which require special calculations not involved in a normal package upgrade.

### Note

Synaptic can also handle a full distribution upgrade, but instead of a different command, it handles such upgrades when 'smart upgrade' is set in Preferences instead of 'default upgrade'.

## Prior to the upgrade

**Read the release notes!** This can't be emphasized enough. The Debian developers are careful to include all the important details on what has changed between releases, and any special steps required prior to and after the upgrade process.

There are two ways to handle a major upgrade: all-at-once, and a little at a time (which we'll refer to as a partial upgrade). The all-at-once upgrade basically involves a single command that updates all packages. The partial upgrade method involves selecting a group of packages and updating them and their dependencies. This reduces the dependency calculations to a more easily handled subset. Generally, one selects one of the meta packages, such as GNOME or Apache2, or a selected set of packages to update, and uses the `install` command in either apt-get or aptitude, or selects the packages in dselect, aptitude, or Synaptic. After they are upgraded, the next set is chosen and updated, until all packages have been updated, along with any new dependencies or removals required. At some point, after the majority of software has been upgraded, the remainder of the upgrade can be handled all at once.

Choosing a subset of packages is fairly simple. The apt-get `dist-upgrade` command and aptitude `full-upgrade` command will provide information on what will be added, upgraded, and removed and ask you to confirm the selection. At this point, you can tell them not to perform the upgrade, then review the packages proposed for upgrading, and select one or a few to use with the apt-get or aptitude command line `install` command (which upgrades already installed packages). A similar procedure works with Synaptic.

### Tip

One way to ease the upgrade process is to perform a standard package upgrade first. This will perform the simpler, standard package upgrades that don't require major changes in dependencies. Once this is done, the full distribution upgrade will involve fewer packages

## During the upgrade

You've selected the packages to upgrade (or are performing an all at once upgrade), and started the process. The first thing to note is what packages are going to be removed. If one or more of them appear to be packages you need, cross-check them with the packages being installed to see if they are being replaced by a new package with similar functions. If they are not, make a note to follow the upgrade with a separate installation of whatever packages are required.

During a distribution upgrade, the system can generally remain in operation, although there will be slight disruptions when a package requires certain services to be restarted or libraries to be loaded.

### Tip

These disruptions are more severe than a normal upgrade, particularly in cases where a package is removed and replaced with a different one to fulfill the same function, or removed because it is obsolete. Therefore, you may want to inform users prior to the upgrade, and keep system activity to a minimum.

The next thing to watch for is the upgrade notes. Major changes are in how some software works are displayed (and e-mailed to the root account for later checking as well) in order to notify the installer of necessary post-installation steps to be taken.

Finally, when the administrator has made changes to the configuration of a package, the upgrade process will notify him/her of the non-default configuration and ask for help in resolving the differences. This involves leaving the current local version in place, replacing it with the developers' version, or pausing the installation so the differences can be examined and resolved manually.

### Tip

Often there are new options or defaults that should be added to the old configuration. Best practice is to either resolve the changes immediately, or keep the old configuration and cross-check it later with the new default configuration (which is placed in the same directory with a modified name to keep it inactive but available for just this reason).

## After the upgrade

Once the distribution upgrade is complete, there are still a few steps that should be taken. First, if the configuration file issues weren't resolved during installation, now is the time to do this. The new file is in the same place as the old one, with an added `dpkg-new` extension. If the installer selected the developers' version, the old configuration is there with a `dpkg-old` extension. Either way, the administrator can check them for differences and make the necessary changes.

Next, if there are major changes in software operation, any applications that use the software should be modified, or the configuration updated to recreate the old behavior if possible. A good example of this issue is major changes to how PHP works, which often necessitates re-coding web pages that used the changed features or modifying the configuration when it supports operation in a legacy mode. Other major changes may affect users, such as the change from GNOME 2 to GNOME 3, which involves a major change in the user experience. Also, any post-installation steps noted in the release notes should be taken.

# Summary

The package managers in Debian make it easy to upgrade software packages, and even upgrade to a new, major release level. Upgrades don't require major server downtime, although if they involve major software changes that modify how the software is configured or behaves, additional work may be required after the upgrade to return service to normal. Non-Debian repositories can be added so that the package managers can update non-Debian software just as easily as official Debian packages. If software isn't available in Debian format, it can be packaged using the same tools Debian developers use, so that Debian package managers will handle it as well.

Usually after an installation or upgrade, there are additional steps that must be taken. The major one is package configuration, which we will cover in the next chapter.