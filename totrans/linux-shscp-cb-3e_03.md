# File In, File Out

In this chapter, we will be covering the following recipes:

*   Generating files of any size
*   The intersection and set difference (A-B) on text files
*   Finding and deleting duplicate files
*   Working with file permissions, ownership, and the sticky bit
*   Making files immutable
*   Generating blank files in bulk
*   Finding symbolic links and their targets
*   Enumerating file type statistics
*   Using loopback files
*   Creating ISO files and hybrid ISO
*   Finding the difference between files, and patching
*   Using head and tail for printing the last or first 10 lines
*   Listing only directories - alternative methods
*   Fast command-line navigation using `pushd` and `popd`
*   Counting the number of lines, words, and characters in a file
*   Printing the directory tree
*   Manipulating video and image files

# Introduction

Unix provides a file-style interface to all devices and system features. The special files provide direct access to devices such as USB sticks and disk drives and provide access to system functions such as memory usage, sensors, and the process stack. For example, the command terminal we use is associated with a device file. We can write to the terminal by writing to the corresponding device file. We can access directories, regular files, block devices, character-special devices, symbolic links, sockets, named pipes, and so on as files. Filename, size, file type, modification time, access time, change time, inode, links associated, and the filesystem the file is on are all attributes and properties files can have. This chapter deals with recipes to handle operations or properties related to files.

# Generating files of any size

A file of random data is useful for testing. You can use such files to test application efficiency, to confirm that an application is truly input-neutral, to confirm there's no size limitations in your application, to create loopback filesystems (**loopback files** are files that can contain a filesystem itself and these files can be mounted similarly to a physical device using the `mount` command), and more. Linux provides general utilities to construct such files.

# How to do it...

The easiest way to create a large file of a given size is with the `dd` command. The `dd` command clones the given input and writes an exact copy to the output. Input can be `stdin`, a device file, a regular file, and so on. Output can be `stdout`, a device file, a regular file, and so on. An example of the `dd` command is as follows:

```
$ dd if=/dev/zero of=junk.data bs=1M count=1
1+0 records in
1+0 records out
1048576 bytes (1.0 MB) copied, 0.00767266 s, 137 MB/s

```

This command creates a file called `junk.data` containing exactly 1 MB of zeros.

Let's go through the parameters:

*   `if` defines the `input` file
*   `of` defines the `output` file
*   `bs` defines bytes in a block
*   `count` defines the number of blocks to be copied

Be careful while using the `dd` command as root, as it operates on a low level with the devices. A mistake could wipe your disk or corrupt the data. Double-check your `dd` command syntax, especially your of `=` parameter for accuracy.
In the previous example, we created a 1 MB file, by specifying `bs` as 1 MB with a count of 1\. If `bs` was set to `2M` and `count` to `2`, the total file size would be 4 MB.

We can use various units for **block****size** (**bs**). Append any of the following characters to the number to specify the size:

| **Unit size** | **Code** |
| Byte (1 B) | `C` |
| Word (2 B) | `W` |
| Block (512 B) | `B` |
| Kilobyte (1024 B) | `K` |
| Megabyte (1024 KB) | `M` |
| Gigabyte (1024 MB) | `G` |

We can generate a file of any size using **bs**. Instead of MB we can use any other unit notations, such as the ones mentioned in the previous table.

`/dev/zero` is a character special device, which returns the zero byte (`\0`).

If the input parameter (`if`) is not specified, dd will read input from `stdin`. If the output parameter (`of`) is not specified, `dd` will use `stdout`.

The `dd` command can be used to measure the speed of memory operations by transferring a large quantity of data to `/dev/null` and checking the command output (for example, `1048576 bytes (1.0 MB) copied, 0.00767266 s, 137 MB/s`, as seen in the previous example).

# The intersection and set difference (A-B) on text files

Intersection and set difference operations are common in mathematics classes on set theory. Similar operations on strings are useful in some scenarios.

# Getting ready

The `comm` command is a utility to perform a comparison between two sorted files. It displays lines that are unique to file 1, file 2, and lines in both files. It has options to suppress one more column, making it easy to perform intersection and difference operations.

*   **Intersection**: The intersection operation will print the lines the specified files have in common with one another
*   **Difference**: The difference operation will print the lines the specified files contain and that are not the same in all of those files
*   **Set difference**: The set difference operation will print the lines in file `A` that do not match those in all of the set of files specified (`B` plus `C`, for example)

# How to do it...

Note that `comm` takes two sorted files as input. Here are our sample input files:

```
$ cat A.txt
apple
orange
gold
silver
steel
iron

$ cat B.txt
orange
gold
cookies
carrot

$ sort A.txt -o A.txt ; sort B.txt -o B.txt

```

1.  First, execute `comm` without any options:

```
        $ comm A.txt B.txt 
 apple
 carrot
 cookies
 gold
 iron
 orange
 silver
 steel

```

 The first column of the output contains lines that are only in `A.txt`. The second column contains lines that are only in `B.txt`. The third column contains the common lines from `A.txt` and `B.txt`. Each of the columns are delimited using the tab (`\t`) character.

2.  In order to print the intersection of two files, we need to remove the first and second columns and print the third column. The `-1` option removes the first column, and the `-2` option removes the second column, leaving the third column:

```
        $ comm A.txt B.txt -1 -2
 gold
 orange

```

3.  Print only the lines that are uncommon between the two files by removing column `3`:

```
        $ comm A.txt B.txt  -3
 apple
 carrot
 cookies
 iron
 silver
 steel

```

 This output uses two columns with blanks to show the unique lines in file1 and file2\. We can make this more readable as a list of unique lines by merging the two columns into one, like this:

```
        apple
 carrot
 cookies
 iron
 silver
 steel

```

4.  The lines can be merged by removing the tab characters with `tr` (discussed in [Chapter 2](36986eeb-141a-496a-a6b1-4f78f612c14e.xhtml), *Have a Good Command*)

```
        $ comm A.txt B.txt  -3 | tr -d '\t'
 apple
 carrot
 cookies
 iron
 silver
 steel

```

5.  By removing the unnecessary columns, we can produce the set difference for `A.txt` and `B.txt`, as follows:

*   Set difference for `A.txt`:

```
                $ comm A.txt B.txt -2 -3

```

 `-2 -3` removes the second and third columns

*   Set difference for `B.txt`:

```
                $ comm A.txt B.txt -1 -3

```

 `-2 -3` removes the second and third columns

# How it works...

These command-line options reduce the output:

*   `-1`: Removes the first column
*   `-2`: Removes the second column
*   `-3`: Removes the third column

The set difference operation enables you to compare two files and print all the lines that are in the `A.txt` or `B.txt` file excluding the common lines in `A.txt` and `B.txt`. When `A.txt` and `B.txt` are given as arguments to the `comm` command, the output will contain column-1 with the set difference for `A.txt` with regard to `B.txt` and column-2 will contain the set difference for `B.txt` with regard to `A.txt`.

