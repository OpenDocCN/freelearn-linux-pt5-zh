# Chapter 1. Debian Basics for Administrators

"What is the best distribution for my needs? What do I need to know to administer a Debian system? What's different about Debian? What is the best way to handle something specific in Debian? I ran an Internet search on these questions and got millions of results. Now what do I do? Can someone help me?"

The answer to the last question is yes. Answering the others requires a bit of background. This discussion is oriented towards those who are new to Debian. In it, we'll cover Debian's place among the various Linux distributions, project organization (and how that impacts administration), and licensing issues. Those who are already familiar with Debian may wish to skip ahead to the next chapter.

# Linux distributions

Debian is just one of many Linux distributions. Selecting which distribution is best for your deployment can be a rather daunting task. The reason for so many distributions is that the developers or sponsors of each have a different vision of which software should be installed by default, which software is appropriate for particular tasks, and how the system is best administered. This means that selecting a distribution that matches your purpose and preferences will make installation and administration easier.

### Note

Any distribution can be made to reflect an administrator's preferences by installing non-default software or, in some cases, software not native to the distribution software and using non-default configurations. However, selecting an appropriate distribution means less effort is necessary to fulfill the administrator's requirements.

# The three branches

Linux distributions can be broken down into three branches, named from their original distribution or their package managers: SLS, RPM, and DPKG.

## SLS

The **Softlanding Linux System** (**SLS**) distribution, which evolved into the Slackware distribution, is one of the oldest. Distributions in this branch generally made minimal or no changes to the original software packages before including them. Distributions using this format generally provided no native software management and depended on third-party utilities for package management and administration. These utilities were readily available and often included, so this was not necessarily a disadvantage.

### Note

These distributions are also known as Sorcerer/Lunar-Linux/Source Mage (SLS) distributions for the most common distributions using the format.

These distributions are mostly obsolete and not often seen. However, the package format is still used by many software projects.

## RPM

The **Red Hat Package Manager** (**RPM**) was developed by Red Hat in order to provide some structure for software management. It provides all of the customary software management features which are as follows:

*   Software installation, including resolution of software dependencies during the process
*   Various reports on the installed software
*   Software verification and control
*   The ability for users to package their own software so that it can also be managed

Most RPM-based distributions are sponsored by a company that also sells an enhanced version of the distribution and provide extensive, paid support. This also means that unified administrative utilities are available, at least in the paid version, and often in the free version with somewhat reduced features. Many administrators prefer this approach, which makes most common administrative tasks available through a single starting place.

The most common distributions using this format are Red Hat (and the paid version, **Red Hat Enterprise Linux** or **RHEL**) and SuSE (the free version is known as OpenSuSE and the paid version is often referred to as **SuSE Linux Enterprise Server** or **SLES**).

## DPKG or DEB

