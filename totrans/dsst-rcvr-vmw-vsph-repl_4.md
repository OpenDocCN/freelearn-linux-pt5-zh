# Chapter 4. Deploying vSphere Replication 5.5

In this chapter, you will learn the following topics:

*   New features in vSphere Replication 5.5
*   Understanding the vSphere Replication architecture
*   Downloading the vSphere Replication bundle
*   Deploying the vSphere Replication Appliance
*   Setting up the VRA hostname and a VRM site name for the VRA
*   Configuring a SQL database for VRMS
*   Deploying a vSphere Replication Server
*   Registering vSphere Replication Servers

# Introduction

Most organizations will have a DR plan in place, regardless of whether it is a large enterprise or a small or medium business. VMware Site Recovery Manager leveraging an array-based replication is a very effective DR solution. However, an array-based replication can be a costly solution for some businesses, especially the small and medium businesses. VMware's proprietary replication engine called vSphere Replication offers a very cost-effective DR solution without the need for investments on storage replication technologies.

One of the advantages for businesses using vSphere Replication is the fact that the replication can be managed without the need for an SRM license. The vSphere Web Client GUI provides an interface to configure and manage replication on virtual machines.

In this chapter, we learn what vSphere Replication actually is, its architecture, and how it can be deployed in your vSphere environment.

vSphere Replication is a replication engine that can be leveraged to configure replication on individual virtual machines. It can replicate a virtual machine and its disks from one location to another without the need to incorporate an expensive array-based replication. What it really does is provide a mechanism to replicate a virtual machine using the existing Ethernet infrastructure and recover them when there is a need.

The concept of vSphere Replication was introduced with VMware Site Recovery Manager Version 5.0\. At that time, vSphere Replication was not a standalone product. vSphere v5.1 onwards, vSphere Replication became a standalone product and directly integrates into the vSphere platform. It registers itself as a plugin to the vCenter Server. All of the replication and recovery operations are done using the vSphere Web Client. It is included in Essentials Plus and higher license models.

It is storage agnostic, which means that a virtual machine or its disk files can be replicated to a datastore, regardless of it being a VMFS volume or an NFS mount. For instance, if the virtual machine that you want to protect by enabling replication is located on a VMFS volume, then the replica can be either on another VMFS volume or an NFS mount. This stands true both ways.

### Note

vSphere Replication can protect a maximum of 500 virtual machines.

VMware Site Recovery Manager can be configured to leverage the vSphere Replication engine to perform recovery tests, Failovers, Planned Migration, Failback, and so on.

## New features of vSphere Replication 5.5

vSphere Replication was greatly improved with the release of the Version 5.5\. Some of the new features and improvements are explored in the following sections.

### Multiple points-in-time replication snapshots (historical retention)

You can now have multiple points-in-time snapshots (a maximum of 24 snapshots) of your replicated data. The number of snapshots and the retention period can be specified when configuring replication for a virtual machine.

### Multiple vSphere Replication Server appliances per vCenter

With vSphere 5.5, you can now deploy up to 10 vSphere Replication Server appliances per vCenter in a standalone mode.

### Note

Keep in mind that a **vSphere Replication Server** (**VR Server**) is not a **vSphere Replication Appliance** (**VRA**). We will discuss more about the differences in the architecture section.

By standalone mode, we mean that the replication is managed without the use of **Site Recovery Manager** (**SRM**). Prior to Version 5.5, the number of VR appliances that can be deployed in the standalone mode were limited to only one and with SRM, it was 10\. Deploying multiple VRs can have several use cases. It is not mandatory to have a vCenter at a datacenter to deploy a VR appliance.

A VR appliance can simply be deployed at a remote datacenter to handle the replication traffic sent to it and the writing of the data onto a chosen datastore at that site. This new ability drives several use cases. For instance, if you were to maintain a remote datacenter only to hold replicated data, then you don't necessarily have to deploy a vCenter Server at that site. You can manage the datacenter with an existing vCenter Server, and a VR appliance deployed on one of ESXi hosts at the remote site should do the job of collating the replication traffic and writing it onto the intended datastore.

