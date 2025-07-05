# 17

# Infrastructure and Automation with Ansible

If your day-to-day system administration or development work involves tedious and repetitive operations, **Ansible** could help you run your tasks while saving you precious time. Ansible is a tool for automating **software provisioning**, **configuration management**, and **application deployment workflows**. Initially developed by Michael DeHaan in 2012, Ansible was acquired by Red Hat in 2015 and is now maintained as an open source project.

In this chapter, you’ll learn about the fundamental concepts of Ansible, along with a variety of hands-on examples. In particular, we’ll explore the following topics:

*   Introducing Ansible architecture and configuration management
*   Installing Ansible
*   Working with Ansible

# Technical requirements

First, you should be familiar with the Linux command-line Terminal in general. Intermediate knowledge of Linux will help you understand some of the intricacies of the practical illustrations used throughout this chapter. You should also be proficient in using a Linux-based text editor.

For the hands-on examples, we recommend setting up a lab environment similar to the one we’re using. To replicate this environment, your CPU will need to have at least 6 physical cores and 6 virtual cores (12 in total). A quad-core CPU with hyper-threading is not enough. Also, OpenSSH should be installed on all the hosts. You’ll find related instructions in the *Setting up the lab environment* section of this chapter. If you don’t configure a lab environment, you will still benefit from the detailed explanations associated with the practical examples in this chapter.

Now, let’s start our journey by covering introductory concepts surrounding Ansible.

# Introducing Ansible architecture and configuration management

In the introduction to this chapter, we captured one of the essential aspects of Ansible – it’s a tool for automating workflows. Almost any Linux system administration task can be automated using Ansible. Using the Ansible CLI, we can invoke simple commands to change the **desired state** of a system. Usually, with Ansible, we execute tasks on a remote host or a group of hosts.

Let’s use the classic illustration of **package management**. Suppose you’re managing an infrastructure that includes a group of web servers, and you plan to install the latest version of a web server application (Nginx or Apache) on all of them. One way to accomplish this task is to SSH into each host and run the related shell commands to install the latest web server package. If you have a lot of machines, this will be a big task. You could argue that you can write a script to automate this job. This is possible, but then you’d have yet another job on your hands; that is, maintaining the script, fixing possible bugs, and, with your infrastructure growing, adding new features.

At some point, you may want to manage users or databases or configure network settings on multiple hosts. Soon, you’ll be looking at a Swiss Army knife tool, with capabilities that you’d rather get for free instead of writing them yourself. Here’s where Ansible comes in handy. With its myriad of modules – for almost any system administration task you can imagine – Ansible can remotely configure, run, or deploy your management jobs of choice, with minimal effort and in a very secure and efficient way.

We’ll consolidate these preliminary thoughts with a brief look at the Ansible architecture.

## Understanding the Ansible architecture

The core Ansible framework is written in Python. Let’s mention upfront that Ansible has an **agentless** architecture. In other words, it runs on a **control node** that executes commands against remote hosts, without the need for a remote endpoint or service to be installed on the managed host to communicate with the control node. At a minimum, the only requirement for Ansible communication is SSH connectivity to the managed host. Yet, the number of Ansible operations would be relatively limited to only running scripts and raw SSH commands if the host didn’t have a Python framework installed. The vast majority of server OS platforms already have Python installed by default.

Ansible can manage a fleet of remote hosts from a single control node using secure SSH connections. The following diagram shows the logical layout of a managed infrastructure using Ansible:

![Figure 17.1 – The logical layout of a managed infrastructure using Ansible](img/B19682_17_01.jpg)

Figure 17.1 – The logical layout of a managed infrastructure using Ansible

Production-grade enterprise environments usually include a **configuration management database** (**CMDB**) for organizing their IT infrastructure assets. Examples of IT infrastructure assets are servers, networks, services, and users. Although not directly part of the Ansible architecture, the CMDB describes the assets and their relationship within a managed infrastructure and can be leveraged to build an **Ansible inventory**.

The inventory is local storage on the Ansible control node – typically an **INI** or **YAML** file – that describes managed **hosts** or **groups** of hosts. The inventory is either inferred from the CMDB or manually created by the system administrator.

Now, let’s have a closer look at the high-level Ansible architecture shown in the following diagram:

![Figure 17.2 – The Ansible architecture](img/B19682_17_02.jpg)

Figure 17.2 – The Ansible architecture

The preceding diagram shows the Ansible control node interacting with managed hosts in a private or public cloud infrastructure. Here are some brief descriptions of the blocks featured in the architectural view:

*   **API and core framework**: The main libraries encapsulating Ansible’s core functionality; the Ansible core framework is written in Python
*   **Plugins**: Additional libraries extending the core framework’s functionality – for instance, the following:
    *   **Connection plugins**, such as cloud connectors
    *   **Test plugins**, verifying specific response data
    *   **Callback plugins** for responding to events
*   `user` module for managing users
*   `package` module for managing software packages

*   **Inventory**: The INI or YAML file describing the hosts and groups of hosts targeted by Ansible commands and playbooks*   **Playbooks**: The Ansible execution files describing a set of tasks that target managed hosts*   **Private or public clouds**: The managed infrastructure, hosted on-premises or in various cloud environments (for example, VMware, **Amazon Web Services** (**AWS**), and Azure)*   **Managed hosts**: The servers targeted by Ansible commands and playbooks*   `ansible`, `ansible-playbook`, `ansible-doc`, and so on*   **Users**: The administrators, power users, and automated user processes running Ansible commands or playbooks

Now that we have a basic understanding of the Ansible architecture, let’s look at what makes Ansible a great tool for automating management workflows. We’ll introduce the concept of **configuration** **management** next.

## Introducing configuration management

If we look back at the old days, system administrators usually managed a relatively low number of servers, running everyday administrative tasks by using a remote shell on each host. Relatively simple operations, such as copying files, updating software packages, and managing users, could easily be scripted and reused regularly. With the recent surge in apps and services, driven by the vast expansion of the internet, modern-day on-premises and cloud-based IT infrastructures – sustaining the related platforms – have grown significantly. The sheer amount of configuration changes involved would by far exceed the capacity of a single admin running and maintaining a handful of scripts. Here’s where configuration management comes to the rescue.

With configuration management, managed hosts and assets are grouped into logical categories, based on specific criteria, as suggested in *Figure 17**.1*. Managing assets other than hosts ultimately comes down to performing specific tasks on the servers hosting those assets. The **configuration management manifest** is the Ansible inventory file that manages these hosts and assets. Thus, Ansible becomes the configuration management endpoint.

With Ansible, we can run single one-off commands to carry out specific tasks, but a far more efficient configuration management workflow can be achieved via `ansible-playbook` runs for regular maintenance and configuration management tasks is a common practice in IT infrastructure automation. We will discuss Ansible playbooks later in this chapter.

Running Ansible tasks repeatedly (or on a scheduled basis) against a specific target raises the concern of unwanted changes in the desired state due to repetitive operations. This issue brings us to one of the essential aspects of configuration management – the **idempotency** of configuration changes. We’ll look at what idempotent changes are next.

Explaining idempotent operations

In configuration management, an operation is **idempotent** when running it multiple times yields the same result as running it once. In this sense, Ansible is an idempotent configuration management tool.

Let us explain how an idempotent operation works. Let us suppose we have an Ansible task creating a user. When the task runs for the first time, it creates the user. Running it for a second time – when the user has already been created – would result in a **no-operation** (**no-op**). *Without* idempotency, subsequent runs of the same task would produce errors due to attempting to create a user that already exists.

We should note that Ansible is not the only configuration management tool on the market. We have **Chef**, **Puppet**, and **SaltStack**, to name a few. Most of these platforms have been acquired by larger enterprises, such as SaltStack, being owned by VMware, and some may argue that Ansible’s success could be attributed to Red Hat’s open sourcing of the project. Ansible appears to be the most successful configuration management platform of our day. The industry consensus is that Ansible provides a user-friendly experience, high scalability, and affordable licensing tiers in enterprise-grade deployments.

With the introductory concepts covered, let’s roll up our sleeves and install Ansible on a Linux platform of choice.

# Installing Ansible

In this section, we’ll show you how to install Ansible on a **control node**. On Linux, we can install Ansible in a couple of ways:

*   Using the platform-specific package manager (for example, `apt` on Ubuntu/Debian)
*   Using `pip`, the Python package manager

The Ansible community recommends `pip` for installing Ansible since it provides the most recent stable version of Ansible. In this section, we’ll use Ubuntu as our distribution of choice. For a complete Ansible installation guide for all major OS platforms, please follow the online documentation at [https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

On the control node, Ansible requires Python, so before installing Ansible, we need to make sure we have Python installed on our system.

Important note

Python 2 is no longer supported as of January 1, 2020\. Please use Python 3.8 (or newer) instead.

Let’s start by installing Ansible on Ubuntu.

## Installing Ansible on Ubuntu

With Ubuntu 22.04 LTS, we have Python 3 installed by default. We can proceed with installing Ansible by following these steps:

1.  Let’s first check the Python 3 version we have by using the following command:

    ```
    apt, we need to add the Ansible apt repository:

    ```
    sudo apt update
    ```

    ```

2.  Next, we must add the Ansible PPA:

    ```
    sudo apt install -y software-properties-common
    sudo apt-add-repository --update ppa:ansible/ansible -y
    ```

3.  Now, we can install the Ansible package with the following command:

    ```
    sudo apt install ansible -y
    ```

4.  With Ansible installed, we can check its current version:

    ```
    ansible --version
    ```

    In our case, the relevant excerpt in the output of the previous command is as follows:

    ```
    ansible [core 2.15.4]
    ```

Next, we’ll look at how to install Ansible using `pip`.

## Installing Ansible using pip

Before we install Ansible with `pip`, we need to make sure Python is installed on the system. We assume Python 3 is installed based on the steps presented in the previous section. When installing Ansible using `pip`, it is a safe practice to previously uninstall any version of Ansible that was installed using the local package manager (such as `apt`). This will ensure that `pip` will install the latest version of Ansible successfully. To proceed with the installation, follow these steps:

1.  We should remove any existing version of Ansible that’s been installed with the platform-specific package manager (for example, `apt` or `yum`). To uninstall Ansible on Ubuntu, run the following command:

    ```
    pip is installed. The following command should provide the current version of pip:

    ```
    pip installer first:
    ```

    ```
    pip and Ansible with the following commands:

    ```
    python3 get-pip.py --user
    pip and Ansible for the current user (hence the --user option we used).
    ```

    ```

    ```

2.  If you wish to install Ansible *globally* on the system, the equivalent commands are as follows:

    ```
    sudo python3 get-pip.py
    sudo python3 -m pip install ansible
    ```

3.  After the installation completes, you may have to log out and log back in to your Terminal again before using Ansible. You can check the Ansible version you have installed with the following command:

    ```
    ansible --version
    ```

    In our case, the output shows the following:

    ```
    ansible [core 2.15.4]
    ```

As you can see, you can get the most recent version (as of writing) of Ansible by using `pip`. Therefore, it is the recommended method of installing Ansible.

With Ansible installed on our control node, let’s look at some practical examples of using Ansible.

# Working with Ansible

In this section, we’ll use the Ansible CLI tools extensively to perform various configuration management tasks. To showcase our practical examples, we’ll work with a custom lab environment, and we highly encourage you to reproduce it for a complete configuration management experience.

Here’s a high-level outline of this section:

*   Setting up the lab environment
*   Configuring Ansible
*   Using Ansible ad hoc commands
*   Using Ansible playbooks
*   Using templates with Jinja2
*   Using Ansible roles

Let’s start with an overview of the lab environment.

## Setting up the lab environment

Our lab uses **Kernel-based Virtual Machine** (**KVM**) as a hypervisor for the virtual environment, but any other hypervisor will do. [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with Virtual Machines*, describes the process of creating Linux **virtual machines** (**VMs**) in detail. We deployed the following VMs using Ubuntu Server LTS to mimic a real-world configuration management infrastructure:

*   `neptune`: The Ansible control node
*   `ans-web1`: Web server
*   `ans-web2`: Web server
*   `ans-db1`: Database server
*   `ans-db2`: Database server

All VMs have the default server components installed. On each host, we created a default admin user called `packt` with SSH access enabled. Each VM will have 2 vCPUs, 2 GB of RAM, and 20 GB minimum.

Now, let’s briefly describe the setup for these VMs, starting with managed hosts.

### Setting up managed hosts

There are a couple of key requirements for managed hosts to fully enable configuration management access from the Ansible control node:

*   They must have an OpenSSH server installed and running
*   They must have Python installed

As specified in the *Technical requirements* section, we assume you have OpenSSH enabled on your hosts. For installing Python, you may follow the related steps described in the *Installing* *Ansible* section.

Important note

Managed hosts don’t require Ansible to be installed on the system.

To set the hostname on each VM, you may run the following command (for example, for the `ans-web1` hostname):

```
sudo hostnamectl set-hostname ans-web1
```

We also want to disable the `sudo` login password on our managed hosts to facilitate unattended privilege escalation when running automated scripts. If we don’t make this change, remotely executing Ansible commands will require a password.

To disable the `sudo` login password, edit the `sudo` configuration with the following command:

```
sudo visudo
```

Add the following line and save the configuration file. Replace `packt` with your username if it’s different:

```
packt ALL=(ALL) NOPASSWD:ALL
```

You’ll have to make this change on all managed hosts.

Next, we’ll look at the initial setup for the Ansible control node.

### Setting up the Ansible control node

The Ansible `neptune`) interacts with `ans-web1`, `ans-web2`, `ans-db1`, and `ans-db2`) using Ansible commands and playbooks. For convenience, our examples will reference the managed hosts by their hostnames instead of their IP addresses. To easily accomplish this, we added the following entries to the `/etc/hosts` file on the Ansible control node (`neptune`):

