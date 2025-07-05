# Chapter 4. Creating a Desktop Application Policy

In this chapter, we will cover the following topics:

*   Researching the application's logical design
*   Creating a skeleton policy
*   Setting context definitions
*   Defining application role interfaces
*   Testing and enhancing the policy
*   Ignoring permissions we don't need
*   Creating application resource interfaces
*   Adding conditional policy rules
*   Adding build-time policy decisions

# Introduction

Up until now, we've modified and enhanced existing policies and interacted with the SELinux subsystem through the available administrative commands. But, in order to truly benefit from the protection measures that SELinux provides, we need to create our own policies for applications that would otherwise run with either too many privileges, or not run at all.

Desktop applications are a good example. The end user domains (`unconfined_t` for policies which support unconfined domains, and `user_t`, `staff_t`, and the like for the other policies) have many privileges assigned to them to allow generic applications to be executed while remaining in the user domain.

This has a huge downside: vulnerabilities within desktop applications or malfunctioning applications can create havoc with the users' files and resources, potentially exposing information to malicious users. If all end user applications run within the same domain, then we cannot talk about a least privilege environment. After all, this single user domain then has to have the sum of all privileges needed by various applications.

In this chapter, we will create a desktop application policy for Microsoft Skype™, a popular text messaging, voice, and video call application, which also runs on Linux systems, but is proprietary and thus its code cannot be reviewed to find what it might do. Confining this application ensures that the application can only perform the actions we allow it to do.

# Researching the application's logical design

Before embarking on a policy development spree, we need to look at the application's behavior and logical design. We will get to know the application and its interactions as we begin to model this into the SELinux policy.

## How to do it…

To prepare an SELinux policy for the application, let's first look at how the application behaves:

1.  Look into the files and directories that the application will interact with and write down the privileges that the application needs. Try to structure access based on the functionalities of the application.
2.  Figure out which network resources are required by the application, which ports does the application bind (listen) to (if any), and which ports does it need to connect to.
3.  If the application needs to interact with other SELinux domains (processes), how does this interaction look (or what is it for)?
4.  Does the application require specific hardware access or other kernel-provided resources?

## How it works…

Gathering information on at least these four resources (files, network, applications, and hardware/kernel) helps us to start with a skeleton policy file. In the end, we might have a schematic representation of these resources, as shown in the following diagram:

![How it works…](img/9669OS_04_01.jpg)

Let's look at how this works out for our example.

### Files and directories

There are three main file accesses needed for the Skype™ application.

The first is its own, user-specific configuration, which is stored at `~/.Skype/`. This will contain all settings for the application, including contact list, chat history, and more. In SELinux, user-specific configuration entries are labeled as `*_home_t` and marked as user home content, allowing the end user to still manage these resources.

The second consists of the generic user files, which our application needs access to in order to upload or download files. This can be any end user file, although some distributions create specific support for this (such as through a `~/Downloads/` location).

The third consists of the general resources of the Unix system that are available for the application. This access is needed for the application to load the necessary libraries. During application policy development, this is often not mentioned, as it is a default access provided to all applications.

### Network resources

The application needs to interact with network resources through its messaging, voice, and video chat functionality.

In general, we know that the application needs to connect to the central Skype™ infrastructure for all centrally managed services, such as authentication, directory searches, and more. This connection will be through TCP.

Next to the central infrastructure, the application will also connect to the Skype™ instances of other users for direct communication. This connection will be through both TCP and UDP (as UDP is more common for video and voice).

### Processes

As the application is a graphical application, we know that it needs to interact with the X11 server running on the workstation. As we will see in the recipes in this chapter, this automatically requires a set of types and permissions to be assigned to the application.

Other than that, there are no specific interactions with other domains.

### Hardware and kernel resources

Finally, on the hardware level, the application will need access to the video and sound devices (for the webcam and voice call functionality, respectively).

The application will also need to use the user terminals in case of errors (so that the error message can be displayed).

# Creating a skeleton policy

With the logical setup now in place, we can draft a skeleton policy. This policy will be a translation from the logical setup we encountered to SELinux policy rules.

The entire policy is written in a `myskype.te` file. The final result of this set of recipes is also available through the download pack of this book as a reference.

## How to do it…

