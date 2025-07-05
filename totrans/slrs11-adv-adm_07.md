# Chapter 7. Configuring and Administering RBAC and Least Privileges

In this chapter, we will cover the following topics:

*   Configuring and using RBAC
*   Playing with least privileges

# Introduction

**Role-based access control** (**RBAC**) is an amazing feature, which also exists on Oracle Solaris 11 (its origin was in Oracle Solaris 8), that primarily makes it possible to restrict the granted privileges to a normal user for executing tasks. Putting this another way, RBAC makes it feasible to delegate only the necessary privileges for a regular user to be able to accomplish administrative tasks in a way similar to that of a sudo program. When compared with a sudo program, the main difference is the fact that RBAC is completely integrated in the operating system, and it is used during the user logon process to Oracle Solaris 11\. Moreover, RBAC offers a more granular access to privileges than sudo does, and integration with another great feature from Oracle Solaris 11 named least privilege, which is used to cut out unnecessary privileges from processes and programs, allows you to reduce the attack surface of a hacker.

# Configuring and using RBAC

Before explaining and implementing the RBAC feature, it is necessary to remember why RBAC is necessary and, afterwards, to learn some fundamental concepts.

According to our previous study on Oracle Solaris 11, it would not be possible for a normal user to reboot an Oracle Solaris 11 system, as shown in the following command:

```
root@solaris11-1:~# useradd -d /export/home/aborges -m -s /bin/bash aborges
80 blocks
root@solaris11-1:~# passwd aborges
New Password: hacker123!
Re-enter new Password: hacker123!
passwd: password successfully changed for aborges
root@solaris11-1:~# su - aborges
Oracle Corporation  SunOS 5.11  11.1  September 2012
aborges@solaris11-1:~$ reboot
reboot: permission denied
aborges@solaris11-1:~$
```

A simple and completely inappropriate solution would be to give a password from the `root` account to user `aborges`. However, this is unimaginable in a professional company. Another and a recommended solution is to use RBAC, which is a security feature that allows regular users to accomplish administrative tasks such as rebooting a system, as we have tried previously.

The RBAC framework contains the following objects:

*   **Role**: This is a special type of user that is created to execute administrative tasks, although it isn't possible to log in to a system and the correct procedure is to log in as a user and to assume the role using the `su` command. As the role is a kind of user, it is configured in the `/etc/passwd` file and it has a password defined in the `/etc/shadow` file. However, different from a user, it isn't possible to log in to Oracle Solaris 11 using a role. The user must log in using a normal account and then they can assume a role using the `su` command.
*   **Profile**: This is a set of commands. Any role assigned to a profile can execute any command from this profile. All system profiles are defined in the `/etc/security/prof_attr.d/core-os` file, and local profiles can be defined in the `/etc/security/prof_attr` file. To list all the profiles, use the following command:

    ```
    root@solaris11-1:~# getent prof_attr | more
    Software Installation:RO::Add application software to the system:auths=solaris.smf.manage.servicetags;profiles=ZFS File System Management;help=RtSoftwareInst
    all.html
    NTP Management:RO::Manage the NTP service:auths=solaris.smf.manage.ntp,solaris.smf.value.ntp
    Desktop Configuration:RO::Configure graphical desktop software:auths=solaris.smf.manage.dt.login,solaris.smf.manage.x11,solaris.smf.manage.font,solaris.smf.m
    anage.opengl
    Device Security:RO::Manage devices and Volume Manager:auths=solaris.smf.manage.dt.login,solaris.device.*,solaris.smf.manage.vt,solaris.smf.manage.allocate;he
    lp=RtDeviceSecurity.html
    Desktop Removable Media User:RO::Access removable media for desktop user:
    (truncated output)

    ```

