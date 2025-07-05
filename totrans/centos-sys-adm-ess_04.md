# Chapter 4. YUM – Software Never Looked So Good

It is now time to encompass all that is good with the **Yellowdog Updater Modified** (**YUM**) software repository system, and the array of tools that we can use to make this work for us. YUM provides the power house behind the **Red Hat Package Manager** (**RPM**) software package. Without YUM, you, the administrator, will have to locate the RPM file to install and all of the dependency RPMs; YUM, on the other hand, will do this for you. You will learn valuable lesser known options, create your own RPM files, and add them to your own local repositories. In this chapter, we will go through the following sections:

*   **Managing software installation with RPM files**: We will review the basics of the RPM package management.
*   **Creating your own RPM file**: You will learn how to create an RPM file, learning more in the process of how software installation works with RPMs.
*   **Using YUM**: We will discover the power of YUM and learn the skills needed to help us make the best use of our system and network.
*   **YUM plugins**: We take an overview of YUM plugins and the extensions that can be added to this system.
*   **Creating a YUM repository**: We will create our own repository using both standard software from CentOS and our own custom RPM. Having our own repository means that the installation source is a part of our system and we do not need to rely on external connections to install the software. We could also choose to share this with other hosts on our network to increase the benefits of our own repositories.

# Managing software installation with RPM files

An element of Linux administration that can provide almost a daily dose of entertainment is managing the life cycle of software on your CentOS desktops and servers; this includes installing, updating, and removing software that can take the form of programs, documentation, and drivers, as well as more or less anything that consists of one or more files within Linux. Installing software using RPM files is preferable to install scripts. With an RPM-based installation, we can always query the database for information regarding what is currently installed on the system. Software removal is simplified because we have an inventory of what was added by the package and will require removal. The big downside of RPM management is the *dependency nightmare* we can find ourselves in when we try to install an RPM file that requires other software. We may well be able to locate the required RPM but for certain, there will be more dependencies to trace when we try to install the first dependency package.

