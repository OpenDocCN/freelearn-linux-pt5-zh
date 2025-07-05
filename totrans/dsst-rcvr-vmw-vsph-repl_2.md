# Chapter 2. Creating Protection Groups and Recovery Plans

In the previous chapter, we learned how to install SRM and configure it and also how to lay the groundwork for an SRM-protected environment. We learned how to create resources, folder, and network mappings and configure placeholder datastores and array managers.

In this chapter, we will cover the following topics:

*   Creating Protection Groups
*   Creating Recovery Plans

Once you have done the groundwork required to form an SRM-protected environment, the next step is to grant protection to the virtual machines. Before we delve into the procedural steps involved in protecting virtual machines, it is very important to understand a couple of basic concepts such as datastore and Protection Groups.

# Datastore groups

A datastore group is a container that aggregates one or more replication-enabled datastores. The datastore groups are created by SRM and cannot be manually altered. A replication-enabled datastore is a datastore who's LUN has a replication schedule enabled at the array.

![Datastore groups](img/6442EN_04_01.jpg)

A datastore group will contain only a single datastore if the datastore doesn't store files of virtual machines from other datastores. See the preceding single-datastore datastore group conceptual diagram.

A datastore group can also contain more than one datastore. SRM aggregates multiple datastores into a single group if they have virtual machines whose files are distributed onto these datastores. For example, if VM-A has two VMDKs placed on datastores Datastore-M and Datastore-N each, and then both these datastores become part of the same datastore group. These datastore groups further aid in the creation of Protection Groups.

# Protection Groups

Unlike **vSphere Replication**, SRM cannot enable protection on individual virtual machines. All the virtual machines that are hosted on the datastores in a datastore group are protected. Meaning, with SRM, protection is enabled at the datastore group level. This is because, with an array-based replication, the LUNs backing the datastores are replicated. The array doesn't know which VMs are hosted on the datastore. It just replicates the LUN, block by block. So, at the SRM layer, the protection is enabled at the datastore level. In a way, a Protection Group is nothing but a software construct to which datastore groups are added, which in turn includes all the VMs stored on them in the Protection Group.

When creating a Protection Group, you will have to choose the datastore groups that will be included. Keep in mind that you cannot individually select the datastores in a datastore group. If it were ever allowed to do so, then you will have virtual machines with not all of its files protected. Let's assume that you have a virtual machine, VM-A, with two disks (VMDK-1 and VMDK-2) placed on two different datastores. Let's also say VMDK-1 is on Datastore-X and VMDK-2 is on Datastore-Y. When creating a Protection Group, if you were allowed to select the individual datastores and if you choose only one of them, then you will leave the remaining disks of the VM unprotected. Hence, SRM doesn't allow selecting individual datastores from a datastore group as a measure to prevent such a scenario. The following diagram shows the modified conceptual structure of the datastore group:

![Protection Groups](img/6442EN_04_02.jpg)

Here, even though we have both the datastore groups included in the same Protection Group, **Protection Group-A**, it is possible to form separate Protection Groups for each of the datastore groups.

### Tip

Note that a datastore group cannot be a part of two Protection Groups at the same time.

# Creating a Protection Group

A Protection Group is created in the SRM UI at the protected site. The following procedure will guide you through the steps required to create a Protection Group:

1.  Navigate to the vCenter Server's inventory home and click on **Site Recovery**.
2.  Click on **Protection Groups** in the left pane of the window.
3.  Click on **Create Protection Group** to bring up the **Create Protection Group** wizard as shown in the following screenshot:![Creating a Protection Group](img/6442EN_04_03.jpg)
4.  In the wizard, make sure the site that you want to protect is selected.

    Keep in mind that the local site (the one that you're locked into) is always selected as the **Protected Site**, as shown in the following screenshot. In case you are using the SRM UI from the **Recovery Site**, you have to manually select the protected site. If the wizard shows more than one array pair, make sure you select the correct one to proceed.

    Click on **Next** to continue.

    ![Creating a Protection Group](img/6442EN_04_04.jpg)
5.  On the next screen, choose a datastore group that you would like to protect. When you select a datastore group, the bottom pane will list all the VMs hosted on the datastores in the group. You cannot individually select the VMs though. Although I have selected only a single datastore group, we can select multiple datastore groups to become part of the Protection Group.

    Click on **Next** to continue as shown in the following screenshot:

    ![Creating a Protection Group](img/6442EN_04_05.jpg)
6.  In the next screen, provide the **Protection Group Name** and an optional description, and click on **Next** to continue.

    The **Protection Group Name** can be any name that you would prefer to identify the Protection Group with. The common naming convention is to indicate the type or purpose of the VMs. This is because, in most cases, the VMs serving a common purpose or VMs of the same type/priority in an SRM-protected environment are segregated onto separate datastores to aid easier management of the Protection Groups.

7.  For instance, if you were protecting the SQL Server VMs, then you might name the Protection Group as `SQL Server Protection Group`; or, if it were to be a set of hyphenate VMs, you may name it as `High Priority VMs Protection Group`.
8.  On the **Ready to Complete** screen, as shown in the following screenshot, review the wizard options selected and click on **Finish** to create a Protection Group:![Creating a Protection Group](img/6442EN_04_06.jpg)

## So, what exactly happens when you create a Protection Group?

When you create a Protection Group, it enables protection on all the VMs in the chosen datastore group and creates shadow VMs at the recovery site. In detail, this means that at the protected site vCenter Server, you should see a **Create Protection Group** task complete; subsequently a **Protect VM** task completes successfully for each of the VMs in the Protection Group. See the following screenshot for reference:

![So, what exactly happens when you create a Protection Group?](img/6442EN_04_07.jpg)

At the recovery site of the vCenter Server, you should see the **Create Protection Group**, **Protect VM** (one for each VM), **Create virtual machine** (one for each VM), and **Recompute Datastore Groups** tasks completed successfully.

![So, what exactly happens when you create a Protection Group?](img/6442EN_04_08.jpg)

As shown in the following screenshot, the shadow VMs appear in the vCenter Server's inventory at the recovery site:

![So, what exactly happens when you create a Protection Group?](img/6442EN_04_09.jpg)

As they are solely placeholders, you cannot perform any power operations on it. There are other operations that are possible but are not recommended. Hence, a warning will be displayed, requesting a confirmation, as shown in the following screenshot:

![So, what exactly happens when you create a Protection Group?](img/6442EN_04_10.jpg)

The placeholder datastores will only have the configuration file (`.vmx`), teaming configuration file (`.vmxf`), and a snapshot metadata file (`.vmsd`) for each VM.

![So, what exactly happens when you create a Protection Group?](img/6442EN_04_11.jpg)

These files will be automatically deleted when you delete the Protection Group.

# Recovery Plans

Recovery Plans are created at the recovery site so that they are accessible and can be run from the recovery site when there is a disaster at the protected site. A Recovery Plan is executed to Failover the virtual machine workload that was running at the protected site to the recovery site. It can also be used to perform Planned Migrations. A Recovery Plan is a series of configuration steps that has to be performed to Failover the protected virtual machines to the recovery site.

### Tip

A Recovery Plan should be associated with at least one Protection Group.

# Creating a Recovery Plan

Once you have Protection Groups created, the next step would be to create a Recovery Plan for these Protection Groups. The Recovery Plan should be created at the recovery site SRM. This is because, in the event of a disaster, the protected site may become inaccessible. Hence, for very obvious reasons, a Recovery Plan is always created at the recovery site. The following steps show you how to create a Recovery Plan:

1.  Navigate to the vCenter Server's inventory home and click on **Site Recovery**.
2.  Click on **Recovery Plans** [**A**] on the left pane.
3.  Click on **Create Recovery Plan** [**B**] to bring up the **Create Recovery Plan** wizard as shown in the following screenshot:![Creating a Recovery Plan](img/6442EN_04_12.jpg)
4.  In the **Create Recovery Plan** wizard, select the **Recovery Site** and click on **Next** to continue. If the Recovery Plan wizard is initiated at a site, then the wizard will select the other site in the site pair as the recovery site. For example, if you were to initiate the Recovery Plan wizard at SITE-A, then the wizard will autoselect SITE-B as the recovery site and vice versa. Refer to the following screenshot:![Creating a Recovery Plan](img/6442EN_04_13.jpg)
5.  As shown in the following screenshot, select the Protection Group that you would like to use and click on **Next** to continue:![Creating a Recovery Plan](img/6442EN_04_14.jpg)
6.  In the next wizard screen, click on **Test Networks**. The test networks are set to **Auto** by default. The Auto networks are isolated bubble networks and don't connect to any physical network. They are used when testing a Recovery Plan. We will discuss more about how to test a Recovery Plan and the use of bubble networks in [Chapter 3](ch03.html "Chapter 3. Testing and Performing a Failover and Failback"), *Testing and Performing a Failover and Failback*. So unless you have manually created an isolated test network port group at the recovery site, you can leave it at the **Auto** setting. Click on **Next** to continue:![Creating a Recovery Plan](img/6442EN_04_15.jpg)
7.  In the next screen, enter a **Recovery Plan Name** and an optional **Description** and click on **Next** to continue. The Recovery Plan name can be any name of your choice.![Creating a Recovery Plan](img/6442EN_04_16.jpg)
8.  In the **Ready to Complete** window, click on **Finish** to create the Recovery Plan.![Creating a Recovery Plan](img/6442EN_04_17.jpg)
9.  You should see the **Create Recovery Plan** task completed successfully in the **Recent Tasks** pane.

# Summary

In this chapter, we learned how to create Protection Groups and create Recovery Plans for them. In the next chapter, we learn how to test the Recovery Plans, execute a Failover, reprotect, and a Failback.