The `comm` command will accept a `-` character on the command line to read one file from `stdin`. This provides a way to compare more than one file with a given input.

Suppose we have a `C.txt` file, like this:

```
    $> cat C.txt
 pear
 orange
 silver
 mithral

```

We can compare the `B.txt` and `C.txt` files with `A.txt`, like this:

```
    $> sort B.txt C.txt | comm - A.txt
 apple
 carrot
 cookies
 gold
 iron
 mithral
 orange
 pear
 silver
 steel

```

# Finding and deleting duplicate files

If you need to recover backups or you use your laptop in a disconnected mode or download images from a phone, you'll eventually end up with duplicates: files with the same content. You'll probably want to remove duplicate files and keep a single copy. We can identify duplicate files by examining the content with shell utilities. This recipe describes finding duplicate files and performing operations based on the result.

# Getting ready

We identify the duplicate files by comparing file content. Checksums are ideal for this task. Files with the same content will produce the same checksum values.

# How to do it...

Follow these steps for finding or deleting duplicate files:

1.  Generate some test files:

```
        $ echo "hello" > test ; cp test test_copy1 ; cp test test_copy2;
 $ echo "next" > other;
 # test_copy1 and test_copy2 are copy of test

```

2.  The code for the script to remove the duplicate files uses `awk`, an interpreter that's available on all Linux/Unix systems:

```
        #!/bin/bash 
        #Filename: remove_duplicates.sh 
        #Description: Find and remove duplicate files and 
        # keep one sample of each file.
        ls -lS --time-style=long-iso | awk 'BEGIN { 
          getline; getline; 
          name1=$8; size=$5 
        } 
        { 
           name2=$8; 
           if (size==$5) 
        { 
           "md5sum "name1 | getline; csum1=$1; 
           "md5sum "name2 | getline; csum2=$1; 
           if ( csum1==csum2 ) 
           { 
              print name1; print name2 
            } 
        }; 

        size=$5; name1=name2; 
        }' | sort -u > duplicate_files 

         cat duplicate_files | xargs -I {} md5sum {} | \ 
         sort | uniq -w 32 | awk '{ print $2 }' | \ 
         sort -u > unique_files 

         echo Removing..  
         comm duplicate_files unique_files -3 | tee /dev/stderr | \
               xargs rm 
         echo Removed duplicates files successfully.

```

3.  Run the code as follows:

```
        $ ./remove_duplicates.sh

```

# How it works...

The preceding code will find the copies of the same file in a directory and remove all except one copy of the file. Let's go through the code and see how it works.

`ls -lS` lists the details of the files in the current folder sorted by file size. The `--time-style=long-iso` option tells `ls` to print dates in the ISO format. `awk` reads the output of `ls -lS` and performs comparisons on columns and rows of the input text to find duplicate files.

The logic behind the code is as follows:

*   We list the files sorted by size, so files of the same size will be adjacent. The first step in finding identical files is to find ones with the same size. Next, we calculate the checksum of the files. If the checksums match, the files are duplicates and one set of the duplicates are removed.
*   The `BEGIN{}` block of `awk` is executed before the main processing. It reads the "total" lines and initializes the variables. The bulk of the processing takes place in the `{}` block, when `awk` reads and processes the rest of the `ls` output. The `END{}` block statements are executed after all input has been read. The output of `ls -lS` is as follows:

```
        total 16
 -rw-r--r-- 1 slynux slynux 5 2010-06-29 11:50 other
 -rw-r--r-- 1 slynux slynux 6 2010-06-29 11:50 test
 -rw-r--r-- 1 slynux slynux 6 2010-06-29 11:50 test_copy1
 -rw-r--r-- 1 slynux slynux 6 2010-06-29 11:50 test_copy2

```

*   The output of the first line tells us the total number of files, which in this case is not useful. We use `getline` to read the first line and then dump it. We need to compare each of the lines and the following line for size. In the `BEGIN` block, we read the first line and store the name and size (which are the eighth and fifth columns). When `awk` enters the `{}` block, the rest of the lines are read, one by one. This block compares the size obtained from the current line and the previously stored size in the `size` variable. If they are equal, it means that the two files are duplicates by size and must be further checked by `md5sum`.

We have played some tricks on the way to the solution.

The external command output can be read inside `awk` as follows:

```
      "cmd"| getline

```

Once the line is read, the entire line is in `$0` and each column is available in `$1`, `$2`, ..., `$n`. Here, we read the md5sum checksum of files into the `csum1` and `csum2` variables. The `name1` and `name2` variables store the consecutive filenames. If the checksums of two files are the same, they are confirmed to be duplicates and are printed.

We need to find a file from each group of duplicates so we can remove all other duplicates. We calculate the `md5sum` value of the duplicates and print one file from each group of duplicates by finding unique lines, comparing `md5sum` from each line using `-w 32` (the first 32 characters in the `md5sum` output; usually, the `md5sum` output consists of a 32-character hash followed by the filename). One sample from each group of duplicates is written to `unique_files`.

Now, we need to remove the files listed in `duplicate_files`, excluding the files listed in `unique_files`. The `comm` command prints files in `duplicate_files` but not in `unique_files`.

For that, we use a set difference operation (refer to the recipes on intersection, difference, and set difference).

`comm` only processes sorted input. Therefore, `sort -u` is used to filter `duplicate_files` and `unique_files`.

The `tee` command is used to pass filenames to the `rm` command as well as `print`. The `tee` command sends its input to both `stdout and a file`. We can also print text to the terminal by redirecting to `stderr`. `/dev/stderr` is the device corresponding to `stderr` (standard error). By redirecting to a `stderr` device file, text sent to `stdin` will be printed in the terminal as standard error.

# Working with file permissions, ownership, and the sticky bit

File permissions and ownership are one of the distinguishing features of the Unix/Linux filesystems. These features protect your information in a multi-user environment. Mismatched permissions and ownership can also make it difficult to share files. These recipes explain how to use a file's permission and ownership effectively.

Each file possesses many types of permissions. Three sets of permissions (user, group, and others) are commonly manipulated.

The **user** is the owner of the file, who commonly has all access permitted. The **group** is the collection of users (as defined by the system administrator) that may be permitted some access to the file. **Others** are any users other than the owner or members of the owner's group.

The `ls` command's `-l` option displays many aspects of the file including type, permissions, owner, and group:

```
    -rw-r--r-- 1 slynux users  2497  2010-02-28 11:22 bot.py
 drwxr-xr-x 2 slynux users  4096  2010-05-27 14:31 a.py
 -rw-r--r-- 1 slynux users  539   2010-02-10 09:11 cl.pl

```

The first column of the output defines the file type as follows:

*   `-`: This is used if it is a regular file
*   `d`: This is used if it is a directory
*   `c`: This is used for a character device
*   `b`: This is used for a block device
*   `l`: This is used if it is a symbolic link
*   `s`: This is used for a socket
*   `p`: This is used for a pipe

