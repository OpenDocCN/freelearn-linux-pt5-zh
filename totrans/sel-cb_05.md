# Chapter 5. Creating a Server Policy

In this chapter, we will cover the following recipes:

*   Understanding the service
*   Choosing resource types wisely
*   Differentiating policies based on use cases
*   Creating resource-access interfaces
*   Creating exec, run, and transition interfaces
*   Creating a stream-connect interface
*   Creating the administrative interface

# Introduction

Desktop application policies protect a user from vulnerabilities within the application or from unwanted behavior exerted by the application. On a server, however, the impact can be much larger. Server policies are used to protect the entire system from unwanted behavior, abusive access by users, or exploited vulnerabilities within the application.

Services also have a long lifetime. Unlike desktop applications, which usually start up and shut down together with the users' daily work cycle, services tend to run nonstop, 24/7\. This not only provides a larger time window to try and exploit these services, but also happens in the background with services that the administrator might not be actively watching.

# Understanding the service

The first aspect of designing server policies is to understand the service at hand. Each service has its own internal architecture, and understanding how the various processes and resources interact with each other is extremely important.

Only when the internal architecture is fully understood will we be able to create a properly functioning policy. Otherwise, we risk that the policy will be too broad (too many access rights) or too restricted. Unlike applications, which are usually easy to test from an end user point of view, services often have activities that are much harder to test (or even consider).

## How to do it…

Just like with desktop applications, understanding the application behavior is of key importance to create good SELinux policies. Research into and analysis of the behavior can be done by performing the following steps:

1.  Research the service at large by looking for online architecture drawings or architecture documentation.
2.  Try to explore the service in a sandbox environment.
3.  Follow some tutorials for the service with relation to both administration tasks as well as end user tasks.
4.  Structurally document how the service should be used.

## How it works…

Understanding a service means to get some degree of experience with the administration of the service. Trying to create a server policy for a specific database technology, but not knowing how this database technology works, will be almost impossible.

### Online research

Most services have well documented architectural information available online. By using an Internet search engine, we can easily come to the architecture information for a particular service.

While developing service policies, it is considered a best practice that the types and domains are named similar to the functional services that are used. For instance, in a Postfix architecture, functional services such as `pickup`, `cleanup`, `smtpd`, `qmgr`, and many more are basic services that a Postfix administrator has to deal with. In SELinux policies, we should try to have the domains labeled similarly (so the domain will be labeled `postfix_qmgr_t` for the `qmgr` service, `postfix_spool_maildrop_t` for the `maildrop` queue, and so on).

### Sandbox environment

Being able to play around with a service in a sandbox environment allows us to see the interactions at hand. It also allows us to follow online tutorials or administration guides to get to know the service.

There are many technologies available nowadays to play around with technologies. Virtualization allows users to run complete systems in an isolated environment and has led to the creation of virtual appliances.

Virtual appliances are virtual images that can be easily installed in a virtualized environment. However, a pure virtualization still requires users to install an operating system, install the service, and configure it before really starting to use it; virtual appliances provide preconfigured systems that host one or more services out of the box.

Next to virtualization, containers are also starting to play a large role. Unlike virtualization, software running inside containers is isolated from other software but is still part of the operating system itself.

### The structural documentation

After having a thorough read through the architecture of the application and perhaps even playing around with the software, we might need to document the architecture of the service further in order to deduce the right SELinux types and resources, as well as interfaces and roles related to the service.

In order not to forget anything important, the logical architecture of a service can be documented using the **FAMOUS** abbreviation:

*   **Feeds**: This tells us which external resources provide input to the service in a more-or-less batch-oriented approach as well as which external resources the service interacts with.
*   **Administration**: This informs us how the service is administered (command-line interfaces, user interfaces, or other applications).
*   **Monitoring**: This informs us about logfiles used or commands that are supported to verify the state of the service.
*   **Operations**: This documents the day-to-day runtime behavior of all the processes (and the flows, using the CRUD method—Create, Read, Update, Delete). This is usually the architecture information found earlier during the online research phase.
*   **Users and rights**: This documents how users are defined and managed in the service. This also documents which authentication or authorization backends are used, how different roles within the service behave, and so forth.
*   **Security-related features**: These tell us about security-related features such as application-based access controls, firewall requirements (which in our case are important for the policy network rules), and so forth.