```
127.0.0.1 neptune localhost
192.168.122.70 ans-web1
192.168.122.147 ans-web2
192.168.122.254 ans-db1
192.168.122.25 ans-db2
```

You’ll have to match the hostnames and IP addresses according to your VM environment.

Next, we must install Ansible on our managed hosts too. Use the related procedure described in the *Installing Ansible* section earlier in this chapter, when we showed you how to install Ansible on the control node. In our case, we followed the steps in the *Installing Ansible using pip* section to benefit from the latest Ansible release at the time of writing.

Finally, we’ll set up SSH key-based authentication between the Ansible control node and the managed hosts.

### Setting up SSH key-based authentication

Ansible uses SSH communication with managed hosts. The SSH key authentication mechanism enables remote SSH access without the need to enter user passwords. To enable SSH key-based authentication, run the following commands on the Ansible control host (`neptune`).

Use the following command to generate a secure key pair and follow the default prompts:

```
ssh-keygen
```

With the key pair generated, copy the related public key to each managed host. You’ll have to target one host at a time and authenticate with the remote `packt` user’s password. Accept the SSH key exchange when prompted:

```
ssh-copy-id -i ~/.ssh/id_rsa.pub packt@ans-web1
ssh-copy-id -i ~/.ssh/id_rsa.pub packt@ans-web2
ssh-copy-id -i ~/.ssh/id_rsa.pub packt@ans-db1
ssh-copy-id -i ~/.ssh/id_rsa.pub packt@ans-db2
```

Now, you should be able to SSH into any of the managed hosts from the Ansible control node (`neptune`) without being prompted for a password. For example, to access `ans-web1`, you can test it with the following command:

```
ssh packt@ans-web1
```

The command will take you to the remote server’s (`ans-web1`) Terminal. Make sure you go back to the Ansible control node’s Terminal (on `neptune`) before following the next steps.

We’re now ready to configure Ansible on the control node.

## Configuring Ansible

This section explores some of the basic configuration concepts of Ansible that are related to the Ansible **configuration file** and **inventory**. Using a configuration file and the parameters within, we can change the *behavior* of Ansible, such as privilege escalation, connection timeout, and default inventory file path. The inventory defines managed hosts, acting as the CMDB of Ansible.

Let’s look at the Ansible configuration file first.

### Creating an Ansible configuration file

The following command provides some helpful information about our Ansible environment, including the current configuration file:

```
ansible --version
```

Here’s the complete output of the preceding command:

![Figure 17.3 – The default Ansible configuration settings](img/B19682_17_03.jpg)

Figure 17.3 – The default Ansible configuration settings

A default Ansible installation will set the configuration file path to `/etc/ansible/ansible.cfg`. As you can probably guess, the default configuration file has a *global scope*, which means that it’s used by default when we run Ansible tasks.

Now, let’s look at some different scenarios and how they can be addressed:

*   What if there are multiple users on the same control host running Ansible tasks? Our instinct suggests that each user may have their own set of configuration parameters. Ansible resolves this problem by looking into the user’s home directory for the `~/.ansible.cfg` file. Let’s verify this behavior by creating a dummy configuration file in our user’s (`packt`) home directory:

    ```
    ansible --version command now yields the following config file path:
    ```

![Figure 17.4 – Changing the default configuration file](img/B19682_17_04.jpg)

Figure 17.4 – Changing the default configuration file

In other words, `~/.ansible.cfg` takes precedence over the global `/etc/ansible/ansible.cfg` configuration file.

*   Now, suppose our user (`packt`) creates multiple Ansible projects, some managing on-premises hosts and others interacting with public cloud resources. Again, we may need a different set of Ansible configuration parameters (such as a connection timeout and inventory file). Ansible accommodates this scenario by looking for the `./ansible.cfg` file in the current folder.

    Let’s create a dummy `ansible.cfg` file in a new `~/``ansible/` directory:

    ```
    mkdir ~/ansible
    ~/ansible directory and invoking the ansible --version command shows the following config file:
    ```

![Figure 17.5 – Changing the current directory and configuration file](img/B19682_17_05.jpg)

Figure 17.5 – Changing the current directory and configuration file

We could have named our project directory anything, not necessarily `/home/packt/ansible`. Ansible prioritizes the `./ansible.cfg` file over the `~/.ansible.cfg` configuration file in the user’s home directory.

*   Finally, we may want the ultimate flexibility of a configuration file that doesn’t depend on the directory or location originating from our Ansible commands. Such a feature could be helpful while testing ad hoc configurations without altering the main configuration file. For this purpose, Ansible reads the `ANSIBLE_CONFIG` environment variable for the path of the configuration file.

    Assuming we are in the `~/ansible` project folder, where we already have our local `ansible.cfg` file defined, let’s create a dummy test configuration file named `test.cfg`:

    ```
    cd ~/ansible
    touch test.cfg
    ANSIBLE_CONFIG=test.cfg ansible --version
    ```

    The output shows the following:

![Figure 17.6 – Verifying reading of the new configuration file](img/B19682_17_06.jpg)

Figure 17.6 – Verifying reading of the new configuration file

We should note that the configuration file should always have the `.cfg` extension. Otherwise, Ansible will discard it.

Here’s a list summarizing the order of precedence in descending order for Ansible configuration files:

1.  The `ANSIBLE_CONFIG` environment variable
2.  The `./ansible.cfg` file in the local directory
3.  The `~/.ansible.cfg` file in the user’s home directory
4.  `/``etc/ansible/ansible.cfg`

In our examples, we’ll rely on the `ansible.cfg` configuration file in a local project directory (`~/ansible`). Let’s create this configuration file and leave it empty for now:

```
mkdir ~/ansible
cd ~/ansible
touch ansible.cfg
```

For the rest of this chapter, we’ll run our Ansible commands from the `~/ansible` folder unless we specify otherwise.

Unless we specifically define (override) configuration parameters in our configuration file, Ansible will assume the system defaults. One of the attributes that we’ll add to the config file is the inventory file path. But first, we’ll need to create an inventory. The following section will show you how.

### Creating an Ansible inventory

An Ansible inventory is a regular **INI** or **YAML** file describing the managed hosts. In its simplest form, the inventory could be a flat list of hostnames or IP addresses, but Ansible can also organize the hosts into **groups**. Ansible inventory files are either **static** or **dynamic**, depending on whether they are created and updated manually or dynamically. For now, we’ll use a static inventory.

In our demo environment with two web servers (`ans-web1`, `ans-web2`) and two database servers (`ans-db1`, `ans-db2`), we can define the following inventory (in INI format) in a file named `hosts` (located inside `~/ansible`) that we will create later on, as shown in *Figure 17**.7*:

```
[webservers]
ans-web1
ans-web2
[databases]
ans-db1
ans-db2
```

We classified our hosts into a couple of groups, featured in bracketed names; that is, `[webservers]` and `[databases]`. As discussed earlier, groups are logical arrangements of hosts based on specific criteria. Hosts can be part of multiple groups. Group names are case-sensitive, should always start with a letter, and should not contain hyphens (`-`) or spaces.

Ansible has two default groups:

*   `all`: Every host in the inventory
*   `ungrouped`: Every host in `all` that is not a member of another group

We can also define groups based on specific patterns. For example, the following group includes a range of hostnames starting with `ans-web` and ending with a number in the range of `1`-`2`:

```
[webservers]
ans-web[1:2]
```

Patterns are helpful when we’re managing a large number of hosts. For example, the following pattern includes all the hosts within a range of IP addresses:

```
[all_servers]
172.16.191.[11:15]
```

Ranges are defined as `[START:END]` and include all values from `START` to `END`. Examples of ranges are `[1:10]`, `[01:10]`, and `[a-g]`.

Groups can also be nested. In other words, a group may contain other groups. This nesting is described with the `:children` suffix. For example, we can define a `[platforms]` group that includes the `[ubuntu]` and `[debian]` groups (all our VMs are running on Ubuntu, so this is only for explaining purposes):

```
[platforms:children]
ubuntu
debian
```

Let’s name our inventory file `hosts`. Please note that we are in the `~/ansible` directory. Using a Linux editor of your choice, add the following content to the `hosts` file:

![Figure 17.7 – The inventory file in INI format](img/B19682_17_07.jpg)

Figure 17.7 – The inventory file in INI format

After saving the inventory file, we can validate it with the following command:

```
ansible-inventory -i ./hosts –-list --yaml
```

Here’s a brief explanation of the command’s parameters:

*   `-i (--inventory)`: Specifies the inventory file; that is, `./hosts`
*   `--list`: Lists the current inventory, as read by Ansible
*   `--yaml`: Specifies the output format as YAML