### Storage vMotion of protected VMs

You now perform storage vMotion of a replicated, protected VM to any datastore. However, this can only be done with VMs at a protected site and not with the replicas at the recovery site. The replicas are not registered to any of the ESXi hosts at the target site.

### Storage profiles and vSAN compatibility

VM storage profiles can now be used to with vSphere Replication. You can also now use vSphere Replication with vSAN, but there are several caveats. However, vSAN itself is an experimental feature with vSphere 5.5.

For more information regarding the use of vSphere Replication with vSAN, refer to page 51 of the vSphere Replication Administration guide at [http://bit.ly/VRAdminGuide](http://bit.ly/VRAdminGuide).

### Performance improvement

VMware claims that the replication is now faster. It uses a new TCP stack optimization for latency handling. The following are the two aspects:

*   It implements buffered I/O for improved NFC write performance
*   Disk blocks sent across are coalesced or aggregated, and it is only then the disk is opened for the `WRITE` operation to be performed

### Note

There are several other improvements. For more information, read the What's New section of the vSphere Replication 5.5 Release Notes at [http://bit.ly/VR_WhatIsNew](http://bit.ly/VR_WhatIsNew).

This performance improvement, however, does not affect the stated RPO (which should be greater or equal to 15 minutes and less than or equal to 24 hours) for vSphere Replication. The performance improvement is for the data transfer and handling. This means it can now handle more replications, hence more data.

## Understanding the vSphere Replication architecture

vSphere Replication is VMware's hypervisor-based replication solution. Unlike the array-based replication, the data is replicated over the network using the VMware **Network File Copy** (**NFC**) protocol. VMware NFC is a proprietary VMware protocol that is used to transfer disk (VMDK) blocks between ESXi hosts.

The vSphere Replication architecture involves the following components:

*   One or more instances of vCenter Servers managing the protected and the recovery sites
*   A **vSphere Replication Appliance** (**VRA**) deployed at the protected site
*   A VRA or a vSphere Replication Server at the recovery site
*   The VRM plugin for the vSphere Web Client
*   The vSphere Replication agent that runs on every ESXi host

A vSphere Replication Appliance is a vApp comprising of the **vSphere Replication Management Server** (**VRMS**) and a VR Server. For vSphere Replication to work, you will need a VRMS at the protected site and a VR Server at the target recovery site, be it local or remote; and there can be only one VRMS per vCenter Server. Refer to the following diagram:

![Understanding the vSphere Replication architecture](img/6442EN_01_29.jpg)

The recovery site is where you plan to maintain the replicas of the virtual machines from the protected site. It is the site to which you will be sending the replication traffic. The vSphere Replication Appliances provides a replication management interface via the vSphere Web Client. It registers itself as a plugin for the vSphere Web Client.

Every ESXi host starting with Version 5.1 has a vSphere Replication agent already built into the VMkernel, thereby removing the dependency on Site Recovery Manager and installing additional packages into each ESXi host.

# Downloading the vSphere Replication bundle

The vSphere Replication Server appliance is available for download as an ISO data file or a compressed ZIP package, both of which contain separate OVFs for vSphere Replication Appliance and add-on servers.

The **vSphere Replication Appliance** (**VRA**) includes a vSphere Replication Management Server and a vSphere Replication Server. It includes a plugin for the vSphere Web Client and also uses a vPostgreSQL embedded database.

To download the ISO or ZIP bundle, use the following steps:

1.  Go to the vSphere downloads page at [www.vmware.com/go/download-vsphere](http://www.vmware.com/go/download-vsphere).
2.  Click on the **Go to Downloads** hyperlink corresponding to **vSphere Replication 5.5** listed under your license model.
3.  Log in to your **My VMware** account when prompted.
4.  Download either the ZIP or ISO package.

# Deploying the vSphere Replication Appliance

The vSphere Replication Appliance should be installed at a site where you have virtual machines that need to be protected. It may or may not be required to be installed on both the protected and recovery sites. You will need VRA to be deployed at the recovery site only if you intend to pair it with the protected site. Refer to the following diagram:

![Deploying the vSphere Replication Appliance](img/6442EN_01_30.jpg)

The pairing is done by adding the recovery site as a target site to the VRMS at the protected site. Read the *Adding a remote site as a target* section in [Chapter 5](ch05.html "Chapter 5. Configuring and Using vSphere Replication 5.5"), *Configuring and Using vSphere Replication 5.5*, for information on how to achieve this.

The total number of the virtual machines that can be protected by vSphere Replication is 500 per site. The limit is per VRMS, and as there can be only one VRMS registered to a site's vCenter, the 500 VM limit cannot be exceeded. If the VRMSs are paired, then the cumulative limit of VMs from both the sites becomes 500 and not 1000\. To deploy the vSphere Replication Appliances, you need the following components:

*   vCenter 5.5
*   ESXi hosts compatible with Version 5.5 of vCenter Server
*   The downloaded bundle for vSphere Replication 5.5

The appliance's OVF can be deployed using the **Deploy OVF Template** wizard. The wizard can be initiated from various levels (vCenter or datacenter or ESXi). We will do it at the vCenter level, which however is not a technical requirement. Refer to the following steps:

1.  Extract (unzip) the downloaded package, or in case you have downloaded the ISO bundle, then mount the ISO to the vCenter Server virtual machine or the machine on which the vSphere Client is being accessed from.
2.  Right-click on the **vCenter Server** and click on **Deploy OVF Template**. Refer to the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_01.jpg)
3.  On the wizard screen, set the source as **Local file** and click on **Browse**, as shown in the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_02.jpg)
4.  Navigate to the extracted bundle folder and then to the `bin` subfolder, as shown in the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_03.jpg)
5.  Select the OVF file `vSphere_Replication_OVF10.ovf` and click on **Open** to make the selection and return to the wizard. Refer to the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_04.jpg)
6.  On the wizard screen, click on **Next** to continue.
7.  The **Review details** screen summarizes the appliance's details. Click on **Next** to continue.
8.  On the **License Agreement** screen, select **Accept** to receive the license and click on **Next** to continue.
9.  Provide a name for the VM, select an inventory destination for it, and click on **Next** to continue, as shown in the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_05.jpg)
10.  Select a compute location for the VM. The compute location could be an ESXi host or a cluster of ESXi hosts. Once a selection has been made, click on **Next** to continue. Refer to the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_06.jpg)
11.  Select the VMDK type and a datastore for the VM and click on **Next** to continue, as shown in the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_07.jpg)
12.  Select a network (port group) for the VM's vNIC, and choose between IPv4 or IPv6, and an IP allocation policy (DHCP/Static). We have selected **Static**, hence we will have to manually specify the DNS Server, the Subnet Mask, and the gateway of the subnet the VM will be a part of. Once done, click on **Next** to continue. Refer the following screenshot:![Deploying the vSphere Replication Appliance](img/6442EN_01_08.jpg)
13.  Set the password and the static IP for the appliance as shown in the following screenshot. Click on **Next** to continue.![Deploying the vSphere Replication Appliance](img/6442EN_01_09.jpg)
14.  The next screen will show the vService binding details. There is nothing to modify here, so click on **Next** to continue.
15.  On the **Ready to complete** screen, review the settings and then click on **Finish** to deploy the appliance VM. You can select the **Power on after deployment** checkbox to start the VM if the deployment is successful.

