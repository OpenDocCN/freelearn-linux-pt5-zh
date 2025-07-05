# 2

# The Linux Shell and Filesystem

Understanding the **Linux filesystem**, **file management** fundamentals, and the basics of the **Linux shell** and **command-line interface** (**CLI**) is essential for a modern-day Linux professional.

In this chapter, you will learn how to use the Linux shell and some of the most common commands in Linux. You will learn about the structure of a basic Linux command and how the Linux filesystem is organized. We’ll explore various commands for working with files and directories. Along the way, we’ll introduce you to the most common command-line text editors. We hope that by the end of this chapter, you’ll be comfortable using the Linux CLI and be ready for future, more advanced explorations. This chapter will set the foundation for using the Linux shell, and for more information about the shell, go to [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux* *Shell Scripting*.

We’re going to cover the following main topics:

*   Introducing the Linux shell
*   The Linux filesystem
*   Working with files and directories
*   Using text editors to create and edit files

# Technical requirements

This chapter requires a working installation of a standard Linux distribution, on either server, desktop, PC, or **Virtual Machine** (**VM**). Our examples and case studies use the Ubuntu and Fedora platforms, but the commands and examples explored are equally suitable for any other Linux distribution.

# Introducing the Linux shell

Linux has its roots in the Unix operating system, and one of its main strengths is the command-line interface. In the old days, this was called *the shell*. In `sh` command. The shell is a program that has two streams: an *input stream* and an *output stream*. The input is a command given by the user, and the output is the result of that command, or an interpretation of it. In other words, the shell is the primary interface between the user and the machine.

The main shell in major Linux distributions is called **Bash**, which is an acronym for **Bourne Again Shell**, named after Steve Bourne, the original creator of the shell in UNIX. Alongside Bash, there are other shells available in Linux, such as **ksh**, **tcsh**, and **zsh**. In this chapter and throughout the book, we will cover the Bash shell, as it is the most widely used shell in modern Linux distributions.

Important note

Distributions such as Debian, Ubuntu, Fedora, CentOS Stream, RHEL, openSUSE, SLE, and Linux Mint, just to name a few, use the *Bash* shell by default. Other distributions, such as Kali Linux, have switched to *zsh* by default. Manjaro offers zsh on some editions. For those who use macOS, you should know that zsh has been the default shell for some years now. Nevertheless, you can install any shell you want on Linux and make it your default one. In general, shells are pretty similar, as they do the same thing, but they add different *extras* to usability and features. If you are interested in a specific shell, feel free to use it and test out the differences between others.

One shell can be assigned to each user. Users on the same system can use different shells. One way to check the default shell is by accessing the `/etc/passwd` file. More details about this file and user accounts will be discussed in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users and Groups*. For now, it is important to know where to look for the default shell. In this file, the last characters from each line represent the user’s default shell. The `/etc/passwd` file has the users listed on each line, with details about their **process identification number** (**PID**), **group identification number** (**GID**), username, home directory, and basic shell.

To see the default shell for each user, execute the following command by using your user’s name (in our case, it is `packt`):

```
cat /etc/passwd | grep packt
```

The output should be a list of the contents of the `/etc/passwd` file. Depending on the number of users you have on your system, you will see all of them, each one on a separate line. An easier way to see the *current shell* is by running the following command:

```
echo $0
```

This shows what exactly is running your command, which in the case of the CLI is the shell. The `$0` part is a `echo $0` command on Ubuntu and on Debian:

![Figure 2.1 – Commands used to see the running shell](img/Figure_02.01_B19682.jpg)

Figure 2.1 – Commands used to see the running shell

As you can see, running the `echo $0` command gives us different outputs but with the same message: the running shell is Bash. If you have other shells that you prefer, you can easily assign another shell to your user, if you already have it installed. However, if you know Bash, you will be comfortable with all the other available shells.

Important note

*The Linux shell is case-sensitive*. This means that everything you type inside the command line should respect this. For example, the `cat` command used earlier used lowercase. If you type `Cat` or `CAT`, the shell will not recognize it as being a command. The same rule applies to file paths. You will notice that default directories in your home directory use uppercase for the first letter, as in `~/Documents`, `~/Downloads`, and so on. Those names are different from `~/documents` or `~/downloads`.

In this chapter, you will learn how to use Linux commands and the shell in addition to learning about its filesystem. You will learn about software management in [*Chapter 3*](B19682_03.xhtml#_idTextAnchor075), thus showing you how to install another shell now would mean that we will get ahead of ourselves. We want you to slowly but steadily build your Linux knowledge, so we will show you in the next chapter how to install a new shell. For now, Bash is sufficient, and we will use it throughout the book.

If you want to see all the shells that are installed on your system, you can run the following command:

```
cat /etc/shells
```

In our case, the output with all the installed shells (by default) in Ubuntu Server 22.04.2 LTS is shown in the following image:

![Figure 2.2 – The shells available by default in Ubuntu](img/Figure_02.02_B19682.jpg)

Figure 2.2 – The shells available by default in Ubuntu

This will show you all the shells installed. You can use any of those or can install new ones as we will show you in [*Chapter 3*](B19682_03.xhtml#_idTextAnchor075). Also, in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), when we work with user accounts, you will get to learn how you can change a user’s shell.

The following section will introduce you to shell connection types.

## Establishing the shell connection

We can make two different types of connections to the shell: `tty` and `pts`. The name `tty` stands for **teletypewriter**, which was a type of terminal used at the beginning of computing. This connection is considered a native one, with ports that are direct connections to your computer. The link between the user and the computer is mainly found to be through a keyboard, which is considered to be a native terminal device.

The `pts` connection is generated by SSH or Telnet types of links. Its name stands for `ssh` or `xterm`. It is the slave of the `pty`.

In the next section, we will further explore the connections to virtual terminals available in Linux.

### Virtual consoles/terminals

The terminal was thought of as a device that manages the input strings (which are commands) between a process and other I/O devices such as a keyboard and a screen. There are also **pseudo terminals**, which are emulated terminals that behave the same way as a **classical terminal**. The difference is that it does not interact with devices directly, as it is all emulated by the Linux kernel, which transmits the I/O to a program called the shell.

`tty1`, `tty2`, `tty3,` `tty4`, `tty5`, and `tty6`, respectively, on your computer.

We will explain this using an Ubuntu 22.04.2 LTS Server VM installation, but it is identical in Rocky Linux too. After starting the VM and being prompted to log in with your username and password, the first line on the screen will be something similar to the following output:

```
Ubuntu 22.04.2 LTS neptune tty1
```

If you press any of the preceding key combinations, you will see your terminal change from `tty1` to any of the other `tty` instances. For example, if you press *Ctrl* + *Alt* + *F6*, you will see this:

```
Ubuntu 22.04.2 LTS neptune tty6
```

As we were using the server edition of Ubuntu, we did not have the GUI installed. But if you were to use a desktop edition, you will be able to use *Ctrl* + *Alt* + *F7* to enter `X graphical` mode, for example. The `neptune` string is the name we gave to our virtual machine.

If you are not able to use the preceding keyboard combinations, there is a dedicated command for changing virtual terminals. The command is called `chvt` and has the syntax `chvt N`. Even though we have not discussed shell commands yet, we will show you an example of how to use them and other related commands. This action can only be performed by an administrator account or by using `sudo`. Briefly, `sudo` stands for *superuser do* and allows any user to run programs with administrative privileges or with the privileges of another user (more details about this in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users* *and Groups*).

In the following example, we will use Ubuntu to show you how to change virtual terminals. First, we will see which virtual terminal we are currently using to change it to another one without using the *Ctrl* + *Alt* + *Fn* keys.

The `who` command will show you information about the users currently logged in to the computer. In our case, as we are connected through SSH to our virtual machine, it will show that the user `packt` is currently using pseudo-terminal zero (`pts/0`):

```
packt    pts/0        2023-02-28 10:45 (192.168.122.1)
```

If we were to run the same command in the console of the virtual machine directly, we would have the following output:

```
packt    pts/0        2023-02-28 10:45 (192.168.122.1)
packt    tty1         2023-02-28 10:50
```

It shows that the user is connected to both the virtual terminal 1 (`tty1`) and also through SSH from our host operating system to the virtual machine (`pts/0`).

Now, by using the `chvt` command, we will show you how to change to the sixth virtual terminal. After running `sudo chvt 6`, you will be prompted to provide your password and immediately be switched to virtual terminal number six. Running `who` once more will show you all logged-in users and the virtual terminals they use. In our case will be `pts/0`, `tty2`, and `tty6`. Please take into consideration that your output could be different, as in different virtual terminal numbers.

Now that we know what types of shell connections are established, let us learn about the shell’s prompt in the next section.

## The command-line prompt

The **command-line prompt** or **shell prompt** is the place where you type in the commands. Usually, the command prompt will show the username, hostname, present working directory, and a symbol that indicates the type of user running the shell.

Here is an example from the Ubuntu 22.04.2 LTS Server edition (similar to Debian):

```
packt@saturn:~$
```

Here is an example from the Fedora 37 server (similar to Rocky Linux, RHEL, or AlmaLinux):

```
[packt@localhost ~]$
```

Here is a short explanation of the prompt:

*   `packt` is the name of the user currently logged in
*   `saturn` and `localhost` are the hostnames
*   `~` represents the home directory (it is called a tilde)
*   `$` shows that the user is a regular user (when you are logged in as an administrator, the sign changes into a hashtag, `#`)

Also, when using openSUSE, you will notice that the prompt is different than the ones in Ubuntu/Debian and Fedora/RHEL. The following is an example of the prompt while running the Leap 15.4 server edition:

```
packt@localhost:~>
```

As you can see, there is no dollar sign (`$`) or hashtag (`#`), only a greater than sign (`>`). This might be confusing at first, but when you will use the root user, the sign will eventually change to the hashtag (`#`). The following is an example:

```
localhost:/home/packt #
```

Let’s look at the shell command types next.

## Shell command types

Shells work with `type` command. For example, you can check what type of command `cd` (change directory) is:

```
packt@neptune:~$ type cd
cd is a shell builtin
```

The output shows that the `cd` command is an internal one, built inside the shell. If you are curious, you could find out the types of other commands that we will show you in the following sections by writing type in front of the command’s name. Let us see some more examples in the following image:

![Figure 2.3 – Different types of commands in Linux](img/Figure_02.03_B19682.jpg)

Figure 2.3 – Different types of commands in Linux

Now that you know some of the types of Linux commands, let us dissect the command’s structure and learn about its components.

## Explaining the command structure

We have already used some commands, but we did not explain the structure of a Linux command. We will do that now for you to be able to understand how to use commands. In a nutshell, Unix and Linux commands have the following form:

*   The command’s name
*   The command’s options
*   The command’s arguments

Inside the shell, you will have a general structure such as the following:

```
command [-option(s)] [argument(s)]
```

A suitable example would be the use of the `ls` command (`ls` comes from a *list*). This command is one of the most-used commands in Linux. It lists files and directories and can be used with both options and arguments.

We can use `ls` in its simplest form, without options or arguments. It lists the contents of your present working directory (`pwd`). In our case, it is the home directory, indicated by the `~` tilde character in the shell’s prompt (see *Figure 2**.10*).

The `ls` command with the `-l` option (lowercase L) uses a long listing format, giving you extra information about files and directories from your present working directory (`pwd`):

![Figure 2.4 – Using the ls command with both options and attributes](img/Figure_02.04_B19682.jpg)

Figure 2.4 – Using the ls command with both options and attributes

In the preceding example, we used `ls -l ~/Documents/` to show the contents of the `~/Documents` directory. Shown here is a way to use the command with both options and attributes, without changing our present working directory to `~/Documents`.

In the following section, we will show you how to use the manual pages available by default in Linux.

## Consulting the manual

Any Linux system administrator’s best friend is the manual. Each command in Linux has a manual page that gives the user detailed information about its use, options, and attributes. If you know the command you want to learn more about, simply use the `man` command to explore. For the `ls` command, for example, you use `man ls`.

The manual organizes its command information into different sections, with each section being named by convention to be the same on all distributions. Briefly, those sections are `name`, `synopsis`, `configuration`, `description`, `options`, `exit status`, `return value`, `errors`, `environment`, `files`, `versions`, `conforming to`, `notes`, `bugs`, `example`, `authors`, `copyright`, and `see also`.

Similar to the manual pages, almost all commands in Linux have a `-help` option. You can use this for quick reference.

For more information about the `help` and `man` pages, you can check each command’s help or manual page. Try the following commands:

```
$ man man
$ help help
```

When you use the manual, keep in mind that it is not a step-by-step how-to guide. It is technical documentation that might be confusing at first. Our advice is to use the `man` pages as much as you can. Before you search for anything on the internet, try to read the manual first. This will be a good exercise, and you will become proficient with Linux commands in no time.

Consider the manual your friend, similar to the textbooks you used in high school or college. It will give you first-hand information when you most need it. If you take into consideration situations where outside internet access is limited, with no access to search engines, the built-in manual will be your best companion. Learn to use its powers to your advantage.

In the following section, you will learn about the Linux filesystem.

# The Linux filesystem

The `/ (root)` filesystem and another the `/home` filesystem. Or, there can be just one that contains all filesystems.

Generally, using one filesystem per partition is considered to be good practice by allowing for logical maintenance and management. As everything in Linux is a file, physical devices such as hard drives, DVD drives, USB devices, and floppy drives are treated as files too. In this section, you will learn about the directory structure, how to work with files, and some very useful editing techniques from the command line.

## Directory structure

Linux uses a `` `/` ``) at the base of the filesystem. From that point, all the branches (directories) spread throughout the filesystem.

The **Filesystem Hierarchy Standard** (**FHS**) defines the structure of Unix-like filesystems. However, Linux filesystems also contain some directories that aren’t yet defined by the standard.

### Exploring the Linux filesystem from the command line

Feel free to explore the filesystem yourself by using the `tree` command. In Fedora Linux, it is already installed, but if you use Ubuntu, you will have to install it by using the following command:

```
$ sudo apt install tree
```

Do not be afraid to explore the filesystem, because no harm will be done just by looking around. You can use the `ls` command to list the contents of directories, but `tree` offers different graphics. The following image shows you the differences between the outputs:

![Figure 2.5 – Comparing the output of ls -la commands and tree -a commands](img/Figure_02.05_B19682.jpg)

Figure 2.5 – Comparing the output of ls -la commands and tree -a commands

The `tree` command has different options available, and you can learn about them by using the manual. Let us use the `tree` command by invoking the `-L` option, which tells the command how many levels down to go, and the last attribute states which directory to start with. In our example, the command will go down one level, starting from the `root` directory, represented by the forward slash as an argument (see *Figure 2**.12*):

```
$ tree -L 1 /
```

Start exploring the directories from the structure by using the `tree` command, as shown here:

![Figure 2.6 – The tree command with option and argument on Ubuntu](img/Figure_02.06_B19682.jpg)

Figure 2.6 – The tree command with option and argument on Ubuntu

Important note

Remember that some of the directories you are about to open will contain a large number of files and/or other directories, which will clutter your terminal window.

The following are the directories that exist on almost all versions of Linux. Here’s a quick overview of the Linux root filesystem:

*   `/`: Root directory. The root for all other directories.
*   `/bin`: Essential command binaries. The place where binary programs are stored.
*   `/boot`: Static files of the boot loader. The place where the kernel, bootloader, and `initramfs` are stored.
*   `/dev`: Device files. Nodes to the device equipment, a kernel device list.
*   `/etc`: Host-specific system configuration. Essential config files for the system, boot time loading scripts, `crontab`, `fstab` device storage tables, `passwd` user accounts file.
*   `/home`: user Home directory. The place where the user’s files are stored.
*   `/lib`: Essential shared libraries and kernel modules. Shared libraries are similar to **Dynamic Link Library** (**DLL**) files in Windows.
*   `/media`: Mount point for removable media. For external devices and USB external media.
*   `/mnt`: Mount point for mounting a filesystem temporarily. Used for legacy systems.
*   `/opt`: Add-on application software packages. The place where *optional* software is installed.
*   `/proc`: Virtual filesystem managed by the kernel. a special directory structure that contains files essential for the system.
*   `/sbin`: Essential system binaries. Vital programs for the system’s operation.
*   `/srv`: Data for services provided by this system.
*   `/tmp`: Temporary files.
*   `/usr`: Secondary hierarchy. The largest directory in Linux that contains support files for regular system users:
    *   `/usr/bin` – system-executable files
    *   `/usr/lib` – shared libraries from `/usr/bin`
    *   `/usr/local` – source compiled programs not included in the distribution
    *   `/usr/sbin` – specific system administration programs
    *   `/usr/share` – data shared by the programs in `/usr/bin` such as config files, icons, wallpapers or sound files
    *   `/usr/share/doc` – documentation for the system-wide files
*   `/var`: Variable data. Only data that is modifiable by the user is stored here, such as databases, printing spool files, user mail, and others; `/var/log` – contains log files that register system activity

Next, we’re going to learn how to work with these files and directories.

# Working with files and directories

Remember that everything in Linux is a file. A directory is a file too. As such, it is essential to know how to work with them. Working with files in Linux implies the use of several commands for basic file and directory operations, file viewing, file creation, file location, file properties, and linking. Some of the commands, which will not be covered here, have uses closely related to files. These will be covered in the following section.

## Understanding file paths

Each file in the FHS has a *path*. The path is the file’s location represented in an easily readable representation. In Linux, all the files are stored in the root directory by using the FHS as a standard to organize them. Relations between files and directories inside this system are expressed through the forward-slash character (`/`). Throughout computing history, this was used as a symbol that described addresses. Paths are, in fact, addresses for files.

There are two types of paths in Linux, relative ones and absolute ones. An **absolute path** always starts with the root directory and follows the branches of the system up to the desired file. A **relative path** always refers to the **present working directory** (**pwd**) and represents the relative path to it. Thus, a relative path is always a path that is relative to your present working directory.

For example, let us refer to an existing file from our home directory, a file called `poem`. Being inside the home directory, and our `pwd` command being the `home` directory for `packt`, the absolute path of the file called `poem` would be as follows:

```
/home/packt/poem
```

If we were to show the contents of that file using the `cat` command, for example, we would use the command with the absolute path:

```
cat /home/packt/poem
```

The relative path to the same file would be relative to `pwd`, so in our case, where we’re already inside the home directory, using the `cat` command would be like this:

```
cat poem
```

The absolute path is useful to know about when you work with files. After some practice, you will come to learn the paths to the most-used files. For example, one file that you will need to learn the path for is the `passwd` file. It resides in the `/etc` directory. Thus, when you will refer to it, you will use its absolute path, `/etc/passwd`. Using a relative path to that file would imply that you are either inside its parent directory or somewhere close in the FHS.

Working with relative paths involves knowing two special characters used to work with the FHS. One special character is the dot (`.`), and it refers to the current directory. The other is two consecutive dots (`..`) and refers to the parent directory of the current directory. When working with relative paths, make sure that you always check what directory you are in. Use the `pwd` command to show your present working directory.

A good example of working with relative paths is when you are already inside the parent directory and need to refer to it. If you need to see the accounts list from your system, which is stored inside the `passwd` file, you can refer to it by using a relative path. For this exercise, we are inside our home directory:

![Figure 2.7 – Using the relative path of a file](img/Figure_02.07_B19682.jpg)

Figure 2.7 – Using the relative path of a file

Here, we first check our present working directory with the `pwd` command, and the output will be our home directory’s path, `/home/packt`. Secondly, we try to show the contents of the `passwd` file using the `cat` command right from the home directory, but the output will be an error message saying that there is no such file or directory inside our home directory. We used the relative path, which is always relative to our present working directory, hence the error. Thirdly, we use the double-consecutive dots special characters to refer to the file with its relative path. In this case, the command is `cat ../../etc/passwd`.

Tip

Always use the *Tab* key on your keyboard for autocompletion and to check whether the path you typed is correct or not. In the preceding example, we typed `../../etc` and pressed *Tab*, which autocompleted with a forward slash. Then, we typed the first two letters of the file we were looking for and pressed *Tab* again. This showed us a list of files inside the `/etc` directory that started with `pa`. Seeing that `passwd` was in there, we knew that the path was right, so we typed two more `s` characters and pressed *Tab* again. This completed the command for us and we pressed *Enter*/*Return* to execute the command.

The path in the final command is relative to our home directory and it translates as follows: *concatenate the file with the* `passwd` *name that is located in the* `/etc` *directory in the parent directory (first two dots) of the parent directory (second two dots) of our current directory (home)*. Therefore, the `/etc/passwd` absolute path is translated into a relative path to our home directory like this: `../../etc/passwd`.

Next, we are going to learn about basic file operations in Linux.

## Basic file operations

Daily, as a system administrator, you will manipulate files. This includes creating, copying, moving, listing, deleting, linking, and so on. The basic commands for these operations have already been discussed throughout this chapter, but now it is time to get into more detail about their use, options, and attributes. Some more advanced commands will be detailed in the following sections.

### Creating files

There are situations when you will need to `touch` command. When you use it, it will create a new file with you as the file owner and with a size of zero, because it is an empty file.

In the following example, we create a new file called `new-report` inside the `~/``packt/` directory:

![Figure 2.8 – Using the touch command to create and alter files](img/Figure_02.08_B19682.jpg)

Figure 2.8 – Using the touch command to create and alter files

The `touch` command is also used to change the modification time of a file without changing the file itself. Notice the difference between the initial time when we first created the `new-report` file and the new time after using the `touch` command on it. You can also change the access time by using the `-a` option of the `touch` command. By default, the long listing of the `ls` command shows only the modification/creation time. If you want to see the access time, there is the `atime` parameter you can use with the `- time` option. See the example used in the following figure:

![Figure 2.9 – Using touch to alter the access time](img/Figure_02.09_B19682.jpg)

Figure 2.9 – Using touch to alter the access time

The modification, creation, and access time stamps are very useful, especially when using commands such as `find`. They give you a more *granular* search pattern. We will get back to this command with more examples in future sections.

Files can also be created by using redirection and the `echo` command. `echo` is a command that prints the string given as a parameter to the standard output (the screen). The output of the `echo` command can be written directly to a file by using the output redirection:

![Figure 2.10 – Using echo with output redirection](img/Figure_02.10_B19682.jpg)

Figure 2.10 – Using echo with output redirection

In the preceding example, we redirected the text from the `echo` command to the presentation file. It did not exist at the beginning, so it was automatically created by the command. The first `echo` command added a line to the file by using the `>` operator. The second `echo` command appended a new line of text to the end of the file, by using the `>>` operator.

### Listing files

We have already used some examples with the `ls` command before, so you are somewhat familiar with it. We covered the `-l` option as an example of the command’s structure. Thus, we will not cover it any further here. We will explore new options for this essential and useful command:

*   `ls -lh`: The `-l` option lists the files in an extended format, while the `-h` option shows the size of the file in a human-readable format, with the size in kilobytes or megabytes rather than bytes.
*   `ls -la`: The `-a` option shows all the files, including hidden ones. Combined with the `-l` option, the output will be a list of all the files and their details.
*   `ls -ltr`: The `-t` option sorts files by their modification time, showing the newest first. The `-r` option reverses the order of the sort.
*   `ls – lS`: The `-S` option sorts the files by their size, with the largest file first.
*   `ls -R`: The `-R` option shows the contents of the current or specified directory in recursive mode.

A method used frequently for listing files and directories is called `ls -la` command. Let’s look at it in detail here, even though we will discuss this thoroughly in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users* *and Groups*.

One example of a long listing used on our home directory can be seen in *Figure 2**.11*, when we compared the output of `ls -la` with the output of the `tree` command. The following code snippet is a short example:

```
total 48
drwxr-x--- 5 packt packt 4096 Mar  2 14:44 .
drwxr-xr-x 3 root  root  4096 Feb 27 08:58 ..
-rw-rw-r-- 1 packt packt   55 Mar  2 14:46 art-file
-rw------- 1 packt packt 1039 Mar  1 18:56 .bash_history
```

In the output, the first row after the command shows the number of blocks inside the directory listed. After that, each line represents one file or subdirectory, with the following detailed information:

*   The first character is the type of the file: `d` for the directory, `:` for the file, `l` for a link, `c` for the character device, and `b` for the block device
*   The following nine characters represent the permissions (detailed in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users* *and Groups*)
*   The hard link for that file (see the *Working with links* subsection in this chapter)
*   The owner’s PID and GID (details in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users* *and Groups*)
*   The size of the file (the number depends on whether it is in human-readable format or not)
*   The last modification time of the file
*   The name of the file or directory

The first two lines are a reference to itself (the dot) and to the parent directory (the two dots from the second line).

The next section will teach you how to copy and move files.

### Copying and moving files

To copy files in Linux, the `cp` command is used. The `mv` command moves files around the filesystem. This command is also used to rename files.

To copy a file, you can use the `cp` command in the simplest way:

```
cp source_file_path destination_file_path
```

Here, `source_file_path` is the name of the file to be copied, and `destination_file_path` is the name of the destination file. You can also copy multiple files inside a directory that already exists. If the destination directory does not exist, the shell will signal to you that the target is not a directory.

Now let’s look at some variations of these commands:

*   `cp -a`: The `-a` option copies an entire directory hierarchy in recursive mode by preserving all the attributes and links. In the following example, we copied the entire `dir1` directory that we previously created inside our home directory, to a newly created directory called `backup_dir1` by using the `-``a` option:

![Figure 2.11 – Using the copy command with the -a option](img/Figure_02.11_B19682.jpg)

Figure 2.11 – Using the copy command with the -a option

*   `cp -r`: This option is similar to `-a`, but it does not preserve attributes, only symbolic links.
*   `cp -p`: The `-p` option retains the file’s permissions and timestamps. Otherwise, just by using `cp` in its simplest form, copies of the files will be owned by your user with a timestamp of the time you did the copy operation.
*   `cp -R`: The `-R` option allows you to copy a directory recursively. In the following example, we will use the `ls` command to show you the contents of the `~/packt/` directory, and then the `cp -R` command to copy the contents of the `/files` directory to the `/new-files` one. The `/new-files` directory did not exist. The `cp -R` command created it:

![Figure 2.12 – Using the cp -R command](img/Figure_02.12_B19682.jpg)

Figure 2.12 – Using the cp -R command

Moving files around is done with the `mv` command. It is either used to move files and directories from one destination to another or to rename a file. The following is an example in which we rename `files1` into `old-files1` using the `mv` command: `mv` `files1 old-files1`.

There are many other options that you could learn about just by visiting the manual pages. Feel free to explore them and use them in your daily tasks.

### Working with links

Links are a compelling option in Linux. They can be used as a means of protection for the original files, or just as a tool to keep multiple copies of a file without having separate hard copies. Consider it as a tool to create alternative names for the same file.

The `ln` command can be used to do this and create two types of links:

*   Symbolic links
*   Hard links

Those two links are different types of files that point to the original file. A **symbolic link** is a physical file that points to the original file; they are linked and have the same content. Also, it can span different filesystems and physical media, meaning that it can link to original files that are on other drives or partitions with different types of filesystems. The command used is as follows:

```
ln -s [original_filename] [link_filename]
```

Here is an example in which we listed the contents of the `~/packt` directory and then created a symbolic link to the `new-report` file using the `ln -s` command and then listed the contents again:

![Figure 2.13 – Using symbolic links](img/Figure_02.13_B19682.jpg)

Figure 2.13 – Using symbolic links

You can see that the link created is named `new-report-link` and is visually represented with an `->` arrow that shows the original file that it points to. You can also distinguish the difference in size between the two files, the link and the original one. The permissions are different too. This is a way to know that they are two *different* physical files. To double-check that they are different physical files, you can use the `ls -i` command to show the `new-report` and `new-report-link` have different inodes:

![Figure 2.14 – Comparing the inodes for the symbolic link and original file](img/Figure_02.14_B19682.jpg)

Figure 2.14 – Comparing the inodes for the symbolic link and original file

If you want to know where the link points to and you do not want to use `ls -l`, there is the `readlink` command. It is available in both Ubuntu and CentOS. The output of the command will simply be the name of the file that the symbolic link points to. It only works in the case of symbolic links:

```
packt@neptune:~$ readlink new-report-link
new-report
```

In the preceding example, you can see that the output shows that the `new-report-link` file is a symbolic link to the file named `new-report`.

In contrast, a `ln` without any options:

```
ln [original-file] [linked-file]
```

In the following example, we created a hard link for the `new-report` file, and we named it `new-report-ln`:

![Figure 2.15 – Working with hard links](img/Figure_02.15_B19682.jpg)

Figure 2.15 – Working with hard links

In the output, you will see that they have the same size and the same inode, and after altering the original file using `echo` and output redirection, the changes were available to both files. The two files have a different representation than symbolic links. They appear as two different files in your listing, with no visual aids to show which file is pointed to. Essentially, a hard link is linked to the inode of the original file. You can see it as a new name for a file, similar to but not identical to renaming it.

### Deleting files

In Linux, you have the remove (`rm`) command for deleting files. In its simplest form, the `rm` command is used without an option. For more control over how you delete items, you could use the `-i`, `-f`, and `-``r` options:

*   `rm -i`: This option enables interactive mode by asking you for acceptance before deleting:

![Figure 2.16 – Removing a file interactively](img/Figure_02.16_B19682.jpg)

Figure 2.16 – Removing a file interactively

In the preceding example, we deleted `art-file` by using the `-i` option. When asked to interact, you have two options. You can approve the action by typing `y` (yes), or `n` (no) to cancel the action.

*   `rm -f` : The `-f` option deletes the file by force, without any interaction from the user:

![Figure 2.17 – Force remove a file](img/Figure_02.17_B19682.jpg)

Figure 2.17 – Force remove a file

We deleted the `new-report-link` file created earlier by using the `rm -f` command. It did not ask for our approval and deleted the file directly.

*   `rm -r`: This option deletes the files in recursive mode, and it is used to delete multiple files and directories. For example, we will try to delete the `new-files` directory. When using the `rm` command in its simplest way, the output will show an error saying that it cannot delete a directory. But when used with the `-r` option, the directory is deleted right away:

![Figure 2.18 – Remove a directory recursively](img/Figure_02.18_B19682.jpg)

Figure 2.18 – Remove a directory recursively

Important note

We advise *extra caution* when using the `rm` command. The most destructive mode is to use `rm -rf`. This will delete files, directories, and anything without warning. Pay attention as there is no going back from this. Once used, the damage will be done.

Most of the time, removing files is a one-way street, with no turning back. This makes the process of deleting files a very important one, and a backup before a deletion could save you a lot of unnecessary stress.

### Creating directories

In Linux, you can create a new directory with the `mkdir` command. In the following example, we will create a new directory called `development`:

```
mkdir development
```

If you want to create more directories and sub-directories at once, you will need to use the `-p` option (`p` from the parent), as shown in the following figure:

![Figure 2.19 – Creating parent directories](img/Figure_02.19_B19682.jpg)

Figure 2.19 – Creating parent directories

Directories are files too in Linux, only that they have special attributes. They are essential to organizing your filesystem. For more options with this useful tool, feel free to visit the manual pages.

### Deleting directories

The Linux command for removing directories is called `rmdir`. It is designed to default by deleting only empty directories. Let’s see what happens if we try to delete a directory that is not empty:

![Figure 2.20 – Using the rmdir command](img/Figure_02.20_B19682.jpg)

Figure 2.20 – Using the rmdir command

This is a precautionary measure from the shell, as deleting a directory that is not empty could have disastrous consequences, as we’ve seen when using the `rm` command. The `rmdir` command does not have a `-i` option such as `rm`. The only way to delete the directory using the `rmdir` command is to delete files inside it first manually. However, the `rm -r` command shown earlier is still useful and more versatile when deleting directories.

Now that you know how to work with directories in Linux, we will proceed to show you commands for file viewing.

## Commands for file viewing

As everything in Linux is a file, being able to view and work with file contents is an essential asset for any system administrator. In this section, we will learn commands for file viewing, as almost all files contain text that, at some point, should be readable.

### The cat command

This command was used in some of our previous examples in this chapter. It is short for *concatenate* and is used to print the contents of the file to the screen. We have used `cat` several times during this chapter, but here is yet another example. We have two existing files, one called `new-report` and the other called `users`. Let us show you how to use `cat` in the following image:

![Figure 2.21 – Example of using the cat command](img/Figure_02.21_B19682.jpg)

Figure 2.21 – Example of using the cat command

In this example, we first used the command to show the contents of only one file, the one called `new-report`. The second command was used to show the contents of two files at once, both `new-report` and `users`. The `cat` command is showing the contents of both on the screen. Both files are located in the same directory, which is also the user’s working directory. If you would like to concatenate the contents of files that are not inside your present working directory, you would need to use their absolute path.

The `cat` command has several options available, which we will not cover here, as most of the time, its purest form will be the most used. For more details, see the manual pages.

### The less command

There are times when a file has so much text that it will cover many screens, and it will be difficult to view on your terminal using just `cat`. This is where the `less` command is handy. It shows one screen at a time. How much a screen means, it all depends on the size of your terminal window. Let’s take, for example, the `/etc/passwd` file. It could have multiple lines that you would not be able to fit in just one screen. You could use the following command:

```
$ less /etc/passwd
```

When you press *Enter*, the contents of the file will be shown on your screen. To navigate through it, you could use the following keys:

*   Space bar: Move forward one screen
*   *Enter*: Move forward one line
*   *b*: Move backward one screen
*   */*: Enter search mode; this searches forward in your file
*   *?*: Search mode; this searches backward in your file
*   *v*: Edit your file with the default editor
*   *g*: Jump to the beginning of the file
*   *Shift* + *g*: Jump to the end of the file
*   *q*: Exit the output.

The `less` command has a multitude of options that could be used. We advise you to consult the manual pages to find out more information about this command.

### The head command

This command is handy when you only want to print to the screen the beginning (the head) of a text file. By default, it will print only the first 10 lines of the file. You can use the same `/etc/passwd` file for the head exercise and execute the following command. Watch what happens. It prints the first 10 lines and then exits the command, taking you back to the shell prompt:

```
head /etc/passwd
```

One useful option of this command is to print more or less than 10 lines of the file. For this, you can use the `-n` argument or simply just `–` with the number of lines you want to print. For the `/etc/passwd` file, we will first use the head command without any options, and then we will use it with the number of lines argument, as shown in the following figure:

![Figure 2.22 – Using the head command](img/Figure_02.22_B19682.jpg)

Figure 2.22 – Using the head command

Many other options that this command provides can prove useful for your work as a system administrator, but we will not cover them here. Feel free to explore them yourself.

### The tail command

The `tail` command is similar to the `head` command, only it prints the last 10 lines of a file, by default. You can use the same `-n` argument as for the head command, to see a specific number of lines from the end of a file. However, the `tail` command is commonly used for actively watching log files that are constantly changing. It can print the last lines of the file as other applications are writing to it. Take, for example, the following line:

```
tail -f /var/log/syslog
```

Using the `-f` option will make the command watch the `/var/log/syslog` file as it is being written. It will show you the contents of the file on the screen effectively. The `-f` option will cause the tail command to stop during a log rotation, and in this case, the `-F` option should be used instead. When using the `-F` option, the command will continue to show the output even during a log rotation. To exit that screen, you will need to press *Ctrl* + *C* to go back to the shell prompt. The following is an example of the output of the previous command:

![Figure 2.23 – The use of tail command for real-time log  file observation](img/Figure_02.23_B19682.jpg)

Figure 2.23 – The use of tail command for real-time log file observation

Next, let us learn how to view file properties in Linux.

## Commands for file properties

There could be times when just viewing the contents of a file is not enough, and you need extra information about that file. There are other handy commands that you could use, and we describe them in the following sections.

### The stat command

The `stat` command gives you more information than the `ls` command does. The example in the following figure shows a comparison between the `ls` and `stat` outputs for the same file:

![Figure 2.24 – Using the stat command](img/Figure_02.24_B19682.jpg)

Figure 2.24 – Using the stat command

The `stat` command gives you more information about the name, size, number of blocks, type of file, inode, number of links, permissions, UID and GID, and `atime`, `mtime`, and `ctime`. To find out more information about it, please refer to the Linux manual pages.

### The file command

This command simply reports on the type of file. Here is an example of a text file and a command file:

![Figure 2.25 – Using the file command](img/Figure_02.25_B19682.jpg)

Figure 2.25 – Using the file command

Linux does not rely on file extensions and types as some other operating systems do. In this respect, the `file` command determines the file type more by its contents than anything else.

### Commands for configuring file ownership and permissions

In Linux, `chown` command. When setting up group ownership, you determine the permissions for everyone in that group. This is set up using the `chgrp` command. When it comes to other users, the reference is to everyone else on that system, someone who did not create the file, so it is not the owner, and who does not belong to the owner’s group. Other is also known as, or referred to as the world.

Besides setting user ownership, the system must know how to determine user behavior, and it does that through the use of `ls -``l` command:

![Figure 2.26 – Long listing output](img/Figure_02.26_B19682.jpg)

Figure 2.26 – Long listing output

In the preceding examples, you see two different types of permissions for the files inside our home directory. Each line has 12 characters reserved for special attributes and permissions. Out of those 12, only 10 are used in the preceding examples. Nine of them represent the permissions, and the first one is the file type. There are three easy-to-remember abbreviations for permissions:

*   `r` is for `read` permission
*   `w` is for `write` permission
*   `x` is for `execute` permission
*   `-` is for no permission

The nine characters are divided into three regions, each consisting of three characters. The first three characters are reserved for user permissions, the following three characters are reserved for group permissions, and the last three characters represent other, or global permissions.

File types also have their codes, as follows:

*   `d`: The letter `d` shows that it is a directory
*   `-`: The hyphen shows that it is a file
*   `l`: The letter `l` shows that it is a symbolic link
*   `p`: The letter `p` shows that it is a named pipe; a special file that facilitates the communication between programs
*   `s`: The letter `s` shows that it is a socket, similar to the pipe but with bi-directional and network communications
*   `b`: The letter `b` shows that it is a block device; a file that corresponds to a hardware device
*   `c`: The letter `c` shows that it is a character device; similar to a block device

The permission string is a 10-bit string. The first bit is reserved for the file type. The next nine bits determine the permissions by dividing them into 3-bit packets. Each packet is expressed by an **octal number** (because an octal number has three bytes). Thus, permissions are represented using a power of two:

*   `read` is 2 ^ 2 (two to the power of two), which equals 4
*   `write` is 2 ^ 1 (two to the power of one), which equals 2
*   `execute` is 2 ^ 0 (two to the power of zero), which equals 1

In this respect, file permissions should be represented according to the following diagram:

![Figure 2.27 – File permissions explained](img/Figure_02.27_B19682.jpg)

Figure 2.27 – File permissions explained

In the preceding diagram, you have the permissions shown as a string of nine characters, just like you would see them in the `ls -la` output. The row is divided into three different sections, one for owner/user, one for group, and one for other/world. These are shown in the first two rows. The other two rows show you the types of permissions (`read`, `write`, and `execute`) and the octal numbers in the following paragraph.

This is useful as it relates the octal representations to the character representations of permissions. Thus, if you were to translate a permission shown as `rwx r-x` into octal, based on the preceding diagram, you could easily say it is `755`. This is because, for the first group, the owner, you have all of them active (`rwx`), which translates into *4+2+1=7*. For the second group, you have only two permissions active, `r` and `x`, which translates into *4+1=5*. Finally, for the last group, you have also two permissions active, similar to the second group (`r` and `x`), which translates to *4+1=5*. Now you know that the permission in the octal is `755`.

As an exercise, you should try to translate into octal the following permissions:

*   `rwx rwx`
*   `rwx r-x`
*   `rwx r-x - - -`
*   `rwx - - - - - -`
*   `rw-` `rw- rw-`
*   `rw- rw- r - -`
*   `rw- rw- - - -`
*   `rw- r- -` `r- -`
*   `rw- r- - - - -`
*   `rw- - - - - - -`
*   `r - - - - - - - -`

Important note

There are some other vital commands such as `umask`, `chown`, `chmod`, and `chgrp`, which are used to change or set the default creation mode, owner, mode (access permissions), and group, respectively. They will be briefly introduced here as they involve setting the file’s properties, but for a more detailed description, please refer to [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users* *and Groups*.

### Commands for file compression, uncompression, and archiving

In Linux, the standard tool for archiving is called `tar`, from tape archive. It was initially used in Unix to write files to external tape devices for archiving. Nowadays, in Linux, it is also used to write to a file in a compressed format. Other popular formats, apart from `tar` archives, are `gzip` and `bzip` for compressed archives, together with the popular `zip` from Windows. Now let’s look at the `tar` command in detail.

#### The tar command for compressing and un-compressing

This command is used with options and does not offer compression by default. To use compression, we would need to use specific options. Here are some of the most useful arguments available for `tar`:

*   `tar -c`: Creates an archive
*   `tar -r`: Appends files to an already existing archive
*   `tar -u`: Appends only changed files to an existing archive
*   `tar -A`: Appends an archive to the end of another archive
*   `tar -t`: Lists the contents of the archive
*   `tar -x`: Extracts the archive contents
*   `tar -z`: Uses `gzip` compression for the archive
*   `tar -j`: Uses `bzip2` compression for the archive
*   `tar -v`: Uses verbose mode by printing extra information on the screen
*   `tar -p`: Restores original permission and ownership for the extracted files
*   `tar -f`: Specifies the name of the output file

There is a chance that in your daily tasks, you will have to use these arguments in combination with each other.

For example, to create an archive of the `files` directory, we used the `-cvf` arguments combined, as shown here:

![Figure 2.28 – Using the tar command](img/Figure_02.28_B19682.jpg)

Figure 2.28 – Using the tar command

The archive created is not compressed. To use compression, we would need to add the `-z` or `-j` arguments. Next, we will use the `-z` option for the `gzip` compression algorithm. See the following example and compare the size of the two archive files. As a general rule, it is advised to use an extension for such files:

![Figure 2.29 – Compressing a tar archive using gzip](img/Figure_02.29_B19682.jpg)

Figure 2.29 – Compressing a tar archive using gzip

To uncompress a tar archive, you can use the `-x` option (shown at the beginning of this subsection). For example, let us uncompress the `files-archive.tar` file that we created earlier in this subsection, and also add a target directory for the uncompressed files to be added to by using the `-C` option. The target directory needs to be created beforehand. To do this, we will use the following commands:

```
mkdir uncompressed-directory
tar -xvf files-archive.tar -C uncompressed-directory
```

This will extract the files from the archive and add them to the uncompressed-directory directory. To uncompress a `gzip`-compressed archive, for example, `files-archive-gzipped.tar.gz`, we will add the `-z` option to the ones already used in the previous command, as shown in the following snippet:

```
tar -xvzf files-archive-gzipped.tar.gz -C uncompressed-directory
```

There you go, now you know how to archive and unarchive files in Linux. There are other useful archiving tools in Linux, but `tar` is still the most commonly used one. Feel free to explore the others or other useful options for `tar`.

### Commands for locating files

Locating files in Linux is an essential task for any system administrator. As Linux systems contain vast numbers of files, finding files might be an intimidating task. Nevertheless, you have handy tools at your disposal, and knowing how to use them will be one of your greatest assets. Among these commands, we will discuss `locate`, `which`, `whereis`, and `find` in the following sections.

#### The locate command

The `locate` command is not installed by default on Ubuntu. To install it, use the following command to create an index of all the file locations on your system:

```
sudo apt install mlocate
```

Thus, when you execute the command, it searches for your file inside the database. It uses the `updatedb` command as its partner.

Before starting to use the locate command, you should execute `updatedb` to update the location database. After you do that, you can start locating files. In the following example, we will locate any file that has `new-report` in its name:

![Figure 2.30 – Using the locate command](img/Figure_02.30_B19682.jpg)

Figure 2.30 – Using the locate command

If we were to search for a file with a more generic name, such as `presentation`, the output would be too long and irrelevant. Here is an example where we used output redirection to a file and the `wc` (word count) command to show only the number of lines, words, and bytes of the file to the standard output:

![Figure 2.31 – Using the locate command with output redirection and the wc command](img/Figure_02.31_B19682.jpg)

Figure 2.31 – Using the locate command with output redirection and the wc command

In the preceding output, the resulting file has eight lines. This means that there were eight files located that have the string `presentation` in their name. The exact number is used for the words inside the file, as there are no spaces between the paths, so every line is detected as a single word. Also, the resulting file has 663 bytes. Feel free to experiment with other strings. For more options for the `locate` command, please refer to the Linux manual pages.

#### The which command

This command locates an executable file (program or command) in the shell’s search path. For example, to locate the `ls` command, type the following:

```
packt@neptune:~$ which ls
/usr/bin/ls
```

You see that the output is the path of the `ls` command: `/usr/bin/ls`.

Now try it with the `cd` command:

```
which cd
```

You will see that there is no output. This is because the `cd` command is built inside the shell and has no other location for the command to show.

#### The whereis command

This command finds *only* executable files, documentation files, and source code files. Therefore, it might not find what you want, so use it with caution:

![Figure 2.32 – Using the whereis command](img/Figure_02.32_B19682.jpg)

Figure 2.32 – Using the whereis command

Once again, the output for the `cd` command shows nothing relevant, as it is a built-in shell command. As for the `ls` command, the output shows the location of the command itself and the location of the manual pages.

#### The find command

This command is one of the most powerful commands in Linux. It can search for files in directories and subdirectories based on certain criteria. It has more than 50 options. Its main drawback is the syntax, as it is somehow different from other Linux commands. The best way to learn how the `find` command works is by example. This is why we will show you a large number of examples using this command, hoping that you will become proficient in using it. To see its powerful options, please refer to the manual pages.

The following is a series of exercises using the `find` command, that we thought would be useful for you to know. We will provide the commands to use, but we will not provide you with all the resulting outputs, as some can be fairly long.

*   Find, inside the root directory, all the files that have the `e100` string in the name and print them to the standard output:

    ```
    sudo find / -name e100 -print
    ```

*   Find, inside the root directory, all the files that have the `file` string in their name and are of type `file`, and print the results to the standard output:

    ```
    sudo find / -name file -type f -print
    ```

*   Find all the files that have the `print` string in their name, by looking only inside the `/opt`, `/usr`, and `/``var` directories:

    ```
    sudo find /opt /usr /var -name print -type f -print
    ```

*   Find all the files in the root directory that have the `.``conf` extension:

    ```
    sudo find / type f -name "*.conf"
    ```

*   Find all the files in the root directory that have the `file` string in their name and no extension:

    ```
    sudo find / -type f -name "file*.*"
    ```

*   Find, in the root directory, all the files with the following extensions: `.c`, `.sh`, and `.py`, and add the list to a file named `findfile`:

    ```
    sudo find / -type f \( -name "*.c" -o -name "*.sh" -o -name "*.py" \) > findfile
    ```

*   Find, in the root directory, all the files with the `.c` extension, sort them, and add them to a file:

    ```
    sudo find / -type f -name "*.c" -print | sort > findfile2
    ```

*   Find all the files in the root directory, with the permission set to `0664`:

    ```
    sudo find / -type f -perm 0664
    ```

*   Find all the files in the root directory that are read-only (have read-only permission) for their owner:

    ```
    sudo find / -type f -perm /u=r
    ```

*   Find all the files in the root directory that are executable:

    ```
    sudo find / -type f -perm /a=x
    ```

*   Find all the files inside the root directory that were modified two days ago:

    ```
    sudo find / -type f -mtime 2
    ```

*   Find all the files in the root directory that have been accessed in the last two days:

    ```
    sudo find / -type f -atime 2
    ```

*   Find all the files that have been modified in the last two to five days:

    ```
    sudo find / -type f -mtime +2 -mtime -5
    ```

*   Find all the files that have been modified in the last 10 minutes:

    ```
    sudo find / -type f -mmin -10
    ```

*   Find all the files that have been created in the last 10 minutes:

    ```
    sudo find / -type f -cmin -10
    ```

*   Find all the files that have been accessed in the last 10 minutes:

    ```
    sudo find / -type f -amin -10
    ```

*   Find all the files that are 5 MB in size:

    ```
    sudo find / -type f -size 5M
    ```

*   Find all the files that have a size between 5 and 10 MB:

    ```
    sudo find / -type f -size +5M -size -10M
    ```

*   Find all the empty files and empty directories:

    ```
    sudo find / -type f -empty
    sudo find / -type d -empty
    ```

*   Find all the largest files in the `/etc` directory and print to the standard output the first five. Please take into account that this command could be very resource heavy. Do not try to do this for your entire root directory, as you might run out of system memory:

    ```
    sudo find /etc -type f -exec ls -l {} \; | sort -n -r | head -5
    ```

*   Find the smallest first five files in `/``etc` directory:

    ```
    sudo find /etc -type f -exec ls -s {} \; | sort -n | head -5
    ```

Feel free to experiment with as many types of find options as you want. The command is very permissive and powerful. Use it with caution.

### Commands for text manipulation

`grep`, `tee`, and the more powerful ones such as `sed` and `awk`. However, we will come back to those commands in [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), when we will show you how to create and use scripts. In this section, we will only give you a hint on how to use them on the command line.

#### The grep command

This is one of the most powerful commands in Linux. It is also an extremely useful one. It has the power to search for strings inside text files. It has many powerful options too:

*   `grep -v`: Show the lines that are not according to the search criteria
*   `grep -l`: Show only the filenames that match the criteria
*   `grep -L`: Show only the lines that do *not* comply with the criteria
*   `grep -c`: A counter that shows the number of lines matching the criteria
*   `grep -n`: Show the line number where the string was found
*   `grep -i`: Searches are case insensitive
*   `grep -r`: Search recursively inside a directory structure
*   `grep -R`: Search recursively inside a directory structure *AND* follow all symbolic links
*   `grep -E`: Use extended regular expressions
*   `grep -F`: Use a strict list of strings instead of regular expressions

Here are some examples of how to use the `grep` command:

*   Find out the last time the `sudo` command was used:

    ```
    sudo grep sudo /var/log/auth.log
    ```

*   Search for the `packt` string inside text files from the `/``etc` directory:

    ```
    grep -R packt /etc
    ```

*   Show the exact line where the match was found:

    ```
    grep -Rn packt /etc
    ```

*   If you don’t want to see the filename of each file where the match was found, use the `-h` option. Then, `grep` will only show you the lines where the match was found:

    ```
    grep -Rh packt /etc
    ```

*   To show only the name of the file where the match was found, use `-l`:

    ```
    grep -Rl packt /etc
    ```

Most likely, `grep` will be used in combination with shell pipes. Here are some examples:

*   If you want to see only the directories from your current working directory, you could pipe the `ls` command output to `grep`. In the following example, we listed only the lines that start with the letter `d`, which represent directories:

    ```
    ls -la | grep '^d'
    ```

*   If you want to display the model of your CPU, you could use the following command:

    ```
    cat /proc/cpuinfo | grep -i 'Model'
    ```

You will find `grep` to be one of your closest friends as a Linux system administrator, so don’t be afraid to dig deeper into its options and hidden gems.

#### The tee command

This command is very similar to the `cat` command. Basically, it does the same thing, by copying the standard input to standard output with no alteration, but it also copies that into one or more files.

In the following example, we use the `wc` command to count the number of lines inside the `/etc/passwd/` file. We pipe the output to the `tee` command using the `-a` option (append if the file already exists), which writes it to a new file called `no-users` and prints it to the standard output at the same time. We then use the `cat` command to double-check the contents of the new file:

![Figure 2.33 – Using the tee command](img/Figure_02.33_B19682.jpg)

Figure 2.33 – Using the tee command

The `tee` command is more of an underdog of file-manipulating commands. While it is quite powerful, its use can easily be overlooked. Nevertheless, we encourage you to use its powers as often as you can.

In the following section, we will show you how to use text editors from the command line in Linux.

# Using text editors to create and edit files

Linux has several command-line text editors that you can use. There are **nano**, **Emacs**, and **Vim**, among others. Those are the most used ones. There are also **Pico**, **JOE**, and **ed** as text editors that are less frequently used than the aforementioned ones. We will cover Vim, as there is a very good chance that you will find it on any Linux system that you work with. Nevertheless, the current trend is to replace Vim with nano as the default text editor. Ubuntu, for example, does not have Vim installed by default, but CentOS does. Fedora is currently looking to make nano the default text editor. Therefore, you might want to learn nano, but for legacy purposes, Vim is a very useful tool to know.

## Using Vim to edit text files

**Vim** is the improved version of **vi**, the default text editor from Unix. It is a very powerful editing tool. This power comes with many options that can be used to ease your work, and this can be overwhelming. In this sub-section, we will introduce you to the basic commands of the text editor, just enough to help you be comfortable using it.

Vim is a mode-based editor, as its operation is organized around different modes. In a nutshell, those modes are as follows:

*   `command` mode is the default mode, waiting for a command
*   `insert` mode is the text insert mode
*   `replace` mode is the text replace mode
*   `search` mode is the special mode for searching a document

Let’s see how we can switch between these modes.

### Switching between modes

When you first open Vim, you will be introduced to an empty editor that only shows information about the version used and a few help commands. You are in `command` mode. This means that Vim is waiting for a command to operate.

To activate `insert` mode, press *I* on your keyboard. You will be able to start inserting text at the current position of your cursor. You can also press *A* (for append) to start editing to the right of your cursor’s position. Both *I* and *A* will activate `insert` mode. To exit the current mode, press the *Esc* key. It will get you back to `command` mode.

If you open a file that already has text in it, while in `command` mode, you can navigate the file using your arrow keys. As Vim inherited the vi workflow, you can also use *H* (to move left), *J* (to move down), *K* (to move up), and *L* (to move right). Those are legacy keys from a time when terminal keyboards did not have separate arrow keys.

While still in `command` mode (the default mode), you can activate `replace` mode by pressing *R* on your keyboard. You can replace the character that is right at the position of your cursor.

While in `command` mode, `search` mode is activated by pressing the */* key. Once in your mode, you can start typing a search string and then press *Enter*.

There is also `last line` mode, or `ex command` mode. This mode is activated by pressing *:*. This is an extended mode where commands such as `w` for saving the file, `q` for quitting, or `wq` for saving and quitting at the same time.

### Basic Vim commands

Working with Vim implies that you are comfortable with using keyboard shortcuts for using basic commands. We will guide you to the Vim documentation page ([https://vimdoc.sourceforge.net/](https://vimdoc.sourceforge.net/)) for all the commands available, and we will give you a quick glimpse of the most useful ones in the following image:

![Figure 2.34 – Basic Vim commands](img/Figure_02.34_B19682.jpg)

Figure 2.34 – Basic Vim commands

Vim can be quite intimidating for newcomers to Linux. There is no shame if you prefer other editors, as there are plenty to choose from. Now, we will show you a glimpse of nano.

## The nano text editor

Vim is a powerful text editor and knowing how to use it is an important thing for any system administrator. Nevertheless, other text editors are equally powerful and even easier to use.

This is the case with `.bashrc` file by using the `$EDITOR` variable. However, in Ubuntu, you can check the default editor on your system by using the following command:

![Figure 2.35 – Checking the default text editor on Ubuntu](img/Figure_02.35_B19682.jpg)

Figure 2.35 – Checking the default text editor on Ubuntu

You can invoke the nano editor by using the `nano` command on Ubuntu/Debian and Fedora/Rocky Linux or openSUSE. When you type the command, the nano editor will open, with a very straightforward interface that can be easier to use than Vim or Emacs for example. Feel free to use your preferred text editor.

# Summary

In this chapter, you learned how to work with the most commonly used commands in Linux. You now know how to manage (create, delete, copy, and move) files, how the filesystem is organized, how to work with directories, and how to view file contents. You now understand the shell and basic permissions. The skills you have learned will help you manage files in any Linux distribution and edit text files. You have learned how to work with Vim, one of the most widely used command-line text editors in Linux. Those skills will help you to learn how to use other text editors such as nano and Emacs. You will use these skills in almost every chapter of this book, as well as in your everyday job as a system administrator.

In the next chapter, you will learn how to manage packages, including how to install, remove, and query packages in both Debian and Red Hat-based distributions. This skill is important for any administrator and must be part of any basic training.

# Questions

In our second chapter, we covered the Linux filesystem and the basic commands that will serve as the foundation for the entire book. Here are some questions for you to test your knowledge and for further practice:

1.  What is the command that creates a compressed archive with all the files inside the `/etc` directory that use the `.``conf` extension?

`tar` command just as shown in this chapter.

1.  What is the command that lists the first five files inside `/etc` and sorts them by dimension in descending order?

`find` combined with `sort` and `head`.

1.  What command creates a hierarchical directory structure?

`mkdir` just as shown in this chapter.

1.  What is the command that searches for files with three different extensions inside the root?

`find` command.

1.  Find out which commands inside Linux have the **Set owner User ID** (**SUID**) set up.

`find` command with the `-``perm` parameter.

1.  Which command is used to create a file with 1,000 lines of randomly generated words (one word per line)?

`shuf` command (not shown in this chapter).

1.  Perform the same exercise as before, but this time generate a file with 1,000 randomly generated numbers.

`for` loop.

1.  How do you find out when `sudo` was last used and which commands were executed by it?

`grep` command.

# Further reading

For more information about what was covered in this chapter, please refer to the following Packt titles:

*   *Fundamentals of Linux*, by *Oliver Pelz*
*   *Mastering Ubuntu Server – Fourth Edition*, by *Jay LaCroix*