*   **Authorization**: This represents a special form of privilege that is set in order to accomplish specific tasks, such as accessing a CD-ROM and managing the CUPS printing service, NTP service, Zones, SMF framework, and so on. Typically, authorizations are created either from the Oracle Solaris installation or from new installed software. All system authorizations are defined in the `/etc/security/auth_attr.d/core-os` file, and local authorizations are defined in the `/etc/security/auth_attr` file. To list all the authorizations, we run the following command:

    ```
    root@solaris11-1:~# getent auth_attr | more
    solaris.smf.read.ocm:::Read permissions for protected Oracle Configuration Manager Service Properties::
    solaris.smf.value.ocm:::Change Oracle Configuration Manager System Repository Service values::
    solaris.smf.manage.ocm:::Manage Oracle Configuration Manager System Repository Service states::
    solaris.smf.manage.cups:::Manage CUPS service states::help=ManageCUPS.html
    solaris.smf.manage.zfs-auto-snapshot:::Manage the ZFS Automatic Snapshot Service::
    solaris.smf.value.tcsd:::Change TPM Administation value properties::
    (truncated output)

    ```

*   **Privilege**: This is a singular right that can be assigned to a user, role, command, or even a system.
*   **Execution attributes**: These are commands that are defined in the `/etc/security/exec_attr.d/core-os` (system execution attributes) or `/etc/security/exec_attr` files (local definitions), and they are assigned to one or more profiles. To list all the execution attributes, we run the following command:

    ```
    root@solaris11-1:~# getent exec_attr | more
    DTrace Toolkit:solaris:cmd:::/usr/dtrace/DTT/*/*:privs=dtrace_kernel,dtrace_proc,dtrace_user
    Desktop Configuration:solaris:cmd:RO::/usr/bin/scanpci:euid=0;privs=sys_config
    Desktop Configuration:solaris:cmd:RO::/usr/X11/bin/scanpci:euid=0;privs=sys_config
    OpenLDAP Server Administration:suser:cmd:RO::/usr/sbin/slapd:uid=openldap;gid=openldap;privs=basic,net_privaddr
    OpenLDAP Server Administration:suser:cmd:RO::/usr/sbin/slapacl:uid=openldap;gid=openldap
    (truncated output)

    ```

*   **Profile shell**: This is a special kind of profile (`pfbash`, `pfsh`, `pfcsh`, or `pfzsh`) assigned to users during a `su` command to assume a role or a login shell that allows access to specific privileges. It is necessary to use any one of these profile shells.
*   **Security policy**: This defines default privileges and profiles for users. The related configuration file is `/etc/security/policy.conf`, as shown in the following command:

    ```
    root@solaris11-1:~# more /etc/security/policy.conf

    ```

There are two ways to use RBAC. The first method is simpler and more straightforward; you can create and assign a profile directly to a user account in order to log in as a normal user and use the `pfexec` command to execute additional commands from the assigned profile.

The second method is to put all mentioned concepts about RBAC (commands, authorizations, profiles, roles, and users) together following a schema as shown next (from right to left):

User <-- Role <-- Profile <-- Commands and/or Authorizations

The second method is more complex, and the required steps to use RBAC, as described in the previous sequence, are as follows:

1.  Create a role using the `roleadd` command.
2.  Create a profile, editing the `/etc/security/prof_attr` file.
3.  Assign commands to the created profile (step 2) in `/etc/security/exec_attr` or assign authorizations (`/etc/security/auth_attr`) to the profile in the `/etc/security/prof_attr` file.
4.  Assign the profile to the role using the `rolemod` command.
5.  Create a password for the role using the `passwd` command.
6.  Assign one or more users to the role using the `usermod` command.
7.  When the user needs to use the assigned commands, execute `su - <rolename>`.

This is nice! This is a summary of the concepts required to manage RBAC. We will learn how to execute a step-by-step procedure for both methods.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 and with at least 2 GB RAM.

## How to do it…

We are going to learn both the methods to allow a regular user to be able to reboot a system, that is, using the `pfexec` command (simpler) and RBAC's role (more complex).

Using the `pfexec` command is easy. First, create the `aborges` regular user with `hacker123!` as the password, as shown in the following commands:

```
root@solaris11-1:~# useradd -d /export/home/aborges -m -s /bin/bash aborges
80 blocks
root@solaris11-1:~# passwd aborges
New Password: hacker123!
Re-enter new Password: hacker123!
passwd: password successfully changed for aborges
```

