# Chapter 2. ZFS

In this chapter, we will cover the following recipes:

*   Creating ZFS storage pools and filesystems
*   Playing with ZFS faults and properties
*   Creating a ZFS snapshot and clone
*   Performing a backup in a ZFS filesystem
*   Handling logs and caches
*   Managing devices in storage pools
*   Configuring spare disks
*   Handling ZFS snapshots and clones
*   Playing with COMSTAR
*   Mirroring the root pool
*   ZFS shadowing
*   Configuring ZFS sharing with the SMB share
*   Setting and getting other ZFS properties
*   Playing with the ZFS swap

# Introduction

ZFS is a 128-bit transactional filesystem offered by Oracle Solaris 11, and it supports 256 trillion directory entries, does not have any upper limit of files, and is always consistent on disk. Oracle Solaris 11 makes ZFS its default filesystem, which provides some features such as storage pool, snapshots, clones, and volumes. When administering ZFS objects, the first step is to create a ZFS storage pool. It can be made from full disks, files, and slices, considering that the minimum size of any mentioned block device is 128 MB. Furthermore, when creating a ZFS pool, the possible RAID configurations are stripe (Raid 0), mirror (Raid 1), and RAID-Z (a kind of RAID-5). Both the mirror and RAID-Z configurations support a feature named self-healing data that works by protecting data. In this case, when a bad block arises in a disk, the ZFS framework fetches the same block from another replicated disk to repair the original bad block. RAID-Z presents three variants: raidz1 (similar to RAID-5) that uses at least three disks (two data and one parity), raidz2 (similar to RAID-6) that uses at least five disks (3D and 2P), and raidz3 (similar to RAID-6, but with an additional level of parity) that uses at least eight disks (5D and 3P).

# Creating ZFS storage pools and filesystems

To start playing with ZFS, the first step is to create a storage pool, and afterwards, all filesystems will be created inside these storage pools. To accomplish the creation of a storage pool, we have to decide which raid configuration we will use (stripe, mirror, or RAID-Z) to create the storage pool and, afterwards, the filesystems on it.

## Getting ready

To follow this recipe, it is necessary to use a virtual machine (VMware or VirtualBox) that runs Oracle Solaris 11 with 4 GB RAM and eight 4 GB disks. Once the virtual machine is up and running, log in as the root user and open a terminal.

## How to do it…

A storage pool is a logical object, and it represents the physical characteristics of the storage and must be created before anything else. To create a storage pool, the first step is to list all the available disks on the system and choose what disks will be used by running the following command as the root role:

```
root@solaris11-1:~# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0
       1\. c8t1d0 <VBOX-HARDDISK-1.0-16.00GB>
          /pci@0,0/pci1000,8000@14/sd@1,0
       2\. c8t2d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@2,0
       3\. c8t3d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@3,0
       4\. c8t4d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@4,0
       5\. c8t5d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@5,0
       6\. c8t6d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@6,0
       7\. c8t8d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@8,0
       8\. c8t9d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@9,0
       9\. c8t10d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@a,0
      10\. c8t11d0 <VBOX-HARDDISK-1.0 cyl 2046 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@b,0
Specify disk (enter its number):
```

Following the selection of disks, create a `zpool create` storage pool and verify the information about this pool using the `zpool list` and `zpool status` commands. Before these steps, we have to decide the pool configuration: stripe (default), mirror, raidz, raidz2, or raidz3\. If the configuration isn't specified, stripe (raid0) will be assumed as default. Then, a pool is created by running the following command:

```
root@solaris11-1:~# zpool create oracle_stripe_1 c8t3d0 c8t4d0
'oracle_stripe_1' successfully created, but with no redundancy; failure of one device will cause loss of the pool
```

To list the pool, execute the following commands:

```
root@solaris11-1:~# zpool list oracle_stripe_1
NAME              SIZE   ALLOC  FREE   CAP  DEDUP  HEALTH  ALTROOT
oracle_stripe_1   7.94G  122K   7.94G  0%   1.00x  ONLINE  -
```

To verify the status of the pool, run the following commands:

```
root@solaris11-1:~# zpool status oracle_stripe_1
  pool: oracle_stripe_1
 state: ONLINE
  scan: none requested
config:

  NAME             STATE     READ WRITE CKSUM
  oracle_stripe_1  ONLINE       0     0     0
  c8t3d0           ONLINE       0     0     0
  c8t4d0           ONLINE       0     0     0

errors: No known data errors
```

Although it's out of the scope of this chapter, we can list some related performance information by running the following command:

```
root@solaris11-1:~# zpool iostat -v oracle_stripe_1
                  capacity       operations    bandwidth
pool              alloc   free   read  write   read  write
----------------  -----  -----  -----  -----  -----  -----
oracle_stripe_1   128K    7.94G    0      0    794     56
  c8t3d0          53K     3.97G    0      0    391     24
  c8t4d0          74.5K   3.97G    0      0    402     32
----------------  -----  -----  -----  -----  -----  -----
```

If necessary, a second and third storage pool can be created using the same commands but taking different disks and, in this case, by changing to the `mirror` and `raidz` configurations, respectively. This task is accomplished by running the following commands:

```
root@solaris11-1:~# zpool create oracle_mirror_1 mirror c8t5d0 c8t6d0
root@solaris11-1:~# zpool list oracle_mirror_1
NAME              SIZE   ALLOC  FREE   CAP  DEDUP  HEALTH  ALTROOT
oracle_mirror_1   3.97G  85K    3.97G  0%   1.00x  ONLINE  -
root@solaris11-1:~# zpool status oracle_mirror_1
  pool: oracle_mirror_1
 state: ONLINE
  scan: none requested
config:
  NAME             STATE      READ  WRITE  CKSUM
  oracle_mirror_1  ONLINE       0     0      0
  mirror-0         ONLINE       0     0      0
  c8t5d0           ONLINE       0     0      0
  c8t6d0           ONLINE       0     0      0

errors: No known data errors
root@solaris11-1:~# zpool create oracle_raidz_1 raidz c8t8d0 c8t9d0 c8t10d0
root@solaris11-1:~# zpool list oracle_raidz_1
NAME             SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
oracle_raidz_1  11.9G   176K  11.9G   0%  1.00x  ONLINE  -
root@solaris11-1:~# zpool status oracle_raidz_1
pool: oracle_raidz_1
 state: ONLINE
  scan: none requested
config:

  NAME            STATE     READ WRITE CKSUM
  oracle_raidz_1  ONLINE       0     0     0
  raidz1-0        ONLINE       0     0     0
  c8t8d0          ONLINE       0     0     0
  c8t9d0          ONLINE       0     0     0
  c8t10d0         ONLINE       0     0     0

errors: No known data errors
```

Once the storage pools are created, it's time to create filesystems in these pools. First, let's create a filesystem named `zfs_stripe_1` in the `oracle_stripe_1` pool. Execute the following command:

```
root@solaris11-1:~# zfs create oracle_stripe_1/zfs_stripe_1

```

Repeating the same syntax, it's easy to create two new filesystems named `zfs_mirror_1` and `zfs_raidz_1` in `oracle_mirror_1` and `oracle_raidz_1`, respectively:

```
root@solaris11-1:~# zfs create oracle_mirror_1/zfs_mirror_1
root@solaris11-1:~# zfs create oracle_raidz_1/zfs_raidz_1

```

The listing of recently created filesystems is done by running the following command:

```
root@solaris11-1:~# zfs list
NAME                            USED   AVAIL  REFER  MOUNTPOINT
(truncated output)
oracle_mirror_1                 124K   3.91G  32K    /oracle_mirror_1
oracle_mirror_1/zfs_mirror_1    31K    3.91G  31K  /oracle_mirror_1/zfs_mirror_1
oracle_raidz_1                  165K   7.83G  36.0K  /oracle_raidz_1
oracle_raidz_1/zfs_raidz_1      34.6K  7.83G  34.6K  /oracle_raidz_1/zfs_raidz_1
oracle_stripe_1                 128K   7.81G  32K    /oracle_stripe_1
oracle_stripe_1/zfs_stripe_1    31K    7.81G  31K  /oracle_stripe_1/zfs_stripe_1
(truncated output)
root@solaris11-1:~# zfs list oracle_stripe_1 oracle_mirror_1 oracle_raidz_1
NAME             USED  AVAIL  REFER  MOUNTPOINT
oracle_mirror_1  124K  3.91G    32K  /oracle_mirror_1
oracle_raidz_1   165K  7.83G  36.0K  /oracle_raidz_1
oracle_stripe_1  128K  7.81G    32K  /oracle_stripe_1
```

The ZFS engine has automatically created the mount-point directory for all the created filesystems, and it has been mounted on them. This can also be verified by executing the following command:

```
root@solaris11-1:~# zfs mount
rpool/ROOT/solaris              /
rpool/ROOT/solaris/var          /var
rpool/VARSHARE                  /var/share
rpool/export                    /export
rpool/export/home               /export/home
oracle_mirror_1                 /oracle_mirror_1
oracle_mirror_1/zfs_mirror_1    /oracle_mirror_1/zfs_mirror_1
oracle_stripe_1                 /oracle_stripe_1
oracle_stripe_1/zfs_stripe_1    /oracle_stripe_1/zfs_stripe_1
rpool                           /rpool
oracle_raidz_1                  /oracle_raidz_1
oracle_raidz_1/zfs_raidz_1      /oracle_raidz_1/zfs_raidz_1
```

The last two lines confirm that the ZFS filesystems that we created are already mounted and ready to use.

### An overview of the recipe

This recipe has taught us how to create a storage pool with different configurations such as stripe, mirror, and raidz. Additionally, we learned how to create filesystems in these pools.

# Playing with ZFS faults and properties

ZFS is completely oriented by properties that can change the behavior of storage pools and filesystems. This recipe will touch upon important properties from ZFS, and we will learn how to handle them.

## Getting ready

To follow this recipe, it is necessary to use a virtual machine (VMware or VirtualBox) that runs Oracle Solaris 11 with 4 GB RAM and eight 4 GB disks. Once the virtual machine is up and running, log in as the root user and open a terminal.

## How to do it…

Every ZFS object has properties that can be accessed and, most of the time, changed. For example, to get the pool properties, we must execute the following command:

```
root@solaris11-1:~# zpool get all oracle_mirror_1
NAME                PROPERTY       VALUE               SOURCE
(truncated output)
oracle_mirror_1     bootfs         -                   default
oracle_mirror_1     cachefile      -                   default
oracle_mirror_1     capacity       0%                  -
oracle_mirror_1     dedupditto     0                   default
oracle_mirror_1     dedupratio     1.00x               -
oracle_mirror_1     delegation     on                  default
oracle_mirror_1     failmode       wait                default
oracle_mirror_1     free           3.97G               -
oracle_mirror_1     guid           730796695846862911  -
(truncated output)

```

Some useful information from the previous output is that the free space is 3.97 GB (the `free` property), the pool is online (the `health` property), and `0%` of the total capacity was used (the `capacity` property). If we need to know about any problem related to the pool (referring to the `health` property), it's recommended that you get this information by running the following command:

```
root@solaris11-1:~# zpool status -x 
all pools are health
root@solaris11-1:~# zpool status -x oracle_mirror_1
pool 'oracle_mirror_1' is healthy
root@solaris11-1:~# zpool status oracle_mirror_1
  pool: oracle_mirror_1
 state: ONLINE
  scan: none requested
config:

    NAME             STATE     READ WRITE CKSUM
    oracle_mirror_1  ONLINE       0     0     0
    mirror-0         ONLINE       0     0     0
    c8t5d0           ONLINE       0     0     0
    c8t6d0           ONLINE       0     0     0
```

Another fantastic method to check whether all data in the specified storage pool is okay is using the `zpool scrub` command that examines whether the checksums are correct, and for replicated devices (such as mirror and raidz configurations), the `zpool scrub` command repairs any discovered problem. To follow the `zpool scrub` results, the `zpool status` command can be used as follows:

```
root@solaris11-1:~# zpool scrub oracle_mirror_1
root@solaris11-1:~# zpool status oracle_mirror_1
  pool: oracle_mirror_1
 state: ONLINE
scan: scrub in progress since Tue Jun 10 04:04:56 2014
    2.53G scanned out of 3.91G at 24.0M/s, 0h1m to go
    0 repaired, 64.71% done
config:

    NAME             STATE     READ WRITE CKSUM
    oracle_mirror_1  ONLINE       0     0     0
    mirror-0         ONLINE       0     0     0
    c8t5d0           ONLINE       0     0     0
    c8t6d0           ONLINE       0     0     0
```

After some time, if everything went well, the same `zpool` status command should show the following output:

```
root@solaris11-1:~# zpool status oracle_mirror_1
  pool: oracle_mirror_1
 state: ONLINE
scan: scrub repaired 0 in 0h4m with 0 errors on Tue Jun 10 04:09:48 2014
config:

    NAME             STATE     READ WRITE CKSUM
    oracle_mirror_1  ONLINE    0     0     0
        mirror-0  ONLINE       0     0     0
          c8t5d0  ONLINE       0     0     0
          c8t6d0  ONLINE       0     0     0
```

During an analysis of possible disk errors, the following `zpool history` command, which shows all the events that occurred on the pool, could be interesting and suitable:

```
root@solaris11-1:~# zpool history oracle_mirror_1
History for 'oracle_mirror_1':
2013-11-27.19:14:15 zpool create oracle_mirror_1 mirror c8t5d0 c8t6d0
2013-11-27.19:57:31 zfs create oracle_mirror_1/zfs_mirror_1
(truncated output)

```

The Oracle Solaris Fault Manager, through its `fmd` daemon, is a framework that receives any information related to potential problems that were detected by the system, diagnoses these problems and, eventually, takes a proactive action to keep the system integrity such as disabling a memory module. Therefore, this framework offers the following `fmadm` command that, when used with the `faulty` argument, displays information about resources that the Oracle Solaris Fault Manager believes to be faulty:

```
root@solaris11-1:~# fmadm faulty

```

The following `dmesg` command confirms any suspicious hardware error:

```
root@solaris11-1:~# dmesg

```

From the `zpool status` command, there are some possible values for the `status` field:

*   `ONLINE`:. This means that the pool is good
*   `FAULTED`: This means that the pool is bad
*   `OFFLINE`: This means that the pool was disabled by the administrator
*   `DEGRADED`: This means that something (likely a disk) is bad, but the pool is still working
*   `REMOVED`: This means that a disk was hot-swapped
*   `UNAVAIL`: This means that the device or virtual device can be opened

Returning to ZFS properties, it's easy to get property information from a ZFS filesystem by running the following commands:

```
root@solaris11-1:~# zfs list -r oracle_mirror_1
NAME                          USED  AVAIL  REFER  MOUNTPOINT
oracle_mirror_1               124K  3.91G    32K  /oracle_mirror_1
oracle_mirror_1/zfs_mirror_1   31K  3.91G    31K  /oracle_mirror_1/zfs_mirror_1
root@solaris11-1:~# zfs get all oracle_mirror_1/zfs_mirror_1
NAME                          PROPERTY          VALUE         SOURCE
oracle_mirror_1/zfs_mirror_1  aclinherit        restricted    default
oracle_mirror_1/zfs_mirror_1  aclmode           discard       default
oracle_mirror_1/zfs_mirror_1  atime             on            default
oracle_mirror_1/zfs_mirror_1  available         3.91G         -
oracle_mirror_1/zfs_mirror_1  canmount          on            default
oracle_mirror_1/zfs_mirror_1  casesensitivity   mixed         -
oracle_mirror_1/zfs_mirror_1  checksum          on            default
(truncated output)

```

The previous two commands deserve an explanation—`zfs list –r` shows all the datasets (filesystems, snapshots, clones, and so on) under the `oracle_mirror_1` storage pool. Additionally, `zfs get all oracle_mirror_1/zfs_mirror_1` displays all the properties from the `zfs_mirror_1` filesystem.

There are many filesystem properties (some of them are read-only and others read-write), and it's advisable to know some of them. Almost all are inheritable—a child (for example, a snapshot or clone object) inherits a configured value for a parent object (for example, a filesystem).

Setting a property value is done by executing the following command:

```
root@solaris11-1:~# zfs set mountpoint=/oracle_mirror_1/another_point oracle_mirror_1/zfs_mirror_1
root@solaris11-1:~# zfs list -r oracle_mirror_1
NAME                          USED  AVAIL  REFER  MOUNTPOINT
oracle_mirror_1               134K  3.91G    32K  /oracle_mirror_1
oracle_mirror_1/zfs_mirror_1   31K  3.91G    31K  /oracle_mirror_1/another_point
```

The old mount point was renamed to the `/oracle_mirror_1/another_point` directory and remounted again. Later, we'll return to this point and review some properties.

When it's necessary, a ZFS filesystem has to be renamed by running the following command:

```
root@solaris11-1:~# zfs rename oracle_stripe_1/zfs_stripe_1 oracle_stripe_1/zfs_test_1
root@solaris11-1:~# zfs list -r oracle_stripe_1
NAME                         USED  AVAIL  REFER  MOUNTPOINT
oracle_stripe_1              128K  7.81G    32K  /oracle_stripe_1
oracle_stripe_1/zfs_test_1   31K   7.81G    31K  /oracle_stripe_1/zfs_test_1
root@solaris11-1:~# df -h /oracle_stripe_1/*
Filesystem             Size   Used  Available Capacity  Mounted on
oracle_stripe_1/zfs_test_1
                       7.8G    31K       7.8G     1%    /oracle_stripe_1/zfs_test_1
```

Oracle Solaris 11 automatically altered the mount point of the renamed filesystem and remounted it again.

To destroy a ZFS filesystem or storage pool, there can't be any process that accesses the dataset. For example, if we try to delete the `zfs_test` filesystem when a process is using the directory, we get an error:

```
root@solaris11-1:~# cd /oracle_stripe_1/zfs_test_1
root@solaris11-1:~# zfs list -r oracle_stripe_1
NAME                          USED  AVAIL  REFER  MOUNTPOINT
oracle_stripe_1               128K  7.81G    32K  /oracle_stripe_1
oracle_stripe_1/zfs_test_1  31.5K  7.81G  31.5K  /oracle_stripe_1/zfs_test_1
root@solaris11-1:~# zfs destroy oracle_stripe_1/zfs_test_1
cannot unmount '/oracle_stripe_1/zfs_test_1': Device busy
```

This case presents several possibilities—first (and the most recommended) is to understand what processes or applications are using the mentioned filesystem. Once the guilty processes or applications are found, the next step is to stop them. Therefore, everything is solved without losing any data. However, if there isn't any possibility to find the guilty processes, then killing the offending process(es) would be a feasible and unpredictable option, where data loss would be probable. Finally, using the `-f` option would cause a *forced destroy*, which, obviously, is not advisable and would probably cause data loss. The following is the second procedure (killing the problematic process) by running the following commands:

