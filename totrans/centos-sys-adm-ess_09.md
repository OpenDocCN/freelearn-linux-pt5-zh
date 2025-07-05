# Chapter 9. Puppet – Now You Are the Puppet Master

Puppet from Puppet Labs allows for central administration of your Linux devices. The central Puppet server is known as the Puppet master, continuing the analogy with puppetry. This master device certainly allows you to control servers and desktops (nodes in Puppet terms) from a single device, albeit not in the marionette style with pieces of a string. The Puppet master specifies the desired state to each node, and every 30 minutes, the node connects to the Puppet master and sends facts about its resources; if it does not meet the desired state, then the node will fix itself to meet it. During the course of this chapter, we will investigate the Puppet configuration including:

*   **Installing the Puppet master**: We will install and configure the Puppet master from the Puppet Labs repository. The Puppet master will act as the central configuration server and store the desired configuration state for each node.
*   **Puppet resource**: We will use the `puppet resource` command to manually manage resources on the node. Resources represent the fundamental building blocks of a desired state and can include files, users, services, cron jobs, and software packages on the node.
*   **Managing packages, services, and file**s: These three resources represent the main trifecta in Puppet management, and if we can manage these, we can pretty much manage the node. We will create the resource declarations in manifest files and test them on local and remote puppet agents.

# Installing the Puppet master

As we know with many services that have to be installed on CentOS, we have to make sure that the plumbing is correct before we start. The plumbing, in the case of the Puppet master, means:

*   TCP port `8140` must be open through the firewall
*   The Puppet master should be resolvable by the hostname `puppet` by DNS or local hosts files
*   Time should be synchronized
*   A configured Puppet Labs yum repository

I've detailed these in the following sections.

## Configuring the firewall

I am not using a host-based firewall in the demonstration machine. This may not be the case on your systems, and if you are using a firewall, you will need to allow TCP port `8140` through the INPUT chain. The status of the firewall can be checked with the following command:

```
# iptables -L

```

This will list the rules that are in place and the default policy. If there are no rules in place and the default policy is ACCEPT, then you will not have firewall-related issues, and you can relax.

## DNS

Puppet agents are running on each client or node and will try and communicate with the Puppet master using the default hostname, `puppet`. This can be changed in the `/etc/puppet.conf` file. This change will need to be implemented on each agent so it is often the easiest way to create an ADDRESS record or CNAME record in the local DNS, which can resolve the host puppet to the IP address of your desired Puppet master host. In the demonstration lab, I have the correct CNAME record in place. Using the `ping` command, we can see that the hostname is resolvable, and the output is shown in the following screenshot:

![DNS](img/5920OS_09_01.jpg)

## Network Time Protocol

If you have not already configured the time on your Puppet master server and agent nodes, then you should do so using the **Network Time Protocol** (**NTP**). This will ensure that they all share the same accurate time. Accurate time is required across all devices, as the Puppet master will act as a certificate server issuing certificates to trusted nodes, and the timestamp on the certificate cannot be in the future. In setting up NTP on the Puppet master, we will first synchronize the time to an NTP server and then configure regular time updates using NTP and entries stored within the `/etc/ntp.conf` file, as follows:

```
# ntpdate uk.pool.ntp.org

```

The previous command does a single, one-off update with a UK-based NTP server. This sets the time so that regular updates may take place. If this is not set, then it is possible that the time will not be synchronized as the NTP client must be within 1,000 seconds of the NTP server for regular updates to take place. We can now start the NTP service and configure it for autostart. If we do not make any changes to the configuration file, `/etc/ntp.conf`, then time will synchronize with servers from the NTP pool. If you have a local time server already up and running on your network, then it will be worthwhile to use that device as the time source as follows:

```
# service ntpd start
# chkconfig ntpd on

```

## The Puppet lab repository

The Puppet master is not available in the standard CentOS repositories, nor unfortunately, the EPEL repositories, which we have already configured. This requires us to add the Puppet Labs software repositories. These repositories will provide the latest Puppet agent and master software. The Puppet agent is required on all nodes and the master software on the server. We will create the YUM repository for Puppet directly by installing the RPM from a web URL. The RPM will define only the repository file in `/etc/yum.repos.d`.

```
#  rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm

```