With this information at hand, we can have a clear overview of how the service behaves. For instance, a high-level view of the PostgreSQL database service looks like the following diagram:

![The structural documentation](img/9669OS_05_02.jpg)

Such a drawing helps us to identify types later on, both for the processes as well as the resources involved. Any interactions with the service provided by third-party services is shown as well, as these interactions will result in privileges that need to be assigned to the other processes (that is, interfaces in the SELinux policy).

It is not easy to document how a service works without understanding the service at hand. Because of the complexity of the service, it is a good practice to get experts or developers of the service together and guide us in understanding the service. These developers and engineers can later be used to challenge the SELinux policy that is being written.

## See also

A nonexhaustive list of open source virtual appliance providers is as follows:

*   Artica ([http://www.artica.fr](http://www.artica.fr)) for proxy, mail, and NAS appliances.
*   Turnkey Linux ([http://www.turnkeylinux.org/](http://www.turnkeylinux.org/)) offers more than a hundred ready-to-use solutions.
*   Vagrant ([http://www.vagrantup.com/](http://www.vagrantup.com/)) is a management platform for virtual systems, and has a large community of Vagrant boxes that provide virtual appliance-like setups for many free software services.
*   Docker ([https://www.docker.io/](https://www.docker.io/)) is not a true virtualization setup, but rather a container-based approach. From the Docker Index ([https://index.docker.io/](https://index.docker.io/)), many containers can be freely downloaded.

Many commercial technologies also provide development virtual machines to deploy. Virtualization technology providers such as VMware® have solution-exchange communities, where virtual images for various technologies are freely available.

# Choosing resource types wisely

Services interact with resources, and the label that we assign to the resources is used by the fine-grained access controls assigned to these resources. End user files (for users that have a Linux account on the system) are labeled as `user_home_t`, which suffices for most uses. However, when we deal with services, the choice of the resource label defines if and how other applications can access those resources and is much more fine-grained than what we currently use for end user files.

There are some best practices concerning resource type selection within SELinux policies, which we will now look into.

## How to do it…

The service resource types need to be carefully chosen. Their naming implies the functional use of the resource, which already pushes the development of the policy in a certain structure. The types and their affiliated permissions can be developed by completing the following steps:

1.  Look for the processes that will run within their own specific domain and create the domain types. For each domain, look for the entry files of that domain and create an `_exec_t` type. Mark the type as either an init daemon type (when the service is launched through a service script) or a D-Bus daemon (when the service is launched through the D-Bus service). For instance, for the BIND service:

    ```
    type named_t;
    type named_exec_t;
    init_daemon_domain(named_t, named_exec_t)
    ```

2.  Look for all sets of logical resources that are used by the application. These are often files specific to the service architecture (such as database files for a database service), but shouldn't be limited to files only.
3.  Create specific types for these resources. For instance, for the Qemu virtual guest images:

    ```
    type qemu_image_t;
    files_type(qemu_image_t)
    ```

4.  Grant the domains the proper access to these resources. For instance, the `qemu` process (running as `qemu_t`) will need manage rights on the images:

    ```
    manage_files_pattern(qemu_t, qemu_image_t, qemu_image_t)
    ```

5.  Go through the infrastructural resources (PID files, logfiles, and configuration files) and label these accordingly. For instance, for the `named` variable, the runtime data will be named as follows:

    ```
    type named_var_run_t;
    files_pid_file(named_var_run_t)
    ```

6.  Grant the domains the proper access to these resources, and if possible, enable a proper file transition:

    ```
    allow named_t named_var_run_t:file manage_files_perms;
    allow named_t named_var_run_t:sock_file manage_sock_file_perms;
    files_pid_filetrans(named_t, named_var_run_t, { file sock_file });
    ```

## How it works…

An application policy always provides a common set of privileges. It starts with proper domain definitions (showing how the policy will be structured) and is followed by the resource access patterns. Resources can be functional in nature (specific to the application that is being investigated for the policy) or more infrastructural (such as logfiles and configuration files).

### Domain definitions

Service domains are used to identify long-running processes that have a similar functional scope. An example could be the BIND named process (which is defined as `named_t`) or the Apache `httpd` processes (which are all running as `httpd_t`).

These service domains are usually launched from an `init` script, which results in the use of the `init_daemon_domain` interface. If a service is launched by D-Bus, then the interface to use is `dbus_system_domain`. Of course, multiple interfaces can be used: the PPP daemon, for instance, supports both `init` scripts and D-Bus.

If a service daemon is launched by another daemon instead, then it is sufficient to mark the process domain as a domain type and the executable type as the entry point:

```
type postfix_bounce_t;
type postfix_bounce_exec_t;
domain_type(postfix_bounce_t)
domain_entry_file(postfix_bounce_t, postfix_bounce_exec_t)
```

In this case, we need to provide the parent domain (in our case, `postfix_master_t`) the rights to execute (`postfix_bounce_exec_t`) and transition (to `postfix_bounce_t`):

```
domtrans_pattern(postfix_master_t, postfix_bounce_exec_t, postfix_bounce_t)
```

### Logical resources

The logical resources are the files that are specific to the applications' functional design. For instance, a virtualization layer such as Qemu will have a logical resource for the image files (`qemu_image_t`). The logical resources for a web server have already been discussed in an earlier chapter (such as `httpd_sys_content_t` for standard system read-only web content).

Such resources are declared as regular file resources and the proper permissions are granted to the various domains. Further down the document, when privileges for the `qemu_t` domain are summed up, the `manage_files_pattern` line can be added to allow the `qemu_t` domain to manage the images.

By making separate labels for each of the logical resources, we can create interfaces for other processes that might need to interact with these resources without having to grant those applications too many privileges.

Think of a backup application, such as Amanda. The actual backup data itself (`amanda_data_t`) should only be accessible by the Amanda application. Other service administrators on the same system should not have access to these files—backups can contain sensitive information, so only the backup tool itself should have access to this data. Even the backup administrators, who need to manage the backup infrastructure, might not need direct access to this data.

### Infrastructural resources

Infrastructural resources are file types that are often set for applications.

Logfiles are marked through the `logging_log_file` interface and usually end with the `_log_t` suffix, such as `amanda_log_t`. By marking it as a logfile, domains that are assigned an operation concerning all logfiles (such as `logging_read_all_logs`) automatically have these privileges on the newly defined type. Often, a file transition is set so that files created in `/var/log/` automatically get the right type. This is done through the `logging_log_filetrans` interface:

```
type amanda_log_t;
logging_log_file(amanda_log_t)
# Directories created by amanda_t domain in /var/log (var_log_t) get the amanda_log_t type:
logging_log_filetrans(amanda_t, amanda_log_t, dir)
```

Configuration files are marked as regular files (through `files_type`) and end with either `_conf_t` or `_etc_t`. Some policy developers like to use `_conf_t` for real configuration files and `_etc_t` for other miscellaneous files in the `/etc/` directory structure that are not direct configuration files. In most cases though, this is only for semantic reasons as all related domains need the same set of privileges on both types.

Temporary files are marked through the `files_tmp_file` interface and end with the `_tmp_t` suffix. A file transition is almost always put in place to ensure that the temporary files are properly labeled:

```
type amanda_tmp_t;
files_tmp_file(amanda_tmp_t)
# All files, directories and symbolic links created by amanda_t in a tmp_t location should get the amanda_tmp_t label:
files_tmp_filetrans(amanda_t, amanda_tmp_t, { dir lnk_file file })
```

PID files and other generic run files are usually labeled ending with `_var_run_t` and are marked as a PID file through the `files_pid_file` interface. As with logfiles, a file transition is usually put in place as well:

```
type amanda_var_run_t;
files_pid_file(amanda_var_run_t)
# Files and sockets created in /var/run should become amanda_var_run_t:
files_pid_filetrans(amanda_t, amanda_var_run_t, { file sock_file })
```

Other variable data that is not given a logical resource name is often labeled ending with `_var_lib_t`. Such files are marked as regular files (using `file_type`) and a file transition can be defined using `files_var_lib_filetrans`.

# Differentiating policies based on use cases

As services mature, they often gain more features, which might not always be necessary. For instance, daemons that are able to optionally connect to various network resources depending on their configuration should not be allowed by the SELinux policy to always connect to various network resources.

To govern these features, SELinux policy developers include Booleans to selectively toggle policies based on the administrator's requirements.

## How to do it…

Booleans allow policy developers to create policy rules that only participate in access control when the administrator has elected to use them. For services in particular, this is often used to optionally allow privileges based on the use case of the service and is implemented as follows:

1.  Identify the policy blocks that should be marked as optional, depending on the configuration. For instance, this could be a set of policy rules that allow PostgreSQL to connect to other PostgreSQL databases:

    ```
    corenet_tcp_connect_postgresql_port(postgresql_t)
    corenet_sendrecv_postgresql_client_packets(postgresql_t)
    ```

2.  For each block, create a well-chosen SELinux Boolean that administrators can easily identify as the right Boolean to toggle for their specific use case. For instance, we can create a `postgresql_connect_db` Boolean:

    ```
    ## <desc>
    ##   <p>
    ##     Determine if the PostgreSQL daemons can connect to other databases.
    ##   </p>
    ## </desc>
    gen_tunable(postgresql_connect_db, false)
    ```

3.  Surround the policy blocks that need to be toggled with a `tunable_policy` statement for the chosen SELinux Boolean, as follows:

    ```
    tunable_policy(`postgresql_connect_db',`
      corenet_tcp_connect_postgresql_port(postgresql_t)
      corenet_sendrecv_postgresql_client_packets(postgresql_t)
    ')
    ```

## How it works…

Although we shouldn't over-tune policies by generating dozens of Booleans, isolating functionality that is often abused in exploits is a good practice.

Consider a database engine. Databases can have features that allow them to connect to other databases (for instance, to set up database links or support some kind of cluster), but in many situations, these features are not needed. If a database is compromised (through SQL injection, for instance), it is better to make sure that this database cannot access other databases (so the compromised database is sufficiently contained).

The configuration that toggles this behavior in a PostgreSQL setup could be named `postgresql_connect_db` (for database-specific connections) or `postgresql_connect_all_ports` (for any target connection) and developed as shown in the previous example (the example includes the in-line comment documentation that would be used if the policy is meant to become part of the distribution policy or reference policy project).

Accessing other resources on the network is a common feature that, if it is not part of the standard behavior of the application, should be considered for making optional.

There are many other use cases that should be considered. Here is a nonexhaustive list:

*   An application that can optionally execute system scripts or user-provided scripts should be governed through an `_exec_scripts` or `_exec_user_scripts` Boolean.
*   Allowed domain transitions to higher-privileged domains or increased privileges due to some functionality is usually governed through `_use_*` Booleans. For instance, a domain optionally supporting Java can have a `_use_java` Boolean.
*   Access to specific filesystems or devices is also governed through `_use_*` Booleans, such as `_use_cifs` (for SMB-CIFS filesystems) or `_use_nfs`.
*   Functional support (such as Nginx support for various protocols) can be made optional through `_enable_*` Booleans, such as `nginx_enable_imap_server` or `nginx_enable_pop3_server`.

# Creating resource-access interfaces

With all the resources defined, we now need to ensure that other domains can use those resources as needed. As we've seen, resources can be functional in nature (specific to a service) or more infrastructural (such as logfiles).

Access to resources is provided through SELinux policy rules that need to be provided through access interfaces. These interfaces are then used by third-party SELinux policy modules to document and allow access to the resource types. Without the access interfaces, the resource types we define are not easily accessible by other policy developers.

## How to do it…

To create resource-access interfaces, add the proper interface definition in the module's `.if` file. For instance, to create a set of resource interfaces to access ClamAV's configuration files, follow the next set of steps:

1.  For each resource, create an overview of the privileges that will be needed. For file class resources, these are often search, read, write, and manage privileges. In case of logfiles, some applications only need append privileges (which ensures that they cannot modify existing data, only add data to it).
2.  Create the interface in the module's `.if` file and ensure that it is properly documented, as shown in the following code:

    ```
    ##########################################
    ## <summary>
    ##   Read clamav configuration files
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain allowed access
    ##   </summary>
    ## </param>
    #
    interface(`clamav_read_config','
      gen_require(`
        type clamd_etc_t;
      ')
      files_search_etc($1)
      allow $1 clamd_etc_t:file read_file_perms;
    ')
    ```

3.  Consider creating a `dontaudit` interface as well to assign to SELinux domains that might attempt to perform this action while not needing the privilege:

    ```
    ########################################
    ## <summary>
    ##   Do not audit attempts to read the clamd configuration files
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain not to audit
    ##   </summary>
    ## </param>
    #
    interface(`clamav_dontaudit_read_config',`
      gen_require(`
        type clamd_etc_t;
      ')
      dontaudit $1 clamd_etc_t:file read;
    ')
    ```

## How it works…

The resource-access interfaces are needed to allow interaction with the SELinux types managed through the SELinux module. The build environment does not have a default set of privilege interfaces that are generated out of the box, so we need to create these interfaces ourselves.

One might be tempted to only create the resource interfaces that are known to be used in the immediate future. However, it is recommended to create the proper interfaces for all resources and each individually with a coherent set of supported privileges. This is because we never know how the resources will be used by others, and by not creating the proper resources, we are forcing other developers to create their own `my*` modules to provide interfaces.

By covering most access patterns towards the resources, we provide a nice set of interfaces that other developers can use while keeping the interfaces all bound to a single module.

Even the `dontaudit` related interfaces will play an important role for the users of the SELinux policy. When policy developers commit policy improvements to repositories, they usually do not `dontaudit` unless they are 100 percent convinced that these will hide cosmetic denials and thus can be ignored. As a result, default SELinux system deployments will have quite a few denials in the audit logs that need to be looked into by the system administrator.

If the administrator doesn't believe that the denials need to be enabled, then they will need to be able to `dontaudit` them. Although the administrator can create the proper interfaces themselves, it is much easier if the `dontaudit` interface definitions are already provided.

# Creating exec, run, and transition interfaces

Service domains usually have a few binaries that are executed by user domains or through other service or application domains. Each case of these executions need to be properly investigated to see if a domain transition is needed (that is, a specific domain needs to be created for that execution environment) or if the command can run within the privileges of the caller domain.

From an interface point of view, this is provided through the `_exec`, `_run`, and `_domtrans` interfaces.

## How to do it…

Execution-related interfaces allow for other policy modules to define the interaction with this application. This interaction can be a regular execution, but can also contain a domain transition to switch the application domain to the newly defined one. The set of execution interfaces are created as follows:

1.  For each execution where the application itself needs to run in the caller domain (so no transition has to occur), create an `_exec` interface as follows:

    ```
    #######################################
    ## <summary>
    ##   Execute wm in the caller domain
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain allowed access
    ##   </summary>
    ## </param>
    #
    interface(`wm_exec',`
      gen_require(`
        type wm_exec_t;
      ')
      corecmd_search_bin($1)
      can_exec($1, wm_exec_t)
    ')
    ```

2.  For each execution by a domain that is in the same role as the service (usually, `system_r`) and where a transition has to occur, create a `_domtrans` interface as follows:

    ```
    ##########################################
    ## <summary>
    ##   Execute vlock in the vlock domain
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain allowed to transition
    ##   </summary>
    ## </param>
    #
    interface(`vlock_domtrans',`
      gen_require(`
        type vlock_t, vlock_exec_t;
      ')
      corecmd_search_bin($1)
      domtrans_pattern($1, vlock_exec_t, vlock_t)
    ')
    ```

3.  For each execution by a domain that might not have standard access to the application domain, and where a domain transition has to occur, create a `_run` interface as follows:

    ```
    #########################################
    ## <summary>
    ##   Execute vlock in the vlock domain and allow the specific role the vlock domain
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain allowed to transition
    ##   </summary>
    ## </param>
    ## <param name="role">
    ##   <summary>
    ##   Role allowed to access the vlock domain
    ##   </summary>
    ## </param>
    #
    interface(`vlock_run',`
      gen_require(`
        attribute_role vlock_roles;
      ')
      vlock_domtrans($1)
      roleattribute $2 vlock_roles;
    ')
    ```

## How it works…

The use of `_exec`, `_run`, and `_domtrans` are standard interface patterns in policy development. The `_role` interface that we created during desktop application policy development not only includes domain transition and role support, but also resource accesses related to the user domain interacting with the desktop application domain.

In the `_run` interface, the only set of privileges that is provided is to transition to the right domain and assign the domain to the right role (as part of SELinux role-based access control). It is common practice that the order of the parameters of a `_run` interface are the domain first and then the role—unlike the `_role` interfaces, where the role comes first and then the domain.

In a `_domtrans` interface, only the domain transition is enabled. Usually, the `_run` interfaces call the `_domtrans` interface so that both interfaces are defined and the right one for the job is called by the caller SELinux policy module. But unlike the `_run` interfaces, the `_domtrans` interfaces do not extend roles and are usually called by other modules for service domain interaction.

For instance, the `procmail_t` domain (for the procmail daemon) might call the `clamscan` application (part of the ClamAV setup) needing to transition to `clamscan_t`. It does so through the `clamav_domtrans_clamscan` interface:

```
optional_policy(`
  clamav_domtrans_clamscan(procmail_t)
')
```

Finally, the `_exec` interface allows a domain to execute a binary without any transition. This interface is needed when a binary is labeled as a specific executable type (not `bin_t` or `shell_exec_t`) as most domains then do not have the privilege to access this file at all, let alone execute it. For instance, the Postfix `local` daemon might call the `clamscan` executable but does not need to transition, resulting in the following call:

```
optional_policy(`
  clamav_exec_clamscan(procmail_local_t)
')
```

## See also

*   Assigning the newly created interfaces to roles is covered in [Chapter 6](ch06.html "Chapter 6. Setting Up Separate Roles"), *Setting Up Separate Roles*

# Creating a stream-connect interface

Be it through the specific executable types or by the generic `bin_t` labeled commands, executions that remain in the caller domain might still require additional privileges to be assigned to the caller domain. These additional privileges could be reading of configuration files or interacting with the main domain through Unix domain sockets or TCP/UDP sockets.

In this recipe, we'll set up a stream-connect interface (as the other privilege enhancements are already covered through the regular resource-access interfaces or network-access interfaces).

## How to do it…

Interaction with an application socket can be done either through a socket file or through a named Unix domain socket. This is application-specific, so consulting the application documentation might be necessary up front.

### For a Unix domain socket with a socket file

If the stream connection is through a Unix domain socket with a socket file, the interaction with an application socket can be done by completing the following steps:

1.  Identify and register the proper types in the `.te` file. Socket files usually have the `_var_run_t` suffix as they reside in `/var/run/`.
2.  Create a stream-connect interface that calls `stream_connect_pattern` as follows:

    ```
    interface(`ldap_stream_connect',`
      gen_require(`
        type slapd_t, slapd_var_run_t;
      ')
      files_search_pids($1)
      stream_connect_pattern($1, slapd_var_run_t, slapd_var_run_t, slapd_t)
    ')
    ```

### For an abstract Unix domain socket

If the stream connection is through an abstract Unix domain socket (so no socket files are involved), create a stream-connect interface that only provides the `connectto` privilege, as follows:

```
interface(`init_stream_connect',`
  gen_require(`
    type init_t;
  ')
  allow $1 init_t:unix_stream_connect connectto;
')
```

## How it works…

Daemons often provide methods to interact with them. Many services support Unix domain socket-based communication between a client application (which usually runs within the privileges of the caller domain) and the daemon itself.

In such cases, the daemon itself creates a socket file (usually in `/var/run/`) as some sort of access point (applications can also use abstract namespaces, where no socket file is needed anymore) and the caller domain is allowed to write to this socket and through it connect to the Unix domain socket held by the daemon. The set of privileges is provided by the `stream_connect_pattern` definition and can be visually represented as follows:

![How it works…](img/9669OS_05_01.jpg)

The most important privilege here is the `connectto` privilege between the caller domain and the daemon domain. In case of abstract Unix domain sockets, no socket file is used at all and only the `connectto` privilege is needed.

These privileges are then written in the following domain-specific interface that calls the `stream_connect_pattern` definition, which provides the proper privileges in one go:

```
~$ seshowdef stream_connect_pattern
define(`stream_connect_pattern',`
  allow $1 $2:dir search_dir_perms;
  allow $1 $3:sock_file write_sock_file_perms;
  allow $1 $4:unix_stream_socket connectto;
')
```

If stream-connection-oriented applications are used whose binaries are not labeled as `bin_t`, then a `_stream_connect` interface call is usually seen together with an `_exec` interface call.

# Creating the administrative interface

To end the SELinux module development for services, we need to create proper role-based interfaces. Whereas the `_role` interface is usually for nonprivileged user roles, an `_admin` interface is used to provide all the necessary privileges to fully administer a service.

## How to do it…

An administrative interface which we can later assign to the user and role that will administer the environment is created with the following steps:

1.  Create a specific `init` script type for the `init` scripts of the daemon. For instance, for the `virtd` daemon inside `virt.te`, the following policy rules create the proper `init` script type:

    ```
    type virtd_initrc_exec_t;
    init_script_file(virtd_initrc_exec_t)
    ```

2.  Make sure that this `init` script is labeled correctly through the `.fc` file:

    ```
    /etc/rc\.d/init\.d/libvirtd  --  gen_context(system_u:object_r:virtd_initrc_exec_t,s0)
    ```

3.  Start with a skeleton `_admin` interface:

    ```
    ##########################################
    ## <summary>
    ##   All rules related to administer a virt environment
    ## </summary>
    ## <param name="domain">
    ##   <summary>
    ##   Domain allowed access
    ##   </summary>
    ## </param>
    ## <param name="role">
    ##   <summary>
    ##   Role allowed access
    ##   </summary>
    ## </param>
    #
    interface(`virt_admin',`
      gen_require(`
        …
    ')
    ```

4.  Identify all the resources that an administrator would need access to. Keep in mind that administrators might need to directly modify files that are otherwise managed through the service-related commands—do not take away this right from administrators. A common pattern to use here is `admin_pattern`. Add in the proper rights in the interface (and do not forget to update the `gen_require` block at the beginning). Consider the following example:

    ```
    files_search_tmp($1)
    admin_pattern($1, virt_tmp_t)
    ```

5.  Look through the administration guides for other operations that administrators might need with regards to processes. Perhaps there are certain signals that could be allowed to be sent to the daemons:

    ```
    # Allow the admin to run strace or other tracing tools against the daemons
    allow $1 virtd_t:process { ptrace signal_perms };
    # Allow admins to view all information related to the processes
    ps_process_pattern($1, virtd_t)
    ```

6.  Allow the administrator to run the `init` script(s):

    ```
    init_labeled_script_domtrans($1, virtd_initrc_exec_t)
    domain_system_change_exemption($1)
    role_transition $2 virtd_initrc_exec_t system_r;
    allow $2 system_r;
    ```

## How it works…

The `_admin` interface is meant to contain all the privileges needed for an (otherwise) unprivileged user to administer a service. In essence, this unprivileged user will become privileged for this particular service, gaining just those rights that the user needs in order to manage the service, but nothing more.

We start by defining a particular `init` script type for the service. By default, the `init` scripts are labeled `initrc_exec_t` and only the system administrator is allowed to execute them. As we do not want to give a specific service administrator the privileges to execute any `init` script, we create a specific script type (`_initrc_exec_t`) and then allow the user, through the `_admin` interface, to execute that particular script type.

The latter, however, is more than just creating execute rights (which is done through the `init_labeled_script_domtrans` call). Executing the script also means that the script itself has to run in the `system_r` role. If we do not enforce this, then the script would (attempt to) run in the role of the caller domain (such as `virtadm_r`) and fail, as the `initrc_t` domain (the type used for the `init` scripts) is not allowed for the `virtadm_r` role.

Transitioning a role upon executing a file is done through the `role_transition` directive. In our example, we configure that the user role (such as `virtadm_r`) transitions to the `system_r` role upon executing `virtd_initrc_exec_t`:

```
role_transition $2 virtd_initrc_exec_t system_r;
```

We need to allow the `system_r` role for the given user role as well, which is done through the `allow $2 system_r` call. But even that is not sufficient.

SELinux has a constraint in place that prevents transitions to `system_r`, as the `system_r` role is used for all system services and, as such, is a highly privileged role. The constraint is defined so that only specific domains can trigger a transition to `system_r`. With the `domain_system_change_exemption` call, we mark the user domain as one of these domains.

Besides the `init` script-related permissions, most `_admin` interfaces provide administrative rights to almost all resources provided by the module. To simplify policy development, the `admin_pattern` call is used. This pattern not only provides manage rights (read, write, execute, delete, and so on) on the resources, but also relabel rights, allowing the administrator to relabel files and directories as the resource types used in the module (or vice versa, relabel from those types to other types the administrator has relabel privileges to).

With these relabel rights, administrators can call `restorecon` against files to label them correctly (if properly defined in the SELinux policy) or use `chcon` to specifically set a label.

## See also

*   Creating new administrative roles is covered in [Chapter 6](ch06.html "Chapter 6. Setting Up Separate Roles"), *Setting Up Separate Roles*