## How does it work?

Once deployed, the appliance will power on and finish the initial configuration, which includes the configuration of the embedded database and the process of registering VRMS to the vCenter Server.

You can only have a single instance of VRMS registered to a vCenter Server. This means that you can't deploy multiple instances of vSphere Replication Appliance at a site. If you do so, then the appliance will detect that there is another appliance that has already been registered to the vCenter Server and will prompt for an override or a shutdown of the newly deployed appliance. The following screenshot shows an appliance initialization detecting the presence of another VRA:

![How does it work?](img/6442EN_01_10.jpg)

### Tip

If you choose the **Continue** option, you should shut down the already registered VR appliance.

You can, however, deploy multiple instances of the vSphere Replication Server (add-on) server appliance, which doesn't initialize the VRMS component. The add-on server appliance is deployed using a different OVF.

Like with any VMware appliance, the VRS Appliance also has a web-based management interface that can be accessed for appliance-specific configuration tasks. This web interface is called **Virtual Appliance Management Interface** (**VAMI**).

You can use the IP address of the appliance to connect to its management web interface using the URL format: `https://<IP address or FQDN>:5480`.

When you reach the login page for the appliance, log in using the root user and the password you set during the OVF deployment wizard. Once you are in, you will see a **Getting Started** tab, which is a subtab under the **VR** tab. There is nothing much you can do at the **Getting Started** tab. The other subtabs that are available under VR are **Configuration**, **Security**, and **Support**. We do not need to review or change the options under these subtabs unless necessary.

