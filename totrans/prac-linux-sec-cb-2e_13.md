# Vulnerability Scanning and Intrusion Detection

In this chapter, we will discuss the following:

*   Network security monitoring using Security Onion
*   Finding vulnerabilities with OpenVAS
*   Using Nikto for web server scanning
*   Hardening using Lynis

# Network security monitoring using Security Onion

**Security Onion** is a Linux-based distribution built for the purpose of network security monitoring. Monitoring the network for security-related events can be proactive, if used to identify vulnerabilities, or it can be reactive, in cases such as incident response.

Security Onion helps by providing insight into the network traffic and context around alerts.

# Getting ready

We discussed the process of installing and configuring Security Onion on a system in previous chapters. Having followed those steps, we have an up-and-running system with Security Onion installed on it.

No other prerequisites are needed for using Security Onion.

# How to do it...

In this section, we will walk through a few tools included in Security Onion that can help in security monitoring:

1.  Once we are done with the setup of the security tools included in Security Onion, we have to create a user account to use these tools. Open the Terminal and run the following command to create a user for the tools:

![](img/4114c9cf-5ebc-4407-b666-d6119547be1d.png)

In the preceding step, we have created a user named `pentest1` and then configured the password for them.

2.  Once we have created the user account, we can start using the tools.
3.  On the desktop, we can find the icon for the SGUIL tool. Double-click on the icon to run the tool.

4.  A login screen will open, as shown here. Enter the user details configured in the previous step and click on **OK**:

![](img/f901923a-7121-46d6-a247-f54b7432c48a.png)

5.  Once the user details are validated, the next window will ask to select the network to monitor. Select the interface from the options available and click on **Start SGUIL** to proceed:

![](img/3295d476-5e84-449c-a3bc-9aae527aeddc.png)

6.  We get the window shown next. This is the main screen of the SGUIL tool. Here, we can monitor the real-time events happening on the network selected in the previous step, along with the session data and raw packet captures:

![](img/9166f8e1-adda-4255-b751-7e4f238bc89e.png)

