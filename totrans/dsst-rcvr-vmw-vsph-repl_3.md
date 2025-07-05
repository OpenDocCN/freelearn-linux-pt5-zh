# Chapter 3. Testing and Performing a Failover and Failback

In the previous chapter, we learned how to create Protection Groups and Recovery Plans. As discussed, Recovery Plans are nothing but a previously created workflow for the recovery of a failed site. In this chapter, we will learn how to test Recovery Plans that are already created, how to use them to perform a Failover, panned migration, reprotect, and Failback.

The following is a list of topics that will be covered in this chapter:

*   Testing a Recovery Plan
*   Performing a Planned Migration
*   Performing a disaster recovery (Failover)
*   Reprotecting a site
*   Failback to the protected site
*   Configuring VM recovery properties

# Testing a Recovery Plan

A Recovery Plan should be tested for its readiness to make sure that it would work as expected in the event of a real disaster. Most organizations periodically review and update their recovery runbook to make sure that they have an optimized, working plan for a recovery.

With SRM, the testing of a Recovery Plan can now be automated. It is important to understand the workflow involved in testing a Recovery Plan before we delve into details of what really happens in the background.

The following steps will guide you through the procedure for testing an already-existing Recovery Plan:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Click on the Recovery Plan that you want to test and click on the **Test** toolbar item to bring up the **Test** wizard, as shown in the following screenshot:![Testing a Recovery Plan](img/6442EN_05_01.jpg)
4.  As shown in the following screenshot, the first screen of the wizard will indicate which of the sites have been designated as the protected and recovery sites, the site connection status, and the number of VMs protected:![Testing a Recovery Plan](img/6442EN_05_02.jpg)

    By default, the storage option **Replicate the recent changes to the recovery site** is selected. I would recommend not deselecting this option because we replicate the recent changes during a Planned Migration. So, it is important that the ability of the array to respond to a nonscheduled replication request is tested. However, we might not need to do this if the replication is synchronous. Click on **Next** to continue.

5.  The next screen will summarize the selected options as shown in the following screenshot. Review them and click on **Start** to initiate the test:![Testing a Recovery Plan](img/6442EN_05_03.jpg)
6.  You should now see a **Test Recovery Plan** task in the **Recent Tasks** pane. Navigate to the **Recovery Steps** tab to watch the progress of the test as shown in the following screenshot:![Testing a Recovery Plan](img/6442EN_05_04.jpg)
7.  Once the test completes successfully, you will see the following **Test Complete** banner appear in the **Summary** tab of the Recovery Plan:![Testing a Recovery Plan](img/6442EN_05_05.jpg)

## How does a test work?

The testing of a Recovery Plan is done in such a way that it doesn't affect the current operations, which include replication schedules and the actual replicas, or the protected virtual machines. In this section of the chapter, we will learn how this is achieved.

The following figure shows an overview of the steps involved during the testing of a Recovery Plan:

![How does a test work?](img/6442EN_05_06.jpg)

When you initiate a test, SRM instructs the **Storage Replication Adapter** to execute a replication cycle to replicate the latest changes to the replica LUN at the recovery site. However, this will only happen if you had chosen to leave the default option to replicate recent changes checked. You wouldn't need to replicate the recent change when the replication is synchronous, as the replica would already have the latest change. Refer to your replication vendor documentation for more information.

Once the replication is complete, it then needs to find a way to present the replica LUN's data to the recovery ESXi hosts so that the virtual machines can be powered on. This is achieved differently by different storage arrays. The most common methodology is to create a writable snapshot of the replica LUN, and then present the snapshot to the recovery ESXi hosts. The hosts will subsequently scan their HBAs to detect the VMFS volumes on the LUN.

Before the snapshot is presented to the ESXi hosts, SRM needs to make sure that there is enough room (compute capacity) at the recovery site to power on the recovered VMs. To make room for the VMs, SRM could power off the noncritical VMs (*if included in the Recovery Plan*) and also power up the ESXi hosts that were put into the standby mode (if any) by the **Distributed Power Management** (**DPM**).

### Note

The noncritical VMs that SRM chooses to suspend are those that were marked as noncritical for the Recovery Plan, using the **Add Non-Critical VM** option.

To power on the recovered VMs, they have to be registered to the recovery site's vCenter Server. This is achieved by replacing the shadow VM entries with the entries corresponding to the recovered VMs. Keep in mind that the shadow VMs are mere inventory objects and that they do not have any VMDKs mapped to them.

