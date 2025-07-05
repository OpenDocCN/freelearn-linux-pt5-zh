# Chapter 7. Backup Strategies

Now that you have data on your FreeNAS server and you can access it from your PC, Mac, Linux or UPnP device, it is time to think about backup. In this chapter, we shall explore the different options that exist to back up the data on the FreeNAS server including using RSYNC to a second local disk as well as to a remote machine.

From one point of view, the fact that the FreeNAS server has no support for tape or optical disk (DVD or Blu-ray) backup is a weakness. But from another point of view, this is normal as the nature of Network Attached Storage is that it is accessible from the network. All operations including configuration, management, and data access occur over the network. As such, backup is also performed over the network.

There are two mains strategies for performing a network-based backup. The first is to initiate a backup directly to tape, compressed file archive (like ZIP or compressed tar) or even optical disk from a remote server or workstation. Here, the data is pulled from the FreeNAS server (via CIFS, NFS or AFP) and written to the backup store (tape, hard drive or optical store). The second option is to copy the data, internally, inside the FreeNAS server. Here, the data remains on the FreeNAS server but is stored on a second disk or RAID set.

## Backup Your FreeNAS Using Windows XP's Built-In Backup Utility

Windows XP includes its own backup program and to start it click **Start**, point to **All Programs**, point to **Accessories**, point to **System Tools**, and then click **Backup**.

### Note

**Windows XP Home**

If you are using Windows XP Professional, the Windows Backup utility should be ready for use. If you use Windows XP Home Edition, you'll need to follow these steps to install the utility:

1\. Insert your Windows XP CD into the drive and, if necessary, double-click the CD icon in **My Computer**.

2\. On the Welcome to Microsoft Windows XP screen, click **Perform Additional Tasks**.

3\. Click **Browse this CD**.

4\. In Windows Explorer, double-click **the ValueAdd** folder, then **Msft**, and then **Ntbackup**.

5\. Double-click **Ntbackup.msi** to install the Backup utility.

1.  1\. By default, the Backup utility starts in Wizard mode, you can opt to go to advanced mode if you feel confident of configuring the backup manually. Here, we will use the Wizard mode for ease of use.

2.  2\. The first Wizard question is **What do you want to do?** referring to doing a backup or restoring files. The default should be **Back up files and settings**, which is what you want, so just click **Next**.

3.  3\. You can now choose what you want to back up. The backup program is orientated to backing up the local machine, however, it can be used to back up a network share. Select the last option **Let me choose what to back up** and click **Next**.

