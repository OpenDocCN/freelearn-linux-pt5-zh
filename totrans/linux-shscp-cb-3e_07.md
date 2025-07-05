# The Backup Plan

In this chapter, we will cover the following recipes:

*   Archiving with `tar`
*   Archiving with `cpio`
*   Compressing data with `gzip`
*   Archiving and compressing with `zip`
*   Faster archiving with `pbzip2`
*   Creating filesystems with compression
*   Backing up snapshots with `rsync`
*   Differential archives
*   Creating entire disk images using `fsarchiver`

# Introduction

Nobody cares about backups until they need them, and nobody makes backups unless forced. Therefore, making backups needs to be automated. With advances in disk drive technology, it's simplest to add a new drive or use the cloud for backups, rather than backing up to a tape drive. Even with cheap drives or cloud storage, backup data should be compressed to reduce storage needs and transfer time. Data should be encrypted before it's stored on the cloud. Data is usually archived and compressed before encrypting. Many standard encryption programs can be automated with shell scripts. This chapter's recipes describe creating and maintaining files or folder archives, compression formats, and encrypting techniques.

# Archiving with tar

The `tar` command was written to archive files. It was originally designed to store data on tape, thus the name, **Tape ARchive**. Tar allows you to combine multiple files and directories into a single file while retaining the file attributes, such as owner and permissions. The file created by the `tar` command is often referred to as a **tarball**. These recipes describe creating archives with `tar`.

# Getting ready

The `tar` command comes by default with all Unix-like operating systems. It has a simple syntax and creates archives in a portable file format. It supports many arguments to fine-tune its behavior.

# How to do it...

The `tar` command creates, updates, examines, and unpacks archives.

1.  To create an archive file with tar:

```
        $ tar -cf output.tar [SOURCES]  

```

The `c` option creates a new archive and the `f` option tells tar the name of a file to use for the archive. The f option must be followed by a filename:

```
        $ tar -cf archive.tar file1 file2 file3 folder1 ..  

```

2.  The `-t` option lists the contents of an archive:

```
        $ tar -tf archive.tar
 file1
 file2  

```

3.  The `-v` or `-vv` flag includes more information in the output. These features are called verbose (`v`) and very-verbose (`vv`). The `-v` convention is common for commands that generate reports by printing to the terminal. The `-v` option displays more details, such as file permissions, owner group, and modification date:

```
        $ tar -tvf archive.tar 
 -rw-rw-r-- shaan/shaan       0 2013-04-08 21:34 file1
 -rw-rw-r-- shaan/shaan       0 2013-04-08 21:34 file2  

```

The filename must appear immediately after the `-f` and it should be the last option in the argument group. For example, if you want verbose output, you should use the options like this:
`$ tar -cvf output.tar file1 file2 file3 folder1 ..`

# How it works...

The tar command accepts a list of filenames or wildcards such as `*.txt` to specify the sources. When finished, `tar` will archive the source files into the named file.

We cannot pass hundreds of files or folders as command-line arguments. So, it is safer to use the append option (explained later) if many files are to be archived.

# There's more...

Let's go through additional features supported by the `tar` command.

# Appending files to an archive

The `-r` option will append new files to the end of an existing archive:

```
    $ tar -rvf original.tar new_file

```

The next example creates an archive with one text file in it:

```
    $ echo hello >hello.txt
 $ tar -cf archive.tar hello.txt

```

The `-t` option displays the files in an archive. The `-f` option defines the archive name:

```
    $ tar -tf archive.tar
 hello.txt

```

The `-r` option appends a file:

```
    $ tar -rf archive.tar world.txt
 $ tar -tf archive.tar
 hello.txt
 world.txt

```

The archive now contains both the files.

# Extracting files and folders from an archive

The `-x` option extracts the contents of the archive to the current directory:

```
    $ tar -xf archive.tar

```

