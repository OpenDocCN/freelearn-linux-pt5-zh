# Chapter 5. Using SELinux

Here is an overview of the recipes presented in this chapter:

*   Changing file contexts
*   Configuring SELinux booleans
*   Configuring SELinux port definitions
*   Troubleshooting SELinux
*   Creating SELinux policies
*   Applying SELinux policies

# Introduction

SELinux is a Linux kernel module that allows supporting **mandatory access control** (MAC) security policies. The Red Hat implementation of SELinux combines **role-based access control** (**RBAC**) with **type enforcement** (**TE**). Optionally, **multilevel security** (**MLS**) is also available but isn't widely used as it implements fewer policies than the default Red Hat SELinux policies.

SELinux is enabled by default in RHEL 7 and supported for all software packaged by Red Hat.

The recipes presented in this chapter will not only provide you with a solid base to troubleshoot SELinux issues and fix them, but also a peek into how to create your own SELinux policies.

# Changing file contexts

Files and processes are labeled with a SELinux context, which contains additional information about a SELinux user, role type, and level. This information is provided by the SELinux kernel module to make access control decisions.

The SELinux user, a unique identity known by the SELinux policy, is authorized for a number of roles.

SELinux roles, as we already alluded to before, are attributes of SELinux users and part of the RBAC SELinux policy. SELinux roles are authorized for SELinux domains.

SELinux types define the type for files and domain for processes. SELinux policies define access between types and other files and processes. By default, if there is no specific rule in the SELinux policy, access is denied.

The SELinux level is only used when the SELinux type is set to MLS and should be avoided altogether on anything other than servers. This set of policies doesn't cover the same domains as defined by the default Red Hat SELinux policy. The SELinux level is an attribute of MLS and **multi-category security** (**MCS**).

## Getting ready

All files and processes on a system are labeled to represent security-relevant information. This information is called the SELinux context. To view the contexts of files (and directories), execute the following:

```
~# ls -Z
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 file
~#

```

## How to do it…

You can temporarily change the context of a file (or files) or permanently change their context. The first option allows easy troubleshooting if you need to figure out whether changing the context solves your problem. Persistent changes are mostly used when your applications refer to data that is not in the standard location—for example, if your web server serves data from `/srv/www`.

### Temporary context changes

Temporary SELinux context changes remain until the file, or the filesystem that the file resides on, is relabeled.

To change the SELinux user of a file, execute the following:

```
~# chcon --user <SELinux user> <filename>

```

To change the SELinux role of a file, execute the following:

```
~# chcon --role <SELinux role> <filename>

```

To change the SELinux type of a file, execute the following:

```
~# chcon --type <SELinux typs> <filename>

```

### Persistent file context changes

Changing the application data location doesn't automatically modify SELinux contexts to allow your application to access this data.

To permanently relabel files or directories, perform the following:

1.  Change the SELinux user for your files or directories via this command:

    ```
    ~# semanage fcontext -a --seuser <SELinux user> <filename|dirname>

    ```

2.  Change the SELinux type of your files or directories by running the following:

    ```
    ~# semanage fcontext -a --type <SELinux type> <filename|dirname>

    ```

3.  Finish with this command line by applying the directive to the `files/directories`:

    ```
    ~# restorecon <filename|dirname>

    ```

## There's more…

To show all the available SELinux users, execute the following:

```
~# semanage user -l

```