The main idea is to associate a profile (that is, a set of commands) directly to the user (`aborges`). In this case, the desired profile already exists; if not, we have to create a new one. To avoid creating an unnecessary profile, verify that there is a line in the `/etc/security/exec_attr.d/core-os` file with the `reboot` command by executing the following command:

```
root@solaris11-1:~# cat /etc/security/exec_attr.d/core-os | grep reboot
Maintenance and Repair:solaris:cmd:RO::/usr/sbin/reboot:uid=0
```

This is excellent! There is one profile named `"Maintenance and Repair"` that includes the reboot command. For accomplishing our task, associate this profile (using the `–P` option) with the `aborges` user, as shown in the following command:

```
root@solaris11-1:~# usermod -P "Maintenance and Repair" aborges
root@solaris11-1:/# more /etc/user_attr.d/local-entries | grep aborges
aborges::::profiles=Maintenance and Repair
```

As we realized, it created an entry for the `aborges` user in the `/etc/ user_attr.d/local-entries` file. However, even including this entry, which associates the `aborges` user with the `"Maintenance and Repair"` profile, the user is still not able to reboot the system, as shown in the following command:

```
root@solaris11-1:/# su – aborges
Oracle Corporation  SunOS 5.11  11.1  September 2012
aborges@solaris11-1:~$ reboot
reboot: permission denied
```

Nonetheless, if the `aborges` user wants to execute the same command using `pfexec`, the result will be different, as shown in the following command:

```
aborges@solaris11-1:~$ pfexec reboot

```

It worked! The system will be rebooted as expected.

The approach using the `pfexec` command is wonderful, but the mode chosen to configure it (taking a ready profile) can bring about two little side effects:

*   The `"Maintenance and Repair"` profile has other commands, and we have also assigned these commands to the `aborges` user, as shown in the following command:

    ```
    root@solaris11-1:~# cat /etc/security/exec_attr.d/core-os | grep -i "Maintenance and Repair"
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/mdb:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/coreadm:euid=0;privs=proc_owner
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/croinfo:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/date:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/ldd:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/vmstat:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/eeprom:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/halt:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/init:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/pcitool:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/poweroff:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/prtconf:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/reboot:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/syslogd:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/bootadm:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/ucodeadm:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/cpustat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/pgstat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/kstat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/ilomconfig:privs=sys_config,sys_ip_config,sys_dl_config
    Maintenance and Repair:solaris:cmd:RO::/usr/lib/ilomconfig.builtin:privs=sys_config,sys_ip_config,sys_dl_config
    ```

    To prevent this, it would be better to create a new profile and assign only the reboot command to it.

*   The second side effect is that the procedure using the `pfexec` command should be done for each user that needs to use the `reboot` command, but it can take additional time.

The second method to reach our goal is using roles, profiles, and/or authorizations together. The advantage in this case is that privileges are not associated with users directly, but they are assigned to roles instead. Then, if a regular user needs to reboot the system (for example), it assumes the role using the `su` command and executes the appropriate command.

Create another user (different from the previous one) to be used in this method by running the following command:

```
root@solaris11-1:~# useradd -d /export/home/rbactest -m -s /bin/bash rbactest
80 blocks
root@solaris11-1:~# passwd rbactest
New Password: oracle123!
Re-enter new Password: oracle123!
passwd: password successfully changed for rbactest
```

To confirm that the `brbactest` user can't reboot the system, execute the following commands:

```
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012

rbactest@solaris11-1:~$ reboot
reboot: permission denied
```

Create a role that will be configured later by running the following commands:

```
root@solaris11-1:~# roleadd -m -d /export/home/r_reboot -s /bin/pfbash r_reboot
80 blocks
root@solaris11-1:~# grep r_reboot /etc/passwd
r_reboot:x:103:10::/export/home/r_reboot:/bin/bash
root@solaris11-1:~# grep r_reboot /etc/shadow
r_reboot:UP:::::::
```