The next nine characters are divided into three groups of three letters each (--- --- ---). The first three characters correspond to the permissions of the user (owner), the second sets of three characters correspond to the permissions of the group, and the third sets of three characters correspond to the permissions of others. Each character in the nine-character sequence (nine permissions) specifies whether permission is set or unset. If the permission is set, a character appears in the corresponding position, otherwise a `-` character appears in that position, which means that the corresponding permission is unset (unavailable).

The three common letters in the trio are:

*   `r Read`: When this is set, the file, device, or directory can be read.
*   `w Write`: When this is set, the file, device, or directory can be modified. On folders, this defines whether files can be created or deleted.
*   `x execute`: When this is set, the file, can be executed. On folders, this defines whether the files in the folder can be accessed.

Let's take a look at what each of these three character sets mean for the user, group, and others:

*   **User** (permission string: `rwx------`): These define the options a user has. Usually, the user's permission is `rw-` for a data file and `rwx` for a script or executable. The user has one more special permission called `setuid` (`S`), which appears in the position of execute (`x`). The `setuid` permission enables an executable file to be executed effectively as its owner, even when the executable is run by another user. An example of a file with `setuid` permission set is `-rwS------`.
*   **Group** (permission string: `---rwx---`): The second set of three characters specifies the group permissions. Instead of `setuid`, the group has a `setgid` (`S`) bit. This enables the item to run an executable file with an effective group as the owner group. But the group, which initiates the command, may be different. An example of group permission is `----rwS---`.
*   **Others** (permission string: `------rwx`): Other permissions appear as the last three characters in the permission string. If these are set, anyone can access this file or folder. As a rule you will want to set these bits to `---`.

Directories have a special permission called a **sticky bit**. When a sticky bit is set for a directory, only the user who created the directory can delete the files in the directory, even if the group and others have write permissions. The sticky bit appears in the position of execute character (`x`) in the others permission set. It is represented as character `t` or `T`. The `t` character appears in the `x` position if the execute permission is unset and the sticky bit is set. If the sticky bit and the execute permission are set, the `T` character appears in the `x` position. Consider this example:

```
    ------rwt , ------rwT

```

A typical example of a directory with sticky bit turned on is `/tmp`, where anyone can create a file, but only the owner can delete one.

In each of the `ls -l` output lines, the string `slynux users` corresponds to the user and group. Here, `slynux` is the owner who is a member of the group users.

# How to do it...

In order to set permissions for files, we use the `chmod` command.

Assume that we need to set the permission, `rwx rw- r-`.

Set these permissions with chmod:

```
    $ chmod u=rwx g=rw o=r filename

```

The options used here are as follows:

*   `u`: This specifies user permissions
*   `g`: This specifies group permissions
*   `o`: This specifies others permissions

Use `+` to add permission to a user, group, or others, and use `-` to remove the permissions.

Add the executable permission to a file, which has the permission, `rwx rw- r-`:

```
    $ chmod o+x filename

```

This command adds the `x` permission for others.

Add the executable permission to all permission categories, that is, for user, group, and others:

```
    $ chmod a+x filename

```

Here `a` means all.

In order to remove a permission, use `-`. For example, **$ chmod a-x filename**.

Permissions can be denoted with three-digit octal numbers in which each digit corresponds to user, group, and other, in that order.

Read, write, and execute permissions have unique octal numbers, as follows:

*   `r` = 4
*   `w` = 2
*   `x` = 1

We calculate the required combination of permissions by adding the octal values. Consider this example:

*   `rw-` = 4 + 2 = 6
*   `r-x` = 4 + 1 = 5

The permission `rwx rw- r--` in the numeric method is as follows:

*   `rwx` = 4 + 2 + 1 = 7
*   `rw-` = 4 + 2 = 6
*   `r--` = 4

Therefore, `rwx rw- r--` is equal to `764`, and the command to set the permissions using octal values is `$ chmod 764 filename`.

# There's more...

Let's examine more tasks we can perform on files and directories.

# Changing ownership

The `chown` command will change the ownership of files and folders:

```
    $ chown user.group filename

```

Consider this example:

```
    $ chown slynux.users test.sh

```

Here, `slynux` is the user, and `users` is the group.

# Setting the sticky bit

The sticky bit can be applied to directories. When the sticky bit is set, only the owner can delete files, even though others have write permission for the folder.

The sticky bit is set with the `+t` option to `chmod`:

```
    $ chmod a+t directory_name

```

# Applying permissions recursively to files

Sometimes, you may need to change the permissions of all the files and directories inside the current directory recursively. The `-R` option to `chmod` supports recursive changes:

```
    $ chmod 777 . -R

```

The `-R` option specifies to change the permissions recursively.

We used `.` to specify the path as the current working directory. This is equivalent to `$ chmod 777 "$(pwd)" -R`.

# Applying ownership recursively

The `chown` command also supports the `-R` flag to recursively change ownership:

```
    $ chown user.group . -R

```

# Running an executable as a different user (setuid)

Some executables need to be executed as a user other than the current user. For example, the http server may be initiated during the boot sequence by root, but the task should be owned by the `httpd` user. The `setuid` permission enables the file to be executed as the file owner when any other user runs the program.

First, change the ownership to the user that needs to execute it and then log in as the user. Then, run the following commands:

```
    $ chmod +s executable_file
 # chown root.root executable_file
 # chmod +s executable_file
 $ ./executable_file

```

Now it executes as the root user regardless of who invokes it.

The `setuid` is only valid for Linux ELF binaries. You cannot set a shell script to run as another user. This is a security feature.

# Making files immutable

The Read, Write, Execute, and Setuid fields are common to all Linux file systems. The **Extended File Systems** (ext2, ext3, and ext4) support more attributes.

One of the extended attributes makes files immutable. When a file is made immutable, any user or super user cannot remove the file until the immutable attribute is removed from the file. You can determine the type of filesystem with the `df -T` command, or by looking at the `/etc/mtab` file. The first column of the file specifies the partition device path (for example, `/dev/sda5`) and the third column specifies the filesystem type (for example, ext3).

Making a file immutable is one method for securing files from modification. One example is to make the `/etc/resolv.conf` file immutable. The `resolv.conf` file stores a list of DNS servers, which convert domain names (such as packtpub.com) to IP addresses. The DNS server is usually your ISP's DNS server. However, if you prefer a third-party server, you can modify `/etc/resolv.conf` to point to that DNS. The next time you connect to your ISP, `/etc/resolv.conf` will be overwritten to point to ISP's DNS server. To prevent this, make `/etc/resolv.conf` immutable.

In this recipe, we will see how to make files immutable and make them mutable when required.

# Getting ready

The `chattr` command is used to change extended attributes. It can make files immutable, as well as modify attributes to tune filesystem sync or compression.

# How to do it...

To make the files immutable, follow these steps:

1.  Use `chattr` to make a file immutable:

```
        # chattr +i file

```

2.  The file is now immutable. Try the following command:

```
        rm file
 rm: cannot remove `file': Operation not permitted

```

3.  In order to make it writable, remove the immutable attribute, as follows:

```
        chattr -i file

```

# Generating blank files in bulk

Scripts must be tested before they are used on a live system. We may need to generate thousands of files to confirm that there are no memory leaks or processes left hanging. This recipe shows how to generate blank files.

# Getting ready

The `touch` command creates blank files or modifies the timestamp of existing files.

# How to do it...

To generate blank files in bulk, follow these steps:

1.  Invoking the touch command with a non-existent filename creates an empty file:

```
        $ touch filename

```

2.  Generate bulk files with a different name pattern:

```
        for name in {1..100}.txt 
        do 
          touch $name 
        done 

```

In the preceding code, `{1..100}` will be expanded to a string `1, 2, 3, 4, 5, 6, 7...100`. Instead of `{1..100}.txt`, we can use various shorthand patterns such as `test{1..200}.c`, `test{a..z}.txt`, and so on.

If a file already exists, the `touch` command changes all timestamps associated with the file to the current time. These options define a subset of timestamps to be modified:

*   `touch -a`: This modifies the access time
*   `touch -m`: This modifies the modification time

Instead of the current time, we can specify the time and date:

```
      $ touch -d "Fri Jun 25 20:50:14 IST 1999" filename

```

The date string used with `-d` need not be in this exact format. It will accept many simple date formats. We can omit time from the string and provide only dates such as *Jan 20*, *2010*.

# Finding symbolic links and their targets

Symbolic links are common in Unix-like systems. Reasons for using them range from convenient access, to maintaining multiple versions of the same library or program. This recipe will discuss the basic techniques for handling symbolic links.

Symbolic links are pointers to other files or folders. They are similar in function to aliases in MacOS X or shortcuts in Windows. When symbolic links are removed, it does not affect the original file.

# How to do it...

The following steps will help you handle symbolic links:

1.  To create a symbolic link run the following command:

```
        $ ln -s target symbolic_link_name

```

 Consider this example:

```
        $ ln -l -s /var/www/ ~/web

```

 This creates a symbolic link (called **web**) in the current user's home directory, which points to `/var/www/`.

2.  To verify the link was created, run this command:

```
        $ ls -l ~/web
 lrwxrwxrwx 1 slynux slynux 8 2010-06-25 21:34 web -> /var/www

```

 `web -> /var/www` specifies that `web` points to `/var/www`.

3.  To print symbolic links in the current directory, use this command:

```
        $ ls -l | grep "^l"

```

4.  To print all symbolic links in the current directory and subdirectories, run this command:

```
        $ find . -type l -print

```

5.  To display the target path for a given symbolic link, use the `readlink` command:

```
        $ readlink web
 /var/www

```

# How it works...

When using `ls` and `grep` to display symbolic links in the current folder, the `grep ^l` command filters the `ls -l` output to only display lines starting with `l`. The `^` specifies the start of the string. The following `l` specifies that the string must start with l, the identifier for a link.

When using `find`, we use the argument -`type``l`, which instructs find to search for symbolic link files. The `-print` option prints the list of symbolic links to the standard output (`stdout`). The initial path is given as the current directory.

# Enumerating file type statistics

Linux supports many file types. This recipe describes a script that enumerates through all the files inside a directory and its descendants, and prints a report with details on types of files (files with different file types), and the count of each file type. This recipe is an exercise on writing scripts to enumerate through many files and collect details.

# Getting ready

On Unix/Linux systems, file types are not defined by the file extension (as Microsoft Windows does). Unix/Linux systems use the file command, which examines the file's contents to determine a file's type. This recipe collects file type statistics for a number of files. It stores the count of files of the same type in an associative array.

The associative arrays are supported in bash version 4 and newer.

# How to do it...

To enumerate file type statistics, follow these steps:

1.  To print the type of a file, use the following command:

```
        $ file filename

 $ file /etc/passwd
 /etc/passwd: ASCII text

```

2.  Print the file type without the filename:

```
        $ file -b filename
 ASCII text