More information about using the tool can be found at [http://bammv.github.io/sguil/index.html](http://bammv.github.io/sguil/index.html).

7.  There are other tools also included in Security Onion, such as Kibana. To access this tool, we can find the shortcut on the desktop. Once we double-click on the shortcut, it will open the browser pointing at the URL: `https://localhost/app/kibana`.
8.  The browser will give a warning regarding **Insecure connection/Connection is not private** as a self-signed SSL certificate is being used. Ignore the error, shown as follows, click on Advanced, and proceed:

![](img/70450764-83f3-4bcd-a6b7-8cc8435f3729.png)

9.  Next, Kibana will ask for user details to log in. Use the user details configured in the first step. Once successfully logged in, we get the following window:

![](img/f362b4f4-1528-4f51-8c08-0cfb93a65da4.png)

10.  Kibana helps in visualizing Elasticsearch data and also navigating the Elastic stack.

11.  Security Onion includes other tools that can be used to monitor various activities in the network. Explore the tools to get more insight into them.

# How it works...

Security Onion is an open source Linux distribution used for enterprise security monitoring, intrusion detection, and log management. To help administrators perform security monitoring, it includes various security tools, such as Sguil, Kibana, Suricata, Snort, OSSEC, Squert, NetworkMiner, and many others.

# Finding vulnerabilities with OpenVAS

As a Linux administrator, we would like to keep track of vulnerabilities that may exist in the system. Finding these vulnerabilities in good time would also help in fixing them before any attack exploits them.

To find the vulnerabilities, we can use a vulnerability scanning tool such as **OpenVAS**. It is one of the most advanced open source vulnerability scanning tools around.

# Getting ready

To use OpenVAS, we have to first install and configure it on our system. For more information about the installation and configuration process, we can refer to the official website of OpenVAS: [http://www.openvas.org/](http://www.openvas.org/).

# How to do it...

Once we are done with the installation and initial configuration of OpenVAS, we can use it to scan the servers in our network. In this section, we will see how to configure and run a scan:

1.  To access OpenVAS, access this URL in the browser: `https://127.0.0.1:9392`.
2.  We will get a login screen as shown here. Enter the user details configured during the installation of OpenVAS:

![](img/b56dc8b6-41b3-4611-94b6-cb0ef56b4f38.png)

3.  After being logged in, we get the following window. In the top menu, we can find different options to use, such as **Scan**, **Assets**, and **Configuration**:

![](img/c57e29b4-00c6-4fdf-bc79-b738de331549.png)

4.  To scan a server, we will first add it as a target to scan. To do this, click on **Configuration** and then click on **Target**, as shown here:

![](img/9ede5d0b-097a-40d9-8570-510ea57899d8.png)

5.  We will get the following window. In the top left, we can see a star icon. Once we click on this icon, it will open a new window to add the target server:

![](img/3e23ccac-d6a5-4282-8ba6-542930db8aab.png)

6.  In the new window, enter the details of the target server. Give it a name to identify the target easily, and then enter the IP address as shown here:

![](img/bda48ed9-6c41-40fa-a3c8-ad703021907f.png)

Once the details have been entered, click on Create to save the target in the target list.

7.  We can see out target server under the target list here:

![](img/f377cd8a-a48a-46f0-891e-e6eede3d067d.png)

8.  Next, we click on the **Scan** menu and then click on **Tasks** to start creating a scan task:

![](img/572b6133-22a7-47c7-b55e-0fefbcb5fd37.png)

9.  In the next window, click on the blue star icon and then click on **New Task** to continue:

![](img/5f60e844-bd44-4104-9fc5-c0598a7dd771.png)

10.  Now, we will give a name to the scan we are creating and then select our target server using the list under the **Scan Targets** menu, as shown here:

![](img/22b4a528-84c2-452c-b031-6793ece28e07.png)

For the schedule option, check the Once box to run the scan only once. We can schedule the scan to be run multiple times, as per requirements. Next, click on Create.

11.  Once we click on Create, our scan task has been created and can be seen in the task list, shown as follows:

![](img/b9dba7d9-d952-47e8-9f20-9cd3d25ba542.png)

12.  Toward the extreme right of the scan that we have created, we can see some buttons in the **Actions** column, shown as follows. Here, we can start or pause a scan task created earlier:

![](img/7e4035d8-2f45-47c4-a089-22a6731ff03e.png)

13.  Once we click on the start or play button shown in the preceding screenshot, our scan will start running. Depending on the network speed and other resources, the time to complete the scan may vary.
14.  Once the scan completes, it can be seen in the **Scan Task** list, shown as follows:

![](img/81886fc4-915d-4335-96e0-4ea3bbec7598.png)

15.  The **Severity** column shows the summary of the scan. It shows the count of issues found based on their severity.

![](img/47ea5c8f-0314-4b7b-9269-2d366e5a2717.png)

16.  To check the complete list of the vulnerabilities found by the scanner, we can click on the **Scan name** and we will see the list of the vulnerabilities found by OpenVAS, as seen here:

![](img/ca91cbb7-be48-40b4-a95a-410ba9b4c3ff.png)

List of the vulnerabilities found by OpenVAS

# How it works...

OpenVAS lets us add the servers we wish to scan using the **Target** option. Once the server has been added, we create a **Scan task** by using the **Scan menu**. In the **Scan task**, we select the target created earlier, on which the scan needs to be performed.

When the **Scan task** is successfully configured, we run the scan. At completion, we can see the list of vulnerabilities found by OpenVAS.

# Using Nikto for web server scanning

If our Linux server is configured to run as a web server, there is a chance that the web server and the web application hosted on the web server may have vulnerabilities. In such a cases, we can use a web application scanning tool to identify these vulnerabilities, and Nikto is one such open source web scanner.

It can be used with any web server and can scan for a large number of items to detect vulnerabilities, misconfigurations, risky files, and so on.

# Getting ready

To use Nikto to scan our web server, we have to first install it on our system, from where the scan will be done. If we are using Kali Linux, Nikto comes preinstalled in it. For other Linux distributions, we can install the tool using the following command:

![](img/37b42b26-9804-4fd3-a878-bb3fbf41118c.png)

# How to do it...

In this section, we will see how to use Nikto to examine the web server and report potential vulnerabilities:

1.  To see more details about the options supported by Nikto, we run the following command:

![](img/c6497e1a-b9ed-473e-aef6-b8a9bb665725.png)

2.  Nikto supports various plugins for finding different vulnerabilities. If we want to see the list of plugins, we can use the following command:

![](img/1039d1f8-2a41-4dc9-9cea-3cec8033424e.png)

3.  Now, let's use Nikto to run the scan on our web server with the IP address `192.168.43.100`. We start the scan using the following command:

![](img/9def49f5-9347-44fd-bace-2a8484ff0c68.png)

Once we run the command, the scan will start running. Depending on the network speed and the number of vulnerabilities that may exist, the time to complete the scan may vary.

4.  We can see in the following screenshot that a few vulnerabilities have been identified by Nikto in our web server. It also tells us that the web server is running Apache 2.2.8, as seen before:

![](img/02b46aef-864e-49c7-b307-d7c0ddbfe08e.png)

# How it works...

Nikto comes with over 6,700 plugins, using which it can test for possible security issues in a web server.

Once we run the scan, Nikto uses these plugins, checks for all the vulnerabilities, and reports them if found.

# Hardening using Lynis

**Lynis** is a open source security tool that helps in auditing Unix-like systems. It performs an extensive scan of the system and, based on the results, provides guidance for system hardening and compliance testing.

Lynis can be used for various purposes, including vulnerability detection, penetration testing, security auditing, compliance testing, and system hardening.

# Getting ready

Lynis is supported on almost all Unix-based operating systems and versions. We can obtain a copy of Lynis from its official website by visiting the following link:

[https://cisofy.com/documentation/lynis/get-started/](https://cisofy.com/documentation/lynis/get-started/)

For our example, we are using an Ubuntu system to install Lynis. We run the following command to install the tool:

![](img/a2be2190-7ca2-459a-bcfa-7deab04599b2.png)

# How to do it...

In this section, we will see how to use Lynis to perform a detailed audit of the system security aspects and configurations:

1.  Once Lynis is installed on our system, we can run the `lynis` command, as follows, to check out more information about the options supported by the tool:

![](img/e85df688-59d9-4e35-a24e-36a1ada8e583.png)

2.  We can check whether this version of Lynis is the latest by running the following command:

![](img/e554274f-623e-4f58-b9fe-bb5de9f8d555.png)

We can see in the output that the current version is `211` and the latest version available is `266`. If we wish to update the version, we can continue with the steps shown in the output.

3.  Now, we will start the scan to audit our system and identify the gaps by running the following command:

![](img/2e9aa972-be84-4c61-8025-85d4272f452c.png)

4.  As the scan progresses, we can see the findings of the scan in the output shown here:

![](img/7cb8b158-8f54-4e9b-b2b3-48cd9ee3aae8.png)

5.  In the following output, we can see that Lynis has identified missing modules on the server:

![](img/b4c0d88a-5457-4809-9aa8-016b595ae441.png)

6.  When the scan completes, we can see the summary of the scan, as shown here:

![](img/d0484510-62df-482b-b81b-4a96dcf9f1bb.png)

# How it works...

When we use Lynis to audit the system, it first initializes and performs basic checks to determine the operating system and tools. Lynis will then run the enabled plugins and security tests, as per the categories defined

Lynis performs hundreds of tests, which will help in determining the security state of the system.

Once the scan completes, Lynis will report the status of the scan.