When `-x` is used, the `tar` command extracts the contents of the archive to the current directory. The `-C` option specifies a different directory to receive the extracted files:

```
    $ tar -xf archive.tar -C /path/to/extraction_directory

```

The command extracts the contents of an archive to a specified directory. It extracts the entire contents of the archive. We can extract just a few files by specifying them as command arguments:

```
    $ tar -xvf file.tar file1 file4

```

The preceding command extracts only `file1` and `file4`, and it ignores other files in the archive.

# stdin and stdout with tar

While archiving, we can specify `stdout` as the output file so another command in a pipe can read it as `stdin` and process the archive.

This technique will transfer data through a **Secure Shell** (**SSH**) connection, for example:

```
    $ tar cvf - files/ | ssh user@example.com "tar xv -C Documents/"

```

In the preceding example, the files/directory is added to a tar archive which is output to `stdout` (denoted by `-`) and extracted to the `Documents` folder on the remote system.

# Concatenating two archives

The `-A` option will merge multiple tar archives.

Given two tarballs, `file1.tar` and `file2.tar`, the following command will merge the contents of `file2.tar` into `file1.tar`:

```
    $ tar -Af file1.tar file2.tar

```

Verify it by listing the contents:

```
    $ tar -tvf file1.tar

```

# Updating files in an archive with a timestamp check

The append option appends any given file to the archive. If a file already exists inside the archive, tar will append the file, and the archive will contain duplicates. The update option `-u` specifies only appending files that are newer than existing files inside the archive.

```
    $ tar -tf archive.tar 
 filea
 fileb
 filec  

```

To append `filea` only if `filea` has been modified since the last time it was added to `archive.tar`, use the following command:

```
    $ tar -uf archive.tar filea

```

Nothing happens if the version of `filea` outside the archive and the `filea` inside `archive.tar` have the same timestamp.

Use the `touch` command to modify the file timestamp and then try the `tar` command again:

```
    $ tar -uvvf archive.tar filea
 -rw-r--r-- slynux/slynux     0 2010-08-14 17:53 filea

```

The file is appended since its timestamp is newer than the one inside the archive, as shown with the `-t` option:

```
    $ tar -tf archive.tar 
 -rw-r--r-- slynux/slynux     0 2010-08-14 17:52 filea
 -rw-r--r-- slynux/slynux     0 2010-08-14 17:52 fileb
 -rw-r--r-- slynux/slynux     0 2010-08-14 17:52 filec
 -rw-r--r-- slynux/slynux     0 2010-08-14 17:53 filea

```

Note that the new `filea` has been appended to the `tar` archive. When extracting this archive, tar will select the latest version of `filea`.

# Comparing files in the archive and filesystem

The `-d` flag compares files inside an archive with those on the filesystem. This feature can be used to determine whether or not a new archive needs to be created.

```
    $ tar -df archive.tar
 afile: Mod time differs
 afile: Size differs

```

# Deleting files from the archive

The `-delete` option removes files from an archive:

```
    $ tar -f archive.tar --delete file1 file2 ..

```

Alternatively,

```
    $ tar --delete --file archive.tar [FILE LIST]

```

The next example demonstrates deleting a file:

```
    $ tar -tf archive.tar
 filea
 fileb
 filec
 $ tar --delete --file archive.tar filea
 $ tar -tf archive.tar
 fileb
 filec

```

# Compression with the tar archive

By default, the `tar` command archives files, it does not compress them. Tar supports options to compress the resulting archive. Compression can significantly decrease the size of the files. Tarballs are often compressed into one of the following formats:

*   **gzip format**: `file.tar.gz` or `file.tgz`
*   **bzip2 format**: `file.tar.bz2`
*   **Lempel-Ziv-Markov format**: `file.tar.lzma`

Different `tar` flags are used to specify different compression formats:

*   `-j` for **bunzip2**
*   `-z` for **gzip**
*   `--lzma` for **lzma**