```

3.  The script for file statistics is as follows:

```
     #!/bin/bash 
     # Filename: filestat.sh 

     if [ $# -ne 1 ]; 
     then 
       echo "Usage is $0 basepath"; 
       exit 
     fi 
     path=$1 

     declare -A statarray; 

     while read line; 
     do 
       ftype=`file -b "$line" | cut -d, -f1` 
       let statarray["$ftype"]++; 

     done < (find $path -type f -print) 

     echo ============ File types and counts ============= 
     for ftype in "${!statarray[@]}"; 
     do 
       echo $ftype :  ${statarray["$ftype"]} 
     done 

```

The usage is as follows:

```
        $ ./filestat.sh /home/slynux/temp

```

5.  A sample output is shown as follows:

```
        $ ./filetype.sh /home/slynux/programs
 ============ File types and counts =============
 Vim swap file : 1
 ELF 32-bit LSB executable : 6
 ASCII text : 2
 ASCII C program text : 10

```

# How it works...

This script relies on the associative array `statarray`. This array is indexed by the type of file: **PDF**, **ASCII**, and so on. Each index holds the count for that type of file. It is defined by the `declare -A statarray` command.

The script then consists of two loops: a while loop, that processes the output from the find command, and a `for` loop, that iterates through the indices of the `statarray` variable and generates output.

The while loop syntax looks like this:

```
while read line; 
do something 
done < filename 

```

For this script, we use the output of the find command instead of a file as input to `while`.

The `(find $path -type f -print)` command is equivalent to a filename, but it substitutes the filename with a subprocess output.

Note that the first `<` is for input redirection and the second `<` is for converting the subprocess output to a filename. Also, there is a space between these two so the shell won't interpret it as the `<<` operator.

The `find` command uses the `-type``f` option to return a list of files under the subdirectory defined in $path. The filenames are read one line at a time by the `read` command. When the read command receives an **EOF** (**End of File**), it returns a *fail* and the `while` command exits.

Within the `while` loop, the file command is used to determine a file's type. The `-b` option is used to display the file type without the name.

The file command provides more details than we need, such as image encoding and resolution (in the case of an image file). The details are comma-separated, as in the following example:

```
    $ file a.out -b
 ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
    dynamically linked (uses shared libs), for GNU/Linux 2.6.15, not
    stripped

```

We need to extract only `ELF 32-bit LSB executable` from the previous details. Hence, we use the `-d,` option to specify `,` as the delimiter and `-f1` to select the first field.

`<(find $path -type f -print)` is equivalent to a filename, but it substitutes the filename with a subprocess output. Note that the first `<` is for input redirection and the second `<` is for converting the subprocess output to a filename. Also, there is a space between these two so that the shell won't interpret it as the `<<` operator.

In Bash 3.x and higher, we have a new operator `<<<` that lets us use a string output as an input file. Using this operator, we can write the done line of the loop, as follows:

```
    done <<< "`find $path -type f -print`"

```

`${!statarray[@]}` returns the list of array indexes.

# Using loopback files

Linux filesystems normally exist on devices such as disks or memory sticks. A file can also be mounted as a filesystem. This filesystem-in-a-file can be used for testing, for customized filesystems, or even as an encrypted disk for confidential information.

# How to do it...

To create a 1 GB ext4 filesystem in a file, follow these steps:

1.  Use `dd` to create a 1 GB file:

```
        $ dd if=/dev/zero of=loobackfile.img bs=1G count=1
 1024+0 records in
 1024+0 records out
 1073741824 bytes (1.1 GB) copied, 37.3155 s, 28.8 MB/s

```

 The size of the created file exceeds 1 GB because the hard disk is a block device, and hence, storage must be allocated by integral multiples of blocks size.

2.  Format the 1 GB file to ext4 using the `mkfs` command:

```
        $ mkfs.ext4 loopbackfile.img

```

3.  Check the file type with the file command:

```
        $ file loobackfile.img
 loobackfile.img: Linux rev 1.0 ext4 filesystem data,   
        UUID=c9d56c42-   
        f8e6-4cbd-aeab-369d5056660a (extents) (large files) (huge files)

```

4.  Create a mount point and mount the loopback file with `mkdir` and mount:

```
        # mkdir /mnt/loopback
 # mount -o loop loopbackfile.img /mnt/loopback

```

The `-o loop` option is used to mount loopback filesystems.

This is a short method that attaches the loopback filesystem to a device chosen by the operating system named something similar to `/dev/loop1` or `/dev/loop2`.

5.  To specify a specific loopback device, run the following command:

```
        # losetup /dev/loop1 loopbackfile.img
 # mount /dev/loop1 /mnt/loopback

```

6.  To umount (`unmount`), use the following syntax:

```
        # umount mount_point

```

Consider this example:

```
        # umount /mnt/loopback

```

7.  We can also use the device file path as an argument to the `umount` command:

```
        # umount /dev/loop1

```

Note that the mount and umount commands should be executed as a root user, since it is a privileged command.

# How it works...

First we had to create a file to make a loopback filesystem. For this, we used `dd`, which is a generic command for copying raw data. It copies data from the file specified in the `if` parameter to the file specified in the `of` parameter. We instruct `dd` to copy data in blocks of size 1 GB and copy one such block, creating a 1 GB file. The `/dev/zero` file is a special file, which will always return 0 when you read from it.

We used the `mkfts.ext4` command to create an ext4 filesystem in the file. A filesystem is needed on any device that can be mounted. Common filesystems include ext4, ext3, and vfat.

The `mount` command attaches the loopback file to a **mountpoint** (`/mnt/loopback` in this case). A mountpoint makes it possible for users to access the files stored on a filesystem. The mountpoint must be created using the `mkdir` command before executing the `mount` command. We pass the `-o loop` option to mount to tell it that we are mounting a loopback file, not a device.

When `mount` knows it is operating on a loopback file, it sets up a device in `/dev` corresponding to the loopback file and then mounts it. If we wish to do it manually, we use the `losetup` command to create the device and then the `mount` command to mount it.

# There's more...

Let's explore some more possibilities with loopback files and mounting.

# Creating partitions inside loopback images

Suppose we want to create a loopback file, partition it, and finally mount a sub-partition. In this case, we cannot use `mount -o loop`. We must manually set up the device and mount the partitions in it.

To partition a file of zeros:

```
    # losetup /dev/loop1 loopback.img
 # fdisk /dev/loop1

```

`fdisk` is a standard partitioning tool on Linux systems. A very concise tutorial on creating partitions using `fdisk` is available at [http://www.tldp.org/HOWTO/Partition/fdisk_partitioning.html](http://www.tldp.org/HOWTO/Partition/fdisk_partitioning.html) (make sure to use `/dev/loop1` instead of `/dev/hdb` in this tutorial).

Create partitions in `loopback.img` and mount the first partition:

```
    # losetup -o 32256 /dev/loop2 loopback.img

```

Here, `/dev/loop2` represents the first partition,`-o` is the offset flag, and `32256` bytes are for a DOS partition scheme. The first partition starts 32256 bytes from the start of the hard disk.

We can set up the second partition by specifying the required offset. After mounting, we can perform all regular operations as we can on physical devices.

# Mounting loopback disk images with partitions more quickly

We can manually pass partition offsets to `losetup` to mount partitions inside a loopback disk image. However, there is a quicker way to mount all the partitions inside such an image using `kpartx`. This utility is usually not installed, so you will have to install it using your package manager:

```
    # kpartx -v -a diskimage.img 
 add map loop0p1 (252:0): 0 114688 linear /dev/loop0 8192
 add map loop0p2 (252:1): 0 15628288 linear /dev/loop0 122880

```

This creates mappings from the partitions in the disk image to devices in `/dev/mapper`, which you can then mount. For example, to mount the first partition, use the following command:

```
    # mount /dev/mapper/loop0p1 /mnt/disk1

```

When you're done with the devices (and unmounting any mounted partitions using `umount`), remove the mappings by running the following command:

```
    # kpartx -d diskimage.img 
 loop deleted : /dev/loop0

```

# Mounting ISO files as loopback

An ISO file is an archive of an optical media. We can mount ISO files in the same way that we mount physical disks using loopback mounting.

We can even use a nonempty directory as the mount path. Then, the mount path will contain data from the devices rather than the original contents, until the device is unmounted. Consider this example:

```
    # mkdir /mnt/iso
 # mount -o loop linux.iso /mnt/iso

```

Now, perform operations using files from `/mnt/iso`. ISO is a read-only filesystem.

# Flush changing immediately with sync

Changes on a mounted device are not immediately written to the physical device. They are only written when the internal memory buffer is full. We can force writing with the `sync` command:

```
    $ sync

```

# Creating ISO files and hybrid ISO

An ISO image is an archive format that stores the exact image of an optical disk such as CD-ROM, DVD-ROM, and so on. ISO files are commonly used to store content to be burned to optical media.

This section will describe how to extract data from an optical disk into an ISO file that can be mounted as a loopback device, and then explain ways to generate your own ISO file systems that can be burned to an optical media.

We need to distinguish between bootable and non-bootable optical disks. Bootable disks are capable of booting from themselves and also running an operating system or another product. Bootable DVDs include installation kits and *Live* systems such as Knoppix and Puppy.

Non-bootable ISOs cannot do that. Upgrade kits, source code DVDs, and so on are non-bootable.

Note that copying files from a bootable CD-ROM to another CD-ROM is not sufficient to make the new one bootable. To preserve the bootable nature of a CD-ROM, it must be copied as a disk image using an ISO file.

Many people use flash drives as a replacement for optical disks. When we write a bootable ISO to a flash drive, it will not be bootable unless we use a special hybrid ISO image designed specifically for the purpose.

These recipes will give you an insight into ISO images and manipulations.

# Getting ready

As mentioned previously, Unix handles everything as files. Every device is a file. Hence, if we want to copy an exact image of a device, we need to read all data from it and write to a file. An optical media reader will be in the `/dev` folder with a name such as `/dev/cdrom`, `/dev/dvd`, or perhaps `/dev/sd0`. Be careful when accessing an `sd*.` Multiple disk-type devices are named `sd#`. Your hard drive may be `sd0` and the CD-ROM `sd1`, for instance.

The `cat` command will read any data, and redirection will write that data to a file. This works, but we'll also see better ways to do it.

# How to do it...

In order to create an ISO image from `/dev/cdrom`, use the following command:

```
    # cat /dev/cdrom > image.iso

```

Though this will work, the preferred way to create an ISO image is with `dd`:

```
    # dd if=/dev/cdrom of=image.iso

```

The `mkisofs` command creates an ISO image in a file. The output file created by `mkisofs` can be written to CD-ROM or DVD-ROM with utilities such as `cdrecord`. The `mkisofs` command will create an ISO file from a directory containing all the files to be copied to the ISO file:

```
    $ mkisofs -V "Label" -o image.iso source_dir/ 

```

The `-o` option in the `mkisofs` command specifies the ISO file path. The `source_dir` is the path of the directory to be used as content for the ISO file and the `-V` option specifies the label to use for the ISO file.

# There's more...

Let's learn more commands and techniques related to ISO files.

# Hybrid ISO that boots off a flash drive or hard disk

Bootable ISO files cannot usually be transferred to a USB storage device to create a bootable USB stick. However, special types of ISO files called hybrid ISOs can be flashed to create a bootable device.

We can convert standard ISO files into hybrid ISOs with the `isohybrid` command. The `isohybrid` command is a new utility and most Linux distros don't include this by default. You can download the [syslinux package](http://syslinux.zytor.com/) from [http://www.syslinux.org](http://www.syslinux.org). The command may also be available in your yum or `apt-get` repository as `syslinux-utils`.

This command will make an ISO file bootable:

```
    # isohybrid image.iso

```

The ISO file can now be written to USB storage devices.

To write the ISO to a USB storage device, use the following command:

```
    # dd if=image.iso of=/dev/sdb1 

```

Use the appropriate device instead of `/dev/sdb1`, or you can use `cat`, as follows:

```
    # cat image.iso >> /dev/sdb1

```

# Burning an ISO from the command line

The `cdrecord` command burns an ISO file to a CD-ROM or DVD-ROM.

To burn the image to the CD-ROM, run the following command:

```
    # cdrecord -v dev=/dev/cdrom image.iso

```

Useful options include the following:

*   Specify the burning speed with the `-speed` option:

```
 -speed SPEED 

```

Consider this example:

```
      # cdrecord -v dev=/dev/cdrom image.iso -speed 8

```

Here, `8` is the speed specified as 8x.

*   A CD-ROM can be burned in multi-sessions such that we can burn data multiple times on a disk. Multisession burning can be done with the `-multi` option:

```
      # cdrecord -v dev=/dev/cdrom image.iso -multi

```

# Playing with the CD-ROM tray

If you are on a desktop computer, try the following commands and have fun:

```
    $ eject

```

This command will eject the tray.

```
    $ eject -t

```

This command will close the tray.

For extra points, write a loop that opens and closes the tray a number of times. It goes without saying that one would never slip this into a co-workers `.bashrc` while they are out getting a coffee.

# Finding the difference between files, and patching

When multiple versions of a file are available, it is useful to highlight the differences between files rather than comparing them manually. This recipe illustrates how to generate differences between files. When working with multiple developers, changes need to be distributed to the others. Sending the entire source code to other developers is time consuming. Sending a difference file instead is helpful, as it consists of only lines which are changed, added, or removed, and line numbers are attached with it. This difference file is called a **patch file**. We can add the changes specified in the patch file to the original source code with the `patch` command. We can revert the changes by patching again.

# How to do it...

The `diff` utility reports the differences between two files.

1.  To demonstrate diff behavior, create the following files:

File 1: `version1.txt`

```
         this is the original text 
         line2 
         line3 
         line4 
         happy hacking ! 

```

File 2: `version2.txt`

```
       this is the original text  
       line2 
       line4 
       happy hacking !  
       GNU is not UNIX 

```

2.  Nonunified `diff` output (without the `-u` flag) is:

```
        $ diff version1.txt version2.txt 
 3d2
 <line3
 6c5
 > GNU is not UNIX

```

3.  The unified `diff` output is:

```
        $ diff -u version1.txt version2.txt
 --- version1.txt  2010-06-27 10:26:54.384884455 +0530 
 +++ version2.txt  2010-06-27 10:27:28.782140889 +0530 
 @@ -1,5 +1,5 @@ 
 this is the original text 
 line2
 -line3
 line4 
 happy hacking ! 
 -
 +GNU is not UNIX

```

 The `-u` option produces a unified output. Unified diff output is more readable and is easier to interpret.

In unified `diff`, the lines starting with `+` are the added lines and the lines starting with `-` are the removed lines.

4.  A patch file can be generated by redirecting the `diff` output to a file:

```
        $ diff -u version1.txt version2.txt > version.patch

```

The `patch` command can apply changes to either of the two files. When applied to `version1.txt`, we get the `version2.txt` file. When applied to `version2.txt`, we generate `version1.txt`.

5.  This command applies the patch:

```
        $ patch -p1 version1.txt < version.patch
 patching file version1.txt

```

We now have `version1.txt` with the same contents as `version2.txt`.

6.  To revert the changes, use the following command:

```
        $ patch -p1 version1.txt < version.patch 
 patching file version1.txt
 Reversed (or previously applied) patch detected!  Assume -R? [n] y
 #Changes are reverted.

```

As shown, patching an already patched file reverts the changes. To avoid prompting the user with `y/n`, we can use the `-R` option along with the `patch` command.

# There's more...

Let's go through additional features available with `diff`.

# Generating difference against directories

The `diff` command can act recursively against directories. It will generate a difference output for all the descendant files in the directories. Use the following command:

```
    $ diff -Naur directory1 directory2

```

The interpretation of each of the options in this command is as follows:

*   `-N`: This is used for treating missing files as empty
*   `-a`: This is used to consider all files as text files
*   `-u`: This is used to produce unified output
*   `-r`: This is used to recursively traverse through the files in the directories

# Using head and tail for printing the last or first 10 lines

When examining a large file, thousands of lines long, the `cat` command, which will display all the line,s is not suitable. Instead, we want to view a subset (for example, the first 10 lines of the file or the last 10 lines of the file). We may need to print the first *n* lines or last *n* lines or print all except the last *n* lines or all except the first *n* lines, or the lines between two locations.

The `head` and `tail` commands can do this.

# How to do it...

The `head` command reads the beginning of the input file.

1.  Print the first 10 lines:

```
        $ head file

```

2.  Read the data from `stdin`:

```
        $ cat text | head

```

3.  Specify the number of first lines to be printed:

```
        $ head -n 4 file

```

This command prints the first four lines.

4.  Print all lines excluding the last `M` lines:

```
        $ head -n -M file 

```

Note that it is negative M.

 For example, to print all the lines except the last five lines, use the following command line:

```
        $ seq 11 | head -n -5
 1
 2
 3
 4
 5
 6

```

This command prints lines 1 to 5:

```
      $ seq 100 | head -n 5

```

5.  Printing everything except the last lines is a common use for `head`. When examining log files we most often want to view the most recent (that is, the last) lines.
6.  To print the last 10 lines of a file, use this command:

```
      $ tail file

```

7.  To read from `stdin`, use the following command:

```
      $ cat text | tail

```

8.  Print the last five lines:

```
      $ tail -n 5 file

```

9.  To print all lines excluding the first M lines, use this command:

```
      $ tail -n +(M+1)

```

For example, to print all lines except the first five lines, *M + 1 = 6*, the command is as follows:

```
      $ seq 100 | tail -n +6 

```

This will print from 6 to 100.

One common use for `tail` is to monitor new lines in a growing file, for instance, a system log file. Since new lines are appended to the end of the file, `tail` can be used to display them as they are written. To monitor the growth of the file, `tail` has a special option `-f` or `--follow`, which enables `tail` to follow the appended lines and display them as data is added:

```
    $ tail -f growing_file

```

You will probably want to use this on logfiles. The command to monitor the growth of the files would be this:

```
    # tail -f /var/log/messages

```

Alternatively, this command can be used:

```
    $ dmesg | tail -f

```

The `dmesg` command returns contents of the kernel ring buffer messages. We can use this to debug USB devices, examine disk behavior, or monitor network connectivity. The `-f` tail can add a sleep interval `-s` to set the interval during which the file updates are monitored.

The `tail` command can be instructed to terminate after a given process ID dies.

Suppose a process `Foo` is appending data to a file that we are monitoring. The `-f` tail should be executed until the process `Foo` dies.

```
    $ PID=$(pidof Foo)
 $ tail -f file --pid $PID

```

When the process `Foo` terminates, `tail` also terminates.

Let's work on an example.

1.  Create a new file `file.txt` and open the file in your favorite text editor.
2.  Now run the following commands:

```
        $ PID=$(pidof gedit)
 $ tail -f file.txt --pid $PID

```

3.  Add new lines to the file and make frequent file saves.

When you add new lines to the end of the file, the new lines will be written to the terminal by the `tail` command. When you close the edit session, the `tail` command will terminate.

# Listing only directories - alternative methods

Listing only directories via scripting is deceptively difficult. This recipe introduces multiple ways of listing only directories.

# Getting ready

g ready There are multiple ways of listing directories only. The `dir` command is similar to `ls`, but with fewer options. We can also list directories with `ls` and `find`.

# How to do it...

Directories in the current path can be displayed in the following ways:

1.  Use `ls` with `-d` to print directories:

```
        $ ls -d */

```

2.  Use `ls -F` with `grep`:

```
        $ ls -F | grep "/$"

```

3.  Use `ls -l` with `grep`:

```
        $ ls -l | grep "^d"

```

4.  Use `find` to print directories:

```
         $ find . -type d -maxdepth 1 -print

```

# How it works...

When the `-F` parameter is used with `ls`, all entries are appended with some type of file character such as `@`, `*`, `|`, and so on. For directories, entries are appended with the `/` character. We use `grep` to filter only entries ending with the `/$` end-of-line indicator.

The first character of any line in the `ls -l` output is the type of file character. For a directory, the type of file character is `d`. Hence, we use `grep` to filter lines starting with `"d`.`"``^` is a start-of-line indicator.

The `find` command can take the parameter type as directory and `maxdepth` is set to `1` since we don't want it to search inside the subdirectories.

# Fast command-line navigation using pushd and popd

When navigating around multiple locations in the filesystem, a common practice is to cd to paths you copy and paste. This is not efficient if we are dealing with several locations. When we need to navigate back and forth between locations, it is time consuming to type or paste the path with each `cd` command. Bash and other shells support `pushd` and `popd` to cycle between directories.

# Getting ready

`pushd` and `popd` are used to switch between multiple directories without retyping directory paths. `pushd` and `popd` create a stack of paths-a **Last****In****First****Out** (**LIFO**) list of the directories we've visited.

# How to do it...

The `pushd` and `popd` commands replace cd for changing your working directory.

1.  To push and change a directory to a path, use this command:

```
        ~ $ pushd /var/www

```

Now the stack contains `/var/www ~` and the current directory is changed to `/var/www`.

2.  Now, push the next directory path:

```
        /var/www $ pushd /usr/src

```

 Now the stack contains `/usr/src``/var/www ~` and the current directory is `/usr/src`.

You can push as many directory paths as needed.

3.  View the stack contents:

```
        $ dirs
 /usr/src /var/www ~ /usr/share /etc
 0           1              2  3              4 

```

4.  Now when you want to switch to any path in the list, number each path from `0` to `n`, then use the path number for which we need to switch. Consider this example:

```
        $ pushd +3

```

 Now it will rotate the stack and switch to the `/usr/share` directory.

 `pushd` will always add paths to the stack. To remove paths from the stack, use `popd`.

5.  Remove a last pushed path and change to the next directory:

```
        $ popd

```

Suppose the stack is `/usr/src /var/www ~ /usr/share /etc`, and the current directory is `/usr/src`. The `popd` command will change the stack to `/var/www ~ /usr/share /etc` and change the current directory to `/var/www`.

6.  To remove a specific path from the list, use `popd +num`. `num` is counted as `0` to `n` from left to right.

# There's more...

Let's go through the essential directory navigation practices.

# pushd and popd are useful when there are more than three directory paths used. However, when you use only two locations, there is an alternative and easier way, that is, cd -.

The current path is `/var/www`.

```
    /var/www $  cd /usr/src
/usr/src $ # do something

```

Now, to switch back to `/var/www`, you don't have to type `/var/www`, just execute:

```
    /usr/src $ cd -

```

To switch to `/usr/src`:

```
    /var/www $ cd -

```

# Counting the number of lines, words, and characters in a file

Counting the number of lines, words, and characters in a text file is frequently useful. This book includes some tricky examples in other chapters where the counts are used to produce the required output. **Counting LOC** (**Lines of Code**) is a common application for developers. We may need to count a subset of files, for example, all source code files, but not object files. A combination of `wc` with other commands can perform that.

The `wc` utility counts lines, words, and characters. It stands for **word count**.

# How to do it...

The `wc` command supports options to count the number of lines, words, and characters:

1.  Count the number of lines:

```
      $ wc -l file

```

2.  To use `stdin` as input, use this command:

```
      $ cat file | wc -l

```

3.  Count the number of words:

```
      $ wc -w file
 $ cat file | wc -w

```

4.  Count the number of characters:

```
      $ wc -c file
 $ cat file | wc -c

```

 To count the characters in a text string, use this command:

```
      echo -n 1234 | wc -c
 4

```

 Here, `-n` deletes the final newline character.

5.  To print the number of lines, words, and characters, execute `wc` without any options:

```
      $ wc file
 1435   15763  112200

```

Those are the number of lines, words, and characters.

6.  Print the length of the longest line in a file with the `-L` option:

```
      $ wc file -L
 205

```

# Printing the directory tree

Graphically representing directories and filesystems as a tree hierarchy makes them easier to visualize. This representation is used by monitoring scripts to present the filesystem in an easy-to-read format.

# Getting ready

The `tree` command prints graphical trees of files and directories. The `tree` command does not come with preinstalled Linux distributions. You must install it using the package manager.

# How to do it...

The following is a sample Unix filesystem tree to show an example:

```
    $ tree ~/unixfs
 unixfs/
 |-- bin
 |   |-- cat
 |   `-- ls
 |-- etc
 |   `-- passwd
 |-- home
 |   |-- pactpub
 |   |   |-- automate.sh
 |   |   `-- schedule
 |   `-- slynux
 |-- opt
 |-- tmp
 `-- usr
 8 directories, 5 files

```

The `tree` command supports several options:

*   To display only files that match a pattern, use the `-P` option:

```
      $ tree path -P PATTERN # Pattern should be wildcard in single    
      quotes

```

Consider this example:

```
      $ tree PATH -P '*.sh' # Replace PATH with a directory path
 |-- home
 |   |-- packtpub
 |   |   `-- automate.sh

```

*   To display only files that do not match a pattern, use the `-I` option:

```
      $ tree path -I PATTERN

```

*   To print the size along with files and directories, use the `-h` option:

```
      $ tree -h

```

# There's more...

The tree command can generate output in HTML as well as to a terminal.

# HTML output for tree

This command creates an HTML file with the tree output:

```
    $ tree PATH -H http://localhost -o out.html

```

Replace `http://localhost` with the URL where you are planning to host the file. Replace `PATH` with a real path for the base directory. For the current directory, use `.` as `PATH`.

The web page generated from the directory listing will look as follows:

![](img/Ch03_img.jpg)

# Manipulating video and image files

Linux and Unix support many applications and tools for working with images and video files. Most Linux distributions include the **imageMagick** suite with the **convert** application for manipulating images. The full-function video editing applications such as **kdenlive** and **openshot** are built on top of the **ffmpeg** and **mencoder** command line applications.

The convert application has hundreds of options. We'll just use the one that extracts a portion of an image.

`ffmpeg` and `mencoder` have enough options and features to fill a book all by themselves. We'll just look at a couple simple uses.

This section has some recipes for manipulating still images and videos.

# Getting ready

Most Linux distributions include the **ImageMagick** tools. If your system does not include them, or if they are out of date, there are instructions for downloading and installing the latest tools on the ImageMagick website at [www.imagemagick.org](http://www.imagemagick.org/).

Like ImageMagick, many Linux distributions already include the `ffmpeg` and `mencoder` toolsets. The latest releases can be found at the `ffmpeg` and `mencoder` websites at [http://www.ffmpeg.org](http://www.ffmpeg.org/) and [http://www.mplayerhq.hu](http://www.mplayerhq.hu/).

Building and installing the video tools will probably require loading codecs and other ancillary files with confusing version dependencies. If you intend to use your Linux system for audio and video editing, it's simplest to use a Linux distribution that's designed for this, such as the Ubuntu Studio distributions.

Here are some recipes for a couple of common audio-video conversions:

# Extracting Audio from a movie file (mp4)

Music videos are fun to watch, but the point of music is to listen to it. Extracting the audio portion from a video is simple:

# How to do it...

The following command accepts an `mp4` video file (`FILE.mp4`) and extracts the audio portion into a new file (`OUTPUTFILE.mp3`) as an `mp3`:

```
ffmpeg -i FILE.mp4 -acodec libmp3lame OUTPUTFILE.mp3

```

# Making a video from a set of still images

Many cameras support taking pictures at intervals. You can use this feature to do your own time-lapse photography or create stop-action videos. There are examples of this on [www.cwflynt.com](http://www.cwflynt.com/). You can convert a set of still images into a video with the OpenShot video editing package or from a command line using the mencoder tool.

# How to do it...

This script will accept a list of images and will create an MPEG video file from it:

```
$ cat stills2mpg.sh
echo $* | tr ' ' '\n' >files.txt mencoder mf://@files.txt -mf fps=24 -ovc lavc \ -lavcopts vcodec=msmpeg4v2 -noskip -o movie.mpg

```

To use this script, copy/paste the commands into a file named `stills2mpg.sh`, make it executable and invoke it as follows:

```
./stills2mpg.sh file1.jpg file2.jpg file3.jpg ...

```

Alternatively, use this to invoke it:

```
./stills2mpg.sh *.jpg

```

# How it works...

The `mencoder` command requires that the input file be formatted as one image file per line. The first line of the script echoes the command line arguments to the tr command to convert the space delimiters to newlines. This transforms the single-line list into a list of files arranged one per line.

You can change the speed of the video by resetting the **FPS** (**frames-per-second**) parameter. For example, setting the fps value to `1` will make a slide show that changes images every second.

# Creating a panned video from a still camera shot

If you decide to create your own video, you'll probably want a panned shot of some landscape at some point. You can record a video image with most cameras, but if you only have a still image you can still make a panned video.

# How to do it...

Cameras commonly take a larger image than will fit on a video. You can create a motion-picture pan using the convert application to extract sections of a large image, and stitch them together into a video file with `mencoder`:

```
$> makePan.sh
# Invoke as: 
# sh makePan.sh OriginalImage.jpg prefix width height xoffset yoffset # Clean out any old data
rm -f tmpFiles
# Create 200 still images, stepping through the original xoffset and yoffset
# pixels at a time
for o in `seq 1 200`
 do
 x=$[ $o+$5 ]
 convert -extract $3x$4+$x+$6 $1 $2_$x.jpg
 echo $2_$x.jpg >> tmpFiles
done
#Stitch together the image files into a mpg video file
mencoder mf://@tmpFiles -mf fps=30 -ovc lavc -lavcopts \  
        vcodec=msmpeg4v2 -noskip -o $2.mpg

```

# How it works...

This script is more complex than the ones we've looked at so far. It uses seven command-line arguments to define the input image, a prefix to use for the output files, the width and height for the intermediate images, and the starting offset into the original image.

Within the `for` loop, it creates a set of image files and stores the names in a file named `tmpFiles`. Finally, the script uses `mencoder` to merge the extracted image files into an MPEG video that can be imported into a video editor such as kdenlive or OpenShot.