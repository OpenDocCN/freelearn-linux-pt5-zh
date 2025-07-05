# Chapter 2. Dealing with File Labels

In this chapter, we will cover how file labels are set and managed, and learn how to configure the SELinux policy ourselves to use and assign the right file labels. The recipes that this chapter covers are as follows:

*   Defining file contexts through patterns
*   Using substitution definitions
*   Enhancing an SELinux policy with file transitions
*   Setting resource-sensitivity labels
*   Configuring sensitivity categories

# Introduction

Setting, resetting, and governing file labels are the most common tasks administrators have to perform on an SELinux-enabled system. The policies that are provided by policy developers as well as Linux distributions offer sane defaults to use, but many implementations harbor different locations for services and files. Companies often install their custom scripts and logfiles in nondefault locations, and many daemons can be configured to support multiple instances on the same system—each of them using a different base directory.

System administrators will know how to set context definitions through the `semanage` application and then reset the contexts of the target files using `setfiles` or `restorecon`:

```
~# semanage fcontext –a –t httpd_sys_content_t "/srv/web/zone/htdocs(/.*)?"
~# restorecon –RF /srv/web/zone/htdocs

```

This, however, is a local definition, which, if necessary, needs to be exported and imported in order to transfer it to other systems:

```
~# semanage export -f local_selinux.mods
~# semanage import -f local_selinux.mods

```

By moving context definitions into the SELinux policy realm, such definitions can be easily installed on multiple systems and managed centrally as we've seen for SELinux policy modules.

# Defining file contexts through patterns

SELinux policy modules can contain file context definitions through their `.fc` files. In these files, path expressions are used to point to the various locations that should match a particular file context, and class identifiers are used to differentiate file context definitions based on the file class (directories, regular files, symbolic links, and more).

In this recipe, we'll create a `mylogging` SELinux module, which defines additional path specifications for logging-related contexts. We will use direct file paths as well as regular expressions, and take a look at the various class identifiers.

## How to do it…

To define a file context through an SELinux policy module, use the following approach:

1.  With `matchpathcon`, we can check what is the context that the SELinux tools would reset the resource to:

    ```
    ~# matchpathcon /service/log
    /service/log  system_u:object_r:default_t

    ```

2.  Create the `mylogging.te` file in which we mention the types that are going to be used in the definition. It is a best practice to handle types that are not defined by the SELinux module itself through a different SELinux module. In this example though, we also declare `var_t` to keep the example simple:

    ```
    policy_module(mylogging, 0.2)
    gen_require(`
      type var_t;
      type var_log_t;
      type auditd_log_t;
    ')
    ```

3.  Next, create the `mylogging.fc` file in which we declare the path expressions and their associated file context:

    ```
    /service(/.*)?    gen_context(system_u:object_r:var_t,s0)
    /service/log(/.*)?    gen_context(system_u:object_r:var_log_t,s0)
    /service/log/audit(/.*)?    gen_context(system_u:object_r:auditd_log_t,s0)
    /lxc/.*/log  -d  gen_context(system_u:object_r:var_log_t,s0)
    /var/opt/oracle/listener\.log  --  gen_context(system_u:object_r:var_log_t,s0)
    ```

4.  Now, build the policy module and load it:

    ```
    ~$ make mylogging.pp
    ~$ semodule –i mylogging.pp

    ```

5.  With `matchpathcon`, we can now verify whether the context known to the SELinux tools is the correct one:

    ```
    ~# matchpathcon /service/log
    /service/log  system_u:object_r:var_log_t

    ```

## How it works…

An SELinux policy module contains everything SELinux needs to properly handle a set of policy rules. This includes the rules themselves (which are declared in a `.te` file) with optional interface declarations (in the `.if` files), which define interfaces that other policies can call in order to generate specific SELinux rules. The third and final part of an SELinux policy module is the related file contexts file —hence the `.fc` file suffix.

### Note

Context declarations in a `.fc` file do not automatically enforce and set these contexts. These are merely definitions used by the SELinux utilities and libraries when a relabeling operation occurs.

This contexts file contains, per line:

*   A path expression to which an absolute file path should match
*   An optional class identifier to discern contexts (files, directories, sockets, symbolic links, and so on)
*   The context to be assigned to this path

Each part of the context definition is whitespace delimited:

```
<path>  [<class identifier>]  <context>
```

The lines can be ordered to the policy developers' liking. Most developers order paths in an alphabetical order with grouping based on the top-level directory.

### Path expressions

The regular expression support in the SELinux tools and libraries is based on **Perl-Compatible Regular Expressions** (**PCRE**).

Of all possible expressions, the simplest expression to use is the one without globbing, such as the following code:

```
/var/opt/oracle/listener\.log
```

An important part of this is the escape of the period—if we don't escape the period, then the PCRE support would treat the period as any character matching not only a `listener.log` file, but also `listener_log` or `listenerslog`.

A very common expression is the one that matches a particular directory and all subdirectories and files inside, which is represented in the following example:

```
/service(/.*)?
```

This ensures that there is always a context definition for a file or directory within.

### The order of processing

Given the exhaustive list of path expressions that a regular system has, a file path can match multiple rules, so which one will the SELinux utilities use?

Basically, the SELinux utilities follow the principle of *most specific first*. Given two lines A and B, this is checked in the following order, where the first match wins:

1.  If line A has a regular expression in it and B doesn't, then B is more specific.
2.  If the number of characters before the first regular expression in line A is less than the number of characters before the first regular expression in line B, then B is more specific.
3.  If the number of characters in line A is less than the number of characters in line B, then line B is more specific.
4.  If line A does not specify an SELinux type (so that the context part of it is `<<none>>`) and line B does, then line B is more specific.

The SELinux utilities will load in the definitions given through the files available at `/etc/selinux/mcs/contexts/files/`, but will give preference to the ones in `file_contexts.local` (and then `file_contexts.homedirs`) as those are the definitions made by the system administrator locally. However, if a local definition uses a regular expression and a policy-provided definition doesn't, then the policy-provided definition is still used. This is the only exception to the preference rules between the various context files.

The SELinux utilities provide a tool called `findcon` (part of the `setools` or `setools-console` package) that can be used to analyze this ordering, which shows the matching patterns within a single (!) context definition file and orders them from least specific to most specific:

```
~$ findcon /etc/selinux/mcs/contexts/files/file_contexts -p /var/log/aide
/.*    system_u:object_r:default_t:s0
/var/.*    system_u:object_r:var_t:s0
/var/log/.*  system_u:object_r:var_log_t:s0
/var/log/aide(/.*)?  system_u:object_r:aide_log_t:s0