It is possible to use compression formats without explicitly specifying special options as earlier. `tar` can compress based on the extension of the output or decompress based on an input file's extension. The `-a` or - **auto-compress** option causes tar to select a compression algorithm automatically based on file extension:

```
    $ tar acvf archive.tar.gz filea fileb filec
 filea
 fileb
 filec
 $ tar tf archive.tar.gz 
 filea
 fileb
 filec  

```

# Excluding a set of files from archiving

The `-exclude [PATTEN]` option will exclude files matched by wildcard patterns from being archived.

For example, to exclude all `.txt` files from archiving use the following command:

```
    $ tar -cf arch.tar * --exclude "*.txt"

```

Note that the pattern should be enclosed within quotes to prevent the shell from expanding it.

It is also possible to exclude a list of files provided in a list file with the `-X` flag as follows:

```
    $ cat list
 filea
 fileb

 $ tar -cf arch.tar * -X list

```

Now it excludes `filea` and `fileb` from archiving.

# Excluding version control directories

One use for tarballs is distributing source code. Much source code is maintained using version control systems such as subversion, Git, mercurial, and CVS, (refer to the previous chapter). Code directories under version control often contain special directories such as `.svn` or `.git`. These are managed by the version control application and are not useful to anyone except a developer. Thus, they should be eliminated from the tarball of the source code being distributed to users.

In order to exclude version control related files and directories while archiving use the `--exclude-vcs` option along with `tar`. Consider this example:

```
    $ tar --exclude-vcs -czvvf source_code.tar.gz eye_of_gnome_svn

```

# Printing the total bytes

The `-totals` option will print the total bytes copied to the archive. Note that this is the number of bytes of actual data. If you include a compression option, the file size will be less than the number of bytes archived.

```
    $ tar -cf arc.tar * --exclude "*.txt" --totals
 Total bytes written: 20480 (20KiB, 12MiB/s)

```

# See also

*   The *Compressing data with gzip* recipe in this chapter explains the `gzip` command

# Archiving with cpio

The `cpio` application is another archiving format similar to `tar`. It is used to store files and directories in an archive with attributes such as permissions and ownership. The `cpio` format is used in RPM package archives (which are used in `distros` such as Fedora), `initramfs` files for the Linux kernel that contain the kernel image, and so on. This recipe will give simple examples of `cpio`.

# How to do it...

The `cpio` application accepts input filenames via `stdin` and it writes the archive to `stdout`. We have to redirect `stdout` to a file to save the `cpio` output:

1.  Create test files:

```
        $ touch file1 file2 file3

```

2.  Archive the test files:

```
        $ ls file* | cpio -ov > archive.cpio

```

3.  List files in a `cpio` archive:

```
        $ cpio -it < archive.cpio

```

4.  Extract files from the `cpio` archive:

```
        $ cpio -id < archive.cpio

```

# How it works...

For the archiving command, the options are as follows:

*   `-o`: This specifies the output
*   `-v`: This is used for printing a list of files archived

Using `cpio`, we can also archive using files as absolute paths. `/usr/``somedir` is an absolute path as it contains the full path starting from root (`/`).
A relative path will not start with `/` but it starts the path from the current directory. For example, `test/file` means that there is a directory named `test` and `file` is inside the `test` directory.
While extracting, `cpio` extracts to the absolute path itself. However, in the case of `tar`, it removes the `/` in the absolute path and converts it to a relative path.

The options in the command for listing all the files in the given `cpio` archive are as follows:

*   `-i` is for specifying the input
*   `-t` is for listing

In the command for extraction, `-o` stands for extracting and `cpio` overwrites files without prompting. The `-d` option tells `cpio` to create new directories as needed.

# Compressing data with gzip

The **gzip** application is a common compression format in the GNU/Linux platform. The `gzip`, `gunzip`, and `zcat` programs all handle `gzip` compression. These utilities only compress/decompress a single file or data stream. They cannot archive directories and multiple files directly. Fortunately, `gzip` can be used with both tar and `cpio`.

