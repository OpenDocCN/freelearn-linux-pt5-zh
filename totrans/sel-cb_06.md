# Chapter 6. Setting Up Separate Roles

In this chapter, we will cover the following topics:

*   Managing SELinux users
*   Mapping Linux users to SELinux users
*   Running commands in a specified role with sudo
*   Running commands in a specified role with runcon
*   Switching roles
*   Creating a new role
*   Initial role based on entry
*   Defining role transitions
*   Looking into access privileges

# Introduction

Roles provide a flexible, manageable approach to grant multiple users the proper rights. Instead of assigning privileges to individual users, roles are created to which privileges are granted. Users are then granted the role and inherit the privileges associated with this role.

In SELinux, roles are used to grant access to domains. An application domain that is used to manage certificates on a system is assigned to one or more roles, thus allowing users with that role to possibly transition into that application domain. If the user role does not have this privilege, then the necessary permissions to manage certificates through that application domain are not accessible for the user.

The following diagram shows the relation between Linux logins (regular Linux accounts), SELinux users, SELinux roles, and SELinux domains:

![Introduction](img/9669OS_06_01.jpg)

To assign roles to users, Linux accounts are first mapped to an SELinux user. An SELinux user defines which roles are accessible (as users can have multiple roles assigned) as well as which security clearance the user can have at most (although lower security clearances can be assigned to users individually as well).

On systems where SELinux is primarily meant to confine network-facing services and not the users, this chapter will have little value. All users on these systems are mapped to the `unconfined_u` SELinux user, which has a default user domain of `unconfined_t` and is meant to be almost unrestricted—hence, the name, unconfined. When this is applicable, most distributions call the SELinux policy store **targeted** to reflect that the confinement is targeting specific applications and not the entire system.

# Managing SELinux users

In order to grant a Linux login the right set of roles, we first need to create an SELinux user that has just those roles assigned. Existing SELinux users can be modified easily, and if an SELinux user was added previously, it can be removed from the system as well.

## How to do it…

Managing SELinux users is done as follows:

1.  Use `semanage user` to list the currently available SELinux users:

    ```
    ~# semanage user -l
     Labeling   MLS/       MLS/
    SELinux User    Prefix     MCS Level  MCS Range                      SELinux Roles

    git_shell_u     user       s0         s0                             git_shell_r
    guest_u         user       s0         s0                             guest_r
    root            user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
    staff_u         user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
    sysadm_u        user       s0         s0-s0:c0.c1023                 sysadm_r
    system_u        user       s0         s0-s0:c0.c1023                 system_r unconfined_r
    unconfined_u    user       s0         s0-s0:c0.c1023                 system_r unconfined_r
    user_u          user       s0         s0                             user_r
    xguest_u        user       s0         s0                             xguest_r

    ```

2.  If no SELinux user exists yet, with the right set of roles, create it with `semanage user`. For instance, to create a database administration SELinux user, run the following command:

    ```
    ~# semanage user -a -R "staff_r dbadm_r" dbadm_u

    ```

3.  Existing users can be modified as follows:

    ```
    ~# semanage user -m -R "staff_r dbadm_r" staff_u

    ```

4.  An SELinux user can also be removed from the system:

    ```
    ~# semanage user -d dbadm_u

    ```

## How it works…

When an SELinux user is created, SELinux will update its configuration files at `/etc/selinux/` to include support for this SELinux user. It is a general best practice to name SELinux users after their functional purpose, so a **database administrator** (**DBA**) is called `dbadm_u`, whereas a website administrator is called `webadm_u`.

The set of roles that are available to the administrator can be obtained using `seinfo`:

```
~# seinfo -r

```

Existing SELinux users can be modified. However, it is important that logged-in users are logged out (and perhaps temporarily locked) from the system during the change. Otherwise, the SELinux policy could suddenly mark their session as having an invalid context and interrupt those users in their operations.

When an SELinux user is removed from the system, it is also important that all the remaining files that have this SELinux user in their context are relabeled. Otherwise, these files (and other resources) are labeled with an invalid context, making the files and resources inaccessible to others.

Once an SELinux user is created, it is ready to be assigned to one or more Linux users.

## There's more...

With SELinux users, MLS settings can be provided as well. For instance, to set a specific security clearance, the following command is used:

```
~# semanage user -a -r s0-s0:c0.c110 dbadm_u

```