```

If only the actual context definition is needed (and not the full set of matching expressions with the precedence order as `findcon` shows), then `matchpathcon` can be used instead:

```
~# matchpathcon /var/log/aide
/var/log/aide  system_u:object_r:aide_log_t:s0

```

### Class identifiers

The second part of the context definition is an optional part—a class identifier. Through a class identifier, developers can tell the SELinux utilities that a context definition is only applicable if the path expression matches a particular file class. If the class identifier is omitted, then any class matches.

If a class identifier is shown, then one (per line) of the following identifiers can be used:

*   The '`--`' identifier is used for regular files
*   The '`-d`' identifier is used for directories
*   The '`-l`' identifier is used for symbolic links
*   The '`-b`' identifier is used for block devices
*   The '`-c`' identifier is used for character devices
*   The '`-p`' identifier is used for FIFO files
*   The '`-s`' identifier is used for sockets

### Context declaration

The final part of a context definition is the context itself that is to be assigned to all matching resources. It is generated through the `gen_context` macro, as follows:

```
gen_context(system_u:object_r:var_t,s0)
```

The `gen_context` macro is used to differentiate context definitions based on policy features. If the target policy does not support MLS, then only the first argument (`system_u:object_r:var_t`, in the example) is used. If the policy supports MLS but only a single sensitivity (`s0`), then `:s0` is appended to the context. Otherwise, the second argument (coincidentally also `s0` in the example) is appended (with a colon in front).

Generally, contexts only differ on the SELinux type. The SELinux owner and SELinux role of the resource usually remain `system_u` and `object_r` respectively.

A special value for the context is `<<none>>`, like in the following definition:

```
/proc  -d  <<none>>
```

This tells the SELinux utilities that they should never try to set the context of this resource. Whenever an administrator triggers a filesystem relabeling operation, these specific locations will not have their label changed (regardless of their current label). This does *not* mean that an existing context should be removed!

## There's more...

In the recipe, we covered how to define labels in great detail. If many changes are made, it makes sense to force a relabel on the entire system. On Red Hat systems, this can be accomplished by creating a flag file and rebooting the system:

```
~# touch /.autorelabel
~# reboot

```

On Gentoo systems, the entire system can be relabeled using the `rlpkg` command:

```
~# rlpkg -a -r

```

On Red Hat systems, the command to relabel the system is called `fixfiles`:

```
~# fixfiles relabel

```

This is also needed if a system has been (temporarily) booted without SELinux support or with SELinux disabled as files will be created that have no file context. When an SELinux-enabled system is booted again, it will mark those files as `unlabeled_t`, which is a type that most domains have no access to (SELinux-wise).

# Using substitution definitions

Sometimes, applications and their resources get installed at different locations than expected by the SELinux policy. Trying to accommodate this by defining additional context definitions for each and every subdirectory can easily become unmanageable.

To help administrators, the SELinux utilities support substitution entries, which tell SELinux "if a path starts with *this*, then label it as if it starts with *that*". Administrators can set such a substitution (which is called an **equivalence class**) using `semanage`, as follows:

```
~# semanage fcontext –a –e / /mnt/chroots/bind