# How to do it...

`gzip` will compress a file and `gunzip` will decompress it back to the original:

1.  Compress a file with `gzip`:

```
        $ gzip filename
 $ ls
 filename.gz

```

2.  Extract a `gzip` compressed file:

```
        $ gunzip filename.gz
 $ ls
 filename

```

3.  In order to list the properties of a compressed file, use the following command:

```
        $ gzip -l test.txt.gz
 compressed        uncompressed  ratio uncompressed_name
 35                   6 -33.3% test.txt

```

4.  The `gzip` command can read a file from `stdin` and write a compressed file to `stdout`.

Read data from `stdin` and output the compressed data to `stdout`:

```
         $ cat file | gzip -c > file.gz

```

The `-c` option is used to specify output to `stdout`.

The gzip `-c` option works well with `cpio`:

```
        $ ls * | cpio -o | gzip -c > cpiooutput.gz
 $ zcat cpiooutput.gz | cpio -it

```

5.  We can specify the compression level for `gzip` using `--fast` or the `--best` option to provide low and high compression ratios, respectively.

# There's more...

The `gzip` command is often used with other commands and has advanced options to specify the compression ratio.

# Gzip with tarball

A gzipped tarball is a tar archive compressed using gzip. We can use two methods to create such tarballs:

*   The first method is as follows:

```
        $ tar -czvvf archive.tar.gz [FILES]

```

Alternatively, this command can be used:

```
        $ tar -cavvf archive.tar.gz [FILES]

```

The `-z` option specifies `gzip` compression and the `-a` option specifies that the compression format should be determined from the extension.

*   The second method is as follows:

First, create a tarball:

```
        $ tar -cvvf archive.tar [FILES]

```

Then, compress the tarball:

```
        $ gzip archive.tar

```

If many files (a few hundred) are to be archived in a tarball and need to be compressed, we use the second method with a few changes. The problem with defining many files on the command line is that it can accept only a limited number of files as arguments. To solve this problem, we create a `tar` file by adding files one by one in a loop with the append option (`-r`), as follows:

```
FILE_LIST="file1  file2 file3  file4  file5" 
for f in $FILE_LIST;
  do
  tar -rvf archive.tar $f
done
gzip archive.tar

```

The following command will extract a gzipped tarball:

```
    $ tar -xavvf archive.tar.gz -C extract_directory

```

In the preceding command, the `-a` option is used to detect the compression format.

# zcat - reading gzipped files without extracting

The `zcat` command dumps uncompressed data from a `.gz` file to `stdout` without recreating the original file. The `.gz` file remains intact.

```
    $ ls
 test.gz

 $ zcat test.gz
 A test file
 # file test contains a line "A test file"

 $ ls
 test.gz

```

# Compression ratio

We can specify the compression ratio, which is available in the range 1 to 9, where:

*   1 is the lowest, but fastest
*   9 is the best, but slowest

You can specify any ratio in that range as follows:

```
      $ gzip -5 test.img

```

By default, `gzip` uses a value of `-6`, favoring a better compression at the cost of some speed.

# Using bzip2

`bzip2` is similar to `gzip` in function and syntax. The difference is that `bzip2` offers better compression and runs more slowly than `gzip`.

To compress a file using `bzip2` use the command as follows:

```
      $ bzip2 filename

```

Extract a bzipped file as follows:

```
      $ bunzip2 filename.bz2

```

The way to compress to and extract from tar.bz2 files is similar to tar.gz, discussed earlier:

```
      $ tar -xjvf archive.tar.bz2

```

Here `-j` specifies compressing the archive in the `bzip2` format.

# Using lzma

The `lzma` compression delivers better compression ratios than `gzip` and `bzip2`.

To compress a file using `lzma` use the command as follows:

```
    $ lzma filename

```

To extract a `lzma` file, use the following command:

```
    $ unlzma filename.lzma

```

A tarball can be compressed with the `-lzma` option:

```
    $ tar -cvvf --lzma archive.tar.lzma [FILES]

```

Alternatively, this can be used:

```
    $ tar -cavvf archive.tar.lzma [FILES]

```

To extract a tarball created with `lzma` compression to a specified directory, use this command:

```
    $ tar -xvvf --lzma archive.tar.lzma -C extract_directory

```

In the preceding command, `-x` is used for extraction. `--lzma` specifies the use of `lzma` to decompress the resulting file.

Alternatively, use this:

```
    $ tar -xavvf archive.tar.lzma -C extract_directory

```

# See also

*   The *Archiving with tar* recipe in this chapter explains the `tar` command

# Archiving and compressing with zip

ZIP is a popular compressed archive format available on Linux, Mac, and Windows. It isn't as commonly used as `gzip` or `bzip2` on Linux but is useful when distributing data to other platforms.

# How to do it...

1.  The following syntax creates a zip archive:

```
        $ zip archive_name.zip file1 file2 file3...

```

Consider this example:

```
        $ zip file.zip file

```

Here, the `file.zip` file will be produced.

2.  The `-r` flag will archive folders recursively:

```
        $ zip -r archive.zip folder1 folder2

```

3.  The `unzip` command will extract files and folders from a ZIP file:

```
        $ unzip file.zip

```

The unzip command extracts the contents without removing the archive (unlike `unlzma` or `gunzip`).

4.  The `-u` flag updates files in the archive with newer files:

```
 $ zip file.zip -u newfile 

```

5.  The `-d` flag deletes one or more files from a zipped archive:

```
 $ zip -d arc.zip file.txt 

```

6.  The `-l` flag to unzip lists the files in an archive:

```
 $ unzip -l archive.zip 

```

# How it works...

While being similar to most of the archiving and compression tools we have already discussed, `zip`, unlike `lzma`, `gzip`, or `bzip2`, won't remove the source file after archiving. While `zip` is similar to `tar`, it performs both archiving and compression, while `tar` by itself does not perform compression.

# Faster archiving with pbzip2

Most modern computers have at least two CPU cores. This is almost the same as two real CPUs doing your work. However, just having a multicore CPU doesn't mean a program will run faster; it is important that the program is designed to take advantage of the multiple cores.

The compression commands covered so far use only one CPU. The `pbzip2`, `plzip`, `pigz`, and `lrzip` commands are multithreaded and can use multiple cores, hence, decreasing the overall time taken to compress your files.

None of these are installed with most distros, but can be added to your system with apt-get or yum.

# Getting ready

`pbzip2` usually doesn't come preinstalled with most distros, you will have to use your package manager to install it:

```
    sudo apt-get install pbzip2

```

# How to do it...

1.  The `pbzip2` command will compress a single file:

```
        pbzip2 myfile.tar

```

`pbzip2` detects the number of cores on your system and compresses `myfile.tar`, to `myfile.tar.bz2`.

2.  To compress and archive multiple files or directories, we use `pbzip2` in combination with `tar`, as follows:

```
 tar cf sav.tar.bz2 --use-compress-prog=pbzip2 dir

```

Alternatively, this can be used:

```
        tar -c directory_to_compress/ | pbzip2 -c > myfile.tar.bz2

```

3.  Extracting a `pbzip2` compressed file is as follows:

The `-d` flag will decompress a file:

```
        pbzip2 -d myfile.tar.bz2

```

A tar archive can be decompressed and extracted using a pipe:

```
        pbzip2 -dc myfile.tar.bz2 | tar x

```

# How it works...

The `pbzip2` application uses the same compression algorithms as `bzip2`, but it compresses separate chunks of data simultaneously using `pthreads`, a threading library. The threading is transparent to the user, but provides much faster compression.