The VMs are then configured to connect to the test network. The test network can either be a port group that you created for the test or an auto bubble network created on a new vSphere Standard Switch with no physical uplinks.

The following screenshot shows a vSphere **Standard Switch** and a **Port Group** that was automatically created during a test for the auto bubble network:

![How does a test work?](img/6442EN_05_07.jpg)

Once the VMs have been configured to connect to the test network, they are powered on. Keep in mind that the testing of a Recovery Plan does not affect the power state of the protected virtual machines at the protected site.

## Performing the cleanup after a test

We know from the previous section that during the course of the testing of a Recovery Plan, SRM executes the creation of certain elements to enact a disaster recovery in a manner that will not affect the running environment. Hence, the changes made and the objects created are temporary and have to be cleaned up after a successful test. Fortunately, this is not a manual process either. SRM provides an automated method to perform a cleanup.

The following actions will occur during a cleanup:

*   The ESXi hosts will be put back into the DPM standby mode
*   The Recovery VMs will be powered off
*   The Suspended noncritical VMs will be powered on
*   The inventory entries of the Recovery VMs will be replaced with their corresponding Shadow VM entries
*   The VMFS volume will be unmounted
*   The LUN device will be detached
*   The storage initiators and Refresh Storage System will be rescanned
*   The writable snapshot that was created will be deleted
*   The Port Group and the vSwitch that were created for the bubble network will be removed

The following procedure will guide you through the steps required for the cleanup:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Select the Recovery Plan with the status **Test Complete**.
4.  Click on the **Cleanup** item in the toolbar, as shown in the following screenshot, to bring up the cleanup wizard:![Performing the cleanup after a test](img/6442EN_05_08.jpg)
5.  In the cleanup wizard, the details regarding the current protected and recovery sites, their connection status, and the number of protected VMs are displayed. Note that the **Force Cleanup** option is grayed out. This option will only be available if the cleanup operation attempt has failed during the previous attempt. Click on **Next** to continue.
6.  The next screen will summarize the cleanup options selected. Click on **Start** to initiate the cleanup.
7.  The **Recent Tasks** pane should show the **Cleanup Test Recovery** task as successfully completed.

# Performing a Planned Migration

VMware SRM can be used to migrate your workload from one site to another. A Planned Migration is done when the protected site is available and is running the virtual machine workload.

There are many use cases, of which the following two are prominent:

*   When migrating your infrastructure to a new hardware
*   When migrating your virtual machine storage from one array to another

### Note

A Planned Migration will replicate the most recent changes with the help of storage replication. This is not optional.

The following procedure will guide you through the steps required to perform a Planned Migration:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Select the Recovery Plan that was created for the Planned Migration and click on the **Recovery** toolbar item, as shown in the following screenshot, to bring up the recovery wizard:![Performing a Planned Migration](img/6442EN_05_09.jpg)
4.  As shown in the following screenshot, the first screen will seek a **Recovery Confirmation**. The **Recovery Type** will be preselected as **Planned Migration**. You should acknowledge the recovery confirmation to proceed further. Click on **Next** to continue.![Performing a Planned Migration](img/6442EN_05_10.jpg)
5.  The next screen will summarize the wizard options that were selected. Click on **Start** to initiate the migration.
6.  The **Recent Tasks** pane should now show the **Failover Recovery Plan** task as successfully completed.

The Planned Migration will not proceed further if any of the recovery steps fail. However, when you re-attempt the Planned Migration, it would resume the operation from the step at which it failed. This enables you to fix the problem and resume from where it failed, saving a considerable amount of time.

The following flowchart shows the logical sequence of events that would occur during the course of a Planned Migration:

![Performing a Planned Migration](img/6442EN_05_11.jpg)

# Performing a disaster recovery (Failover)

A Failover is performed when the protected site becomes fully or partially unavailable. We use a Recovery Plan that is already created and tested to perform the Failover. Keep in mind that SRM does not automatically determine the occurrence of a disaster at the protected site; hence, a recovery is always to be manually initiated.

