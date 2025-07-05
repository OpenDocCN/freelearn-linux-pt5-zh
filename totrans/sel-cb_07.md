# Chapter 7. Choosing the Confinement Level

In this chapter, we will cover the following recipes:

*   Finding common resources
*   Defining common helper domains
*   Documenting common privileges
*   Granting privileges to all clients
*   Creating a generic application domain
*   Building application-specific domains using templates
*   Using fine-grained application domain definitions

# Introduction

During the development of additional policies, developers can opt to use a very fine-grained policy model, a domain-per-application model, or a coarse-grained, functionality-based policy model. The relationship between these confinement models is shown in the following diagram:

![Introduction](img/9669OS_07_01.jpg)

In very fine-grained policies, multiple domains are defined, so functionally different processes of the same application are all running in their own specialized SELinux domain. A coarse-grained policy, on the other hand, allows to have different applications with a similar functionality run with the same context. Application-level policies are somewhere in the middle: they focus on one domain (or a very small set of domains) for one application.

Most policies are developed using a **one domain per application** principle. Still, the choice of development patterns in policy development reflects the confinement level of an application, as shared, coarse-grained policies might allow for more interaction between applications and resources than intended, whereas, a fine-grained policy is much harder to develop and maintain.

When we look at a functional level, we usually focus on shared resources or resources that cannot be tied to a particular application. An example is the `mta` SELinux policy, which manages the main infrastructure-related shared content such as e-mail aliases (`etc_mail_t`), user mailboxes (`mail_home_rw_t`), e-mail spool files (`mail_spool_t`), and more.

# Finding common resources

During policy development, some of the resources used by the policy are or could be shared with other policies. If that is the case, a functionality-driven policy module is created in which those common resources are placed. This allows other policies to use these resources and assign the right permissions through the interfaces declared in the functionality-driven policy.

## How to do it…

Most of the work in this recipe is to figure out what resources are shared. This is done by completing the following steps:

1.  Look for common files and directories that might be shared with other applications and whose ownership is not specifically tied to an application, but is more functional in nature. For these resources, declare them in a functionality-driven policy.
2.  Check whether there are devices used that are functionally related to the policy but not to a specific application in particular.
3.  Validate if there is specific user-provided content that is functionally related but not tied to a particular application, and where the default user content types (such as `user_home_t`) are better not used. These resources need to be declared in the functionality-driven policy and probably made customizable as well:

    ```
    type public_content_t; # customizable
    files_type(public_content_t)
    ```