![Backup Your FreeNAS Using Windows XP's Built-In Backup Utility](img/4688_07_01.jpg)

1.  4\. The next step is to choose which items to back up. In the left hand pane, you can select files from **My Computer, My Documents**, and also from **My Network Places**, which is what you need to back up your FreeNAS server. Click on the small plus sign **+** next to **My Network Places** and this will expand the **My Network Places** tree. Under this, the available shares on FreeNAS server will be listed. Tick each share that you wish to back up. Using the sample configuration from the quick start guide in chapter 2, there is a share called store. Tick the box next to store to make a backup of it.

![Backup Your FreeNAS Using Windows XP's Built-In Backup Utility](img/4688_07_02.jpg)

1.  5\. Click **Next** to move on to **Backup Type, Destination**, and **Name** page. Here, you can specify a location for the backup data. If you want to use a tape drive then Backup utility gives you a choice of options in the **Select the backup type** box. If you don't have a tape drive then the default backup type will be **File**. By default, Backup proposes saving everything to the floppy drive. This clearly isn't a sane option so instead click **Browse** and choose where you would like to save the backup.

    There are a number of places you put your backup:

    *   The computer's hard disk. You need to be sure that the local hard disk is big enough for the backup.

    *   A shared network drive. You can back up the FreeNAS server to another network server, even another FreeNAS server. One thing to note is that this might not be the most efficient method as it is most likely that this middle Windows XP machine could be taken out of the equation and the two remaining machines (the FreeNAS server and the other network server) can talk directly to make the backup.

    *   An external hard disk drive. USB 2.0 and FireWire drives have become very cheap and drives space is now measured in terabytes. Adding an external hard drive and using it as backup solution is both practical and inexpensive.

    ### Note

    The built-in backup program in Windows XP doesn't support backing up to an optical drive like a DVD or Blu-ray disk. To get these features, you will need to explore some of the other freeware and commercial backup programs.

2.  6\. The final step is to enter a name for the backup. As always, be descriptive. Click **Next** to display the wizard's final page and then click **Finish** to begin backing up immediately.

## Setting Scheduled Backups with XP's Built-In Backup Utility

XP's built-in backup utility also has some advanced options including the ability to schedule regular backups.

1.  1\. To schedule a backup, start making a backup as described above BUT when you get to the final page of the Backup Wizard, don't click **Finish**, instead, click the **Advanced** button.

2.  2\. The first page of the Advanced options is the **Type of Backup** page. Here you can choose what type of backup you wish to make including Full (**Normal**) or **Differential**. If you are unsure. leave it at **Normal** (but note this will create a full backup of all the data on the FreeNAS every time the schedule backup runs). If you want to back up only the files created or changed since the last normal backup, choose **Differential**.

3.  3\. Click **Next** to move on to to the **How to Back Up** page.

### Note

**Types of Backups**

Following are the different kinds of backups:

**Normal Backup**—A normal backup copies all selected files and marks each file as having been backed up (in other words, the archive attribute is cleared). With normal backups, you need only the most recent copy of the backup file or tape to restore all of thefiles. You usually perform a normal backup the first time you create a backup set.

**Differential Backup**—A differential backup copies files created or changed since the last normal or incremental backup. It does not mark files as having been backed up (in other words, the archive attribute is not cleared). If you are performing a combination of normal and differential backups, restoring files and folders requires that you have the last normal as well as the last differential backup. The Differential Backup, used in combination with the Normal Backup, is the easiest way to create backups without having to create a full backup of all the data every time the backup is performed.

**Copy Backup**—A copy backup copies all selected files but does not mark each file as having been backed up (in other words, the archive attribute is not cleared). Copying is useful if you want to back up files between normal and incremental backups because copying does not affect these other backup operations.

**Daily Backup**—A daily backup copies all selected files that have been modified the day the daily backup is performed. The backed-up files are not marked as having been backed up (in other words, the archive attribute is not cleared).

**Incremental Backup**—An incremental backup backs up only those files created or changed since the last normal or incremental backup. It marks files as having been backed up (in other words, the archive attribute is cleared). If you use a combination of normal and incremental backups, you will need to have the last normal backup set as well as all incremental backup sets in order to restore your data.

1.  4\. On this page, you can tick the box to have your backup data verified after the backup has occurred. Verifying the backup gives you that extra assurance that the backup worked correctly, but it will lengthen the time needed to make the backup. Tick **Verify data after back up** if required and click **Next**. On the **Backup Options** page, leave the **Append this backup to existing backups** selected and click **Next**. It is not advisable to use the **Replace existing backups** option when using any kind of incremental or differential backup, as the previous backups are essential for restoring the files, should that be necessary later on.

2.  5\. On the **When to Back Up** page, choose **Later** rather than **Now**. Enter a name for this backup job (for example FreeNAS nightly backup) and then click **Set Schedule..**.

3.  6\. Now, you can set different time intervals for the backups including daily, weekly, and monthly. The screenshot below shows the settings for a 12:05AM backup every day of the week except the weekends.

![Setting Scheduled Backups with XP's Built-In Backup Utility](img/4688_07_03.jpg)

1.  7\. Click on **OK** to exit from the scheduling page and click **Next**. You may be asked to enter a username and password for the user who will run this job. It is best to enter the Administrator username and password.

2.  8\. On the final summary page, click **Finish** to program the scheduled backup.

### Note

**Removing scheduled backups**

If you wish to remove a previously scheduled backup job, then go to **Control Panel** and double click on **Scheduled Tasks**. From there, you can delete the backup job.

# Restoring a FreeNAS Backup Made with XP's Built-In Backup Utility

Once you have your backup, it is important to know how to restore it if the worst has happened.

1.  1\. Start the Backup program and choose **Restore files and settings** from the first page of the wizard.

2.  2\. Click **Next** and then double click on the name of the backup file that will be listed in the right-hand pane.

3.  3\. To do a full restore, then tick all the backup sets listed and click **Next**.

4.  4\. To selectively restore certain files, then expand the different backup sets and find the files you want to restore. Tick the small box next to the file or folder and click **Next**.

    ![Restoring a FreeNAS Backup Made with XP's Built-In Backup Utility](img/4688_07_04.jpg)
5.  5\. On the summary page, click **Finish** and the restore will start.

### Note

By default, the files will be restored to their original locations, which in this case is the FreeNAS server. If you want the files to be restored to another location, then before clicking **Finish** on the summary page click on **Advanced**. There you will be able to specify a different restore location and also control how the restore is performed (for example should files be overwritten and so on).

# Backing Up the FreeNAS Configuration Files

While talking about backup, it is useful to also mention backing up the configuration files. This is important for 2 reasons:

*   When performing an upgrade, it is always advisable to back up your configuration information in case something goes wrong during the upgrade and you wish to return to a previously good known state.

*   If you need to perform a reinstall of the FreeNAS software (for whatever reason including failed hardware) then the new installation can be configured exactly like the old installation in a matter of seconds by restoring the configuration files.

## Backup Configuration

To backup your configuration, go to the **System: Backup/Restore** page. This page is in two sections: one for backup and another for restore. To backup the configuration, click on **Download configuration**. The FreeNAS server will then send the configuration file to your web browser. Your web browser will then ask you if you would like to save the file. You should save the file on your hard disk. The filename for the configuration file is in the format *config-<hostname>-<year><month><day> <hour><minute>.xml* for example: *config-f6862a.local-20080304150414.xml*.

### What is XML?

XML stands for eXtensible Markup Language and it is a general-purpose specification for creating simple, very flexible text files that describe different types of data. It is called extensible because it allows its users to define their own elements. In FreeNAS, it is used to store all the information about your system.

A snippet from the FreeNAS configuration file would look like this:

```
<interfaces>
<lan>
<ipaddr>192.168.1.251</ipaddr>
<subnet>24</subnet>
<gateway>192.168.1.254</gateway>
</lan>
</interfaces>

```

Since XML is relatively human-legible, we can see that this sample is about the networking components of the FreeNAS server, and we can find the IP address, the subnet mask, and the default gateway with relative ease.

## Restore Configuration

To restore a configuration file, click on the **Browse..**. button and find the configuration `.xml` file you wish to restore. Now, click on **Restore configuration**. The FreeNAS configuration file will be restored and then the FreeNAS server will be rebooted.

# Using Another FreeNAS Server as a Backup Server

Clearly, one very useful option is to use a second FreeNAS server as the backup for your primary FreeNAS server. There are two possible ways to do this:

*   Using the built-in Windows backup software

*   Interfacing two FreeNAS servers

The first method has been covered in the previous section where we create a backup of the FreeNAS primary server on a FreeNAS backup server.

The second method has the added benefit that if the hardware fails on the first machine, the second machine will be ready quite quickly to take the failed machine's place. To transfer the data between the primary server and the backup machine, we shall use RSYNC.

RSYNC is a network protocol, specifically designed for performing network backups. RSYNC creates an exact copy of your data over the network, but to save network bandwidth, it has a built-in algorithm that only copies the parts of files which are different from the original. This makes it efficient and effective.

Before you start, you need to set up a second FreeNAS server. You need to follow the normal steps for settings up a FreeNAS server:

*   Burn a CD and boot from it (and optionally install to a hard disk)

*   Configure the networking

*   Configure storage (either RAID or simply disks)

Now, on the primary FreeNAS server, you need to configure the RSYNC server.

1.  1\. Go to **Services: RSYNCD** and enable the RSYNC Daemon. You can leave the rest of the settings as they are.

2.  2\. Click on the **Modules** tab.

3.  3\. In RSYNC, talk modules are like shares in CIFS. To give others access to a particular area on your FreeNAS server, you need to create a module for it. Click on the add circle to add a new module.

There are 3 mandatory fields; name, comment, and path:

*   **Name**—This is a label for the module and it will be used by the RSYNC client to identify this particular shared resource.

*   **Comment**—This is a description of the module, for example: Sales Material. Having a good comment here is essential for debugging and problem solving.

*   **Path**— This is the path to the storage that is to be shared using RSYNC. The format of this is */mnt/storagename* where storage name is the mount point name of the disk or RAID set you configured in the **Disks** section. Click on **...** at the end of the Path section. This will bring up a simple file system browser. Click the desired mount point (for example store) and click **OK** and you will be taken back to the RSYNC modules page. Now, the mount point (e.g. */mnt/store/)* has been added as the path.

1.  4\. For extra safety, you can set the **Access mode** to **Read only**, which will ensure that this module can't be written to by other RSYNC clients. Once the RSYNC server is configured, any RSYNC client on the network can access these files. To stop mishaps and accidents, it is best to limit it to read only.

2.  5\. The rest of the options can be left at their defaults.

3.  6\. Click **Add** and apply the changes.

![Using Another FreeNAS Server as a Backup ServerFreeNAS configuration filesconfiguration, restoring](img/4688_07_05.jpg)

Now on the FreeNAS backup server, we need to create an RSYNC client. The client will connect to the server and copy the files in its local storage. Every time the RSYNC client runs, it will back up the data on the primary server to the backup server. Because of the nature of the RSYNC algorithms, only data that has changed will be copied over and so reduce the overheads in copying the files.

1.  1\. To create the RSYNC client go to **Services: RSYNCD** on the backup server.

2.  2\. Click on the **Client** tab.

3.  3\. Click on the add circle to create a new client.

    On the **Client: Add** page there are 4 obligatory fields; Local share, Remote RSYNC Server, Remote module name, and Synchronization Time:

    *   **Local share**—This is the local storage where the file from the primary FreeNAS server will be copied. It needs to be big enough to hold all the files that will be copied over from the primary server. The format of this is **/mnt/storagename** where storage name is the mount point name of the disk or RAID set you configured in the **Disks** section. Click on **...** at the end of the Path section. This will bring up a simple file system browser. Click the desired mount point (for example backup) and click **OK** and you will be taken back to the RSYNC modules page. Now the mount point (e.g. */mnt/backup)* has been added as the path.

    *   **Remote RSYNC Server**—This is the IP address of the primary FreeNAS server. The address will be in dot notation, for example 192.168.1.250.

    *   **Remote module name**—This is the label or module name you configured in the Modules page of the RSYNC server on the primary FreeNAS machine. You need to enter it exactly here as you entered it there.

    *   **Synchronization Time**—The client runs to synchronize the backup server with primary server at scheduled times. These synchronizations are scheduled by selecting which minute, hour, day, and month you want them to occur. As this is a re-occurring event, you can choose which time, date, and day of the week the backup will be made. To schedule the backup, select the time you want by selecting the appropriate minute, hour, day, month, and week day. For example, to back up every morning at 12:05 AM, Monday to Friday you would select:

        *   5 from the minutes section

        *   0 from the hours (remember it is a 24 hour clock)

        *   Monday through to Friday from the week days

        *   Days and months would remain as ALL

        ### Note

        Use *CTRL*-click (or Command-click on the Mac) to select and de-select minutes, hours, days, months, and week days.

4.  4\. You can also set the optional parameter **Delete files that don't exist on sender**. Ticking this means that if a file is deleted on the primary FreeNAS server, it will also be deleted on the backup server when the synchronization occurs. The negative side of this is that when a file is deleted on the primary server, it is effectively lost forever (unless you are also using some other type of backup system) as it will be deleted on the backup server as well. The other thing to note is that if files are not deleted, then the size of the backup data will keep increasing and never go down even when files are deleted from the primary server.

5.  5\. Now click **Save** and apply the changes.

That is it, when the next synchronization interval comes around, the two servers should synchronize automatically.

## Debugging Your RSYNC Setup

If your backups are scheduled to happen once a day, it can be a long process to verify that the backups are occurring. For every simple typing mistake, you have to wait 24 hours to see if the backups happened. This can be very frustrating. There are a few things you can do to ensure that your RSYNC configuring is correct.

One thing you can do to shorten the wait is to temporarily schedule the backup for a few minutes from now. Then, when you know the backup is working correctly, you can schedule your desired backup time.

On the backup server, it is possible to check that the RSYNC client can communicate with the RSYNC server on the primary FreeNAS server and that the module is visible. Go to **Diagnostics: Information** and click on the **RSYNC Client** tab. Here, each configured RSYNC client will be listed along with its configuration parameters. The most useful information is the **Detected shares on this server** section. Here, the RSYNC client has contacted the server and asked for a list of modules. The modules available are listed.

If the modules are listed and specifically the one needed by the backup server to perform its backup, then you know that the configuration is on the right track.

![Debugging Your RSYNC Setup](img/4688_07_06.jpg)

On the primary server, there is a log file of RSYNC activity. This will allow you to check that the backups are occurring. Go to **Diagnostics: Logs** and click on the **RSYNCD** tab. By default, the latest entries are shown at the bottom on the log. You are looking for something similar to this:

**Mar 4 12:35:00 rsyncd[3545]: connect from UNKNOWN (192.168.1.252)**

**Mar 4 12:35:00 rsyncd[3545]: rsync on store from UNKNOWN (192.168.1.252)**

**Mar 4 12:35:00 rsyncd[3545]: building file list**

**Mar 4 12:35:00 rsyncd[3545]: sent 673 bytes received 70 bytes total size 3998023**

This shows that a connection was made (in this example from my backup server) and that synchronization was started for the *store* module. The last line shows the amount of network traffic, which in this case was low as there were no changes to synchronize.

# RSYNC Internal Backup

In a similar way that two FreeNAS servers can use RSYNC to synchronize data, FreeNAS has an option to use RSYNC to synchronize data from one hard disk to another inside the same FreeNAS server. This is known as local RSYNC synchronization and to use it, you need at least two hard disks or RAID sets (the source and the destination), which are both configured and mounted.

To enable this feature, go to **Services: RSYNCD** and click on the **Local** tab. Now click on the add circle to configure a new local synchronization. You need to complete three fields to configure the synchronization: **Source share, Destination share**, and **Synchronization Time:**

*   **Source share**—This is the mount point of the data that needs backing up. The format of this is */mnt/storagename* where storage name is the mount point name of the disk or RAID set you configured in the **Disks** section. Click on **...** at the end of the Path section. This will bring up a simple file system browser. Click the desired mount point (for example store) and click **OK**. You will be taken back to the RSYNC modules page. Now, the mount point (e.g. /mnt/store/) has been added as the path.

*   **Destination share**—This is the mount point where you want the data to be copied. The format of this is /mnt/storagename where storage name is the mount point name of the disk or RAID set you configured in the Disks section. Click on "..." at the end of the Path section. This will bring up a simple file system browser. Click the desired mount point (for example backup) and click OK and you will be taken back to the RSYNC modules page. Now the mount point (e.g. /mnt/backup/) has been added as the path.

*   **Synchronization time**—The program runs to synchronize the two storage areas at scheduled times. These synchronizations are scheduled by selecting which minute, hour, day and month you want them occur. As this is a re-occurring event, you can choose which time, date, and day of the week the backup will be made.

## Debugging Your Internal RSYNC Setup

As with network-based RSYNC processes, if you backups are scheduled to happen once a day, it can be a long process to verify that the backups are occurring. There are a couple of things you can do to ensure that your RSYNC configuring is correct.

Making sure the backup works correctly is to schedule the first backup for a few minutes from now. Then, when you know the backup is working correctly, you can schedule your desired backup time.

You can also check the RSYNC activity in a log file. To do this, go to **Diagnostics: Logs** and click on the **RSYNCD** tab. By default, the latest entries are shown at the bottom on the log. Look for something similar to this:

**Mar 4 14:48:00 root: Start of local RSYNC from /mnt/store2/ to /mnt/backup/**

**Mar 4 14:48:00 root: End of local RSYNC synchronization from /mnt/store2/ to /mnt/backup/**

To distinguish this from network-based RSYNC activity, notice that the first line says **Start of local RSYNC** and then lists the source and destination. Unfortunately, there isn't much more information, but you can see at what time the synchronization finished. In this particular example, it was in under 1 second, but there wasn't any synchronization to perform and it had a limited number of files.

# Mirroring vs Conventional Backups

If you use RSYNC (or RAID), do you need to make other types of backup? This is an important question and the simple answer is yes, you do. The primary gain of using RAID or mirrored backups (via RSYNC) is that you are protected from hardware failure. In the RAID scenario, if a disk fails, the server continues and after the disk is replaced, the system continues as before and no data is lost. When mirroring the data on another server, if the primary server fails, the data is intact on the other server.

However, RAID and RSYNC don't protect you from human error. The scenario goes like this: you have an important document that someone manages to delete. The only copy of that document was on the FreeNAS server. If you are using RAID, then the file has been deleted on all the disks in the RAID set at the moment it was deleted. If you are using mirroring, then you have a small window until the mirrored copy will also be deleted. There is no history with these types of backups, no way to go back in time to find a certain version of a file or a file that was deleted.

An offline backup to a tape or optical disk has the advantage that once the backup is made, the tape or disk can be stored on a shelf (or better still in a safe) and when you need that backup in 1 month or 1 years time, it will still be there along with the data on it (assuming you verified the backup when it was made).

Along with RAID and RSYNC mirroring, you should consider making other types of offline backup to tape or disk similar to the procedure described in the section *Backup your FreeNAS using Windows XP's built-in Backup utility* earlier in this chapter. There are many commercial and freeware programs that can help you do this.

# Summary

In this chapter, we have looked at the different backup strategies that exist for your FreeNAS server. Being network orientated, backups must also be performed over the network. We looked at how Windows XP can be used to back up the server along with using RSYNC to create copies of the data on a remote server and on another hard drive.

In the next chapter, we shall look at advanced system configuration.