The following steps show how to perform a Failover:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Select the Recovery Plan that was created for the disaster recovery and click on the **Recovery** toolbar item to bring up the recovery wizard.
4.  In the recovery wizard, as shown in the following screenshot, agree to the **Recovery Confirmation**, set the **Recovery Type** as **Disaster Recovery**, and click on **Next** to continue:![Performing a disaster recovery (Failover)](img/6442EN_05_12.jpg)
5.  The next screen will summarize the selected wizard options. Click on **Start** to perform the recovery.
6.  The **Recovery Steps** tab of the Recovery Plan will show the progress of each of the steps involved.
7.  Once the Failover is complete, the status of the Recovery Plan should read **Recovery Complete**.

The recovery steps involved in a disaster recovery (Failover) is the same as in that of a Planned Migration, except for the fact that SRM ignores any unsuccessful attempts to pre-synchronize the storage or shut down the protected virtual machines.

# Forced Recovery

Forced Recovery is used when the protected site is no longer operational enough to allow SRM to perform its tasks at the protected site before the Failover.

For instance, there is an unexpected power outage at the protected site causing not just the ESXi hosts but also the storage array to become unavailable. In this scenario, SRM cannot perform any of its tasks, such as shutting down the protected VMs or replicating the most recent storage changes (if the replication is asynchronous), at the protected site.

## Enabling Forced Recovery for a site

Forced Recovery is not enabled by default, but it can be enabled at the site's advanced settings. To do so, perform the following steps:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Sites** on the left pane.
3.  Right-click on the site and click on **Advanced Settings**.![Enabling Forced Recovery for a site](img/6442EN_05_15.jpg)
4.  In the **Advanced Settings** windows, select the category **recovery** from the left pane.
5.  Select the checkbox against the **recovery.forceRecovery** setting, as shown in the following screenshot, and click on **OK** to enable Forced Recovery:![Enabling Forced Recovery for a site](img/6442EN_05_16.jpg)

## Running Forced Recovery

Running Forced Recovery will skip all the steps that otherwise should have been performed against the protected site. You should use Forced Recovery only during circumstances where the protected site is completely down, leaving no connectivity to either the ESXi hosts or the storage array. If Forced Recovery were to be executed while the protected site is still online and available, then it would leave the virtual machines running at both the protected and recovery sites, causing a split-brain condition for SRM. Furthermore, if the array-based replication was asynchronous, then the chances are that the virtual machines that were started at the recovery site are running old data compared to the ones in the protected site. So, it very important that you take caution before you plan to execute Forced Recovery.

The following steps show how Forced Recovery is executed:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Right-click on the Recovery Plan that you want to run and click on **Recovery**.
4.  In the recovery wizard, select the **I understand that this process will permanently alter the virtual machines and infrastructure of both the protected and recovery datacenters** checkbox.
5.  As shown in the following screenshot, select the **Recovery Type** as **Disaster Recovery**, select the checkbox **Forced Recovery – recovery site operations only**, and click on **Next**:![Running Forced Recovery](img/6442EN_05_17.jpg)
6.  You will be prompted to confirm the **Forced Recovery**. Click on **Yes** to confirm.![Running Forced Recovery](img/6442EN_05_18.jpg)
7.  Review the operation summary and click on **Start** to initiate the Forced Recovery.

# Reprotecting a site

After you Failover the workload from a protected site to the recovery site, the recovery site has no protection enabled for the new workload that it has begun hosting. SRM provides a method to enable protection of the recovery site. This method is called **Reprotect**.

A Reprotect operation will reverse the direction of the replication, thus designating the recovery site as the new protected site. The Reprotect operation can only be done on a Recovery Plan with the **Recovery Complete** status. Also, keep in mind that a Reprotect operation can only be executed when you have repaired the failed site and made it available to become a recovery site.

For instance, let's assume that SITE-A and SITE-B are the protected and recovery sites, respectively. If workload at SITE-A were failed over to SITE-B, then to Reprotect SITE-B, SITE-A should be made accessible. This would mean fixing the problems that caused the failure at SITE-A.

The following steps show how to perform the Reprotect operation:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** in the left pane.
3.  Select the **Recovery Plan** with the **Recovery Complete** status, as shown in the following screenshot, and click on the toolbar item **Reprotect**:![Reprotecting a site](img/6442EN_05_13.jpg)
4.  In the **Reprotect** wizard screen, agree to the **Reprotect Confirmation** and click on **Next** to continue:![Reprotecting a site](img/6442EN_05_14.jpg)
5.  In the next screen, click on **Start** to begin the Reprotect operation.
6.  You should see a progressing **Reprotect Recovery Plan** task in the **Recent Tasks** pane. Also, the **Recovery Steps** tab will show the progress of every step involved in the Reprotect operation.
7.  The status of the Recovery Plan after a successful Reprotect operation should read **Ready**.