For an SELinux user, this is the upper limit of the security clearance that a users' context can be in. When we assign users to an SELinux user, it is possible to force a lower security clearance individually so that there is no need to create separate SELinux users for every difference in security clearance.

# Mapping Linux users to SELinux users

With the SELinux users available, we can now map Linux users to SELinux users. This will ensure that the users, when logged in to the system, are assigned a default context aligned with this SELinux user.

## How to do it…

In order to map Linux users to SELinux users, the following steps can be taken:

1.  List the existing mappings with `semanage login`:

    ```
    ~# semanage login -l
    Login Name           SELinux User              MLS/MCS Range

    __default__          user_u                    s0-s0:c0.c1023
    root                 root                      s0-s0:c0.c1023
    system_u             system_u                  s0-s0:c0.c1023
    %wheel               sysadm_u                  s0-s0:c0.c1023

    ```

2.  For an individual user account, map the account to an SELinux user with `semanage login`:

    ```
    ~# semanage login -a -s dbadm_u user1

    ```

3.  It is also possible to assign a group of users to an SELinux user through their primary Linux group. For instance, if a `dba` group exists, it can be assigned to an SELinux user as follows:

    ```
    ~# semanage login -a -s dbadm_u %dba

    ```

4.  Mappings can be modified easily:

    ```
    ~# semanage login -m -s webadm_u user1

    ```

5.  If a mapping is no longer needed, it can be removed as well:

    ```
    ~# semanage login -d user1

    ```

## How it works…

The `semanage login` application manages the `seusers` file in `/etc/selinux/`. This file is used by SELinux's `pam_selinux.so` authentication library that is called when a user logs in to a system. Upon invocation, SELinux will check the `seusers` file to see which SELinux user a Linux account is mapped to. It will then perform an SELinux context switch so that the rest of the login process (including the shell or graphical environment that is launched) will have the right SELinux context assigned to it.

Creating login mappings does not influence the existing sessions, so if a user is already logged in, it is wise to have the user log out first. Also, any files created by the user in the past might have a wrong SELinux user associated with them. Any login that isn't specifically mentioned will be assigned a default SELinux user. If the SELinux user changes, then the files owned by this Linux login will suddenly have a wrong SELinux user set. If the user-based access control feature in SELinux is enabled, then these files will not be accessible anymore by the user. In this case, the administrator will need to relabel the files forcefully (which includes resetting the SELinux user):

```
~# restorecon -RF /home/user1

```

In case of both user mappings and group-based mappings, the first mapping that is mentioned in the `seusers` file that matches a particular login is used.

When a user logs in and no mapping matches the login itself (either through a direct match against a Linux account name or through a group membership), then SELinux will look at the login mapping for the `__default__` user. This is a special rule that acts as a fallback rule. On systems with unconfined users, the `__default__` user is usually mapped to the `unconfined_u` SELinux user. On systems without unconfined users, `__default__` usually maps to the (unprivileged) `user_u` SELinux user.

# Running commands in a specified role with sudo

When a user has been assigned multiple roles, they usually work with their primary role (such as `staff_r`) and only selectively execute commands with the other role. This can be accomplished through the `sudo` command, as these commands usually also require a different Linux user (which can be `root` or the `postgresql` account for DBA tasks on the PostgreSQL database server).

## How to do it…

In order to configure `sudo` to perform the right role and type transition, execute the following steps:

1.  Open up the `sudoers` file through `visudo`:

    ```
    ~# visudo

    ```

2.  Define the commands that the user(s) are allowed to execute. For instance, to allow all users in the `dba` group to call `initdb` in the `dbadm_r` role, define the commands as follows:

    ```
    %dba ALL=(postgres) ROLE="dbadm_r" TYPE="dbadm_t" /usr/sbin/initdb
    ```

3.  The users in the `dba` group can now call `initdb`, and `sudo` will automatically switch to the `dbadm_r` role and the `dbadm_t` user domain when `initdb` is called:

    ```
    ~$ sudo -u postgres initdb

    ```

## How it works…

The regular user domains that users run with are, by default, not that privileged. Although it is possible to extend the privileges of the role and user domains directly, the best segregation is provided through different roles. Such an approach allows unprivileged user domains, such as `staff_t`, to be used by multiple, different organizational roles (and thus, SELinux users).

