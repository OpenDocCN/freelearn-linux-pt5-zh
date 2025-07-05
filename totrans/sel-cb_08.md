# Chapter 8. Debugging SELinux

In this chapter, we will look at SELinux debugging through the following recipes:

*   Identifying whether SELinux is to blame
*   Analyzing SELINUX_ERR messages
*   Logging positive policy decisions
*   Looking through SELinux constraints
*   Ensuring an SELinux rule is never allowed
*   Using strace to clarify permission issues
*   Using strace against daemons
*   Auditing system behavior

# Introduction

On an SELinux-enabled system, the SELinux policy defines how applications should behave. Any change in behavior might trigger SELinux denials for certain actions of that application. As a result, end users can notice unexpected permission issues or erratic application behavior.

Troubleshooting such situations is usually done through analysis of the AVC events. Many resources already cover AVC events in great detail. The basic premise is that an AVC event uses a set of key-value pairs, as follows:

```
type=AVC msg=audit(1369306885.125:4702304): avc: denied { append } for pid=1787 comm="syslog-ng" name="oracle_audit.log" dev=dm-18 ino=65 scontext=system_u:system_r:syslogd_t:s0 tcontext=system_u:object_r:usr_t:s0 tclass=file
```

In this example, we can deduce the following from the AVC event:

*   The event is a denial (`avc: denied`)
*   The operation that was denied is appending to a file (`{ append } … tclass=file`)
*   The process that tried to append to the file has PID `1787` and name `syslog-ng` (`pid=1787 comm="syslog-ng"`)
*   The process' context is `syslogd_t` (`scontext=system_u:system_r:syslogd_t:s0`)
*   The target file is called `oracle_audit.log` and has an inode number `65` on the filesystem, stored on the `/dev/dm-18` metadevice (`name="oracle_audit.log" dev=dm-18 ino=65`)
*   The file's context is `usr_t` (`tcontext=system_u:object_r:usr_t:s0`)

However, sometimes it isn't sufficient to find out where the problem is. Luckily, there are many more options available to debug the problem.

# Identifying whether SELinux is to blame

Before blaming the SELinux subsystem and policies for a problem, it is important to verify whether SELinux is to blame at all. Too often, hours of troubleshooting are put in analyzing the SELinux policies and subsystem only to find out that the problem also persists when SELinux is not enabled.

## How to do it…