We start with a base skeleton that we can enhance later. This skeleton is developed as follows:

1.  We start with the declaration of the various types. From the design, we can deduce four types:

    *   `skype_t` as the main process domain
    *   `skype_exec_t` as the label for the Skype executable(s)
    *   `skype_home_t` for the user configuration files and directories of the `skype_t` domain
    *   `skype_tmpfs_t` is needed for shared memory and the X11 interaction

    The code to deduce these four types is as follows:

    ```
    policy_module(myskype, 0.1)

    attribute_role skype_roles;

    type skype_t;
    type skype_exec_t;
    userdom_user_application_domain(skype_t, skype_exec_t)
    role skype_roles types skype_t;

    type skype_home_t;
    userdom_user_home_content(skype_home_t)

    type skype_tmpfs_t;
    userdom_user_tmpfs_file(skype_tmpfs_t);
    ```

2.  Next, we write up the policy rules for accessing the various types, starting with the manage rights on `~/.Skype/`:

    ```
    # Allow manage rights on ~/.Skype
    manage_dirs_pattern(skype_t, skype_home_t, skype_home_t)
    manage_files_pattern(skype_t, skype_home_t, skype_home_t)
    userdom_user_home_dir_filetrans(skype_t, skype_home_t, dir, ".Skype")
    ```

3.  We enable the `X11` access and shared memory. This is a common set of privileges that need to be assigned to X11-enabled applications:

    ```
    # Shared memory (also needed for X11)
    manage_files_pattern(skype_t, skype_tmpfs_t, skype_tmpfs_t)
    manage_lnk_files_pattern(skype_t, skype_tmpfs_t, skype_tmpfs_t)
    manage_fifo_files_pattern(skype_t, skype_tmpfs_t, skype_tmpfs_t)
    manage_sock_files_pattern(skype_t, skype_tmpfs_t, skype_tmpfs_t)
    fs_tmpfs_filetrans(skype_t, skype_tmpfs_t, { file lnk_file fifo_file sock_file })

    # Application is an X11 application
    xserver_user_x_domain_template(skype, skype_t, skype_tmpfs_t)
    ```

4.  Next, we write down the network access rules, as follows:

    ```
    # Network access
    corenet_tcp_bind_generic_node(skype_t)
    corenet_udp_bind_generic_node(skype_t)
    # Central skype services
    corenet_tcp_connect_http_port(skype_t)
    corenet_tcp_connect_all_unreserved_ports(skype_t)
    # Listen for incoming communication
    corenet_tcp_bind_all_unreserved_ports(skype_t)
    corenet_udp_bind_all_unreserved_ports(skype_t)
    ```

5.  Finally, we have the device accesses:

    ```
    # Voice and video calls
    dev_read_sound(skype_t)
    dev_read_video_dev(skype_t)
    dev_write_sound(skype_t)
    dev_write_video_dev(skype_t)
    # Terminal (tty) output
    userdom_use_user_terminals(skype_t)
    ```

## How it works…

In the skeleton policy, we start with the SELinux policy rules that we know will be necessary. If we are somewhat uncertain about one or more rules, it is perfectly fine to comment them out for starters and enable those as we move on to the testing phase in the *Testing and enhancing the policy* recipe later.

The skeleton starts off with the type declarations, which focus on the resources of the application. We then enhance the application domain with the proper privileges towards these resources. After the resource access, we look at the X11 privileges and finish with the network interaction of the application.

### Type declarations

The first part of any policy is the declaration of types and roles. We first create a role attribute called `skype_roles` to which the `skype_t` SELinux domain is granted. This role attribute will then be assigned to the users who are allowed to call the application. Next, we list the various SELinux types that the policy will provide and also give those types a specific meaning. For instance, the `skype_t` and `skype_exec_t` types are given the proper meaning through the `userdom_user_application_domain` template. This template looks like the following:

```
interface(`userdom_user_application_domain',`
        application_domain($1, $2)
        ubac_constrained($1)