As we have mentioned previously, profiles are very important and are used during RBAC configuration. The system already has some defined system profiles that are configured in the `/etc/security/prof_attr.d/core-os` file, as shown in the following command:

```
root@solaris11-1:~# more /etc/security/prof_attr.d/core-os 
(truncated output)
All:RO::\
Execute any command as the user or role:\
help=RtAll.html

Administrator Message Edit:RO::\
Update administrator message files:\
auths=solaris.admin.edit/etc/issue,\
solaris.admin.edit/etc/motd;\
help=RtAdminMsg.html

Audit Configuration:RO::\
Configure Solaris Audit:\
auths=solaris.smf.value.audit;\
help=RtAuditCfg.html

Audit Control:RO::\
Control Solaris Audit:\
auths=solaris.smf.manage.audit;\
help=RtAuditCtrl.html
(truncated output)

```

Therefore, according to the suggested steps in the introduction of this recipe, create a profile named `Reboot` at the end of the profile configuration file, as shown in the following commands:

```
root@solaris11-1:~# vi /etc/security/prof_attr
#
# The system provided entries are stored in different files
# under "/etc/security/prof_attr.d".  They should not be
# copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
Reboot:RO::\
For authorized users to reboot the system:\
help=RebootByRegularUser.html

```

We know from this file that the profile name is `Reboot` and the `RO` (read-only) characters indicate that it isn't modifiable by any tool that changes this database. The lines that follow denote the description and the help file (it is unnecessary to create it). It will be possible to bind authorizations (the `auths` key), other profiles (the `profiles` key), and privileges (the `priv` key) to this `Reboot` profile.

Following the profile creation, we have to assign one or more commands to this profile, and local modifications occur by editing the `/etc/security/exec_attr` file, as shown in the following command:

```
root@solaris11-1:~# vi /etc/security/exec_attr
#
# The system provided entries are stored in different files
# under "/etc/security/exec_attr.d".  They should not be
# copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
Reboot:solaris:cmd:RO::/usr/sbin/reboot:uid=0

```

```
 explained as follows:
```

*   `Reboot`: This is the profile name.
*   `solaris`: This is the security policy associated with the `Reboot` profile. This security policy is able to recognize privileges. Oracle Solaris 11 has another possible value for this field named `suser` (not shown previously), which is very similar to the `solaris` value, but it is not able to understand and recognize privileges.
*   `cmd`: This is a type of object. In this case, it is a command to be executed by a shell.
*   `RO`: This indicates that this line isn't modifiable by any tool that changes this file.
*   `/usr/sbin/reboot`: This is the command to be executed by a user when they assume the role that contains this `Reboot` profile.
*   `Uid=0`: This command is run with the real ID of the user's root (`uid=0`). This is the case when a user has to run the command; the command will be executed as run by a root user. Other good and useful possible keys are `euid` (effective user ID, which is similar to running a command with `setuid` set as the executable) and `privs` (privileges).

Again, it is very interesting to check the already existing system execute attributes defined in the `/etc/security/exec_attr.d/core-os` file, as shown in the following command:

```
root@solaris11-1:~# more /etc/security/exec_attr.d/core-os 
(truncated output)
All:solaris:cmd:RO::*:
Audit Control:solaris:cmd:RO::/usr/sbin/audit:privs=proc_owner,sys_audit
Audit Configuration:solaris:cmd:RO::/usr/sbin/auditconfig:privs=sys_audit
Audit Review:solaris:cmd:RO::/usr/sbin/auditreduce:euid=0
Audit Review:solaris:cmd:RO::/usr/sbin/auditstat:privs=proc_audit
Audit Review:solaris:cmd:RO::/usr/sbin/praudit:privs=file_dac_read
Contract Observer:solaris:cmd:RO::/usr/bin/ctwatch:\
  privs=contract_event,contract_observer
Cron Management:solaris:cmd:RO::/usr/bin/crontab:euid=0
Crypto Management:solaris:cmd:RO::/usr/sbin/cryptoadm:euid=0
Crypto Management:solaris:cmd:RO::/usr/bin/kmfcfg:euid=0
Crypto Management:solaris:cmd:RO::/usr/sfw/bin/openssl:euid=0
Crypto Management:solaris:cmd:RO::/usr/sfw/bin/CA.pl:euid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/dhcpconfig:uid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/dhtadm:uid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/pntadm:uid=0
(truncated output)

```