The other main tabs available are **Network**, **Update**, and **System**. These will be covered in the later sections of the chapter, which would require accessing these tabs.

# Setting up the VRA hostname and a VRM site name for the VRA

Although not mandatory, you might need to set a hostname and a target name for the vSphere Replication Appliance that you have deployed.

## The VRA hostname

The hostname for the appliance can be set from the appliance's VAMI. The default hostname post deployment will be `localhost.localdom`. This can, however, be modified.

The following procedure will guide you through the steps required to modify the hostname:

1.  Connect to the VAMI of the appliance. Enter `https://<IP address or FQDN>:5480` as the URL.
2.  Log in using the root user and the password that was supplied during the OVF deployment wizard.
3.  Navigate to the **Address** tab in the **Network** menu.

    ### Tip

    Prior to providing the hostname, it is important to create a new host (*A*) record at the DNS Server. Only then will you be able to connect to the appliance using its hostname.

4.  Provide a **Hostname** at the input box corresponding to it and click on **Save Settings**, as shown in the following screenshot:![The VRA hostname](img/6442EN_01_11.jpg)

## The VRM site name

Every VRM Server registered to a vCenter Server will have a site name. By default, during the initial configuration of the appliance, the address of the vCenter Server to which the VRMS gets registered to is set as the VRM site name. The site name is only a display name; hence, it is not mandatory that you change it. For instance, the VRM site name of a VRMS registered to the protected site vCenter Server can be just called "Protected Site".

The following procedure will guide you through the steps required to modify the VRM site name:

1.  Connect to the VAMI of the appliance by entering `https://<IP address or FQDN>:5480` as the URL.
2.  Log in using the root user and the password that was supplied during the OVF deployment wizard.
3.  Navigate to the **Configuration** tab under the **VR** option.
4.  Modify **VRM Site Name** using the input box corresponding to it and restart the VRM Service by clicking on **Save and Restart Service**, as shown in the following screenshot:![The VRM site name](img/6442EN_01_12.jpg)

### Note

Sometimes, it can take a while for the appliance to save the setting and restart the VRM service.

# Configuring a SQL database for VRMS

The vSphere Replication Appliance, by default, initializes the default embedded vPostgreSQL database. All of the initial configuration data and the replication configuration data will be stored in the embedded database. Therefore, it is important that you plan on the type of database before you configure replication for the VMs. This is because if you were to reconfigure the VRMS component to use an external database, then you will lose the existing replication configuration information. You will need to reconfigure the replication on the VMs. Backup and restoration of an external database is easier because you will have to only back up the database files. If you were to plan a backup of the embedded database, then you will have to back up the entire appliance.