')
```

The `application_domain` template, which is called from within `userdom_user_application_domain`, has the following definition:

```
interface(`application_domain',`
        application_type($1)
        application_executable_file($2)
        domain_entry_file($1, $2)
')
```

This results in the `skype_t` domain to be marked as an application type (a true domain), whereas `skype_exec_t` is an executable file, which can be used as an entry point to the `skype_t` domain. Finally, `skype_t` is marked as `ubac_constrained`, which is used in case of **User-based access control** (**UBAC**), where access to resources is not only governed through the types and its access vectors, but also through the SELinux user. In other words, if the SELinux user, `userX_u`, would somehow be able to access the processes of another SELinux user (`userY_u`), then the `skype_t` domain will not be reachable as the UBAC constraints would come in action, preventing any interaction between the two.

All `userdom_user_*` templates mark the associated resources as UBAC constrained, together with the true file type association, so `userdom_user_tmpfs_file` marks the file not only as a `tmpfs_t` file (the type used for shared memory), but also makes it UBAC constrained.

### Managing files and directories

Next, we provide the access rights to files and resources. In the example, we limit access to `~/.Skype/` only and automatically mark `~/.Skype/` as `skype_home_t` when it is created inside a user home directory (through `userdom_user_home_dir_filetrans`), even though we identified the need to manage user content files as well. This is because we need to make a policy design decision here—do we want the application to have full access to all user resources or would we rather limit the access? And inversely, do we want other applications that can access user content to access Skype™ user (configuration) data?

If we do not want the application to access any user content, then we do not need to add any rules: the policy will only allow search rights through the user home directory (in order to locate `~/.Skype/`) and deny everything else.

If we would like to grant the application access to the user content, we can add in the following calls:

```
userdom_manage_user_home_content_files(skype_t)
userdom_manage_user_home_content_dirs(skype_t)
```

This will grant full manage rights on user files and directories to the `skype_t` domain.

In the Gentoo Linux policy, additional types have been made available to provide a finer-grained access control to user files. These types map to the **XDG Base Directory Specification** (**XDGBDS**) as provided by the Free Desktop community, and include the `xdg_downloads_home_t` type. End users can mark files and directories as `xdg_downloads_home_t` and allow applications to have selective access to user files, without risking that these applications have access to the more private files of that user.

In Gentoo, this means that the following call can be added to the policy:

```
xdg_manage_downloads_home(skype_t)
```

### X11 and shared memory

When an application needs to interact with the X11 server (as a client application), much of this interaction is done through shared memory. In Linux, shared memory can be interpreted as files on a tmpfs mount (think `/dev/shm/`) although other shared memory constructions are still possible without tmpfs.

In SELinux, policy developers want to make sure that this shared memory is labeled specifically for the domain. For this, they create a type with `_tmpfs_t` as the suffix. In our example, this is `skype_tmpfs_t`. Of course, we need to grant manage rights on the shared memory (for all classes that will be used) to the `skype_t` domain. In case of X11 interaction, these are files, symbolic links, FIFOs, and sockets.

Next to the manage rights, we also include a file transition: whenever `skype_t` creates a file, symbolic link, FIFO, or socket in a `tmpfs_t` labeled location, then this resource should be automatically labeled `skype_tmpfs_t`. This is done through the `fs_tmpfs_filetrans` call.

Finally, we use `xserver_user_x_domain_template` that contains all the SELinux privileges necessary for both the X11 client as well as X11 server to interact with each other. This template uses a prefix argument (the first argument, which we provided as `skype`), which will be used to create an X11 resource type called `skype_input_xevent_t`. Similar to what we've seen for web servers (where an `apache_content_template` call was used), this template gives an easy approach to automatically build additional types and enable the X11 support.

Next to the prefix, the domain itself is passed (`skype_t`) and the label used for the shared memory (`skype_tmpfs_t`) are passed on as those are needed for the X11 server support.

### The network access

For the network access, we start by providing the `skype_t` domain with bind privileges on a TCP socket and its IP address (which is represented by `node_t`).

Next, we allow the `skype_t` domain to connect to the central Skype™ services, which are available on HTTPS port `443` (authentication) and various seemingly random high TCP ports (network nodes). The HTTP target port is identified as an `http_port_t` type, the others are for the unreserved ports.

Finally, we allow the `skype_t` domain to listen for incoming communications. By default, this is on a high TCP port for messages and state information, while for voice and video chat, this is through UDP.

A simple way to identify the necessary types is to look at the `netstat` output, as it shows us what ports a process is listening on, the protocol family (TCP or UDP), as well as which ports it is connecting to:

```
~$ netstat -naput | grep skype
tcp  0  0 0.0.0.0:34431  0.0.0.0:*  LISTEN  8160/skype
tcp  0  0 10.221.44.241:40650  111.221.77.150:40008  ESTABLISHED  8160/skype
…
udp  0  0 0.0.0.0:34302  0.0.0.0:*    8160/skype

```

## There's more...

The access to the sound and video devices is trivial, but during the design, it is very well possible that many more accesses are already identified (as ours is just an example). As we continue developing policies, writing a skeleton policy will become more trivial.

A great source for learning more about the policies is to look for an existing policy of a similar application, or an application that has certain functionalities that resemble the functionalities offered by the application we're writing a policy for. For Skype™, we could look at the policy of Gift (a peer-to-peer file sharing application), which is an end user, graphical application with peer-to-peer communication flows, supporting uploading and downloading files.

After all, SELinux policies are a write-down of what the expected behavior is of a domain. If another application has the same or similar behavior, then its policy will be very similar too.

In the previous example, we grouped the permissions together based on the functional need. However, the coding style for SELinux policy files, as mentioned by the reference policy, uses a different grouping, so make sure that if the policy would be sent upstream, this coding style is followed instead.

## See also

*   For more information about the XDGBDS, see [http://standards.freedesktop.org/basedir-spec/latest/](http://standards.freedesktop.org/basedir-spec/latest/)

# Setting context definitions

The next step in the policy development is to mark its resources with the proper file contexts. This will label the files of the application correctly, making sure that the SELinux policy makes the right decisions.

## How to do it…

To update the file context definitions, follow the next set of steps:

1.  Create the `myskype.fc` file and add in the definition for `~/.Skype/`:

    ```
    HOME_DIR/\.Skype(/.*)?  gen_context(system_u:object_r:skype_home_t,s0)
    ```

2.  Next, add in the definitions for the `skype` binaries:

    ```
    /opt/skype/skype  --  gen_context(system_u:object_r:skype_exec_t,s0)
    /opt/bin/skype  --  gen_context(system_u:object_r:skype_exec_t,s0)
    /usr/bin/skype  --  gen_context(system_u:object_r:skype_exec_t,s0)
    ```

## How it works…

The definitions for the binaries are standard, path-based context declarations. The one for the user home directory, however, is special.

As can be seen from the example, the path starts with `HOME_DIR`. This is a special variable used by SELinux libraries, which automatically maps to all Linux users' home directories. Rather than creating a `/home/[^/]*/\.Skype(/.*)?` context directly, which has the design problem that home directories on other locations (such as `/export/home/user/`) will not match, the SELinux libraries will check the home directories of all real users (with a user ID starting at `500`, although this is configurable) and for each different home root directory (`/home/` is the most commonly used one), it will generate the proper contexts.

The result of this operation is stored as the `file_contexts.homedirs` file inside `/etc/selinux/mcs/contexts/files/` and is automatically created during policy build (through the `genhomedircon` command).

Next to `HOME_DIR`, other supported variables are `HOME_ROOT` (which represents the home root path) and `ROLE` (which is the first role associated with a user).

# Defining application role interfaces

Finally, before testing the policy, we need to create a role interface and assign it to the user domain that will be used to test (and run) the application. If we don't create a role interface and assign it to a user domain, then the user domain will either have no privileges to execute the application at all, or the application will run with the user context rather than the newly defined `skype_t` domain. If the user domain isn't unconfined, then chances are that the application will fail.

## How to do it…

Role interfaces are the gateways of a policy. They ensure that domains and SELinux users can interact with the application and that the set of privileges for a particular application are coherent.

We create such an interface in the `.if` file and then assign this interface to a user domain in order to test the interface:

1.  Create the `myskype.if` file with the following interface in it:

    ```
    interface(`skype_role',`
      gen_require(`
        type skype_t, skype_exec_t, skype_tmp_t, skype_home_t;
      ')
      # Allow the skype_t domain for the user role
      roleattribute $1 skype_roles;
      # Allow domain transition for user domain to skype_t
      domtrans_pattern($2, skype_exec_t, skype_t)
      # Interact with skype process
      ps_process_pattern($2, skype_t)
      allow $2 skype_t:process { ptrace signal_perms };
      # Manage skype file resources
      manage_dirs_pattern($2, skype_home_t, skype_home_t)
      manage_files_pattern($2, skype_home_t, skype_home_t)
      manage_lnk_files_pattern($2, skype_home_t, skype_home_t)
      # Allow user to relabel the resources if needed
      relabel_dirs_pattern($2, skype_home_t, skype_home_t)
      relabel_files_pattern($2, skype_home_t, skype_home_t)
      relabel_lnk_files_pattern($2, skype_home_t, skype_home_t)
    ')
    ```

2.  Create a policy for the user domain (for instance, `myunprivuser.te`) that grants regular users access to the `skype_t` domain, by assigning the user domain the `skype_role` call:

    ```
    policy_module(myunprivuser, 1.0)
    gen_require(`
      type user_t;
      role user_r;
    ')
    optional_policy(`
      skype_role(user_r, user_t)
    ')
    ```

3.  Build both policies and load them. Then, relabel the `skype` binary files (and possibly preexisting `~/.Skype/` locations):

    ```
    ~# restorecon /opt/skype/bin/skype /opt/bin/skype /usr/bin/skype
    ~# restorecon -RF /home/user/.Skype

    ```

## How it works…

Although we have defined all the rules for the `skype_t` domain that we think are needed (in the next recipe, the policy will be extended until it really works), we have not defined the rules yet to allow a user domain to actually execute the `skype_exec_t` binaries and have the process run in the `skype_t` domain.

To accomplish that, we need to ensure that a domain transition occurs to the `skype_t` domain when the user executes `skype_exec_t`. This is handled by the `domtrans_pattern` call. But before we allow the domain transition, we first need to allow the `skype_t` domain for the user role, which is done through the `roleattribute` call.

Until now, we focused primarily on type enforcement rules (that is, granting privileges to SELinux domains based on the label of the target resource). In order to allow certain users to run an application, the application domain itself needs to be granted to the user role. This is supported through SELinux's **role-based access control** **(RBAC**) model. This RBAC model ensures that a certain domain (`skype_t`, in our example) can only be used by the roles we configure it for (`user_r`, in our example). Other roles, such as DBA roles (`dbadm_r`) might have no need for running the Skype™ application, so they will not be granted access to the `skype_t` domain.

### Note

Not granting a domain does not necessarily prevent the application from executing within the user domain itself. To accomplish that, we need to make sure that the executable file type cannot be executed by other roles. Instead of using `userdom_user_application_domain` for the `skype_t` and `skype_exec_t` types (which would assign a generic executable attribute to the `skype_exec_t` type), we would use something similar to the following:

```
application_type(skype_t)
files_type(skype_exec_t)
allow skype_t skype_exec_t:file { entrypoint mmap_file_perms ioctl lock };
ubac_constrained(skype_t)
```

As the user domain, which needs to be able to execute Skype™, also needs to manage the `skype_home_t` files (in case, manual intervention in `~/.Skype/` is needed or to make backups), we grant it both manage privileges as well as relabel privileges. The relabel privileges are needed when, for instance, a backup is restored.

For the user domain, we then call the `skype_role` interface we just created. In the example, we used the `optional_policy` statement. This allows policy modules to be loaded even when one of the calls cannot be resolved or is not supported.

Suppose we need to unload the `myskype` module. Without the `optional_policy` statement, the `myunprivuser` module would need to be unloaded as well, even though this policy module might contain other rules that are important for the user domain to work correctly (in the example, we only called the `skype_role` interface, but after some time, the module might call many other interfaces as well). If we don't unload the module and no `optional_policy` statements are used, then SELinux will warn the administrator about unresolved dependencies between the modules.

With the `optional_policy` statement, the SELinux tools know that the call might become unresolvable, in which case, the entire block (everything inside the `optional_policy` block) will be ignored while the module remains loaded.

## There's more...

At the beginning of the recipe, we mentioned that unconfined user domains will be able to execute the application without a domain transition. This is to be expected, as the entire idea behind unconfined domains is that they are, well, unconfined.

It is considered a bad practice to, in general, create domain transitions from an unconfined domain to confined domains. Only in very specific circumstances do domain transitions from an unconfined domain to confined domains make sense (such as when the target domain is used to confine potentially vulnerable applications, such as a sandbox domain).

From a security perspective, it makes more sense to confine users immediately and use the proper domain transitions between (confined) user domains and the application domains.

# Testing and enhancing the policy

With the policy ready and loaded, it is time to start testing the application from a user's perspective, while keeping an eye on the audit logs (for denials) and application output.

Testing the application is an important phase of policy development and will also be the most time consuming task. During testing, several functional features of the application will be tried and the resulting permissions (SELinux-wise) will need to be added to the policy.

In previous recipes, such as *Creating a skeleton policy*, we enabled a set of permissions based on other policies and common sense. However, these permissions have not been validated and tested yet. In this recipe, we will assert that the permissions are truly needed, as we do not want to create a policy with too many rights associated with it.

## How to do it…

Testing policies is a repetitive task. Every try-out means that the AVC denials leading up to the start need to be discarded (as we do not want to include privileges not related to the test) after which the application is tested and the results are documented. Depending on how the application acts, new policy rules are added to the policy:

1.  Write down the current timestamp or create a reference point inside the audit logs (for instance, by reloading the SELinux policy), so we know from which point in the audit logs we need to look at the audit events:

    ```
    ~# semodule -R

    ```

2.  As an end user, start the application (from a terminal window) and watch what happens.
3.  Write down the error that is displayed (if any):

    ```
    ~$ skype
    skype: error while loading shared libraries: cannot restore segment prot after reloc: Permission denied

    ```

4.  Look into the denials as displayed in the audit logs:

    ```
    ~# ausearch -m avc -ts recent

    ```

5.  For each first denial or denial related to the error shown earlier, try to enhance the policy with the proper call and try again.

## How it works…

In this phase, we are enhancing the policy step by step. Some policy developers like to run the application in permissive mode (either by running the entire system in permissive mode or by marking this particular domain as a permissive domain), registering all accesses performed (through the AVC denials) and enhancing the policy based on that information. Although this might give a faster working policy, these developers will also risk that they add too many privileges to a policy, something that is very difficult to challenge and change later.

Instead, we let SELinux prevent accesses and look at how the application reacts. Based on the error logging of the application or the behavior of the application and the AVC denial(s) seen through the logs, we can have a good picture of what privileges are really needed.

For instance, simultaneously with the error presented in the example, the following denial occurred:

```
type=AVC msg=audit(1398930752.113:608): avc: denied { execmod } for pid=8943 comm="skype" path="/opt/bin/skype" dev="dm-2" ino=801 scontext=user_u:user_r:skype_t:s0 tcontext=user_u:user_r:skype_exec_t:s0 tclass=file
```

It is important that we focus on the first set of denials that occur and not on all denials shown. It is very likely that denials shown after the first set of denials are from error handling routines, either by the application or the system in general, which would never be triggered in the first place if the proper permissions are granted to the domain. Trying to grant those privileges as well would result in a too broadly defined set of permissions.

The preceding denial shown would result in the following addition to the policy:

```
# Error 'cannot restore segment prot after reloc'
allow skype_t skype_exec_t:file execmod;
```

# Ignoring permissions we don't need

After repeated testing, we will have a policy that works, even though denials might still show up in the audit logs. In order not to alarm any administrator, we might want to disable auditing of those specific denials (while, of course, ensuring that critical access vectors are still logged by the audit daemon).

## How to do it…

In order to disable logging of certain denials that do not influence an application's behavior, trigger the denial and then register the `dontaudit` statements as explained in the following steps:

1.  For each denial shown in the audit logs, we need to find the corresponding `dontaudit` rule set. Consider the following instance:

    ```
    type=AVC msg=audit(1398936489.877:2464): avc: denied { search } for pid=8241 comm="skype" name="modules" dev="dm-0" ino=1322041 scontext=user_u:user_r:skype_t:s0 tcontext=user_u:object_r:user_home_t:s0 tclass=dir
    ```

2.  Search through the SELinux policies for `dontaudit` statements on this matter:

    ```
    ~$ sefindif dontaudit.*user_home_t.*search
    interface(`userdom_dontaudit_search_user_home_content',`
     dontaudit $1 user_home_t:dir search_dir_perms;

    ```

3.  Add in the interface call to the policy, rebuild the policy, and then reload it. Repeat until all cosmetic denials are no longer visible.

## How it works…

Many operations performed by applications can be seen as cosmetic—although in the example, the application really performs the searches through the user files, they are not needed for the application to function correctly. For instance, it might be searching through the entire directory until it finds its own files, which it does have access to.

By adding the `dontaudit` statements for these operations, we ensure that the audit logs stay clean.

In case of problems, the administrator can still disable the `dontaudit` statements in the policy, revealing every denial that SELinux has triggered (even those that are explicitly marked as `dontaudit`):

```
~# semodule -DB

```

To re-enable the `dontaudit` statements, rebuild and reload the policy:

```
~# semodule -B

```

In certain situations, there might not be an interface related to `dontaudit` available. In that case, create a new interface (as part of an SELinux policy module) with the `dontaudit` rules defined in it. For instance, for a `dontaudit` rule set to ignore getting the attributes of `mozilla_home_t` content, we would create a `mymozilla` policy module with the `mozilla_dontaudit_getattr_home` interface declared in it.

# Creating application resource interfaces

Our application policy is almost ready for deployment. However, it currently is mainly end user focused, and there are no ways of interacting with the `skype_t` domain (or other resources managed by the `skype` module) except through the `skype_role` interface.

In this recipe, we'll add an interface for reading `skype_home_t`.

## How to do it…

Alongside the `skype_role` interface that we created in the *Defining application role interfaces* recipe, we need to create additional resource interfaces so that other domains can easily interact with the newly created policy:

1.  Open the `myskype.if` file and add in the following content:

    ```
    interface(`skype_read_home',`
      gen_require(`
        type skype_home_t;
      ')
      userdom_search_user_home_dirs($1)
      allow $1 skype_home_t:dir list_dir_perms;
      allow $1 skype_home_t:file read_file_perms;
      allow $1 skype_home_t:lnk_file read_lnk_file_perms;
    ')
    ```

## How it works…

The recipe itself is simple—for each interaction with resources managed by the `skype` module, we need to create an interface that can be called by other modules.

Each interface should be complete. For instance, in order to read the `skype_home_t` content, a domain will first need to be able to search through the user's home directory (`user_home_dir_t`, which is not the same as `user_home_t` as the former is the type for the home directory while the latter is for its contents); hence, the call to `userdom_search_user_home_dirs`.

Then, the necessary privileges are assigned to the domain. As we do not provide any class identifier in the interface name, the interface will grant read access to all (significant) classes related to the `skype_home_t` type.

If we only want to grant read access to files (and not to the `directory` class), then the interface would be called `skype_read_home_files`.

# Adding conditional policy rules

We can further fine-tune our policy with conditionals. Some of the access vectors identified earlier might not be necessary in all circumstances, so it makes sense to make them optional and configurable through SELinux Booleans.

Two of the identified access vectors that are candidates for configurable policies are as follows:

*   Accessing the video and sound devices (in order to reduce the risk of malware or vulnerabilities in the application to access the webcam or sound device and spy on the unsuspecting users)
*   Accessing all user content (instead of only the `skype_home_t` content)

## How to do it…

The following set of steps allows us to make the policy more flexible for the administrators to handle by introducing Booleans. These Booleans modify the behavior of the policy and are added to a policy.

1.  Inside `myskype.te`, create the definitions for both Booleans. This is usually done before the type declarations:

    ```
    gen_tunable(skype_use_audio, false)
    gen_tunable(skype_use_video, false)
    gen_tunable(skype_manage_user_content, false)
    ```

2.  Inside the policy, group the statements that we want to trigger through the Booleans:

    ```
    tunable_policy(`skype_use_audio',`
      dev_read_sound(skype_t)
      dev_write_sound(skype_t)
    ')
    tunable_policy(`skype_use_video',`
      dev_read_video_dev(skype_t)
      dev_write_video_dev(skype_t)
    ')
    tunable_policy(`skype_manage_user_content',`
      userdom_manage_user_home_content_dirs(skype_t)
      userdom_manage_user_home_content_files(skype_t)
    ')
    ```

## How it works…

The `gen_tunable` declarations will generate Booleans that administrators can toggle on the system. The first argument of each declaration is the name of the Boolean to be created, while the second argument sets the default value of the Boolean.

Once Booleans are defined, the `tunable_policy` statements allow for grouping the statement calls that need to be made configurable.

It is possible to have rules enabled when a Boolean is disabled as well. For instance, for the `skype_manage_user_content` one, the following code can be used:

```
tunable_policy(`skype_manage_user_content',`
  # boolean enabled
  userdom_manage_user_home_content_dirs(skype_t)
  userdom_manage_user_home_content_files(skype_t)
  ',`
  # boolean disabled
  userdom_dontaudit_manage_user_home_content_dirs(skype_t)
  userdom_dontaudit_read_user_home_content_files(skype_t)
  …
')
```

Booleans can also be combined, as shown in the following code:

```
tunable_policy(`use_nfs_home_dirs && skype_manage_user_content',` … ')
```

In such situations, the policy group rules will only take effect if both the Booleans are enabled.

It is also possible to only enable rules if a Boolean is not set, as shown in the next line of code:

```
tunable_policy(`!use_nfs_home_dirs',` … ')
```

## There's more...

Tunable policies are a powerful extension to SELinux. However, there are some caveats to this:

*   It is not simple to make the description of SELinux Booleans available to the administrator. The descriptions are defined through in-policy comments, but this is not used for custom modules—a full policy build needs to be made in order to generate the `policy.xml` file that contains all descriptions.
*   It is not possible to assign attributes within a `tunable_policy` group. Instead, policy developers will need to make the permissions related to the attribute configurable (if possible) or not assign the attribute at all.
*   It is not possible to use named file transitions within a `tunable_policy` group. In general, that doesn't matter that much—there are a few situations where a named file transition would depend on a Boolean, but these situations do occur.
*   It is not possible to have the `optional_policy` statements within a `tunable_policy` group. Instead, wrap the `tunable_policy` call with an `optional_policy` statement first. It might be needed to create multiple blocks if a single Boolean would trigger multiple policy calls that warrant the use of an `optional_policy` block.

Efforts are being made to remove these shortcomings from the SELinux subsystem though.

# Adding build-time policy decisions

The last enhancement we might want to look at is build-time policy decisions. Unlike SELinux Booleans, these are policy blocks that are enabled (or disabled) based on build parameters. We have encountered a few of these in the past already, namely `enable_mcs`, `enable_mls` as well as distribution selection parameters, such as `distro_gentoo` or `distro_redhat`.

In this recipe, we will enable the `xdg_manage_downloads_home` call but only when the policy is built for a Gentoo system.

## How to do it…

Build-time decisions are added to the policy using the `ifdef` statements, as can be seen through the next set of steps:

1.  Open `myskype.te` and add in the following block of code:

    ```
    ifdef(`distro_gentoo',`
      xdg_manage_downloads_home(skype_t)
    ')
    ```

2.  Rebuild the policy. On a Gentoo system, we can confirm that the access is now granted through `sesearch`, whereas other distributions probably don't even know the `xdg_downloads_home_t` type:

    ```
    ~$ sesearch -s skype_t -t xdg_downloads_home_t -A

    ```

## How it works…

The reference policy build system automatically defines a couple of parameters that can be used by the `ifdef` macros. The build system uses definitions inside the `build.conf` file available at `/usr/share/selinux/mcs/include/` or `/usr/share/selinux/devel/include/` to generate such parameters.

For instance, the distribution parameter in `build.conf` is set as follows:

```
DISTRO ?= gentoo
```

Inside `Makefile`, this is converted into an `M4PARAM` setting:

```
ifneq ($(DISTRO),)
        M4PARAM += -D distro_$(DISTRO)
endif
```

Through these `M4` parameters, we can then use the `ifdef` statements to query the existence of these parameters and make build-time decisions.

## There's more...

It is possible to add our own set of parameters. For this, we set the `M4PARAM` environment variable before we call the `make` command (used while building the policy modules).

For instance, to support the `debug` statements, we could set the following in the policy:

```
ifdef(`debug',` … ')
```

During policy build, we can enable these statements as follows:

```
~$ export M4PARAM="-D debug"
~$ make mypolicy.pp

```