It's time to bind the `r_reboot` role to the `Reboot` profile (the `–P` option) by executing the following commands:

```
root@solaris11-1:~# rolemod -P Reboot r_reboot
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;profiles=Reboot;roleauth=role

```

According to the previous output, `r_reboot` is of type `role` and it is associated with the `Reboot` profile.

The `r_reboot` role does not have any password, so we should set a new password for it by running the following command:

```
root@solaris11-1:~# passwd r_reboot
New Password: hacker321!
Re-enter new Password: hacker321!
passwd: password successfully changed for r_reboot
root@solaris11-1:~# grep r_reboot /etc/shadow
r_reboot:$5$q75Eiy5/$u9mgnYsvlszbNXkSuH4kZwVVnFOhemnCTMF//cvBWD9:16178::::::19216
```

The RBAC configuration is almost complete. To assume this `r_reboot` role, the `rbactest` user must be assigned to it by using the `-R` option from the `usermod` command, as shown in the following command:

```
root@solaris11-1:~# usermod -R r_reboot rbactest
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;profiles=Reboot;roleauth=role
rbactest::::roles=r_reboot

```

To confirm every executed task until now, run the following command:

```
root@solaris11-1:~# roles rbactest
r_reboot
root@solaris11-1:~# profiles rbactest
rbactest:
          Basic Solaris User
          All
root@solaris11-1:~# profiles r_reboot
r_reboot:
          Reboot
 Basic Solaris User
 All

```

It is worth remembering that `rbactest` is a user while `r_reboot` is a role, and as explained previously, it is not possible to log in to the system using a role. Additionally, the existing profiles are `Basic Solaris User`, which enables users to use the system according to the established security limits, and `All`, which provides access to the commands that do not have any security attributes.

Continuing the verification, we have to check the authorizations for the `r_reboot` role and the `rbactest` user as well as for the assigned profiles to the `r_reboot` role. These tasks are done by executing the following sequence of commands:

```
root@solaris11-1:~# auths r_reboot
solaris.admin.wusb.read,solaris.mail.mailq,solaris.network.autoconf.read
root@solaris11-1:~# auths rbactest
solaris.admin.wusb.read,solaris.mail.mailq,solaris.network.autoconf.read
root@solaris11-1:~# profiles -l r_reboot
r_reboot:
      Reboot
          /usr/sbin/reboot           uid=0

      Basic Solaris User
      auths=solaris.mail.mailq,solaris.network.autoconf.read,solaris.admin.wusb.read
      profiles=All
              /usr/bin/cdrecord.bin      privs=file_dac_read,sys_devices,proc_lock_memory,proc_priocntl,net_privaddr
          /usr/bin/readcd.bin        privs=file_dac_read,sys_devices,net_privaddr
          /usr/bin/cdda2wav.bin      privs=file_dac_read,sys_devices,proc_priocntl,net_privaddr

      All
          *
```

There are a few points to be highlighted:

*   The `rbactest` user is assigned to the `r_reboot` role.
*   There is no authorization assigned either to the `rbactest` user or to the `r_reboot` role.
*   The `All` profile grants unrestricted access to all unrestricted commands from Oracle Solaris 11\. In this case, the `r_reboot` role is associated with three profiles: `Reboot`, `Basic Solaris User`, and `All`.
*   The `Basic Solaris User` profile can execute some related CD-ROM commands using specific privileges.

Finally, we can verify that the `rbactest` user is able to reboot the system by executing the following command:

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11	11.1	September 2012
rbactest@solaris11-1:~$ id
uid=102(rbactest) gid=10(staff)
rbactest@solaris11-1:~$ profiles
          Basic Solaris User
          All
rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation	SunOS 5.11	11.1	September 2012
r_reboot@solaris11-1:~$ id
uid=103(r_reboot) gid=10(staff)
r_reboot@solaris11-1:~$ profiles
          Reboot
          Basic Solaris User
          All