In order to be confident that SELinux is (or isn't) to blame, the following set of steps can be taken:

1.  Is it possible to get more information through the application's internal debugging system? Consider the following instance:

    ```
    ~# puppet master
    Error: Could not find class puppet::agent for foo.bar on node foo.bar
    ~# puppet master --debug --no-daemonize --verbose
    ```

2.  Is an AVC denial related to the problem shown in the audit logs? If not, try disabling the `dontaudit` rules and try again:

    ```
    ~# semodule -DB

    ```

3.  Is the application that gives problems SELinux-aware? Most SELinux-aware applications are linked with the `libselinux.so` library, so we can verify whether this is the case using `ldd` or `scanelf`:

    ```
    ~# ldd /usr/bin/dbus-daemon
     linux-vdso.so.1 =>  (0x00007fff56df4000)
     libexpat.so.1 => /lib64/libexpat.so.1 (0x00007f55710ae000)
     libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f5570e8f000)
     libaudit.so.1 => /lib64/libaudit.so.1 (0x00007f5570c72000)
     libcap-ng.so.0 => /lib64/libcap-ng.so.0 (0x00007f5570a6d000)
     libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5570850000)
     librt.so.1 => /lib64/librt.so.1 (0x00007f5570647000)
     libc.so.6 => /lib64/libc.so.6 (0x00007f55702b3000)
     libdl.so.2 => /lib64/libdl.so.2 (0x00007f55700af000)
     /lib64/ld-linux-x86-64.so.2 (0x0000003458000000)

    ```

4.  Is the issue login related? If so, an application might not be SELinux-aware but still behave differently, as it uses PAM under the hood, which calls the `pam_selinux.so` library.
5.  Does the problem still persist if the application domain is put in permissive mode? To check this, issue the following command:

    ```
    ~# semanage permissive -a portage_t

    ```

6.  If the application domain is unknown, try putting the entire system in permissive mode (if allowed) to see whether the problem is still showing up. If it is, then SELinux might not be the cause after all:

    ```
    ~# setenforce 0

    ```

## How it works…

Ensuring that SELinux is the cause of a problem is the first step to enlightenment. Numerous hours of SELinux investigations to resolve issues are spent only to find out that the problem was not with SELinux to begin with.

Getting more information from the application (or applications) involved is the first step to troubleshooting issues. Many applications have command-line flags that increase logging verbosity, and many daemons can be configured to log more of their inner workings. The resulting debug information (or even trace information, if the application supports it) will provide a massive help to the administrator to troubleshoot a problem.

If additional logging does not help, then it is important to verify whether there are AVC denials in the audit logs. As some AVC denials can be hidden during regular operations, disabling the `dontaudit` rules temporarily might be necessary. Don't stare blindly at AVC denials though, and take a broader look at logfiles and audit events. For instance, in the next recipe (*Analyzing SELINUX_ERR messages*), a more in-depth analysis of a particular audit event type is discussed.

Look through the various logs on the system as well. The output of `dmesg` is important if the problem is kernel, hardware, or core-system related. The `messages` logfile (in `/var/log/`) usually contains pointers when issues come up with system daemons.

When no denials are shown and there is no specific logging that can assist with the troubleshooting of an application, the next step is to assure ourselves that the application is not SELinux-aware.

SELinux-aware applications (applications that know they run on an SELinux-enabled system and interact with the SELinux subsystem) can act differently based on the SELinux policy that is loaded, without actually triggering any SELinux decision in the SELinux subsystem. On account of their awareness, the in-kernel SELinux subsystem access controls might not be called, so no logging will be shown even though the problem is somewhat SELinux-related.

Although there is not any 100 percent certain method to check whether an application is SELinux-aware, the two most common approaches are as follows:

*   Checking whether the application binary is linked with the `libselinux.so` library
*   Checking whether the application uses PAM

An application that is linked with the `libselinux.so` library is SELinux-aware and will be able to query SELinux policies, possibly acting differently when SELinux is enabled and often regardless of SELinux being in the enforcing or permissive mode.

Besides the `ldd` command, it is also possible to use the `scanelf` application as provided by the `pax-utils` package. This application does not need execute privileges against the binary (which `ldd` requires) but has the downside that it only shows the requirements for the binary, while `ldd` also includes the libraries linked by the libraries themselves:

```
~$ scanelf -n /usr/bin/dbus-daemon
 TYPE   NEEDED FILE
ET_DYN libexpat.so.1,libselinux.so.1,libaudit.so.1,libcap-ng.so.0,libpthread.so.0,librt.so.1,libc.so.6 /usr/bin/dbus-daemon

```

Applications that use PAM can also be influenced by SELinux, since their PAM configuration might call the `pam_selinux.so` library (or not call it, which can be equally damaging for the functionality of the application as no transition will occur then, having the user session still run with the context of the daemon).

If the application does not interact with the SELinux subsystem to query the SELinux policy, and it also doesn't handle SELinux labels directly (that is, it has no knowledge of SELinux labels and does not actively work with them code-wise), then running the application in the permissive mode should show us whether SELinux is to blame. In the permissive mode, the SELinux subsystem access controls do not prevent any action. If a problem still persists in the permissive mode, chances are that SELinux is not to blame at all.

## See also

*   More information about SELinux-aware applications and how to write one is covered in [Chapter 10](ch10.html "Chapter 10. Handling SELinux-aware Applications"), *Handling SELinux-aware Applications*

# Analyzing SELINUX_ERR messages

When the SELinux subsystem is asked to perform an invalid SELinux-specific operation, it will log this through the audit subsystem using the `SELINUX_ERR` message type.

## Getting ready

Make sure that the audit subsystem is up and running as we will be using the `ausearch` application to (re)view audit events:

```
~# service auditd start

```

## How to do it…

Analyzing `SELINUX_ERR` messages is done by viewing the entry in the audit logs and understanding the individual fields; this is done by completing the following steps:

1.  Note the current date/time, or reload the SELinux policy, to have a clear point in the audit logs from where to look:

    ```
    ~# semodule -R

    ```

2.  Trigger the behavior in the application.
3.  Ask the audit subsystem to show the last events of the `SELINUX_ERR` and `MAC_POLICY_LOAD` types:

    ```
    ~# ausearch -m SELINUX_ERR,MAC_POLICY_LOAD -ts recent

    ```

4.  Look at the beginning of the message to find out what problematic situation SELinux is informing us about.

## How it works…

The SELinux subsystem will log any incorrect request. If it is application behavior, it is usually logged through the AVC type; but when the request is SELinux-specific and incorrect, an `SELINUX_ERR` message type is displayed. In the example, we also looked for the `MAC_POLICY_LOAD` type, so we know at which stage the SELinux policy was reloaded, giving us a good starting point for the investigation.

Some examples of the `SELINUX_ERR` messages are as follows:

*   `security_compute_sid`: Invalid context
*   `security_validate_transition`: Denied
*   `security_bounded_transition`: Denied

Some other messages exist as well, although these are mostly for SELinux-internal problems (related to the SELinux subsystem inside the Linux kernel, such as supported netlink types), which need to be resolved by the SELinux maintainers themselves, and not by policy developers.

### Invalid contexts

An invalid context is triggered when a context that is not valid according to the RBAC and SELinux user rules is created. This is usually the case during a domain transition, where the target type is not allowed for the role:

```
time->Wed Aug 4 03:19:04 2014
type=SYSCALL msg=audit(10590262134.246:135): arch=c000003e syscall=59
success=no exit=-13 a0=187b190 a1=187b120 a2=187ac30 a3=7ffff2dc3ec0 items=0
ppid=14696 pid=15085 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0
egid=0 sgid=0 fsgid=0 tty=(none) ses=21 comm="logwatch" exe="/usr/bin/perl"
subj=system_u:system_r:logwatch_t:s0-s0:c0.c1023 key=(null)
type=SELINUX_ERR msg=audit(10590262134.246:135): security_compute_sid:
invalid context system_u:system_r:logwatch_mail_t:s0-s0:c0.c1023 for
scontext=system_u:system_r:logwatch_t:s0-s0:c0.c1023
tcontext=system_u:object_r:sendmail_exec_t:s0 tclass=process
```

Another reason for an invalid context can be that a role transition is triggered, but this role is not allowed for an SELinux user:

```
type=SELINUX_ERR audit(1257378096.775:46): security_compute_sid: invalid context
dbadm_u:system_r:mysqld_safe_t:s0 for scontext=dbadm_u:dbadm_r:initrc_t:s0
tcontext=system_u:object_r:mysqld_safe_exec_t:s0 tclass=process
```

In both cases, it is important to look at the presented context and the `scontext` and `tcontext` fields. These show the contexts that SELinux finds invalid (presented context) as well as the source (domain initiating the action) and the object context (label through which the new context was decided upon). Based on these, it should be fairly easy to deduce what the error is about.

The first example shows an attempt to transition from the `logwatch_t` domain (which is allowed for the `system_r` role) to the `logwatch_mail_t` domain (which is not allowed for the `system_r` role). To solve this, `logwatch_mail_t` needs to be allowed for the `system_r` role:

```
allow system_r types logwatch_mail_t;
```

The second example is triggered through a role transition. A database administrator launches an `init` script, resulting in the `dbadm_u:dbadm_r:initrc_t` context. This domain executes the `mysqld_safe` application (whose file is labeled `mysqld_safe_exec_t`) that, through the SELinux policy, attempts to perform a role transition to the `system_r` role. Although the `system_r:mysqld_safe_t` context is a valid set, the database administration user itself is not allowed the `system_r` role.

The main issue in this second example is that the context to start from (`dbadm_u:dbadm_r:initrc_t`) shouldn't be used. The `initrc_t` domain should only be allowed for the `system_r` role. This, by itself, requires that the `dbadm_u` SELinux user is also allowed the `system_r` role. So, even though allowing the `system_r` role is the right resolution, the approach taken in the example is wrong (role transition from `initrc_t` to `mysqld_safe_t` instead of role transitioning upon instantiating `initrc_t`).

### Denied transition validation

Consider the following error message, which came up when an `init` script tried to increase the sensitivity of a file:

```
type=SELINUX_ERR audit(125482134923.234:25): security_validate_transition:
denied for oldcontext=system_u:object_r:selinux_config_t:s0
newcontext=system_u:object_r:selinux_config_t:s15:c0-c1023
taskcontext=system_u:system_r:initrc_t=s0-s16:c0.c1023 tclass=file
```

Such a message occurs when a file transition is performed, but where the target security context is not allowed. SELinux validates whether this is allowed; if not allowed, it logs this through the message.

AVC-like denials will be in place here, but the access vector cache system is only able to validate pair-wise contexts (the source and target contexts), whereas the transition validation needs to be done on three levels (old file context, new file context, and process context).

The solution for the presented error will be to either allow `initrc_t` to raise the security level of a file (through the `mls_file_upgrade` interface) or to not have the `init` script domain try to update the MLS level of a file in the first place.

### Denied security-bounded transitions

An example where security-bounded transitions occur is when the `mod_selinux` module is used with Apache (which uses bounded domains and transitions for individual requests). When the target domain is not bounded by the source domain (that is, the SELinux policy does not prevent the target domain from executing an action not allowed by the source domain, as done through the `typebounds` statement), then the following error is displayed:

```
type=SELINUX_ERR msg=audit(1245311998.599:17):
op=security_bounded_transition result=denied
oldcontext=system_u:system_r:httpd_t:s0
newcontext=system_u:system_r:guest_webapp_t:s0
```

When this occurs, a bounded transition is requested by the main application domain (such as when a transition is done for threads), but the target domain is not marked as a bounded domain.

Note that this is different from when a bounded domain is given more privileges—in such cases, SELinux will deny the specific permissions when they are invoked, showing AVC denials.

## There's more...

SELinux logging and audit logging is continuously being improved. Work is on the way to make the audit logs easier to parse by scripts and to provide more information. For instance, at the time of writing, a patch has just been accepted to add permissive state information in the AVC logging.

## See also

More in-depth analysis and explanation of AVC messages is handled in *SELinux System Administration*, *Packt Publishing*. More resources related to SELinux audit events are available at the following links:

*   [http://www.selinuxproject.org/page/NB_AL](http://www.selinuxproject.org/page/NB_AL) (including an overview of all possible fields in AVC events)
*   [https://wiki.gentoo.org/wiki/SELinux/Tutorials/Where_to_find_SELinux_permission_denial_details](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Where_to_find_SELinux_permission_denial_details)

# Logging positive policy decisions

On some occasions, the system performs actions that the administrator might not expect, but which are allowed by the SELinux policy, making it harder to debug potential problems. An application might be SELinux-aware, causing its own behavior to depend on the SELinux policy, without actually using the SELinux subsystem to enforce access. The SELinux policy might also be configured to behave differently than expected.

In such situations, it might be important to have SELinux log activities that were actually allowed rather than denied; for instance, logging domain transitions to make sure that a transition has indeed occurred.

## How to do it…

In order to have domain transitions logged, create an SELinux policy by performing the following steps:

1.  Identify the source and target domains to look out for.
2.  Create an SELinux policy that calls the `auditallow` statement on the access vector we want to log:

    ```
    auditallow initrc_t postgresql_t:process transition;
    ```

3.  Build and load the SELinux policy and try to reproduce the situation.
4.  Look at the audit logs and check whether an AVC granted message is displayed:

    ```
    type=AVC msg=audit(1401379369.009:6171): avc:  granted  { transition } for pid=4237 comm="rc" path="/usr/lib64/postgresql-9.3/bin/pg_ctl" dev="dm-3" ino=821490 scontext=system_u:system_r:initrc_t:s0 tcontext=system_u:system_r:postgresql_t:s0 tclass=process 
    ```

## How it works…

Of the many policy statements that SELinux supports, the `auditallow` statement is interesting and does not alter the decisions made by SELinux: having an `auditallow` statement does not allow the action, but rather has the SELinux subsystem log it if it is allowed (through another `allow` statement).

This makes it possible for SELinux policy developers and system administrators to explicitly ask the SELinux subsystem to inform them about decisions taken if the decision is to grant something rather than deny.

Using the `auditallow` statement, we can track SELinux policy decisions and assist in the development of policies and debugging of application behavior, especially when a process is invoked in a very short time frame, as this makes it difficult for administrators to see whether the context of the process is correct (`ps -Z` or by checking the `/proc/<pid>/` contexts).

Some administrators might want to put in some additional logging inside the scripts or commands that they invoke (such as to capture the output of `id -Z`). However, it is very much possible that the SELinux policy does not allow the script to execute the `id` command, let alone show its output or direct its output to a specific logfile.

Enhancing the SELinux policy with additional log types, enabling terminal output, allowing the execution of binaries, and more is quite some overhead just to find out whether the context of the process is as it should be. Using the `auditallow` statement is a great solution to this.

It goes beyond domain transitions, of course. If a file has been changed, and the administrator or engineer is uncertain which process or which context is causing the change, then it is possible to have SELinux audit writes on the file label, as follows:

```
auditallow domain postgresql_etc_t:file write;
```

Thanks to the additional information in the AVC log, we can see which process (PID) running in a particular context (`scontext`) is responsible for writing to the file.

# Looking through SELinux constraints

Some denials are caused by SELinux constraints—additional restrictions imposed by the SELinux policy that are not purely based on the SELinux types, but also on the SELinux role and SELinux user. This is often not clear from the denial.

The `audit2why` application helps in informing developers that a denial came from a constraint violation:

```
~# ausearch -m avc -ts recent | grep type=AVC | audit2why
type=AVC msg=audit(1401134596.932:62843): avc:  denied  { search } for  pid=19384 comm="mount.nfs4" scontext=system_u:system_r:mount_t:s0 tcontext=system_u:object_r:nfs_t:s0 tclass=dir

 Was caused by:
 Policy constraint violation.

 May require adding a type attribute to the domain or type
 to satisfy the constraint.

 Constraints are defined in the policy sources in
 policy/constraints (general), policy/mcs (MCS), and
 policy/mls (MLS).

```

This is, however, not always the case, so we need to find a way to investigate whether denials come from constraint violations too.

## How to do it…

Although SELinux constraints can be queried easily, they are currently difficult to work with. The following approach helps in validating whether a constraint is applicable for a particular AVC denial that is under investigation:

1.  Look through the SELinux policy to see whether the (denied) access has an AVC allow rule or not:

    ```
    ~$ sesearch -s staff_t -t user_home_t -c file -p read -A
    Found 1 semantic av  rules:
     allow staff_t user_home_t : file { … read … };

    ```

2.  Assuming there is an allow rule, see whether there are constraints applicable to the operation. This takes into account the class (in the example, this is `file`) and the permission (in the example, this is `read`):

    ```
    ~$ seinfo --constrain | grep 'constrain .* file .* read' -A 1

    ```

3.  If constraints might exist, look at the attributes of the source and target contexts, as this is usually how constraints are documented in the policy:

    ```
    ~$ seinfo -tstaff_t -x
    ~$ seinfo -tuser_home_t -x

    ```

4.  Inside the SELinux policy, look through the `constraints` file (usually at `${POLICY_LOCATION}/policy/`) and the `mcs` or `mls` file (if the policy uses MCS or MLS), and look for the constraints on the class and permission requested, validating whether there are any expressions concerning the attributes mentioned.

## How it works…

Constraints are currently difficult to validate. Luckily, there aren't many constraints in place, but still, not being able to easily verify and look at the constraints is a nuisance for developers.

The complexity increases as the `seinfo --constrain` output, which is the only available method to query constraints next to reading the sources, has the following drawbacks:

*   It does not provide any name yet on the constraints (so referring to constraints is difficult)
*   It uses **Reverse Polish Notation** (**RPN**), which isn't very user-friendly (although it is powerful for computers, people do not generally read RPN fluently)
*   It shows expanded attributes, so we get huge lists of types, rather than a limited set of attributes

The constraint definitions inside the `constraints`, `mcs`, and `mls` files (which are only accessible through the policy source code) are easier to look at. The following example is from the `constraints` file; constraints from `mcs` and `mls` will use the `mlsconstrain` keyword:

```
constrain process { transition dyntransition noatsecure siginh rlimitinh }
(
..r1 == r2
..or ( t1 == can_change_process_role and t2 == process_user_target )
..or ( t1 == cron_source_domain and t2 == cron_job_domain )
..or ( t1 == can_system_change and r2 == system_r )
..or ( t1 == process_uncond_exempt )
);
```

The controls shown use attributes, which are easier to map with a specific situation. It also shows how flexible constraints can be. Next to pure type-oriented rules (`t1` and `t2`), constraints also work with roles (`r1` and `r2`) and can deal with SELinux users (`u1` and `u2`). The number is used to differentiate between the subject (`1`) and object (`2`).

As an example, in constraint language, saying that something is allowed if the SELinux users are equal, or the SELinux user of the subject is `system_u`, will be documented as follows:

```
(
  u1 == u2
  or ( u1 == system_u)
)
```

The output of the `seinfo --constrain` command has the advantage that it is easy for computer programs to interpret. Computer programs or scripts, which use the output of `seinfo` to visualize constraint information in a tree-like manner, can be created.

The following GraphViz-generated graph shows the UBAC constraints applicable to file reads, showing only the user domains and the `user_home_t` types (to not overload the graph):

![How it works…](img/9669OS_08_01.jpg)

This graph shows how the UBAC constraints are constructed. File reads are prohibited (regardless of the type enforcement rules that are made in the policy), unless they match one of the rules shown in the graph, which are as follows:

*   The SELinux user of the subject (domain) and object (resource) are the same
*   The SELinux user of the subject is `system_u`
*   The SELinux user of the object is `system_u`
*   The SELinux type of the subject does not match any of the mentioned types (only a subset is shown in the drawing)
*   The SELinux type of the object does not match any of the mentioned types (only a subset is shown in the drawing)
*   The SELinux type of the subject is `sysadm_t`

## See also

More information on SELinux constraints can be found at the following resources:

*   [https://wiki.gentoo.org/wiki/SELinux/Constraints](https://wiki.gentoo.org/wiki/SELinux/Constraints)
*   [http://www.selinuxproject.org/page/ConstraintStatements](http://www.selinuxproject.org/page/ConstraintStatements)

# Ensuring an SELinux rule is never allowed

It is possible to include statements in the SELinux policy that ensure that a particular access vector cannot be allowed, not even by enhancing the SELinux policy later. This is done with the `neverallow` statement.

## How to do it…

To include the `neverallow` statements in the policy and enforce them, go through the following steps:

1.  In `/etc/selinux/semanage.conf`, enable support for the `neverallow` statements by setting the `expand-check` variable to `1`:

    ```
    expand-check=1
    ```

2.  Create an SELinux policy in which the access vectors that should be explicitly forbidden are listed. Consider the following instance:

    ```
    neverallow user_t system_mail_t:process transition;
    ```

3.  Build and load the policy.
4.  Generate another policy that will allow the statement and attempt to load it:

    ```
    ~$ semodule -i mytest.pp
    libsepol.check_assertion_helper:  neverallow violated by allow user_t system_mail_t:process { transition };
    libsemanage.semanage_expand_sandbox: Expand module failed
    semodule: Failed!

    ```

## How it works…

Not all distributions enable the assertion checks by default as they incur some performance penalty during policy builds. Some distributions might even have policy incompatibilities due to this, because if the assertions are disabled, then the `neverallow` statements are never processed: the `neverallow` statement isn't really a policy decision, but more a rule that influences loading of new policies, and is enforced by the policy linker (which combines the various policy modules in one final policy binary). As can be deduced from the (failure) output, the `neverallow` statements are implemented as assertions.

Some `neverallow` statements are available as part of the base policy. For instance, the following statement ensures that only the domains with the `selinux_unconfined_type` or `can_load_policy` attribute set can actually load an SELinux policy:

```
neverallow ~{ selinux_unconfined_type can_load_policy } security_t:security load_policy;
```

This example uses the negation operator (`~`), which means *all types except those mentioned*.

Unlike constraints (that can also be used to implement restrictions), the `neverallow` statements help by not accepting any policy that will violate the rule. It is also possible to add the `neverallow` rules through modules, unlike constraints that need to be part of the base SELinux policy (and as such, are governed by Linux distribution, an upstream policy, or developers that manage complete policies rather than individual SELinux policy modules).

The `expand-check` variable in `/etc/selinux/semanage.conf` tells the SELinux user space libraries that the assertion has to be checked. If this variable is set to `0`, then the `neverallow` statements have no impact on the policy and its loading whatsoever.

# Using strace to clarify permission issues

The `strace` application is a popular debugging application on Linux systems. It allows developers and administrators to look at various system calls made by an application. As SELinux often has access controls on specific system calls, using `strace` can prove to be very useful in debugging permission issues.

## How to do it…

To properly use `strace`, follow the next set of steps:

1.  Enable the `allow_ptrace` Boolean:

    ```
    ~# setsebool allow_ptrace on

    ```

2.  Run the application with `strace`:

    ```
    ~$ strace -o strace.log -f -s 256 tmux

    ```

3.  In the resulting logfile, look for the error message that needs to be debugged.

## How it works…

The `allow_ptrace` Boolean (on some distributions, the inverse Boolean called `deny_ptrace` is available) needs to be toggled so that the domain that calls `strace` can use `ptrace` (the method that `strace` uses to view system calls) against the target domain. As the `ptrace` method can be a security concern (it allows reading target process' memory, for instance), it is, by default, disabled.

Once an application has been executed through the `strace` application, the logfile will contain all relevant system call information. Of course, on larger applications, or on daemons, this logfile can become massive, so it makes sense to limit the `strace` operation towards a particular subset of system calls, as shown in the following command:

```
~$ strace -e open,access -o strace.log -f -s 256 tmux

```

In this example, only the `open` and `access` system calls are looked at.

In the resulting logfile, the SELinux permission usually issues results in failed system calls with an `EACCES (Permission denied)` error code:

```
7313  stat("/", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
7313  stat("/home", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
7313  stat("/home/swift", {st_mode=S_IFDIR|0755, st_size=12288, ...}) = 0
7313  stat("/home/swift/.pki", {st_mode=S_IFDIR|0700, st_size=4096, ...}) = 0
7313  stat("/home/swift/.pki/nssdb", {st_mode=S_IFDIR|0700, st_size=4096, ...}) = 0
7313  statfs("/home/swift/.pki/nssdb", 0x3c3cab6fa50) = -1 EACCES (Permission denied)
```

Although an AVC denial will also be shown for most accesses, these denials often do not give a complete picture as to at what stage a denial is in. By using `strace`, we can follow the logic that the application performs.

Sometimes, it isn't obvious why a failure occurs. In this case, it might be interesting to run the application twice—once in enforcing mode and once in permissive mode—and look at the differences in the `strace` logs.

# Using strace against daemons

The `strace` application not only makes sense for command-line applications but also for daemons. A popular approach to debugging daemons is to start them from the command line, possibly with a specific debug flag, so that the daemon doesn't detach and run in the background. However, this is often not possible on SELinux: the policy will not allow the daemon to run as a command-line foreground process.

## How to do it…

The approach to use `strace` against daemons is similar as with command lines, focusing on the process ID rather than the command:

1.  Find out what the process ID of the daemon is:

    ```
    ~$ pidof postgres
    2557

    ```

2.  Use `strace` to attach to the running process:

    ```
    ~$ strace -o strace.log -f -s 256 -p 2557

    ```

3.  Specify which system calls to watch out for. For instance, permission issues while binding or connecting to ports or sockets can be filtered as follows:

    ```
    ~$ strace -e poll,select,connect,recvfrom,sendto -o strace.log -f -s 256 -p 2557

    ```

4.  Press *Ctrl* + *C* to interrupt the `strace` session; don't worry, the daemon will continue to run in the background, unharmed.

## How it works…

A popular approach to debugging daemons, which is to start the daemon in the foreground from the command line, often does not work on SELinux systems:

```
~$ postgres -D /etc/postgresql-9.3 --data-directory=/srv/pgsql/data
LOG:  could not bind IPv6 socket: Permission denied
WARNING: could not create listen socket for "localhost"
FATAL: could not create any TCP/IP sockets

```

If a user has the rights to execute the daemon binary directly (which isn't default either), then the daemon usually runs with the permissions of the user domain—who hardly has the privileges needed to run the daemon—as there is no transition from the user domain to the daemon domain.

By using `strace` against the daemons, it is possible to debug them in more detail. The `strace` application will bind to the process (using the `ptrace` method) and be notified of every system call that the daemon performs. The `-f` option also ensures that new processes that the daemon launches (for instance, worker processes) are also looked at by `strace`.

To end the `strace` session, it is enough to kill the `strace` session or interrupt it with *Ctrl* + *C*. The daemon itself is left untouched.

## There's more...

Many other system analysis tools, which can be used in a very similar manner, exist. Some examples are SystemTap and Sysdig, with a port of DTrace to Linux being actively developed.

## See also

The following resources cover the use of `strace`, SystemTap, and Sysdig in more detail:

*   [http://www.dedoimedo.com/computers/strace.html](http://www.dedoimedo.com/computers/strace.html)
*   [http://www.thegeekstuff.com/2011/11/strace-examples/](http://www.thegeekstuff.com/2011/11/strace-examples/)
*   [http://www.sourceware.org/systemtap/](http://www.sourceware.org/systemtap/)
*   [http://www.sysdig.org/wiki/](http://www.sysdig.org/wiki/)

# Auditing system behavior

Another approach to debugging application behavior is through Linux auditing, especially when it is not clear which process is responsible for performing a specific action, as this might make SELinux development a lot more difficult. When developers do not know which domain(s) they need to update privileges for, or do not know how exactly a resource is created, then the Linux audit subsystem can help.

With the Linux auditing subsystem, administrators can enable rules to log activities. In the audit log, the SELinux context of the subject (process) is shown as well, allowing SELinux developers to properly identify the domain to work with.

## How to do it…

Let's look at how we can ask the Linux audit subsystem which process is responsible for creating a particular directory in a user's home directory through the following steps:

1.  As the root Linux user (and in an SELinux role with sufficient privileges), tell the audit subsystem to log all write- and attribute-changing operations inside the user's home directory:

    ```
    ~# auditctl -w /home/john/ -p wa -k policydev

    ```

2.  Perform the necessary action(s) to trigger the behavior that needs to be debugged.
3.  Query the audit subsystem for the recent audit events with the `policydev` key:

    ```
    ~# ausearch -ts recent -k policydev

    ```

4.  Later, disable the audit rule again so that the audit logs are not cluttered with development-related events:

    ```
    ~# auditctl -W /home/john/ -p wa -k policydev

    ```

## How it works…

The Linux audit subsystem uses audit rules to identify which activities need to be logged to the audit log. The rules can be manipulated using the `auditctl` command (audit control).

In our example, a rule was added for the `/home/john/` path (`-w /home/john`) for which the write and attribute changes (`-p wa`) are logged. The events are tagged, so to speak, with a key called `policydev`. Administrators can choose this key freely. Its purpose is to structure audit events and simplify search queries.

When the `auditctl` command is invoked, the rule is immediately active, so after executing the test, audit events will be displayed as follows:

```
time->Sun Jun  8 11:16:47 2014
type=PATH msg=audit(1402219007.623:80705): item=1 name=".dcinforc" inode=8364 dev=fd:0c mode=040755 ouid=475395 ogid=475395 rdev=00:00 obj=user_u:object_r:user_home_t:s0 nametype=CREATE
type=PATH msg=audit(1402219007.623:80705): item=0 name="/home/john" inode=229 dev=fd:0c mode=040700 ouid=475395 ogid=475395 rdev=00:00 obj=user_u:object_r:user_home_dir_t:s0 nametype=PARENT
type=CWD msg=audit(1402219007.623:80705):  cwd="/home/john"
type=SYSCALL msg=audit(1402219007.623:80705): arch=c000003e syscall=83 success=yes exit=0 a0=7fff33d50330 a1=1ff a2=7fff33d50330 a3=a items=2 ppid=23132 pid=23929 auid=475395 uid=475395 gid=475395 euid=475395 suid=475395 fsuid=475395 egid=475395 sgid=475395 fsgid=475395 tty=pts3 ses=11203 comm="java" exe="/usr/bin/java" subj=user_u:user_r:java_t:s0 key="policydev"
```

The logs show that it is a `java` process that is responsible for creating a directory called `.dcinforc/` in the user's home directory. The important fields to consider here are the `nametype=CREATE` (which tells us that an object was created) and `syscall=83` (informing us which system call was trapped by the audit subsystem—in this case, the `mkdir` system call) fields, and of course the `subj=` and `obj=` parameters.

From the example, we can see that there are two distinct `obj=` parameters:

*   The first, `obj=user_u:object_r:user_home_t:s0`, is mentioned for the created directory, and it tells us what label the newly created directory received
*   The second, `obj=user_u:object_r:user_home_dir_t:s0`, is mentioned for the parent directory (`nametype=PARENT`), informing us what the label of the directory in which `.dcinforc/` is created is

Now, this is just an example of creating directories, but the audit system can trap many types of activities. This is where the `syscall=` field becomes important. This field tells us what specific system call was trapped and logged by the audit subsystem.

A list of system calls and their associated numbers can be found in the proper `C` header file. For instance, the `/usr/include/asm/unistd_64.h` file (referenced indirectly through `/usr/include/syscalls.h`) contains the following code:

```
#define __NR_rename  82  __SYSCALL(__NR_rename, sys_rename)
#define __NR_mkdir  83  __SYSCALL(__NR_mkdir, sys_mkdir)
#define __NR_rmdir  84  __SYSCALL(__NR_rmdir, sys_rmdir)
```

Through this, we know that the directory was created using the `mkdir` system call and not by any other means (such as creating the directory as a different one first and then renaming it).

## There's more...

The audit subsystem receives the rules it needs to follow up on at boot. Most Linux distributions offer a file called `audit.rules` inside `/etc/audit/`, which contains various commands, locations, and system calls that need to be trapped and logged. This file is then read at boot time by the audit daemon `init` script.

If we need to have certain rules loaded automatically—and not just for the duration of a short test—then it is recommended to add the rules to this `audit.rules` script, together with the appropriate comment explaining why this needs to be trapped.

Now, we only used path-based auditing capabilities in the example. The Linux audit subsystem, however, can do much more than just that. For instance, it is possible to audit particular system calls. This allows administrators to keep a close eye on suspicious system call usages, such as the use of `unshare` (which is used for Linux namespaces):

```
~# auditctl -a entry,always -S unshare -k namespace_suspect

```

## See also

*   A good set of default audit rules to work with is mentioned in the CISecurity Benchmark for Red Hat Enterprise Linux, available at [https://benchmarks.cisecurity.org/](https://benchmarks.cisecurity.org/)