4.  Create the proper interfaces to handle or interact with these common resources:

    ```
    interface(`miscfiles_read_public_files',`
      gen_require(`
        type public_content_t;
      ')
      read_files_pattern($1, public_content_t, public_content_t)
    ')
    ```

## How it works…

Functionality-driven policy modules handle common resources for multiple applications and policies. Some example policies that handle the functional resources for multiple applications are the mail transfer agent policy (`mta`) and the web server policy (`apache`). Although the web server policy was originally intended to be purely for the Apache HTTPd, it has since evolved into a more functionality-driven policy supporting a large amount of web server technologies.

### Shared file locations

A helpful method for finding out what resources are considered to be functional in nature (rather than application-specific) is to imagine switching one application in favor of another. What resource types would remain the same if we switch from one system logger (say `syslog-ng`) to another (say `rsyslog`), or from Courier-IMAP to Cyrus? Having knowledge of multiple similar applications helps in finding out where (or what) the shared locations are.

However, having similar functional requirements doesn't necessarily make them shared. The locations should also remain the same (or at least be consistent and on well-known locations). Consider database files: the database files for PostgreSQL and SQLite databases both have the same functional purpose, but it makes no sense to label them both with the same label. Database files are specific to a particular database implementation and require specific labels, so with every potential common resource, make sure that the resource itself can be shared across multiple implementations.

Device nodes are a nice example to consider for a functionality-driven policy. An example device type definition would look like the following:

```
type cachefiles_device_t;
dev_node(cachefiles_device_t)
```

Devices are usually shared across multiple applications. Most devices are defined in the `devices.te` policy module with the proper interfaces being declared to allow access to the device (such as `dev_rw_cachefiles` for read/write access to the previously mentioned `cachefiles_device_t` type). Not all files in `/dev/` are such device files though.

Consider the `/dev/log` socket, which is used to send log events to the system logger. This socket, which is available regardless of the system logger being used, is made available through the following logging SELinux policy module:

```
type devlog_t;
files_type(devlog_t)
mls_trusted_object(devlog_t)
```

The `mls_trusted_object` interface makes the device (labeled `devlog_t`) accessible for all security levels in an MLS-enabled policy.

### User content and customizable types

User-provided content is also important to consider. For instance, for e-mail-related daemons, a user's `.forward` file (which tells the system where to forward the e-mails of the user) is available in his or her home directory and is definitely not owned by a particular application. Hence, its label (`mail_home_t`) is tied to a functionality-driven policy (`mta`).

Don't forget to mark user content as user content through the `userdom_user_home_content` interface; otherwise, end users will not be able to label or manipulate these files:

```
type mail_home_t;
userdom_user_home_content(mail_home_t)
```

Some user content is also best marked as customizable. A **customizable type**, when assigned to a resource, is ignored during standard relabel operations (usually performed by the system administrator) and as such, the resource label will not be changed back to what the SELinux configuration files have defined. This is particularly useful for resources whose path is not a fixed location and usually not made part of the SELinux file context definitions.

If the administrator does a forced relabel operation, then the file context is reset, even if the current type assigned to the resource is a customizable type:

```
~# restorecon -R -F /home/*

```

In a modular policy development, there is no notation available to mark a type as being a customizable type. To do this, the type needs to be added to the `customizable_types` file in `/etc/selinux/mcs/contexts/`.

Marking files with a customizable type is a solution when the path of the resource isn't fixed. The `.forward` file has a fixed path, so there is no need for customizable content. User content that should be publicly accessible, however, (marked as `public_content_t` or `public_content_rw_t`) does not have a fixed path; hence, those types are (by default) marked as customizable.

When full policy development is done (for instance, through the Linux distribution policy or because the developer controls the entire policy and not just additional modules), then the `# customizable` comment can be placed behind the type declaration, as can be seen from the following example of the CVS policy module:

```
type cvs_data_t; # customizable
files_type(cvs_data_t)
```

The reference policy build system will then automatically add the type to the `customizable_types` file during the build process.

## There's more...

Other common resources that can be considered are the TCP and UDP ports. Indeed, network-facing applications bind to one or more ports, which are usually the same for applications sharing the same functionality.

However, the TCP and UDP ports cannot be declared inside SELinux policy modules; instead, they need to be labeled as part of the base policy. Updating a base policy, however, is either done by the Linux distribution maintainers or the upstream reference policy project. The basic rule is that the ports are named after the service they are generally used by:

```
~$ getent services 6667
ircd    6667/tcp
~$ seinfo --portcon=6667
portcon tcp 6667 system_u:object_r:ircd_port_t

```

# Defining common helper domains

Next to the common resources, some applications share the same set of helper commands. The `sendmail` command is a nice example of this, which is executed by a large set of domains (usually, applications that need to send e-mails without using the SMTP protocol themselves). The `sendmail` application is well understood and most MTA applications support it for command-line e-mail sending operations.

Supporting such helper domains is usually done through a functionality-driven policy.

## How to do it…

Creating helper domains is similar to creating regular application domains, but the use of attributes allows the policy to be very flexible and usable by the application-specific policy modules developed further. Let's look at the MTA definition as an example of how this can be accomplished:

1.  Define an attribute for the command type:

    ```
    attribute mta_exec_type;
    ```

2.  Create a proper label type for the command, and assign it the `mta_exec_type` attribute:

    ```
    type sendmail_exec_t, mta_exec_type;
    application_executable_file(sendmail_exec_t);
    ```

3.  Configure an application domain for the command:

    ```
    type system_mail_t;
    application_domain(system_mail_t, sendmail_exec_t)
    ```

4.  If the application is for system purposes, assign the domain to the `system_r` role:

    ```
    role system_r types system_mail_t
    ```

5.  If the application is meant to be executed by end users, do not forget to include a `_run` or `_role` interface.
6.  Make an interface callable by third-party application domains to allow them to interact with the helper application:

    ```
    interface(`mta_send_mail',`
      gen_require(`
        attribute mta_exec_type;
        type system_mail_t;
      ')
      corecmd_search_bin($1)
      domtrans_pattern($1, mta_exec_type, system_mail_t)
    ')
    ```

7.  Make another interface allowing specific policies to mark their own helper executables usable for the same purpose (as they might not always use the same type):

    ```
    interface(`mta_agent_executable',`
      gen_require(`
        attribute mta_exec_type;
      ')
      typeattribute $1 mta_exec_type;
      application_executable_file($1)
    ')
    ```

## How it works…

Helper domains are meant to provide reusable functionality across multiple implementations. To support the flexibility of having multiple implementations, attributes are usually assigned to the types so that extensions can be easily created.

Consider the `sendmail` example again. Most implementations will have the command-line `sendmail` application marked as `sendmail_exec_t`. However, there are implementations whose `sendmail` binary has many more features, especially when called from the implementation processes themselves. Some implementations even have the file as a symbolic link to a more generic e-mail-handler application.

The Exim implementation, for instance, uses `exim_exec_t` instead of using `sendmail_exec_t`. With the use of the attributes, the Exim policy module can just call the proper interface (`mta_agent_executable`, in this case), so third-party applications can still execute the command (even though it is `exim_exec_t` and not `sendmail_exec_t`) and have it behave as expected (that is, with a transition to the `user_mail_t` or `system_mail_t` domain as expressed by the MTA policy):

```
type exim_exec_t;
mta_mailserver(exim_t, exim_exec_t)
mta_agent_executable(exim_exec_t)
```

Attributes allow other domains to interact with the newly defined type without having to update the policy modules that define these domains. This is because those domains are granted execute rights on all types that have the `mta_exec_type` attribute assigned, and will invoke a domain transition to the `system_mail_t` helper domain when they execute such a file. This privilege is provided through the `mta_send_mail` interface, which is a good example of a helper domain interface to be assigned to other domains:

```
interface(`mta_send_mail',`
  gen_require(`
    type system_mail_t;
    attribute mta_exec_type;
  ')
  corecmd_search_bin($1)
  domtrans_pattern($1, mta_exec_type, system_mail_t)
  allow $1 mta_exec_type:lnk_file read_lnk_file_perms;
')
```

# Documenting common privileges

Next to the helper domains, most functionality-driven policies also group privileges that can be assigned to domains. Such privileges could be to not only manage the common resources, but also to extend other domains with functional requirements as managed by the common policy.

All e-mail daemons need to be able to bind to the proper TCP ports, handle user mailboxes, and so on. By bundling these common privileges on the functional policy level, any evolution pertaining to the policy can be immediately granted to all domains inheriting privileges from the functional policy, rather than having to update each domain individually.

## How to do it…

Common privileges can be found in a wide variety. How common privileges are assigned depends on the use case. The following method, based on the e-mail server definition in the MTA policy, provides a flexible approach to this:

1.  Create an attribute for the functional domain to which common privileges are granted:

    ```
    attribute mailserver_domain;
    ```

2.  Define an interface where the attribute is assigned to a specified domain:

    ```
    interface(`mta_mailserver',`
      gen_require(`
        attribute mailserver_domain;
      ')
      typeattribute $1 mailserver_domain;
    ')
    ```

3.  Build an interface that assigns the functionally related common privileges to the specified argument. It should not assign attributes though! This is done with the following code:

    ```
    interface(`mta_mailserver_privs,`
      gen_require(`
        type mail_home_t;
      ')
      allow $1 mail_home_t:file read_file_perms;
       …
    ')
    ```

4.  Now, use the newly created interface to grant the proper permissions on the attribute:

    ```
    mta_mailserver_privs(mailserver_domain)
    ```

5.  If a specific application always has to inherit the privileges, assign the attribute to it:

    ```
    mta_mailserver(exim_t)
    ```

6.  If a specific application, however, optionally inherits the privileges, use the domain interface:

    ```
    tunable_policy(`nginx_enable_mailproxy',`
      mta_mailserver_privs(nginx_t)
    ')
    ```

## How it works…

When assigning privileges to a domain, there are two approaches that can be taken: either the privileges are assigned to an attribute (which is then associated with a domain) or the privileges are directly assigned to the domain. Which one to pick depends on how the policy is going to be used. Due to restrictions in policy development, it is not possible to optionally (that is, triggered through SELinux Booleans) assign attributes. Any attempt to do so will result in a build failure, as follows:

```
~$ make mymodule.pp
Compiling mcs mymodule module
checkmodule: loading policy configuration from tmp/mymodule.tmp
mymodule.te:23:ERROR 'syntax error' at token 'typeattribute' on line 1309:
#line 23
 typeattribute $1 mta_exec_type;
checkmodule: error(s) encountered while parsing configuration

```

As a result, whenever permissions can be granted optionally (through SELinux Booleans), policy developers will have to make sure that the permissions are granted directly (instead of assigning an attribute to the domain).

However, in most cases, using attributes for domains makes sense. The policy itself does not increase in size that much (as rules remain on an attribute level) and administrators can easily query which domains participate in the functional approach:

```
~# seinfo -amailserver_domain -x
 mailserver_domain
 system_mail_t
 exim_t
 courier_smtpd_t

```

Granting the permissions through an interface also allows us to quickly look at the impact of assigning an attribute, as we can then use the `seshowif` command:

```
~$ seshowif mta_mailserver_privs

```

The example given uses a server-domain approach, but the same can be done for a client.

# Granting privileges to all clients

The approach of using interfaces to aggregate privileges not only benefits domains that have the same functional purpose, but also clients. By combining the privileges for the set of clients, it is possible to enhance client privileges by only updating the interface rather than having to update all the clients' policy modules.

## How to do it…

Create a client interface that can be assigned to all clients of a particular functional purpose. The following steps extend an example policy with antimalware support:

1.  In the antimalware generic policy, create an `avcheck_client` attribute:

    ```
    attribute avcheck_client;
    ```

2.  Create the interface that assigns the attribute to a client domain:

    ```
    interface(`av_check_client',`
      gen_require(`
        attribute avcheck_client;
      ')
      typeattribute $1 avcheck_client;
    ')
    ```

3.  Create the interface that assigns the common privileges for client domains:

    ```
    interface(`av_check_client_privs',`
      …
    ')
    ```

4.  In the created interface, add the privileges that need to be assigned to all client domains. For instance, to enable a domain transition for the ClamAV `check` command, the following code is used:

    ```
    optional_policy(`
      clamav_domtrans_check($1)
    ')
    ```

5.  All domains that act as a client are either assigned the `av_check_client` (if the attribute can be assigned) or `av_check_client_privs` interface.

## How it works…

Suppose a new antimalware policy is developed for ClamAV, and we want the clients to be able to execute the `clamav_check_exec_t` applications and transition them to the `clamav_check_t` domain. Instead of updating all clients with a `clamav_domtrans_check` call, we only do this in the generic antimalware policy's `av_check_client_privs` interface, as follows:

```
optional_policy(`
  clamav_domtrans_check($1)
')
```

This ensures that all proper domains— not only those with the `avcheck_client` attribute—get the necessary privileges assigned.

Another example that uses this principle is the PulseAudio policy. An interface called `pulseaudio_client_domain` is made available and should be used by PulseAudio clients. Whenever the permissions for a PulseAudio client need to be updated, then the policy developer only needs to update the `pulseaudio_client_domain` interface instead of all client policy modules.

Such an approach makes policy development much more flexible and efficient, as developers do not need to update all possible client domains with the added privileges.

# Creating a generic application domain

In some situations, it makes sense to create a generic application domain, even though multiple implementations exist for the same functionality. Examples are the Java domain (which works for all the popular Java™ implementations) and init domain. When this occurs, carefully consider whether the generic application domain will always be sufficient, or whether specific application domains might come into play later. When this isn't clear, make sure that the policy being developed is flexible enough to cater both situations.

## How to do it…

In order to create a generic application policy that is still flexible with respect to potential specific policies that would be developed later, follow the upcoming set of steps:

1.  Identify the permissions that are (almost) always applicable to the functional domain, regardless of the implementation.
2.  Assign those permissions to a *base* implementation. For instance, for Java™ implementations, assign permissions as follows:

    ```
    attribute javadomain;
    # Minimal permissions
    java_base_runtime_domain(javadomain);

    type java_t;
    # Assigns javadomain attribute
    java_base_runtime(java_t);
    ```

3.  Add permissions that are applicable to at least one (or a few) of the implementations to the standard type. In our example, this would be to `java_t`. This ensures that `java_t` is generally usable for most Java™ implementations.
4.  Add the proper file contexts to allow most implementations to benefit from the generic application policy:

    ```
    /usr/lib/bin/java[^/]*  --  gen_context(system_u:object_r:java_exec_t,s0)
    /opt/(.*/)?bin/java[^/]*  --  gen_context(system_u:object_r:java_exec_t,s0)
    ```

## How it works…

With the given implementation, most Java™ implementations on an SELinux-enabled system will run, when executed, in the generic `java_t` domain: their executables are all marked as `java_exec_t` through generic file context expressions, and the `java_t` domain holds not only the set of least privileges for Java™ domains (as granted through the `javadomain` attribute that gets them from the `java_base_runtime_privs` interface), but also those privileges that are common for quite a few implementations. This means that the `java_t` domain has more privileges than needed in most cases, as it has to support a broad set of Java™ implementations.

However, when a specific implementation will be created with a different policy profile than the existing `java_t` domain, policy developers can easily mark this domain as a Java domain, inheriting the permissions that are necessary for every Java™ implementation (for instance, because they are mandated through the specifications of Java™) while staying clear from the other permissions that are granted to the generic `java_t` domain:

```
type icedtea_java_t;
java_base_runtime(icedtea_java_t)
```

By creating a more specific file context definition, the executable of the newly created type will get this label assigned (as the other expressions are more generic, and the SELinux utilities use a *most specific definition first* approach):

```
/opt/icedtea7/bin/java  --  gen_context(system_u:object_r:icedtea_java_exec_t,s0)
```

Building a proper set of least privilege rules is not easy and requires experience in policy development. If uncertain, it might be a good idea to use SELinux Booleans, such as used by the (generic) `cron` policy:

```
# Support extra rules for fcron
gen_tunable(fcron_crond, false)
…
tunable_policy(`fcron_crond',`
  allow admin_crontab_t self:process setfscreate;
')
```

Through this approach, specific implementations can still benefit from the generic policy declaration, if the amount of additional permissions is small. As the policy is enhanced with other implementation details, the need for the `tunable_policy` statement might be removed or a specific implementation for `fcron` can be developed separately.

# Building application-specific domains using templates

Specific domains have the advantage that they can contain those privileges needed by the domain, and no more. As there are no other application implementations using the specific domain, the privileges can be tailored to the needs of the application.

In certain situations though, it might be beneficial to automatically generate the types together with the basic permissions. Generating types is done through templates (rather than interfaces, although the underlying implementation of interfaces and templates is quite similar). The approach and development method is aligned with interface definitions and should pose no difficulties for developers to understand.

An example to consider with templates would be to automatically create system `cron` job domains for individual applications. Through a template, we can automatically create the domain, executable type, and temporary resource types as well as properly document the interactions of that domain with the main `cron` daemon (which is needed for communicating job failures or success, handling output, logging, and so on).

## How to do it…

Creating templates is similar to creating interfaces. To create templates, the following approach can be used:

1.  Start with a skeleton template inside the `.if` file, but call it `template` instead of `interface`:

    ```
    template(`cron_system_job_template',`
      …
    ')
    ```

2.  Add in the following type declarations:

    ```
    type $1_cronjob_t;
    type $1_cronjob_exec_t;
    application_domain($1_cronjob_t, $1_cronjob_exec_t)

    type $1_cronjob_tmp_t;
    files_tmp_file($1_cronjob_tmp_t)
    ```

3.  Grant the proper interactions between the main daemon and the newly defined types that are still inside the template definition:

    ```
    allow crond_t $1_cronjob_t:fd use;
    allow crond_t $1_cronjob_t:key manage_key_perms;
    domtrans_pattern(crond_t, $1_cronjob_exec_t, $1_cronjob_t)
    …
    ```

4.  In the application policy, call the template so that the new types are created. For instance, to create the `cron` job domains for Puppet, add the following code to `puppet.te`:

    ```
    cron_system_job_template(puppet)
    ```

5.  Enhance the (now available) `puppet_cronjob_t` domain with the permissions needed:

    ```
    allow puppet_cronjob_t …
    ```

## How it works…

The use of templates has been discussed earlier in the chapter on web server content. Indeed, the `apache_content_template` definition, too, is a template that creates additional types and documents the interaction between the newly created types and the (main) web server domain.

The use of templates allows for rapid policy development as well as properly isolated permission handling. When the main application evolves and requires additional permissions with respect to the specific application domains, or certain permissions are no longer needed, then only the template needs to be adjusted. All that is needed to apply the changes is to rebuild the SELinux policy modules, without any need to alter their individual source files.

It is a best practice to use prefix and/or suffix notations for template-provided types and to end the name of the template with `_template`. In theory, it is perfectly possible to create a template that creates the specified type(s) without any prefix and postfix expressions, instead requiring the various types to be passed on one at a time:

```
cron_system_job_template(puppet_cronjob_t, puppet_cronjob_exec_t, puppet_cronjob_tmp_t)
```

However, this approach is inflexible under the following circumstances:

*   If additional types need to be supported, then the interface API itself (the number of arguments passed to it and their meaning) needs to be altered, which makes such changes incompatible with earlier releases. This is important because there might be policy developers who are using this interface without their policy being available in the repository that we're developing in, so we cannot refactor this code ourselves.
*   If a type is no longer needed, then either the interface API itself needs to be changed (making it incompatible with earlier releases) or the interface will be made to ignore a particular type (which easily becomes a development nightmare).
*   Developers will continuously need to look at the order and meaning of the types in order not to mistakenly have the executable type marked as a domain and vice versa.

Such an approach would also make it possible to create confusing type definitions:

```
cron_system_job_template(puppetjob_t, pj_exec_t, ptmp_t)
```

Through such an approach, developers and administrators would lose sight over the relation between types.

Using proper prefix and postfix notations allows for a simplified management. The use of a template such as `cron_system_job_template` easily informs developers that there will be several types matching `*_cronjob_t`, `*_cronjob_exec_t`, and `*_cronjob_tmp_t`. Policy developers and system administrators easily learn that these are related with each other.

# Using fine-grained application domain definitions

The use of templates earlier in this chapter is a start to support more fine-grained application domain definitions. Instead of running a workload inside the same domain as the main application, specific types are created that are meant to optimize the interaction between one domain and another, ensuring that the permissions granted to a particular domain remain small and manageable.

Using fine-grained application domains goes a step further, having processes of the same application run inside their own specific domains. This is not always possible (not all applications use multiple, distinct processes), but when it is, using fine-grained domains provides an even more secure environment, where each task runs with just the permissions needed for that individual task, even though the application, in general, needs more permissions.

An example implementation of fine-grained application domain definitions is the postfix policy, which will be used as an example in this recipe. The Postfix e-mail server is well documented and its architecture has been quite stable, making it a prime candidate for a fine-grained policy development approach.

However, when fine-grained application domains are used, policy development and maintenance itself becomes harder. Individual interaction changes between processes (which might be the case with newer versions of an application) require policy updates much more often than when all processes run within the same SELinux domain.

## How to do it…

The following checks can be taken to see whether fine-grained application domains make sense or not:

1.  Does the application architecture use multiple processes, with each process having a distinct functional task? If not, then creating fine-grained application domains will not help much as every domain will have the same permissions anyhow.
2.  Are there processes with different access vectors (and thus are vulnerable to different threats than others)? For instance, whether some processes are directly accessible through the network whereas others are local? If so, then using fine-grained application domains might make sense to reduce the impact in case of the vulnerability exploitation.
3.  Is there an interaction between a subset of the processes with other domains (not managed through the same application), whereas the other processes do not need to interact with these domains? If so, then using fine-grained application domains might make sense to limit exposure of resources to other applications.
4.  Does the application support different roles that might need to interact with some (but not all) of the processes? A single full-application administrator might still need administrative privileges to all processes and resources, but other roles might not have this requirement. Using fine-grained application domains allows for fine-grained roles as well.

## How it works…

Supporting fine-grained application domains is usually done for risk mitigation. But besides risk mitigation, it also provides advantages in role management as well as a more efficient approach to managing types that are inherited from the domain.

### Reducing exploit risks

Consider a part of the Postfix architecture, as shown in the next diagram:

![Reducing exploit risks](img/9669OS_07_02.jpg)

The **smtpd** daemon handles the reception of an e-mail through the **network**, and as such, is more prone to remote vulnerability exploits than to locally running processes such as the **cleanup** process or even the **qmgr** process.

By limiting resource access of the **smtpd** daemon to just the resources it needs, exploits that would attempt to access the queues (resources not usually accessed by **smtpd** but used by **qmgr**) would fail as the least privilege approach used in the **smtpd** domain (`postfix_smtpd_t`) disallows access to the **maildrop** queues (`postfix_spool_maildrop_t`).

Proper risk reduction is only possible if the resources of the application (such as the specific queues) are also defined in a fine-grained manner. If the application has multiple configuration files and these configuration files are read by different functional processes, then the configuration files should be labeled more specifically as well (for instance, configuration files for routing and configuration files for network settings).

If the application resources are labeled in a generic fashion, we risk that all fine-grained domains have the same rights towards the generic resources, making it more plausible for a vulnerable application to be exploited with larger consequences to the entire application architecture.

### Role management

Using fine-grained application domains goes further than just mitigation of exploits. With individual domains, role access can be granted to users allowing them to take specific actions without requiring full application privileges.

For instance, operator roles can be created that allow manipulation of the Postfix deferred queue and signaling of the `qmgr` process without granting those users any specific rights towards the other processes. Assuming the user domain for this role is `postoper_t`, this would be accomplished as follows:

```
postfix_signal_qmgr(postoper_t)
postfix_manage_maildrop(postoper_t)
```

### Type inheritance and transitions

When a domain creates new resources, these resources are assigned a type based on the label of the domain as well as the transitions defined in the SELinux policy. A process that is launched by a domain by default (that is, when no transitions are defined in the policy) inherits the label of the parent domain, while a file created inside a directory by default inherits the type of that parent directory. In the case of labeled network support, the packets are labeled based on the parent socket label.

Sometimes the creation of a resource cannot be tied to a parent domain or parent resource, making it impossible for SELinux to deduce the label to assign to this resource. For this reason, **initial SIDs** are provided by the SELinux policy. These tell the SELinux subsystem what the default label is for such resources if no label can be deduced.

For instance, the initial SIDs for a (TCP/UDP) port and for a file are as follows:

```
sid port gen_context(system_u:object_r:port_t,s0)
sid file gen_context(system_u:object_r:unlabeled_t,s0)
```

The definition of initial SIDs is part of the base SELinux policy and cannot be altered using SELinux policy modules. Luckily, there is little reason for SELinux developers to ever touch the initial SID definitions.

These label inheritance rules are important in a fine-grained application domain design. Applications that use multiple processes also tend to use resources such as shared memory for **inter-process communication** (**IPC**). When all processes run with the same domain, the shared memory is also labeled the same (such as `postgresql_tmpfs_t` for the PostgreSQL managed shared memory) as a file transition would be put in place:

```
# /dev/shm/ shared memory
type postgresql_$1_tmpfs_t;
files_tmpfs_file(postgresql_$1_tmpfs_t)
…
fs_tmpfs_filetrans(postgresql_$1_t, postgresql_$1_tmpfs_t, file)
```

When using multiple domain definitions, it is possible that shared memory segments are labeled differently as well (depending on which process creates the shared memory segments, of course), so even IPC can then be properly governed. Separate file transitions would be put in place depending on the domain that is creating a shared memory segment.

Next to file transitions, policy developers can also introduce domain transitions (which changes the label of the newly created process) using the `domtrans_pattern` definition. Inside the Postfix policy, this is used to create the fine-grained process architecture:

```
domtrans_pattern(postfix_master_t, postfix_postqueue_exec_t, postfix_postqueue_t)
domtrans_pattern(postfix_master_t, postfix_showq_exec_t, postfix_showq_t)
```

Such domain transitions can also be supported through the interfaces, as we've seen in the earlier chapters, such as the `postfix_domtrans_smtp` interface:

```
interface(`postfix_domtrans_smtp',`
  gen_require(`
    type postfix_smtp_t, postfix_smtp_exec_t;
  ')
  corecmd_search_bin($1)
  domtrans_pattern($1, postfix_smtp_exec_t, postfix_smtp_t)
')
```

A third transition type that SELinux supports is the dynamic domain transition. Such SELinux policy rules inform the SELinux subsystem that a process can change its own type dynamically—without needing to execute a file. This does require the application to be SELinux-aware (that is, be able to interact with the SELinux subsystem itself). For instance, inside the FTP policy, the following interface is made available to support domains dynamically transitioning to the `anon_sftpd_t` domain:

```
interface(`ftp_dyntrans_anon_ftpd',`
  gen_require(`
    type anon_sftpd_t;
  ')
  dyntrans_pattern($1, anon_sftpd_t)
')
```

In our Postfix example, we used the `/dev/shm/` shared memory, but there is also POSIX shared memory, which is governed through the `shm` class. This shared memory inherits the label from the domain itself, so if two applications (such as `postfix_pickup_t` and `postfix_cleanup_t`) use POSIX shared memory, then the target label is inherited from the process that creates the shared memory region:

```
allow postfix_pickup_t postfix_cleanup_t:shm rw_shm_perms;
```

Without fine-grained access controls, this would all be handled by a single domain (say `postfix_t`) and shared memory access controls would be very limited.