This will complete quickly, as the only file that it needs to create is the repository definition. With the repository set, we are now ready to install the Puppet master. The following command will install the Puppet master and agent to the latest version available from the Puppet Labs repositories:

```
# yum install puppet-server

```

As always, we should start the service and enable it for autostart. When this is done, we can use the `netstat` command to show that the Puppet master is listening on the TCP port `8140`:

```
# service puppetmaster start
# chkconfig puppetmaster on
# netstat -antl | grep :8140

```

# Puppet resource

Versions of Puppet from 2.6 and later (the current release is 3.6) use the single binary puppet with subcommands for specific tasks. The earlier version had separate binaries for all of the subcommands. In the previous set of commands, we used the traditional CentOS syntax to start the Puppet master and then to enable the service for autostart; we could achieve the same result using the `/usr/bin/puppet` command along with the `resource` subcommand:

```
# puppet resource Service puppetmaster enable=true ensure=running

```

With this command, we direct our attention to the `puppetmaster` service, enable it for autostart (`enable=true`), and start it if required (`ensure=running`). This represents the very essence of how Puppet works. Of course, to manage many clients, we will create manifest files with similar resource rules to enforce the desired state. In itself though, we will configure the desired state of the node with the use of the `puppet resource` command.

Along with setting the desired state, we can view the state of all services or a single named service using very similar commands; the following are two such commands; the first command will display all services, and the second command will display only the `puppetmaster` service:

```
# puppet resource Service
# puppet resource Service puppetmaster

```

The output from the command specific to the `puppetmaster` service is shown in the following screen capture:

![Puppet resource](img/5920OS_09_02.jpg)

As mentioned earlier in the introduction to this chapter, the three main resources that we manage with Puppet include:

*   Service
*   Files
*   Packages

Along with these main resources, we have others, which include the following:

*   Users
*   Groups
*   Cron jobs
*   Notify
*   Yumrepo
*   `ssh_authorized_key`
*   Interface

To gain an understanding of how Puppet can manage these resources, we will work through an example using the `puppet resource` command to manually enforce a desired state on our node. Even though the node on which we run the command is the Puppet master, for all intents and purposes, we will only use the client, which is the Puppet agent at this stage. The example we used earlier with puppet resource demonstrates what can be achieved with Puppet before moving the desired state configuration into manifest files on the Puppet master.

Using puppet resource user, we can ensure that a user account is present on a system, by referencing an account that does not exist, Puppet will create the account and set the given attribute's password. If we need to delete an account, we can use the `ensure=>absent` attribute.

To begin, we must obtain the encrypted password for the new user account. There are different mechanisms that can be used to do this, but here I will use Python from the command line to generate the password:

```
# python -c 'import crypt; print crypt.crypt("Password1","$5$RA")'

```

The output from this command will be the SHA-256 password to be used by the new user. We are now ready to use Puppet to create the user:

```
# puppet resource User newuser ensure=present uid='2222' gid='100' home='/home/newuser' managehome=true shell='/bin/bash' password='<encrypted password>'

```

### Tip

Blocks of code like these that describe the resource are known as **resource declarations**.

This will create the user with the set of desired attribute values. The home directory for the user will be created along with the user account. This behavior is controlled with the `managehome` attribute; setting this to be `true` will create the directory.

Although we would not want this set manually on all servers, as in this case, we could use a similar method to allow periodic password changes for the root account across all nodes as well as ensure other system accounts exist.

# Managing packages, services, and files

We will move on from this manual configuration and become familiar with Puppet as a central configuration server, whereby we can define settings within manifest files that will be distributed to the required nodes. To begin this, we will create the manifest file; these are just text files, and apply it locally on the Puppet master using `puppet apply`. Once we have verified that the manifest is working and enforcing the desired state, we will enlist the clients and see true Puppet automation at work.

The building blocks for Puppet start with the resource declarations that we have already looked at. These declarations are written to manifest files, which have the extension `.pp`. Within the manifest file, resources can be grouped together into classes. A class often represents related resources, such as the `openssh-server` package, the `sshd` service, and the `/etc/ssh/sshd_config` configuration file. It would seem reasonable to group these resources together in a class definition.