Like `gzip` or `bzip2`, `pbzip2` does not create archives. It only works on a single file. To compress multiple files and directories, we use it in conjunction with `tar` or `cpio`.

# There's more...

There are other useful options we can use with `pbzip2`:

# Manually specifying the number of CPUs

The `-p` option specifies the number of CPU cores to use. This is useful if automatic detection fails or you need cores free for other jobs:

```
    pbzip2 -p4 myfile.tar

```

This will tell `pbzip2` to use 4 CPUs.

# Specifying the compression ratio

The options from `-1` to `-9` specify the fastest and best compression ratios with **1** being the fastest and **9** being the best compression

# Creating filesystems with compression

The `squashfs` program creates a read-only, heavily compressed filesystem. The `squashfs` program can compress 2 to 3 GB of data into a 700 MB file. The Linux LiveCD (or LiveUSB) distributions are built using `squashfs`. These CDs make use of a read-only compressed filesystem, which keeps the root filesystem on a compressed file. The compressed file can be loopback-mounted to load a complete Linux environment. When files are required, they are decompressed and loaded into the RAM, run, and the RAM is freed.

The `squashfs` program is useful when you need a compressed archive and random access to the files. Completely decompressing a large compressed archive takes a long time. A loopback-mounted archive provides fast file access since only the requested portion of the archive is decompressed.

# Getting ready

Mounting a `squashfs` filesystem is supported by all modern Linux distributions. However, creating `squashfs` files requires `squashfs-tools`, which can be installed using the package manager:

```
    $ sudo apt-get install squashfs-tools

```

Alternatively, this can be used:

```
    $ yum install squashfs-tools

```

# How to do it...

1.  Create a `squashfs` file by adding source directories and files with the `mksquashfs` command:

```
        $ mksquashfs SOURCES compressedfs.squashfs

```

Sources can be wildcards, files, or folder paths.

Consider this example:

```
        $ sudo mksquashfs /etc test.squashfs
 Parallel mksquashfs: Using 2 processors
 Creating 4.0 filesystem on test.squashfs, block size 131072.
 [=======================================] 1867/1867 100%  

```

More details will be printed on the terminal. The output is stripped to save space.

2.  To mount the `squashfs` file to a mount point, use loopback mounting, as follows:

```
        # mkdir /mnt/squash
 # mount -o loop compressedfs.squashfs /mnt/squash

```

You can access the contents at `/mnt/squashfs`.

# There's more...

The `squashfs` filesystem can be customized by specifying additional parameters.

# Excluding files while creating a squashfs file

The `-e` flag will exclude files and folders:

```
    $ sudo mksquashfs /etc test.squashfs -e /etc/passwd /etc/shadow

```

The `-e` option excludes `/etc/``passwd and` `/etc/``shadow` files from the `squashfs` filesystem.

The `-ef` option reads a file with a list of files to exclude:

```
    $ cat excludelist
 /etc/passwd
 /etc/shadow

 $ sudo mksquashfs /etc test.squashfs -ef excludelist

```

If we want to support wildcards in excludes lists, use `-wildcard` as an argument.

# Backing up snapshots with rsync

Backing up data is something that needs to be done regularly. In addition to local backups, we may need to back up data to or from remote locations. The `rsync` command synchronizes files and directories from one location to another while minimizing transfer time. The advantages of `rsync` over the `cp` command are that `rsync` compares modification dates and will only copy the files that are newer, `rsync` supports data transfer across remote machines, and `rsync` supports compression and encryption.

# How to do it...

1.  To copy a source directory to a destination, use the following command:

```
        $ rsync -av source_path destination_path

```

Consider this example:

```
        $ rsync -av /home/slynux/data 
        slynux@192.168.0.6:/home/backups/data

```

In the preceding command:

*   `-a` stands for archiving
*   `-v` (verbose) prints the details or progress on stdout