```

In this example, any location under `/mnt/chroots/bind/` will be labeled as if it started from the main `/` directory (so `/mnt/chroots/bind/etc/` becomes `etc_t` as `/etc/` is `etc_t`).

Target locations for `chroots` are a good use case for this. A `chroot` is an alternate location on the filesystem, which will act as the root filesystem for one or a set of applications.

For administrators who want to set substitutions across multiple systems, it is not possible to make this part of an SELinux policy module. The file that we need to manage is called `file_contexts.subs` (there is also one that ends with `.subs_dist` and is managed by the Linux distribution, which we will not touch). Having that said, we can always look at how to update this file in a more or less sane manner.

## Getting ready

The easiest method would be to use a central configuration management utility, such as Puppet, CFEngine, Chef, or Ansible, as these systems allow administrators to force the content of specific files to a particular value. The use of a configuration management tool is an entire book in itself, so this is outside the scope of this book. If you do want to pursue this, remember that the `file_contexts.subs` file is (also) managed by the `semanage` command. Administrators might want to add in local definitions that the central configuration management utility isn't aware of (and thus might revert the change).

In this recipe, we'll cover a generic approach, but it does require that there is a way to do both a file transfer followed by a single line command (executed with proper permissions). This, however, shouldn't be much of a challenge to most system administrators.

## How to do it…

In order to apply changes to a wide range of systems, follow the next set of steps:

1.  Apply the change locally to the system:

    ```
    ~# semanage fcontext -a -e / /mnt/chroot/bind

    ```

2.  Export the definitions to a single file:

    ```
    ~# semanage export -f local_selinux.mods

    ```

3.  Edit the `local_selinux.mods` file and remove all entries that are not related to the change but need to be distributed.
4.  Distribute the resulting file to the target systems.
5.  Apply the changes locally to the system:

    ```
    ~# semanage import -f local_selinux.mods

    ```

## How it works…

The `semanage fcontext` command instantiates an equivalence class for `/mnt/chroot/bind/`, which has all subdirectories and files inside of it labeled as if they were at /. This ensures that administrators do not need to define a large amount of file contexts for each and every `chroot` location they manage.

However, this might become problematic as `semanage fcontext` only applies changes locally, and on a larger infrastructure, the same settings might need to be applied to multiple systems. For this, `semanage export` and `semanage import` can be used.

The output of the `semanage export` command is a set of instructions for `semanage` and follows the syntax of the `semanage` commands to the letter.

When exporting the `semanage` definitions, the first set of commands that are stored are the `delete all` statements such as `fcontext -D` (delete all locally made `semanage fcontext` settings). Of course, if we only need to distribute the substitution definitions, then deleting all previously made local statements is incorrect. Hence, the need to manually edit the `local_selinux.mods` file. If only the equivalence class definition needs to be distributed, then the file might just contain the following:

```
fcontext -a -e / /mnt/chroot/bind
```

The exported file can then be distributed to all target systems and loaded through the `semanage import` command effectively applying the same set of changes to the system.

If the definition was already applied on a system, then the `import` command will fail:

```
~# semanage import -f local_selinux.mods
ValueError: Equivalence class for /mnt/chroot/bind already exists