We can view these building blocks by taking a look inside an example manifest file, as shown in the following diagram:

![Managing packages, services, and files](img/5920OS_09_03.jpg)

## Classes

Classes are reusable because a class can be used by multiple node definitions and are said to be `singleton` in the sense that once a given class is used on a node, it can only be used once and cannot be redeclared for that node. The class we have created here is named `ssh`. A class has to be first defined and then declared. The following code block is an example of a class definition:

```
class web-servers {
  code……
}
```

The following example code shows the same class being declared:

```
include web-servers
```

## Resource definition

Resource definitions, such as what we looked at earlier for the user resource, do not need to be enclosed within classes; however, related resources are often grouped together by means of the class for ease of assignment to nodes. In this example, we define a file resource and a service resource. The name of the service resource must match the name of the service on the node to which it will be assigned; in the case of CentOS, the OpenSSH server is the `sshd` service.

A resource in Puppet is an instance of a specific resource type. To list all the available types in CentOS, we can use the `describe` subcommand:

```
# puppet describe -l

```

A resource type has a defined schema that states which attributes are available. To list the schema details of a resource type, we can use the `describe` subcommand again:

```
# puppet describe -s User
# puppet describe User

```

A short description is shown with and without the `-s` option; a full listing of the resource type schema is listed. In the previous commands, we display information for the `user` resource type.

Earlier in this chapter, we created a new user account from the command line using `puppet resource`. If we needed a system account on many nodes and wished that Puppet provision the account, we can create a `user` resource definition within a manifest file similar to the following:

```
user { 'puppetuser' :
  ensure => present,
  uid => '99',
  gid => '99',
  shell => '/bin/false',
  password => '$5$RA$e7cMcsFNqvFkZrlm62fnzy0vpN2GxrOjzpsLaVQzIc4',
  home => '/tmp',
  managehome => false,
}
```

## Puppet facts

In the example manifest we listed earlier, we defined a file resource for the `/etc/motd` file. This is displayed when a user logs into the system, be it locally or via a remote SSH connection. The Puppet agent will compare facts from the node's configuration to see if it matches the desired state. These facts are gathered from the machine's configuration using the `/usr/bin/facter` command. We can display these facts in the following way:

```
$ facter

```

The preceding command will display all the facts, whereas the following command will display just the IP address:

```
$ facter ipaddress

```

We can further expand the resource definition using additional attributes for the file and fill out the content with some facts as follows:

```
file {'/etc/motd' :
  ensure => file,
  mode => 0644,
  content => "Welcome to TUP
This is a ${operatingsystem}  ${operatingsystemrelease} host with IP ${ipaddress}
",
}
```

If this resource definition was applied to a node, it would ensure that the file is of the type "file"; rather than a directory, the permission would be set to `rw- r-- r--`, and the contents would expand with three variables created from facts. This will create contents similar to the following screenshot:

![Puppet facts](img/5920OS_09_04.jpg)

Remember that we only need to create the resource definition once. On the Puppet server, this one definition will then be applied to all the nodes it is assigned to. However, using variables based on facts from each node, we can create unique content for each individual `/etc/motd` file.

## Using include

The `include` statement declares the use of the class. If we define a class and do not use the `include` statement, then none of the resource definitions will be used. The class can be defined within the same manifest in which it is declared, but more often, classes are defined in separate manifest files created within the puppet module path. The `modulepath` defaults to the `/etc/puppet/modules` and `/usr/share/puppet/modules` directories. You can view the module path, which is colon delimited, using the following command:

```
# puppet config print modulepath

```

The output from my CentOS system shows the default settings, as shown in the following screen capture:

![Using include](img/5920OS_09_05.jpg)

## Creating and testing manifests

Manifests are ASCII text files that have the `.pp` extension. These files contain class declarations and/or resource definitions. Classes are also defined and declared within manifests; however, as mentioned before, they are often defined in separate manifest files to those where they are declared. This allows for greater modularity of your code. The manifest file can be supplied as a local file and invoked via the `apply` subcommand of Puppet or, more often, from the Puppet master. We will apply the manifest locally using `puppet apply`. The file that we will create will be consistent with a client server deployment so that we can reuse the same file once we have tested it locally; for this, we will create the file as `/etc/puppet/manifests/site.pp`. Nodes, when connecting to the Puppet master, will look for the `site.pp` file for their configuration. The example manifest is shown in the following code:

```
class tup {
  file {'/etc/motd' :
  ensure => file,
  mode => 0644,
  owner => 'root',
  group => 'root',
  content => "Welcome to TUP
This is a ${operatingsystem}  ${operatingsystemrelease} host with IP ${ipaddress}
",
 }
service {'sshd':
   ensure => running,
   enable => true,
 }
package { 'openssh-server' :
    ensure => installed,
 }
}
include tup
```

With the manifest created and saved under `/etc/puppet/manifests/site.pp`, we can validate the syntax of the file with the following command:

```
# puppet parser validate  /etc/puppet/manifests/site.pp

```

If errors can be seen, then we can re-edit the file to correct these errors, and when the output is error free, we can manually apply the file:

```
# puppet apply /etc/puppet/manifests/site.pp

```

Using the `cat` command, we can validate the contents of the `/etc/motd` file:

```
$ cat /etc/motd

```

If we now stop the `sshd` service and change the permissions of the file, we can see how Puppet ensures a consistent configuration:

```
# service sshd stop
# chmod 777 /etc/motd

```

With the changes made, we have diverged from the desired state and can now reapply the manifest; under normal client-server operations, the Puppet agent will check into the server every 30 minutes.

```
# puppet apply /etc/puppet/manifests/site.pp

```

The output should include notices similar to those in the following screenshot, indicating that the service has been started and the mode has been changed:

![Creating and testing manifests](img/5920OS_09_06.jpg)

## Enrolling remote puppet agents

As we saw, Puppet can be effective in maintaining a consistent configuration, but we do not want to create the manifests on each device and run the puppet commands manually. To see the real strength of Puppet as a central configuration server, we need to enroll clients and have the puppet agent run as a service. As we mentioned before, the agent will check its desired state every 30 minutes automatically when the agent service is running.

From a remote CentOS 6.5 system, we will check whether we can resolve the hostname of the Puppet master, using the follow command line:

```
$ host puppet

```

As seen before, we will need to ensure that we have time synchronization on our remote node:

```
# ntpdate uk.pool.ntp.org
# service ntpd start
# chkconfig ntpd on

```

We will add the remote Puppet labs repository to the remote client CentOS system:

```
# rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el6.noarch.rpm

```

Finally, we will install the Puppet agent on the client system and display the version of Puppet:

```
# yum install puppet
# puppet --version

```

At the time of writing this, the version of the Puppet Labs repository is 3.6.2.

We are now ready to test the client. The first step towards this is to start the agent manually so that we can enroll the node on the server. This will submit a certificate signing request to the Puppet master, as the node is not yet enrolled:

```
# puppet agent test

```

Returning now to the console of the Puppet master, we can check the certificate authority for agent signing request:

```
# puppet ca list

```

We should see the request from the client machine on our lab setup; the client request shows `centos65.tup.com`. We can accept and sign this request using the following command:

```
 #  puppet ca --sign centos65.tup.com

```

We will now return to the client machine and rerun the test agent; this will download the signed certificate, and the agent will then download and apply the `site.pp` manifest:

```
# puppet agent test

```

We can now check the contents of the `/etc/motd` file. We should have the content we saw before, but with the IP address of this node. Using the `cat` command from the remote client machine, the output will look similar to the following screenshot:

![Enrolling remote puppet agents](img/5920OS_09_07.jpg)

Now that we have installed the signed certificate onto the client, we can start the agent service and leave the system to manage itself; we can have even more time on the golf course now!

```
# service puppet start
# chkconfig puppet on

```

On CentOS, the agent service is just Puppet and with the service running, the agent will check the configuration every 30 minutes.

# Summary

In this chapter, we looked at how we can implement central configuration management using Puppet. Although we only looked at it on CentOS, the configuration can work across many operating systems including Linux, Windows, and Unix. The main server is the Puppet master and agents connected on the TCP port `8140` to download the site manifest. This manifest can include other classes but will determine the desired configuration for a node.

As we move onto the next chapter, we will look at how we can use **pluggable authentication modules** (**PAM**) to help harden the CentOS host, as well as venture into the world of SELinux.