Upon successfully validating the inventory, the command will show the equivalent YAML output (the default output format of the `ansible-inventory` utility is JSON).

So far, we’ve expressed the Ansible inventory in INI format, but we may as well use a YAML file instead. The following screenshot shows the output of the preceding command in YAML format:

![Figure 17.8 – The output in YAML inventory format](img/B19682_17_08.jpg)

Figure 17.8 – The output in YAML inventory format

The YAML representation could be somewhat challenging, especially with large configurations, due to the strict indentation and formatting requirements. We’ll continue to use the INI inventory format throughout the rest of this chapter.

Next, we’ll point Ansible to our inventory. Edit the `./ansible.cfg` configuration file and add the following lines:

![Figure 17.9 – Pointing to our inventory file](img/B19682_17_09.jpg)

Figure 17.9 – Pointing to our inventory file

After saving the file, we’re ready to run Ansible commands or tasks that target our managed hosts. There are two ways we can perform Ansible configuration management tasks: using one-off **ad hoc commands** and via **Ansible playbooks**. We’ll look at ad hoc commands next.

## Using Ansible ad hoc commands

Ad hoc commands execute a single Ansible task and provide a quick way to interact with our managed hosts. These simple operations are helpful when we’re making simple changes and performing testing.

The general syntax of an Ansible ad hoc command is as follows:

```
ansible [OPTIONS] -m MODULE -a ARGS PATTERN
```

The preceding command uses an Ansible `MODULE` to perform a particular task on select hosts based on a `PATTERN`. The task is described via arguments (`ARGS`). You may recall that modules encapsulate a specific functionality, such as managing users, packages, and services. To demonstrate the use of ad hoc commands, we’ll use some of the most common Ansible modules for our configuration management tasks. Let’s start with the Ansible `ping` module.

### Working with the ping module

One of the simplest ad hoc commands is the Ansible `ping` test:

```
ansible -m ping all
```

The command performs a quick test on all managed hosts to check their SSH connectivity and ensure the required Python modules are present. Here’s an excerpt from the output:

![Figure 17.10 – A successful ping test with a managed host](img/B19682_17_10.jpg)

Figure 17.10 – A successful ping test with a managed host

The output suggests that the command was successful (`| SUCCESS`) and that the remote servers responded with `"pong"` to our ping request (`"ping": "pong"`). Please note that the Ansible `ping` module doesn’t use `ping` module inside Ansible is just a test module that requires Python; it does not rely on the `ping` command we use when troubleshooting network issues.

Next, we’ll look at ad hoc commands while using the Ansible `user` module.

### Working with the user module

Here’s another example of an ad hoc command. This one is checking if a particular user (`packt`) exists on all the hosts:

```
ansible -m user -a "name=packt state=present" all
```

The following is an excerpt of the output yielded by a successful check:

![Figure 17.11 – Checking if a user account exists](img/B19682_17_11.jpg)

Figure 17.11 – Checking if a user account exists

The preceding output also suggests that we could be even more specific when checking for a user account by also making sure they have a particular user and group ID:

```
ansible -m user -a "name=packt state=present uid=1000 group=1000" all
```

We can target ad hoc commands against a limited subset of our inventory. The following command, for example, would only ping the `web1` host for Ansible connectivity:

```
ansible -m ping ans-web1
```

Host patterns can also include wildcards or group names. Here are a few examples:

```
ansible -m ping ans-web*
ansible -m ping webservers
```

Let’s look at the available Ansible modules next. Before we do that, you may want to add the following line to `./ansible.cfg`, under the `[defaults]` section, to keep the noise down about deprecated modules:

```
deprecation_warnings = False
```

To list all the modules available in Ansible, run the following command:

```
ansible-doc --list
```

You may search or `grep` the output for a particular module. For detailed information about a specific module (for example, `user`), you can run the following command:

```
ansible-doc user
```

Make sure you check out the `EXAMPLES` section in the `ansible-doc` output for a specific module. You will find hands-on examples of using the module with ad hoc commands and playbook tasks.

Furthermore, we can create new users for different use cases on our Ansible hosts. Next, we will show you how to create a new user.

#### Creating a new user

If we want to create a new user (`webuser`) on all our web servers, we can perform the related operation with the following ad hoc command:

```
ansible -bK -m user -a "name=webuser state=present" webservers
```

Let’s explain the command’s parameters:

*   `-b` (`--become`): Changes the execution context to `sudo` (`root`)
*   `-K` (`--ask-become-pass`): Prompts for the `sudo` password on the remote hosts; the same password is used on all managed hosts
*   `-m`: Specifies the Ansible module (`user`
*   `-a`: Specifies the `user` module arguments as key-value pairs; `name=webuser` represents the username, while `state=present` checks whether the user account exists before attempting to create it
*   `webservers`: The group of managed hosts targeted by the operation

Creating a user account requires administrative (`sudo`) privileges on the remote hosts. Using the `-b` (`--become`) option invokes the related **privilege escalation** for the Ansible command to act as a *sudoer* on the remote system.

Important note

By default, Ansible does not enable `sudo` privileges, you must explicitly set the `-b` `(--become`) flag. You can override this behavior in the Ansible configuration file.

To enable unattended privilege escalation by default, add the following lines to the `ansible.cfg` file:

![Figure 17.12 – Adding privilege escalation rules](img/B19682_17_12.jpg)

Figure 17.12 – Adding privilege escalation rules

Now, you don’t have to specify the `--b` (`--become`) flag anymore with your ad hoc commands.

If the sudoer account on the managed hosts has the `sudo` login password enabled, we’ll have to provide it to our ad hoc command. Here is where the `-K` (`--ask-become-pass`) option comes in handy. Consequently, we’re asked for a password with the following message:

```
BECOME password:
```

This password is used across all managed hosts targeted by the command.

As you may recall, we disabled the `sudo` login password on our managed hosts (see the *Setting up the lab environment* section earlier in this chapter). Therefore, we can rewrite the previous ad hoc command without explicitly asking for privilege escalation and the related password:

```
ansible -m user -a "name=webuser state=present" webservers
```