The preceding command will recursively copy all the files from the source path to the destination path. The source and destination paths can be either remote or local.

2.  To backup data to a remote server or host, use the following command:

```
        $ rsync -av source_dir username@host:PATH

```

To keep a mirror at the destination, run the same `rsync` command at regular intervals. It will copy only changed files to the destination.

3.  To restore the data from the remote host to `localhost`, use the following command:

```
        $ rsync -av username@host:PATH destination  

```

The `rsync` command uses SSH to connect to the remote machine hence, you should provide the remote machine's address in the `user@host` format, where user is the username and host is the IP address or host name attached to the remote machine. `PATH` is the path on the remote machine from where the data needs to be copied.
Make sure that the OpenSSH server is installed and running on the remote machine. Additionally, to avoid being prompted for a password for the remote machine, refer to the *Password-less auto-login with SSH* recipe in [Chapter 8](5ba784d5-fa8b-4840-b4c5-cac906e484f9.xhtml), *The Old-Boy Network*.

4.  Compressing data during transfer can significantly optimize the speed of the transfer. The `rsync-z` option `specifies` compressing data during transfer:

```
        $ rsync -avz source destination    

```

5.  To synchronize one directory to another directory, use the following command:

```
        $ rsync -av /home/test/ /home/backups    

```

The preceding command copies the source (`/home/test`) to an existing folder called backups.

6.  To copy a full directory inside another directory, use the following command:

```
        $ rsync -av /home/test /home/backups  

```

This command copies the source (`/home/test`) to a directory named backups by creating that directory.

For the PATH format, if we use `/` at the end of the source, `rsync` will copy the contents of the end directory specified in the `source_path` to the destination.
If `/` is not present at the end of the source, `rsync` will copy the end directory itself to the destination.
Adding the `-r` option will force `rsync` to copy all the contents of a directory, recursively.

# How it works...

The `rsync` command works with the source and destination paths, which can be either local or remote. Both paths can be remote paths. Usually, remote connections are made using SSH to provide secure, two-way communication. Local and remote paths look like this:

*   `/home/user/data` (local path)
*   `user@192.168.0.6:/home/backups/data` (remote path)

`/home/user/data` specifies the absolute path in the machine in which the `rsync` command is executed. `user@192.168.0.6:/home/backups/data` specifies that the path is `/home/backups/data` in the machine whose IP address is `192.168.0.6` and is logged in as the `user` user.

# There's more...

The `rsync` command supports several command-line options to fine-tune its behavior.

# Excluding files while archiving with rsync

The `-exclude` and -exclude-from options specify files that should not be transferred:

```
    --exclude PATTERN

```

We can specify a wildcard pattern of files to be excluded. Consider the following example:

```
$ rsync -avz /home/code/app /mnt/disk/backup/code --exclude "*.o" 

```

This command excludes the `.o` files from backing up.

Alternatively, we can specify a list of files to be excluded by providing a list file.

Use `--exclude-from FILEPATH`.

# Deleting non-existent files while updating rsync backup

By default, `rsync` does not remove files from the destination if they no longer exist at the source. The `-delete` option removes those files from the destination that do not exist at the source:

```
    $ rsync -avz SOURCE DESTINATION --delete

```

# Scheduling backups at intervals

You can create a `cron` job to schedule backups at regular intervals.

A sample is as follows:

```
    $ crontab -ev

```

Add the following line:

```
    0 */10 * * * rsync -avz /home/code user@IP_ADDRESS:/home/backups

```

The preceding `crontab` entry schedules `rsync` to be executed every 10 hours.

`*/10` is the hour position of the `crontab` syntax. `/10` specifies executing the backup every 10 hours. If `*/10` is written in the minutes position, it will execute every 10 minutes.