# Failback to the protected site

In a scenario where, after a Failover, the original protected site is fixed and is made available to host the virtual machine workload, you can use SRM to automate a Failback.

The Failback, although automated, is a two-step process, which is as follows:

*   Step 1 will be to perform a Reprotect operation. Read the *Reprotecting a site* section in this chapter to learn how to perform a Reprotect operation.
*   Step 2 will be to perform a Failover. Read the *Performing a disaster recovery (Failover)* section in this chapter to learn how to perform a Failover.

# Configuring VM recovery properties

The VM recovery properties help in further customizing the recovery procedure at a per-VM level. Although these properties are only available via a Recovery Plan, the changes made to these properties are retained for the VM, regardless of the Recovery Plan they would be included in.

The following are the properties that can be set on a protected virtual machine:

*   **IP Settings**
*   **Priority Group**
*   **VM Dependencies**
*   **Shutdown Action**
*   **Startup Action**
*   **Pre-power On Steps**
*   **Post Power On Steps**

Here is how you could get to the **Recovery Properties** of a virtual machine:

1.  Navigate to the vCenter Server's inventory home page and click on **Site Recovery**.
2.  Click on **Recovery Plans** on the left pane.
3.  Select a **Recovery Plan** (**A**) and navigate to its **Virtual Machines** tab (**B**), which should list all the virtual machines that are included in the Recovery Plan.
4.  Select a virtual machine from the list (**C**) and click on **Configure Recovery** (**D**) to bring up the VM recovery properties:![Configuring VM recovery properties](img/6442EN_05_19.jpg)

The VM **Properties** windows will show all the properties available for customization. We will visit each of these properties further in this section.

## IP settings

The IP settings property is a per-vNIC property for the virtual machine that is part of the Recovery Plan. It is used to customize the IP configuration of the virtual machine during a Planned Migration/Failover.

As the settings are per vNIC, you should enable the IP customization for each of the vNIC that you want to supply the IP settings for. It is done by choosing the vNIC from the left pane (**A**) and then selecting the **Customize IP setting during recovery** checkbox (**B**), as shown in the following screenshot:

![IP settings](img/6442EN_05_20.jpg)

The IP setting can be separate for both the protected and recovery sites.

To configure the IP settings for a Recovery from the original recovery site to the original protected site, use the **Configure Protection** option (**P**), and to configure the IP settings for a Recovery from the protected site to the recovery site, use the **Configure Recovery** option (**R**), as shown in the following screenshot:

![IP settings](img/6442EN_05_21.jpg)

When you hit either **Configure Protection** or **Configure Recovery**, a new IP settings window is shown. You can either choose to use DHCP, IPv4, or IPv6\. It also has options to supply the DNS Server details. You could also choose to fetch the current IP configuration of the VM using the **Retrieve** button. For the retrieve operation to work, you need VMware Tools installed and running in the VM:

![IP settings](img/6442EN_05_22.jpg)

Once the settings have been supplied, click on **OK** to save the IP customization.

## Priority Group

Priority Groups are used to set the startup order of the virtual machines. SRM uses five Priority Groups numbered from 1 to 5\. Their priority and startup order are shown in the following table:

| Priority Group | Priority | Startup order |
| --- | --- | --- |
| **Priority Group 1** | Highest | First |
| **Priority Group 2** | Higher than group 3 | Before group 3 |
| **Priority Group 3** | Higher than group 4 | Before group 4 |
| **Priority Group 4** | Higher than group 5 | Before group 5 |
| **Priority Group 5** | Lowest | Last |

As shown in the following screenshot, the VMs of the lowest numbered Priority Group are started first:

![Priority Group](img/6442EN_05_23.jpg)

By default, every VM is in **Priority Group 3**. During a Failover, SRM will wait for all the VMs in a **Priority Group** to Failover successfully or fail to Failover before it attempts to power on VMs from a group with lower priority. For instance, SRM will wait for all the VMs in **Priority Group 2** to Failover before it can attempt the Failover of the VMs in **Priority Group 3**. This is because **Priority Group 2** has a higher priority than **Priority Group 3**.

## VM Dependencies

So, what happens if all the VMs running the services of the three-tier application are in the same Priority Group? Can we configure a startup order within a Priority Group? The answer is yes. This is where VM Dependencies come in handy.