![There's more…](img/00041.jpeg)

Alternatively, you can install the `setools-console` package and run the following:

```
~# seinfo -u

```

![There's more…](img/00042.jpeg)

To show all the available SELinux types, install the `setools-console` package and run the following:

```
~# seinfo -t

```

![There's more…](img/00043.jpeg)

To show the available SELinux roles, install the `setools-console` package and run the following:

```
~# seinfo -r

```

![There's more…](img/00044.jpeg)

The `semanage` tool doesn't have an option to include all files recursively, but there is a solution to this. The filename or dirname you specify is actually a regular expression filter. So, for example, if you want to recursively include all the files in `/srv/www`, you could specify `"/srv/www(/.*)?"`.

### Tip

For now, there's no way to change the SELinux role using `semanage`. A way to get around this is to change the SELinux user or type using `semanage` and then edit it, as follows: `/etc/selinux/targeted/contexts/files/file_contexts.local`.

Here's a wrong SELinux context example of an AVC denial report found in the `audit.log` file:

```
type=AVC msg=audit(1438884962.645:86): avc:  denied  { open } for  pid=1283 comm="httpd" path="/var/www/html/index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file
```

This command can be explained as follows:

| Commands | Description |
| --- | --- |
| `type=AVC` | This is the log type |
| `msg=audit(1438884962.645:86)` | This is the log entry timestamp |
| `avc` | This is a repetition of the log type |
| `denied` | This states whether enforcing is enabled |
| `{ open }` | This is a permission that causes AVC denial |
| `for pid=1283` | This is the process ID |
| `comm="httpd"` | This is the process command |
| `path="/var/www/html/index.html"` | This is the path that is accessed |
| `dev="dm-5"` | This blocks the device that the preceding file is located on |
| `ino=1089` | This is the inode of the preceding file |
| `scontext=system_u:system_r:httpd_t:s0` | This is the source SELinux context |
| `tcontext=system_u:object_r:user_home_t:s0` | This is the target SELinux context |
| `tclass=file` | This is the target SELinux class |

## See also

Refer to the man page for *chcon (1)* and *semanage-fcontext (8)* for more information.

# Configuring SELinux booleans

SELinux booleans allow you to change the SELinux policy at runtime without the need to write additional policies. This allows you to change the policy without the need for recompilation, such as allowing services to access NFS volumes.

## How to do it…

This is the way to temporarily or permanently change SELinux booleans.

### Listing SELinux booleans

For a list of all booleans and an explanation of what they do, execute the following:

```
~# semanage boolean -l

```

![Listing SELinux booleans](img/00045.jpeg)

Now, let's try to get the value of a particular SELinux boolean. It is possible to get the value of a single SELinux boolean without the use of additional utilities, such as **grep** and/or **awk**. Simply execute the following:

```
~# getsebool <SELinux boolean>

```

This shows you whether or not the boolean is set. Here's an example:

```
~# getsebool virt_use_nfs
virt_use_nfs --> off
~#

```

### Changing SELinux booleans

To set a boolean value to a particular one, use the following command:

```
~# setsebool <SELinux boolean> <on|off>

```

Here's an example command:

```
~# setsebool virt_use_nfs on

```

This command allows you to change the value of the boolean, but it is not persistent across reboots. To allow persistence, add the `-P` option to the command line, as follows:

```
~# setsebool -P virt_use_nfs on

```

## There's more…

If you would like a list of all the bare bones of SELinux booleans and their values, `getsebool -a` is an alternative, as follows:

```
~# getsebool -a

```

![There's more…](img/00046.jpeg)

Managing SELinux booleans can be rather complex as there are a lot of booleans, and their names are not always simple to remember. For this reason, the `setsebool`, `getsebool`, and `semanage` tools come with tab completion. So, whenever you type any boolean name, you can use the `tab` key to complete or display the possible options.

Here's an example of an AVC denial report found in the `audit.log` file that can be solved by enabling a boolean:

```
type=AVC msg=audit(1438884483.053:48): avc:  denied  { open } for  pid=1270 comm="httpd" path="/nfs/www/html/index.html" dev="0:38" ino=2717909250 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:nfs_t:s0 tclass=file
```

This is an example of a service (`httpd` in this case) accessing a file located on an NFS share, which is disabled by default.

This can be allowed by setting the `httpd_use_nfs` boolean to "`on`".

# Configuring SELinux port definitions

SELinux also controls access to your TCP/IP ports. If your application is confined by SELinux, it will also deny access to your ports when starting up the application.

This recipe will show you how to detect which ports are used by a particular SELinux type and change it.

## How to do it…

Let's allow the HTTP daemon to listen on the nonstandard port `82` through the following steps:

1.  First, look for the ports that are accessed by HTTP via these commands:

    ```
    ~# semanage port -l |grep http
    http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
    http_cache_port_t              udp      3130
    http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
    pegasus_http_port_t            tcp      5988
    pegasus_https_port_t           tcp      5989
    ~#

    ```

    The SELinux port assignment we're looking for is `http_port_t`. As you can see, only the displayed ports (`80`, `81`, `443`, `488`, `8008`, `8009`, `8443`, and `9000`) are allowed to be used to listen on by any process that is allowed to use the `http_port_t` type.

2.  Add port `82` to the list of allowed ports, as follows:

    ```
    ~# semanage port -a -t http_port_t -p tcp 82
    ~#

    ```

3.  Next, verify the port assignment, as follows:

    ```
    ~# semanage port -l |grep ^http_port_t
    http_port_t                    tcp      82, 80, 81, 443, 488, 8008, 8009, 8443, 9000
    ~#

    ```

## There's more…

In this example, there is reference to the HTTP daemon as the SELinux policy governing HTTP daemons is implemented not only for the Apache web server, but also for Nginx. So, as long as you use the packages provided by Red Hat, the SELinux policies will be used correctly.

Take a look at the following example of an AVC denial report found in the `audit.log` file that is caused because the domain is not allowed to access a certain port:

```
type=AVC msg=audit(1225948455.061:294): avc: denied { name_bind } for pid=4997 comm="httpd" src=82 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=system_u:object_r:port_t:s0 tclass=tcp_socket
```

This AVC denial shows that the `httpd` daemon attempted to listen (`name_bind`) on port `82` but was prohibited by SELinux.

# Troubleshooting SELinux

Troubleshooting SELinux is not as straightforward as it may seem as at the time of writing this book, there is no integration with SELinux to return SELinux-related events back to the applications. Usually, you will find that access is denied with no further description of it in log files.

## Getting ready

Make sure that `setroubleshoot-server` and `setools-console` are installed by executing the following command:

```
~# yum install -y setroubleshoot-server setools-console

```

If you have X server installed on your system, you can also install the GUI, as follows:

```
~# yum install -y setroubleshoot

```

Make sure that `auditd`, `rsyslog`, and `setroubleshootd` are installed and running before reproducing the issue.

## How to do it…

There are several ways to detect SELinux issues.

This is a classic issue where the SELinux context of a file is incorrect, causing the application trying to access the file to fail.

In this case, the context of `/var/www/html/index.html` is set to `system_u:object_r:user_home_t:s0` instead of `system_u:object_r:httpd_sys_content_t:s0`, causing `httpd` to throw a `404`. Take a look at the following command:

```
# ls -Z /var/www/html/index.html
-rw-r--r--. apache apache system_u:object_r:user_home_t:s0 /var/www/html/index.html
~#

```

### audit.log

Use the following command to look for denied or failed entries in the audit log:

```
~# egrep 'avc.*denied' /var/log/audit/audit.log
ype=AVC msg=audit(1438884962.645:86): avc:  denied  { open } for  pid=1283 comm="httpd" path="/var/www/html/index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file
~#

```

### syslog

You can look for SELinux messages in `/var/log/messages` via the following command:

```
~# grep 'SELinux is preventing' /var/log/messages
Aug  6 20:16:03 localhost setroubleshoot: SELinux is preventing /usr/sbin/httpd from read access on the file index.html. For complete SELinux messages., run sealert -l dc544bde-2d7e-4f3f-8826-224d9b0c71f6
Aug 6 20:16:03 localhost python: SELinux is preventing /usr/sbin/httpd from read access on the file index.html.
~#

```

### ausearch

Use the audit search tool to find SELinux errors, as follows:

```
~# ausearch -m avc
time->Thu Aug  6 20:16:02 2015
type=SYSCALL msg=audit(1438884962.645:86): arch=c000003e syscall=2 success=yes exit=25 a0=7f1bcfb65670 a1=80000 a2=0 a3=0 items=0 ppid=1186 pid=1283 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1438884962.645:86): avc:  denied  { open } for  pid=1283 comm="httpd" path="/var/www/html/index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file
type=AVC msg=audit(1438884962.645:86): avc:  denied  { read } for  pid=1283 comm="httpd" name="index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file
~#

```

Once we restore the context of `/var/www/html/index.html` to its original, the file is accessible again. Take a look at the following commands:

```
~# restorecon /var/www/html/index.html
~# ls -Z /var/www/html/index.html
-rw-r--r--. apache apache system_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
~#

```

## There's more…

It's not always easy to determine whether a file has the correct context. To view the actual SELinux context and compare it to what it should be without modifying anything, execute this command:

```
~# matchpathcon -V index.html
index.html has context system_u:object_r:user_home_t:s0, should be system_u:object_r:httpd_sys_content_t:s0
~#

```

This tells you what the current context is and what it should be.

As you can see in the preceding syslog example, the output comes with the following command:

```
... run sealert -l dc544bde-2d7e-4f3f-8826-224d9b0c71f6

```

This command provides you with a richer description of the problem:

```
~# sealert -l dc544bde-2d7e-4f3f-8826-224d9b0c71f6
SELinux is preventing /usr/sbin/httpd from read access on the file index.html.

*****  Plugin catchall_boolean (89.3 confidence) suggests   ******************

If you want to allow httpd to read user content
Then you must tell SELinux about this by enabling the 'httpd_read_user_content' boolean.
You can read 'None' man page for more details.
Do
setsebool -P httpd_read_user_content 1

*****  Plugin catchall (11.6 confidence) suggests   **************************

If you believe that httpd should be allowed read access on the index.html file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# grep httpd /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp

Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context                system_u:object_r:user_home_t:s0
Target Objects                index.html [ file ]
Source                        httpd
Source Path                   /usr/sbin/httpd
Port                          <Unknown>
Host                          localhost.localdomain
Source RPM Packages           httpd-2.4.6-31.el7.rhel.x86_64
Target RPM Packages 
Policy RPM                    selinux-policy-3.13.1-23.el7_1.7.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Permissive
Host Name                     localhost.localdomain
Platform                      Linux localhost.localdomain
 3.10.0-229.4.2.el7.x86_64 #1 SMP Wed May 13
 10:06:09 UTC 2015 x86_64 x86_64
Alert Count                   1
First Seen                    2015-08-06 20:16:02 CEST
Last Seen                     2015-08-06 20:16:02 CEST
Local ID                      dc544bde-2d7e-4f3f-8826-224d9b0c71f6

Raw Audit Messages
type=AVC msg=audit(1438884962.645:86): avc:  denied  { read } for  pid=1283 comm="httpd" name="index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file

type=AVC msg=audit(1438884962.645:86): avc:  denied  { open } for  pid=1283 comm="httpd" path="/var/www/html/index.html" dev="dm-5" ino=1089 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:user_home_t:s0 tclass=file

type=SYSCALL msg=audit(1438884962.645:86): arch=x86_64 syscall=open success=yes exit=ENOTTY a0=7f1bcfb65670 a1=80000 a2=0 a3=0 items=0 ppid=1186 pid=1283 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: httpd,httpd_t,user_home_t,file,read
~#

```

This will actually give you more details about the problem at hand, and it will also make a couple of suggestions. Of course, in this case, the real solution is to restore the SELinux context of the file.

If you have installed a graphical desktop environment, you will get a notification each time your system encounters an "AVC denied" alert:

![There's more…](img/00047.jpeg)

Clicking on the icon will present you with the following dialog:

![There's more…](img/00048.jpeg)

Clicking on the **Troubleshoot** button will provide you with additional information and a (or multiple) possible solution(s) for your problem, as shown in the following screenshot:

![There's more…](img/00049.jpeg)

In this case, the first option (the one marked with a green line) is the correct solution.

Some AVC denial messages may not be logged when SELinux denies access. Applications and libraries regularly probe for more access than is actually required to perform their tasks. In order to not flood the audit logs with these kinds of messages, the policy can silence the AVC denials that are without permissions using `dontaudit` rules. The downside of this is that it may make troubleshooting SELinux denials more difficult.

To disable the `dontaudit` rules, execute the following command:

```
~# semanage dontaudit off

```

This will disable the `dontaudit` rules and rebuild the SELinux policy.

It is advisable to reenable the `dontaudit` rules when you're done troubleshooting as this may flood your disks. You can do this by executing the following command:

```
~# semanage dontaudit on

```

To get a full list of `dontaudit` rules, run the following:

```
~# sesearch --dontaudit
Found 8361 semantic av rules:
 dontaudit user_ssh_agent_t user_ssh_agent_t : udp_socket listen ;
 dontaudit openshift_user_domain sshd_t : key view ;
 dontaudit user_seunshare_t user_seunshare_t : process setfscreate ;
 dontaudit ftpd_t selinux_config_t : dir { getattr search open } ;
 dontaudit user_seunshare_t user_seunshare_t : capability sys_module ;
 dontaudit xguest_dbusd_t xguest_dbusd_t : udp_socket listen ;
 dontaudit tuned_t tuned_t : process setfscreate ;
...
~#

```

If you know the domain that you wish to check for `dontaudit` rules, add the `-s` argument followed by the domain, as shown here:

```
~# sesearch --dontaudit -s httpd_t
Found 182 semantic av rules:
 dontaudit httpd_t snmpd_var_lib_t : file { ioctl read write getattr lock open } ;
 dontaudit domain rpm_var_lib_t : file { ioctl read write getattr lock append } ;
 dontaudit httpd_t snmpd_var_lib_t : dir { ioctl read getattr lock search open } ;
 dontaudit domain rpm_var_lib_t : dir getattr ;
 dontaudit httpd_t snmpd_var_lib_t : lnk_file { read getattr } ;
...
~#

```

## See also

Take a look at the man page for *ausearch (8)*, *matchpathcon (8)*, and *sealert (8)* for more information.

# Creating SELinux policies

In some cases, you'll need to create a new SELinux policy—for instance, when installing a piece of software from source. Although I do not recommend installing software from source on enterprise systems, this is sometimes your only option for company-developed software.

It is then time to create your own SELinux policy.

## Getting ready

For this recipe, you need to have `policycoreutils-python` installed.

## How to do it…

We'll use the `denied` entries in the `audit.log` log file to build our SELinux policy with `audit2allow`.

In this recipe, we'll use the same example as in the previous recipe: the SELinux context of `/var/www/html/index.html` that is changed to `system_u:object_r:user_home_t:s0`. Perform the following steps:

1.  First, create a human readable policy for verification via the following command:

    ```
    ~# egrep 'avc.*denied' /var/log/audit/audit.log |audit2allow -m example_policy

    module example_policy 1.0;

    require {
     type httpd_t;
     type user_home_t;
     class file { read open };
    }

    #============= httpd_t ==============

    #!!!! This avc can be allowed using the boolean 'httpd_read_user_content'
    allow httpd_t user_home_t:file { read open };
    ~#

    ```

2.  When this policy is validated, you can create a compiled SELinux policy file, as follows:

    ```
    egrep 'avc.*denied' /var/log/audit/audit.log |audit2allow -M example_policy
    ******************** IMPORTANT ***********************
    To make this policy package active, execute:

    semodule -i example_policy.pp
    ~#

    ```

## How it works…

When you generate a module package, two files are created: a type enforcement file (`.te`) and a policy package file (`.pp`) file. The `te` file is the human readable policy as generated using `audit2allow -m`.

The `pp` file is the SELinux policy module package, which will later be used to enable the new policy.

## There's more…

If you believe you have discovered a bug in an existing SELinux policy, you'll need to produce a type enforcing and policy package file to report with Red Hat Bugzilla.

It's important to make sure that you only parse the correct `AVC denial` entries with `audit2allow` as it may result in more access than required. It's a good idea to pipe the `AVC denial` entries to a temporary file and remove what is not needed before you parse the file with `audit2allow`.

If the policy you generate in this way is not exactly what you need, you can always edit the generated `te` policy file, and when you're done, compile a new policy file using the `te` policy file. You can do this as follows:

1.  Build a binary policy module out of the policy file through this command:

    ```
    ~# checkmodule -M -m -o example_policy.mod example_policy.te
    checkmodule:  loading policy configuration from example_policy.te
    checkmodule:  policy configuration loaded
    checkmodule:  writing binary representation (version 17) to example_policy.mod
    ~#

    ```

2.  Create the SELinux policy module package by executing the following:

    ```
    ~# semodule_package -o example_policy.pp -m example_policy.mod
    ~#

    ```

## See also

Take a look at the man page for *audit2allow(1)* for more options on creating a policy

To report bugs, go to [https://bugzilla.redhat.com/](https://bugzilla.redhat.com/).

# Applying SELinux policies

We've learned how to create SELinux policies in the previous recipe. This recipe will show you how to apply your newly created SELinux policies.

## Getting ready

In order to apply a policy, we need a policy package file (`pp`). This can be obtained by parsing AVC denials to `audit2allow` or compiling your own policy package file, as explained in the *Create SELinux policies* recipe.

## How to do it...

Follow these steps:

1.  Activate the policy (this can take quite a while, depending on the number of policies applied to your system) by running the following command:

    ```
    ~# semodule -i example_policy.pp
    ~#

    ```

2.  Next, verify that the policy is actually activated via these commands:

    ```
    ~# semodule -l |grep example_policy
    example_policy  1.0
    ~#

    ```

## How it works…

When executing the `semodule` command, the policy file is copied to `/etc/selinux/targeted/modules/active/modules/`, and the complete SELinux policy is recompiled and applied.

### Tip

Be careful when applying custom-made policies as these may allow more access than required!

## There's more…

To remove policies, execute the following command:

```
~# semodule -r example_policy
~#

```

This is particularly practical when you want to test the effect with and without the policy.

There's also a way to upgrade the module without removing it first, which is as follows:

```
~# semodule -u example_policy
~#

```

## See also

Refer to the man page for *semodule (8)* for more information.