```
root@solaris11-1:~# fuser -cu /oracle_stripe_1/zfs_test_1
/oracle_stripe_1/zfs_test_1:     1977c(root)
root@solaris11-1:~# ps -ef | grep 1977
    root  1977  1975   0 07:03:14 pts/1       0:00 bash
```

We used the `fuser` command that enables us to look for processes that access a specific file or directory. Therefore, according to the previous two outputs, there's a process using the `/oracle_stripe_1/zfs_test_1` filesystem, and the `ps –ef` command reveals that `bash` is the guilty process, which is correct because we changed the mount point before trying to delete it. To solve this, it would be enough to leave the `/oracle_stripe_1/zfs_test_1` directory. Nonetheless, if we didn't know how to solve the problem, the last resource would be to kill the offending process by running the following command:

```
root@solaris11-1:~# kill -9 1977

```

At this time, there isn't a process accessing the filesystem, so it's possible to destroy it:

```
root@solaris11-1:~# zfs destroy oracle_stripe_1/zfs_test_1

```

To verify whether the filesystem was correctly destroyed, execute the following command:

```
root@solaris11-1:~# zfs list -r oracle_stripe_1
NAME              USED   AVAIL  REFER  MOUNTPOINT
oracle_stripe_1   89.5K  7.81G    31K  /oracle_stripe_1
```