Once a privileged command needs to be executed, users will need to switch their active role. If this is only needed for a small set of commands, which also require switching the Linux user itself (such as switching to the `postgres` runtime account), then privilege delegation tools such as `sudo` are often used.

The `sudo` command is an SELinux-aware application that can be configured to assist in switching the SELinux context as well. This can be done through the command line directly if the user wants:

```
~$ sudo -u postgres -r dbadm_r -t dbadm_t initdb

```

However, most administrators will want to configure this in the `sudoers` file. This is more user friendly as the end user does not need to continuously pass the role and type parts of the context in which commands need to be executed.

Of course, this requires that the SELinux user that is calling `sudo` has the privilege to run commands in the `dbadm_r` role. If not, then even if the `sudoers` file mentions that the user can execute the command, the transition (and thus, the command) will fail, as shown in the following command:

```
~$ sudo -u postgres initdb
sudo: webadm_u:dbadm_r:dbadm_t:s0-s0:c0.c1023 is not a valid context

```

## See also

For more information on `sudo` and the `sudoers` file, check out their associated manual pages:

```
~$ man sudo
~$ man sudoers

```

The main project site for the `sudo` application is at [https://www.sudo.ws](https://www.sudo.ws).

# Running commands in a specified role with runcon

Using `sudo` is not mandatory. SELinux also provides a command called `runcon` that allows users to run a command in a different context. Of course, SELinux restrictions still apply—the user must have the proper privileges to execute commands with a different context.

## How to do it…

Running a command using a specified role and type is done by completing the following steps:

1.  Identify the domain in which the command should run, usually by checking the executables' context and searching for the `entrypoint` definition:

    ```
    ~$ ls -Z auditctl
    system_u:object_r:auditctl_exec_t    auditctl
    ~$ sesearch -t auditctl_exec_t -c file -p entrypoint -A
    Found 1 semantic av rules:
     allow auditctl_t auditctl_exec_t : file { … entrypoint … };

    ```

2.  Call the command, passing along the role and target type:

    ```
    ~$ runcon -r secadm_r -t auditctl_t auditctl -l

    ```

## How it works…

The `runcon` application tells SELinux that the invocation of the command should result in a type and role transition towards the specified type (`auditctl_t`) and role (`secadm_r`). SELinux will perform multiple checks and validations before this will actually succeed. These checks are as follows:

*   Does the current user have the right to execute `auditctl` (execute rights on `auditctl_exec_t`)?
*   Is a role switch from the current role (say `staff_r`) to the new role (`secadm_r`) allowed?
*   Is there a policy in place that allows transition from the current type (say `staff_t`) to the selected type (`auditctl_t`)?
*   Is `auditctl_t` a valid target domain if the executed file is `auditctl_exec_t` (which is the `entrypoint` check)?
*   Is the target context (such as `staff_u:secadm_r:auditctl_t`) a valid context (which implies that the current SELinux user has access to the given role)?

The `runcon` application can be used when no Linux user transition needs to occur (although this doesn't exclude the use of `sudo`). In the example of `auditctl`, this means that the regular access controls on Linux still apply—if the current user does not have the rights to access the files used by `auditctl`, then using `runcon` will not suffice.

# Switching roles

When a role transition is needed for more than just a couple of commands, it is necessary to open a shell with the new role. This will ensure that the entire session is now running with the new role assigned to it. Every activity performed from within this session will then run with the target role.

## How to do it…

Switching roles with `sudo` or `newrole` is done as follows:

1.  Switching a role can be done using `sudo -i` or `sudo -s` if allowed by the `sudoers` file. If the `ROLE` and `TYPE` attributes are set, then the target shell will have the proper context assigned:

    ```
    ~$ id -Z
    dbadm_u:staff_r:staff_t:s0
    ~$ sudo -u postgres -i
    Password: 
    ~$ id -Z
    dbadm_u:dbadm_r:dbadm_t:s0

    ```

2.  Switching roles can also be done using `newrole`:

    ```
    ~$ newrole -r dbadm_r

    ```

## How it works…

Getting a shell after switching roles is not all that different from executing commands. However, the SELinux policy might not allow running shells and regular binaries in the target domain. For instance, a user who is allowed the `puppetca_t` domain through some role will not be able to run a shell in this domain, as `puppetca_t` is not allowed to be used through a shell—it is a domain for a particular set of commands.

Most user roles have a default user domain associated with them. The default user domain for a `dbadm_r` role is `dbadm_t`; the default domain for a `webadm_r` role is `webadm_t`. These user domains do have the privileges to be used through a shell.

The `newrole` command only requires the target role, as it will check the default type of a role (which is documented in the `default_type` file inside `/etc/selinux/mcs/contexts/`) and use this as the target type.

# Creating a new role

Roles are part of SELinux policies. In order to create a new role, it isn't possible to just invoke a few `semanage` commands. Instead, an SELinux policy module will need to be created.

## How to do it…

The SELinux policy needs to be updated in order to create a new role. The following steps can be used to do just that:

1.  Create a new policy module named after the role to be created, such as `pgsqladm` (for a PostgreSQL administration role).
2.  In the policy module, call the `userdom_login_user_template` interface:

    ```
    userdom_login_user_template(pgsqladm)
    ```

3.  Assign the proper privileges to the `pgsqladm_r` role and `pgsqladm_t` type:

    ```
    postgresql_admin(pgsqladm_t, pgsqladm_r)
    ```

4.  Edit the `default_type` file in `/etc/selinux/mcs/contexts/` to make `pgsqladm_t` the default type for the `pgsqladm_r` role:

    ```
    pgsqladm_r:pgsqladm_t
    ```

5.  Edit the `default_contexts` file in `/etc/selinux/mcs/contexts/` to inform the system to which types a transition has to be made when a user switch is triggered by an application. For instance, for a local login session, the following code can be used for this purpose:

    ```
    system_r:local_login_t  user_r:user_t … pgsqladm_r:pgsqladm_t …
    ```

6.  Now, build and load the policy, and verify that the new role is available:

    ```
    ~# seinfo -r | grep pgsqladm_r

    ```

## How it works…

Creating new roles for an SELinux system requires changes on multiple levels. Updating the SELinux policy is just one of these.

### Defining a role in the policy

The first step is to create a new role and user domain through the SELinux policy. There are a couple of templates available in the reference policy to easily build new roles. The relation between these templates is visualized in the following diagram:

![Defining a role in the policy](img/9669OS_06_02.jpg)

The various blocks in the diagram represent the following templates:

*   In `userdom_base_user_template`, the basic rules and privileges for roles and user domains are documented, regardless of their future use. If a role needs to be declared with an absolute minimum of privileges, the use of this template is preferred.
*   Inside `userdom_login_user_template`, `userdom_base_user_template` is called and extended with privileges related to interactive logins. When a role is created that is meant to be logged on directly (without the need to call `newrole` or `sudo`), then this interface is needed.
*   Within `userdom_restricted_user_template`, the `userdom_login_user_template` interface is called, but the user domain is also associated with the `unpriv_userdomain` attribute, meant for end user domains that have little security impact on the system.
*   The `userdom_common_user_template` interface adds privileges and rules that are common for both unprivileged and privileged roles.
*   The `userdom_unpriv_user_template` interface calls both `userdom_common_user_template` and `userdom_restricted_user_template` and is meant to declare unprivileged roles and user domains with interactive logon and general system access.
*   The `userdom_admin_user_template` interface calls both `userdom_common_user_template` and `userdom_login_user_template`, and creates a role and user domain that is meant to be used for administrative purposes.

Whenever such an appropriate interface is called, the proper role and type is created and can be used in the remainder of the policy module.

### Extending the role privileges

In the example, we assigned PostgreSQL administrative rights to the `pgsqladm_t` user domain and allowed the `pgsqladm_r` role the proper PostgreSQL domains (if any).

The reference policy tends to provide two types of interfaces that can be assigned to new roles:

*   Administrative roles, whose interface name usually ends with `_admin`
*   End user roles, whose interface name usually ends with `_role` or `_run`

Administrative roles allow for rights on all resources related to a particular domain. In case of the `postgresql_admin` interface, the role and user domain (which are passed on to the interface) are allowed to send signals to the PostgreSQL services, execute the `init` script (to launch or shut down the service), and manage the various resources of the domain (such as the database files, configuration files, and logs).

Services almost always have an `_admin` interface. These are called after the domain, such as `puppet_admin` for Puppet administration and `samba_admin` for Samba administration. Sometimes, an SELinux policy module has multiple administrative interfaces when there are different domains involved. An example would be the `logging_admin_audit` and `logging_admin_syslog` interfaces, as both auditing and system logging are provided by the same SELinux policy module, but the administration of these two services can be segregated.

End user roles allow the user to execute client applications or interact with services. Such interfaces, such as `puppet_run_puppetca` (which allows a user domain to run the `puppetca` application and transition to it) and `openvpn_run` (which allows users to run OpenVPN services), can still be somewhat administrative in nature, so make sure to validate the content of the interface. However, most of the time, this is governed through the application side and not infrastructure side—being able to launch VPN services does not mean that the user can manipulate routing tables as they see fit, even though the VPN service domain (`openvpn_t`) can.

It is important to review the interfaces before blindly granting them to new roles and users. In case of PostgreSQL, the `postgresql_role` role, for instance, does not allow the user to interact with the PostgreSQL service; instead, the interface is used to support SEPostgreSQL (SELinux-enabled PostgreSQL), which provides additional access controls in PostgreSQL based on SELinux policies. When users are assigned the `postgresql_role` role, they are granted basic privileges inside a PostgreSQL environment.

To allow users to interact with PostgreSQL, the `postgresql_stream_connect` and `postgresql_tcp_connect` interfaces can be used.

### Default types and default contexts

The `default_types` file informs SELinux what the default type is if no context is specified otherwise, and it is used by commands such as `newrole` to know what the default type is for a user.

The `default_contexts` file (which can be overridden through SELinux user-specific files in the `users/` subdirectory) informs the SELinux libraries and subsystem what specific SELinux type to transition to when a user and role switch has occurred from within a specified domain. For instance, a `cron` daemon runs in the `system_r:crond_t` context, but when it executes the user `cron` jobs, these jobs themselves need to run in a different SELinux role and SELinux type. The following `default_contexts` configuration snippet would have the jobs of a user (whose role is `pgsqladm_r`) run as `cronjob_t` (rather than `pgsqladm_t`):

```
system_r:crond_t  pgsqladm_r:cronjob_t
```

These files are generated as part of the base policy. Sadly, there are no `default_types.local` or `default_contexts.local` files that can be used to provide system-specific changes. As a result, updates on the base SELinux policy might overwrite these files depending on how the Linux distribution treats these files. If the files are seen as configuration files (such as with Gentoo Linux), then they are not altered by system updates; instead, the system administrator is informed that an update on these files might be needed, keeping the manual changes made by the administrator in the past.

# Initial role based on entry

Users will often have multiple roles associated with them. Depending on how they interact with the system, a different initial role (and a user domain) might be needed. Consider a user who interacts with a system locally (through the console), remotely through SSH (for administrative purposes), and through FTP (as an end user), as depicted in the following diagram:

![Initial role based on entry](img/9669OS_06_03.jpg)

We want to make sure that the default role in which the user session starts on the system depends on the entry point on the system. Direct console logon can be in the administrative role, `sysadm_r`, whereas remote logon is first in the `staff_r` role (to ensure a stolen SSH key cannot be used to perform administrative tasks on the system without knowing the users' system password). The use of the FTP server should result in an unprivileged role, `ftp_shell_r`.

### Note

The `ftp_shell_r` role is a nondefault role and will not be available by default. Using SELinux with an FTP server in this setup requires that the FTP server is either SELinux aware (and supports context transitions) or uses PAM for its authentication rather than internal user accounts.

## How to do it…

To configure the role to be used when a user logs on or starts a session, execute the following steps:

1.  First of all, make sure that the user is assigned the various roles:

    ```
    ~# semanage user -m -R "staff_r sysadm_r ftp_shell_r" staff_u

    ```

2.  Edit the `default_contexts` file by reordering the contexts, making sure that the right role is always mentioned before the others (or that the others are not mentioned at all):

    ```
    system_r:local_login_t:s0  user_r:user_t:s0  sysadm_r:sysadm_t:s0 staff_r:staff_t:s0
    system_r:sshd_t:s0  user_r:user_t:s0  staff_r:staff_t:s0
    system_r:ftpd_t:s0  ftp_shell_r:ftp_shell_t:s0
    ```

3.  Check whether the domains have support for specific Booleans that explicitly enable or disable transitioning into particular domains. For instance, consider the SSH daemon:

    ```
    ~# setsebool -P ssh_sysadm_login off

    ```

## How it works…

When applications call PAM to set up the user context, the PAM configuration will invoke methods provided by the `pam_selinux.so` file. These methods will check the `default_contexts` file to see what the context should be for a user. When `pam_selinux.so` is loaded through a daemon in the `system_r:sshd_t` context, for instance, then the lines for that particular daemon are interpreted:

```
system_r:sshd_t:s0  user_r:user_t:s0  staff_r:staff_t:s0
```

For the given user, the set of supported roles is obtained. In our case, this is `staff_r sysadm_r ftp_shell_r`. The entries in the `default_contexts` file are then looked at one by one, and the first role that is mentioned in the `default_contexts` file, that is also an allowed role for the user, will be used.

In the given example, as `user_r` is not an allowed role, `staff_r` is the next one on the list. This role is allowed, so when the user logs on through SSH, then its default role will be the `staff_r` role (and its associated user domain will be `staff_t`).

Some domains are also configured to allow or disallow direct logins into administrative roles. The SSH policy, for instance, uses an SELinux Boolean called `ssh_sysadm_login`, which allows transitioning into any user (`ssh_sysadm_login=on`) or only to unprivileged users (`ssh_sysadm_login=off`), specified policy-wise as follows:

```
tunable_policy(`ssh_sysadm_login',`
  userdom_spec_domtrans_all_users(sshd_t)
  userdom_signal_all_users(sshd_t)
',`
  userdom_spec_domtrans_unpriv_users(sshd_t)
  userdom_signal_all_users(sshd_t)
')
```

A similar approach can easily be built into custom policies. Note that the use of `userdom_spec_domtrans_unpriv_users` will only allow using the daemon for roles and types created through `userdom_unpriv_user_template`, as this interface assigns the `unpriv_userdomain` attribute that is used by the `userdom_spec_domtrans_unpriv_users` interface.

# Defining role transitions

It is possible to have SELinux automatically switch roles when a certain application is executed. The usual checks still apply (such as if the role is a valid one for the user, does the current user domain have execute rights, and many more), but then, there is no longer a need to call `runcon` or `sudo` to switch the role.

## How to do it…

Role transitions can be configured as follows:

1.  Identify the executable type on which a role transition has to occur:

    ```
    ~$ ls -Z puppetca
    system_u:object_r:puppetca_exec_t  puppetca

    ```

2.  In the SELinux policy, create an interface that includes the role transitions:

    ```
    interface(`puppet_roletrans_puppetca',`
      gen_require(`
        role puppetadm_r;
        type puppetca_t, puppetca_exec_t;
      ')
      allow $1 puppetadm_r;
      role_transition $1 puppetca_exec_t puppetadm_r;
      domtrans_pattern($2, puppetca_exec_t, puppetca_t)
    ')
    ```

3.  Assign the newly created interface to the user:

    ```
    puppet_roletrans_puppetca(staff_r, staff_t)
    ```

## How it works…

The first rule that is activated is a role-allow rule. Such a rule tells SELinux what role switch is allowed and in which direction. The set of allowed role switches can be queried using `sesearch`:

```
~# sesearch --role_allow

```

Consider the following role-allow rule(s) for the `puppetadm_r` role:

```
  allow staff_r puppetadm_r
```

In this case, *only* the `staff_r` role is allowed to switch to the `puppetadm_r` role. Switching from the `puppetadm_r` role back to the `staff_r` role is not allowed.

The second rule tells SELinux that if a `puppetca_exec_t` labeled file is executed by the selected role (`staff_r`, in our case), then the role should switch to `puppetadm_r`. Of course, this is only done when the SELinux user is allowed the target role.

The third rule will perform a domain transition from `staff_t` to `puppetca_t` if `staff_t` executes a `puppetca_exec_t` labeled file.

It should be noted though that a forced role transition (that is, through the SELinux policy) is not a preferred method in the majority of cases, as it doesn't provide any flexibility to the administrator. If this is implemented, then using multiple roles is more difficult as some domains are hardcoded to a particular role.

# Looking into access privileges

To finish off, let's look at how to verify access privileges granted to users. Specifying roles and privileges allows users to do their job, but from a security point of view, it is also important to verify if (and which) users can manipulate certain resources. Auditors will want to have an overview of who is able to, say, manipulate SELinux policies or read private keys.

## How to do it…

To properly investigate access rights, the following approach can help in identifying users (and processes) that have the permissions we want to be informed about:

1.  Verify file permissions that are not related to SELinux.
2.  Verify direct access to the resource (such as read rights on private keys).
3.  Look at who (users or applications) has the right to manipulate the SELinux policy.
4.  Check users and domains that are granted direct access to filesystems and raw devices.
5.  See when memory can be accessed directly.
6.  Review who can update authentication files.
7.  Analyze who can boot the system.

## How it works…

Reviewing access is a lengthy process. It isn't sufficient to just look into file ownership (user and group) and look at the permissions of the file to find out who is actually able to read or modify the file (assuming that the privilege looked into is file access). Privilege delegation tools such as `sudo` (through the `sudoers` file or the `sudo` configuration in an LDAP server) need to be checked as well, together with the `setuid` application access, backup file access (when read access is to be examined), and more.

With the mandatory access controls that SELinux provides, checking the policy for access rights is an important part of such an evaluation. The `sesearch` application can assist in this quest.

### Direct access inspection

To check direct access, we need to query both the access rights (such as write privileges on the resource) as well as relabeling rights. After all, a domain that is allowed to change the SELinux context of a file to another resource can theoretically switch the context, modify the file, and reset the context.

```
~# sesearch -t lvm_etc_t -c file -p write -ACST
Found 6 semantic av rules:
 allow sysadm_t non_auth_file_type : file { … };
 allow portage_t file_type : file { … };
...
~# sesearch -t lvm_etc_t -c file -p relabelfrom,relabelto -ACST
Found 5 semantic av rules:
 allow sysadm_t non_auth_file_type : file { … };
 allow restorecond_t non_auth_file_type : file { … };
 allow setfiles_t file_type : file { … };
…

```

This code shows not only the user domains that have the privileges, but also the application domains. In a review of permissions, it is necessary to also validate who can access and manipulate processes that run in these domains. This can be done by checking the transition permission:

```
~# sesearch -t setfiles_t -c process -p transition -ACST

```

For each of the domains, studying who can manipulate these processes is a time-consuming process and requires intimate knowledge of the application(s) that run in the given domain. For instance, the `restorecond` daemon will only reset file contexts to the context known by the SELinux tools (so, modifying the context temporarily is not possible through `restorecond`) and only on those locations that are configured in the `restorecond` configuration file.

### Policy manipulation

Checking the SELinux policy isn't sufficient as the policy can be manipulated as well. Loading a new policy is governed through, among various other privileges, the `load_policy` permission:

```
~# sesearch -t security_t -c security -p load_policy -ACS
Found 2 semantic av rules:
EF allow kernel_t security_t : security load_policy  ; [ secure_mode_policyload ]
EF allow load_policy_t security_t : security load_policy ; [ secure_mode_policyload ]

```

Similarly, the access towards the selected domains (and the `load_policy_t` domain in particular) needs to be verified.

As can be seen from the output, manipulating the SELinux policy can also be controlled through an SELinux Boolean called `secure_mode_policyload`. When this Boolean is enabled, loading a new policy is no longer possible. If this Boolean is enabled and persisted, then even rebooting a system will not help unless the system is booted in the permissive mode.

Similarly, checking who can put the system in the permissive mode can be verified as well:

```
~# sesearch -p setenforce -ACS

```

This is governed through the same SELinux Boolean though.

Another way to manipulate the SELinux policy would be to boot the system in the permissive mode or even with SELinux disabled. This means that reviewing access to the boot files is also important (the `boot_t` type).

### Indirect access

It is also possible to access resources indirectly, for instance, by manipulating the raw devices (such as disk devices or memory). Access to device files is already quite privileged on Linux systems. With SELinux, additional controls might be put in place.

Disk devices are usually labeled as `fixed_disk_device_t`. Access to these files should only be granted to application domains, although some privileged user domains might be able to relabel such device nodes or manipulate application domains to perform actions not granted to the regular user.

```
~# sesearch -t fixed_disk_device_t -ACS

```

Users who are able to manipulate files related to system authentication can grant themselves different user roles, for instance, by logging on to the system as a different user (who does have the rights needed). This includes access to `/etc/pam.d/` (usually labeled as `etc_t`) or the authentication libraries themselves in `/lib/security/` (usually labeled as `lib_t`).