```

It is important to note here that if one command in the file fails to apply, then none of the commands in the file are applied (the file is processed in one go). This is why the `delete all` rules are originally made part of the exported set of commands.

This makes distributed management of such settings more challenging if locally applied changes need to be kept as well, unless the distributed set of changes are singular (one exported instruction, which is allowed to fail).

## There's more...

Most files inside the `/etc/selinux/mcs/contexts/` location shouldn't be managed through any tool except either the Linux distribution package management system (through the installation of the base SELinux policy) or `semanage`.

That being said, most files inside this location don't change much (except for the `files/file_contexts` file). It might be beneficial to hook into the package management system to update these files (if supported) or bluntly take over the management of these files, assuming you track the changes that the distribution would make closely.

## See also

The following resources dive deeper into the topics discussed in this recipe:

*   To find out more about the various configuration files, check out [http://selinuxproject.org/page/PolicyConfigurationFiles](http://selinuxproject.org/page/PolicyConfigurationFiles)
*   The interaction of SELinux with `chroots` is discussed in more detail in [Chapter 9](ch09.html "Chapter 9. Aligning SELinux with DAC"), *Aligning SELinux with DAC*

# Enhancing an SELinux policy with file transitions

Up until now, we've only handled the configuration part on file contexts: if we would ask SELinux utilities to relabel files, then the changes we made would come into effect. However, unless you run with the `restorecond` daemon checking out all possible file modifications (which would really be a resource hog) or run `restorecon` manually against all files regularly, the newly defined contexts will not be applied to the files.

What we need to do is modify the local SELinux policy so that, upon creation of these files, the Linux kernel automatically assigns the right label to those files. This is handled through file transitions, which is a specific case of a **type transition**.

In a type transition, we configure a policy so that if a given domain creates a file (or other resource class) inside a directory with a specified label, then the created object should automatically get a specific label. Policy-wise, this is written as follows:

```
type_transition <domain> <directory_label>:<resource_class> <specific_label>
```

SELinux has also added in support for named file transitions (from Linux 2.6.39 onwards, and available in Gentoo, Fedora 16+, and Red Hat Enterprise Linux 7+). In that case, such a transition only occurs if the created resource matches a particular filename exactly (so no regular expressions):

```
type_transition <domain> <directory_label>:<resource_class> <specific_label> <filename>
```

Through the reference policy macro's, this is supported with the `filetrans_pattern` definition.

## Getting ready

In order to properly define file transitions, we need to know what the source domain is that is responsible for creating the resource. For instance, a `/var/run/snort/` directory might be created by an `init` script, but if there is no file transition, then this directory will be created with the type of the parent directory (which is `var_run_t`) instead of the proper type (`snort_var_run_t`).

So make sure to write down all the involved labels (as an example, we will use `initrc_t` for an `init` script, `var_run_t` for the parent directory, and `snort_var_run_t` for the target directory) before embarking on this recipe.

## How to do it…

Defining a file transition can be done as follows:

1.  Search through the SELinux policies to see if there is an interface that will provide a file transition from a given domain to `snort_run_t`:

    ```
    ~$ sefindif filetrans.*snort_var_run_t

    ```

2.  Assuming that none have been found, search for interfaces that allow `initrc_t` created resources to transition to a given type:

    ```
    ~$ sefindif filetrans.*initrc_t
    system/init.if: interface(`init_daemon_pid_file',`
    system/init.if:   files_pid_filetrans(initrc_t, $1, $2, $3)

    ```

3.  Bingo! Now, let's create an enhancement for the snort SELinux module (through a `mysnort` policy file) with the following declaration in it:

    ```
    policy_module(mysnort, 0.1)
    gen_require(`
      type snort_t;
      type snort_var_run_t;
    ')
    # If initrc_t creates a directory called "snort" in a var_run_t dir,
    # make sure this one is immediately labeled as snort_var_run_t.
    init_daemon_pid_file(snort_var_run_t, dir, "snort")
    ```

4.  Build the new policy and load it. Then check with `sesearch` if a type transition is indeed declared:

    ```
    ~$ sesearch –s initrc_t –t var_run_t –T | grep "snort"
    type_transition initrc_t var_run_t : dir snort_var_run_t "snort"

    ```

## How it works…

Linux distributions that support SELinux already provide an SELinux policy that works in a majority of deployments. The default policy is extensive and works mostly out of the box. If specific changes are needed, chances are that these particular SELinux rules are already defined (as part of policy interfaces) and only need to be instantiated and loaded.

Policy interfaces usually exist in the following two types:

*   Interfaces whose subject is delivered through an argument, and where the object (against which operations are performed) and perhaps target (in our case, to which a transition should occur) are hardcoded
*   Interfaces whose subject is hardcoded and where the object, target, or both are arguments to the interface

An example of the first interface type that can be used in our example would look like the following code:

```
interface(`snort_generic_pid_filetrans_pid',`
  gen_require(`
    type snort_var_run_t;
  ')
  files_pid_filetrans($1, snort_var_run_t, dir, $2)
')
```

We could then call this interface like this:

```
snort_generic_pid_filetrans_pid(initrc_t, "snort")
```

However, such interfaces would be a burden to maintain. For every daemon support added to the system, the `init` policy would need to be changed with a named file transition together with the newly added policy rules for the daemon. Considering the amount of daemons that can run on a system, the `init` policy would literally be filled with a massive amount of named file transitions—at least one for every daemon.

The interface declaration that we encountered in the example is much more manageable. The interface is meant to be called by the daemon policy itself and immediately ensures that the `initrc_t` type can create directories of the given type (`snort_var_run_t`) inside the generic run directory (`var_run_t`). New additions to the policy leave the `init` policy at rest, making maintenance of the policies easier.

### Finding the right search pattern

To find the right pattern, we use the `sefindif` interface to search through the available interfaces. Finding the right expression is a matter of experience.

As we know, we want to search for file transitions, the line we are looking for will contain `filetrans_pattern`. Then, one of the arguments involved is the type we are going to transition to (`snort_var_run_t`). So the expression we used in the example was changed to `filetrans.*snort_var_run_t`. As that didn't result in anything, the next search involved the domain from which a transition has to be made (`initrc_t`) so that the expression was changed to `filetrans.*initrc_t`.

However, let's assume we don't know that `filetrans_pattern` needs to be searched for. The type itself (`snort_var_run_t`) or domain (`initrc_t`) might be sufficient to search through, like in the following searches:

```
~$ sefindif snort_var_run_t
~$ sefindif initrc_t

```

From the resulting list of interfaces, we can then see if an interface is available that suits our needs.

### Patterns

Patterns such as `filetrans_pattern` are important supporting definitions inside the reference policy. They bundle a set of permissions related to a functional approach (such as read files, which are handled through a `read_files_pattern`) and are not tied to a particular type (unlike interfaces).

The need for patterns comes from the very fine-grained access controls that SELinux has on Linux activities. Reading a file is a nice example: it is not sufficient to just allow a type to perform the `read` action:

```
allow initrc_t snort_var_run_t:file read;
```

Most applications first check the attributes of the file (`getattr`) and open the file before they can read the file. Depending on the purpose, they might also want to lock the file or perform I/O operations on it through `ioctl`. So instead of just the preceding access vector, the rule was changed to:

```
allow initrc_t snort_var_run_t:file { getattr lock open read ioctl }
```

The reference policy provides a single permission set for this called `read_file_perms`, which turns the access vector into the following:

```
allow initrc_t snort_var_run_t:file read_file_perms;
```

Second, the policy developers often want to allow a domain to read a file inside a directory that is labeled similarly. For instance, a `snort_var_run_t` file can be at `/var/run/snort/snort.pid` with the `/var/run/snort/` directory also being labeled as `snort_var_run_t`. So we would also need to grant the `initrc_t` type search rights inside the directory—which again is a set of permissions as can be seen from the `search_dir_perms` definition:

```
~$ seshowdef search_dir_perms
define(`search_dir_perms',`{ getattr search open }')

```

Instead of creating multiple rules for this, a pattern is created, called `read_files_pattern`, which looks like the following:

```
~$ seshowdef read_files_pattern
define(`read_files_pattern',`
 allow $1 $2:dir search_dir_perms;
 allow $1 $3:file read_file_perms;
')

```

This allows policy developers to use a single call:

```
read_files_pattern(initrc_t, snort_var_run_t, snort_var_run_t)
```

To see the various patterns supported for policy development, use `sefinddef` with the '`define.*_pattern`' expression:

```
~$ sefinddef define.*_pattern

```

Using patterns allows developers to create readable policy rules using a functional approach rather than a full sum-up of each individual access vector.

## There's more...

In the `snort_generic_pid_filetrans_pid` interface presented earlier, we used a named file transition: the transition occurs only if the filename passed on as the last argument matches the filename of the file created.

Named file transitions take precedence over normal file transitions. A good example for this are the file transitions supported for the `initrc_t` domain:

```
~# semanage –s initrc_t –t var_run_t –T
Found 2 semantic te rules:
 type_transition initrc_t var_run_t : file initrc_var_run_t;
 type_transition initrc_t var_run_t : dir initrc_var_run_t;
Found 16 named file transition rules:
type_transition initrc_t var_run_t : dir udev_var_run_t "udev";
type_transition initrc_t var_run_t : dir tor_var_run_t "tor";
…

```

In this case, if an `init` script creates a directory called `udev` or `tor` (or any of the other transition rules that are not shown in the example), then a proper file transition occurs. If the filename doesn't match, then a transition occurs to the `initrc_var_run_t` type.

File transitions on regular files and directories are the most common, but transitions can also occur on various other classes, such as sockets, FIFO files, symbolic links, and more.

## See also

*   Domain transitions (which assign a different context to a process rather than a file) are covered in [Chapter 3](ch03.html "Chapter 3. Confining Web Applications"), *Confining Web Applications* in more detail and are used in [Chapter 4](ch04.html "Chapter 4. Creating a Desktop Application Policy"), *Creating a Desktop Application Policy* and [Chapter 5](ch05.html "Chapter 5. Creating a Server Policy"), *Creating a Server Policy*

# Setting resource-sensitivity labels

When an SELinux policy is MLS-enabled and supports multiple sensitivities (which is not the case with MCS, as MCS only has a single sensitivity), then SELinux can govern information flow and access between a domain and one or more resources based on the clearance of the domain and the sensitivity level of the resource. But even with a single sensitivity (as is the case with MCS), SELinux has additional constraint support to ensure that domains cannot access resources that have one of the categories assigned that the domain doesn't have clearance for.

A sensitivity level consists of a sensitivity (`s0` is generally being used for the lowest sensitivity and `s15`—which is a policy build-time constant and thus can be configured—is the highest sensitivity) together with a category set (which can be a list such as `c0,c5,c8.c10`).

A security clearance is similar to a sensitivity level but shows a sensitivity range (such as `s0-s3`) instead of a single sensitivity level. A security clearance can be seen as a range going from the lowest sensitivity level to the highest sensitivity level allowed by the domain.

When policies are being developed for such systems, context definitions and policy rules can take sensitivities into account. In this recipe, we will do the two most common operations for MLS-enabled systems:

*   Define a context with a higher-level sensitivity
*   Set the clearance of a process policy-wise on a domain transition

To accomplish this, we will use the snort intrusion detection system as an example, forcing it to be always executed with the `s3` sensitivity and all possible categories.

This example will also show us how to substitute an existing policy rather than enhance it, as we are going to update a definition that would otherwise collide with the existing definition.

## How to do it…

To modify an existing domain to support specific sensitivity levels, execute the following steps:

1.  Copy the `snort.te` and `snort.fc` files from the distribution policy repository to the local environment:

    ```
    ~$ cp ${POLICY_LOCATION}/policy/modules/contrib/snort.* ${DEVROOT}/local

    ```

2.  Rename the files to `mysnort` (or `customsnort`), so we always know this is a customized policy. Don't forget to update the `policy_module` call in the `.te` file.
3.  Open the `mysnort.te` file and look for the `init_daemon_domain` call. Substitute the call with the following:

    ```
    init_ranged_daemon_domain(snort_t, snort_exec_t,  s3:mcs_allcats)
    ```

4.  In `mysnort.fc`, label the snort resources with the `s3` sensitivity. For instance, for the snort binary, label it as follows:

    ```
    /usr/bin/snort  --  gen_context(system_u:object_r:snort_exec_t,s3)
    ```

5.  Build the `mysnort` policy, remove the currently loaded snort SELinux policy module, and load the `mysnort` one:

    ```
    ~# /etc/init.d/snort stop
    ~# semodule –r snort
    ~# semodule –i mysnort.pp

    ```

6.  Relabel all files related to snort and then start snort again.

## How it works…

There are three important aspects to this recipe:

1.  We replace the entire policy rather than create an enhancement.
2.  We update the policy to use a ranged daemon domain.
3.  We update the file contexts to use the right sensitivity.

The file context update is obvious but the reason for fully replacing the policy might not be.

### Full policy replacement

In the example, we copied the existing policy for the snort SELinux module and made the updates in the copy, rather than trying to enhance the policy by creating an additional module.

This is needed because we are making changes to the SELinux policy that are mutually exclusive to the already running SELinux policy. For instance, the file context changes would confuse SELinux as it would then have two fully matching definitions through policy modules, but each with a different resulting context.

In the example, we only copied the type enforcement declarations (`snort.te`) and file context declarations (`snort.fc`). If we would copy the interface definitions as well (`snort.if`), the policy build would give us a warning that there are duplicate interface definitions—the ones provided by the Linux distribution are still on the system after all.

### Ranged daemon domain

In the SELinux policy itself, we substituted the `init_daemon_domain(snort_t, snort_exec_t)` entry with the following:

```
init_ranged_daemon_domain(snort_t, snort_exec_t, s3:mcs_allcats)
```

Let's take a look at the contents of this interface:

```
~$ seshowif init_ranged_daemon_domain
interface(`init_ranged_daemon_domain',`
 gen_require(`
 type initrc_t;
 ')
 init_daemon_domain($1, $2)
 ifdef(`enable_mcs',`
 range_transition initrc_t $2:process $3;
 ')
 ifdef(`enable_mls',`
 range_transition initrc_t $2:process $3;
 mls_rangetrans_target($1)
 ')
')

```

The newly called interface calls the original `init_daemon_domain`, but enhances it with MCS- and MLS-related logic. In both cases, it calls `range_transition` so that when the snort `init` script (running as `initrc_t`) transitions to the `snort_t` domain, then the active sensitivity range is also changed to the third parameter.

In our case, the third parameter is `s3:mcs_allcats`, where `mcs_allcats` is a definition that expands to all categories supported by the policy (such as `c0.c255` if the policy supports 256 categories).

In case of MLS, it also calls `mls_rangetrans_target`, which is an interface that sets an attribute to the `snort_t` domain, which is needed for the MLS constraints enabled in the policy.

From the expanded code, we can see that there are `ifdef()` statements. These are blocks of SELinux policy rules that are enabled (or ignored) based on build-time parameters. The `enable_mcs` and `enable_mls` parameters are set if an MCS or MLS policy is enabled. Other often used build-time parameters are distribution selections (such as `distro_redhat` if the SELinux policy rules are specific for Red Hat Enterprise Linux and Fedora systems) and `enable_ubac` (which is when user-based access control is enabled).

### Constraints

Most, if not all, SELinux policy development focuses on type enforcement rules and context definitions. SELinux does support various other statements, one of which is the `constrain` statement used to implement constraints.

A constraint restricts permissions further based on a set of expressions that cover not only the type of the object or subject, but also SELinux role and SELinux user. The constraint that is related to the `mlsrangetrans` attribute (which is set by the `mls_rangetrans_target` interface) looks like the following:

```
mlsconstrain process transition
  (( h1 dom h2 ) and
   (( l1 eq l2 ) or ( t1 == mlsprocsetsl ) or
    (( t1 == privrangetrans ) and ( t2 == mlsrangetrans ))));
```

The constraint tells us the following things about a transition:

*   The transition can occur only when the highest sensitivity level of the subject (domain/actor) dominates the highest sensitivity level of the object
*   The lowest sensitivity level of the subject is the same as the lowest sensitivity level of the object
*   If not, then the type of the subject has to have the `mlsprocsetsl` attribute set
*   If not, then both of the following statements have to be true:

    *   The type of the subject has the `privrangetrans` attribute set
    *   The type of the object has the `mlsrangetrans` attribute set

Domination means that the sensitivity level of the first security level is equal to or higher than the sensitivity level of the second security level, and the categories of the first security level are the same or a superset of the categories of the second security level.

Constraints in the SELinux policy are part of the base policy set—this means that we are not able to add constraints through loadable SELinux policies. If we want to include additional constraints, we would need to build the entire policy ourselves, patching the `constraints`, `mls`, and `mcs` files inside the policy repository's `policy/` subdirectory.

Knowing about constraints is important, but we probably never need to write constraints ourselves.

## See also

The SELinux project site is a good start for learning about constraints and their related statements:

*   The MLS statements at [http://selinuxproject.org/page/NB_MLS](http://selinuxproject.org/page/NB_MLS)
*   The constraint statements at [http://selinuxproject.org/page/ConstraintStatements](http://selinuxproject.org/page/ConstraintStatements)

# Configuring sensitivity categories

Although MCS policies are MLS-enabled, they are configured to only support a single sensitivity (namely `s0`). Yet even with this limitation, an MCS policy can be very useful, for instance, in situations where a system hosts services for multiple customers. This is because MCS can still benefit from security clearances based on categories.

Unlike sensitivities, categories are more like a discretionary access control system. Categories are meant to be used by users (or administrators) to label files and other resources as being a member of one or more categories. Access to those resources is then based on the clearance level of the process and the categories assigned to the resource. Categories are also not hierarchically structured.

An example of a use case where categories play a major role is in multitenant deployments: systems that host one or more services for multiple tenants (multiple customers), which, of course, require proper security segregation so that one tenant cannot access resources of another tenant.

In most cases, administrators will try to separate those services through the runtime user (and group membership). This is, however, not always possible. There are situations where these separate processes still need to run as the same runtime user (although with support for additional Linux security subsystems—such as capabilities—the number of situations has significantly reduced again).

In this recipe, we'll configure a system to use multiple categories to differentiate between resources of different customers for a web server that the customers also have shell access to. Through categories, we can provide more protection for the resources of other customers, in case one of the customers is able to execute an exploit that would elevate their privileges.

## Getting ready

You need to prepare a system for the multiple tenants. For instance, we can host the entire website content in `/srv/web/<companyname>/` and have the web server configuration at `/etc/apache/conf/<companyname>/`.

In this recipe, as an example, we will configure the system for two companies called `CompanyX` and `CompanyY`. Each company also has a regular user (`userX` for the first company and `userY` for the second).

## How to do it…

To instantiate different categories, follow this approach:

1.  Settle on the category naming (and numbers) for different customers and configure those in the `setrans.conf` file inside `/etc/selinux/mcs/`:

    ```
    s0:c100=CompanyX
    s0-s0:c100=CompanyXClearance
    s0:c101=CompanyY
    s0-s0:c101=CompanyYClearance
    ```

2.  Restart the `mcstrans` service so that it is aware of this configuration.
3.  List the categories to make sure that the changes are properly interpreted:

    ```
    ~$ chcat –L
    s0    SystemLow
    s0-s0:c0.c1023  SystemLow-SystemHigh
    s0:c0.c1023  SystemHigh
    s0:c100    CompanyX
    s0-s0:c100  CompanyXClearance
    s0:c101    CompanyY
    s0-s0:c101  CompanyYClearance

    ```

4.  Create SELinux users that have clearance to handle the right categories:

    ```
    ~# semanage user –a –L s0 –r CompanyXClearance –R "user_r" userX_u
    ~# semanage user –a –L s0 –r CompanyYClearance –R "user_r" userY_u

    ```

5.  Configure the Linux users (logins) with the right security clearance:

    ```
    ~# semanage login –m –s userX_u userX
    ~# semanage login –m –s userX_u userY

    ```

6.  Set the right category on the company resources:

    ```
    ~# chcon –l CompanyX –R /srv/web/www.companyX.com/ /etc/apache/conf/companyX/
    ~# chcon –l CompanyY –R /srv/web/www.companyY.com/ /etc/apache/conf/companyY/

    ```

7.  Configure the Apache `init` scripts to launch Apache with the right security level by launching it through `runcon`. For instance, on a Red Hat Enterprise Linux 6 system for the first company's web server, the following script is used:

    ```
    LANG=$HTTPD_LANG daemon --pidfile=${pidfile} runcon –t httpd_t –l CompanyX $httpd $OPTIONS
    ```

8.  (Re)start the web server and validate that it is running with the right security level:

    ```
    ~# ps –efZ | grep httpd

    ```

## How it works…

We started by configuring the system so that we can name categories and ranges rather than having to use the integer representations. Next, we created an SELinux user for each company and assigned each (regular) Linux account to the right SELinux user. After updating the contexts of all company-related files, we configured Apache to start in the right context.

### The mcstrans and setrans.conf files

The `setrans.conf` file is a regular text file that the MCS transition daemon (`mcstransd`) uses to substitute the real security level (such as `s0:c100`) with a human readable string (such as `CompanyX`).

The Linux utilities themselves (such as `ls` and `ps`) use the SELinux libraries to get information about the contexts of files and processes. These libraries then connect with the `mcstransd` process (through the `/var/run/setrans/.setrans-unix` socket), sending the real security level and retrieving the human-readable representation for it.

It is important to remember that this is only a representation and not how the security level is stored. In other words, do not use this in file context definition files (that is, the SELinux policy `.fc` files).

### SELinux users and Linux user mappings

In the example, an SELinux user is created for each company. This SELinux user is given the clearance to work with resources tagged with the category of the respective companies. The real Linux accounts are then mapped to this SELinux user.

From the example, we see that there are two definitions for each company:

```
s0:c100    CompanyX
s0-s0:c100  CompanyXClearance
```

The first one is a security level and can be assigned to both resources as well as processes (users). The second one is a security clearance (a range). In this particular example, the clearance tells us that the high security level (which can be seen as *what the process is allowed to access*) are the resources of the company (`s0:c100`), and the low security level (which can be seen as *the security level of the process itself* ) is just `s0`.

The users for the company, therefore, have clearance to access the files (and other resources) that have their company's category assigned to it. However, all activities performed by these user accounts do not get this category by default—the users will need to use `chcon` to set the category, as follows:

```
~$ chcon –l CompanyX public_html/index.html

```

It is possible to give the users the security level itself rather than the clearance. When that occurs, any resource created by the user will also get the proper category set. But, do not use this as a way to confine resources—users can always remove categories from resources.

Granting the security level can be done on the SELinux user level, but it is also possible to do this through the SELinux user mapping as long as the range passed on is dominated by the range set on the SELinux user level. For instance, to set `CompanyX` (`s0:c100`) as the security level rather than `CompanyXClearance`, which is the default for users mapped to the `userX_u` SELinux user, the following command can be used:

```
~# semanage login –m –r CompanyX user1

```

### Running Apache with the right context

The last change made in the example was to configure the system to start the web server with the right security level. This is done through the `runcon` command, where we pass on the sensitivity level (and not the security clearance) to make sure that every resource created through the web server inherits the right category as well as the target type.

The SELinux policy knows that if an `init` script launches the Apache binary (`httpd`), then this application has to run in the `httpd_t` domain. However, now the `init` script launches `runcon`—which the SELinux policy sees as a regular binary—so the application would continue to run in the `initrc_t` domain. Hence, we need to pass on the target type (`httpd_t`). On systems with an SELinux policy without unconfined domains, forgetting this would prevent the web server to run. On systems with an SELinux policy with unconfined domains, this might result in the web server to run in an unconfined domain (`initrc_t`), effectively disabling the SELinux protections we need for the web server!

## See also

The following are some more examples on multitenancy and how SELinux interacts with it:

*   sVirt ([http://selinuxproject.org/page/SVirt](http://selinuxproject.org/page/SVirt)) uses SELinux categories to segregate virtual guests from one another
*   Linux containers, such as through the LXC project ([https://linuxcontainers.org](https://linuxcontainers.org)), use SELinux for further isolation of containers from the main system
*   Apache has support for multitenancy through the `mod_selinux` module, which is covered in [Chapter 3](ch03.html "Chapter 3. Confining Web Applications"), *Confining Web Applications*