Let's consider a three-tier application, which has a database, a server component, and the user interface component hosted on three different VMs. Now, in this case the server component is dependent on the availability of the database, and the user interface component is dependent on the server component. This means the database VM should start first, then the VM hosting the service component, and finally the VM hosting the user interface component. Such a startup order can be achieved using the VM Dependencies Recovery Property.

The following steps show how a VM Dependency startup order is created:

1.  Get to the Recovery Properties of the virtual machine.
2.  Select the **VM Dependencies** property from the left pane (**A**).
3.  Click on **Add** to browse and add VMs to the list (**B**). Only the VMs from the Protection Groups that are part of the Recovery Plan can be added as shown in the following screenshot:![VM Dependencies](img/6442EN_05_24.jpg)
4.  Once the virtual machines are added, click on **OK** to save the settings. Keep in mind that the virtual machines are started in the order of their appearance in the list. So make sure that you plan to add the VMs in the order in which you want them to start. You can also remove a VM from the list by hitting the **Remove** button.

### Note

Note that the dependencies between VMs of different Priority Groups are ignored by SRM. Also, the dependencies are not mandatory rules; hence, they wouldn't stop the Recovery Plan from continuing. If a VM dependency fails, it throws a warning and proceeds with the execution of the Recovery Plan.

## The Shutdown Action

The Shutdown Action VM Recovery property will let you choose whether a virtual machine at the protected site will be attempted to gracefully shut down or powered off during a Planned Migration or disaster recovery. It also lets you set the amount of time SRM waits on VMware Tools to shut down the virtual machine before it issues a power off. The default timeout value is **5** minutes:

![The Shutdown Action](img/6442EN_05_25.jpg)

A disaster recovery will power off the VM after the timeout, but a Planned Migration will not proceed further if the VM cannot be gracefully shut down.

## The Startup Action

The Startup Action property will let you choose whether or not to power on a virtual machine at the recovery site during a test or a Recovery operation. By default, after a virtual machine is powered on at the recovery site, SRM waits for 5 minutes to determine whether the virtual machine tools have started. If it sees no response from VMware Tools, it marks a failure of that task and proceeds with the Recovery operation. However, the Recovery operation will have an **Incomplete Recovery** status. You can also add further delay before any of the dependent VM are started or before the execution of any **Post Power On Steps**. This is commonly used to give the service running in the VM an additional time to start.

![The Startup Action](img/6442EN_05_26.jpg)

## The Pre-power On Steps and Post Power On Steps properties

The **Pre-power On Steps** and **Post Power On Steps** are **VM Recovery Properties** that allow the insertion of additional steps into the Recovery Plan.

With the **Pre-power On Steps** property, you can create the following type of steps:

*   A Windows batch command on the SRM Server
*   A message prompt, which the user/administrator should dismiss before the Recovery Plan can begin its execution

With the **Post Power On Steps** property, you can create the following type of steps:

*   A command on the Recovered virtual machine
*   A Windows batch command on the SRM Server

Creating a message prompt is straightforward. It is done in the following way:

1.  Select the **Pre-power On Step** / **Post Power On Step** property from the left plane and click on **Add**.
2.  In the **Add Pre-power On Step** / **Add Post Power On Step** window, select the **Type** as **Prompt (requires a user to dismiss before the plan will continue)**.
3.  Supply a **Name** and **Content** and click on **OK** to save the setting.

To create a Windows batch command on the SRM Server, perform the following steps:

1.  Select the **Pre-power On Step** / **Post Power On Steps** property from the left plane and click on **Add**.
2.  In the **Add Pre-power On Step** / **Add Post Power On Step** window, select **Command on SRM Server** as the **Type**.
3.  Supply a **Name** of the script and the command script in the **Content** section. Type the actual command. For instance, to run a batch script in `D:\demoscript.bat`, include the following command:

    ```
    c:\windows\system32\cmd.exe /c d:\demoscript.bat

    ```

    ### Tip

    For `cmd.exe`, always mention the absolute path `c:\windows\system32\cmd.exe`

The default timeout for a batch command is 5 minutes. If the batch file doesn't finish executing within 5 minutes, then the execution of the Recovery Plan will stop with an error indicating the same.

# Summary

In this chapter, we learned how to use the SRM to orchestrate the testing of a Recovery Plan, performing Planned Migrations and recovery. We also learned how to Failback to a designated protected site after a Failover.