r_reboot@solaris11-1:~$ reboot

```

The system is reinitiated immediately. That's fantastic!

RBAC allows you to integrate all the concepts that you have learned about (roles, profiles, authorizations, and commands) with privileges; therefore, it offers us a more fine-grained and integrated control than a sudo program does.

When working with Oracle Solaris 11, we can use RBAC with services from the SMF framework. For example, the DNS client and DHCP server have the following authorizations:

```
root@solaris11-1:~# svcprop -p general/action_authorization dns/client
solaris.smf.manage.name-service.dns.client
root@solaris11-1:~# svcprop -p general/action_authorization dhcp/server:ipv4
solaris.smf.manage.dhcp
```

Without these appropriate authorizations, the `rbactest` user isn't able to manage these services, as shown in the following commands:

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ id
uid=102(rbactest) gid=10(staff)
rbactest@solaris11-1:~$ svcadm restart dns/client
svcadm: svc:/network/dns/client:default: Permission denied.
rbactest@solaris11-1:~$ svcadm restart dhcp/server:ipv4
svcadm: svc:/network/dhcp/server:ipv4: Permission denied.
```

It's easy to solve these problems, assigning the respective authorization to the `r_reboot` role, by executing the following command:

```
root@solaris11-1:~# rolemod -A solaris.smf.manage.name-service.dns.client,solaris.smf.manage.dhcp r_reboot

```

To verify that the previous command has worked, check the altered file:

```
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;auths=solaris.smf.manage.name-service.dns.client,solaris.smf.manage.dhcp;profiles=Reboot;defaultpriv=basic,file_dac_read;roleauth=role
rbactest::::defaultpriv=basic,file_dac_read;roles=r_reboot
```

That's nice! It's time to test whether our modifications have worked by executing the following command:

```
root@solaris11-1:~# su - rbactest
Oracle Corporation  SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation  SunOS 5.11  11.1  September 2012
r_reboot@solaris11-1:~$ svcadm -v restart dns/client
Action restart set for svc:/network/dns/client:default.
r_reboot@solaris11-1:~$ svcadm -v restart dhcp/server:ipv4
Action restart set for svc:/network/dhcp/server:ipv4.
```

That's excellent! The integration of RBAC with SMF is perfect, and a normal user such as `rbactest` is able to manage both the services (the DNS client and the DHCP server) as it is the root user.

If we want to unbind the `r_reboot` role from the `rbactest` user to prevent them from rebooting, or to perform any other action on the system, execute the following command:

```
root@solaris11-1:~# roles rbactest
r_reboot
root@solaris11-1:~# usermod -R "" rbactest
root@solaris11-1:~# roles rbactest
root@solaris11-1:~#
```

A final and additional note: it is possible to configure default RBAC authorizations and profiles for every user in the `/etc/security/policy.conf` file. In the same way, there is the option to configure the default privilege and its limit, as shown in the following command:

```
root@solaris11-1:~# more /etc/security/policy.conf 
(truncated output)
AUTHS_GRANTED=
PROFS_GRANTED=Basic Solaris User
CONSOLE_USER=Console User
(truncated output)
#
#PRIV_DEFAULT=basic
#PRIV_LIMIT=all
#
(truncated output)

```

### An overview of the recipe

In this section, we learned how to use RBAC in order to allow a regular user to reboot the system. Furthermore, we have tested how to find and grant the necessary authorization to manage services from the SMF framework. The same procedure should be applied for any user and any number of commands.

# Playing with least privileges

Oracle Solaris 11, like other good UNIX-like operating systems, has a flaw in its inception; there is a privileged account called root that has all special privileges on a system and other accounts that have limited permissions such as regular users. Under this model, a process either has all special privileges or none. Therefore, if we grant permission for a regular user to run a program, usually we are granting much more than is needed, and unfortunately, it could be a problem if a hacker is to crack the application or the system.