Have a look at the *Scheduling with a cron* recipe in [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Call*s, to understand how to configure `crontab`.

# Differential archives

The backup solutions described so far are full copies of a filesystem as it exists at that time. This snapshot is useful when you recognize a problem immediately and need the most recent snapshot to recover. It fails if you don't realize the problem until a new snapshot is made and the previous good data has been overwritten by current bad data.

An archive of a filesystem provides a history of file changes. This is useful when you need to return to an older version of a damaged file.

`rsync`, `tar`, and `cpio` can be used to make daily snapshots of a filesystem. However, backing up a full filesystem every day is expensive. Creating a separate snapshot for each day of the week will require seven times as much space as the original filesystem.

Differential backups only save the data that's changed since the last full backup. The dump/restore utilities from Unix support this style of archived backups. Unfortunately, these utilities were designed around tape drives and are not simple to use.

The find utility can be used with `tar` or `cpio` to duplicate this type of functionality.

# How to do it...

Create an initial full backup with tar:

```
    tar -cvz /backup/full.tgz /home/user

```

Use find's `-newer` flag to determine what files have changed since the full backup was created, and create a new archive:

```
    tar -czf day-`date +%j`.tgz `find /home/user -newer 
    /backup/full.tgz`

```

# How it works...

The find command generates a list of all files that have been modified since the creation of the full backup `(/backup/full.tgz`).

The date command generates a filename based on the Julian date. Thus, the first differential backup of the year will be `day-1.tgz`, the backup for January 2 will be `day-2.tgz`, and so on.

The differential archive will be larger each day as more and more files are changed from the initial full backup. When the differential archives grow too large, make a new full backup.

# Creating entire disk images using fsarchiver

The `fsarchiver` application can save the contents of a disk partition to a compressed archive file. Unlike `tar` or `cpio`, `fsarchiver` retains extended file attributes and can be restored to a disk with no current filesystem. The `fsarchiver` application recognizes and retains Windows file attributes as well as Linux attributes, making it suitable for migrating Samba-mounted partitions.

# Getting ready

The `fsarchiver` application is not installed in most distros by default. You will have to install it using your package manager. For more information, go to [http://www.fsarchiver.org/Installation](http://www.fsarchiver.org/Installation).

# How to do it...

1.  Create a backup of a `filesystem/partition`.

Use the `savefs` option of `fsarchiver` like this:

```
        fsarchiver savefs backup.fsa /dev/sda1

```

Here `backup.fsa` is the final backup file and `/dev/sda1` is the partition to backup

2.  Back-up more than one partition at the same time.

Use the `savefs` option as earlier and pass the partitions as the last parameters to `fsarchiver`:

```
        fsarchiver savefs backup.fsa /dev/sda1 /dev/sda2

```

3.  Restore a partition from a backup archive.

Use the `restfs` option of `fsarchiver` like this:

```
        fsarchiver restfs backup.fsa id=0,dest=/dev/sda1

```

`id=0` denotes that we want to pick the first partition from the archive to the partition specified as `dest=/dev/sda1`.

Restore multiple partitions from a backup archive.

As earlier, use the `restfs` option as follows:

```
      fsarchiver restfs backup.fsa id=0,dest=/dev/sda1    
      id=1,dest=/dev/sdb1

```

Here, we use two sets of the `id,dest` parameter to tell `fsarchiver` to restore the first two partitions from the backup to two physical partitions.

# How it works...

Like tar, `fsarchiver` examines the filesystem to create a list of files and then saves those files in a compressed archive file. Unlike tar which only saves information about the files, `fsarchiver` performs a backup of the filesystem as well. This makes it easier to restore the backup on a fresh partition as it is not necessary to recreate the filesystem.

If you are seeing the `/dev/sda1` notation for partitions for the first time, this requires an explanation. `/dev` in Linux holds special files called device files, which refer to a physical device. The `sd` in `sda1` refers to **SATA** disk, the next letter can be a, b, c, and so on, followed by the partition number.

![](img/b05265_07_lastimg.jpg)