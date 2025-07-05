# Appendix A. Building OpenLDAP from Source

In this appendix, we will walk through the process of building OpenLDAP from source code. We will begin by configuring our Linux platform to compile OpenLDAP. Then we will configure, compile, and install OpenLDAP. Compiling OpenLDAP might sound daunting, but it is not, and I have attempted to provide instructions straightforward enough that even those without experience of C will be able to quickly compile from source.

# Why Build from Source?

Many Linux and UNIX distributions are slow to migrate from one version of OpenLDAP to another. The reasons for this are open to speculation, but one reason may be that distribution maintainers are reluctant to quickly adopt new versions of software when it already performs well, is integrated with other services, and performs a task that is security-sensitive and functionally central to many organizations. OpenLDAP, providing authentication services, is just such a service.

Because of this reluctance, you may not find the latest and greatest version of OpenLDAP included in your Linux or UNIX distribution of choice. If you need (or want) the newest features that OpenLDAP has to offer, you may want to fetch a clean copy of the source code and build from scratch.

# Getting the Code

To get the latest version of the code visit the official OpenLDAP website at [http://openldap.org](http://openldap.org). This site is hosted by the **OpenLDAP Foundation**, a non-profit group that governs and oversees the OpenLDAP project.

On the home page you will find a link to the current release in a highlighted box in the lower right-hand corner,as shown in the screenshot:.

![Getting the Code](img/1021_AppA_01.jpg)

You can download the most recent stable version directly from there, or you can visit the download page (listed as **Download!**) in the center column of the table of links to find other versions (past versions, current experimental and beta versions, and so on).

# The Tools for Compiling

Whenever you build an application from source code, you need the right set of tools and libraries. OpenLDAP is no exception. Thankfully, OpenLDAP is a little lighter on requirements than some server applications out there.

Compiling is done on the command line, so you will need to open a terminal or otherwise gain access to the shell.

## Build Tools

You will need the standard tool chain for working with C and C++ applications; a C compiler, a linker, and a make program. Fortunately, all of these come as standard with almost every Linux distribution available. You can test your system for the appropriate tools using the `which` command, which will tell you where the tools are located on your filesystem (assuming they are in one of the directories listed in your `$PATH` environment variable).

Here's a quick example of how you can check to see where the tools are and what the current version of each tool is. My system is Ubuntu Linux 6.06\. Version numbers on your own system may vary. That's okay. OpenLDAP should compile on all modern Linux distributions, and probably on all modern UNIX distributions as well.

```
$ gcc --version
gcc (GCC) 4.0.3 (Ubuntu 4.0.3-1ubuntu5)
Copyright (C) 2006 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
$ which ld
/usr/bin/ld
$ ld --version
GNU ld version 2.16.91 20060118 Debian GNU/Linux
Copyright 2005 Free Software Foundation, Inc.
This program is free software; you may redistribute it under the terms of the GNU General Public License.  This program has absolutely no warranty.
$ which make
/usr/bin/make
$ make --version
GNU Make 3.81beta4
Copyright (C) 2003  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i486-pc-linux-gnu
$ 
```

In each case I used the `which` tool to know where the tool was located. The programs I checked were `gcc`, `ld`, and `make`—the compiler, linker, and make program (respectively). As long as some path is returned, this indicates that the tool is installed. If no command is found, `which` returns without any output. Thus, if I searched for a fake command, `blah`, the output would look like this:

```
$ which blah
$
```

Thus, if you run `which` for any of the programs (`gcc`, `ld`, or `make`), and get no output, it indicates that you do not have the required tool.

### Note

On some UNIX systems, the GCC compiler (`gcc`) may not be present, but another C compiler may exist. The *de* *facto* name for C compilers is `cc`, and if `which` `gcc` yields no result, you may want to try `which` `cc`.

After the `which` command in the given example, I ran each command with the `--version` flag (with *two* dashes before `version`) to tell me which version was installed. The `--version` flag is a GNU standard, but non-GNU programs (such as other versions of `make` or `cc`) may not support it.

The next thing to do is set a couple of environment variables that will provide some basic settings for the given tools. While there are many options that you can provide to your tools through environment variables, here we will just provide the basics for building OpenLDAP.

### Note

Some Linux and UNIX distributions set the necessary environment variables for you. In such cases, it is almost always better to use the already-defined environment variables, which are often optimized specifically for your system, rather than the generic ones we will be setting now.

To find out if you have the necessary environment variables, run the `env` command (with no arguments) and check the output to see if `CC`, `CFLAGS`, and `PATH` are defined.

One way to set environment variables is with the `export` command. When you use the `export` command, the environment variables will be stored for the duration of your shell session (in other words, until you exit from the shell or close the terminal window). Here we will set the necessary environment variables using `export`:

```
$ export CC=gcc
$ export CFLAGS="-O2"
```

The first export sets the `$CC` environment variable to `gcc`. The make program will use this to determine which compiler to use. (If you are using the `cc` compiler instead, then adjust the example to point to `cc` instead of `gcc`). Note that when you *set* an environment variable, you do not use the dollar sign (`$`) before the variable name. When you *reference* the variable however, you will need to include the dollar sign.

The second line sets the `$CFLAGS` variable. The `$CFLAGS` variables are the options that get passed to the compiler during compilation. In this case, we are passing it the option -`O2` (that's a captial letter O, not a zero). This tells the compiler to use level 2 optimization when compiling the code.

The `$PATH` environment variable should also be set. However, by using the `which` command to see where our tools were, we have already verified that the necessary directories (that is, the directories that contain our tools) are specified in the `$PATH` variable.

If you are dealing with a non-standard system or non-standard builds of any of the libraries, or if you are interested in passing some other options to the build tools, you may also have to use some additional environment variables. You can use `$CPPFLAGS` to pass options to the C preprocessor (`cpp`, part of GCC). Likewise, you can pass the linker (`ld`) options with the `$LDFLAGS` variable. Finally, if you have libraries (compiled modules of code used by other applications) that are stored in non-standard places, you can use the `$LIBS` variable to specify the location of these libraries. If you need to use the variables you should consult the documentation for the tools and libraries.

At any point you can check your environment variables with some simple commands. The `env` command (executed with no arguments) will list all of the environment variables currently defined, as well as their values. You can also check an individual environment variable with the `echo` command. Simply type `echo`, followed by the name of the environment variable to display the value of that environment variable:

```
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/bin
/X11:/usr/games
```

In this example `echo` `$PATH` shows the list of directories the shell searches to find programs. As you may recall the `which` command, when run, printed out the location of the specified tool. To find tools it searched each of the directories specified in the `$PATH` variable.

At this point we are ready to move on to the next step: installing the necessary dependencies.

## Installing Dependencies

Dependencies are packages that OpenLDAP will require to compile and to run. Installing these dependencies will vary from platform to platform (and from Linux distro to Linux distro). Here, I will use the Debian tools (included in Ubuntu Linux) to install packages.

OpenLDAP requires standard C libraries, regular expression libraries, and the Berkeley DB 4.2 (or later) libraries. These are almost always included in modern Linux distributions. In addition to the libraries, the header files are also required. Often these are stored in separate packages (usually called DEV packages). To install, for example, the Berkeley DB 4.2 development package in Ubuntu, you can execute the following command from the command line:

```
 $ sudo apt-get install libdb4.2-dev

```

This will fetch and install the required package.

There are various other packages that are useful to install, and we will need them in order to build all of the features that we use in this book. You will need to install:

*   OpenSSL (for SSL and TLS support)
*   SASL 2 (for SASL authentication support)

If you are interested in storing your directory in a relational database engine, such as MySQL or Oracle, you might also want to install iODBC2 (for database backend support).

All of these packages are common on modern Linux system. Make sure that the packages are installed, and that the DEV (or `-dev`) add-ons for each of these is installed as well. In Ubuntu 6.06 these can be installed with one (rather long) command:

```
 $ apt-get install libssl0.9.8 libssl-dev libiodbc2 libiodbc2-dev 
 libsasl2 libsasl2-dev

```

Other distributions may use different installers, and even different package names, but you should have no problem finding them based on the bullet list of names that have been provided.

### Note

OpenLDAP includes many optional modules that provide additional functionality (such as debugging or integration with other services). These modules are not covered in this book, though you may choose to explore them on your own. Some of these modules require additional libraries. Consult the OpenLDAP documentation included in the source code for more specific information.

At this point you have all the tools and requirements necessary for building OpenLDAP. Now we move onward to the actual compiling process.

# Compiling OpenLDAP

In the last section we prepared all of the tools and libraries for building OpenLDAP. In this section we will configure, compile, and test OpenLDAP.

First, we need to get our OpenLDAP server source code moved into a temporary directory for building. Copy the `openldap-2.3.x.tgz` file into the appropriate directory and then unpack the file:

```
$ mkdir build/
$ cp openldap-2.3.37.tgz build/
$ cd build
$ tar -zxf openldap-2.3.37.tgz 
```

Here, I created a new directory (called `build/`), copied the OpenLDAP source code archive into the new directory, changed the working directory to `build/`, and then unpacked the file with the `tar` utility (the flags `-zxf` instruct tar to uncompress (`z`) and extract the contents (`x`) of the file (`f`) `openldap-2.3.37.tgz`). Once this is done, the `build/` directory should contain a directory called `openldap-2.3.37`. Use `cd` to change to that directory: `cd` `openldap-2.3.37`.

## Configuring

Now we need to run the configuration script to prepare the source code for compiling. This script determines how OpenLDAP will be built, and what options will be enabled or disabled by default. OpenLDAP is very configurable, and there are many different options from which to choose. To see the complete list of options, you can run the configuration script with the `--help` flag:

```
 $ ./configure --help

```

This command will print out a list of every option available for configuring OpenLDAP. It will also indicate whether or not the option is on by default. For example, the first few lines of the SLAPD database backend section looks like this:

```
SLAPD Backend Options:
    --enable-backends     enable all available backends no|yes|mod
    --enable-bdb          enable Berkeley DB backend no|yes|mod [yes]
    --enable-dnssrv       enable dnssrv backend no|yes|mod [no]
```

We can see that each option has three possible states: yes, no, and mod (which builds the component as a pluggable module instead of building it into slapd). Of those listed, only the flag `--enable-bdb`, which enables the Berkeley DB backend, is on by default.

For the most part, the defaults are good. All of the crucial options are turned on by default. However, there are a few additional modules discussed in this book that are not on by default, and we will want to turn them on manually. They are:

*   `--enable-ldap`: Enables the LDAP backend storage mechanism (see Chapter 7)
*   `--enable-ppolicy`: Enables the password policy overlay (see Chapter 6)

If you do not plan to use the ODBC database backend, you can add `--enable-sql`, but you will need to make sure that you install the iODBC2 packages discussed in the previous section.

### Note

By default, OpenLDAP will be installed (with the final make install step) into subdirectories at `/usr/local`. This is the recommended place to put applications and libraries that are "local" applications. Packages that are not distributed as standard pre-configured applications (like deb or RPM packages) are considered local packages. If you want to put the package somewhere else, use the `--prefix` and `--exec-prefix` flags.

Now we are ready to run the configuration command:

```
 $ ./configure --enable-ldap --enable-sql --enable-ppolicy

```

This will kick off an evaluation process that may take several minutes. The configuration script will systematically evaluate your system settings, determining what tools you are using, how it should build, and whether or not the system has all of the necessary libraries.

If the configure process terminates with an error, it will indicate why it failed. Usually, this failure will indicate that one of the required libraries or tools is not present. For example, if it dies with an error stating that `sql.h` is missing, this indicates that the iODBC2 header files (from `libiodbc2-dev` in Ubuntu) were not found. This usually indicates that they are not installed at all, though it may also indicate that they were installed in a non-standard location.

Some missing libraries will not stop configure from running. Such packages will generate errors instead of warnings. Two examples of this are the OpenSSL libraries and the SASL libraries. Once the configuration script has completed, scroll back through the results and make sure there are no lines that look like this:

```
configure: WARNING: Could not locate TLS/SSL package
configure: WARNING: TLS data protection not supported!
```

or

```
configure: WARNING: Could not locate Cyrus SASL
configure: WARNING: SASL authentication not supported!
```

If you see these you will probably want to make sure the appropriate packages (remember the DEV packages) are installed, and then re-run the `./configure` script.

Once the configuration script has run through, and there are no warnings or errors, you are ready to build OpenLDAP's source code.

## Building with make

Building with make is a two-step process. First, the auxiliary libraries must be built, then the main tools and servers must be built. Fortunately, all this hard work can be done in one short command:

```
 $ make depend && make

```

This will compile all of the libraries (`make` `depend`) and then, if the first part was successful, it will run the main build (`make`). Compiling may take a long time.

Usually, the configuration script makes sure everything is in order before the main compilation begins. On rare occasions though, one of the two make commands may fail. If this happens you will have to evaluate the error message and determine what steps to take to fix the problem. In most cases the problem has to do with an unsatisfied dependency—some package or tool that OpenLDAP requires is not installed, and (for one reason or another), this gap was not noticed by the configuration script.

Sometimes the documentation included with OpenLDAP (`README`, `make`, and the documentation in the `docs/`, `libraries/`, and `servers/` directories) will point out possible problems.

### Note

If the make fails and you cannot find the problem, your best bet may be to search the OpenLDAP mailing list archives (visit [http://openldap.org](http://openldap.org)) or, if all else fails, subscribe to the mailing list and ask about the problem there.

Once the compiling process ends, it is a good idea to run the automated testing procedure to make sure that the code was built correctly. This is also done with `make`:

```
 $ make test

```

Because the test includes frequent programmatic delays and performs dozens of tests, this process may take several minutes to complete. When it is done, review the output and make sure there are no errors. Note that some of the test will be skipped because we did not compile OpenLDAP with all of the possible options turned on. Skipped tests are normal and are nothing to worry about.

Now we are ready to install our fresh new OpenLDAP server.

# Installation

Installing is done with one additional command:

```
 $ sudo make install

```

On some versions of Linux or UNIX, instead of using `sudo`, you will need to switch users (`su`) to root and run the make install command as root: `su` `-c` `‘make` `install'`. You will be prompted to enter the password for your account (or, if you use `su` instead of `sudo`, the root password). Once you have correctly entered the password, the necessary OpenLDAP files will be copied to subdirectories of `/usr/local`.

On some systems, the directories that contain local executable files (`/usr/local/bin` and `/usr/local/sbin`) are not included in the `$PATH` environment variable. As a result, simply typing an OpenLDAP command at a command line may return an error. One way to get around this problem is to type in the entire path to the command:

```
 $ /usr/local/sbin/slapcat

```

But this can be tedious. You can also append the appropriate paths to your `$PATH` environment variable. Then you will be able to simply issue the command without specifying the absolute path to the command:

```
 $ export PATH= /usr/local/bin:/usr/local/sbin:$PATH
 $ slapcat

```

In this example the export command re-sets `$PATH` for the current session. So the variable `$PATH` is assigned the values `/usr/local/bin`, `/usr/local/sbin`, and the contents of the current `$PATH` variable (which likely contains `/bin`, `/sbin`, `/usr/bin`, and other directories). Order is important. When the shell is searching for a command (`slapcat`, in the given example), it will search from the first directory in `$PATH` on to the last directory. As soon as it finds a match, it will stop searching. So, for example, if there were two `slapcat` commands, the shell would use the first one it found. In our case, it is best to put the two `/usr/local` directories early in the path just in case an older version of LDAP is installed elsewhere on the file system.

Usually, the `export` command should be added to the shell configuration file (for example `~/.bash_profile`) so that the additional path information is automatically added every time you start a shell session.

You are now ready to configure the new version of OpenLDAP.

# Building Everything

In the build mentioned in the previous section we compiled only the basics. This gets us what we need to run just the basics. But there are lots of OpenLDAP backends and overlays that can be useful (many of which are covered in this book). In cases where we want to build everything, typically it is best to compile OpenLDAP with module support, and compile all of the overlays and backends as modules. That way we can have all of the extras available, but only the ones needed (and configured in `slapd.conf`) get loaded at runtime.

### Note

Many of the additional backends and overlays have their own dependencies. For example, the Perl backend requires that the Perl libraries be installed. Most of the necessary dependencies are installed by default in Ubuntu. If you don't have the requisite libraries for a module, the `configure` or `make` programs will let you know what library is missing, and you will have to track down which package contains that library. For this process, you may find the package search on Debian's website useful ([http://www.us.debian.org/distrib/packages#search_contents](http://www.us.debian.org/distrib/packages#search_contents)).

Since we are building OpenLDAP with modules, we will need to make sure that `libtool` and the libtool header files are installed. In Ubuntu, it is not installed by default. Also, since the Perl backend (`back_perl`) will be installed, we will need to install the Perl development package. You can install all of these with one command:

```
 $ sudo apt-get install libtool libltdl3 libltdl3-dev libperl-dev

```

The `libltdl3` library is usually installed by default, but the others are also needed to compile OpenLDAP with module support. Now we are ready to build OpenLDAP with modules.

To build OpenLDAP with all of the extra modules, we just need to use the correct flags with `configure`:

```
 $./configure --enable-dynamic --enable-modules --enable-backends=mod 
\
 --enable-overlays=mod

```

To build everything we need only four flags. The first, `--enable-dynamic` enables shared libraries. Second, `--enable-modules` simply tells `configure` that we want to use modules. The next two indicate what backends and overlays we want built: `--enable-overlays`, which is set to `mod` in order to build modules, and `–enable-backends` (also set to `mod`) to build all of the available backends.

Once `configure` completes, you can run `make`:

```
 $ make depend && make && make test

```

This will build all the dependencies, then build OpenLDAP (and all of the modules), and then test everything. When you are ready to install, you can follow the instructions in the previous section.

# Summary

In this appendix we have briefly examined the process of building OpenLDAP from source. At this point you should have the information necessary for building OpenLDAP from source.

We have looked at a very basic build and also a complete build using modules. But there are many other available options. You can learn more about building OpenLDAP from the documentation included with OpenLDAP.