There are some security concerns regarding privilege escalation, and Ansible has the mechanisms to mitigate the related risks. For more information on this topic, you may refer to [https://docs.ansible.com/ansible/latest/user_guide/become.html](https://docs.ansible.com/ansible/latest/user_guide/become.html).

The preceding command produces the following output:

![Figure 17.13 – Creating a new user using an ad hoc command](img/B19682_17_13.jpg)

Figure 17.13 – Creating a new user using an ad hoc command

You may have noticed that the output text here is highlighted, as it was with our previous ad hoc commands. Ansible highlights the output if it corresponds to a change in the *desired state* of the managed host. If you run the same command a second time, the output will not be highlighted, suggesting that there’s been no change since the user account has been already created. Here, we can see Ansible’s *idempotent operation* at work.

With the previous command, we created a user without a password for demo purposes only. What if we want to add or modify the password? Next, we will show you how to do that.

#### Adding or modifying a password

Glossing through the `user` module’s documentation with `ansible-doc user`, we can use the password field inside the module arguments, but Ansible will only accept `passlib`. Let’s install it on the Ansible control node with the following command:

```
pip install passlib
```

You’ll need the Python package manager (`pip`) to run the previous command. If you installed Ansible using `pip`, you should be fine. Otherwise, follow the instructions in the *Installing Ansible using pip* section to download and install `pip`.

With `passlib` installed, we can use the following ad hoc command to create or modify the user password:

```
ansible webservers -m user \
    -e "password=changeit!" \
    -a "name=webuser \
        update_password=always \
        password={{ password | password_hash('sha512') }}"
```

Here are the additional parameters helping with the user password:

*   `-e` (`--extra-vars`): Specifies custom variables as key-value pairs; we set the value of a custom variable to `password=changeit!`
*   `update_password=always`: Updates the password if it’s different from the previous one
*   `password={{...}}`: Sets the password to the value of the expression enclosed within double braces
*   `password | password_hash('sha512')`: Pipes the value of the `password` variable (`changeit!`) to the `password_hash()` function, thus generating an SHA-512 hash; `password_hash()` is part of the `passlib` module we installed earlier

The command sets the password of `webuser` to `changeit!`, and is an example of using variables (`password`) in ad hoc commands. Here’s the related output:

![Figure 17.14 – Changing the user’s password using an ad hoc command](img/B19682_17_14.jpg)

Figure 17.14 – Changing the user’s password using an ad hoc command

Ansible won’t show the actual password for obvious security reasons.

Now, you can try to SSH into any of the web servers (`web1` or `web2`) using the `webuser` account, and you should be able to authenticate successfully with the `changeit!` password.

#### Deleting a user

To delete the `webuser` account on all web servers, we can run the following ad hoc command:

```
ansible -m user -a "name=webuser state=absent remove=yes force=yes" webservers
```

The `state=absent` module parameter invokes the deletion of the `webuser` account. The `remove` and `force` parameters are equivalent to the `userdel -rf` command, deleting the user’s home directory and any files within, even if they’re not owned by the user.

The related output is as follows:

![Figure 17.15 – Deleting a user account using an ad hoc command](img/B19682_17_15.jpg)

Figure 17.15 – Deleting a user account using an ad hoc command

You may safely ignore `stderr` and `stderr_lines`, which were captured in the output. The message is benign since the user didn’t create a mail spool previously.

We’ll look at the `package` module next and run a few related ad hoc commands.

### Working with the package module

The following command installs the `webserver` group:

```
ansible -m package -a "name=nginx state=present" webservers
```

Here’s an excerpt from the output:

![Figure 17.16 – Installing the nginx package on the web servers](img/B19682_17_16.jpg)

Figure 17.16 – Installing the nginx package on the web servers

We use a similar ad hoc command to install the `databases` group:

```
ansible -m package -a "name=mysql-server state=present" databases
```

Here’s an excerpt from the command’s output:

![Figure 17.17 – Installing the mysql-server package on the database servers](img/B19682_17_17.jpg)

Figure 17.17 – Installing the mysql-server package on the database servers

If we wanted to remove a package, the ad hoc command would be similar but would feature `state=absent` instead.

Although the `package` module provides a good OS-level abstraction across various platforms, certain package management tasks are best handled with platform-specific package managers. We’ll show you how to use the `apt` module next.

### Working with platform-specific package managers

The following ad hoc command installs the latest updates on all Ubuntu machines in our managed environment. As all of our VMs are running Ubuntu, we will choose to run the commands only on the `webservers` group. The command will be the following:

```
ansible -m apt -a "upgrade=dist update_cache=yes" ubuntu
```

If we wanted, we could create one more group inside the `hosts` file, named `[ubuntu]` for example, and add all the VMs in there. This would be easier if we had different operating systems running, but that is not the case for us.

Thee platform-specific package management modules (`apt`, `yum`, and so on) match all the capabilities of the system-agnostic `package` module, featuring additional OS-exclusive functionality.

Let’s look at the `service` module next and a couple of related ad hoc commands.

### Working with the service module

The following command restarts the `nginx` service on all hosts in the `webservers` group:

```
ansible -m service -a "name=nginx state=restarted" webservers
```

Here’s a relevant excerpt from the output:

![Figure 17.18 – Restarting the nginx service on the web servers](img/B19682_17_18.jpg)

Figure 17.18 – Restarting the nginx service on the web servers

In the same way, we can restart the `mysql` service on all database servers, but there’s a trick to it! On Ubuntu, the MySQL service is named `mysql`. We could, of course, target each host with the appropriate service name, but if you had many database servers, it would be a laborious task. Alternatively, we can use the *exclusion pattern* (written as `!`) when targeting multiple hosts or groups.

The following command will restart the `mysql` service on all hosts in the `databases` group, except those that are members of the `debian` group (if we were to have a `debian` group):

```
ansible -m service -a "name=mysql state=restarted" 'databases:!debian'
```

Similarly, we can restart the `mysqld` service on all hosts in the `databases` group, except for those that are members of the `ubuntu` group (if we were to have one), with the following ad hoc command:

```
ansible -m service -a "name=mysqld state=restarted" 'databases:!ubuntu'
```

Always use single quotes (`''`) when you’re targeting multiple hosts or groups with an exclusion pattern; otherwise, the `ansible` command will fail.

Let’s look at one last Ansible module and the related ad hoc command, which is frequently used in upgrade scenarios.

### Working with the reboot module

The following ad hoc command reboots all hosts in the `webservers` group:

```
ansible -m reboot -a "reboot_timeout=3600" webservers
```

Slower hosts may take longer to reboot, especially during substantial upgrades, hence the increased reboot timeout of `3600` seconds. (The default timeout is `600` seconds.)

In our case, the reboot only took a few seconds. The output is as follows:

![Figure 17.19 – Rebooting the webservers group](img/B19682_17_19.jpg)

Figure 17.19 – Rebooting the webservers group

In this section, we showed a few examples of ad hoc commands using different modules. The next section will give you a brief overview of some of the most common Ansible modules and how to explore more.

## Exploring Ansible modules

Ansible has a vast library of modules. As we noted previously, you may use the `ansible-doc --list` command to browse the available Ansible modules on the command-line Terminal. You can also access the same information online, on the Ansible modules index page, at [https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html).

The online catalog provides module-by-category indexing to help you quickly locate a particular module you’re looking for. Here are some of the most typical modules used in everyday system administration and configuration management tasks with Ansible:

*   `apt`: Performs APT package management
*   `yum`: Performs YUM package management
*   `dnf`: Performs DNF package management

*   `users`: Manages users*   `services`: Controls services*   `reboot`: Restarts machines*   `firewalld`: Performs firewall management*   `copy`: Copies local files to the managed hosts*   `synchronize`: Synchronizes files and directories using `rsync`*   `file`: Controls file permissions and attributes*   `lineinfile`: Manipulates lines in text files*   `nmcli`: Controls network settings*   `get_url`: Downloads files over HTTP, HTTPS, and FTP*   `uri`: Interacts with web services and API endpoints*   `raw`: Simply runs a remote command via SSH (which is an unsafe practice); doesn’t need Python installed on the remote host*   `command`: Runs commands securely using Python’s remote execution context*   `shell`: Executes shell commands on managed hosts

We should note that ad hoc commands always execute a *single operation* using a *single module*. This feature is an advantage (for quick changes) but also a limitation. For more complex configuration management tasks, we use Ansible playbooks. The following section will take you through the process of authoring and running Ansible playbooks.

## Using Ansible playbooks

An **Ansible playbook** is essentially a list of tasks that are executed automatically. Ansible configuration management workflows are primarily driven by playbooks. More precisely, a playbook is a YAML file containing one or more **plays**, each with a list of **tasks** executed in the order they are listed. Plays are execution units that run the associated tasks against a set of hosts, and they are selected via a group identifier or a pattern. Each task uses a single module that executes a specific action targeted at the remote host. You may think of a task as a simple Ansible ad hoc command. As the majority of Ansible modules comply with idempotent execution contexts, playbooks are also idempotent. Running a playbook multiple times always yields the same result.

Well-written playbooks can replace laborious administrative tasks and complex scripts with relatively simple and maintainable manifests, running easily repeatable and predictable routines.

We’ll create our first Ansible playbook next.

### Creating a simple playbook

We’ll build our playbook based on the ad hoc command we used for creating a user (`webuser`). As a quick refresher, the command was as follows:

```
ansible -m user -a "name=webuser state=present" webservers
```

As we write the equivalent playbook, you may notice some resemblance to the ad hoc command parameters.

While editing the playbook YAML file, please be aware of the YAML formatting rules:

*   Use only space characters for indentation (no tabs)
*   Keep the indentation length consistent (for example, two spaces)
*   Items at the same level in the hierarchy (for example, list items) must have the same indentation
*   A child item’s indentation is one indentation more than its parent

Now, using a Linux editor of your choice, add the following lines to a `create-user.yml` file. Make sure you create the playbook in the `~/ansible` project directory, which is where we have our current inventory (`hosts`) and Ansible configuration file (`ansible.cfg`):

![Figure 17.20 – A simple playbook for creating a user](img/B19682_17_20.jpg)

Figure 17.20 – A simple playbook for creating a user

Let’s look at each line in our `create-user.yml` playbook:

*   `---`: Marks the beginning of the playbook file
*   `- name`: Describes the name of the play; we can have one or more plays in a playbook
*   `hosts: webservers`: Targets the hosts in the `webservers` group
*   `become: yes`: Enables privilege escalation for the current task; you can leave this line out if you enabled unattended privilege escalation in your Ansible configuration file (with `become = True` in the `[``privileged_escalation]` section)
*   `tasks`: The list of tasks in the current play
*   `- name`: The name of the current task; we can have multiple tasks in a play
*   `user`: The module being used by the current task
*   `name: webuser`: The name of the user account to create
*   `state: present`: The desired state upon creating the user – we want the user account to be present on the system

Let’s run our `create-user.yml` playbook:

```
ansible-playbook create-user.yml
```

Here’s the output we get after a successful playbook run:

![Figure 17.21 – Running the create-user.yml playbook](img/B19682_17_21.jpg)

Figure 17.21 – Running the create-user.yml playbook

Most of the `ansible-playbook` command-line options are similar to the ones for the `ansible` command. Let’s look at some of these parameters:

*   `-i` (`--inventory`): Specifies an inventory file path
*   `-b` (`--become`): Enables privilege escalation to `sudo` (`root`)
*   `-C` `(--check`): Produces a dry run without making any changes and anticipating the end results – a useful option for validating playbooks
*   `-l` (`--limit`): Limits the action of the command or playbook to a subset of managed hosts
*   `--syntax-check`: Validates the playbook’s syntax without making any changes; this option is only available for the `ansible-playbook` command

Let’s experiment with a second playbook, this time for deleting a user. We’ll name the playbook `delete-user.yml` and add the following content:

![Figure 17.22 – A simple playbook for deleting a user](img/B19682_17_22.jpg)

Figure 17.22 – A simple playbook for deleting a user

Now, let’s run this playbook:

```
ansible-playbook delete-user.yml
```

The output of the preceding command is as follows:

![Figure 17.23 – Limiting the delete-user.yml playbook to the Ubuntu host group](img/B19682_17_23.jpg)

Figure 17.23 – Limiting the delete-user.yml playbook to the Ubuntu host group

Next, we’ll look at ways to further streamline our configuration management workflows, starting with the use of variables in playbooks.

### Using variables in playbooks

Ansible provides a flexible and versatile model for working with variables in both playbooks and ad hoc commands. Through variables, we are essentially *parameterizing* a playbook, making it reusable or dynamic.

Take our previous playbook, for example, to create a user. We hardcoded the username (`webuser`) in the playbook. We can’t really reuse the playbook to create another user (for example, `webadmin`) unless we add the related task to it. But then, if we had many users, our playbook would grow proportionally, making it harder to maintain. And what if we wanted to specify a password for each user as well? The complexity of the playbook would grow even more.

Here’s where **variables** come into play. Let’s first look at what variables are and how they are written.

#### Introducing variables

We can substitute the hardcoded values with variables, making the playbook dynamic. In terms of pseudocode, our example of using a playbook to create a user with specific `username` and `password` variables would look like this:

```
User = Playbook(username, password)
```

Variables in Ansible are enclosed in double braces; for example, `{{ username }}`. Let’s see how we can leverage variables in our playbooks. Edit the `create-user.yml` playbook we worked on in the previous section and adjust it as follows:

![Figure 17.24 – Using the “username” variable in a playbook](img/B19682_17_24.jpg)

Figure 17.24 – Using the “username” variable in a playbook

We use the `{{ username }}` variable substituting our previously hardcoded value (`webuser`). Then, we surrounded the double braces with quotes to avoid syntax interference with the YAML dictionary notation. Variable names in Ansible must begin with a letter and only contain alphanumerical characters and underscores.

#### Setting values for variables

Next, we’ll explain *how* and *where* to set values for variables. Ansible implements a hierarchical model for assigning values to variables:

1.  `–extra-vars ansible-playbook` command-line parameter or the `./``group_vars/all` file.
2.  `./group_vars` directory in files named after each group.
3.  `./host_vars` directory in files named after each host. Host-specific variables are also available from `gather_facts` directive. You can learn more about Ansible facts at [https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html#ansible-facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html#ansible-facts).
4.  `vars` directive in a play or `include_vars` tasks.

In the preceding numbered list, the order of precedence for a variable’s value increases with each number. In other words, a variable value defined in a play will overwrite the same variable value specified at the host, group, or global level.

As an example, you may recall the peculiarity related to the MySQL service name on Ubuntu and RHEL/Fedora platforms. On Ubuntu, the service is `mysql`, while on Fedora, the service is `mysqld`. Suppose we want to restart the MySQL service on all hosts in our `databases` group. Assuming most of our database servers run Ubuntu, we can define a group-level `service` variable as `service: mysql`. We set this variable in the local project’s `./group_vars/databases` file. Then, in the play where we control the service status, we can override the `service` variable value with `mysqld` when the remote host’s OS platform is Fedora.

Let’s look at a few examples to illustrate what we’ve learned so far about placing variables and setting their values. Back in our `create-user.yml` playbook, we can define the `username` variable at the play level with the following directive:

```
  vars:
    username: webuser
```

Here’s what it looks like in the overall playbook:

![Figure 17.25 – Defining a variable at the play level](img/B19682_17_25.jpg)

Figure 17.25 – Defining a variable at the play level

Let’s run our playbook with the following command:

```
ansible-playbook create-user.yml
```

A relevant excerpt from the output is shown in the next screenshot:

![Figure 17.26 – Creating a user with a playbook using variables](img/B19682_17_26.jpg)

Figure 17.26 – Creating a user with a playbook using variables

#### Deleting user accounts

To delete user accounts, we can readjust our previous `delete-user.yml` file so that it looks as follows:

![Figure 17.27 – Deleting a user with a playbook using variables](img/B19682_17_27.jpg)

Figure 17.27 – Deleting a user with a playbook using variables

After saving the file, run the following command to delete the `webuser` account on all web servers:

```
ansible-playbook delete-user.yml
```

The relevant output from the preceding command run is as follows:

![Figure 17.28 – Deleting a user with a playbook using variables](img/B19682_17_28.jpg)

Figure 17.28 – Deleting a user with a playbook using variables

#### Enhancing our playbooks

We can improve our `create-user` and `delete-user` playbooks even further. You can follow these steps:

1.  Since the play exclusively targets the `webservers` group, we can define a `username` variable in the `./group_vars/webservers` file instead. This way, we can keep the playbooks more compact. Let’s remove the variable definition from both files.
2.  Next, create a `./group_vars` folder in the local directory (`~/ansible`) and add the following lines to a file named `webservers.yml`:

    ```
    webservers so that it matches the group we’re targeting. However, we should prefer to use the .yml extension so that we’re consistent with the file’s YAML format. Ansible accepts both naming conventions. Here’s the current tree structure of our project directory:
    ```

![Figure 17.29 – The directory tree, including the group_vars folder](img/B19682_17_29.jpg)

Figure 17.29 – The directory tree, including the group_vars folder

If we run our playbooks, the results should be identical to our previous runs:

```
ansible-playbook create-user.yml
ansible-playbook delete-user.yml
```

1.  Now, let’s add one more variable to our `create-user` playbook: the user’s `password` variable. You may recall the ad hoc command we created for the same purpose. See the *Using Ansible ad hoc commands* section earlier in this chapter for more information.

    Add the following lines to the `create-user.yml` file of the `user` task, at the same level as `name`:

    ```
    password: "{{ password | password_hash('sha512') }}"
    update_password: always
    ```

    You may notice how these changes are similar to the related ad hoc command. The updated playbook contains the following content:

![Figure 17.30 – The playbook with username and password variables](img/B19682_17_30.jpg)

Figure 17.30 – The playbook with username and password variables

1.  Next, edit the `./group_vars/webservers.yml` file and add the `password` variable with the `changeit!` value. Your updated file should have the following content:

![Figure 17.31 – Adding the new variable value inside the webservers.yml file](img/B19682_17_31.jpg)

Figure 17.31 – Adding the new variable value inside the webservers.yml file

1.  Let’s run the playbook:

    ```
    webuser) and password (changeit!) by trying to SSH into one of the web servers (for example, ans-web1):

    ```
    webuser account on the web servers with the following command, to get back to our initial state:

    ```
    create-user playbook to create a different user with a different password. Let’s name this user webadmin; we’ll set the password to changeme!. One way to accomplish this task is to use the -e (--extra-vars) option parameter with ansible-playbook:

    ```
    -e (--extra-vars) option parameter takes a JSON string featuring the username and password fields, along with the corresponding values. These values will *override* the values of the same variables defined at the group level in the ./group_vars/webservers.yml file.
    ```

    ```

    ```

    ```

2.  Let’s remove the `webuser` and `webadmin` accounts before we proceed with the next steps. Let’s run the `delete-user` playbook, first without any parameters:

    ```
    webuser account.
    ```

3.  Next, we’ll use the `-e` (`--extra-vars`) option parameter to delete the `webadmin` user:

    ```
    ansible-playbook -e '{"username": "webadmin"}' delete-user.yml
    ```

Using `–extra-vars` with our `create-user` and `delete-user` playbooks, we can act on multiple user accounts by running the playbooks manually or in a loop and feeding the JSON blob with the required variables. While this method could easily be scripted, Ansible provides even more ways to improve our playbooks by using task iteration with loops. We’ll look at loops later in this chapter, but first, let’s handle our passwords more securely with Ansible’s encryption and decryption facilities for managing secrets.

### Working with secrets

Ansible has a dedicated module for managing secrets called **Ansible Vault**. With Ansible Vault, we can encrypt and store sensitive data such as variables and files that are referenced in playbooks. Ansible Vault is essentially a password-protected secure key-value data store.

To manage our secrets, we can use the `ansible-vault` command-line utility. Regarding our playbook, where we’re creating a user with a password, we want to avoid storing the password in clear text. It is currently in the `./group_vars/webservers.yml` file. As a reminder, our `webservers.yml` file has the following content:

![Figure 17.32 – Sensitive data stored in the password variable](img/B19682_17_32.jpg)

Figure 17.32 – Sensitive data stored in the password variable

The last line contains sensitive data; the password is shown in plain text. We have a few options here to protect our data:

*   Encrypt the `webservers.yml` file. If we choose to encrypt the `webservers.yml` file, we could possibly incur the overhead of encrypting non-sensitive data, such as the username or other general-purpose information. If we have many users, encrypting and decrypting non-sensitive data would be highly redundant.
*   Encrypt the `password` variable only. This would work fine for a single user. But with a growing number of users, we’ll have multiple password variables to deal with, each with its own encryption and decryption. Performance will once again be an issue if we have a large number of users.
*   Store the password in a separate protected file. Ideally, we should have a separate file for storing all sensitive data. This file would be decrypted only once during the playbook run, even with multiple passwords stored.

We will pursue the third option and create a separate file to keep our user passwords in.

#### Protecting our data

Let’s look at the steps that need to be followed to ensure that our data is secure:

1.  We’ll name the file `passwords.yml` (we create it inside the `~/ansible/` directory) and add the following content to it:

![Figure 17.33 – The passwords.yml file storing sensitive data](img/B19682_17_33.jpg)

Figure 17.33 – The passwords.yml file storing sensitive data

1.  We added a YAML dictionary (or hash) item matching the `webuser` username related to the password. This item contains another dictionary as a key-value pair: `password: changeit!`. The equivalent YAML representation is as follows:

    ```
    webuser: { password: changeit! }
    ```

    This approach will allow us to add passwords that correspond to different users, like so:

    ```
    webuser: { password: changeit! }
    webadmin: { password: changeme! }
    ```

    We’ll explain the concept behind this data structure and its use when we consume the `password` variable in the playbook, later in this section.

2.  Now, since we keep our password in a different file, we’ll remove the corresponding entry from `webusers.yml`. Let’s add some other user-related information using the `comment` variable. Here’s what our `webusers.yml` file looks like:

![Figure 17.34 – The webusers.yml file storing non-sensitive user data](img/B19682_17_34.jpg)

Figure 17.34 – The webusers.yml file storing non-sensitive user data

1.  Next, let’s protect our secrets by encrypting the `passwords.yml` file using Ansible Vault:

    ```
    passwords.yml file with the following command:

    ```
    cat passwords.yml
    ```

    The output for the preceding command shows the following:
    ```

![Figure 17.35 – The encrypted passwords.yml file](img/B19682_17_35.jpg)

Figure 17.35 – The encrypted passwords.yml file

1.  We can view the content of the `passwords.yml` file with the following command:

    ```
    ansible-vault view passwords.yml
    ```

2.  You’ll be prompted for the vault password we created previously. The output shows the streamlined YAML content corresponding to our protected file:

![Figure 17.36 – Viewing the content of the protected file](img/B19682_17_36.jpg)

Figure 17.36 – Viewing the content of the protected file

1.  If you need to make changes, you can edit the encrypted file with the following command:

    ```
    vi) to edit your changes. If you want to re-encrypt your protected file with a different password, you can run the following command:

    ```
    ansible-vault rekey passwords.yml
    ```

    ```

2.  You’ll be prompted for the current vault password, followed by the new password.

Now, let’s learn how to reference secrets in our playbook.

#### Referencing secrets in playbooks

To reference secrets, follow these steps:

1.  First, let’s make sure we can read our password from the vault. Let’s make a new file with the following lines in it. We will call it `create-user-new.yml`:

![Figure 17.37 – Debugging vault access with new create-user-new.yml file](img/B19682_17_37.jpg)

Figure 17.37 – Debugging vault access with new create-user-new.yml file

We’ve added a couple of tasks:

*   `include_vars` (*lines 6*-*8*): Reading variables from the `passwords.yml` file
*   `debug` (*lines 10*-*12*): Debugging the playbook and logging the password that was read from the vault

None of these tasks are *aware* that the `passwords.yml` file is protected. *Line 12* is where the magic happens:

```
msg: "{{ vars[username]['password'] }}"
```

1.  We use the `vars[]` dictionary to query a specific variable in the playbook. `vars[]` is a *reserved* data structure for storing all the variables that were created via `vars` and `include_vars` in an Ansible playbook. We can query the dictionary based on a key appointed by `username`:

    ```
    {{ vars[username] }}
    ```

    Our playbook gets `username` from the `./group_vars/webservers.yml` file, and its value is `webuser`. Consequently, the `vars[webuser]` dictionary item reads the corresponding entry from the `passwords.yml` file:

    ```
    webuser: { password: changeit! }
    ```

2.  To get the password value from the corresponding key-value pair, we specify the `'password'` key in the `vars[username]` dictionary:

    ```
    {{ vars[username]['password'] }}
    ```

3.  Let’s run this playbook with the following command:

    ```
    --ask-vault-pass option to let Ansible know that our playbook needs vault access. Without this option, we’ll get an error when running the playbook. Here’s the relevant output for our debug task:
    ```

![Figure 17.38 – The playbook successfully reading secrets from the vault](img/B19682_17_38.jpg)

Figure 17.38 – The playbook successfully reading secrets from the vault

Here, we can see that the playbook successfully retrieves the password from the vault.

1.  Let’s wrap up our `create-user-new.yml` playbook by adding the following code:

![Figure 17.39 – The playbook creating a user with a password retrieved from the vault](img/B19682_17_39.jpg)

Figure 17.39 – The playbook creating a user with a password retrieved from the vault

Here are a few highlights of the current implementation:

*   We’ve added a `vars` block (`password` variable (at the play scope) for reading the password from the vault; we are reusing the `password` variable in multiple tasks.
*   The `include_vars` task (`passwords.yml` file.
*   The `debug` task (`no_log: true` enabled (*line 15*) to avoid logging sensitive information in the output. When debugging, you can temporarily set `no_log: false`.
*   The `user` task (`password` variable and hashes the corresponding value. This hashing is required by the Ansible `user` module for security reasons. We also added a `comment` field with additional user information. This field maps to the **Linux General Electric Comprehensive Operating System** (**GECOS**) record of the user. See the *Managing users* section of [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing Users and Groups*, for related information.

1.  Let’s run the playbook with the following command:

    ```
    webuser record in /etc/passwd on the ans-web1 machine:

    ```
    tail -n 10 /etc/passwd
    ```

    ```

2.  You should see the following line in the output (you’ll notice the GECOS field also):

    ```
    webuser:x:1001:1001:Regular web user:/home/webuser:/bin/sh
    ```

You may want to run the `ansible-playbook` command without supplying a vault password, as required by `--ask-vault-pass`. Such functionality is essential in scripted or automated workflows when using Ansible Vault. To make your vault password automatically available when you’re running a playbook using sensitive data, start by creating a regular text file, preferably in your home directory; for example, `~/vault.pass`. Add the vault password to this file in a single line. Then, you can choose *either* of the following options to use the vault password file:

*   Create the following environment variable:

    ```
    ansible.cfg file’s [defaults] section:

    ```
    vault_password_file = ~/vault.pass
    ```

    ```

Now, you can run the `create-user-new` playbook without the `--``ask-vault-pass` option:

```
ansible-playbook create-user.yml
```

Sometimes, protecting multiple secrets with a single vault password raises security concerns. Ansible supports multiple vault passwords through vault IDs.

#### Using vault IDs

A `passwords.yml` file. Suppose we want to secure this file using a vault ID. The following command creates a vault ID labeled `passwords` and prompts us to create a password:

```
ansible-vault create --vault-id passwords@prompt passwords.yml
```

The `passwords` vault ID protects the `passwords.yml` file. Now, let’s assume we also want to secure some API keys associated with users. If we stored these secrets in the `apikeys.yml` file, the following command would create a corresponding vault ID called `apikeys`:

```
ansible-vault create --vault-id apikeys@prompt apikeys.yml
```

Here, we created two vault IDs, each with its own password and protecting different resources.

The benefits of vault IDs are as follows:

*   They provide an improved security context when managing secrets. If one of the vault ID passwords becomes compromised, resources that have been secured by the other vault IDs are still protected.
*   With vault IDs, we can also leverage different access levels to vault secrets. For example, we can define `admin`, `dev`, and `test` vault IDs for related groups of users. Alternatively, we can have multiple configuration management projects, each with its own dedicated vault IDs and secrets; for example, `user-config`, `web-config`, and `db-config`.
*   You can associate a vault ID with multiple secrets. For example, the following command creates a `user-config` vault ID that secures the `passwords.yml` and `api-keys.yml` files:

    ```
    apikeys.yml file, which reads the corresponding vault ID password from the apikeys.pass file:

    ```
    passwords) to a playbook (create-users-new.yml) with the following command:

    ```
    ansible-playbook --vault-id passwords@passwords.pass create-users-new.yml
    ```

    ```

    ```

For more information about Ansible Vault, you may refer to the related online documentation at [https://docs.ansible.com/ansible/latest/user_guide/vault.html](https://docs.ansible.com/ansible/latest/user_guide/vault.html).

So far, we have created a single user account with a password. What if we want to onboard multiple users, each with their own password? As we noted previously, we could call the `create-user` playbook and override the `username` and `password` variables using the `--extra-vars` option parameter. But this method is not a very efficient one, not to mention the difficulty of maintaining it. In the next section, we’ll show you how to use task iteration in Ansible playbooks.

### Working with loops

**Loops** provide an efficient way of running a task repeatedly in Ansible playbooks. There are several loop implementations in Ansible, and we can classify them into the following categories based on their keyword or syntax:

*   `loop`: The recommended way of iterating through a collection
*   `with_<lookup>`: Collection-specific implementations of loops; examples include `with_list`, `with_items`, and `with_dict`, to name a few

In this section, we’ll keep our focus on the `loop` iteration (equivalent to `with_list`), which is best suited for simple loops. Let’s expand our previous use case and adapt it to create multiple users. We’ll start by making a quick comparison between running repeated tasks *with* and *without* loops:

1.  As a preparatory step, make sure `~/ansible` is your current working directory. Also, you can delete the `./group_vars` folder, as we’re not using it anymore. Now, let’s create a couple of playbooks, `create-users1.yml` and `create-users2.yml`, as shown in the following screenshot:

![Figure 17.40 – Playbooks with multiple versus iterative tasks](img/B19682_17_40.jpg)

Figure 17.40 – Playbooks with multiple versus iterative tasks

Both playbooks create three users: `webuser`, `webadmin`, and `webdev`. The `create-users1` playbook has three distinct tasks, one for creating each user. On the other hand, `create-users2` implements a single task iteration using the `loop` directive (in *line 15*):

```
loop: "{{ users }}"
```

The loop iterates through the items of the `users` list, defined as a `play` variable in *lines 6-9*. The `user` task uses the `{{ item }}` variable, referencing each user while iterating through the list.

1.  Before running any of these playbooks, let’s also create one for deleting the users. We’ll name this playbook `delete-users2.yml,` and it will have a similar implementation to `create-users2.yml`:

![Figure 17.41 – A playbook using a loop for deleting users](img/B19682_17_41.jpg)

Figure 17.41 – A playbook using a loop for deleting users

1.  Now, let’s run the `create-users1` playbook while targeting only the `ans-web1` web server:

    ```
    ansible-playbook create-users1.yml --limit ans-web1
    ```

    In the output, we can see that three tasks have been executed, one for each user:

![Figure 17.42 – The output of the create-users1 playbook, with multiple tasks](img/B19682_17_42.jpg)

Figure 17.42 – The output of the create-users1 playbook, with multiple tasks

1.  Let’s delete the users by running the `delete-users2.yml` playbook:

    ```
    ansible-playbook delete-users2.yml --limit ans-web1
    ```

    The output is as shown in the following screenshot:

![Figure 17.43 – Deleting users using the playbook](img/B19682_17_43.jpg)

Figure 17.43 – Deleting users using the playbook

1.  Now, let’s run the `create-users2` playbook, again targeting only the `web1` web server:

    ```
    ansible-playbook create-users2.yml --limit ans-web1
    ```

    This time, the output shows a single task iterating through all the users:

![Figure 17.44 – The output of the create-users2 playbook, with a single task iteration](img/B19682_17_44.jpg)

Figure 17.44 – The output of the create-users2 playbook, with a single task iteration

The difference between the two playbook runs is significant:

*   The first playbook executes a task for each user. While forking a task is not an expensive operation, you can imagine that creating hundreds of users would incur a significant load on the Ansible runtime.
*   On the other hand, the second playbook runs a single task, loading the `user` module three times, to create each user. Loading a module takes significantly fewer resources than running a task.

For more information about loops, you may refer to the related online documentation at [https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html).

Now that we know how to implement a simple loop, we’ll make our playbook more compact and maintainable.

### Configuring our playbooks

Apart from enhancing our playbooks, we’ll also try to come closer to a real-world scenario by storing the users and their related passwords in a reusable and secure fashion.

We will keep the web users’ information in the `users.yml` file. The related passwords are in the `users_passwords.yml` file. Here are the two files, along with some example user data:

![Figure 17.45 – The users.yml and users_passwords.yml files](img/B19682_17_45.jpg)

Figure 17.45 – The users.yml and users_passwords.yml files

Let’s take a closer look at these files:

*   The `users.yml` file contains a dictionary with a single key-value pair:
    *   `webusers`
    *   `username` and `comment` tuples
*   The `users_passwords.yml` file contains a nested dictionary with multiple key-value pairs, as follows:
    *   `<username>` (for example, `webuser`, `webadmin`, and so on)
    *   `password: <value>` key-value pair

You may use the `ansible-vault edit` command to update the `users_passwords.yml` file, or you can create a new one, as we did. Alternatively, after you create a file from scratch, you will then have to encrypt it, following the steps previously described in the *Working with* *secrets* section.

The new `create-users.yml` playbook file has the following implementation:

![Figure 17.46 – The create-users.yml playbook](img/B19682_17_46.jpg)

Figure 17.46 – The create-users.yml playbook

These files are also available in the GitHub repository for this book, in the related chapter folder. Let’s quickly go over the playbook’s implementation. We have three tasks:

*   `Load users`: Reads the web user information from the `users.yml` file and stores the related values in the `users` dictionary
*   `Load passwords`: Reads the passwords from the encrypted `passwords.yml` file and stores the corresponding values in the `passwords` dictionary
*   `Create user accounts`: Iterates through the `users.webusers` list and, for each item, creates a user account with the related parameters; the task performs a password lookup in the `passwords` dictionary based on `item.username`

Before running the playbook, let us encrypt the `users_passwords.yml` file using the following command:

```
ansible-vault encrypt users_passwords.yml
```

Now, run the playbook with the following command:

```
ansible-playbook -–ask-vault-pass create-users.yml
```

Here’s the output:

![Figure 17.47 – Running the create-users.yml playbook](img/B19682_17_47.jpg)

Figure 17.47 – Running the create-users.yml playbook

We can see the following playbook tasks at work:

*   `Gathering Facts`: Discovering managed hosts and related system variables (facts); we’ll introduce Ansible facts later in this chapter
*   `Load users`: Reading the users from the `users.yml` file
*   `Load passwords`: Reading the passwords from the encrypted `passwords.yml` file
*   `Create user accounts`: The task iteration loop creates users

You may verify the new user accounts using the methods presented earlier in the *Working with secrets* section. As an exercise, create the `delete-users.yml` playbook using a similar implementation to the `create-users` playbook.

Now, let’s look at how we can improve our playbook and reuse it to seamlessly create users across all hosts in the inventory, web servers, and databases alike. We’ll use conditional tasks to accomplish this functionality.

### Running conditional tasks

`when` task-level directive to define a condition.

We learned about variables and how to use them in playbooks. Facts and results are essentially variables of a specific type and use. We’ll look at each of these variables in the context of conditional tasks. Let’s start with facts.

#### Using Ansible facts

`ansible_` prefix.

Here are a few examples of Ansible facts:

*   `ansible_distribution`: The OS distribution (for example, `Ubuntu`)
*   `ansible_all_ipv4_addresses`: The IPv4 addresses
*   `ansible_architecture`: The platform architecture (for example, `x86_64` or `i386`)
*   `ansible_processor_cores`: The number of CPU cores
*   `ansible_memfree_mb`: The available memory (in MB)

Now, what if we didn’t have groups explicitly created for classifying our hosts in Ubuntu and Debian systems, for example (or any other distribution, for that matter)? In this case, we could gather facts about our managed hosts, detect their OS type, and perform the conditional update task, depending on the underlying platform. Let’s implement this functionality in a playbook using Ansible facts.

We’ll name our playbook `install-updates.yml` and add the following content to it:

![Figure 17.48 – The install-updates.yml playbook](img/B19682_17_48.jpg)

Figure 17.48 – The install-updates.yml playbook

The playbook targets all hosts and has two conditional tasks, based on the `ansible_distribution` fact:

*   `Install Ubuntu system updates`: Runs exclusively on Ubuntu hosts based on the `ansible_distribution == "Ubuntu"` condition (*line 9*)
*   `Install Debian system updates`: Runs exclusively on Debian hosts based on the `ansible_distribution == "Debian"` condition (*line 13*)

Let’s run our playbook:

```
ansible-playbook install-updates.yml
```

The commands will take a significant time to run if there are any updates to install on the hosts. Here’s the corresponding output:

![Figure 17.49 – Running conditional tasks](img/B19682_17_49.jpg)

Figure 17.49 – Running conditional tasks

There are three tasks illustrated in the preceding output:

*   `Gathering Facts`: The default discovery task that’s executed by the playbook to gather facts about remote hosts
*   `Install Ubuntu system updates`: The conditional task for running all Ubuntu hosts
*   `Install Debian system updates`: The conditional task for skipping all Ubuntu-based hosts, as we don’t currently run any Debian ones

Next, we’ll look at how to use Ansible’s environment-specific variables in conditional tasks.

#### Using magic variables

**Magic variables** describe the local Ansible environment and its related configuration data. Here are a few examples of magic variables:

*   `ansible_playhosts`: A list of active hosts in the current play
*   `group_names`: A list of all groups the current host is a member of
*   `vars`: A dictionary with all variables in the current play
*   `ansible_version`: The version of Ansible

To see magic variables in action while using conditional tasks, we’ll improve our `create-users` playbook even further and create specific groups of users on different host groups. So far, the playbook only creates users on hosts that belong to the `webservers` group (`web1`, `web2`). The playbook creates `webuser`, `webadmin`, and `webdev` user accounts on all web servers. What if we want to create a similar group of users – `dbuser`, `dbadmin`, and `dbdev` – on all our database servers? To achieve this, you can follow these steps:

1.  Start by adding the new user accounts and passwords to the `users.yml` and `users_passwords.yml` files, respectively. Here’s what we have after adding the database’s user accounts and passwords:

![Figure 17.50 – The users.yml and passwords.yml files](img/B19682_17_50.jpg)

Figure 17.50 – The users.yml and passwords.yml files

Note that you can edit the `users_passwords.yml` file using the `ansible-vault edit` command. Alternatively, you can decrypt the file, edit it, and re-encrypt it.

1.  Now, let’s create a `create-users3` playbook with the required conditional tasks to handle both groups – `webusers` and `databases` – selectively. Let us create a new file called `create-users3.yml` with the following content:

![Figure 17.51 – The create-users.yml playbook with conditional tasks](img/B19682_17_51.jpg)

Figure 17.51 – The create-users.yml playbook with conditional tasks

1.  Let’s run the playbook:

    ```
    ansible-playbook -–ask-vault-pass create-users3.yml
    ```

    The following is an excerpt of the output that shows the web user task skipping the database servers and the database user task skipping the web servers, suggesting that the web and database users have been created successfully:

![Figure 17.52 – The web and database user tasks running selectively](img/B19682_17_52.jpg)

Figure 17.52 – The web and database user tasks running selectively

For a complete list of Ansible special variables, including magic variables, please visit [https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html). For more information about facts and magic variables, check out the online documentation at [https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html).

Next, we’ll look at variables for tracking task results, also known as register variables.

#### Using register variables

`register` directive to capture the task’s output in a variable. A typical example of using register variables is collecting the result of a task for debugging purposes. In more complex workflows, a particular task may or may not run, depending on a previous task’s result.

Let’s consider a hypothetical use case. As we onboard new users and create different accounts on all our servers, we want to make sure the number of users doesn’t exceed the maximum number allowed. If the limit is reached, we may choose to launch a new server, redistribute the users, and so on. You can follow these steps:

1.  Let’s start by creating a playbook named `count-users.yml` with the following content:

![Figure 17.53 – The count-users.yml playbook](img/B19682_17_53.jpg)

Figure 17.53 – The count-users.yml playbook

We created the following tasks in the playbook:

*   `Count all users`: A task that uses the `shell` module to count all users; we register the `count` variable by capturing the task output
*   `Debug number of users`: A simple task for debugging purposes that logs the number of users and the maximum limit allowed
*   `Detect limit`: A conditional task that’s run when the limit has been reached; the task checks the value of the `count` register variable and compares it with the `max_allowed` variable

*Line 17* in our playbook needs some further explanation. Here, we take the actual standard output of the register variable; that is, `count.stdout`. As is, the value would be a string, and we need to cast it to an integer; that is, `count.stdout | int`. Then, we compare the resulting number with `max_allowed`.

1.  Let’s run the playbook while targeting only the `ans-web1` host:

    ```
    ansible-playbook count-users.yml --limit ans-web1
    ```

    The output is as follows:

![Figure 17.54 – The conditional task (Detect limit) is executed](img/B19682_17_54.jpg)

Figure 17.54 – The conditional task (Detect limit) is executed

Here, we can see that the number of users is 36, thus exceeding the maximum limit of 30\. In other words, the `Detect limit` task ran as it was supposed to.

1.  Now, let’s edit the `count-users.yml` playbook and change the following:

    ```
    max_allowed: 50
    ```

2.  Save and rerun the playbook. This time, the output shows that the `Detect limit` task was skipped:

![Figure 17.55 – The conditional task (Detect limit) is skipped](img/B19682_17_55.jpg)

Figure 17.55 – The conditional task (Detect limit) is skipped

To learn more about conditional tasks in Ansible playbooks, please visit [https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html). By combining conditional tasks with Ansible’s all-encompassing facts and special variables, we can write extremely powerful playbooks and automate a wide range of system administration operations.

In the following sections, we’ll explore additional ways to make our playbooks more reusable and versatile. We’ll look at dynamic configuration templates next.

## Using templates with Jinja2

One of the most common configuration management tasks is copying files to managed hosts. Ansible provides the `copy` module for serving such tasks. A typical file copy operation has the following syntax in Ansible playbooks:

```
- copy:
    src: motd
    dest: /etc/motd
```

The `copy` task takes a source file (`motd`) and copies it to a destination (`/etc/motd`) on the remote host. While this model would work for copying static files to multiple hosts, it won’t handle host-specific customizations in these files on the fly.

Take, for example, a network configuration file featuring the IP address of a host. Attempting to copy this file on all hosts to configure the related network settings could render all but one host unreachable. Ideally, the network configuration file should have a *placeholder* for the dynamic content (for example, IP address) and adapt the file accordingly, depending on the target host.

To address this functionality, Ansible provides the `template` module with the `template` syntax is very similar to `copy`:

```
- template:
    src: motd.j2
    dest: /etc/motd
```

The source, in this case, is a Jinja2 template file (`motd.j2`) with host-specific customizations. Before copying the file to the remote host, Ansible reads the Jinja2 template and replaces the dynamic content with the host-specific data. This processing happens on the Ansible control node.

To illustrate some of the benefits and internal workings of Ansible templates, we’ll work with a couple of use cases and create a Jinja2 template for each. Then, we’ll create and run the related playbooks to show the templates in action.

Here are the two templates we’ll be creating in this section:

*   `motd`): For displaying a customized message to users about scheduled system maintenance
*   `hosts`): For generating a custom `/etc/hosts` file on each system with the hostname records of the other managed hosts

Let’s start with the message-of-the-day template.

### Creating a message-of-the-day template

In our introductory notes, we used the `/etc/motd` file as an example. On a Linux system, the content of this file is displayed when a user logs in to the Terminal. Suppose you plan to upgrade your web servers on Thursday night and would like to give your users a friendly reminder about the upcoming outage. Your `motd` message could be something like this:

```
This server will be down for maintenance on Thursday night.
```

There’s nothing special about this message, and the `motd` file could be easily deployed with a simple `copy` task. In most cases, such a message would probably do just fine, apart from the rare occasion when users may get confused about which exactly is “`this server`”. You may also consider that Thursday night in the US could be Friday afternoon on the other side of the world, and it would be nice if the announcement were more specific.

Perhaps a better message would state, on the `ans-web1` web server, the following:

```
ans-web1 (172.16.191.12) will be down for maintenance on Thursday, April 8, 2021, between 2 - 3 AM (UTC-08:00).
```

On the `ans-web2` web server, the message would reflect the corresponding hostname and IP address. Ideally, the template should be reusable across multiple time zones, with playbooks running on globally distributed Ansible control nodes. Let’s see how we can implement such a template (we’ll assume your current working directory is `~/ansible`):

1.  First, create a `templates` folder in your local Ansible project directory:

    ```
    ./templates folder.
    ```

2.  Using a Linux editor of your choice, create a `motd.j2` file in `./templates` with the following content:

![Figure 17.56 – The motd.j2 template file](img/B19682_17_56.jpg)

Figure 17.56 – The motd.j2 template file

Note some of the particularities of the Jinja2 syntax:

*   Comments are enclosed in `{# ... #}`
*   Expressions are surrounded by `{% ... %}`
*   External variables are referenced with `{{ ... }}`

Here’s what the script does:

*   *Lines 1*-*4* define an initial set of local variables for storing time boundaries for the outage: the day of the outage (`date`), the starting time (`start_time`), and the ending time (`end_time`).
*   *Line 6* defines the input date-time format (`fmt`) for our starting and ending time variables.
*   *Lines 7*-*8* build `datetime` objects that correspond to `start_time` and `end_time`. These Python `datetime` objects are formatted according to our needs in the custom message.
*   *Line 11* prints the custom message, featuring user-friendly time outputs and a couple of Ansible facts, namely the `ansible_facts.fqdn`) and IPv4 address (`ansible_facts.default_ipv4.address`) of the host where the message is displayed.

1.  Now, let’s create a playbook running the template. We will name the playbook `update-motd.yml` and add the following content:

![Figure 17.57 – The update-motd.yml playbook](img/B19682_17_57.jpg)

Figure 17.57 – The update-motd.yml playbook

The `template` module reads and processes the `motd.j2` file, generating related dynamic content, then copies the file to the remote host in `/etc/motd` with the required permissions.

1.  Now, we’re ready to run our playbook:

    ```
    ansible-playbook update-motd.yml
    ```

    The command should complete successfully. Here is a screenshot of our output:

![Figure 17.58 – Running the update-motd.yml playbook](img/B19682_17_58.jpg)

Figure 17.58 – Running the update-motd.yml playbook

1.  You can immediately verify the `motd` message on any of the hosts (for example, `ans-web1`) with the following command:

    ```
    ans-web1 host and displays the content of the /etc/motd file:
    ```

![Figure 17.59 – The content of the remote /etc/motd file](img/B19682_17_59.jpg)

Figure 17.59 – The content of the remote /etc/motd file

1.  We can also SSH into any of the hosts to verify the `motd` prompt:

    ```
    ssh packt@ans-web1
    ```

    The Terminal shows the following output:

![Figure 17.60 – The motd prompt on the remote host](img/B19682_17_60.jpg)

Figure 17.60 – The motd prompt on the remote host

1.  Now that we know how to write and handle Ansible templates, let’s improve `motd.j2` to make it a bit more reusable. We’ll *parameterize* the template by replacing the hardcoded local variables for date and time with input variables that are passed from the playbook. This way, we’ll make our template reusable across multiple playbooks and different input times for maintenance. Here’s the updated template file (`motd.j2`):

![Figure 17.61 – The modified motd.j2 template with input variables](img/B19682_17_61.jpg)

Figure 17.61 – The modified motd.j2 template with input variables

Relevant changes are in *lines 1*-*2*, where we build `datetime` objects using the `date`, `start_time`, `end_time`, and `utc` input variables. Notice the difference between the *local variables* – `start_time_` and `end_time_` (suffixed with `_`) – and the corresponding *input variables*; that is, `start_time` and `end_time`. You may choose any naming convention for the variables, assuming it is Ansible-compliant.

1.  Now, let’s look at our modified playbook (`update-motd.yml`):

![Figure 17.62 – The modified update-motd.yml playbook with variables](img/B19682_17_62.jpg)

Figure 17.62 – The modified update-motd.yml playbook with variables

Relevant changes are highlighted in the preceding screenshot, where we added variables serving the input for the `motd.j2` template. Running the modified playbook should yield the same result as the previous implementation. We’ll leave the related exercise to you.

Next, we’ll look at another use case featuring template-based deployments: updating the `/etc/hosts` file on managed hosts with the host records of all the other servers in the group.

### Creating a hosts file template

Another example of using an Ansible template is to automatically update the `/etc/hosts` files on every machine by using Jinja2 templating. The `/etc/hosts` file contains numerical IP addresses and the names of all hosts on the network, and updating it regularly is a useful task for a system administrator. We will create a new template for the `hosts` file and then update the specific YAML file to access the new template file. To create a `hosts` file template, follow these steps:

1.  Let’s start by creating a new template file, named `hosts.j2`, in the `~/ansible/templates` directory. Add the following content:

![Figure 17.63 – The hosts.j2 template file](img/B19682_17_63.jpg)

Figure 17.63 – The hosts.j2 template file

Here’s how the template script works:

*   Adds a `localhost` record corresponding to the current host referenced by the `inventory_hostname` Ansible special variable
*   Executes a loop through all hosts in the inventory referenced by the `groups['all']` list (special variable)
*   Checks if the current host in the loop matches the target host, and it will only execute next if the hosts are *different*
*   Adds a new host record by reading the default IPv4 address (`default_ipv4.address`) of the current host from the related Ansible facts (`hostvars[host].ansible_facts`)

1.  Now, let’s create an `update-hosts.yml` playbook file referencing the `hosts.j2` template. Add the following content:

![Figure 17.64 – The update-hosts.yml playbook file](img/B19682_17_64.jpg)

Figure 17.64 – The update-hosts.yml playbook file

This playbook is very similar to `update-motd.yml`. It targets the `/``etc/hosts` file.

1.  With the playbook and template files ready, let’s run the following command:

    ```
    /etc/hosts file on any of the hosts (for example, ans-web1) by using the following command:

    ```
    ansible ans-web1 -a "cat /etc/hosts"
    ```

    The output shows the expected host records:
    ```

![Figure 17.65 – The autogenerated /etc/hosts file on web1](img/B19682_17_65.jpg)

Figure 17.65 – The autogenerated /etc/hosts file on web1

1.  You can also SSH into one of the hosts (for example, `ans-web1`) and ping any of the other hosts by name (for example, `ans-db2`):

    ```
    ssh packt@ans-web1
    ping response:
    ```

![Figure 17.66 – Successful ping by hostname from one host to another](img/B19682_17_66.jpg)

Figure 17.66 – Successful ping by hostname from one host to another

This concludes our study of Ansible templates. However, the topics we covered in this section barely scratch the surface of the powerful features and versatility of Jinja2 templates. We strongly encourage you to explore the related online help resources at [https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html), as well as the titles mentioned in the *Further reading* section at the end of this chapter.

Now, we will turn our attention to another essential feature of modern configuration management platforms: sharing reusable and flexible modules for a variety of system administration tasks. Ansible provides a highly accessible and extensible framework to accommodate this functionality – **Ansible roles** and **Ansible Galaxy**. In the next section, we’ll look at roles for automation reuse.

## Creating Ansible roles

With Ansible roles, you can bundle your automated workflows into reusable units. A role is essentially a package containing playbooks and other resources that have been adapted to a specific configuration using variables. An arbitrary playbook would invoke a role by providing the required parameters and running it just like any other task. Functionally speaking, roles encapsulate a generic configuration management behavior, making them reusable across multiple projects and even shareable with others.

Here are the key benefits of using roles:

*   Encapsulation functionality provides standalone packaging that can easily be shared with others. Encapsulation also enables **separation of concerns** (**SoC**): multiple DevOps and system administrators can develop roles in parallel.
*   Roles can make larger automation projects more manageable.

In this section, we will describe the process of creating a role and how to use it in a sample playbook. When authoring roles, we usually follow these steps and practices:

*   Create or initialize the role directory’s structure. The directory contains all resources required by the role in a well-organized fashion.
*   Implement the role’s content. Create related playbooks, files, templates, and so on.
*   Always start from simple to more advanced functionality. Test your playbooks as you add more content.
*   Make your implementation as generic as possible. Use variables to expose related customizations.
*   Don’t store secrets in your playbooks or related files. Provide input parameters for them.
*   Create a dummy playbook with a simple play running your role. Use this dummy playbook to test your role.
*   Design your role with user experience in mind. Make it easy to use and share it with others if you think it would bring value to the community.

At a high level, creating a role involves the following steps:

1.  Initializing the role directory structure
2.  Authoring the role’s content
3.  Testing the role

We’ll use the `create-users3.yml` playbook we created earlier in the *Using Ansible playbooks* section as our example for creating a role. We will copy this file with a new name; `create-users-role.yml`, for example. Before proceeding with the next steps, let’s add the following line to our `ansible.cfg` file (which is located inside the `~/ansible` directory) in the `[``defaults]` section:

```
roles_path = ~/ansible
```

This configuration parameter sets the default location for our roles.

Now, let’s start by initializing the role directory.

#### Initializing the role directory’s structure

Ansible has a strict requirement regarding the folder structure of the role directory. The directory must have the same name as the role; for example, `create-users-role`. We can create this directory manually or by using a specialized command-line utility for managing roles, called `ansible-galaxy`.

To create the skeleton of our role directory, run the following command:

```
ansible-galaxy init create-users
```

The command completes with the following message:

```
- Role create-users was created successfully
```

You can display the directory structure using the `tree` command:

```
tree
```

You’ll have to manually install the `tree` command-line utility using your local package manager. The output shows the `create-users-role` directory structure of our role:

![Figure 17.67 – The create-users-role role directory](img/B19682_17_67.jpg)

Figure 17.67 – The create-users-role role directory

Here’s a brief explanation of each folder and the corresponding YAML file in the role directory:

*   `defaults/main.yml`: The default variables for the role. They have the lowest priority among all available variables and can be overwritten by any other variable.
*   `files`: The static files referenced in the role tasks.
*   `handlers/main.yml`: The handlers used by the role. Handlers are tasks that are triggered by other tasks. You can read more about handlers at [https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html).
*   `README.md`: Explains the intended purpose of the role and how to use it.
*   `meta/main.yml`: Additional information about the role, such as the author, licensing model, platforms, and dependencies on other roles.
*   `tasks/main.yml`: The tasks played by the role.
*   `Templates`: The template files referenced by the role.
*   `tests/test.yml`: The playbook for testing the role. The `tests` folder may also contain a sample `inventory` file.
*   `vars/main.yml`: The variables used internally by the role. These variables have high precedence and are not meant to be changed or overwritten.

Now that we are familiar with the role directory and the related resource files, let’s create our first role.

#### Authoring the role’s content

It is a common practice to start from a previously created playbook and evolve it into a role. We’ll take the `create-users-role.yml` playbook. Let’s refactor these files to make them more generic.

We will create two new files for the users and passwords, called `users-role.yml` and `users_passwords-role.yml`, inside the `./create-users-role` directory files:

![Figure 17.68 – The users-role.yml and users_passwords-role.yml files](img/B19682_17_68.jpg)

Figure 17.68 – The users-role.yml and users_passwords-role.yml files

As you may have noticed, we renamed the example user accounts and gave them more generic names. We also changed the user dictionary key name from `webusers` to `list` in the `users-role.yml` file. Remember that Ansible requires root-level dictionary entries (key-value pairs) in YAML files that provide variables.

Let’s look at the updated `create-users-role.yml` playbook:

![Figure 17.69 – The modified create-users-role.yml file](img/B19682_17_69.jpg)

Figure 17.69 – The modified create-users-role.yml file

We made the following modifications:

*   We readjusted the `loop` directive to read `users.list` instead of `users.webusers` due to the name change of the related dictionary key in the `users.yml` file
*   We refactored the `include_vars` file references to use variables instead of hardcoded filenames
*   We added a `vars` section, with the `users_file` and `passwords_file` variables pointing to the corresponding YAML files

With these changes in the playbook, we’re now ready to implement our role. Looking at the `create-users-role` role directory, we’ll do the following:

1.  Copy/paste the variables in the `vars` section of `create-users-role.yml` into `defaults/main.yml`.
2.  Copy/paste the tasks from `create-users-role.yml` into `tasks/main.yml`. Make sure you keep the relative indentations.
3.  Create a simple playbook using the role. Use the `tests/test.yml` file for your test playbook. Copy/move `users-role.yml` and `users_passwords-role.yml` to the `tests/` folder.

The following screenshot captures all these changes:

![Figure 17.70 – The files that we changed in the create-users role directory](img/B19682_17_70.jpg)

Figure 17.70 – The files that we changed in the create-users role directory

We also recommend updating the `README.md` file in the `create-users-role` directory with notes about the purpose and usage of the role. You should also mention the requirement of having the `users-role.yml` and `users_passwords-role.yml` files with the related data structures. The names of these files can be changed via the `users_file` and `passwords_file` variables in `defaults/main.yml`. You can also provide some examples of how to use the role. We also created an additional `test2.yml` playbook using a task to run the role:

![Figure 17.71 – Running a role using a task](img/B19682_17_71.jpg)

Figure 17.71 – Running a role using a task

At this point, we’ve finished making the required changes for implementing the role. You may choose to remove all empty or unused folders in the `create-users-role` role directory.

Now, let’s test our role.

#### Testing the role

To test our role, we will use the playbooks in the `tests/` folder and run them with the following commands:

```
ansible-playbook create-users/tests/test.yml
ansible-playbook create-users/tests/test2.yml
```

Both commands should complete successfully.

With that, we’ve provided an exploratory view of Ansible roles, as they are a powerful feature of Ansible, and they enable modern system administrators and DevOps to move quickly from concept to implementation, accelerating the deployment of everyday configuration management workflows.

# Summary

In this chapter, we covered significant ground in terms of Ansible. Due to this chapter’s limited scope, we couldn’t capture all of Ansible’s vast number of features. However, we tried to provide an overarching view of the platform, from Ansible’s architectural principles to configuring and working with ad hoc commands and playbooks. You learned how to set up an Ansible environment, with several managed hosts and a control node, thereby emulating a real-world deployment at a high level. You also became familiar with writing Ansible commands and scripts for typical configuration management tasks. Most of the commands and playbooks presented throughout this chapter closely resemble everyday administrative operations.

Whether you are a systems administrator or a DevOps engineer, a seasoned professional, or on the way to becoming one, we hope this chapter brought new insights to your everyday Linux administration tasks and automation workflows. The tools and techniques you’ve learned here will give you a good start for scripting and automating larger portions of your daily administrative routines.

The same closing thoughts also apply to this book in general. You have come a long way in terms of learning and mastering some of the most typical Linux administration tasks in on-premises and cloud environments alike.

We hope that you have enjoyed our journey together.

# Questions

Let’s try to wrap up some of the essential concepts we learned about in this chapter by completing the following quiz:

1.  What are idempotent operations or commands in Ansible?
2.  You want to set up passwordless authentication with your managed hosts. What steps should you follow?
3.  What is the ad hoc command for checking communication with all your managed hosts?
4.  Enumerate a few Ansible modules. Try to think of a configuration management scenario where you could use each module.
5.  Think of a simple playbook that monitors the memory that’s available on your hosts and will notify you if that memory is above a given threshold.

# Further reading

Here are a few resources we found helpful for learning more about Ansible internals:

*   Ansible documentation: [https://docs.ansible.com/](https://docs.ansible.com/)
*   Ansible use cases, by Red Hat: [https://www.ansible.com/use-cases](https://www.ansible.com/use-cases)
*   *Dive into Ansible – From Beginner to Expert in Ansible* [Video], by James Spurin, Packt Publishing ([https://www.packtpub.com/product/dive-into-ansible-from-beginner-to-expert-in-ansible-video/9781801076937](https://www.packtpub.com/product/dive-into-ansible-from-beginner-to-expert-in-ansible-video/9781801076937))
*   *Practical Ansible 2*, by Daniel Oh, James Freeman, Fabio Alessandro Locati, Packt Publishing ([https://www.packtpub.com/product/practical-ansible-2/9781789807462](https://www.packtpub.com/product/practical-ansible-2/9781789807462))