A simple demonstration illustrating these issues would be to try and install the Foxit PDF reader; this is a lightweight alternative to the Adobe Reader. If we download the RPM file directly from the Foxit site [http://www.foxitsoftware.com/downloads/](http://www.foxitsoftware.com/downloads/), we can install, or try to install it on our CentOS desktop. The following command will almost certainly fail due to unmet dependencies:

```
# rpm -i FoxitReader-1.1-0.fc9.i386.rpm

```

We can list the dependencies using the following command:

```
 # rpm -qpR FoxitReader-1.1-0.fc9.i386.rpm

```

When I count the dependencies on my system, there are 32 dependencies to resolve directly from this RPM; other dependencies may show when we try to install these packages. This is going to take a long time to install using the `rpm` command alone. If we choose to use the following `yum` command to maintain the installation and are connected to repositories with the required dependency packages, we will find the whole process has just become a lot easier, and I can return to my golf!

```
# yum install FoxitReader-1.1-0.fc9.i386.rpm

```

![Managing software installation with RPM files](img/5920OS_04_01.jpg)

Reviewing some more of the basic RPM command-line syntaxes, we can list the following commonly used options:

*   Install software from the RPM file: `rpm -i <example.rpm>`
*   Uninstall RPM: `rpm -e <example>`
*   Update the existing RPM or install if it is not installed. Additionally, display the progress as hash marks or percent: `rpm -Uvh <example.rpm>` or `rpm -Uv --percent <example.rpm>`
*   List all the installed RPM-based software: `rpm -qa`
*   List the RPM to which a file belongs: `rpm -qf /etc/hosts`
*   List all files from an installed package: `rpm -ql <example>`
*   List all files from an RPM file: `rpm -qpl <example.rpm>`
*   To view the version of RPM: `rpm --version`

If you need to locate the RPM database, you will be able to find it in the `/var/lib/rpm` directory.

You may have noticed that when referencing an installed package, the RPM name can be used alone, whereas when using the `rpm` command with an RPM file, the complete filename and extension has to be used. The filename is made up from the following components:

```
<package><version><release><architecture>.rpm

```

For example, let's look at the RPM file for the **Z shell** (**ZSH**) package. The filename is:

```
zsh-4.3.10-7.el6.x86_64.rpm

```

*   Package name: `zsh`
*   Version: `4.30.10`
*   Release: `7-el6`
*   Architecture: `x86_64`

To verify this, query the package file using the `-i` or *information* option. In the following command, we use `-q` for query, `-p` for package, and the aforementioned `–i`.

```
$ rpm -qpi zsh-4.3.10-7.el6.x86_64.rpm

```

# Creating your own RPM file

Even though I am advocating the use of YUM to manage software installations, we are still going to need RPM files. YUM is totally dependent on the underlying RPM files and infrastructure; the RPM file remains the software package that the `yum` command will install and these RPM files are instilled at the heart of the CentOS software management.

You might be able to cast your mind back to [Chapter 2](ch02.html "Chapter 2. Cold Starts"), *Cold Starts*, of this book when we were investigating Plymouth themes during the boot process; we are now going to create our own simple theme to brand our desktop or server with a corporate wallpaper during the system startup and shutdown. Once we have created the theme, the easiest way to install it across many systems is to distribute the theme as an RPM file. Later in this chapter, we will add the RPM to a YUM repository.

## Creating the Plymouth theme

Firstly, we must create the theme, and with that completed, we shall be able to package it as an RPM file. Our theme will have three files contained within one directory. The directory needs to be added, along with the three files that it will house, into the `themes` folder of the target CentOS system, `/usr/share/plymouth/themes`.

The top-level directory that contains the three files for the theme is normally named after the theme to which it belongs. We are creating the `tup` directory to represent our theme as follows:

```
# mkdir /usr/share/plymouth/themes/tup

```

The theme, **The Urban Penguin** (**tup**), will contain three files:

*   `800.png`: This is the wallpaper that will be shown as the splash screen. This has to be a PNG file.
*   `tup.plymouth`: This is the master theme file that contains the theme manifest.
*   `tup.script`: This is the action that the theme will run. This is the simplest form of a script-based theme, so we need this instruction file.

### tup.plymouth

The `tup.plymouth` text file is the main theme file.

```
[Plymouth Theme]
Name=tup
Description=The Urban Penguin
ModuleName=script
[script]
ImageDir=/usr/share/plymouth/themes/tup
ScriptFile=/usr/share/plymouth/themes/tup/tup.script
```

As we can see in the preceding example, `ModuleName` being set to `script` means that we will need to ensure that we have the RPM `plymouth-plugin-script` listed as a dependency for our own RPM. We will build this into the RPM using the required directive during the build.

### tup.script

The Plymouth script module will execute the `tup.script` file when the theme is running. The script that we create will just have one sprite. A sprite places elements on the boot splash, or the wallpaper that is seen during the system startup and shutdown. The wallpaper will scale to the size of the screen and take care to include the semicolons at the end of each line. The image used here is the `800.png` file but you can use any PNG file that you add to the `theme` folder. Consider the following as a sample script:

```
wallpaper_image=Image("800.png");
screen_width=Window.GetWidth();
screen_height=Window.GetHeight();
resized_wallpaper_image=wallpaper_image.Scale(screen_width,screen_height);
wallpaper_sprite=Sprite(resized_wallpaper_image);
wallpaper_sprite.SetZ(-100);
```

## Creating the theme RPM

To build the RPM, we will work as a standard user, either through your own account or an account that is reserved specifically to build RPMs (if creating the account, just add the new account with standard privileges as a local or network account). Initially, we will need root privileges, though, to install the required packages for the build environment:

```
# yum install -y rpm-build rpmdevtools

```

The previous command will install the tools that we will use to create the RPM. Using YUM, we will need to be able to connect to a repository that holds the software, but we do not need to be concerned with the location. With this in place, we can revert to the account that we will use to package the RPM. When logged in, we must make sure that we are in our home directory where we will create the top-level directories as follows:

```
$ cd
$ rpmdev-setuptree

```

This will create a `rpmbuild` directory with five subdirectories, which can be seen in the following screenshot:

![Creating the theme RPM](img/5920OS_04_02.jpg)

These directories become the working directories when building the RPM.

To begin, we will create the directory structure that we require below the `SOURCES` directory:

```
$ cd ~/rpmbuild/SOURCES
$ mkdir -p plymouth-theme-tup-1/usr/share/Plymouth/themes/tup

```

The top-level folder that we create is based on the name and version number that will represent the final RPM. The name will be `plymouth-theme-tup` and the version will be `1`. The following directories represent the structure in the target filesystem; we need to target the `/usr/share/plymouth/themes/tup` directory.

With the directory in place, we can now copy the three files that constitute the theme into the newly created directory:

```
$ cp /usr/share/plymouth/themes/tup/* \
 plymouth-theme-tup-1/usr/share/plymouth/themes/tup 

```

Now we need to create a gzipped archive of the folder structure; Within the `~/rpmbuild/SOURCES/` directory, we can run the following command to create the archive:

```
$ tar -czvf plymouth-theme-tup-1.tar.gz  plymouth-theme-tup-1/

```

The final stage before the build stage is to create the instruction or specs file. We can create this within the `~/rpmbuild/SPECS/` directory using our preferred text editor. We will call the `~/rpmbuild/SPECS/tuptheme.spec` file:

```
Name: plymouth-theme-tup
Version: 1  
Release: 0
Summary: TUP Corporate Theme 
Group: System Environment/Base
License: GPL
URL: http://theurbanpenguin.com   
Source0: plymouth-theme-tup-1.tar.gz
BuildArch: noarch
BuildRoot: %{_tmppath}/%{name}-buildroot
Requires: plymouth-plugin-script
%description
Corporate Plymouth theme displaying the TUP wallpaper

%prep
%setup -q

%install
mkdir -p "$RPM_BUILD_ROOT"
cp -R * "$RPM_BUILD_ROOT"

%clean
rm -rf "$RPM_BUILD_ROOT"

%post
echo ..
echo "Adding theme…"
%files
%defattr(-,root,root,-)
/usr/share/plymouth/themes/tup/800.png
/usr/share/plymouth/themes/tup/tup.script
/usr/share/plymouth/themes/tup/tup.plymouth
```

The file may seem complex, but most of the settings are self-explanatory. What's more? This can be used as a template for any RPM that is delivering text files, scripts, documentation, or pretty much any content that does not need to be compiled. If there were source files written in C that needed to be compiled, we would need to add a `%build` section.

For more information on the SPEC file, refer to the documentation that can be found at [http://www.rpm.org/max-rpm/s1-rpm-build-creating-spec-file.html](http://www.rpm.org/max-rpm/s1-rpm-build-creating-spec-file.html).

Now let's build us an RPM!

```
$ cd
$ rpmbuild -bb rpmbuild/SPECS/tuptheme.spec

```

The RPM file will be created in the `~/rpmbuild/RPMS/noarch/` directory. We can view this from the output of the command tree, as seen in the following screenshot:

![Creating the theme RPM](img/5920OS_04_03.jpg)

We can verify that the files have been added correctly to the RPM. These are the three files that we need for the theme that we looked at earlier in the chapter and referenced within the `%files` section of the SPEC file:

```
$ rpm -qpl ~/rpmbuild/RPMS/noarch/plymouth-theme-tup-1-0.noarch.rpm

```

We should also verify the dependency that we added to the SPEC file using the `Requires` directive:

```
$ rpm -qpR ~/rpmbuild/RPMS/noarch/plymouth-theme-tup-1-0.noarch.rpm

```

The RPM cannot be installed now without the script plugin being present. If installed via YUM, then the dependency will automatically be resolved and installed as required.

## Using YUM

As we have already seen with the Foxit PDF reader, the management of the RPM dependency requirements is greatly simplified by using YUM; almost certainly the ongoing software management of your CentOS will be based on YUM in the same way that **Advanced Packaging Tool** (**APT**) is used on Debian-based systems.

To install software with YUM, you do not need to know the filesystem path or even where the RPM file is. The required packages are located within software repositories, the location of which is configured within `/etc/yum.repos.d/` directory. Any file created in this directory should have the `.repo` extension.

On a standard build, these files will be directed to Internet-based software repositories; however, you can configure repositories locally on your own network, filesystem, or CD-ROM. This will be discussed later. For now, we will concentrate on the YUM commands:

*   `yum install nmap`: Install the `nmap` package from a repository. We know this will be from a repository rather than a local file. For a local file, include the full filename and the `.rpm` extension and, if required, the path to the file. Earlier in this section, we saw how we could install the Foxit PDF reader RPM using YUM.
*   `yum list nmap`: This will show the package and if it is installed or available. The output will also list the repository from which it was installed or is available from.
*   `yum list installed`: This will list all the installed.
*   `yum check-update`: This will show the available updates.
*   `yum update nmap`: This will update just `nmap` if an update is available.
*   `yum update`: This will update all the software on your system.
*   `yum remove nmap`: This will uninstall the `nmap` package.

## YUM plugins

Although YUM is one of the most used packaging systems, many users and administrators are not aware of the functionality that is available with the addition of plugins.

Using CentOS, and reading this book, you will now be very familiar with using the `yum` command to install packages and update your system. We have already seen how much we extend the RPM system with YUM, but that is no reason to ignore how much more is available.

Plugins are Python scripts or programs that extend the features that YUM offers. You will find the plugins located in `/usr/lib/yum-plugins`, and you will find that their configuration files reside under `/etc/yum/pluginconf.d/`. To be able to use any plugin, the plugins directive must be enabled in the YUM configuration file, `/etc/yum.conf`, as follows:

```
plugins=1

```

To search for the available plugins, we can use the following command:

```
$ yum search yum-plugin

```

On a standard build, CentOS will include the fastest mirror plugin, among others. This plugin performs very much as you may think, locating the fastest mirror repository in the time you download the packages. If enabled, this plugin is evoked automatically with the use of the `yum` command.

If you need to disable an individual plugin, rather than all plugins, then you will need to ensure that the enabled directive within the plugins' configuration file is set to `0`. For example, to disable the fastest mirror plugin, you will need to edit the `/etc/yum/pluginconf.d/fastestmirror.conf` file and configure the enabled line to read as follows:

```
enabled=0

```

The fastest mirror plugin is disabled. To enable the plugin, the settings should read as follows:

```
enabled=1

```

Another command plugin that is often installed is the `security` plugin. This is designed to be used in conjunction with the `yum` update commands:

*   `yum --security check-update #List only security updates`
*   `yum --security update #Install security updates only`

# Creating a YUM Repository

The reasons to create a YUM repository are many. You can imagine a situation where you have more than one server on your network. It would make sense that the software is retrieved locally, rather than having all servers cross the WAN to access packages. The same reasoning scales to where CentOS desktops are common place. Centralizing software distribution is an absolute requirement, standardizing software being used and to ensure that your support team only has to support the single version of a package.

Building a local repository in a virtual machine that is just used for testing and development also makes great sense, removing the need to be connected to the network to install software packages. I ensure my classroom virtual machines always have a local file-based repository so that the system can be used as a discrete identity without relying on networking or external components.

![Creating a YUM Repository](img/5920OS_04_04.jpg)

For this little foray, we shall create a local repository using the RPMs from the product DVD and we will use the command-line tool `/usr/bin/yumdownloader` in order to add additional required RPMs as follows:

1.  Firstly, we will create a local directory to act as our local repository; to keep this simple, we just create a directory at the root of the filesystem and add RPMs directly into this folder:

    ```
    # mkdir /repo

    ```

2.  With the product DVD mounted, we can populate the directory with the RPMs from the DVD:

    ```
    # rsync -av /media/<volume name>/Packages/  /repo

    ```

3.  We can add in the RPM what we created for the Plymouth theme earlier in this chapter:

    ```
    # cp /home/user/rpmbuild/RPMS/noarch/plymouth-theme-tup-1-0.noarch.rpm  /repo

    ```

4.  With the network connectivity, we will download, not install, additional RPMS that are not on the DVD, which we require in our repository. The easiest way to achieve this is with the `yumdownloader` command:

    ```
    # cd /repo;  yumdownloader --resolve kernel-source

    ```

5.  The `--resolve` option will ensure that dependency packages are also downloaded. By default, they are downloaded to the current directory, so take care to move them into the `/repo` directory first:
6.  We need to install the `createrepo` package:

    ```
    # yum install createrepo

    ```

7.  We are now ready to create the repository metadata that makes the file directory a repository:

    ```
    # createrepo /repo

    ```

We have now been able to create a local repository; this will not be used until it is referenced by the YUM client. This will be achieved by the creation of a file with an extension of `.repo` within the `/etc/yum.repos.d/` directory. For now, we can bask in the success of our creation. The real importance here is to take away the knowledge gained from using `yumdownloader` and the power that it brings to the repository creation being able to combine selected packages from multiple sources into the custom repository for you and your clients.

## /etc/yum.repos.d/

Client systems need to be told about the location of their software sources or repositories and this is via the use of the `.repo` files. Although it is possible to define all of your repositories within the single file, `/etc/yum.conf`, it is more reasonable to add them as separate files in the directory as is detailed from the following excerpt from `yum.conf`:

![/etc/yum.repos.d/](img/5920OS_04_05.jpg)

The name of the file does not really matter, but the extension has to be `.repo`. We will create a file called `/etc/yum.repos.d/local.repo`:

```
[c6-local]
name=CentOS-6.5 - Local
baseurl=file:///repo
gpgcheck=0
enabled=1
```

The square brackets define the short name of the repository, and the name directive is really more of a description. The `baseurl` directive uses three forward slashes: two for the method of access, the file protocol in this case, and then the filesystem path slash itself. We have disabled public key checking (`gpgcheck=0`), but we could additionally sign the repository metadata should we wish to. The repository is enabled.

If this is the only repository that we would like to use, then we can move the existing repository files to a backup directory; this is probably easier than disabling the repositories by adding the `enabled=0` directive to each repository. Often, there is more than one repository defined per file, so the task is not simple. As I say, the easiest way is to move them to another directory and then add them back, should you later need access to the external repositories.

Having now defined the local repository within the `/etc/yum.repos.d/local.repo` file, we now effectively have access from YUM to all the RPMs that we have added to our own directory. Should we need to share this across a network to the other hosts on our internal network, it is a simple matter of adding a web server to provide access to the repository. Clients will then use `http:// protocol` as the method of access.

[Chapter 8](ch08.html "Chapter 8. Nginx – Deploying a Performance-centric Web Server"), *Nginx – Deploying a Performance-centric Web Server*, will see us perusing the NGINX web server, and among other things we will learn how to direct a web server **Universal Resource Identifier** (**URI**) to this repository.

# Summary

Congratulations! You have passed another milestone in reaching the end of this chapter and coming one step closer to the temple that is reserved for the most elite of administrators. We will now embark on a retrospective tour of this chapter in which we have gained some inside knowledge enabling us to manage software on our systems a little more effectively. Beginning with raw RPM files, we were able to see the strength in the management and inventory benefits that they provide, but the annoyance is in having to locate their dependencies. Liking the idea of RPMs, we ventured out to create our own RPM file to distribute and install a Plymouth theme; a theme that we have created to help brand our systems and reinforce our identity.

It was not long before that we then became more familiar with the YUM repository management system, which allows us to make the best use of the RPM inventory system but combining it with the ability to automatically resolve dependencies. This chapter culminated with the creation of our own local repository creating a standalone system being able to install software independent of any network connectivity.

In the next chapter, we will gain valuable skills to maintain service and process management on the CentOS 6.5 system. We will ensure understanding of the legacy System V services scripts as well as the new upstart system for service control. In addition, we will investigate tools that allow us to effectively manage processes running on the CentOS host.