Everything worked fine, and the filesystem was destroyed. Nonetheless, if there was a snapshot or clone under this filesystem (we'll review and learn about them in the next recipe), we wouldn't have been able to delete the filesystem, and we should use the same command with the`–r` option (for snapshots inside) or `–R` (for snapshots and clones inside). From here, it's also possible to destroy the whole pool using the `zpool destroy` command. Nevertheless, we should take care of a single detail—if there isn't any process using any filesystem from the pool to be destroyed, Oracle Solaris 11 doesn't prompt any question about the pool destruction. Everything inside the pool is destroyed without any question (so different from the Windows system, which prompts a warning before a dangerous action). To prove this statement, in the next example, we're going to create one filesystem in the `oracle_stripe_1` pool, put some information into it, and, at the end, we're going to destroy all pools:

```
root@solaris11-1:~# zfs list -r oracle_stripe_1
NAME              USED  AVAIL  REFER  MOUNTPOINT
oracle_stripe_1  89.5K  7.81G    31K  /oracle_stripe_1
root@solaris11-1:~# zfs create oracle_stripe_1/fs_1
root@solaris11-1:~# cp /etc/[a-e]* /oracle_stripe_1/fs_1
root@solaris11-1:~# zfs list -r oracle_stripe_1     
NAME                   USED  AVAIL  REFER  MOUNTPOINT
oracle_stripe_1       4.01M  7.81G    35K  /oracle_stripe_1
oracle_stripe_1/fs_1  82.5K  7.81G  82.5K  /oracle_stripe_1/fs_1
root@solaris11-1:~# zpool list oracle_stripe_1
NAME              SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
oracle_stripe_1  7.94G  4.01M  7.93G   0%  1.00x  ONLINE  -
root@solaris11-1:~# zpool destroy oracle_stripe_1
root@solaris11-1:~# zpool list
NAME              SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
iscsi_pool       3.97G  2.62M  3.97G   0%  1.00x  ONLINE  -
oracle_mirror_1  3.97G   134K  3.97G   0%  1.00x  ONLINE  -
oracle_raidz_1   11.9G   248K  11.9G   0%  1.00x  ONLINE  -
repo_pool        15.9G  7.64G  8.24G  48%  1.00x  ONLINE  -
rpool            79.5G  31.8G
```

### An overview of the recipe

Taking the `zpool` and `zfs` commands, we created, listed, renamed, and destroyed pools and filesystems. Furthermore, we learned how to view properties and alter them, especially the mount point property that's very essential for daily ZFS administration. We also learned how to see the pool history, monitor the pool, and gather important information about related pool failures.

# Creating a ZFS snapshot and clone

A ZFS snapshot and clone play fundamental roles in the ZFS framework and in Oracle Solaris 11, as there are many uses for these features, and one of them is to execute backup and restore files from the ZFS filesystem. For example, a snapshot could be handy when either there is some corruption in the ZFS filesystem or a user loses a specific file. Using ZFS snapshots makes it possible to completely rollback the ZFS filesystem to a specific point or date.

## Getting ready

To follow this recipe, it is necessary to use a virtual machine (VMware or VirtualBox) that runs Oracle Solaris 11 with 4 GB RAM and eight 4 GB disks. Once the virtual machine is up and running, log in as the root user and open a terminal.

## How to do it…

Creating a snapshot is a fundamental task that can be executed by running the following commands:

```
root@solaris11-1:~# zpool create pool_1 c8t3d0
root@solaris11-1:~# zfs create pool_1/fs_1

```

Before continuing, I suggest that we copy some big files to the `pool_1/fs_1` filesystem. In this case, I used files that I already had on my system, but you can copy anything into the filesystem. Run the following commands:

```
root@solaris11-1:~# cp -r mh* jo* /pool_1/fs_1/
root@solaris11-1:~# zfs list -r pool_1/fs_1 
NAME          USED  AVAIL  REFER  MOUNTPOINT
pool_1/fs_1  63.1M  3.85G  63.1M  /pool_1/fs_1
```

Finally, we create the snapshot by running the following command:

```
root@solaris11-1:~# zfs snapshot pool_1/fs_1@snap1

```

By default, snapshots aren't shown even when using the `zfs list -r` command:

```
root@solaris11-1:~# zfs list -r pool_1
NAME          USED  AVAIL  REFER  MOUNTPOINT
pool_1       63.2M  3.85G    32K  /pool_1
pool_1/fs_1  63.1M  3.85G  63.1M  /pool_1/fs_1
```

This behavior is controlled by the `listsnapshots` property (its value is `off` by default) from the pool:

```
root@solaris11-1:~# zpool get listsnapshots pool_1
NAME    PROPERTY       VALUE  SOURCE
pool_1  listsnapshots  off    local
```

It's necessary to alter `listsnapshots` to `on` to change this behavior:

```
root@solaris11-1:~# zpool set listsnapshots=on pool_1
root@solaris11-1:~# zfs list -r pool_1
NAME                USED  AVAIL  REFER  MOUNTPOINT
pool_1             63.2M  3.85G    32K  /pool_1
pool_1/fs_1        63.1M  3.85G  63.1M  /pool_1/fs_1
pool_1/fs_1@snap1      0      -  63.1M  -
```

It worked as planned. However, when executing the previous command, all datasets (filesystems and snapshots) are listed. To list only snapshots, it is necessary to specify a filter using the`–t` option as follows:

```
root@solaris11-1:~# zfs list -t snapshot
NAME                                         USED  AVAIL  REFER  MOUNTPOINT
pool_1/fs_1@snap1                               0      -  63.1M  -
rpool/ROOT/solaris@install                   106M      -  3.52G  -
rpool/ROOT/solaris@2013-10-10-22:27:20       219M      -  3.77G  -
rpool/ROOT/solaris@2013-11-26-08:38:27      1.96G      -  24.2G  -
rpool/ROOT/solaris/var@install              63.0M      -   189M  -
rpool/ROOT/solaris/var@2013-10-10-22:27:20  66.5M      -   200M  -
rpool/ROOT/solaris/var@2013-11-26-08:38:27   143M      -   291M  -
```

The previous command has shown only the existing snapshots as expected. An interesting fact is that snapshots live inside filesystems, and initially, they don't take any space on disk. However, as the filesystem is being altered, snapshots take free space, and this could be a big concern. Considering this, the `SIZE` property equals zero and `REFER` equals `63.1M`, which is the exact size of the `pool_1/fs_1` filesystem.

The `REFER` field deserves an explanation—when snapshots are explained in any IT area, the classification is the same. There are physical snapshots and logical snapshots. Physical snapshots take the same space from a reference filesystem, and both don't have any impact on each other during the read/write operations. The creation of the snapshot takes a long time, because it's a kind of "copy" of everything from the reference filesystem. In this case, the snapshot is a static picture that represents the filesystem at the exact time when the snapshot was created. After this initial time, snapshots won't be synchronized with the reference filesystem anymore. If the administrator wants both synchronized, they should do it manually.

The other classification, logical snapshots, is very different from the first one. When a logical snapshot is made, only pointers to data from the reference filesystem are created, but there is no data inside the snapshot. This process is very fast and takes little disk space. The disadvantage is that any read operation impacts the reference filesystem. There are two additional effects—when some data changes in the reference filesystem, the operating system copies the data to be modified to the snapshot before being modified itself (this process is called **copy** **on write** (**COW**)). Why? Because of our previous explanation that snapshots are a static picture of an exact time from the reference filesystem. If some data changes, the snapshot has to be unaltered, and it must contain the same data from the time that it was created. A second and worse effect is that if the reference filesystem is lost, every snapshot becomes invalid. Why? Because the reference filesystem doesn't exist anymore, and all pointers become invalid.

Return to the `REFER` field explanation; it means how much data in the reference filesystem is being referenced by a pointer in the snapshot. A clone is a copy of a filesystem, and it's based on snapshots, so to create a clone, a snapshot must be made first. However, there's a fundamental difference between a clone and snapshot—a snapshot is a read-only object, and a clone is a read/write object. Therefore, it's possible to write in a clone as we're able to write in a filesystem. Other interesting facts are that as the snapshot must exist before creating a clone, the clone is dependent on the snapshot, and both must be created in the same pool. Create a pool by executing the following commands:

```
root@solaris11-1:~# zfs clone pool_1/fs_1@snap1 pool_1/clone_1 
root@solaris11-1:~# zfs list -r pool_1
NAME                USED  AVAIL  REFER  MOUNTPOINT
pool_1             63.2M  3.85G    33K  /pool_1
pool_1/clone_1       25K  3.85G  63.1M  /pool_1/clone_1
pool_1/fs_1        63.1M  3.85G  63.1M  /pool_1/fs_1
pool_1/fs_1@snap1      0      -  63.1M  -
```

If we look at this output, it's complicated to distinguish a clone from a filesystem. Nonetheless, we could gather enough details to be able to distinguish the datasets:

```
root@solaris11-1:~# zfs get origin pool_1/fs_1
NAME         PROPERTY  VALUE  SOURCE
pool_1/fs_1  origin    -      -
root@solaris11-1:~# zfs get origin pool_1/fs_1@snap1
NAME               PROPERTY  VALUE  SOURCE
pool_1/fs_1@snap1  origin    -      -
root@solaris11-1:~# zfs get origin pool_1/clone_1   
NAME            PROPERTY  VALUE              SOURCE
pool_1/clone_1  origin    pool_1/fs_1@snap1  -
```

The `origin` property doesn't show anything relevant to pools and snapshots, but when this property is analyzed on a clone context, it shows us that the clone originated from the `pool1_/fs_1@snap1` snapshot. Therefore, it's feasible to confirm that `pool_1/fs_1@snap1` is indeed a snapshot by running the following command:

```
root@solaris11-1:~# zfs get type pool_1/fs_1@snap1
NAME               PROPERTY  VALUE     SOURCE
pool_1/fs_1@snap1  type      snapshot  -
```

In ZFS, the object creation order is `pool` | `filesystem` | `snapshot` | `clone`. So, the destruction order should be the inverse: `clone` | `snapshot` | `filesystem` | `pool`. It's possible to skip steps using special options that we'll learn about later.

For example, if we try to destroy a filesystem that contains a snapshot, the following error will be shown:

```
root@solaris11-1:~# zfs destroy pool_1/fs_1
cannot destroy 'pool_1/fs_1': 
filesystem has children
use '-r' to destroy the following datasets:
pool_1/fs_1@snap1
```

In the same way, if we try to destroy a snapshot without removing the clone first, the following message will be shown:

```
root@solaris11-1:~# zfs destroy pool_1/fs_1@snap1
cannot destroy 'pool_1/fs_1@snap1': 
snapshot has dependent clones
use '-R' to destroy the following datasets:
pool_1/clone_1
```

The last two cases have shown that it's necessary to follow the right order to destroy datasets in ZFS. Execute the following command:

```
root@solaris11-1:~# zfs list -r pool_1
NAME                USED  AVAIL  REFER  MOUNTPOINT
pool_1             63.2M  3.85G    33K  /pool_1
pool_1/clone_1       25K  3.85G  63.1M  /pool_1/clone_1
pool_1/fs_1        63.1M  3.85G  63.1M  /pool_1/fs_1
pool_1/fs_1@snap1      0      -  63.1M  -
root@solaris11-1:~# zfs destroy pool_1/clone_1
root@solaris11-1:~# zfs destroy pool_1/fs_1@snap1
root@solaris11-1:~# zfs destroy pool_1/fs_1
root@solaris11-1:~# zfs list -r pool_1 
NAME     USED  AVAIL  REFER  MOUNTPOINT
pool_1  98.5K  3.91G    31K  /pool_1
```

When the correct sequence is followed, it's possible to destroy each dataset one by one, although, as we mentioned earlier, it would be possible to skip steps. The next sequence shows how this is possible. Execute the following command:

```
root@solaris11-1:~# zfs destroy -R pool_1/fs_1
root@solaris11-1:~# zfs list -r pool_1
NAME    USED  AVAIL  REFER  MOUNTPOINT
pool_1   91K  3.91G    31K  /pool_1
root@solaris11-1:~#
```

Finally, we used the `-R` option, and everything was destroyed—including the clone, snapshot, and filesystem.

### An overview of the recipe

We learned how to manage snapshots and clones, including how to create, list, distinguish, and destroy them. Finally, this closes our review about the fundamentals of ZFS.

# Performing a backup in a ZFS filesystem

Ten years ago, I didn't think about learning how to use any backup software, and honestly, I didn't like this kind of software because I thought it was so simple. Nowadays, I can see why I was so wrong.

Administering and managing backup software is the most fundamental activity in IT, acting as the last line of defense against hackers. By the way, hackers are winning the war using all types of resources—malwares, Trojans, viruses, worms, and spywares, and only backups of file servers and applications can save a company.

Oracle Solaris 11 offers a simple solution composed of two commands (`zfs send` and `zfs recv`) to back up ZFS filesystem data. During the backup operation, data is generated as a stream and sent (using the `zfs send` command) through the network to another Oracle Solaris 11 system that receives this stream (using `zfs recv`).

Oracle Solaris 11 is able to produce two kinds of streams: the replication stream, which includes the filesystem and all its dependent datasets (snapshots and clones), and the recursive stream, which includes the filesystems and clones, but excludes snapshots. The default stream type is the replication stream.

This recipe will show you how to execute a backup and restore operation.

## Getting ready

To follow this recipe, it's necessary to have two virtual machines (VMware or VirtualBox) that run Oracle Solaris 11, with 4 GB RAM each and eight 4 GB disks. The systems used in this recipe are named `solaris11-1` and `solaris11-2`.

## How to do it…

All the ZFS backup operations are based on snapshots. This procedure will do everything from the beginning—creating a pool, filesystem, and snapshot and then executing the backup. Execute the following commands:

```
root@solaris11-1:~# zpool create backuptest_pool c8t5d0
root@solaris11-1:~# zfs create backuptest_pool/zfs1
root@solaris11-1:~# cp /etc/[a-p]* /backuptest_pool/zfs1
root@solaris11-1:/# ls -l /backuptest_pool/zfs1/
total 399
-rw-r--r--   1 root     root        1436 Dec 13 03:30 aliases
-rw-r--r--   1 root     root         182 Dec 13 03:30 auto_home
-rw-r--r--   1 root     root         220 Dec 13 03:30 auto_master
-rw-r--r--   1 root     root        1931 Dec 13 03:30 dacf.conf
(truncated output)
root@solaris11-1:/# zfs list backuptest_pool/zfs1 
NAME                  USED  AVAIL  REFER  MOUNTPOINT
backuptest_pool/zfs1  214K  3.91G   214K  /backuptest_pool/zfs1
root@solaris11-1:/# zfs snapshot backuptest_pool/zfs1@backup1
root@solaris11-1:/# zpool listsnapshots=on backuptest_pool 
root@solaris11-1:/# zfs list -r backuptest_pool
NAME                          USED  AVAIL  REFER  MOUNTPOINT
backuptest_pool               312K  3.91G    32K  /backuptest_pool
backuptest_pool/zfs1          214K  3.91G   214K  /backuptest_pool/zfs1
backuptest_pool/zfs1@backup1     0      -   214K  -
```

The following commands remove some files from the `backuptest_pool/zfs1` filesystem:

```
root@solaris11-1:/# cd /backuptest_pool/zfs1/
root@solaris11-1:/backuptest_pool/zfs1# rm [a-k]*
root@solaris11-1:/backuptest_pool/zfs1# ls -l
total 125
-rw-r--r--   1 root     root        2986 Dec 13 03:30 name_to_major
-rw-r--r--   1 root     root        3090 Dec 13 03:30 name_to_sysnum
-rw-r--r--   1 root     root        7846 Dec 13 03:30 nanorc
-rw-r--r--   1 root     root        1321 Dec 13 03:30 netconfig
-rw-r--r--   1 root     root         487 Dec 13 03:30 netmasks
-rw-r--r--   1 root     root         462 Dec 13 03:30 networks
-rw-r--r--   1 root     root        1065 Dec 13 03:30 nfssec.conf
……….
(truncated output)

```

We omitted a very interesting fact about snapshots—when any file is deleted from the filesystem, it doesn't disappear forever. There is a hidden directory named `.zfs` inside each filesystem; it contains snapshots, and all the removed files go to a subdirectory inside this hidden directory. Let's look at the following commands:

```
root@solaris11-1:~# cd /backuptest_pool/zfs1/.zfs
root@solaris11-1:/backuptest_pool/zfs1/.zfs# ls
shares    snapshot
root@solaris11-1:/backuptest_pool/zfs1/.zfs# cd snapshot/
root@solaris11-1:/backuptest_pool/zfs1/.zfs/snapshot# ls
backup1
root@solaris11-1:/backuptest_pool/zfs1/.zfs/snapshot# cd backup1/
root@solaris11-1:/backuptest_pool/zfs1/.zfs/snapshot/backup1# ls -l
total 399
-rw-r--r--   1 root     root        1436 Dec 13 03:30 aliases
-rw-r--r--   1 root     root         182 Dec 13 03:30 auto_home
-rw-r--r--   1 root     root         220 Dec 13 03:30 auto_master
-rw-r--r--   1 root     root        1931 Dec 13 03:30 dacf.conf
-r--r--r--   1 root     root         516 Dec 13 03:30 datemsk
-rw-r--r--   1 root     root        2670 Dec 13 03:30 devlink.tab
-rw-r--r--   1 root     root       38237 Dec 13 03:30 driver_aliases
………
(truncated output)

root@solaris11-1:/backuptest_pool/zfs1/.zfs/snapshot/backup1# cd

```

Using this information about the localization of deleted files, any file could be restored, and even better, it would be possible to revert the filesystem to the same content as when the snapshot was taken. This operation is named `rollback`, and it can be executed using the following commands:

```
root@solaris11-1:~# zfs rollback backuptest_pool/zfs1@backup1
root@solaris11-1:~# cd /backuptest_pool/zfs1/
root@solaris11-1:/backuptest_pool/zfs1# ls -l    
total 399
-rw-r--r--   1 root     root        1436 Dec 13 03:30 aliases
-rw-r--r--   1 root     root         182 Dec 13 03:30 auto_home
-rw-r--r--   1 root     root         220 Dec 13 03:30 auto_master
-rw-r--r--   1 root     root        1931 Dec 13 03:30 dacf.conf
-r--r--r--   1 root     root         516
 Dec 13 03:30 datemsk
-rw-r--r--   1 root     root        2670 Dec 13 03:30 devlink.tab
-rw-r--r--   1 root     root       38237 Dec 13 03:30 driver_aliases
(truncated output)

```

Every single file was restored to the filesystem, as nothing had happened.

Going a step ahead, let's see how to back up the filesystem data to another system that runs Oracle Solaris 11\. The first step is to connect to another system (`solaris 11-2`) and create and prepare a pool to receive the backup stream from the `solaris11-1` source system by running the following commands:

```
root@solaris11-1:~# ssh solaris11-2
Password: 
Last login: Fri Dec 13 04:29:41 2013
Oracle Corporation      SunOS 5.11      11.1    September 2012
root@solaris11-2:~# zpool create away_backup c8t3d0
root@solaris11-2:~# zpool set readonly=on away_backup
root@solaris11-2:~# zfs list away_backup
NAME         USED  AVAIL  REFER  MOUNTPOINT
away_backup   85K  3.91G    31K  /away_backup
```

We enabled the `readonly` property from `away_pool`. Why? Because we have to keep the metadata consistent while receiving data from another host and afterwards too.

Continuing this procedure, the next step is to execute the remote backup from the `solaris11-1` source machine, sending all filesystem data to the `solaris11-2` target machine:

```
root@solaris11-1:~# zfs send backuptest_pool/zfs1@backup1 | ssh solaris11-2 zfs recv -F away_backup/saved_backup
Password:
```

We used the `ssh` command to send all data through a secure tunnel, but we could have used the `netcat` command (it's included in Oracle Solaris, and there's more information about it on [http://netcat.sourceforge.net/](http://netcat.sourceforge.net/)) if security isn't a requirement.

You can verify that all data is present on the target machine by executing the following command:

```
root@solaris11-2:~# zfs list -r away_backup
NAME                      USED  AVAIL  REFER  MOUNTPOINT
away_backup               311K  3.91G    32K  /away_backup
away_backup/saved_backup  214K  3.91G   214K  /away_backup/saved_backup
root@solaris11-2:~# ls -l /away_backup/saved_backup/
total 399
-rw-r--r--   1 root     root        1436 Dec 13 03:30 aliases
-rw-r--r--   1 root     root         182 Dec 13 03:30 auto_home
-rw-r--r--   1 root     root         220 Dec 13 03:30 auto_master
-rw-r--r--   1 root     root        1931 Dec 13 03:30 dacf.conf
-r--r--r--   1 root     root         516 Dec 13 03:30 datemsk
-rw-r--r--   1 root     root        2670 Dec 13 03:30 devlink.tab
-rw-r--r--   1 root     root       38237 Dec 13 03:30 driver_aliases
-rw-r--r--   1 root     root         557 Dec 13 03:30 driver_classes
-rwxr--r--   1 root     root        1661 Dec 13 03:30 dscfg_format
……..
(truncated output)

```

According to this output, the remote backup, using the `zfs send` and `zfs recv` commands, has worked as expected. The restore operation is similar, so let's destroy every file from the `backuptest_pool/zfs1` filesystem in the first system (`solaris11-1`) as well as its snapshot by running the following commands:

```
root@solaris11-1:~# cd /backuptest_pool/zfs1/
root@solaris11-1:/backuptest_pool/zfs1# rm *
root@solaris11-1:/backuptest_pool/zfs1# cd
root@solaris11-1:~# zfs destroy backuptest_pool/zfs1@backup1
root@solaris11-1:~# zfs list -r backuptest_pool/zfs1
NAME                  USED  AVAIL  REFER  MOUNTPOINT
backuptest_pool/zfs1   31K  3.91G    31K  /backuptest_pool/zfs1
root@solaris11-1:~# 
```

From the second machine (`solaris11-2`), the restore procedure can be executed by running the following commands:

```
root@solaris11-2:~# zpool set listsnapshots=on away_backup
root@solaris11-2:~# zfs list -r away_backup
NAME                              USED  AVAIL  REFER  MOUNTPOINT
away_backup                       312K  3.91G    32K  /away_backup
away_backup/saved_backup          214K  3.91G   214K  /away_backup/saved_backup
away_backup/saved_backup@backup1     0      -   214K  -
```

The restore operation is similar to what we did during the backup, but we have to change the direction of the command where the `solaris11-1` system is the target and `solaris11-2` is the source now:

```
root@solaris11-2:~# zfs send -Rv away_backup/saved_backup@backup1 | ssh solaris11-1 zfs recv -F backuptest_pool/zfs1
sending from @ to away_backup/saved_backup@backup1
Password:
root@solaris11-2:~#
```

You can see that we used the `ssh` command to make a secure transmission between the systems. Again, we could have used another tool such as `netcat` and the methodology would have done the same thing.

Returning to the `solaris11-1` system, verify that all data was recovered by running the following command:

```
root@solaris11-1:~# zfs list -r backuptest_pool/zfs1
NAME                          USED  AVAIL  REFER  MOUNTPOINT
backuptest_pool/zfs1          214K  3.91G   214K  /backuptest_pool/zfs1
backuptest_pool/zfs1@backup1     0      -   214K  -
root@solaris11-1:~# cd /backuptest_pool/zfs1/
root@solaris11-1:/backuptest_pool/zfs1# ls -al
total 407
drwxr-xr-x   2 root     root          64 Dec 13 03:30 .
drwxr-xr-x   3 root     root           3 Dec 13 05:12 ..
-rw-r--r--   1 root     root        1436 Dec 13 03:30 aliases
-rw-r--r--   1 root     root         182 Dec 13 03:30 auto_home
-rw-r--r--   1 root     root         220 Dec 13 03:30 auto_master
-rw-r--r--   1 root     root        1931 Dec 13 03:30 dacf.conf
………
(truncated output)

```

ZFS is amazing. The backup and restore operations are simple to execute, and everything has worked so well. The removed files are back.

### An overview of the recipe

On ZFS, the restore and backup operations are done through two commands: `zfs send` and `zfs recv`. Both operations are based on snapshots, and they make it possible to save data on the same machine or on another machine. During the explanation, we also learned about the snapshot rollback procedure.

# Handling logs and caches

ZFS has some very interesting internal structures that can greatly improve the performance of the pool and filesystem. One of them is **ZFS intent log** (**ZIL**), which was created to get more intensive and sequential write request performance, making more **Input/Output** **Operations Per Second** (**IOPS**) possible and saving any transaction record in the memory until transaction groups (known as TXG) are flushed to the disk or a request is received. When using ZIL, all of the write operations are done on ZIL, and afterwards, they are committed to the filesystem, helping prevent any data loss.

Usually, the ZIL space is allocated from the main storage pool, but this could fragment data. Oracle Solaris 11 allows us to decide where ZIL will be held. Most implementations put ZIL on a dedicated disk or, even better, on a mirrored configuration using SSD disks or flash memory devices, being appropriated to highlight that log devices for ZIL shouldn't be confused with database logfiles' disks. Usually, ZIL device logs don't have a size bigger than half of the RAM size, but other aspects must be considered to provide a consistent guideline when making its sizing.

Another very popular structure of ZFS is the **Adaptive Replacement Cache** (**ARC**), which increases to occupy almost all free memory (RAM minus 1 GB) of Oracle Solaris 11, but without pushing the application data out of memory. A very positive aspect of ARC is that it improves the reading performance a lot, because if data can be found in the memory (ARC), there isn't a necessity of taking any information from disks.

Beyond ARC, there's another type of cache named L2ARC, which is similar to a cache level 2 between the main memory and the disk. L2ARC complements ARC, and using SSD disks is suitable for this type of cache, given that one of the more productive scenarios is when L2ARC is deployed as an accelerator for random reads. Here's a very important fact to be remembered—L2ARC writes data to the cache devices (SSD disks) in an asynchronous way, so L2ARC is not recommended for intensive (sequential) writes.

## Getting ready

This recipe is going to use a virtual machine (from VirtualBox or VMware) with 4 GB of memory, Oracle Solaris 11 (installed), and at least eight 4 GB disks.

## How to do it…

There are two methods to configure a log object in a pool—either the pool is created with log devices (at the same time) or log devices are added after the pool's creation. The latter method is used more often, so the following procedure takes this approach:

```
root@solaris11-1:~# zpool create raid1_pool mirror c8t3d0 c8t4d0

```

In the next command, we'll add a log in the mirror mode, which is very appropriate to prevent a single point of failure. So, execute the following command:

```
root@solaris11-1:~# zpool add raid1_pool log mirror c8t5d0 c8t6d0
root@solaris11-1:~# zpool status raid1_pool
  pool: raid1_pool
 state: ONLINE
  scan: none requested
config:

  NAME        STATE     READ WRITE CKSUM
  raid1_pool  ONLINE       0     0     0
    mirror-0  ONLINE       0     0     0
      c8t3d0  ONLINE       0     0     0
      c8t4d0  ONLINE       0     0     0
  logs
    mirror-1  ONLINE       0     0     0
      c8t5d0  ONLINE       0     0     0
      c8t6d0  ONLINE       0     0     0

errors: No known data errors
```

Perfect! The mirrored log was added as expected. It's appropriate to explain about the `mirror-0` and `mirror-1` objects from `zpool status`. Both objects are virtual devices. When a pool is created, the disks that were chosen are organized under a structure named virtual devices (`vdev`), and then, this `vdev` object is presented to the pool. In a rough way, a pool is composed of virtual devices, and each virtual device is composed of disks, slices, files, or any volume presented by other software or storage. Virtual devices are generated when the `stripe`, `mirror`, and `raidz` pools are created. Additionally, they are also created when a log and cache are inserted into the pool.

If a disk log removal is necessary, execute the following command:

```
root@solaris11-1:~# zpool detach raid1_pool c8t6d0
root@solaris11-1:~# zpool status raid1_pool
  pool: raid1_pool
 state: ONLINE
  scan: none requested
config:

    NAME        STATE     READ WRITE CKSUM
  raid1_pool    ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
        c8t3d0  ONLINE       0     0     0
        c8t4d0  ONLINE       0     0     0
    logs
      c8t5d0    ONLINE       0     0     0

errors: No known data errors
```

It would be possible to remove both log disks at once by specifying `mirror-1` (the virtual device), which represents the logs:

```
root@solaris11-1:~# zpool remove raid1_pool mirror-1
root@solaris11-1:~# zpool status raid1_pool
  pool: raid1_pool
 state: ONLINE
  scan: none requested
config:

    NAME        STATE     READ WRITE CKSUM
  raid1_pool    ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
        c8t3d0  ONLINE       0     0     0
        c8t4d0  ONLINE       0     0     0

errors: No known data errors
root@solaris11-1:~#
```

As we explained at the beginning of this procedure, it's usual to add logs after a pool has been created, but it would be possible and easy to create a pool and, at the same time, include the log devices during the creation process by executing the following command:

```
root@solaris11-1:~# zpool create mir_pool mirror c8t3d0 c8t4d0 log mirror c8t5d0 c8t6d0
root@solaris11-1:~# zpool status mir_pool
  pool: mir_pool
 state: ONLINE
  scan: none requested
config:

  NAME        STATE     READ WRITE CKSUM
  mir_pool    ONLINE       0     0     0
    mirror-0  ONLINE       0     0     0
      c8t3d0  ONLINE       0     0     0
      c8t4d0  ONLINE       0     0     0
  logs
    mirror-1  ONLINE       0     0     0
      c8t5d0  ONLINE       0     0     0
      c8t6d0  ONLINE       0     0     0

errors: No known data errors
root@solaris11-1:~#
```

According to the explanation about the L2ARC cache at the beginning of the recipe, it's also possible to add a cache object (L2ARC) into the ZFS pool using a syntax very similar to the one used when adding log objects by running the following command:

```
root@solaris11-1:~# zpool create mircache_pool mirror c8t3d0 c8t4d0 cache c8t5d0 c8t6d0
root@solaris11-1:~# zpool status mircache_pool
  pool: mircache_pool
 state: ONLINE
  scan: none requested
config:

  NAME           STATE     READ WRITE CKSUM
  mircache_pool  ONLINE       0     0     0
       mirror-0  ONLINE       0     0     0
         c8t3d0  ONLINE       0     0     0
         c8t4d0  ONLINE       0     0     0
  cache
       c8t5d0    ONLINE       0     0     0
       c8t6d0    ONLINE       0     0     0
errors: No known data errors
```

Similarly, like log devices, a pool could be created including cache devices in a single step:

```
root@solaris11-1:~# zpool create mircache_pool mirror c8t3d0 c8t4d0 cache c8t5d0 c8t6d0
root@solaris11-1:~# zpool status mircache_pool
  pool: mircache_pool
 state: ONLINE
  scan: none requested
config:

     NAME        STATE     READ WRITE CKSUM
  mircache_pool  ONLINE       0     0     0
       mirror-0  ONLINE       0     0     0
         c8t3d0  ONLINE       0     0     0
         c8t4d0  ONLINE       0     0     0
  cache
       c8t5d0    ONLINE       0     0     0
       c8t6d0    ONLINE       0     0     0

errors: No known data errors
```

It worked as expected! However, it's necessary to note that cache objects can't be mirrored as we did when adding log devices, and they can't be part of a RAID-Z configuration.

Removing a cache device from a pool is done by executing the following command:

```
root@solaris11-1:~# zpool remove mircache_pool c8t5d0

```

A final and important warning—every time `cache` objects are added into a pool, wait until the data comes into cache (the warm-up phase). It usually takes around 2 hours.

### An overview of the recipe

ARC, L2ARC, and ZIL are common structures in ZFS administration, and we learned how to create and remove both logs and cache from the ZFS pool. There are very interesting procedures and recommendations about performance and tuning that includes these objects, but it's out of the scope of this book.

# Managing devices in storage pools

Manipulating and managing devices are common tasks when working with a ZFS storage pool, and more maintenance activities involve adding, deleting, attaching, and detaching disks. According to Oracle, ZFS supports raid0 (`stripe`), raid1 (`mirror`), raidz (similar to raid5, with one parity disk), raidz2 (similar to raid6, but uses two parity disks), and raidz3 (three parity disks), and additionally, there could be a combination such as raid 0+1 or raid 1+0.

## Getting ready

This recipe is going to use a virtual machine (from VirtualBox or VMware) with 4 GB of memory, a running Oracle Solaris 11 installation, and at least eight 4 GB disks.

## How to do it…

According to the previous recipes, the structure of a mirrored pool is `pool` | `vdev` | `disks`, and the next command shouldn't be new to us:

```
root@solaris11-1:~# zpool create mir_pool2 mirror c8t3d0 c8t4d0
root@solaris11-1:~# zpool status mir_pool2
  pool: mir_pool2
 state: ONLINE
  scan: none requested
config:

  NAME        STATE     READ WRITE CKSUM
  mir_pool2   ONLINE       0     0     0
    mirror-0  ONLINE       0     0     0
      c8t3d0  ONLINE       0     0     0
      c8t4d0  ONLINE       0     0     0

errors: No known data errors
```

Eventually, in a critical environment, it could be necessary to increase the size of the pool, given that there are some ways to accomplish it. However, not all of them are correct, because this procedure must be done with care to keep the redundancy. For example, the next command fails to increase the redundancy because only one disk is added, and in this case, we would have two vdevs, the first being `vdev` (`mirror-0`) with two disks concatenated and a second `vdev` that doesn't have any redundancy. If the second `vdev` fails, the entire pool is lost. Oracle Solaris notifies us about the problem when we try this wrong configuration:

```
root@solaris11-1:~# zpool add mir_pool2 c8t5d0
vdev verification failed: use -f to override the following errors:
mismatched replication level: pool uses mirror and new vdev is disk
Unable to build pool from specified devices: invalid vdev configuration
```

If we wanted to proceed even with this notification, it would be enough to add the `-f` option, but this isn't recommended.

The second example is very similar to the first one, and we tried to add two disks instead of only one:

```
root@solaris11-1:~# zpool add mir_pool2 c8t5d0 c8t6d0
vdev verification failed: use -f to override the following errors:
mismatched replication level: pool uses mirror and new vdev is disk
Unable to build pool from specified devices: invalid vdev configuration
```

Again, the error remains because we added two disks, but we haven't mirrored them. In this case, the explanation is the same, and we would have a single point of failure if we tried to proceed.

Therefore, the correct method to expand the pool and keep the tolerance against failure is by executing the following command:

```
root@solaris11-1:~# zpool add mir_pool2 mirror c8t5d0 c8t6d0
root@solaris11-1:~# zpool status mir_pool2
  pool: mir_pool2
 state: ONLINE
  scan: none requested
config:

     NAME        STATE     READ WRITE CKSUM
    mir_pool2   ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
        c8t3d0  ONLINE       0     0     0
        c8t4d0  ONLINE       0     0     0
      mirror-1  ONLINE       0     0     0
        c8t5d0  ONLINE       0     0     0
        c8t6d0  ONLINE       0     0     0

errors: No known data errors
```

It worked! The final configuration is one that is similar to RAID 1+0, where there are two mirrored vdevs and all the data is spread over them. In this case, if the pool has a failure disk in any vdevs, data information is preserved. Furthermore, there are two vdevs in the pool: `mirror-0` and `mirror-1`. If we wished to remove a single disk from a mirror, it could be done by executing the following command:

```
root@solaris11-1:~# zpool detach mir_pool3 c8t6d0

```

If the plan is to remove the whole mirror (`vdev`), execute the following command:

```
root@solaris11-1:~# zpool remove mir_pool3 mirror-1

```

All deletions were done successfully.

A mirrored pool with two disks is fine and is used very often, but some companies require a more resilient configuration with three disks. To use a more realistic case, let's create a mirrored pool with two disks, create a filesystem inside it, copy some aleatory data into this filesystem (the reader can choose any data), and finally, add a third disk. Perform the following commands:

```
root@solaris11-1:~# zpool create mir_pool3  mirror c8t8d0 c8t9d0
root@solaris11-1:~# zfs create mir_pool3/zfs1
root@solaris11-1:~# cp -r mhvtl-* DTraceToolkit-0.99* dtbook_scripts* john* /mir_pool3/zfs1/

```

Again, in the preceding command, we could have copied any data. Finally, the command that executes our task is as follows:

```
root@solaris11-1:~# zpool attach mir_pool3 c8t9d0 c8t10d0

```

In the preceding command, we attached a new disk (`c8t10d0`) to a mirrored pool and specified where the current data would be copied from (`c8t9d0`). After resilvering (resynchronization), the pool organization is as follows:

```
root@solaris11-1:~# zpool status mir_pool3
  pool: mir_pool3
 state: ONLINE
  scan: resilvered 70.7M in 0h0m with 0 errors on Sat Dec 14 02:49:08 2013
config:

    NAME         STATE     READ WRITE CKSUM
   mir_pool3    ONLINE       0     0     0
     mirror-0   ONLINE       0     0     0
       c8t8d0   ONLINE       0     0     0
       c8t9d0   ONLINE       0     0     0
       c8t10d0  ONLINE       0     0     0

errors: No known data errors
```

Now, the `mir_pool3` pool is a three-way mirror pool, and all data is resilvered (resynchronized).

Some maintenance procedures require that we disable a disk to prevent any reading or writing operation on this device. Thus, when this disk is put to the `offline` state, it remains `offline` even after a reboot. Considering our existing three-way mirrored pool, the last device can be put in `offline`:

```
root@solaris11-1:~# zpool offline mir_pool3 c8t10d0
root@solaris11-1:~# zpool status mir_pool3
  pool: mir_pool3
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
  Sufficient replicas exist for the pool to continue functioning in a
  degraded state.
action: Online the device using 'zpool online' or replace the device with 'zpool replace'.
  scan: resilvered 70.7M in 0h0m with 0 errors on Sat Dec 14 02:49:08 2013
config:

   NAME         STATE     READ WRITE CKSUM
  mir_pool3    DEGRADED     0     0     0
    mirror-0   DEGRADED     0     0     0
      c8t8d0   ONLINE       0     0     0
      c8t9d0   ONLINE       0     0     0
      c8t10d0  OFFLINE      0     0     0

errors: No known data errors
```

There are some interesting findings—the `c8t10d0` disk is `OFFLINE`, `vdev` (`mirror-0`) is in the `DEGRADED` state, and the `mir_pool3` pool is in the `DEGRADED` state too.

The opposite operation to change the status of a disk to `ONLINE` is very easy, and while the pool is being resilvered, its status will be `DEGRADED`:

```
root@solaris11-1:~# zpool online mir_pool3 c8t10d0
warning: device 'c8t10d0' onlined, but remains in degraded state
root@solaris11-1:~# zpool status mir_pool3
  pool: mir_pool3
 state: ONLINE
  scan: resilvered 18K in 0h0m with 0 errors on Sat Dec 14 04:50:03 2013
config:
(truncated output)

```

One of the most useful and interesting tasks when managing pools is disk replacement, which only happens when there are pools using one of the following configurations: `raid1`, `raidz`, `raidz2`, or `raid3`. Why? Because a disk replacement couldn't compromise the data availability, and only these configurations can ensure this premise.

Two kinds of replacement exist:

*   Replacement of a failed device by another in the same slot
*   Replacement of a failed device by another from another slot

Both methods are straight and easy to execute. For example, we're using VirtualBox in this example, and to simulate the first case, we're going to power off Oracle Solaris 11 (`solaris11-1`), remove the disk that will be replaced (`c8t10d0`), create a new one in the same slot, and power on the virtual machine again (`solaris11-1`).

Before performing all these steps, we'll copy more data (here, it can be any data of your choice) to the `zfs1` filesystem inside the `mir_pool3` pool:

```
root@solaris11-1:~# cp -r /root/SFHA601/ /mir_pool3/zfs1/
root@solaris11-1:~# zpool list mir_pool3
NAME        SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
mir_pool3  3.97G  2.09G  1.88G  52%  1.00x  ONLINE  -
root@solaris11-1:~# shutdown –y –g0

```

On the VirtualBox Manager, click on the virtual machine with `solaris11-1`, go to **Settings**, and then go to **Storage**. Once there, remove the disks from slot 10 and create another disk at the same place (slot 10). After the physical replacement is done, power on the virtual machine (`solaris11-1`) again. After the login, open a terminal and execute the following command:

```
root@solaris11-1:~# zpool status mir_pool3
  pool: mir_pool3
 state: DEGRADED
status: One or more devices are unavailable in response to persistent errors.
  Sufficient replicas exist for the pool to continue functioning in a
  degraded state.
action: Determine if the device needs to be replaced, and clear the errors
  using 'zpool clear' or 'fmadm repaired', or replace the device
  with 'zpool replace'.
  Run 'zpool status -v' to see device specific details.
  scan: resilvered 18K in 0h0m with 0 errors on Sat Dec 14 04:50:03 2013
config:

   NAME         STATE     READ WRITE CKSUM
   mir_pool3    DEGRADED     0     0     0
     mirror-0   DEGRADED     0     0     0
       c8t8d0   ONLINE       0     0     0
       c8t9d0   ONLINE       0     0     0
       c8t10d0  UNAVAIL      0     0     0

errors: No known data errors
root@solaris11-1:~#
```

As the `c8t10d0` device was exchanged for a new one, the `zpool status mir_pool3` command shows that it's unavailable (`UNAVAIL`). This is the expected status. According to the previous explanation, the idea is that the failed disk is exchanged for another one in the same slot. Execute the following commands:

```
root@solaris11-1:~# zpool replace mir_pool3 c8t10d0
root@solaris11-1:~# zpool status mir_pool3 
  pool: mir_pool3
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
  scan: resilver in progress since Sat Dec 14 05:56:15 2013
    139M scanned out of 2.09G at 3.98M/s, 0h8m to go
    136M resilvered, 6.51% done
config:

   NAME               STATE     READ WRITE CKSUM
   mir_pool3          DEGRADED     0     0     0
     mirror-0         DEGRADED     0     0     0
       c8t8d0         ONLINE       0     0     0
       c8t9d0         ONLINE       0     0     0
       replacing-2    DEGRADED     0     0     0
         c8t10d0/old  UNAVAIL      0     0     0
         c8t10d0      DEGRADED     0     0     0  (resilvering)

errors: No known data errors
root@solaris11-1:~#
```

The `c8t10d0` disk was replaced and is being resilvered now. This time, we need to wait for the resilvering to complete.

If we're executing the replacement for a disk from another slot, the procedure is easier. For example, in the following steps, we're replacing the `c8t9d0` disk with `c8t3d0` by executing the following steps:

```
root@solaris11-1:~# zpool replace mir_pool3 c8t9d0 c8t3d0
root@solaris11-1:~# zpool status mir_pool3
  pool: mir_pool3
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
  continue to function in a degraded state.
    576M scanned out of 2.09G at 4.36M/s, 0h5m to go
    572M resilvered, 26.92% done
config:

   NAME             STATE     READ WRITE CKSUM
   mir_pool3        DEGRADED     0     0     0
     mirror-0       DEGRADED     0     0     0
       c8t8d0       ONLINE       0     0     0
       replacing-1  DEGRADED     0     0     0
         c8t9d0     ONLINE       0     0     0
         c8t3d0     DEGRADED     0     0     0  (resilvering)
       c8t10d0      ONLINE       0     0     0
```

Again, after the resync process is over, everything will be okay.

### An overview of the recipe

Managing disks is the most important task when working with ZFS. In this section, we learned how to add, remove, attach, detach, and replace a disk. All these processes will take a long time on a normal daily basis.

# Configuring spare disks

In a big company environment, there are a hundred disks working 24/7, and literally, it's impossible to know when a disk will fail. Imagine lots of disks failing during the day and how much time the replacement operations would take. This pictured context is useful to show the importance of spare disks. When deploying spare disks in a pool in a system, if any disk fails, the spare disk will take its place automatically, and data availability won't be impacted.

In the ZFS framework, spare disks are configured per storage pool, and after the appropriate configuration, even when a disk fails, nothing is necessary. The ZFS makes the entire replacement job automatic.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) that runs Oracle Solaris 11 with 4 GB RAM and at least eight disks of 4 GB each.

## How to do it…

A real situation using spare disks is where there's a mirrored pool, so to simulate this scenario, let's execute the following command:

```
root@solaris11-1:~# zpool create mir_pool4 mirror c8t3d0 c8t4d0

```

Adding spare disks in this pool is done by executing the following commands:

```
root@solaris11-1:~# zpool add mir_pool4 spare c8t5d0 c8t6d0
root@solaris11-1:~# zpool status mir_pool4
  pool: mir_pool4
 state: ONLINE
  scan: none requested
config:

   NAME        STATE     READ WRITE CKSUM
   mir_pool4   ONLINE       0     0     0
     mirror-0  ONLINE       0     0     0
       c8t3d0  ONLINE       0     0     0
       c8t4d0  ONLINE       0     0     0
   spares
     c8t5d0    AVAIL   
     c8t6d0    AVAIL   
```

As we mentioned earlier, spare disks will be used only when something wrong happens to the disks. To test the environment with spare disks, a good practice is shutting down Oracle Solaris 11 (`shutdown –y –g0`), removing the `c8t3d0` disk (SCSI slot 3) from the virtual machine's configuration, and turning on the virtual machine again. The status of `mir_pool4` presented by Oracle Solaris 11 is as follows:

```
root@solaris11-1:~# zpool status mir_pool4
  pool: mir_pool4
 state: DEGRADED
status: One or more devices are unavailable in response to persistent errors.
  Sufficient replicas exist for the pool to continue functioning in a
  degraded state.
action: Determine if the device needs to be replaced, and clear the errors
  using 'zpool clear' or 'fmadm repaired', or replace the device
  with 'zpool replace'.
  Run 'zpool status -v' to see device specific details.
  scan: resilvered 94K in 0h0m with 0 errors on Sat Dec 14 18:00:26 2013
config:

   NAME          STATE     READ WRITE CKSUM
   mir_pool4     DEGRADED     0     0     0
     mirror-0    DEGRADED     0     0     0
       spare-0   DEGRADED     0     0     0
         c8t3d0  UNAVAIL      0     0     0
         c8t5d0  ONLINE       0     0     0
       c8t4d0    ONLINE       0     0     0
   spares
     c8t5d0      INUSE   
     c8t6d0      AVAIL   

errors: No known data errors
```

Perfect! The disk that was removed is being shown as unavailable (`UNAVAIL`), and the `c8t5d0` spare disk has taken its place (`INUSE`). The pool is shown as `DEGRADED` to notify the administrator that a main disk is facing problems.

Finally, let's return to the configuration—power off the virtual machine, reinsert the removed disk again to the same SCSI slot 3, and power on the virtual machine. After completing all the steps, run the following command:

```
root@solaris11-1:~# zpool status mir_pool4
  pool: mir_pool4
 state: ONLINE
  scan: resilvered 27K in 0h0m with 0 errors on Sat Dec 14 16:49:29 2013
config:

   NAME          STATE     READ WRITE CKSUM
   mir_pool4     ONLINE       0     0     0
     mirror-0    ONLINE       0     0     0
       spare-0   ONLINE       0     0     0
         c8t3d0  ONLINE       0     0     0
         c8t5d0  ONLINE       0     0     0
       c8t4d0    ONLINE       0     0     0
   spares
     c8t5d0      INUSE   
     c8t6d0      AVAIL   

errors: No known data errors
```

According to the output, the `c8d5d0` spare disk continues to show its status as `INUSE` even when the `c8t3d0` disk is online again. To signal to the spare disk that `c8t3d0` is online again before Oracle Solaris updates it, execute the following commands:

```
root@solaris11-1:~# zpool online mir_pool4 c8t3d0
root@solaris11-1:~# zpool status mir_pool4
  pool: mir_pool4
 state: ONLINE
  scan: resilvered 27K in 0h0m with 0 errors on Sat Dec 14 16:49:29 2013
config:

   NAME        STATE     READ WRITE CKSUM
   mir_pool4   ONLINE       0     0     0
     mirror-0  ONLINE       0     0     0
       c8t3d0  ONLINE       0     0     0
       c8t4d0  ONLINE       0     0     0
   spares
     c8t5d0    AVAIL   
     c8t6d0    AVAIL   

errors: No known data errors
```

ZFS is amazing. Initially, the `c8t3d0` disk has come online again, but the `c8t5d0` spare disk was still in use (`INUSE`). Afterwards, we ran the `zpool online mir_pool4 c8t3d0` command to confirm the online status of `c8t3d0`, and the spare disk (`c8t5d0`) became available and started acting as a spare disk.

Finally, remove the spare disk by executing the following command:

```
root@solaris11-1:~# zpool remove mir_pool4 c8t5d0
root@solaris11-1:~# zpool status mir_pool4
  pool: mir_pool4
 state: ONLINE
  scan: resilvered 27K in 0h0m with 0 errors on Sat Dec 14 16:49:29 2013
config:

   NAME        STATE     READ WRITE CKSUM
   mir_pool4   ONLINE       0     0     0
     mirror-0  ONLINE       0     0     0
       c8t3d0  ONLINE       0     0     0
       c8t4d0  ONLINE       0     0     0
   spares
     c8t6d0    AVAIL   
```

### An overview of the recipe

In this section, you saw how to configure spare disks, and some experiments were done to explain its exact working.

# Handling ZFS snapshots and clones

ZFS snapshot is a complex theme that can have its functionality extended using the hold and release operations. Additionally, other tasks such as renaming snapshots, promoting clones, and executing differential snapshots are crucial in daily administration. All these points will be covered in this recipe.

## Getting ready

This recipe can be followed using a virtual machine (VirtualBox or VMware) with 4 GB RAM, a running Oracle Solaris 11 application, and at least eight disks with 4 GB each.

## How to do it…

From what we learned in the previous recipes, let's create a pool and a filesystem, and populate this filesystem with any data (readers can copy any data into this filesystem) and two snapshots by executing the following commands:

```
root@solaris11-1:~# zpool create simple_pool_1 c8t3d0
root@solaris11-1:~# zfs create simple_pool_1/zfs1
root@solaris11-1:~# cp -r /root/mhvtl-* /root/john* /simple_pool_1/zfs1 
root@solaris11-1:~# zpool list simple_pool_1
NAME            SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
simple_pool_1  3.97G  63.1M  3.91G   1%  1.00x  ONLINE  -

root@solaris11-1:~# zfs snapshot simple_pool_1/zfs1@today
root@solaris11-1:~# zfs snapshot simple_pool_1/zfs1@today_2
root@solaris11-1:~# zpool set listsnapshots=on simple_pool_1
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                         USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1               63.2M  3.85G    32K  /simple_pool_1
simple_pool_1/zfs1          63.1M  3.85G  63.1M  /simple_pool_1/zfs1
simple_pool_1/zfs1@today        0      -  63.1M  -
simple_pool_1/zfs1@today_2      0      -  63.1M  -
```

Deleting a snapshot is easy as we already saw it previously in the chapter, and if it's necessary, it can be done by executing the following command:

```
root@solaris11-1:~# zfs destroy simple_pool_1/zfs1@today_2

```

Like the operation of removing a snapshot, renaming it is done by running the following command:

```
root@solaris11-1:~# zfs rename simple_pool_1/zfs1@today simple_pool_1/zfs1@today_2

```

Both actions (renaming and destroying) are common operations that are done when handling snapshots. Nonetheless, the big question that comes up is whether it would be possible to prevent a snapshot from being deleted. This is where a new snapshot operation named `hold` can help us. When a snapshot is put in `hold` status, it can't be removed. This behavior can be configured by running the following command:

```
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                         USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1               63.1M  3.85G    32K  /simple_pool_1
simple_pool_1/zfs1          63.1M  3.85G  63.1M  /simple_pool_1/zfs1
simple_pool_1/zfs1@today_2      0      -  63.1M  -
root@solaris11-1:~# zfs hold keep simple_pool_1/zfs1@today_2

```

To list the snapshots on hold, execute the following commands:

```
root@solaris11-1:~# zfs holds simple_pool_1/zfs1@today_2
NAME                        TAG   TIMESTAMP                 
simple_pool_1/zfs1@today_2  keep  Sat Dec 14 21:51:26 2013  
root@solaris11-1:~# zfs destroy simple_pool_1/zfs1@today_2
cannot destroy 'simple_pool_1/zfs1@today_2': snapshot is busy
root@solaris11-1:~#
```

Through the `zfs hold keep` command, the snapshot was left in suspension, and afterwards, we tried to remove it without success because of the hold. If there were other descendants from the `simple_pool/zfs1` filesystem, it would be possible to hold all of them by executing the following command:

```
root@solaris11-1:~# zfs hold –r keep simple_pool_1/zfs1@today_2

```

An important detail must be reinforced here—a snapshot can only be destroyed when it's released, and there's a property named `userrefs` that tells whether the snapshot is being held or not. Using this information, the releasing and destruction operations can be executed in a row by running the following command:

```
root@solaris11-1:~# zfs get userrefs simple_pool_1/zfs1@today_2
NAME                        PROPERTY  VALUE  SOURCE
simple_pool_1/zfs1@today_2  userrefs  1   
root@solaris11-1:~# zfs release keep simple_pool_1/zfs1@today_2
root@solaris11-1:~# zfs get userrefs simple_pool_1/zfs1@today_2
NAME                        PROPERTY  VALUE  SOURCE
simple_pool_1/zfs1@today_2  userrefs  0      -
root@solaris11-1:~# zfs destroy simple_pool_1/zfs1@today_2
root@solaris11-1:~# zfs list -r simple_pool_1             
NAME                 USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1       63.2M  3.85G    32K  /simple_pool_1
simple_pool_1/zfs1  63.1M  3.85G  63.1M  /simple_pool_1/zfs1
```

Going a little further, Oracle Solaris 11 allows us to determine what has changed in a filesystem when comparing two snapshots. To understand how it works, the first step is to take a new snapshot named `snap_1`. Afterwards, we have to alter the content of the `simple_pool/zfs1` filesystem to take a new snapshot (`snap_2`) and determine what has changed in the filesystem. The entire procedure is accomplished by executing the following commands:

```
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                 USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1       63.2M  3.85G    32K  /simple_pool_1
simple_pool_1/zfs1  63.1M  3.85G  63.1M  /simple_pool_1/zfs1
root@solaris11-1:~# zfs snapshot simple_pool_1/zfs1@snap1
root@solaris11-1:~# cp /etc/hosts /simple_pool_1/zfs1/
root@solaris11-1:~# zfs snapshot simple_pool_1/zfs1@snap2
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                       USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1             63.4M  3.84G    32K  /simple_pool_1
simple_pool_1/zfs1        63.1M  3.84G  63.1M  /simple_pool_1/zfs1
simple_pool_1/zfs1@snap1    32K      -  63.1M  -
simple_pool_1/zfs1@snap2      0      -  63.1M  -
```

The following command is the most important from this procedure because it takes the differential snapshot:

```
root@solaris11-1:~# zfs diff simple_pool_1/zfs1@snap1 simple_pool_1/zfs1@snap2
M  /simple_pool_1/zfs1/
+  /simple_pool_1/zfs1/hosts
root@solaris11-1:~#
```

The previous command has shown that the new file in `/simple_pool_1/zfs1` is the `hosts` file, and it was expected according to our previous setup. The `+` identifier indicates that a file or directory was added, the `-` identifier indicates that a file or directory was removed, the `M` identifier indicates that a file or directory was modified, and the `R` identifier indicates that a file or directory was renamed.

Now that we are reaching the end of this section, we should remember that earlier in this chapter, we reviewed how to make a clone from a snapshot, but not all operations were shown. The fact about clone is that it is possible to promote it to a normal filesystem and, eventually, remove the original filesystem (if necessary) because there isn't a clone as a descendant anymore. Let's verify the preceding sentence by running the following commands:

```
root@solaris11-1:~# zfs snapshot simple_pool_1/zfs1@snap3
root@solaris11-1:~# zfs clone simple_pool_1/zfs1@snap3 simple_pool_1/zfs1_clone1
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                        USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1              63.3M  3.84G    33K  /simple_pool_1
simple_pool_1/zfs1         63.1M  3.84G  63.1M  /simple_pool_1/zfs1
simple_pool_1/zfs1@snap1     32K      -  63.1M  -
simple_pool_1/zfs1@snap2       0      -  63.1M  -
simple_pool_1/zfs1@snap3       0      -  63.1M  -
simple_pool_1/zfs1_clone1    25K  3.84G  63.1M  /simple_pool_1/zfs1_clone1
```

Until this point, everything is okay. The next command shows us that `simple_pool_1/zfs1_clone` is indeed a clone:

```
root@solaris11-1:~# zfs get origin simple_pool_1/zfs1_clone1
NAME                       PROPERTY  VALUE                     SOURCE
simple_pool_1/zfs1_clone1  origin    simple_pool_1/zfs1@snap3  -
```

The next command promotes the existing clone to an independent filesystem:

```
root@solaris11-1:~# zfs promote simple_pool_1/zfs1_clone1
root@solaris11-1:~# zfs list -r simple_pool_1
NAME                              USED  AVAIL  REFER  MOUNTPOINT
simple_pool_1                    63.3M  3.84G    33K  /simple_pool_1
simple_pool_1/zfs1                   0  3.84G  63.1M  /simple_pool_1/zfs1
simple_pool_1/zfs1_clone1        63.1M  3.84G  63.1M  /simple_pool_1/zfs1_clone1
simple_pool_1/zfs1_clone1@snap1    32K      -  63.1M  -
simple_pool_1/zfs1_clone1@snap2      0      -  63.1M  -
simple_pool_1/zfs1_clone1@snap3      0      -  63.1M  -
root@solaris11-1:~# zfs get origin simple_pool_1/zfs1_clone1
NAME                       PROPERTY  VALUE  SOURCE
simple_pool_1/zfs1_clone1  origin    -      -
root@solaris11-1:~#
```

We're able to prove that `simple_pool_1/zfs1_clone1` is a new filesystem because the clone didn't require any space (size of `25K`), and the recently promoted clone to filesystem takes 63.1M now. Moreover, the `origin` property doesn't point to a snapshot object anymore.

### An overview of the recipe

This section has explained how to create, destroy, hold, and release a snapshot, as well as how to promote a clone to a real filesystem. Furthermore, you saw how to determine the difference between two snapshots.

# Playing with COMSTAR

**Common Protocol SCSI Target** (**COMSTAR**) is a framework that was introduced in Oracle Solaris 11; this makes it possible for Oracle Solaris 11 to access disks in another system that is running any operating system (Oracle Solaris, Oracle Enterprise Linux, and so on). This access happens through the network using protocols such as **iSCSI**, **Fibre** **Channel over Ethernet** (**FCoE**), or **Fibre Channel** (**FC**).

One big advantage of using COMSTAR is that Oracle Solaris 11 is able to reach the disks on another machine without using a HBA board (very expensive) for an FC channel access. There are also disadvantages such as the fact that dump devices don't support the iSCSI disks offered by COMSTAR and the network infrastructure can become overloaded.

## Getting ready

This section requires two virtual machines that run Oracle Solaris 11, both with 4 GB RAM and eight 4 GB disks. Additionally, both virtual machines must be in the same network and have access to each other.

## How to do it…

A good approach when configuring iSCSI is to have an initial plan, a well-defined list of disks that will be accessed using iSCSI, and to determine which system will be the initiator (`solaris11-2`) and the target (`solaris11-1`). Therefore, let's list the existing disks by executing the following command:

```
root@solaris11-1:~# format
AVAILABLE DISK SELECTIONS:
       0\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0
       1\. c8t1d0 <VBOX-HARDDISK-1.0-16.00GB>
          /pci@0,0/pci1000,8000@14/sd@1,0
       2\. c8t2d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@2,0
       3\. c8t3d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@3,0
       4\. c8t4d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@4,0
       5\. c8t5d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@5,0
       6\. c8t6d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@6,0
       7\. c8t8d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@8,0
       8\. c8t9d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@9,0
       9\. c8t10d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@a,0
      10\. c8t11d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@b,0
      11\. c8t12d0 <VBOX-HARDDISK-1.0 cyl 2045 alt 2 hd 128 sec 32>
          /pci@0,0/pci1000,8000@14/sd@c,0
   root@solaris11-1:~# zpool status | grep d0
    c8t2d0  ONLINE       0     0     0
    c8t1d0  ONLINE       0     0     0
    c8t0d0  ONLINE       0     0     0
```

According to the previous two commands, the `c8t3d0` and `c8t12d0` disks are available for use. Nevertheless, unfortunately, the COMSTAR software isn't installed in Oracle Solaris 11 by default; we have to install it to use the iSCSI protocol on the `solaris11-1` system. Consequently, using the IPS framework that was configured in [Chapter 1](part0015_split_000.html#page "Chapter 1. IPS and Boot Environments"), *IPS and Boot Environments*, we can confirm whether the appropriate package is or isn't installed on the system by running the following command:

```
root@solaris11-1:~# pkg search storage-server
INDEX       ACTION VALUE                                PACKAGE
incorporate depend pkg:/storage-server@0.1,5.11-0.133   pkg:/consolidation/osnet/osnet-incorporation@0.5.11-0.175.1.0.0.24.2
pkg.fmri    set    solaris/storage-server               pkg:/storage-server@0.1-0.133
pkg.fmri    set    solaris/storage/storage-server       pkg:/storage/storage-server@0.1-0.173.0.0.0.1.0
pkg.fmri    set    solaris/group/feature/storage-server pkg:/group/feature/storage-server@0.5.11-0.175.1.0.0.24.2
root@solaris11-1:~# pkg install storage-server
root@solaris11-1:~# pkg list storage-server
NAME (PUBLISHER)                       VERSION                    IFO
group/feature/storage-server           0.5.11-0.175.1.0.0.24.2    i—
root@solaris11-1:~# pkg info storage-server

```

The iSCSI target feature was installed through a package named `storage-server`, but the feature is only enabled if the `stmf` service is also enabled. Therefore, let's enable the service by executing the following commands:

```
root@solaris11-1:~# svcs -a | grep stmf
disabled        09:11:13 svc:/system/stmf:default
root@solaris11-1:~# svcadm enable svc:/system/stmf:default
root@solaris11-1:~# svcs -a | grep stmf
online          09:14:19 svc:/system/stmf:default
```

At this point, the system is ready to be configured as an iSCSI target. Before proceeding, let's learn a new concept about ZFS.

ZFS has a nice feature named ZFS volumes that represent and work as block devices. ZFS volumes are identified as devices in `/dev/zvol/dsk/rdsk/pool/[volume_name]`. The other nice thing about ZFS volumes is that after they are created, the size of the volume is reserved in the pool.

It's necessary to create a ZFS volume and, afterwards, a **Logical Unit** (**LUN**) from this ZFS volume to use iSCSI in Oracle Solaris 11\. Eventually, less experienced administrators don't know that the LUN concept comes from the storage world (Oracle, EMC, and Hitachi). A storage box presents a volume (configured as raid0, raid1, raid5, and so on) to the operating system, and this volume is known as LUN, but from the operating system's point view, it's only a simple disk.

So, let's create a ZFS volume. The first step is to create a pool:

```
root@solaris11-1:~# zpool create mypool_iscsi c8t5d0

```

Now, it's time to create a volume (in this case, using a size of 2 GB) by running the following command:

```
root@solaris11-1:~# zfs create -V 2Gb mypool_iscsi/myvolume
root@solaris11-1:~# zfs list mypool_iscsi/myvolume
NAME                    USED  AVAIL  REFER  MOUNTPOINT
mypool_iscsi/myvolume  2.06G  3.91G    16K  -
```

Next, as a requirement to present the volume through the network using iSCSI, it's necessary to create LUN from the `mypool_iscsi/myvolume` volume:

```
root@solaris11-1:~# stmfadm create-lu /dev/zvol/rdsk/mypool_iscsi/myvolume
Logical unit created: 600144F0991C8E00000052ADD63B0001

root@solaris11-1:~# stmfadm list-lu
LU Name: 600144F0991C8E00000052ADD63B0001
```

Our main concern is to make the recently created LUN viewable from any host that needs to access it. So, let's configure the access that is available and permitted from all hosts by running the following command:

```
root@solaris11-1:~# stmfadm add-view 600144F0991C8E00000052ADD63B0001
root@solaris11-1:~# stmfadm list-view -l 600144F0991C8E00000052ADD63B0001
View Entry: 0
    Host group   : All
    Target Group : All
    LUN          : Auto
```

Currently, the iSCSI target service can be disabled; now, it must be checked and enabled if necessary:

```
root@solaris11-1:~# svcs -a | grep target
disabled       16:48:34 svc:/system/fcoe_target:default
disabled       16:48:34 svc:/system/ibsrp/target:default
disabled       14:30:51 svc:/network/iscsi/target:default
root@solaris11-1:~# svcadm enable svc:/network/iscsi/target:default
root@solaris11-1:~# svcs svc:/network/iscsi/target:default
STATE          STIME    FMRI
online         14:31:47 svc:/network/iscsi/target:default
```

It's important to realize the dependencies from this service by executing the following command:

```
root@solaris11-1:~# svcs -l svc:/network/iscsi/target:default
fmri         svc:/network/iscsi/target:default
name         iscsi target
enabled      true
state        online
next_state   none
state_time   Sun Dec 15 14:31:47 2013
logfile      /var/svc/log/network-iscsi-target:default.log
restarter    svc:/system/svc/restarter:default
manifest     /lib/svc/manifest/network/iscsi/iscsi-target.xml
dependency   require_any/error svc:/milestone/network (online)
dependency   require_all/none svc:/system/stmf:default (online)
```

Now that the iSCSI target service is enabled, let's create a new iSCSI target. Remember that to access the available disks through the network and using iSCSI, we have to create a target (something like an access port or an iSCSI server) to enable this access. Then, to create a target in the `solaris11-1` machine, execute the following command:

```
root@solaris11-1:~# itadm create-target
Target iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d successfully created
root@solaris11-1:~# itadm list-target -v
TARGET NAME                                                  STATE    SESSIONS 
iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d  online   0        
  alias:                -
  auth:                 none (defaults)
  targetchapuser:       -
  targetchapsecret:     unset
  tpg-tags:             default
```

The iSCSI target has some important default properties, and one of them determines whether an authentication scheme will be required or not. The following output confirms that authentication (`auth`) isn't enabled:

```
root@solaris11-1:~# itadm list-defaults
iSCSI Target Default Properties:
alias:           <none>
auth:            <none>
radiusserver:    <none>
radiussecret:    unset
isns:            disabled
isnsserver:      <none>
```

From here, we are handling two systems—`solaris11-1` (`192.168.1.106`), which was configured as the iSCSI target, and `solaris11-2` (`192.168.1.109`), which will be used as an initiator. By the way, we should remember that an iSCSI initiator is a kind of iSCS client that's necessary to access iSCSI disks offered by other systems.

To configure an initiator, the first task is to verify that the iSCSI initiator service and its dependencies are enabled by executing the following command:

```
root@solaris11-1:~# ssh solaris11-2
Password: 
Last login: Sun Dec 15 14:13:08 2013
Oracle Corporation      SunOS 5.11      11.1    September 2012
root@solaris11-2:~# svcs -a | grep initiator
online         12:10:22 svc:/system/fcoe_initiator:default
online         12:10:25 svc:/network/iscsi/initiator:default
root@solaris11-2:~# svcs -l svc:/network/iscsi/initiator:default
fmri         svc:/network/iscsi/initiator:default
name         iSCSI initiator daemon
enabled      true
state        online
next_state   none
state_time   Sun Dec 15 12:10:25 2013
logfile      /var/svc/log/network-iscsi-initiator:default.log
restarter    svc:/system/svc/restarter:default
contract_id  89 
manifest     /lib/svc/manifest/network/iscsi/iscsi-initiator.xml
dependency   require_any/error svc:/milestone/network (online)
dependency   require_all/none svc:/network/service (online)
dependency   require_any/error svc:/network/loopback (online)
```

The configured initiator has some very interesting properties:

```
root@solaris11-2:~# iscsiadm list initiator-node 
Initiator node name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
Initiator node alias: solaris11
        Login Parameters (Default/Configured):
                Header Digest: NONE/-
                Data Digest: NONE/-
                Max Connections: 65535/-
        Authentication Type: NONE
        RADIUS Server: NONE
        RADIUS Access: disabled
        Tunable Parameters (Default/Configured):
                Session Login Response Time: 60/-
                Maximum Connection Retry Time: 180/-
                Login Retry Time Interval: 60/-
        Configured Sessions: 1
```

According to the preceding output, `Authentication Type` is configured to `NONE`; this is the same configuration for the target. For now, it's appropriate because both systems must have the same authentication scheme.

Before the iSCSI configuration procedure, there are three methods to find an iSCSI disk on another system: static, send target, and iSNS. However, while all of them certainly have a specific use for different scenarios, a complete explanation about these methods is out of scope. Therefore, we will choose the *send target* method that is a kind of automatic mechanism to find iSCSI disks in internal networks.

To verify the configured method and to enable the send targets methods, execute the following commands:

```
root@solaris11-2:~# iscsiadm list discovery
Discovery:
        Static: disabled
        Send Targets: disabled
        iSNS: disabled
root@solaris11-2:~# iscsiadm modify discovery --sendtargets enable 
root@solaris11-2:~# iscsiadm list discovery
Discovery:
        Static: disabled
        Send Targets: enabled
        iSNS: disabled
```

The `solaris11-1` system was configured as an iSCSI target, and we created a LUN in this system to be accessed by the network. On the `solaris11-2` system (iSCSI initiator), we have to register the iSCSI target system (`solaris11-1`) to discover which LUNs are available to be accessed. To accomplish these tasks, execute the following commands:

```
root@solaris11-2:~# iscsiadm add discovery-address 192.168.1.106
root@solaris11-2:~# iscsiadm list discovery-address
Discovery Address: 192.168.1.106:3260
root@solaris11-2:~# iscsiadm list target
Target: iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d
  Alias: -
  TPGT: 1
  ISID: 4000002a0000
  Connections: 1
```

The previous command shows the configured target on the `solaris11-1` system (first line of the output).

To confirm the successfully added target, iSCSI LUNs available from the iSCSI target (`solaris11-1`) are shown by the following command:

```
root@solaris11-2:~# devfsadm
root@solaris11-2:~# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c0t600144F0991C8E00000052ADD63B0001d0 <SUN-COMSTAR-1.0 cyl 1022 alt 2 hd 128 sec 32>
          /scsi_vhci/disk@g600144f0991c8e00000052add63b0001
       1\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0

(truncated output)

```

The iSCSI volume (presented as a disk for the iSCSI initiator) from the `solaris11-1` system was found, and it can be used normally as it is a local device. To test it, execute the following command:

```
root@solaris11-2:~# zpool create new_iscsi c0t600144F0991C8E00000052ADD63B0001d0
root@solaris11-2:~# zfs create new_iscsi/fs_iscsi
root@solaris11-2:~# zfs list -r new_iscsi
NAME                USED  AVAIL  REFER  MOUNTPOINT
new_iscsi           124K  1.95G    32K  /new_iscsi
new_iscsi/fs_iscsi   31K  1.95G    31K  /new_iscsi/fs_iscsi
root@solaris11-2:~# zpool status new_iscsi
  pool: new_iscsi
 state: ONLINE
  scan: none requested
config:

  NAME                                     STATE     READ WRITE CKSUM
  new_iscsi                                ONLINE       0     0     0
    c0t600144F0991C8E00000052ADD63B0001d0  ONLINE       0     0     0
```

Normally, this configuration (without authentication) is the configuration that we'll see in most companies, although it isn't recommended.

Some businesses require that all data communication be authenticated, requiring both the iSCSI target and initiator to be configured with an authentication scheme where a password is set on the iSCSi target (`solaris11-1`), forcing the same credential to be set on the iSCSI initiator (`solaris11-2`).

When managing authentication, it's possible to configure the iSCSI authentication scheme using the CHAP method (unidirectional or bidirectional) or even RADIUS. As an example, we're going to use CHAP unidirectional where the client (`solaris 11-2`, the iSCSI initiator) executes the login to the server (`solaris11-1`, the iSCSI target) to access the iSCSI target devices (LUNs or, at the end, ZFS volumes). However, if a bidirectional authentication was used, both the target and initiator should present a CHAP password to authenticate each other.

On the `solaris11-1` system, list the current target's configuration by executing the following command:

```
root@solaris11-1:~# itadm list-target
TARGET NAME                                                  STATE    SESSIONS 
iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d  online   1    
root@solaris11-1:~# itadm list-target iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d –v
TARGET NAME                                                  STATE    SESSIONS 
iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d  online   1        
  alias:                -
  auth:                 none (defaults)
  targetchapuser:       -
  targetchapsecret:     unset
  tpg-tags:             default
```

According to the output, currently, the authentication isn't configured to use the CHAP authentication. Therefore, it can be done by executing the following command:

```
root@solaris11-1:~# itadm modify-target -a chap iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d
Target iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d successfully modified
```

That's great, but there isn't any enabled password to make the authentication happen. Thus, we have to set a password (`packt1234567`) to complete the target configuration. By the way, the password is long because the CHAP password must have 12 characters at least:

```
root@solaris11-1:~# itadm modify-target -s iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d
Enter CHAP secret: packt1234567
Re-enter secret: packt1234567
Target iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d successfully modified
```

On the `solaris11-2` system, the CHAP authentication must be set up to make it possible for the initiator to log in to the target; now, execute the following command:

```
root@solaris11-2:~# iscsiadm list initiator-node
Initiator node name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
Initiator node alias: solaris11
        Login Parameters (Default/Configured):
                Header Digest: NONE/-
                Data Digest: NONE/-
                Max Connections: 65535/-
        Authentication Type: NONE
        RADIUS Server: NONE
        RADIUS Access: disabled
        Tunable Parameters (Default/Configured):
                Session Login Response Time: 60/-
                Maximum Connection Retry Time: 180/-
                Login Retry Time Interval: 60/-
        Configured Sessions: 1
```

On the `solaris11-2` system (initiator), we have to confirm that it continues using the iSCSI dynamic discovery (`sendtargets`):

```
root@solaris11-2:~# iscsiadm list discovery
Discovery:
  Static: disabled
  Send Targets: enabled
  iSNS: disabled
```

The same password from the target (`packt1234567`) must be set on the `solaris11-2` system (initiator). Moreover, the CHAP authentication also must be configured by running the following command:

```
root@solaris11-2:~# iscsiadm modify initiator-node --CHAP-secret
Enter secret: packt1234567
Re-enter secret: packt1234567
root@solaris11-2:~# iscsiadm modify initiator-node --authentication CHAP

```

Verifying the authentication configuration from the initiator node and available targets can be done using the following command:

```
root@solaris11-2:~# iscsiadm list initiator-node
Initiator node name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
Initiator node alias: solaris11
        Login Parameters (Default/Configured):
                Header Digest: NONE/-
                Data Digest: NONE/-
                Max Connections: 65535/-
        Authentication Type: CHAP
                CHAP Name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
        RADIUS Server: NONE
        RADIUS Access: disabled
        Tunable Parameters (Default/Configured):
                Session Login Response Time: 60/-
                Maximum Connection Retry Time: 180/-
                Login Retry Time Interval: 60/-
        Configured Sessions: 1

root@solaris11-2:~# iscsiadm list discovery-address
Discovery Address: 192.168.1.106:3260
root@solaris11-2:~# iscsiadm list target 
Target: iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d
        Alias: -
        TPGT: 1
        ISID: 4000002a0000
        Connections: 1
```

Finally, we have to update the device tree configuration using the `devfsadm` command to confirm that the target is available for the initiator (`solaris11-2`) access. If everything has gone well, the iSCSI disk will be visible using the `format` command:

```
root@solaris11-2:~# devfsadm
root@solaris11-2:~# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c0t600144F0991C8E00000052ADD63B0001d0 <SUN-COMSTAR-1.0-2.00GB>
          /scsi_vhci/disk@g600144f0991c8e00000052add63b0001
       1\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0
(truncated output)

```

As a simple example, the following commands create a pool and filesystem using the iSCSI disk that was discovered and configured in the previous steps:

```
root@solaris11-2:~# zpool create new_iscsi_chap c0t600144F0991C8E00000052ADD63B0001d0
root@solaris11-2:~# zfs create new_iscsi_chap/zfs1
root@solaris11-2:~# zfs list -r new_iscsi_chap 
NAME                 USED  AVAIL  REFER  MOUNTPOINT
new_iscsi_chap       124K  1.95G    32K  /new_iscsi_chap
new_iscsi_chap/zfs1   31K  1.95G    31K  /new_iscsi_chap/zfs1
root@solaris11-2:~# zpool list new_iscsi_chap
NAME             SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
new_iscsi_chap  1.98G   124K  1.98G   0%  1.00x  ONLINE  -
root@solaris11-2:~# zpool status new_iscsi_chap
  pool: new_iscsi_chap
 state: ONLINE
  scan: none requested
config:

  NAME                                   STATE     READ WRITE CKSUM
  new_iscsi_chap                         ONLINE       0     0     0
  c0t600144F0991C8E00000052ADD63B0001d0  ONLINE       0     0     0
```

Great! The iSCSI configuration with the CHAP authentication has worked smoothly. Now, to consolidate all the acquired knowledge, the following commands undo all the iSCSI configurations, first on the initiator (`solaris11-2`) and afterwards on the target (`solaris11-1`), as follows:

```
root@solaris11-2:~# zpool destroy new_iscsi_chap
root@solaris11-2:~# iscsiadm list initiator-node
Initiator node name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
Initiator node alias: solaris11
        Login Parameters (Default/Configured):
                Header Digest: NONE/-
                Data Digest: NONE/-
                Max Connections: 65535/-
        Authentication Type: CHAP
                CHAP Name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
        RADIUS Server: NONE
        RADIUS Access: disabled
        Tunable Parameters (Default/Configured):
                Session Login Response Time: 60/-
                Maximum Connection Retry Time: 180/-
                Login Retry Time Interval: 60/-
        Configured Sessions: 1

root@solaris11-2:~# iscsiadm remove discovery-address 192.168.1.106
root@solaris11-2:~# iscsiadm modify initiator-node --authentication none
root@solaris11-2:~# iscsiadm list initiator-node
Initiator node name: iqn.1986-03.com.sun:01:e00000000000.5250ac8e
Initiator node alias: solaris11
        Login Parameters (Default/Configured):
                Header Digest: NONE/-
                Data Digest: NONE/-
                Max Connections: 65535/-
        Authentication Type: NONE
        RADIUS Server: NONE
        RADIUS Access: disabled
        Tunable Parameters (Default/Configured):
                Session Login Response Time: 60/-
                Maximum Connection Retry Time: 180/-
                Login Retry Time Interval: 60/-
        Configured Sessions: 1
```

By updating the device tree (the `devfsadm` and `format` commands), we can see that the iSCSI disk has disappeared:

```
root@solaris11-2:~# devfsadm
root@solaris11-2:~# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0

(truncated output)

```

Now, the unconfiguring process must be done on the target (`solaris11-2`). First, list the existing LUNs:

```
root@solaris11-1:~# stmfadm list-lu
LU Name: 600144F0991C8E00000052ADD63B0001
```

Remove the existing LUN:

```
root@solaris11-1:~# stmfadm delete-lu 600144F0991C8E00000052ADD63B0001

```

List the currently configured targets:

```
root@solaris11-1:~# itadm list-target -v
TARGET NAME                                                  STATE    SESSIONS 
iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d  online   0        
  alias:                -
  auth:                 chap 
  targetchapuser:       -
  targetchapsecret:     set
  tpg-tags:             default
```

Delete the existing targets:

```
root@solaris11-1:~# itadm delete-target -f iqn.1986-03.com.sun:02:51d113f3-39a0-cead-e602-ea9aafdaad3d
root@solaris11-1:~# itadm list-target –v

```

Destroy the pool that contains the iSCSI disk:

```
root@solaris11-1:~# zpool destroy mypool_iscsi

```

Finally, we did it. There isn't an iSCSI configuration anymore.

A few months ago, I wrote a tutorial that explains how to configure a free VTL software that emulates a tape robot, and at the end of document, I explained how to connect to this VTL from Oracle Solaris 11 using the iSCSI protocol. It's very interesting to see a real case about how to use the iSCSI initiator to access an external application. Check the references at the end of this chapter to learn more about this VTL document.

### An overview of the recipe

In this section, you learned about all the iSCSI configurations using COMSTAR with and without the CHAP authentication. Moreover, the undo configuration steps were also provided.

# Mirroring the root pool

Nowadays, systems running very critical applications without a working mirrored boot disk is something unthinkable. However, when working with ZFS, the mirroring process of the boot disk is smooth and requires few steps to accomplish it.

## Getting ready

To follow this recipe, it's necessary to have a virtual machine (VirtualBox or VMware) that runs Oracle Solaris 11 with 4 GB RAM and a disk the same size as the existing boot disk. This example uses an 80 GB disk.

## How to do it…

Before thinking about boot disk mirroring, the first thing to do is check is the `rpool` health:

```
root@solaris11-1:~# zpool status rpool
  pool: rpool
 state: ONLINE
  scan: none requested
config:

  NAME      STATE     READ WRITE CKSUM
  rpool     ONLINE       0     0     0
    c8t0d0  ONLINE       0     0     0
```

According to this output, `rpool` is healthy, so the next step is to choose a disk with a size that is equal to or bigger than the original `rpool` disk. Then, we need to call the `format` tool and prepare it to receive the same data from the original disk as follows:

```
root@solaris11-1:~# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c8t0d0 <VBOX-HARDDISK-1.0-80.00GB>
          /pci@0,0/pci1000,8000@14/sd@0,0
       1\. c8t1d0 <VBOX-HARDDISK-1.0-16.00GB>
          /pci@0,0/pci1000,8000@14/sd@1,0
       2\. c8t2d0 <VBOX-HARDDISK-1.0-4.00GB>
          /pci@0,0/pci1000,8000@14/sd@2,0
       3\. c8t3d0 <VBOX-HARDDISK-1.0 cyl 10441 alt 2 hd 255 sec 63>
          /pci@0,0/pci1000,8000@14/sd@3,0
…..(truncated)

Specify disk (enter its number): 3
selecting c8t3d0
[disk formatted]
No Solaris fdisk partition found.

format> fdisk
No fdisk table exists. The default partition for the disk is:

  a 100% "SOLARIS System" partition

Type "y" to accept the default partition,  otherwise type "n" to edit the
 partition table.
y
format> p
partition> p

Current partition table (default):
Total disk cylinders available: 10440 + 2 (reserved cylinders)

Part      Tag   Flag    Cylinders    Size            Blocks
  0 unassigned   wm     0            0         (0/0/0)             0
  1 unassigned   wm     0            0         (0/0/0)             0
  2     backup   wu     0 - 10439   79.97GB    (10440/0/0) 167718600

(truncated output)
partition> q

root@solaris11-1:~#
```

Once we've chosen which will be the mirrored disk, the second disk has to be attached to the existing root pool (`rpool`) to mirror the boot and system files. Remember that the mirroring process will include all the snapshots from the filesystem under the `rpool` disk. The mirroring process is initiated by running:

```
root@solaris11-1:~# zpool attach rpool c8t0d0 c8t3d0

```

### Note

Make sure that you wait until resilvering is done before rebooting.

To follow the mirroring process, execute the following commands:

```
root@solaris11-1:~# zpool status rpool
  pool: rpool
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
  continue to function in a degraded state.
action: Wait for the resilver to complete.
  Run 'zpool status -v' to see device specific details.
  scan: resilver in progress since Tue Dec 10 02:32:22 2013
    4.19M scanned out of 38.2G at 82.0K/s, 30h42m to go
    4.15M resilvered, 0.02% done
config:

  NAME        STATE     READ WRITE CKSUM
  rpool       DEGRADED     0     0     0
    mirror-0  DEGRADED     0     0     0
      c8t0d0  ONLINE       0     0     0
      c8t3d0  DEGRADED     0     0     0  (resilvering)

errors: No known data errors
```

To avoid executing the previous command several times, it would be simpler to make a script as follows:

```
root@solaris11-1:~# while true
> do
> zpool status | grep done
> sleep 2
> done
    2.15G resilvered, 5.54% done
    2.19G resilvered, 5.70% done

…………
(truncated output)
………..
    38.1G resilvered, 99.95% done
    38.2G resilvered, 100.00% done
```

Finally, the `rpool` pool is completely mirrored as follows:

```
root@solaris11-1:~# zpool status rpool
  pool: rpool
 state: ONLINE
  scan: resilvered 38.2G in 1h59m with 0 errors on Mon Dec 16 08:37:11 2013
config:

  NAME        STATE     READ WRITE CKSUM
  rpool       ONLINE       0     0     0
    mirror-0  ONLINE       0     0     0
      c8t0d0  ONLINE       0     0     0
      c8t3d0  ONLINE       0     0     0
```

### An overview of the recipe

After adding the second disk (mirror disk) into the `rpool` pool and after the entire mirroring process has finished, the system can be booted using the alternative disk (through BIOS, we're able to initialize the system from the mirrored disk). For example, this example was done using VirtualBox, so the alternative disk can be chosen using the *F12* key.

# ZFS shadowing

Most companies have very heterogeneous environments where some machines are outdated and others are new. Usually, it's required to copy data from the old machine to a new machine that runs Oracle Solaris 11, and it's a perfect time to use an excellent feature named Shadow Migration. This feature can be used to copy (migrate) data through NFS or locally (between two machines), and the filesystem types that can be used as the origin are UFS, VxFS (from Symantec), and surely, the fantastic ZFS.

An additional and very attractive characteristic of this feature is the fact that a client application doesn't need to wait for the data migration to be complete at the target, and it can access all data that was already migrated. If the required data wasn't copied to the new machine (target) while being accessed, then ZFS will fail through to the source (original data).

## Getting ready

This recipe requires two virtual machines (`solaris11-1` and `solaris11-2`) with Oracle Solaris 11 installed and 4 GB RAM each. Furthermore, the example will show you how to migrate data from an existing filesystem (`/shadowing_pool/origin_filesystem`) in the `solaris11-2` system (source) to the `solaris11-1` system (target or destination).

## How to do it…

Remember that the source machine is the `solaris11-2` system (from where the data will be migrated), and the `solaris11-1` system is the destination or target. Therefore, the first step to handle shadowing is to install the `shadow-migration` package on the destination machine to where the data will be migrated, by executing the following command:

```
root@solaris11-1:~# pkg install shadow-migration

```

After the installation of the package, it's suggested that you check whether the shadowing service is enabled, by executing the following command:

```
root@solaris11-1:~# svcs -a | grep shadow
disabled       18:35:00 svc:/system/filesystem/shadowd:default
```

As the shadowing service isn't enabled, run the following command to enable it:

```
root@solaris11-1:~# svcadm enable
svc:/system/filesystem/shadowd:default
```

On the second machine (`solaris11-2`, the source host), the filesystem to be migrated must be shared in a read-only mode using NFS. Why must it be read-only ? Because the content can't change during the migration.

Let's set up a test ZFS filesystem to be migrated using Shadow Migration and to make the filesystem read-only:

```
root@solaris11-2:~# zpool create shadowing_pool c8t3d0
root@solaris11-2:~# zfs create shadowing_pool/origin_filesystem
root@solaris11-2:~# zfs list -r shadowing_pool
NAME                              USED  AVAIL  REFER  MOUNTPOINT
shadowing_pool                    124K  3.91G    32K  /shadowing_pool
shadowing_pool/origin_filesystem   31K  3.91G    31K  /shadowing_pool/origin_filesystem
```

The following command copies some data (readers can copy anything) to the `shadowing_pool/origin_filesystem` filesystem from `solaris11-2` to simulate a real case of migration:

```
root@solaris11-2:~# cp -r * /shadowing_pool/origin_filesystem/

```

Share the origin filesystem as read-only data (`-o ro`) using the NFS service by executing the following command:

```
root@solaris11-2:~# share -F nfs -o ro /shadowing_pool/origin_filesystem 
root@solaris11-2:~# share
shadowing_pool_origin_filesystem  /shadowing_pool/origin_filesystem  nfs  sec=sys,ro
```

On the first machine (`solaris11-1`), which is the destination where data will be migrated (copied), check whether the NFS share is okay and reachable by running the following command:

```
root@solaris11-1:~# dfshares solaris11-2
RESOURCE                                   SERVER ACCESS    TRANSPORT
solaris11-2:/shadowing_pool/origin_filesystem  solaris11-2  -
```

The system is all in place. The shadowing process is ready to start from the second system (`solaris11-2`) to the first system (`solaris11-1`). This process will create the `shadowed_pool/shad_filesystem` filesystem by executing the following command:

```
root@solaris11-1:~# zpool create shadowed_pool c8t3d0
root@solaris11-1:~# zfs create -o shadow=nfs://solaris11-2/shadowing_pool/origin_filesystem shadowed_pool/shad_filesystem

```

The shadowing process can be tracked by running the `shadowstat` command:

```
root@solaris11-1:~/Desktop# shadowstat
                               EST
                        BYTES  BYTES    ELAPSED
DATASET                        XFRD    LEFT  ERRORS  TIME
shadowed_pool/shad_filesystem   -       -     -      00:00:13
shadowed_pool/shad_filesystem   -       -     -      00:00:23
shadowed_pool/shad_filesystem   -       -     -      00:00:33
shadowed_pool/shad_filesystem   -       -     -      00:00:43
shadowed_pool/shad_filesystem   -       -     -      00:00:53
shadowed_pool/shad_filesystem   -       -     -      00:01:03
(truncated output)
shadowed_pool/shad_filesystem   -       -     -      00:07:33
shadowed_pool/shad_filesystem   -       -     -      00:07:43
shadowed_pool/shad_filesystem   -       -     -      00:07:53
shadowed_pool/shad_filesystem   1.57G   -     -      00:08:03
No migrations in progress
```

The finished shadowing task is verified by executing the following command:

```
root@solaris11-1:~/Desktop# zfs list -r shadowed_pool 
NAME                            USED  AVAIL  REFER  MOUNTPOINT
shadowed_pool                  1.58G  2.33G    32K  /shadowed_pool
shadowed_pool/shad_filesystem  1.58G  2.33G  1.58G  /shadowed_pool/shad_filesystem
root@solaris11-1:~/Desktop# zfs get -r shadow shadowed_pool/shad_filesystem
NAME                           PROPERTY  VALUE  SOURCE
shadowed_pool/shad_filesystem  shadow    none   -
```

The shadowing process worked! Moreover, the same operation is feasible to be accomplished using two local ZFS filesystems (the previous process was done through NFS between the `solaris11-2` and `solaris11-1` systems). Thus, the entire recipe can be repeated to copy some files to the source filesystem (it can be any data we want) and to start the shadowing activity by running the following commands:

```
root@solaris11-1:~# zfs create rpool/shad_source
root@solaris11-1:~# cp /root/kali-linux-1.0.5-amd64.iso /root/john* /root/mh* /rpool/shad_source/
root@solaris11-1:~# zfs set readonly=on rpool/shad_source
root@solaris11-1:~# zfs create -o shadow=file:///rpool/shad_source rpool/shad_target
root@solaris11-1:~# shadowstat
                              EST
        BYTES                 BYTES       ELAPSED
DATASET                       XFRD  LEFT  ERRORS  TIME
rpool/shad_target               -      -      -  00:00:08
rpool/shad_target               -      -      -  00:00:18
rpool/shad_target               -      -      -  00:00:28
rpool/shad_target               -      -      -  00:00:38
rpool/shad_target               -      -      -  00:00:48
rpool/shad_target               -      -      -  00:00:58
rpool/shad_target               -      -      -  00:01:08
rpool/shad_target               -      -      -  00:01:18
rpool/shad_target               -      -      -  00:01:28
rpool/shad_target               -      -      -  00:01:38
rpool/shad_target               -      -      -  00:01:48
rpool/shad_target               -      -      -  00:01:58
rpool/shad_target               -      -      -  00:02:08
rpool/shad_target               -      -      -  00:02:18
rpool/shad_target               -      -      -  00:02:28
rpool/shad_target               1.58G  2.51G  -  00:02:38
rpool/shad_target               1.59G  150M   -  00:02:48
rpool/shad_target               1.59G  8E     -  00:02:58
No migrations in progress
```

Everything has worked perfectly as expected, but in this case, we used two local ZFS filesystems instead of using the NFS service. Therefore, the completed process can be checked and finished by executing the following command:

```
root@solaris11-1:~# zfs get shadow rpool/shad_source
NAME               PROPERTY  VALUE  SOURCE
rpool/shad_source  shadow    none   -
root@solaris11-1:~# zfs set readonly=off rpool/shad_source

```

### An overview of the recipe

The shadow migration procedure was explained in two contexts—using a remote filesystem through NFS and using local filesystems. In both cases, it's necessary to set the read-only mode for the source filesystem. Furthermore, you learned how to monitor the shadowing using `shadowstat` and even the `shadow` property.

# Configuring ZFS sharing with the SMB share

Oracle Solaris 11 has introduced a new feature that enables a system to share its filesystems through the **Server Message Block** (**SMB**) and **Common Internet File System** (**CIFS**) protocols, both being very common in the Windows world. In this section, we're going to configure two filesystems and access these using CIFS.

## Getting ready

This recipe requires two virtual machines (VMware or VirtualBox) that run Oracle Solaris 11, with 4 GB memory each, and some test disks with 4 GB. Furthermore, we'll require an additional machine that runs Windows (for example, Windows 7) to test the CIFS shares offered by Oracle Solaris 11.

## How to do it…

To begin the recipe, it's necessary to install the smb service by executing the following command:

```
root@solaris11-1:~# pkg install service/file-system/smb

```

Let's create a pool and two filesystems inside it by executing the following command:

```
root@solaris11-1:~# zpool create cifs_pool c8t4d0
root@solaris11-1:~# zfs create cifs_pool/zfs_cifs_1
root@solaris11-1:~# zfs create cifs_pool/zfs_cifs_2
root@solaris11-1:~# zfs list -r cifs_pool
NAME                  USED  AVAIL  REFER  MOUNTPOINT
cifs_pool             162K  3.91G    33K  /cifs_pool
cifs_pool/zfs_cifs_1   31K  3.91G    31K  /cifs_pool/zfs_cifs_1
cifs_pool/zfs_cifs_2   31K  3.91G    31K  /cifs_pool/zfs_cifs_2
```

Another crucial configuration is to set mandatory locking (the `nbmand` property) for each filesystem, which will be offered by CIFS, because Unix usually uses advisory locking and SMB uses mandatory locking. A very quick explanation about these kinds of locks is that an advisory lock doesn't prevent non-cooperating clients (or processes) from having read or write access to a shared file. On the other hand, mandatory clients prevent any non-cooperating clients (or processes) from having read or write access to shared file.

We can accomplish this task by running the following commands:

```
root@solaris11-1:~# zfs set nbmand=on cifs_pool/zfs_cifs_1
root@solaris11-1:~# zfs set nbmand=on cifs_pool/zfs_cifs_2

```

Our initial setup is ready. The following step shares the `cifs_pool/zfs_cifs_1` and `cifs_pool/zfs_cifs_2` filesystems through the SMB protocol and configures a share name (`name`), protocol (`prot`), and path (`file system path`). Moreover, a cache client (`csc`) is also configured to smooth the performance when the filesystem is overused:

```
root@solaris11-1:~# zfs set share=name=zfs_cifs_1,path=/cifs_pool/zfs_cifs_1,prot=smb,csc=auto cifs_pool/zfs_cifs_1
name=zfs_cifs_1,path=/cifs_pool/zfs_cifs_1,prot=smb,csc=auto
root@solaris11-1:~# zfs set share=name=zfs_cifs_2,path=/cifs_pool/zfs_cifs_2,prot=smb,csc=auto cifs_pool/zfs_cifs_2
name=zfs_cifs_2,path=/cifs_pool/zfs_cifs_2,prot=smb,csc=auto
```

Finally, to enable the SMB share feature for each filesystem, we must set the `sharesmb` attribute to `on`:

```
root@solaris11-1:~# zfs set sharesmb=on cifs_pool/zfs_cifs_1
root@solaris11-1:~# zfs set sharesmb=on cifs_pool/zfs_cifs_2
root@solaris11-1:~# zfs get sharesmb  cifs_pool/zfs_cifs_1
NAME                  PROPERTY   VALUE  SOURCE
cifs_pool/zfs_cifs_1  share.smb  on     local
root@solaris11-1:~# zfs get sharesmb  cifs_pool/zfs_cifs_2
NAME                  PROPERTY   VALUE  SOURCE
cifs_pool/zfs_cifs_2  share.smb  on     local
```

The SMB Server service isn't enabled by default. By the way, the **Service Management Facility** (**SMF**) still wasn't introduced, but the `svcs –a` command lists all the installed services and shows which services are online, offline, or disabled. As we are interested only in the `smb/server` service, we can use the `grep` command to filter the target service by executing the following command:

```
root@solaris11-1:~# svcs -a | grep smb/server
disabled          7:13:51 svc:/network/smb/server:default
```

The `smb/server` service is disabled, and to enable it, you need to execute the following command:

```
root@solaris11-1:~# svcadm enable -r smb/server
root@solaris11-1:~# svcs -a | grep smb       
online          7:12:50 svc:/network/smb:default
online          7:13:47 svc:/network/smb/client:default
online          7:13:51 svc:/network/smb/server:default
```

A suitable test is to list the shares provided by the SMB server either by getting the value of the `share` filesystem property or by executing the `share` command as follows:

```
root@solaris11-1:~# zfs get share
NAME                                        PROPERTY  VALUE  SOURCE
cifs_pool/zfs_cifs_1                        share     name=zfs_cifs_1,path=/cifs_pool/zfs_cifs_1,prot=smb,csc=auto  local
cifs_pool/zfs_cifs_2                        share     name=zfs_cifs_2,path=/cifs_pool/zfs_cifs_2,prot=smb,csc=auto  local

root@solaris11-1:~# share
IPC$    smb  -  Remote IPC
c$  /var/smb/cvol  smb  -  Default Share
zfs_cifs_1  /cifs_pool/zfs_cifs_1  smb  csc=auto
zfs_cifs_2  /cifs_pool/zfs_cifs_2  smb  csc=auto
root@solaris11-1:~#
```

To proceed with a real test that accesses an SMB share, let's create a regular user named `aborges` and assign a password to him by running the following command:

```
root@solaris11-1:~# useradd aborges  
root@solaris11-1:~# passwd aborges
New Password: 
Re-enter new Password: 
passwd: password successfully changed for aborges
```

The user `aborges` needs to be enabled in the SMB service, so execute the following command:

```
root@solaris11-1:~# smbadm enable-user aborges
aborges is enabled.
root@solaris11-1:~#
```

To confirm that the user `aborges` was created and enabled for the SMB service, run the following command:

```
root@solaris11-1:~# smbadm lookup-user aborges
aborges: S-1-5-21-3351362105-248310137-3301682468-1104
```

According to the previous output, a **security identifier** (**SID**) was assigned to the user `aborges`. The next step is to enable the SMB authentication by adding a new library (`pam_smb_passwd.so.1`) in the authentication scheme by executing the following command:

```
root@solaris11-1:~# vi /etc/pam.d/other 
……………………..
(truncated)
……………………….
password include  pam_authtok_common
password required  pam_authtok_store.so.1
password required  pam_smb_passwd.so.1  nowarn
```

The best way to test all the steps until here is to verify that the shares are currently being offered to the other machine (`solaris11-2`) by running the following command:

```
root@solaris11-2:~# smbadm lookup-server //solaris11-1
Workgroup: WORKGROUP
Server: SOLARIS11-1
IP address: 192.168.1.119
```

To show which shares are available from the `solaris11-1` host, run the following command:

```
root@solaris11-2:~# smbadm show-shares -u aborges solaris11-1
Enter password:
c$                  Default Share
IPC$                Remote IPC
zfs_cifs_1          
zfs_cifs_2          
4 shares (total=4, read=4)
```

To mount the first ZFS share (`zfs_cifs_1`) using the SMB service on `solaris11-2` from `solaris11-1`, execute the following command:

```
root@solaris11-2:~# mount -o user=aborges -F smbfs //solaris11-1/zfs_cifs_1 /mnt

```

The mounted filesystem is an SMB filesystem (`-F smbfs`), and it's easy to check its content by executing the following commands:

```
root@solaris11-2:~# df -h /mnt
Filesystem             Size   Used  Available Capacity  Mounted on
//solaris11-1/zfs_cifs_1
                       3.9G    40K       3.9G     1%    /mnt
root@solaris11-2:~# ls -l /mnt
total 10
-rwxr-x---+  1 2147483649 2147483650     893 Dec 17 21:04 zfsslower.d
-rwxr-x---+  1 2147483649 2147483650     956 Dec 17 21:04 zfssnoop.d
-rwxr-x---+  1 2147483649 2147483650     466 Dec 17 21:04 zioprint.d
-rwxr-x---+  1 2147483649 2147483650    1255 Dec 17 21:04 ziosnoop.d
-rwxr-x---+  1 2147483649 2147483650     650 Dec 17 21:04 ziotype.d
```

SMB is very common in Windows environments, and then, it would be nice to access these shares from a Windows machine (Windows 7 in this case) by accessing the network shares by going to the **Start** menu and typing `\\192.168.1.119` as shown in the following screenshot:

![How to do it…](img/00008.jpeg)

From the previous screenshot, there are two shares being offered to us: `zfs_cifs_1` and `zfs_cifs_2`. Therefore, we can try to access one of them by double-clicking it and filling out the credentials as shown in the following screenshot:

![How to do it…](img/00009.jpeg)

As expected, the username and password are required according to the rules from the Windows system that enforce the `[Workgroup][Domain]\[user]` syntax. So, after we fill the textboxes, the `zfs_cifs_1 file system` content is shown as seen in the following screenshot:

![How to do it…](img/00010.jpeg)

Everything has worked as we expected, and if we need to undo the SMB sharing offered by the `solaris11-1` system, it's easy to do so by executing the following command:

```
root@solaris11-2:~# umount /mnt
root@solaris11-1:~# zfs set -c share=name=zfs_cifs_1 cifs_pool/zfs_cifs_1
share 'zfs_cifs_1' was removed.
root@solaris11-1:~# zfs set -c share=name=zfs_cifs_2 cifs_pool/zfs_cifs_2
share 'zfs_cifs_2' was removed.
root@solaris11-1:~# share
IPC$            smb     -       Remote IPC
c$      /var/smb/cvol   smb     -       Default Share
root@solaris11-1:~# zfs get share
root@solaris11-1:~#
```

### An overview of the recipe

In this section, the CIFS sharing in Oracle Solaris 11 was also explained in a step-by-step procedure that showed us how to configure and access CIFS shares.

# Setting and getting other ZFS properties

Managing ZFS properties is one of the secrets when we are working with the ZFS filesystem, and this is the reason why understanding the inherence concept is very important.

One ZFS property can usually have three origins as source: `local` (the property value was set locally), `default` (the property wasn't set either locally or by inheritance), and `inherited` (the property was inherited from an ancestor). Additionally, two other values are possible: `temporary` (the value isn't persistent) and `none` (the property is read-only, and its value was generated by ZFS). Based on these key concepts, the sections are going to present different and interesting properties for daily administration.

## Getting ready

This recipe can be followed using two virtual machines (VirtualBox or VMware) with Oracle Solaris 11 installed, 4 GB RAM, and eight disks of at least 4 GB.

## How to do it…

Working as a small review, datasets such as pools, filesystems, snapshots, and clones have several properties that administrators are able to list, handle, and configure. Therefore, the following commands will create a pool and three filesystems under this pool. Additionally, we are going to copy some data (a reminder again—we could use any data) into the first filesystem as follows:

```
root@solaris11-1:~# zpool create prop_pool c8t5d0
root@solaris11-1:~# zfs create prop_pool/zfs_1
root@solaris11-1:~# zfs create prop_pool/zfs_2
root@solaris11-1:~# zfs create prop_pool/zfs_3
root@solaris11-1:~# cp -r socat-2.0.0-b6.tar.gz dtbook_scripts* /prop_pool/zfs_1

```

To get all the properties from a pool and filesystem, execute the following command:

```
root@solaris11-1:~# zpool get all prop_pool
NAME       PROPERTY       VALUE                 SOURCE
prop_pool  allocated      1.13M                 -
prop_pool  altroot        -                     default
prop_pool  autoexpand     off                   default
prop_pool  autoreplace    off                   default
prop_pool  bootfs         -                     default
prop_pool  cachefile      -                     default
prop_pool  capacity       0%                    -
prop_pool  dedupditto     0                     default
prop_pool  dedupratio     1.00x                 -
prop_pool  delegation     on                    default
prop_pool  failmode       wait                  default
prop_pool  free           3.97G                 -
prop_pool  guid           10747479388132741479  -
prop_pool  health         ONLINE                -
prop_pool  listshares     off                   default
prop_pool  listsnapshots  off                   default
prop_pool  readonly       off                   -
prop_pool  size           3.97G                 -
prop_pool  version        34                    default
root@solaris11-1:~# zfs get all prop_pool/zfs_1
NAME             PROPERTY              VALUE                  SOURCE
prop_pool/zfs_1  aclinherit            restricted             default
prop_pool/zfs_1  aclmode               discard                default
prop_pool/zfs_1  atime                 on                     default
prop_pool/zfs_1  available             3.91G   
(truncated output)

```

Both commands have a similar syntax, and we've got all the properties from the `prop_pool` pool and the `prop_pool/zfs_1` filesystem.

In the *ZFS shadowing* section, we touched the NFS subject, and some filesystems were shared using the `share` command. Nonetheless, they could have been shared using ZFS properties, such as `sharenfs`, that have a value equal to `off` by default (when we use this value, it isn't managed by ZFS and is still using `/etc/dfs/dfstab`). Let's take the `sharenfs` property, which will be used to highlight some basic concepts about properties.

As usual, the property listing is too long; it is faster to get only one property's value by executing the following command:

```
root@solaris11-1:~# zfs get sharenfs prop_pool
NAME       PROPERTY   VALUE  SOURCE
prop_pool  share.nfs  off    default
root@solaris11-1:~# zfs get sharenfs prop_pool/zfs_1
NAME             PROPERTY   VALUE  SOURCE
prop_pool/zfs_1  share.nfs  off    default
```

Moreover, the same property can be got recursively by running the following command:

```
root@solaris11-1:~# zfs get -r sharenfs prop_pool      
NAME             PROPERTY   VALUE  SOURCE
prop_pool        share.nfs  off    default
prop_pool/zfs_1  share.nfs  off    default
prop_pool/zfs_2  share.nfs  off    default
prop_pool/zfs_3  share.nfs  off    default 
```

From the last three outputs, we noticed that the `sharenfs` property is disabled on the pool and filesystems, and this is the default value set by Oracle Solaris 11.

The `sharenfs` property can be enabled by executing the following command:

```
root@solaris11-1:~# zfs set sharenfs=on prop_pool/zfs_1
root@solaris11-1:~# zfs get -r sharenfs prop_pool/zfs_1
NAME              PROPERTY   VALUE  SOURCE
prop_pool/zfs_1   share.nfs  on     local
prop_pool/zfs_1%  share.nfs  on     inherited from prop_pool/zfs_1
```

As `sharenfs` was set to `on` for `prop_pool/zfs_1`, the source value has changed to `local`, indicating that this value wasn't inherited, but it was set locally. Therefore, execute the following command:

```
root@solaris11-1:~# zfs get -s local all prop_pool/zfs_1
NAME             PROPERTY     VALUE  SOURCE
prop_pool/zfs_1  share.*      ...    local
root@solaris11-1:~# zfs get -r sharenfs prop_pool
NAME              PROPERTY   VALUE  SOURCE
prop_pool         share.nfs  off    default
prop_pool/zfs_1   share.nfs  on     local
prop_pool/zfs_1%  share.nfs  on     inherited from prop_pool/zfs_1
prop_pool/zfs_2   share.nfs  off    default
prop_pool/zfs_3   share.nfs  off    default
```

The NFS sharing can be confirmed by running the following command:

```
root@solaris11-1:~# share
IPC$    smb  -  Remote IPC
c$  /var/smb/cvol  smb  -  Default Share
prop_pool_zfs_1  /prop_pool/zfs_1  nfs  sec=sys,rw
```

Creating a new file stem under `zfs_1` shows us an interesting characteristic. Execute the following command:

```
root@solaris11-1:~# zfs create prop_pool/zfs_1/zfs_4
root@solaris11-1:~# zfs get -r sharenfs prop_pool
NAME                    PROPERTY   VALUE  SOURCE
prop_pool               share.nfs  off    default
prop_pool/zfs_1         share.nfs  on     local
prop_pool/zfs_1%        share.nfs  on     inherited from prop_pool/zfs_1
prop_pool/zfs_1/zfs_4   share.nfs  on     inherited from prop_pool/zfs_1
prop_pool/zfs_1/zfs_4%  share.nfs  on     inherited from prop_pool/zfs_1
prop_pool/zfs_2         share.nfs  off    default
prop_pool/zfs_3         share.nfs  off    default
```

The new `zfs_4` filesystem has the `sharenfs` property inherited from the `upper zfs_1` filesystem; now execute the following command to list all the inherited properties:

```
root@solaris11-1:~# zfs get -s inherited all prop_pool/zfs_1/zfs_4
NAME                   PROPERTY     VALUE  SOURCE
prop_pool/zfs_1/zfs_4  share.*      ...    inherited
root@solaris11-1:~# share
IPC$    smb  -  Remote IPC
c$  /var/smb/cvol  smb  -  Default Share
prop_pool_zfs_1  /prop_pool/zfs_1  nfs  sec=sys,rw
prop_pool_zfs_1_zfs_4  /prop_pool/zfs_1/zfs_4  nfs  sec=sys,rw
```

That's great! The new `zfs_4` filesystem has inherited the `sharenfs` property, and it appears in the `share` output command.

A good question is whether a filesystem will be able to fill all the space of a pool. Yes, it will be able to! Now, this is the reason for ZFS having several properties related to the amount of space on the disk. The first of them, the `quota` property, is a well-known property that limits how much space a dataset (filesystem in this case) can fill in a pool. Let's take an example:

```
root@solaris11-1:~# zfs list -r prop_pool
NAME                    USED  AVAIL  REFER  MOUNTPOINT
prop_pool               399M  3.52G   391M  /prop_pool
prop_pool/zfs_1        8.09M  3.52G  8.06M  /prop_pool/zfs_1
prop_pool/zfs_1/zfs_4    31K  3.52G    31K  /prop_pool/zfs_1/zfs_4
prop_pool/zfs_2          31K  3.52G    31K  /prop_pool/zfs_2
prop_pool/zfs_3          31K  3.52G    31K  /prop_pool/zfs_3
```

All filesystems struggle to use the same space (`3.52G`), and one of them can fill more space than the other (or all the free space), so it is possible that a filesystem suffered a "run out space" error. A solution would be to limit the space a filesystem can take up by executing the following command:

```
root@solaris11-1:~# zfs quota=1G prop_pool/zfs_3
root@solaris11-1:~# zfs list -r prop_pool      
NAME                    USED  AVAIL  REFER  MOUNTPOINT
prop_pool               399M  3.52G   391M  /prop_pool
prop_pool/zfs_1        8.09M  3.52G  8.06M  /prop_pool/zfs_1
prop_pool/zfs_1/zfs_4    31K  3.52G    31K  /prop_pool/zfs_1/zfs_4
prop_pool/zfs_2          31K  3.52G    31K  /prop_pool/zfs_2
prop_pool/zfs_3          31K  1024M    31K  /prop_pool/zfs_3
```

The `zfs_3` filesystem space was limited to 1 GB, and it can't exceed this threshold. Nonetheless, there isn't any additional guarantee that it has 1 GB to fill. This is subtle—it can't exceed 1 GB, but there is no guarantee that even 1 GB is enough for doing it. Another serious detail—this quota space is shared by the filesystem and all the descendants such as snapshots and clones. Finally and obviously, it isn't possible to set a quota value lesser than the currently used space of the dataset.

A solution for this apparent problem is the `reservation` property. When using `reservation`, the space is guaranteed for the filesystem, and nobody else can take this space. Sure, it isn't possible to make a reservation above the quota or maximum free space, and the same rule is followed—the reservation is for a filesystem and its descendants.

When the `reservation` property is set to a value, this amount is discounted from the total available pool space, and the used pool space is increased by the same value:

```
root@solaris11-1:~# zfs list -r prop_pool
NAME                    USED  AVAIL  REFER  MOUNTPOINT
prop_pool               399M  3.52G   391M  /prop_pool
prop_pool/zfs_1        8.09M  3.52G  8.06M  /prop_pool/zfs_1
prop_pool/zfs_1/zfs_4    31K  3.52G    31K  /prop_pool/zfs_1/zfs_4
prop_pool/zfs_2          31K  3.52G    31K  /prop_pool/zfs_2
prop_pool/zfs_3          31K  1024M    31K  /prop_pool/zfs_3
```

Each dataset under `prop_pool` has its `reservation` property:

```
root@solaris11-1:~# zfs get -r reservation prop_pool
NAME                    PROPERTY     VALUE  SOURCE
prop_pool               reservation  none   default
prop_pool/zfs_1         reservation  none   default
prop_pool/zfs_1%        reservation  -      -
prop_pool/zfs_1/zfs_4   reservation  none   default
prop_pool/zfs_1/zfs_4%  reservation  -      -
prop_pool/zfs_2         reservation  none   default
prop_pool/zfs_3         reservation  none   default
```

The `reservation` property is configured to a specific value (for example, 512 MB), given that this amount is subtracted from the pool's available space and added to its used space. Now, execute the following command:

```
root@solaris11-1:~# zfs set reservation=512M prop_pool/zfs_3
root@solaris11-1:~# zfs list -r prop_pool
NAME                    USED  AVAIL  REFER  MOUNTPOINT
prop_pool               911M  3.02G   391M  /prop_pool
prop_pool/zfs_1        8.09M  3.02G  8.06M  /prop_pool/zfs_1
prop_pool/zfs_1/zfs_4    31K  3.02G    31K  /prop_pool/zfs_1/zfs_4
prop_pool/zfs_2          31K  3.02G    31K  /prop_pool/zfs_2
prop_pool/zfs_3          31K  1024M    31K  /prop_pool/zfs_3
root@solaris11-1:~# zfs get -r reservation prop_pool
NAME                    PROPERTY     VALUE  SOURCE
prop_pool               reservation  none   default
prop_pool/zfs_1         reservation  none   default
prop_pool/zfs_1%        reservation  -      -
prop_pool/zfs_1/zfs_4   reservation  none   default
prop_pool/zfs_1/zfs_4%  reservation  -      -
prop_pool/zfs_2         reservation  none   default
prop_pool/zfs_3         reservation  512M   local
```

The concern about space is usually focused on a total value for the whole pool, but it's possible to limit the available space for individual users or groups.

Setting the quota for users is done through the `userquota` property and for groups using the `groupquota` property:

```
root@solaris11-1:~# zfs set userquota@aborges=750M 
prop_pool/zfs_3
root@solaris11-1:~# zfs set userquota@alexandre=1.5G prop_pool/zfs_3
root@solaris11-1:~# zfs get userquota@aborges prop_pool/zfs_3
NAME             PROPERTY           VALUE  SOURCE
prop_pool/zfs_3  userquota@aborges  750M   local
root@solaris11-1:~# zfs get userquota@alexandre prop_pool/zfs_3
NAME             PROPERTY             VALUE  SOURCE
prop_pool/zfs_3  userquota@alexandre  1.50G  local
root@solaris11-1:~# zfs set groupquota@staff=1G prop_pool/zfs_3 
root@solaris11-1:~# zfs get groupquota@staff prop_pool/zfs_3
NAME             PROPERTY          VALUE  SOURCE
prop_pool/zfs_3  groupquota@staff  1G     local
```

Getting the used and quota space from users and groups is done by executing the following command:

```
root@solaris11-1:~# zfs userspace prop_pool/zfs_3
TYPE        NAME       USED  QUOTA  
POSIX User  aborges       0   750M  
POSIX User  alexandre     0     1G  
POSIX User  root         3K   none  
root@solaris11-1:~# zfs groupspace prop_pool/zfs_3
TYPE         NAME   USED  QUOTA  
POSIX Group  root     3K   none  
POSIX Group  staff     0     1G
```

Removing all the quota values that were set until now is done through the following sequence:

```
root@solaris11-1:~# zfs set quota=none prop_pool/zfs_3
root@solaris11-1:~# zfs set userquota@aborges=none prop_pool/zfs_3
root@solaris11-1:~# zfs set userquota@alexandre=none prop_pool/zfs_3
root@solaris11-1:~# zfs set groupquota@staff=none prop_pool/zfs_3
root@solaris11-1:~# zfs userspace prop_pool/zfs_3
TYPE        NAME  USED  QUOTA  
POSIX User  root    3K   none  
root@solaris11-1:~# zfs groupspace prop_pool/zfs_3
TYPE         NAME  USED  QUOTA
POSIX Group  root    3K   none
```

### An overview of the recipe

In this section, you saw some properties such as `sharenfs`, `quota`, `reservation`, `userquota`, and `groupquota`. All of the properties alter the behavior of the ZFS pool, filesystems, snapshots, and clones. Moreover, there are other additional properties that can improve the ZFS functionality, and I suggest that readers look for all of them in *ZFS Administration Guide*.

# Playing with the ZFS swap

One of the toughest jobs in Oracle Solaris 11 is to calculate the optimal size of the swap area. Roughly, the operating system's virtual memory is made from a sum of RAM and swap, and its correct provisioning helps the application's performance. Unfortunately, when Oracle Solaris 11 is initially installed, the correct swap size can be underestimated or overestimated, given that any possible mistake can be corrected easily. This section will show you how to manage this issue.

## Getting ready

This recipe requires a virtual machine (VMware or VirtualBox) with Oracle Solaris 11 installed and 4 GB RAM. Additionally, it's necessary to have access to eight 4 GB disks.

## How to do it…

According to Oracle, there is an estimate during the installation process that Solaris needs around one-fourth of the RAM space for a swap area in the disk. However, for historical reasons, administrators still believe in the myth that swap space should be equal or bigger than twice the RAM size for any situation. Surely, it should work, but it isn't necessary. Usually (not a rule, but observed many times), it should be something between 0.5 x RAM and 1.5 x RAM, excluding exceptions such as when predicting a database installation. Remember that the swap area can be a dedicated partition or a file; the best way to list the swap areas (and their free space) is by executing the following command:

```
root@solaris11-1:~# swap -l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3         8  4194296  4194296
```

From the previous output, the meaning of each column is as follows:

*   `swapfile`: This shows that swap areas come from two ZFS volumes `(/dev/zvol/dsk/rpool/swap` and `/dev/zvol/dsk/rpool/newswap`)
*   `dev`: This shows the major and minor number of swap devices
*   `swaplo`: This shows the minimum possible swap space, which is limited to the memory page size and its respective value is usually obtained as units of sectors (512 bytes) by executing the `pagesize` command
*   `blocks`: This is the total swap space in sectors
*   `free`: This is the free swap space (4 GB)

An alternative way to collect information about the swap area is using the same `swap` command with the `–s` option, as shown in the following command:

```
root@solaris11-1:~# swap –s
total: 519668k bytes allocated + 400928k reserved = 920596k used, 4260372k available
```

From this command output, we have:

*   `519668k bytes allocated`: This is a swap space that indicates the amount of swap space that already has been used earlier but is not necessarily in use this time. Therefore, it's reserved and available to be used when required.
*   `400928k reserved`: This is the virtual swap space that was reserved (heap segment and anonymous memory) for future use, and this time, it isn't allocated yet. Usually, the swap space is reserved when the virtual memory for a process is created. Anonymous memory refers to pages that don't have a counterpart in the disk (any filesystem). They are moved to a swap area because the shortage of RAM (physical memory) occurs many times because of the sum of stack, shared memory, and process heap, which is larger than the available physical memory.
*   `946696k used`: This is total amount of swap space that is reserved or allocated.
*   `4260372k available`: This is the amount of swap space available for future allocation.

Until now, you've learned how to monitor swap areas. From now, let's see how to add and delete swap space on Oracle Solaris 11 by executing the following commands:

```
root@solaris11-1:~# zfs list -r rpool
NAME                              USED  AVAIL  REFER  MOUNTPOINT
rpool                            37.0G  41.3G  4.91M  /rpool
rpool/ROOT                       26.7G  41.3G    31K  legacy
(truncated output)
rpool/newswap                    2.06G  41.3G  2.00G  -
rpool/shad_source                2.38G  41.3G  2.38G  /rpool/shad_source
rpool/shad_target                1.60G  41.3G  1.60G  /rpool/shad_target
rpool/swap                       2.06G  41.3G  2.00G  -
```

Two lines (`rpool/newswap` and `rpool/swap`) prove that the swap space has a size of 4 GB (2 GB + 2 GB), and both datasets are ZFS volumes, which can be verified by executing the following command:

```
root@solaris11-1:~# ls -ls /dev/zvol/rdsk/rpool/swap 
   0 lrwxrwxrwx   1 root     root           0 Dec 17 20:35 /dev/zvol/rdsk/rpool/swap -> ../../../..//devices/pseudo/zfs@0:2,raw
root@solaris11-1:~# ls -ls /dev/zvol/rdsk/rpool/newswap 
   0 lrwxrwxrwx   1 root     root           0 Dec 20 19:04 /dev/zvol/rdsk/rpool/newswap -> ../../../..//devices/pseudo/zfs@0:3,raw
```

Continuing from the previous section (getting and setting properties), the swap space can be changed by altering the `volsize` property if the pool has free space. Then, run the following command:

```
root@solaris11-1:~# zfs get volsize rpool/swap
NAME        PROPERTY  VALUE  SOURCE
rpool/swap  volsize   2G     local

root@solaris11-1:~# zfs get volsize rpool/newswap
NAME           PROPERTY  VALUE  SOURCE
rpool/newswap  volsize   2G     local
```

A simple way to increase the swap space would be by changing the `volsize` value. Then, execute the following commands:

```
root@solaris11-1:~# zfs set volsize=3G rpool/newswap
root@solaris11-1:~# zfs get volsize rpool/newswap
NAME           PROPERTY  VALUE  SOURCE

rpool/newswap  volsize   3G     local
root@solaris11-1:~# swap –l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3   4194312  2097144  2097144
root@solaris11-1:~# swap -s
total: 451556k bytes allocated + 267760k reserved = 719316k used, 5359332k available
root@solaris11-1:~# zfs list -r rpool/swap
NAME         USED  AVAIL  REFER  MOUNTPOINT
rpool/swap  2.00G  40.4G  2.00G  -
root@solaris11-1:~# zfs list -r rpool/newswap
NAME            USED  AVAIL  REFER  MOUNTPOINT
rpool/newswap  3.00G  40.4G  3.00G  -
```

Eventually, it's necessary to add a new volume because the free space on a pool isn't enough, so it can be done by executing the following commands:

```
root@solaris11-1:~# zpool create swap_pool c8t12d0
root@solaris11-1:~# zpool list swap_pool
NAME        SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
swap_pool  3.97G    85K  3.97G   0%  1.00x  ONLINE  -
root@solaris11-1:~# zfs create -V 1G swap_pool/vol_swap_1 
root@solaris11-1:~# zfs list -r swap_pool
NAME                   USED  AVAIL  REFER  MOUNTPOINT
swap_pool             1.03G  2.87G    31K  /swap_pool
swap_pool/vol_swap_1  1.03G  3.91G    16K  -
```

Once the swap volume has been created, the next step is to add it as a swap device by running the following command:

```
root@solaris11-1:~# swap -a /dev/zvol/dsk/swap_pool/vol_swap_1 
root@solaris11-1:~# swap -l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3   4194312  2097144  2097144
/dev/zvol/dsk/swap_pool/vol_swap_1 285,4         8  2097144  2097144
root@solaris11-1:~# swap -s
total: 456308k bytes allocated + 268024k reserved = 724332k used, 6361756k available
root@solaris11-1:~# zfs list -r swap_pool
NAME                   USED  AVAIL  REFER  MOUNTPOINT
swap_pool             1.03G  2.87G    31K  /swap_pool
swap_pool/vol_swap_1  1.03G  2.91G  1.00G  -
root@solaris11-1:~# zfs list -r rpool | grep swap
rpool/newswap                    3.00G  40.4G  3.00G  -
rpool/swap                       2.00G  40.4G  2.00G  -
```

Finally, the new swap device must be included in the `vfstab` file under `etc` to be mounted during the Oracle Solaris 11 boot:

```
root@solaris11-1:~# more /etc/vfstab 
#device    device    mount    FS  fsck  mount  mount
#to mount  to fsck    point    type  pass  at boot  options
#
/devices  -    /devices  devfs  -  no  -
/proc     -     /proc      proc  -  no  -
(truncated output)
swap                                -    /tmp    tmpfs  -  yes  -

/dev/zvol/dsk/rpool/swap            -    -       swap   -  no   -
/dev/zvol/dsk/rpool/newswap         -    -       swap   -  no   -
/dev/zvol/dsk/swap_pool/vol_swap_1  -    -       swap   -  no   -
```

Last but not least, the task of removing the swap area is very simple. First, the entry in `/etc/vfstab` needs to be deleted. Before removing the swap areas, they need to be listed as follows:

```
root@solaris11-1:~# swap -l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3   4194312  2097144  2097144
/dev/zvol/dsk/swap_pool/vol_swap_1 285,4         8  2097144  2097144
```

Second, the swap volume must be unregistered from the system by running the following command:

```
root@solaris11-1:~# swap -d /dev/zvol/dsk/swap_pool/vol_swap_1
root@solaris11-1:~# zpool destroy swap_pool
root@solaris11-1:~# swap -d /dev/zvol/dsk/rpool/newswap
root@solaris11-1:~# swap -l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3   4194312  2097144  2097144
```

Earlier, the `rpool/newswap` volume was increased. However, it would be impossible to decrease it because `rpool/newswap` was in use (busy). Now, as the first 2 GB space from this volume was removed, this 2 GB part isn't in use at this moment, and the total volume (3 GB) can be reduced. Execute the following commands:

```
root@solaris11-1:~# zfs get volsize rpool/newswap
NAME           PROPERTY  VALUE  SOURCE
rpool/newswap  volsize   3G     local
root@solaris11-1:~# zfs set volsize=1G rpool/newswap
root@solaris11-1:~# zfs get volsize rpool/newswap
NAME           PROPERTY  VALUE  SOURCE
rpool/newswap  volsize   1G     local
root@solaris11-1:~# swap -l
swapfile             dev    swaplo   blocks     free
/dev/zvol/dsk/rpool/swap 285,2         8  4194296  4194296
/dev/zvol/dsk/rpool/newswap 285,3   4194312  2097144  2097144
root@solaris11-1:~# swap -s
total: 456836k bytes allocated + 267580k reserved = 724416k used, 3203464k available
```

### An overview of the recipe

You saw how to add, remove, and monitor the swap space using the ZFS framework. Furthermore, You learned some very important concepts such as reserved, allocated, and free swap.

# References

*   *Oracle Solaris Administration -* *ZFS File Systems* at [http://docs.oracle.com/cd/E23824_01/html/821-1448/preface-1.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1448/preface-1.html#scrolltoc)
*   *How to configure a* *free VTL (Virtual Tape Library)* at [http://alexandreborgesbrazil.files.wordpress.com/2013/09/how-to-configure-a-free-vtl1.pdf](http://alexandreborgesbrazil.files.wordpress.com/2013/09/how-to-configure-a-free-vtl1.pdf)
*   *Oracle Solaris* *Tunable Parameters Reference Manual* at [http://docs.oracle.com/cd/E23823_01/html/817-0404/preface-1.html#scrolltoc](http://docs.oracle.com/cd/E23823_01/html/817-0404/preface-1.html#scrolltoc)
*   *Oracle Solaris* *Administration: SMB and Windows Interoperability* at [http://docs.oracle.com/cd/E23824_01/html/821-1449/toc.html](http://docs.oracle.com/cd/E23824_01/html/821-1449/toc.html)
*   *Playing with Swap Monitoring and Increasing Swap Space Using ZFS Volumes In* *Oracle Solaris 11.1* (by Alexandre Borges) at [http://www.oracle.com/technetwork/articles/servers-storage-admin/monitor-swap-solaris-zfs-2216650.html](http://www.oracle.com/technetwork/articles/servers-storage-admin/monitor-swap-solaris-zfs-2216650.html)
*   *Playing with ZFS Encryption* *In Oracle Solaris 11* (by Alexandre Borges) at [http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris-zfs-encryption-2242161.html](http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris-zfs-encryption-2242161.html)