To know which versions of SQL Server are supported, use the **Solution/Database Interoperability** filter on the **VMware Product Interoperability Matrixes** web portal. The portal can be reached by going to **VMware Compatibility Guides** via [http://www.vmware.com/in/guides.html](http://www.vmware.com/in/guides.html) and clicking on the **Product Interoperability Matrixes** hyperlink.

At the VMware Product Interoperability Matrixes web portal, select **Solution/Database Interoperability** and select **VMware vSphere Replication** as the VMware Product and **5.5** as the **Version**. You can then choose a database from the list and verify its compatibility. Refer to the following screenshot:

![Configuring a SQL database for VRMS](img/6442EN_01_13.jpg)

The following procedure will guide you through the steps required to configure a SQL database for the VRMS:

1.  Log in to your database server and start **Microsoft SQL Server Management Studio**.
2.  From the **Object Explorer** window, right-click on **Database**, and click on **New Database**, as shown in the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_14.jpg)
3.  In the **New Database** window, provide a **Database name** and leave the rest of the attributes at their defaults for now, and click on **OK**. Refer to the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_15.jpg)
4.  From the **Object Explorer** window, expand **Security**, right-click on **Logins**, and click on **New Login**, as shown in the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_16.jpg)
5.  In the **Login-New** window, select **SQL Server authentication** and provide a **Login name**, set the password, and deselect **Enforce password policy**, which will void the other two password policies (**User must change password at the next login** and **Enforce password expiration**) as well.
6.  Set **Default database** to the newly created DB for vSphere Replication, and click on **OK**. Refer to the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_17.jpg)

    ### Tip

    In this example, we created a DB named `VR_DB`, so we should be changing the default database to `VR_DB`.

7.  From the **Object Explorer** window, expand **Databases**, right-click on the new database (`VR_DB`), and click on **Properties**, as shown in the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_18.jpg)
8.  In the **Database properties** window, select the **Files** page and change the database **Owner** to the login you created, as shown in the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_19.jpg)
9.  From the same window, select the **Options** page, and change **Recovery model** to **Simple**. Refer to the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_20.jpg)
10.  Click on **OK** to close the **Database Properties** window.
11.  Now, connect to the VAMI of the vSphere Replication Appliance by entering the URL `https://<IP address or FQDN>:5480`.
12.  Log in using the root user and the password.
13.  Navigate to the **Configuration** tab under the **VR** menu.
14.  Set **Configuration Mode** to **Manual configuration**.
15.  Set **DB Type** to **SQL Server**.
16.  Provide the **DB Host**, which could be the address (FQDN/IP) of the DB Server.
17.  Specify **DB Username**, **DB Password**, and **DB Name** as shown in the following screenshot:![Configuring a SQL database for VRMS](img/6442EN_01_21.jpg)
18.  Click on **Save and Restart Service**.

Hitting **Save and Restart** will save the new settings and restart the vSphere Replication Management Service. It might take some time to finish, owing to the time it needs to prepare the database. Once done, you will have to reconfigure replications on the VM.

# Deploying a vSphere Replication Server

Unlike with the vSphere Replication Appliances, you can deploy additional vSphere Replication Servers using an add-on appliance available in the vSphere Replication deployment bundle that was downloaded. You can deploy up to 10 VR Servers appliances per vCenter Server instance. There are several use cases to deploy additional VR Servers. One of the prime reasons is the manual load distribution. Each VR Server with the default memory configuration of 512 MB can handle up to 100 replications. If there are more than 100 VMs being replicated, then you could either choose to increase the memory of the appliance or deploy additional appliances and load balance by distributing the replication traffic to different appliances.

The following procedure will guide you through the steps required to deploy additional VR Servers:

1.  From the vSphere Web Client's home page, click on **vSphere Replication** to bring up the vSphere Web Client's interface for vSphere Replication, as shown in the following screenshot:![Deploying a vSphere Replication Server](img/6442EN_01_22.jpg)
2.  This page will list the vCenter Server the VRMS is registered to. Click on the toolbar item **Manage**, which should bring up the **Manage** tab for that vCenter Server with the **vSphere Replication** subtab selected. Refer to the following screenshot:![Deploying a vSphere Replication Server](img/6442EN_01_23.jpg)
3.  Select **Replication Servers**, which is on the left-hand side pane, to view a list of registered VR Servers.
4.  Navigate to **Actions** | **All vSphere Replication Actions** | **Deploy VR server** to bring up the **Deploy OVF Template** wizard, as shown in the following screenshot:![Deploying a vSphere Replication Server](img/6442EN_01_24.jpg)
5.  Set the source as **Local file** and click on **Browse**.
6.  Navigate to the `vSphere replication bundle` folder and then to the `bin` subfolder. Select `OVF vSphere_Replication_AddOn_OVF10.ovf` and click on **Open** to return to the wizard screen.
7.  Click on **Next** to continue.
8.  The **Review details** screen will summarize the OVF template details. Note that the description says **Additional vSphere Replication Server**. Click on **Next** to continue. Refer to the following screenshot:![Deploying a vSphere Replication Server](img/6442EN_01_25.jpg)
9.  Provide a name and inventory location of the VM appliance and click on **Next** to continue.
10.  Accept the license agreement and click on **Next**.
11.  Provide a name for the VM and select a compute location. The compute location can be either a cluster of ESXi hosts or a single ESXi host. Click on **Next** to continue.
12.  Set an intended disk format for the appliance's VMDKs and choose a datastore to store the VM files. The default option is **Thick Provisioned Lazy Zeroed**. Click on **Next** to continue.
13.  Select a network (port group) for the VM's vNIC, choose between IPv4 or IPv6, and an IP allocation policy (**DHCP/Static**). Here, we have selected **Static**. Hence, it will have to manually specify the DNS Server, the Subnet Mask, and the gateway the subnet the VM will be part of. Click on **Next** to continue.
14.  Set the password and the static IP for the appliance. Click on **Next** to continue.
15.  On the **Ready to complete** screen, review the settings and click on **Finish** to deploy the appliance VM. You can select the checkbox **Power on after deployment** to start the VM if the deployment is successful.

Once the VRS is deployed, you will have to register the vSphere Replication Server to the VRMS. For instructions on how to do this, read the following section.

# Registering vSphere Replication Servers

The deployed vSphere Replication Servers should be registered to the VRMS for them to be used to handle replication traffic. For instructions on how to deploy vSphere Replication Servers, read the *Deploying a vSphere Replication Server* section in this chapter.

The following procedure will guide you through the steps required to register the vSphere Replication Servers:

1.  From the vSphere Web Client's home page, click on **vSphere Replication** to bring up the vSphere Web Client's interface for vSphere Replication.
2.  This page will list the vCenter Server the VRMS is registered to. Click on the toolbar item **Manage**, which should bring up the **Manage** tab for that vCenter Server with the **vSphere Replication** subtab selected.
3.  Select **Replication Servers**, which is on the left-hand side pane, to view a list of registered VR Servers.
4.  Navigate to **Actions** | **All vSphere Replication Actions** | **Register VR server**, to bring up the **Register vSphere Replication Server** window. Refer to the following screenshot:![Registering vSphere Replication Servers](img/6442EN_01_26.jpg)
5.  Browse through the vCenter inventory to locate the newly deployed VR appliance VM.
6.  Click on the **Virtual Machine** to highlight and click on **OK** to confirm the selection, as shown in the following screenshot:![Registering vSphere Replication Servers](img/6442EN_01_27.jpg)
7.  Once the registration is successful, it should be listed in the **Replication Servers** page as shown in the following screenshot:![Registering vSphere Replication Servers](img/6442EN_01_28.jpg)

# Summary

In this chapter, we learned how to deploy and prepare a vSphere Replication environment. In the next chapter, we will learn how to replicate and recover virtual machines.