In Oracle Solaris 10, developers have introduced a wonderful feature to make the permissions more flexible; **least privilege**. The base concept is easy; the recommendation is to only grant the necessary privilege for a process, user, or program in order to reduce the damage in case of a serious security breach. For example, when we manage the filesystem's security by applying read, write, and execute rights, we usually grant much more privileges to a file than necessary, and this is a big problem. It would be better if we could grant only a few privileges (such as simple and individual rights) that were enough for a role, user, command, or even a process.

There are four sets of privileges for a process:

*   **Effective** (**E**): This represents a set of privileges that are currently in use.
*   **Inherited** (**I**): This is the set of privileges that can be inherited by a child process after a `fork()`/`exec()` call.
*   **Permitted** (**P**): This is the set of privileges that are available to be used.
*   **Limited** (**L**): This represents all the available privileges that can be made available to the permitted set.

Oracle Solaris 11 has several classes of privileges, such as file, sys, net, proc, and ipc. Each one of these privilege classes (some people call categories) have many different privileges, and some of them were chosen as being the basic privileges that are assigned to any user.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 and with at least 2 GB RAM.

## How to do it…

What are the existing privileges? This question is answered either by reviewing the main pages (the `main privileges` command) or by running the following command:

```
root@solaris11-1:~# ppriv -vl | more
contract_event
  Allows a process to request critical events without limitation.
  Allows a process to request reliable delivery of all events on
  any event queue.
contract_identity
  Allows a process to set the service FMRI value of a process
  contract template.
 (truncated output)

```

However, from all existing privileges, only some of them are basic and essential for process operations:

```
root@solaris11-1:~# ppriv -vl basic
file_link_any
  Allows a process to create hardlinks to files owned by a uid
  different from the process' effective uid.
file_read
  Allows a process to read objects in the filesystem.
file_write
  Allows a process to modify objects in the filesystem.
net_access
  Allows a process to open a TCP, UDP, SDP or SCTP network endpoint.
proc_exec
  Allows a process to call execve().
proc_fork
  Allows a process to call fork1()/forkall()/vfork()
proc_info
  Allows a process to examine the status of processes other
  than those it can send signals to.  Processes which cannot
  be examined cannot be seen in /proc and appear not to exist.
proc_session
  Allows a process to send signals or trace processes outside its
  session.
```

When handling process privileges, we can manage them by using the `ppriv` command. For example, to list privileges from the current shell, run the following commands:

```
root@solaris11-1:~# ppriv $$
2590:  bash
flags = <none>
  E: all
  I: basic
  P: all
  L: all
```

We could get the same result by executing `ppriv 2590`, and in both cases, a more comprehensive output could be obtained by using the `-v` option (`ppriv –v 2590` or `ppriv –v $$`). Additionally, there are two common flags that could appear here: `PRIV_AWARE` (the process is aware of the privileges framework) and `PRIV_DEBUG` (the process is in the privilege debugging mode).

We have learned about the possible privileges, so it is time to apply these concepts in real-world cases. For example, if a normal user (the `rbactest` user from the last section) tries to read the `/etc/shadow` content, they are not going to see anything, as shown in the following commands:

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# ls -l /etc/shadow
-r--------   1 root     sys          949 Apr 18 22:57 /etc/shadow
root@solaris11-1:~# su – rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ more /etc/shadow
/etc/shadow: Permission denied
```

It could present a serious problem for us if we didn't have a suitable solution, because we don't want to grant any unnecessary rights to the `rbactest` user, but we need to grant enough rights to accomplish this task of reading the `/etc/shadow` file. If we grant the read rights (R) to the other right group in the `/etc/shadow` file, we are allowing other users to read the file. A better situation arises by using the **Access Control List** (**ACL**) because we can grant read rights (R) on `/etc/shadow` for only the `rbactest` user, but it would be an excessive and dangerous right for a valuable file like this.

The real solution for this problem is to use least privileges. In other words, it is recommended that you assign only necessary privileges for the `rbactest` user to be able to see the `/etc/shadow` content. However, which is the right privilege? It is found by running the `ppriv` command with the `–De` option (debugging and executing), as shown in the following command:

```
rbactest@solaris11-1:~$ ppriv -De more /etc/shadow
more[2615]: missing privilege "file_dac_read" (euid = 102, syscall = 69) for "/etc/shadow" needed at zfs_zaccess+0x245
/etc/shadow: Permission denied
```

The privilege missing is `file_dac_read` and it has the following description:

```
rbactest@solaris11-1:~$ ppriv -vl file_dac_read
file_dac_read
  Allows a process to read a file or directory whose permission
  bits or ACL do not allow the process read permission.