The **Debian Packaging System** (**DPKG**/**DEB**) was developed about the same time as the RPM, and has the same features, although they are implemented differently. DPKG refers to the original software packaging utility. This has been superseded by more flexible and user-friendly utilities, so this branch is often referred to by the extension used by the package files: DEB (`.deb`). Some distributions in this branch have corporate sponsorship (Ubuntu is the most notable) and thus, have a unified administrative utility, similar to SuSE's YaST for example. Others, such as Debian, depend upon third-party software to fulfill this function.

The most common distributions in this branch are Debian and Ubuntu. Most of the others in the branch, such as Mint and BackTrack, are derived from one of these.

# Other differences

There are a couple of other things administrators should know about how Debian differs from other distributions before we get into details.

One thing to note is that the home of a distribution, if you will, can affect the character of a distribution. For example, Red Hat was originally developed in the United States and, as such, reflects the common usage and preferences of American administrators. SuSE, on the other hand, originated in Germany, and reflects European practices. A concrete example of this is that, for Red Hat, GNOME is the preferred window manager, while SuSE is more geared towards the KDE desktop manager, although both window managers, as well as others, are available in both distributions. The primary issue is that a distribution that matches your preferences will require fewer configuration changes or software package installations to match your administrative style. Information on a distribution's history and intended purpose can be found on the distribution's home page, and frequently in Wikipedia entries as well.

The Debian project originated in the United States, but recruited developers worldwide right from the beginning. Thus, defaults and settings reflect the most common best practices worldwide as much as possible, with individual packages reflecting the interpretation of their developer's particular experience.

The best practice is to select a distribution that best matches your preferences. That way, the default configuration will be closest to what you want, and will require less tweaking to match your administrative style.

Next, distributions fall into two main categories: those with corporate sponsorship, and those without it. Corporate sponsorship usually implies that paid support is available, as well as a paid version of the distribution with extra features. This does not mean that it is not available for distributions without such sponsorship, only that one must find third-parties that provide it rather than finding it in one place.

Debian does not have or accept corporate sponsorship, although it does accept and receive a great deal of corporate support in the form of hardware, developer support, and donations. The idea is that Debian is guided by their social contract and their developers, rather than a particular corporate sponsor. Paid support is available from a number of sources (many who have also contributed), and free support from the developers and user community is available via many support pages and forums, as well as an official bug reporting and tracking site.

Another thing that the lack of corporate sponsorship might imply is a lack of structure or direction. This is not the case for Debian. In fact, there is a very strong structure, with supporting processes and administrative responsibilities, guiding Debian development and release. The main impact is more subtle—Debian is guided by a social contract, and a community of developers committed to the idea of quality, free software, widely available, that runs as trouble-free as possible in as many environments as possible.

With that, let's take a look at the Debian Project itself.

# The Debian Project

Debian is, at its heart, a totally free, volunteer-supported distribution. Unlike Ubuntu, Red Hat, or SuSE, it is not sponsored by any corporation. This does not mean it is any less organized. The Debian project is, in fact, well-organized, with a well-defined government, detailed standards and guidelines, and specified procedures for software release, maintenance, and support.

### Note

The name *Debian* comes from the names of the project founder, Ian Murdock, and his wife Debra.

## The social contract

Above everything else, Debian developers believe in free software, as defined by the Free Software Foundation. In essence, this definition ensures that users have the freedom to:

*   Run the program for any purpose
*   Study how the program works and make modifications
*   Redistribute copies
*   Distribute copies of modified versions

All of this is embodied in the Debian Social Contract, and the **Debian Free Software Guidelines** (**DFSG**), both of which may be found at [http://www.debian.org/social_contract](http://www.debian.org/social_contract). All Debian developers commit to this social contract, which states the guiding principles for the Debian Project, and influences all decisions as to what's included in the distribution and how it is distributed and maintained. Of particular note are the provisions for non-free software, and support in many different computing environments.

The non-free provision not only allows for such software to run on Debian systems, but provides for special Debian repositories for that software which can be released without payment. Such software is, in fact, supported by Debian developers who package and support it. The primary distinction is that it is not a part of the official Debian distribution, due to licensing restrictions. Of course, software that must be paid for can also be run on Debian distributions. It just can't be included in the Debian repositories.

## Constitution

The means of achieving the goals of the Debian Social Contract is outlined in the Debian Constitution. It lays out the formal structure and decision-making process. The project has a full organizational structure that includes Officers, Distribution, Publicity, Support, and Infrastructure divisions, with specific positions and responsibilities. Although Debian is an all-volunteer organization, it is every bit as organized as any large corporate entity.

## Policies

In addition to the organization, there are very comprehensive policy manuals that guide everything about development and release, including the structure of the repositories and archives, as well as a number of related standards documents. Information on all of this is available at [http://www.debian.org/devel/](http://www.debian.org/devel/).

One of the most important effects of these policies, and the organization behind them, is the stability of the Debian distribution. At any one time, there are three main versions of Debian available: stable, testing, and unstable. There are also experimental and backports versions, but they are not complete distributions.

### Note

The experimental version contains packages that are incomplete and not ready to be included in the unstable release. Backports contain newer packages compiled especially for the current Debian stable release.

The unstable version is where active development takes place. Once a package has no "release critical" bugs and works on all supported architectures, it is moved to testing, where it gets additional testing. At some point, the testing contents are frozen in preparation for a new stable release. After stability is verified and all packages satisfy Debian requirements for release, testing becomes the new stable release, and the cycle continues.

Requirements for the stable release are quite stringent. In fact, requirements for testing are strict enough as some have commented that the testing version is more stable than many companies' stable releases. Thus, in Debian, stable means just that. A stable release of Debian is extremely dependable, with a system for releasing security and emergency updates that keeps it so. It provides mission-critical, production quality software for servers and development systems. This is one of the main reasons Debian is used on more production web servers than any other Linux distribution (according to W3Tech, as of January 2012).

As with any advantage, there is a corresponding disadvantage. Debian stable does not always contain the latest, leading-edge software. This is done to ensure the distribution is as mature and crash-free as possible. Of course, it is possible to install newer software under Debian with its required dependencies. In fact, the backports set of repositories contains just such software, pre-compiled especially for use on the Debian stable release. Such packages, however, are not guaranteed to be as stable as those that comprise the official stable release.

## Licensing

As mentioned in *The social contract* section, licensing is one of the central issues in Debian. All of the software in the official Debian distribution is released under any one of several free software licenses, usually some version of the GNU **General Public License** (**GPL**), a Berkeley BSD-style license, or some form of the artistic license used by some Perl developers.

What this means for administrators is that they can run Debian on as many different systems as they wish, without licensing fees, and provide as many copies as they wish to others, without restrictions (well, technically, there are restrictions, but mostly they are requirements that will keep the software free, in the spirit of the Free Software Foundation's definition).

This freedom does not prevent an administrator from running proprietary software in Debian. In fact, such freedom is a part of the social contract. The only restrictions are whatever that software's license states.

### What happened to Firefox?

One of the best examples of how careful Debian is about licensing issues involves the Mozilla suite of software, which includes the Thunderbird mail reader and the popular Firefox browser. A whole chapter could be written on the history of the dispute and the issues involved. However, the basic problem is that the Mozilla artwork is not under a free license as defined by the Debian guidelines. For a while, Debian was allowed to use other artwork, but eventually the Mozilla Corporation withdrew that permission. Some of the reasons this changed included the way the Debian developers compiled the software to comply with their policies and the social contract.

After a long argument, the Debian project determined that the best approach was to rename the software, as allowed by the Mozilla license, so it would remain compatible with the DFSG. Thunderbird in Debian is now called Iceowl, and Firefox is called Iceweasel.

### Note

The names evolved from early discussions when Iceweasel was used to describe a hypothetical re-branded version. The name stuck. Other Mozilla software was renamed in a similar fashion.

The advantages for administrators include the following:

*   The Debian version is unencumbered by non-free licensing.
*   Bugs are frequently fixed by the Debian maintainers more quickly. These patches are passed on to the Mozilla maintainers. This is actually required for all patches to any software by Debian developers by policy.
*   Updates are managed via the Debian packaging framework rather than requiring a separate, proprietary update procedure.
*   The software uses standard Debian system libraries rather than installing Mozilla's separate libraries.
*   The software will run on the various Debian supported, non-Intel architectures. For example, do you have an old IBM z Series server? Debian Iceweasel will run on it. How about an old SG or Sparc workstation? Same story, Debian Iceweasel will run just fine.

Nevertheless, Debian Iceweasel is, for all practical purposes, Firefox. It offers the same look and feel, uses the same plugins, and identifies itself to servers as compatible with Firefox. The same is true for the rest of the re-branded Mozilla software.

### Note

The Plugin Search feature is modified in Debian to seek only free plugins, but I've never found this to be a problem. Non-free plugins can still be installed at the user's own discretion, and will work.

## Repositories

Another result of Debian's licensing policies is the existence of three distinct software repositories:

*   **main**: These are packages whose license conforms to the DFSG
*   **contrib**: These packages have licenses that also conform to the DFSG, but that depend on other packages or libraries that do not
*   **non-free**: These are packages whose license does not conform to the DFSG but that are allowed to be distributed with Debian

Users are free to choose whether to allow software from the contrib and non-free classes to be installed. If it is installed, the users are responsible for knowing and following the appropriate licenses.

Other, non-official repositories also exist, which host software that, for one reason or another, isn't included in any of the official Debian repositories.

## Debian environments

Debian includes as wide a variety of software as possible. As of this writing, the Debian stable distribution contains over 48,000 pre-compiled packages in the latest stable version. According to some counts, this is more than any other Linux distribution. To be fair, many of these are niche applications that do not have a wide user base. But the number of packages is only a part of the story.

The support of many different environments is also a distinguishing characteristic of Debian distributions, and probably one of the most startling. In fact, Debian is unique in the number of different processors supported. At the time of writing, they include both 32-bit and 64-bit Intel and AMD chips, ARM (EABI or little endian version), Intel Itanium, MIPS (both big and little endian), PowerPC (yes, this means it will run on IBM servers!), System/390 (the old IBM architecture), and SPARC. In addition, the Alpha architecture was supported up until Debian 6.0, and there are unofficial ports to other ARM architectures as well as Amtel's RISC chip (AVR32), HP's PA-RISC chip (up until Debian 6.0), the Motorola 68000, IBM system Z, and Hitachi SuperH processors. There is also support for FreeBSD as the primary operating system instead of Linux on Intel 32-bit and 64-bit architectures, and there are other unofficial or experimental non-Linux-based Debian distributions for the GNU Hurd operating system.

This commitment results in a distribution that is extremely flexible, which can be used in a great many environments. Because of this, the Debian developers have chosen not to design a default installation package suitable for most users. A default Debian install (with no optional software selected) includes only the basics. The administrator is expected to select as options, or install later, the appropriate software. This is not difficult as the base system includes everything necessary to easily install additional software.

This contrasts with Ubuntu Linux, which is based on Debian. A basic Ubuntu installation is designed to work out of the box for the majority of users. Thus, it includes more software, making it an appropriate distribution for a new Linux user without extensive knowledge of what may be available, or a preference for exploring what is there, as opposed to wading through packages offered for optional installation. However, this may also result in an installation with unnecessary components. Of course, they may be easily removed, but it is another example of choosing the proper distribution to reduce the administrator's workload.

This is one reason Debian is one of the major players in commercial servers, as only the software and services necessary are installed, which generally leads to better performance and simpler system management. This also means that Debian will run acceptably on older, poorer performing equipment. Note that, in spite of this, it is also most certainly possible to install a wide variety of software, both during and after installation, which will allow a Debian system to fulfill even the most insatiable developers.

## Impact on administration

Now that we have the background on how Debian is organized and developed, the question to be answered is, "how does this affect the administrator."

There are three primary areas which an administrator needs to be aware of. They are as follows:

*   The availability of support
*   The availability of proprietary features
*   Licensing issues

### Debian support

The Debian Project has a very large and well-defined support structure that includes a lot of documentation, a Wiki, mailing lists and newsgroups, websites, and forums. Live help is available on IRC, and there is a well-developed and effective bug tracking system, usable by anyone. It is also possible to contact Debian developers and package maintainers directly, something not always possible with other distributions. These and other available support resources may be found at [http://www.debian.org/support](http://www.debian.org/support).

The thing to remember is that these are volunteers (some of them are, in fact, paid by companies that officially donate their time to the Debian Project). A major release occurs about every two years, and is supported with updates for three years, or about a year after the following major release. The response to bug reports and support requests, in my experience, is quite good, and sometimes faster than paid support. Of course, the quality of advice in places like the forums varies with the experience of the person giving the advice. Nevertheless, this works very well for the majority of users. The fact that Debian releases are extremely stable to begin with helps.

For those who prefer to pay for support, there are a number of companies and individuals that provide such a service. In fact, the Debian website has a page that lists such consultants all over the world.

In a similar vein, although Debian is freely available by downloading from any of the numerous Debian servers and mirror sites, and burning one's own set of installation CDs, DVDs, or Blue-ray discs from the images so obtained, it is also possible to purchase ready-made installation media from third-party vendors.

### Proprietary features

Simply put, there is no paid version of Debian with extra features.

One of the side effects of this is that there is no official Debian-unified administration utility. SuSE, for example, provides YaST, and Ubuntu provides UCC. However, there are many configuration and administration tools available in the distribution, and the various window managers, such as GNOME and KDE gather their administrative menu entries in one place for easy use. Likewise, there are third-party applications that work well on Debian that bring most, if not all, common tasks into a single place with a unified and user-friendly interface.

Probably the most important issue the administrator will run into is the problem of supported hardware. While Debian attempts to support as wide a variety of hardware as possible, some manufacturers don't provide information on their proprietary hardware. Without such information (required to write a driver), if a manufacturer doesn't provide a Linux driver, it won't be supported in Debian.

### Note

There are special cases. Certain Windows XP drivers can be used by Linux if they are available, but they require additional steps to install and activate them.

Actually, this isn't so much a Debian issue as a Linux issue. Some distributions that offer a paid version may include proprietary drivers in the enhanced version. However, in general, if your hardware is supported by Linux, it will work with Debian. There are a number of pages available on the Debian Wiki as well as other sites explaining how to get Linux and Debian to run on many systems with unusual hardware. Furthermore, with the gain in popularity of Linux, many manufacturers are providing the necessary drivers, if not free and with a license that allows them to be included in the base distribution, at least in a format that can be installed and used with Debian.

### Tip

Best practice: check hardware support lists and compatibility sites for Linux before purchasing hardware or installing any distribution.

### Where to find installation help and information

So, how do you find out about supported hardware or what to do in case your hardware isn't supported during Debian installation? Probably the best starting place is the current Debian installation guide. Versions for all supported architectures in different languages are available at [http://www.debian.org/releases/stable/installmanual](http://www.debian.org/releases/stable/installmanual), and they are quite thorough. Section 2.1 covers supported hardware, and includes links to more general Linux hardware compatibility sites. The chapter also links to section 6.4 in the same manual, which covers how to provide missing firmware during installation. Some of the architecture specific manuals mention the Linux Hardware Compatibility HOWTO, but some do not. It may be found at [http://www.tldp.org/HOWTO/Hardware-HOWTO/](http://www.tldp.org/HOWTO/Hardware-HOWTO/). Finally, you may find additional information specific to each supported architecture for the current Debian release at [http://www.debian.org/releases/stable/releasenotes](http://www.debian.org/releases/stable/releasenotes).

# Summary

Debian is an extremely stable Linux distribution that includes a great variety of software that runs in many different environments and on many different CPU architectures. It is free, in the spirit of the Free Software Foundation's definition, and thus may be run freely on as many systems as an administrator desires, without limit or licensing fees. It may be freely copied, modified, and re-distributed. Debian is available from many official Debian servers and mirrors, and it is well supported by an official and well-defined, albeit all-volunteer organization, which provides support via many channels. Paid installation media and support are also available from many third parties.

Debian installations tend to install the minimum services necessary, requiring the administrator to add any additional services necessary after the initial installation. This results in systems that are secure, run faster without unnecessary services, and allows Debian to work satisfactorily on older, less capable systems.

Now that we've covered the basics of Debian, it's time to cover the basics of disk layouts, including the structures used for booting and how to determine the partition layouts.