# Chapter 1. The SELinux Development Environment

This chapter covers the setup of the SELinux policy development environment. We will cover the following topics in this chapter:

*   Creating the development environment
*   Building a simple SELinux module
*   Calling refpolicy interfaces
*   Creating our own interface
*   Using the refpolicy naming convention
*   Distributing SELinux policy modules

# Introduction

Similar to any other software development, having a well-functioning development environment is essential to successfully create and manage SELinux policies. Such an environment should not only support version control, but also be able to quickly search through the sources or show definitions.

With SELinux, this means that the policy module sources (which are all readable text files) should be stored in a structured manner, the upstream project that provides SELinux policies should be readily accessible, and the necessary functions or scripts to query and search through the policies should be available.

Adventurous users might want to take a look at the **SELinux Policy IDE** (**SLIDE**) as offered by Tresys Technology ([http://oss.tresys.com/projects/slide](http://oss.tresys.com/projects/slide)). In this book, we do not focus on this IDE; instead, we use whatever file editor the user wants (such as VIM, Emacs, or Kate) and enhance the environment through the necessary shell functions and commands.

Before we cover the setup of the development environment, let's do a quick recapitulation of what SELinux is.

## About SELinux

The **Security Enhanced Linux** (**SELinux**) project is the result of projects initiated and supported by the US **National Security Agency** (**NSA**) and came to life in December 2000\. It is the implementation of a security system architecture with a flexible, policy-driven configuration approach. This architecture is called the **Flux Advanced Security Kernel** (**Flask**), and its related resources are still an important read for everyone involved with SELinux.

Most papers are linked through the Flask website at [http://www.cs.utah.edu/flux/fluke/html/flask.html](http://www.cs.utah.edu/flux/fluke/html/flask.html). The following are some examples of these papers:

*   The paper called *The Inevitability of Failure: The Flawed Assumption of Security in Modern Computing Environments* is still a very topical paper on why mandatory access controls are needed in operating systems
*   The NSA publication, *Implementing SELinux as a Linux Security Module*, available at [http://www.nsa.gov/research/_files/publications/implementing_selinux.pdf](http://www.nsa.gov/research/_files/publications/implementing_selinux.pdf), goes deeper into how SELinux is implemented

Nowadays, SELinux can be best seen as an additional layer of security services on top of the existing Linux operating system. It is part of the mainstream Linux kernel and uses the **Linux Security Modules** (**LSM**) interfaces to hook into the interaction between processes (user space) and resources. It manages various access services (such as the reading of files, getting directory attributes, binding to domain sockets, connecting to TCP sockets, and gaining additional capabilities) through a powerful approach called **type enforcement**.

The following diagram displays the high-level functional position of the SELinux subsystem. Whenever a subject (in the drawing, this is the **Application**) wants to perform an action against a resource, this action is first checked by the Discretionary Access Control mechanism that the Linux kernel provides. After the action is checked and allowed by the DAC mechanism, the LSM implementation (against which SELinux is registered) calls the hooks that the SELinux subsystem has provided. SELinux then checks the policy (through the cache, and if the access is not registered in the cache yet, it checks in the entire policy) and returns whether the access should be allowed or not.

![About SELinux](img/9669OS_01_01.jpg)

SELinux is a Mandatory Access Control system in which the governed activities on the system are defined in rules that are documented through a policy. This policy is applicable to all processes of the system and enforced through the SELinux subsystem, which is part of the Linux kernel. Anything that is not allowed by the policy will not be allowed at all—security is not left at the discretion of the user or correctness of the application. Unlike Linux DAC restrictions, enforcement itself (the SELinux code) is separate from the rules (the SELinux policy). The rules document what should be considered as acceptable behavior on the system. Actions that do not fit the policy will be denied by the SELinux subsystem.

In SELinux, a set of access control mechanisms are supported. The most visible one is its type enforcement in which privileges of a subject (be it the kernel or a Linux process) towards an object (such as a file, device, system capability, or security control) are granted based on the current security context of that subject. This security context is most often represented through a readable string such as `staff_u:staff_r:staff_t:s0:c0,c3`. This string represents the SELinux user (`staff_u`), SELinux role (`staff_r`), SELinux type/domain (`staff_t`), and optionally, the SELinux sensitivity level or security clearance, which provides both the sensitivity (`s0`) as well as assigned categories (`c0,c3`).

Alongside type enforcement, SELinux has several other features as well. For instance, it provides a **role-based access control** system by assigning valid domains (which are SELinux types assigned to running processes) to roles. If a role is not granted a particular domain, then that role cannot execute tasks or applications associated with that domain. SELinux also supports user-based access controls, thus limiting information flow and governing data sharing between SELinux users.

Another stronghold within SELinux is its support for sensitivities (which SELinux displays as integers, but these integers can very well be interpreted as public, internal, confidential, and so on) as well as access categories. Through the constraints that SELinux can impose in its policy, systems can be made to largely abide by the Bell-LaPadula model ([https://en.wikipedia.org/wiki/Bell-LaPadula_model](https://en.wikipedia.org/wiki/Bell-LaPadula_model)). This model supports information flow restrictions such as no read up (lower sensitivities cannot read information from higher sensitivities) and no write down (higher sensitivities cannot leak information to lower sensitivities).

## The role of the SELinux policy

The SELinux policy itself is a representation of what the security administrator (the role that is usually mentioned as being the owner of what is and isn't allowed on a system) finds acceptable, expected, and normalized behavior:

*   **Acceptable**: Application and user behavior will be acceptable because it will be allowed on the system by the policy
*   **Expected**: Application and user behavior will be expected as the policy usually doesn't (or shouldn't) contain access vectors (a permission assigned to a subject towards a particular object) that are not applicable to the system, even if it would be acceptable on other systems in the environment
*   **Normalized**: Application and user behavior will be normalized not in the sense of database normalization, but as in normality—something that is consistent with the most common behavior of the process

As a policy represents these behaviors, correct tuning and development of the policy is extremely important. This is why the *SELinux Cookbook* will focus on policy development and policy principles.

A policy that is too restrictive will cause applications to malfunction, often in ways that its users will find unexpected. It will not be surprising to the security administrator of course, as he knows that the policy dictates what is allowed, and he is (or at least should be) perfectly aware of what the policy says.

However, a policy that is too broad will not result in such behavior. On the contrary, everything will work as expected. Sadly, when nonstandard or abnormal behavior is triggered, the (too) broadly defined policy might still allow this nonstandard or abnormal behavior to happen. When this abnormal behavior is an exploited vulnerability, then SELinux—as powerful as it can be—has nothing to deter the exploit, as the policy itself has granted the access. Another example of this principle would be a network firewall, whose policy can be too open as well.

Through the packaged approach that policies provide (SELinux policies are like loadable kernel modules, but then for the SELinux subsystem), administrators can push the policies to one or more systems, usually through the package management system or centralized configuration management system of choice. Unlike Linux DAC controls, which need to be applied on the files themselves, SELinux policies are much easier to handle and are even versionable—a trait much appreciated by administrators in larger environments.

## The example

Throughout this book, we will often come across settings that are optional or whose value is heavily dependent on the choices made by the system administrator. In order to not repeat documenting and explaining when a setting or value is configurable, we will use the following configuration settings:

*   The SELinux type (which is configured in `/etc/selinux/config`) will be MCS in this book. It uses an MLS-enabled, single-sensitivity policy definition. This means that contexts will always have a sensitivity level or security clearance assigned when displayed, and the location of the SELinux policy configuration will always be shown in `/etc/selinux/mcs/`. Make sure to substitute this path with your own if the policy store on your environment is named differently. For instance, a Red Hat or Fedora installation defaults to `/etc/selinux/targeted/`.
*   Operations will be documented as they run through restricted users, which are aptly named `user` (for an unprivileged end user assigned the `user_r` role), `staff` (for a user that might perform administrative tasks and is assigned the `staff_r` and `sysadm_r` roles), and `root` (which is mapped to the `sysadm_r` role). Distributions might have users associated with the `unconfined_r` role. Whenever a step can be skipped for unconfined users, it will be explicitly mentioned.

# Creating the development environment

We will now create a development environment in which the SELinux policies and upstream project code as well as the functions we use to easily query the SELinux policies will be stored. This environment will have the following three top-level locations:

*   `local/`: This location contains the SELinux rules that are local to the system and not part of a cooperatively developed repository (that is, where other developers work)
*   `centralized/`: This location contains checkouts of the various repositories used in the development environment
*   `bin/`: This location contains the supporting script(s) we will use to query the policy sources as well as troubleshoot the SELinux issues

In this exercise, we will also populate the `centralized/` location with a checkout: the SELinux policy tree that is used by the current system.

## Getting ready

Look for a good location where the development environment should be stored. This usually is a location in the user's home directory, such as `/home/staff/dev/selinux/`. Whenever this book references a location with respect to the development environment, it will use the `${DEVROOT}` variable to refer to this location.

Another piece of information that we need is the location of the repository that hosts the SELinux policies of the current system. This location is distribution specific, so consult the distribution site for more information. At the time of writing this book, the policies for Gentoo Linux and Fedora can be found at the following locations:

*   [https://github.com/sjvermeu/hardened-refpolicy](https://github.com/sjvermeu/hardened-refpolicy)
*   [https://git.fedorahosted.org/git/selinux-policy.git](https://git.fedorahosted.org/git/selinux-policy.git)

Whenever version control is used, we will use `git` in this book. Other version control systems exist as well, but this too is outside the scope of this book.

## How to do it…

To create the development environment used in this book, perform the following steps:

1.  Create the necessary directories:

    ```
    ~$ cd ${DEVROOT}
    ~$ mkdir local centralized bin

    ```

2.  Create a checkout of the distributions' SELinux policy tree (which, in this example, is based on the Gentoo Linux repository):

    ```
    ~$ cd ${DEVROOT}/centralized
    ~$ git clone git://git.overlays.gentoo.org/proj/hardened-refpolicy.git

    ```

3.  Create a `git` repository for the policies that we will develop throughout this book:

    ```
    ~$ cd ${DEVROOT}/local
    ~$ git init

    ```

4.  Add the `functions.sh` script, which is available through the download pack of this book, to the `${DEVROOT}/bin/` location.
5.  Edit the `.profile`, `.bashrc`, or other shell configuration files that are sourced when our Linux user logs on to the system, and add in the following code:

    ```
    # Substitute /home/staff/dev/selinux with your DEVROOT
    DEVROOT=/home/staff/dev/selinux
    # Substitute the next location with your distributions' policy checkout
    POLICY_LOCATION=${DEVROOT}/centralized/hardened-refpolicy/
    source ${DEVROOT}/bin/functions.sh
    ```

6.  Log out and log in again, and verify that the environment is working by requesting the definition of the `files_read_etc_files` interface:

    ```
    ~$ seshowif files_read_etc_files
    interface(`files_read_etc_files',`
     gen_require(`
     type etc_t;
     ')

     allow $1 etc_t:dir list_dir_perms;
     read_files_pattern($1, etc_t, etc_t)
     read_lnk_files_pattern($1, etc_t, etc_t)
    ')

    ```

## How it works…

The setup of the development environment helps us deal with policy development for further recipes. The checkout of the distributions' SELinux policy tree is to query the existing policy rules while developing new policies. SELinux does not require to have the policy sources available on a system (only the compiled binary SELinux policy modules and parts of the SELinux policy rules, which can be used by other policy modules). As a result, default installations usually do not have the policy rules available on the system.

By having the checkout at our disposal, we can review the existing SELinux policy files and happily use examples from it for our own use. A major user of this checkout is the `functions.sh` script, which uses the `${POLICY_LOCATION}` variable to know where the checkout is hosted. This script provides several functions that we'll use throughout this book; they will also help in querying the sources.

By sourcing this `functions.sh` script during log on, these functions are readily available in the user's shell. One of these functions is the `seshowif` function, which displays the rules of a particular interface. The example given shows the definition of the `files_read_etc_files` interface, which we used to validate that our setup is working correctly.

The `functions.sh` script can also work with the interface files that are available through `/usr/share/selinux/devel/` (on Fedora/Red Hat systems), making the checkout of the policy repository optional if the user does not need access to the complete policy rules. The policy location defined then is as follows:

```
export POLICY_LOCATION=/usr/share/selinux/devel/
```

## There's more...

Next to the distributions' SELinux policy tree, one can also use the reference policy SELinux tree. This is the upstream project that most, if not all, Linux distributions use as the source of their policies. It has to be said though that the reference policy often lags behind on the policy repositories of the individual distributions, as it has stricter acceptance criteria for policy enhancements.

It also doesn't hurt to check out the SELinux policy repositories of other distributions. Often, Linux distributions first do SELinux policy updates on their own repository before pushing their changes to the reference policy (which is called upstreaming in the free software development community). By looking at other distributions' repositories, developers can easily see if the necessary changes have been made in the past already (and are thus more likely to be correct).

## See also

For more information about the topics covered in this recipe, refer to the following resources:

*   The reference policy project ([http://oss.tresys.com/projects/refpolicy/](http://oss.tresys.com/projects/refpolicy/))
*   The Git tutorial ([http://git-scm.com/docs/gittutorial](http://git-scm.com/docs/gittutorial))

# Building a simple SELinux module

Now that we have a development environment, it is time to create our first SELinux policy module. As its purpose does not matter at this point, we will focus on a privilege that is by default not allowed (and rightfully so) yet easy to verify, as we want to make sure that our policy development approach works. The privilege we are going to grant is to allow the system logger to write to a logfile labeled `named_conf_t` (the type used for the configuration of the BIND DNS server—known as `named`).

### Note

Building SELinux policy modules is to extend the existing policy with more rules that allow certain accesses. It is not possible to create a policy module that reduces the allowed privileges for a domain. If this is needed, the policy module needs to recreate and substitute the existing policy (and thus, a distribution-provided policy will need to be removed from the system).

## Getting ready

Before we get started, we first need to make sure that we can test the outcome of the change. A simple method would be to change the context of the `/var/log/messages` file (or another general logfile that the system logger is configured to use) and send messages through the system logger using the `logger` command:

```
~$ logger "Just a simple log event"
~$ tail /var/log/messages

```

Verify that the message has indeed been delivered by looking at the last lines shown through the `tail` command. Then, change the context and try again. The event should not be shown, and a denial should be logged by the audit daemon:

```
~# chcon -t named_conf_t /var/log/messages
~$ logger "Another simple log event"
~$ tail /var/log/messages
~# ausearch -m avc -ts recent

```

With this in place, we can now create our first simple SELinux module.

## How to do it…

Building a new SELinux policy is a matter of going through the following steps:

1.  Create a file called `mylogging.te` inside `${DEVROOT}/local` with the following content:

    ```
    policy_module(mylogging, 0.1)
    gen_require(`
      type syslogd_t;
      type named_conf_t;
     ')
    # Allow writing to named_conf_t files
    allow syslogd_t named_conf_t:file { getattr append lock ioctl open write };
    ```

2.  Copy or link the `Makefile` file available in `/usr/share/selinux/devel/` or `/usr/share/selinux/mcs/include/` (the exact location is distribution specific) to the current directory:

    ```
    ~$ ln –s /usr/share/selinux/devel/Makefile

    ```

3.  Build the SELinux policy module through this `Makefile`. The target is to name the (target) policy module with the `.pp` suffix:

    ```
    ~$ make mylogging.pp

    ```

4.  Switch to the root user, and if we are logged on through an unprivileged SELinux domain/role, switch to the `sysadm_r` or `secadm_r` role (this is not needed if the current user domain is already `sysadm_t` or `unconfined_t`):

    ```
    ~$ sudo –r sysadm_r –t sysadm_t -s

    ```

5.  Now, load the SELinux policy module (which will immediately activate the newly defined SELinux policy):

    ```
    ~# semodule –i mylogging.pp

    ```

6.  Verify that the newly defined SELinux policy is active by generating a new log event and by looking at the logfile to see if it has indeed been registered.
7.  Commit the newly created files to the repository:

    ```
    ~$ cd ${DEVROOT}/local
    ~$ git add mylogging.te Makefile
    ~$ git commit –m 'Adding mylogging.te which allows the system logger to write to the named configuration file type named_conf_t'

    ```

    When verified, reset the context of the logfile using `restorecon /var/log/messages` and remove the policy module from the subsystem using `semodule -r mylogging`. After all, we do not want this rule to stay active.

## How it works…

There are three important, new aspects of SELinux policy development that we got in touch with in this recipe:

*   A policy source file called `mylogging.te` was created
*   A generated, binary policy module called `mylogging.pp` was built
*   The binary policy file, `mylogging.pp`, is added to the active policy store on the system

At the end, we committed the file to our local repository. Using version control on policy files is recommended in order to track changes across time. A good hint would be to tag new releases of the policies—if users ever report issues with the policy, you can then ask them for the SELinux policy module version (through `semodule –l`) and use the tags in the repository to easily find rules for that particular policy module.

In the remainder of this book, we will no longer use `git add`/`commit` so that we can focus on the SELinux recipes.

### The policy source file

In the recipe, we created a policy source file called `mylogging.te`, which contains the raw SELinux policy rules. The name, `mylogging`, is not chosen at random; it is a common best practice to name custom modules starting with `my` and followed by the name of the SELinux policy module whose content we are enhancing (in our case, the logging module that provides the SELinux policy for everything that is system-logging related). The `.te` suffix is not just a convention (referring to the type enforcement part of SELinux); the build system requires the `.te` suffix.

The policy module rules start with the `policy_module(…)` call, which tells the build system that the file will become a loadable SELinux policy module with the given name and version. This name and version will be displayed by the `semodule` command if we ask it to list all the currently loaded SELinux policy modules:

```
~# semodule –l
aide  1.8.0
alsa  1.13.0
…
mylogging  0.1
…

```

The best practice is to keep all rules for a single domain within a policy module. If rules for multiple unrelated domains are needed, it is recommended that you create multiple modules, as this isolates the policy rules and makes modifications more manageable.

In this simple example, we did not follow this best practice (yet). Instead, we told the SELinux subsystem that the policy is going to be enhanced with an access vector for `syslogd_t`. The access vector here is to allow this domain a set of permissions against files that are labeled as `named_conf_t`.

### The binary policy module

When we called the `Makefile` file, the underlying scripts built a loadable binary SELinux policy module. Such files have the `.pp` suffix and are ready to be loaded into the policy store. The `Makefile` file called might not be installed by default; some distributions require specific development packages to be installed (such as `selinux-policy-devel` in Fedora).

There is no nice way of retrieving the sources of a policy if we are only given the `.pp` file. Sure, there are commands such as `semodule_unpackage` and `sedismod` available, but these will only give a low-level view of the rules, not the original `.te` code. So, make sure to have backups, and as we saw in the example, use a versioning system to control changes across time.

### Loading a policy into the policy store

To load the newly created policy into the policy store, we called the `semodule` command. This application is responsible for managing the policy store (which is the set of all SELinux policy modules together with the base policy module) and linking or unlinking the modules together into a final policy.

This final policy (which can be found at `/etc/selinux/mcs/policy`) is then loaded into the SELinux subsystem itself and enforced.

## There's more...

Throughout this book, the build system used is based on the reference policy build system. This is a collection of M4 macros and affiliated scripts that make the development of SELinux policies easier. This is, however, not the only way of creating SELinux policies.

When visiting online resources, you might come across SELinux policy modules whose structure looks like the following:

```
module somename 1.0;
require {
  type some_type_t;
  type another_type_t;
}
allow some_type_t another_type_t:dir { read search };
```

### Tip

**Downloading the example code**

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

These are policy files that do not use the reference policy build system. To build such files, we first create an intermediate module file with `checkmodule`, after which we package the module file towards a loadable binary SELinux policy with `semodule_package`:

```
~$ checkmodule -M –m –o somename.mod somename.te
~$ semodule_package –m somename.mod –o somename.pp

```

To keep things simple, we will stick to the reference policy build system.

## See also

Many topics and areas have been touched upon in this recipe. More information can be found at the following resources:

*   Most Linux distributions have distribution-specific information on how SELinux is integrated in the distribution. For Red Hat, these resources can be reached through [https://access.redhat.com](https://access.redhat.com). For Fedora, use [https://docs.fedoraproject.org](https://docs.fedoraproject.org). Gentoo has its documentation at [https://wiki.gentoo.org](https://wiki.gentoo.org).
*   For more information on how to administer SELinux on a system, consult the documentation provided by the distribution or read the *SELinux System Administration* book published by Packt Publishing at [http://www.packtpub.com/selinux-system-administration/book](http://www.packtpub.com/selinux-system-administration/book).
*   Extensive coverage of the SELinux language itself is provided by the SELinux Notebook, which is available online at [http://www.freetechbooks.com/the-selinux-notebook-the-foundations-t785.html](http://www.freetechbooks.com/the-selinux-notebook-the-foundations-t785.html).

# Calling refpolicy interfaces

Writing up SELinux policies entirely using the standard language constructs offered by SELinux is doable, but it is prone to error and not manageable in the long term. To support a more simple language construct, the reference policy project uses a set of M4 macros that are expanded with the underlying SELinux policy statements when the policy is built.

The API that policy developers can use can be consulted online, but most systems also have this information available onsite at `/usr/share/doc/selinux-*/`. Finding proper interfaces requires some practice though, which is why one of the functions that we installed earlier (as part of the development environment) simplifies this for us.

In this recipe, we are going to edit the `mylogging.te` file we generated earlier with the right reference policy macro.

## How to do it…

To use reference policy interfaces in an SELinux policy module, the following approach can be taken:

1.  Use the `sefinddef` function to find permission groups or patterns to write to files:

    ```
    ~$ sefinddef 'file.*write'
    define(`write_files_pattern',`
     allow $1 $3:file write_file_perms;
    …
    define(`write_file_perms',`{ getattr write append lock ioctl open }')
    …

    ```

2.  Use the `seshowdef` function to show the full nature of the `write_files_pattern` definition:

    ```
    ~$ seshowdef write_files_pattern
    define(`write_files_pattern',`
     allow $1 $2:dir search_dir_perms;
     allow $1 $3:file write_file_perms;
    ')

    ```

3.  Use the `sefindif` function to find the interface that will allow writing to `named_conf_t`:

    ```
    ~$ sefindif 'write_files_pattern.*named_conf_t'
    contrib/bind.if: interface(`bind_write_config',`
    contrib/bind.if:   write_files_pattern($1, named_conf_t, named_conf_t)

    ```

4.  Now, update the `mylogging.te` file to use this function. The file should now look like the following:

    ```
    policy_module(mylogging, 0.2)
    gen_require(`
      type syslogd_t;
    ')
    bind_write_config(syslogd_t)
    ```

    ### Note

    Note the use of the backtick (`` ` ``) and single quote (`'`). Whenever definitions are used, they need to start with a backtick and end with a single quote.

5.  Rebuild and reload the policy module, and then rerun the tests we did earlier to verify that this still allows us to write to the `named_conf_t labeled` file.

## How it works…

One of the principles behind the build system of the reference policy is that an SELinux policy module should not directly mention SELinux types that are not related to that module. Whenever a policy module needs to define rules against a type that is defined by a different module, interfaces defined by that different module should be used instead.

In our example, we need the interface used by the BIND SELinux policy (which handles the BIND-named daemon policy rules); this interface allows us to write to the BIND DNS server configuration file type (`named_conf_t`). We can check the online API, the API documentation in `/usr/share/doc/selinux-*`, or just guess the interface name. However, in order to be certain that the interface does what we need, we need to query the interface definitions themselves.

That is where the `sefinddef`, `seshowdef`, `sefindif`, and `seshowif` functions come into play. These functions are not part of any SELinux user space—they are provided through the `functions.sh` file we installed earlier and are simple `awk`/`grep`/`sed` combinations against the SELinux policy files.

With `sefinddef` (the SELinux find definition), we can search through the support macros (not related to a particular SELinux policy module) for any definition that matches the expression given to it. In this recipe, we gave `file.*write` as the expression to look for. The `seshowdef` (SELinux show definition) function shows us the entire definition of the given pattern.

The `sefindif` (SELinux find interface) function then allows us to find an interface that the SELinux policy provides. In this recipe, we used it to search for the interface that allows a domain to write to the BIND DNS server configuration files. There is also a `seshowif` (SELinux show interface) function that shows us the entire interface definition like the following:

```
~$ seshowif bind_write_config
interface(`bind_write_config',`
 gen_require(`
 type named_conf_t;
 ')
 write_files_pattern($1, named_conf_t, named_conf_t)
 allow $1 named_conf_t:file setattr_file_perms;
')

```

This example interface nicely shows how interfaces are handled by the SELinux reference policy build system. Whenever such an interface is called, one or more arguments are given to the interface. In our case, we passed on `syslogd_t` as the first (and only) argument.

The build system then substitutes every `$1` occurrence in the interface with the first argument, effectively expanding the call to the following code:

```
write_files_pattern(syslogd_t, named_conf_t, named_conf_t)
allow syslogd_t named_conf_t:file setattr_file_perms;
```

The call to `write_files_pattern` is then expanded with the definition we saw earlier.

For the policy developer, this is all handled transparently. The sources of the SELinux policy file stay well-formatted and only call the interfaces. It is at build time that the expansion of the various interfaces is done. This allows developers to have nicely segregated, compartmentalized policy definitions.

## See also

*   The reference policy project can be found online at [http://oss.tresys.com/projects/refpolicy/](http://oss.tresys.com/projects/refpolicy/)

# Creating our own interface

Being able to call interfaces is nice, but when we develop SELinux policies, we will run into situations where we need to create our own interface for the SELinux module we are developing. This is done through a file with an `.if` extension.

In this recipe, we are going to extend the `mylogging` policy with an interface that allows other domains to execute the system log daemon binary (but without running this binary with the privileges of the system logger itself; this would be called a domain transition in SELinux).

## How to do it…

1.  If our current context is an unprivileged user domain (as unconfined domains are highly privileged and can do almost everything), we can try executing the system logger daemon (`syslog-ng` or `rsyslog`) directly and have it fail as follows:

    ```
    ~$ /usr/sbin/syslog-ng --help
    bash: /usr/sbin/syslog-ng: Permission denied

    ```

2.  Now, create the `mylogging.if` file (in the same location where `mylogging.te` is) with the following content, granting all permissions needed to execute the binary:

    ```
    ## <summary>Local adaptation to the system logging SELinux policy</summary>

    ##########################################
    ## <summary>
    ##    Execute the system logging daemon in the caller domain
    ## </summary>
    ## <desc>
    ##   <p>
    ##     This does not include a transition.
    ##   </p>
    ## </desc>
    ## <param name="domain">
    ##    <summary>
    ##      Domain allowed access.
    ##    </summary>
    ## </param>
    #
    interface(`logging_exec_syslog',`
      gen_require(`
        type syslogd_exec_t;
      ')
      can_exec($1, syslogd_exec_t)
    ')
    ```

3.  Create a new SELinux policy module for the user domain; this policy should be able to execute the system logger directly. For instance, for the `sysadm_t` domain, we would create a `mysysadm.te` file with the following content:

    ```
    policy_module(mysysadm, 0.1)
    gen_require(`
      type sysadm_t;
    ')
    logging_exec_syslog(sysadm_t)
    ```

4.  Build the `mysysadm` policy module and load it. Then, test to see if the daemon binary can now be executed directly:

    ```
    ~$ /usr/sbin/syslog-ng --help

    ```

## How it works…

Let's first look at how the build system knows where the interface definitions are. Then, we'll cover the in-line comment system used in the example.

### The location of the interface definitions

Whenever an SELinux policy module is built, the build system sources all interface files it finds at the following locations:

*   `/usr/share/selinux/mcs/include/*` or `/usr/share/selinux/devel/include/*` (depending on the Linux distribution)
*   The current working directory

The first location is where the interface files of all the SELinux modules provided by the Linux distribution are stored. The files are inside subdirectories named after particular categories (the reference policy calls these layers, but this is only used to make some structure amongst the definitions, nothing else) such as `contrib/`, `system/`, and `roles/`.

For local development of SELinux policies, this location is usually not writable. If we develop our own policy modules, then this would mean that none of the locally managed SELinux policy files can use interfaces of the other local interface files. The `Makefile` file, therefore, also sources all interface files it finds in the current working directory.

### The in-line documentation

Inside the interface file created, we notice a few XML-like structures as comments. These comments are prefixed by a double hash sign (`##`) and are used by the reference policy build system to generate the API documentation (which can be found at `/usr/share/doc/selinux-*`).

For local policies, this in-line documentation is not used and, thus, not mandatory. However, writing the documentation even for local policies helps us in documenting the rules better. Also, if we ever want to push our changes upstream, this in-line documentation will be requested anyway.

The comment system uses the following constructs:

*   Right before an interface definition, we encounter a `<summary>` element, which provides a one-sentence description of the interface
*   Additional information can then be provided through a `<desc>` element under which the HTML code can be placed for documenting the interface further
*   Every parameter to an interface is documented through a `<param>` entity, which again contains a `<summary>` line

## See also

*   The reference policy API documentation can be found online at [http://oss.tresys.com/docs/refpolicy/api/](http://oss.tresys.com/docs/refpolicy/api/)

# Using the refpolicy naming convention

The interface names used to simplify policy development can be freely chosen. However, the reference policy itself uses a naming convention to try and structure the names used so that the SELinux policy developers can easily find the interfaces they need—if they exist—and give an unambiguous name to an interface they want to create.

The naming convention for the reference policy is available online at [http://oss.tresys.com/projects/refpolicy/wiki/InterfaceNaming](http://oss.tresys.com/projects/refpolicy/wiki/InterfaceNaming).

## Getting ready

In this recipe, we'll do a pen-and-paper exercise to see how the naming convention works. In the example, we will create interface names for three situations:

*   To read all logfiles
*   To connect to the HTTP port over TCP
*   To not audit getting the attributes of user home directories

## How to do it…

1.  First we need to figure out the file types that are involved in the situations:

    *   Generic logfiles are `var_log_t` (as can be seen by querying the label of `/var/log/`itself):

        ```
        ~$ ls -dZ /var/log
        drwxr-xr-x. root root system_u:object_r:var_log_t:s0 /var/log

        ```

    *   When we deal with all logfiles, we can safely assume this is handled by an SELinux attribute. Let's look at the attributes for the generic `var_log_t` type:

        ```
        ~$ seinfo –tvar_log_t –x
         var_log_t
         file_type
         non_security_file_type
         mountpoint
         non_auth_file_type
         logfile

        ```

    *   The `logfile` attribute looks like an interesting hit. We can now grep through the policy sources to figure out which SELinux policy modules handle the `logfile` attribute, or use `sefindif` (assuming that there are interfaces defined that handle this attribute):

        ```
        ~$ sefindif 'attribute logfile'
        system/logging.if: interface(`logging_log_file',`
        …

        ```

    *   For the logfiles example, the module we need is called `logging` as can be seen from the `sefindif` output. Similarly, we will find that for the HTTP port, the module is `corenet`, and home directories are `userdom`.

2.  Next, we check whether there is a modifier. The first two situations have no specific modifier (all the actions are regular verbs). The last example has one: do not audit. In the SELinux policy language, this is known as a `dontaudit` statement.
3.  Now, let's look at the verbs involved. This is mostly based on experience, but the situations show that there is a huge correlation between the verbs and the eventually chosen `refpolicy` name (which usually uses SELinux permission names):

    *   In the first situation, this is `read`
    *   The second one has `connect over TCP`, which is translated into `tcp_connect`
    *   The last situation has `getting the attributes`, so it is translated as `getattr`

4.  Finally, let's look at the object that is being referenced:

    *   In the first situation, this is `all logfiles`, which we will name `all_logs`
    *   In the second situation, this is `HTTP port`, so we will name `http_port`
    *   The third situation has `user home directories`, so we will name `user_home_dirs`

5.  Combining this gives us the following interface names:

    *   **Read all logfiles**: `logging_read_all_logs`
    *   **Connect to the HTTP port over TCP**: `corenet_tcp_connect_http_port`
    *   **Do not audit getting the attributes of user home directories**: `userdom_dontaudit_getattr_user_home_dirs`

## How it works…

The naming convention that the reference policy uses is not mandated in a technical manner. Just like with coding styles, naming conventions are made so that collaboration is easier (everyone uses the same naming convention) and searching through the large set of interfaces can be directed more efficiently.

Using the proper naming convention is a matter of exercise. If uncertain, ask around in `#selinux` on `irc://irc.freenode.net` or on the reference policy mailing list.

## There's more...

Take some time to look through the interface files available at `/usr/share/selinux/devel/include/`. Next, for the more standard permission-based interface names, there are also interface names used for templates and type assignation.

For instance, there is a template called `apache_content_template`. Through it, additional SELinux types and permissions (used for web applications) are created in one go. Similarly, there is an interface called `apache_cgi_domain` that marks a particular type as being a domain that can be invoked through a web servers' CGI support.

Besides the naming convention, the reference policy also has a style guide available at [http://oss.tresys.com/projects/refpolicy/wiki/StyleGuide](http://oss.tresys.com/projects/refpolicy/wiki/StyleGuide). Like the naming convention, this is purely a human aspect for improved collaboration—there is no consequence of violating the coding style beyond the changes that might not be accepted in the upstream repositories.

# Distributing SELinux policy modules

We finish this chapter by explaining how SELinux policy modules can be distributed across multiple systems.

## How to do it…

To distribute SELinux policies, complete the following steps:

1.  Take into account the different system configurations to which the SELinux policies need to be distributed:

    *   If multiple systems have different SELinux policy releases to be active, then build the SELinux policy module against each of these implementations. This is heavily distribution specific. For instance, on Gentoo, this is the version of the `sec-policy/selinux-base` package. On Red Hat and derived distributions, this is the version of the `selinux-policy` package.
    *   If multiple SELinux policy types are active (such as `mcs`, `targeted`, and `strict`) and there are both MLS-enabled as well as MLS-disabled policies, then the SELinux policy module will need to be built against both an MLS-enabled policy as well as an MLS-disabled policy. The output of `sestatus` will tell us whether MLS is enabled on an active policy or not:

        ```
        ~$ sestatus | grep MLS
        Policy MLS status:    enabled

        ```

2.  Package the resulting `.pp` files and distribute them to the various systems. It is a common best practice to place the `.pp` files inside `/usr/share/selinux/mcs/` (this is for an SELinux policy store named `mcs`, you can adjust it where needed).
3.  On each system, make sure that the `.pp` file is loaded through `semodule –I policyfile.pp`.

## How it works…

SELinux policy modules (the files ending with `.pp`) contain everything SELinux needs to activate the policy. By distributing these files across many systems (and loading it through the `semodule` command), these systems receive the wanted updates against their current SELinux policy.

Once loaded (and this only needs to happen once, as a loaded module is retained even after the system reboots), one does not really need the `.pp` files anymore (loaded modules are copied inside `/etc/selinux`). However, it is recommended that you keep the policies there so that administrators can reload policies as needed; this might help in troubleshooting the SELinux policy and system permission issues.

There are a few caveats to take into account though:

*   Changes in interfaces
*   Kernel version changes
*   MLS-enabled or MLS-disabled policies

### Changes in interfaces

The `.pp` files contain all rules that SELinux needs to enforce the additional policy rules. This includes the (expanded) rules that were part of the interface definition files (the `.if` files) of the module itself as well as the interfaces referred to by the policy module.

When an update against an interface occurs, then all SELinux policy modules that might be affected by the change need to be rebuilt. As there is no simple way to know if a module needs to be rebuilt or not, it is recommended that you rebuild all policy modules every time a change has occurred to at least one interface.

Distributions will handle the rebuilding of the policies and the distribution of the rebuilt policies themselves, but for custom policy modules, we need to do this ourselves.

### Kernel version changes

New kernel releases might include updates against the SELinux subsystem. When these updates provide additional features, the binary representation of a policy might be updated. This is then reflected in the binary version of the policy that the kernel supports.

Binary versions are backward compatible, so a system that supports a maximum version of `28` (SELinux's binary versions are integers that are incremented with every change) will also support loading policy modules of a lower binary version:

```
~# sestatus
SELinux status:      enabled
SELinuxfs mount:      /sys/fs/selinux
SELinux root directory:    /etc/selinux
Loaded policy name:    mcs
Current mode:      enforcing
Mode from config file:    enforcing
Policy MLS status:      enabled
Policy deny_unknown status:  denied
Max kernel policy version:  28

```

### Note

When the binary version of an SELinux policy module is higher than the maximum kernel policy version, this SELinux policy module will not load on the target system. A higher version means that the policy uses features that are only available in kernels that support this version, so the administrator will need to update the kernels on those systems to support the higher version or update the SELinux policy module to not use these features so that a rebuild creates a lower-versioned binary SELinux policy module.

### MLS or not

SELinux policy modules might contain sensitivity-related information. When a policy module is built, information is added to reflect whether it is built against an MLS-enabled system or not.

Therefore, if we have hosts that have diverse policy usages (some policy stores are MLS-enabled and some are MLS-disabled), then the SELinux policy module will need to be built against both and distributed separately.

Usually, this is done by providing SELinux policy modules for each particular SELinux policy type (be it `mcs`, `strict`, or `targeted`).