```

The system call that fails is shown in the following command:

```
root@solaris11-1:~# grep 69 /etc/name_to_sysnum
openat64    69
```

It's feasible to get more information about the `mkdirat` system call by executing the following command:

```
rbactest@solaris11-1:~$ man openat
System Calls                                              open(2)
NAME
     open, openat - open a file

SYNOPSIS
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <fcntl.h>

     int open(const char *path, int oflag, /* mode_t mode */);

     int openat(int fildes, const char *path, int oflag,
          /* mode_t mode */);

DESCRIPTION
     The open() function establishes  the  connection  between  a
     file and a file descriptor. It creates an open file descrip-
     tion that refers to a file and a file descriptor that refers 
(truncated output)

```

Now we know the correct privilege, so there are two options to correct the situation: either the `file_dac_read` privilege is granted to the `rbactest` user directly, or it is assigned to a role (for example, `r_reboot` from the previous section).

To assign the `rbactest` user and then to assign the privilege for a role, execute the following commands:

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# usermod -R r_reboot rbactest
root@solaris11-1:~# rolemod -K defaultpriv=basic,file_dac_read r_reboot
root@solaris11-1:~# cat /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;defaultpriv=basic,file_dac_read;profiles=Reboot;roleauth=role
rbactest::::roles=r_reboot 
```

According to the previous step, we have associated the `rbactest` user with the `r_reboot` role (if you have already made it previously) and have kept the existing basic privileges. Furthermore, a new privilege (`file_dac_read`) was appended. To verify that the configuration is correct, run the following commands:

```
root@solaris11-1:~# su - rbactest
Oracle Corporation  SunOS 5.11  11.1  September 2012

rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation  SunOS 5.11  11.1  September 2012
r_reboot@solaris11-1:~$ profiles
          Reboot
          Basic Solaris User
          All
r_reboot@solaris11-1:~$ more /etc/shadow
root:$5$7X5pLA3o$ZTJJeO.MfVLlBGzJI.yzh3vqhvW.xUWBknCCMHRvP79:16179::::::18384
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
(truncated output)

```

It has worked! Another way to get the same result is to grant the `file_dac_read` privilege directly to the `rbactest` user, but this is not the recommend method:

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# usermod -K defaultpriv=basic,file_dac_read rbactest
root@solaris11-1:~# more /etc/user_attr
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;defaultpriv=basic,file_dac_read;profiles=Reboot;roleauth=role
rbactest::::defaultpriv=basic,file_dac_read;roles=r_reboot

root@solaris11-1:~# su – rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ more /etc/shadow
root:$5$oXapLA3o$UTJJeO.MfVlTBGzJI.yzhHvqhvW.xUWBknCCKHRvP79:16179::::::18384
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
(truncated output)

```

This has worked too!

### An overview of the recipe

In this section, we learned how to use the `pfexec` command, RBAC concepts, and least privileges concepts. Moreover, we have seen examples that explain how to apply these techniques in daily administration.

# References

*   *RBAC Access Control* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/rbac-1.html](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbac-1.html)
*   *Privileges* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/prbac-2.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1456/prbac-2.html#scrolltoc)
*   *Viewing and Using RBAC Defaults* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-new-1.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-new-1.html#scrolltoc)
*   *Customizing RBAC for Your Site* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-30.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-30.html#scrolltoc)
*   *Managing RBAC* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-4.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-4.html#scrolltoc)
*   *Using Privileges* at [http://docs.oracle.com/cd/E23824_01/html/821-1456/privtask-1.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1456/privtask-1.html#scrolltoc)