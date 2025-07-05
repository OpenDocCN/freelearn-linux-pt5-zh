# Bash Vulnerability Patching

The following recipes will be covered in this chapter:

*   Understanding the Bash vulnerability – Shellshock
*   Security issues – Shellshock
*   Patch management system
*   Integrating patches on the Linux network
*   Other well-known Linux vulnerabilities

# Understanding the Bash vulnerability – Shellshock

**Shellshock** or **Bashdoor** is a vulnerability that occurs in most versions of Linux and Unix operating systems. It was discovered on September 12, 2014, and affects all distributions of Linux using Bash shell. Shellshock vulnerability makes it possible to execute commands remotely using environment variables.

# Getting ready

To understand Shellshock, we will need a Linux system using a version of Bash prior to 4.3, which is vulnerable to this bug.

# How to do it...

In this section, we will see how to set up our system to understand the internal details of Shellshock vulnerability:

1.  The first step to perform will be to check the version of Bash on the Linux system so that we can find out if our system is vulnerable to **Shellshock**. To check our version of Bash, run the following command:

![](img/e5fc2b9d-74cc-4060-b673-24920443b892.png)

Bash versions through 4.3 have been reported to be vulnerable to Shellshock. For our example, we are using Ubuntu 12.04 LTS, desktop version. From the output in the preceding screenshot, we can understand that this system is vulnerable.

2.  Now, let's check if the vulnerability actually exists or not. To do so, we run the following code:

![](img/43cdcd58-d070-4c1a-9a61-44d2b848310c.png)

Once we run the preceding command, if the output has `shellshock` printed, it confirms the vulnerability.

3.  Now, let's understand the insights of the vulnerability. For this, first, we need to understand the basics of Bash shell variables.

4.  If we want to create a variable named `testvar` in bash and store a value of `'shellshock'` in it, we must run the following command:

```
 testvar="shellshock'
```

Now, if we wish to print the value of this variable, we can use the `echo` command, as follows:

```
          echo $testvar
```

5.  Now, we will open a child process of bash by running the `bash` command. Then, one again, we to try to print the value of the variable `testvar` in the child process:

![](img/2ecf64cc-451d-46f1-8456-6e5e5d3c133a.png)

We can see that we are not able to get any output when we try to print the value in the child process.

6.  Now, we will try to do the same thing by using environment variables of bash. When we start a new shell session of bash, a few variables are available for use, and these are called **environment variables**.
7.  To make our `testvar` variable an environment variable, we will export it. Once exported, we can use it in the child shell also, as follows:

![](img/304f4052-44cc-44ae-a6a8-ba01fe3f8ed5.png)

8.  As we have defined variables and then exported them, in the same way, we can define a function and export it as well, in order to make it available in a child shell. The following steps show how to define a function and export it:

![](img/aa4c6602-48ed-4937-b57a-a3e30cad4a99.png)

We can see in the preceding example that the function `x` has been defined and it has been exported using the `-f` flag.

9.  Now, let's define a new variable, name it `testfunc`, and assign its value, as follows:

```
    testfunc='() { echo 'shellshock';}'
```

The previously defined variable can be accessed in the same way as a regular variable is:

```
    echo $testfunc
```

10.  Next, we will export this variable to make to an environment variable and then try to access it from the child shell, as shown in the following screenshot:

![](img/e3bfc087-e605-45b0-9640-0a15c6d37310.png)

We can see something unexpected in the preceding result. In the parent shell, the variable is accessed as a normal variable. However, in the child shell, it gets interpreted as a function and executes the body of the function.

11.  Next, we will terminate the definition of the function and then pass any arbitrary command, as follows:

![](img/9d16bb04-2380-4cdd-9ac0-5cd72efff415.png)

In the preceding example, we can see that as soon as we start a new `bash` shell, the code that was defined outside the function is executed during the startup of `bash`.

This is the vulnerability in `bash` shell.

# How it works...

We first check the version of bash running on our system. Then, we run the well-known code to confirm if shellshock vulnerability exists.

To understand how shellshock vulnerability works, we create a variable in bash and then try to export it to the child shell and execute it there. Next, we try to create another variable and assign its value as `'() { echo 'shellshock';}'`. After doing this, when we export this variable to a child shell and execute it there, we can see that it gets interpreted as a function and executes the body of the function.

This is what makes bash vulnerable to shellshock, where specially crafted variables can be used to run any command in bash when it is launched.

# Security issues – Shellshock

In this era of almost everything being online, online security is a major concern. Nowadays, many web servers, web-connected devices, and services use Linux as their platform. Most versions of Linux use the Unix bash shell so that the Shellshock vulnerability can affect a huge portion of websites and web servers.

In the previous recipe, we understood the details about Shellshock vulnerability. Now, we will understand how this bug can be exploited through SSH.

# Getting ready

To exploit Shellshock vulnerability, we need two systems. The first system will be used as the victim's, and should be vulnerable to Shellshock. In our case, we are using an Ubuntu system as the vulnerable system. The second system will be used as the attacker, and can have any Linux version running on it. For our case, we are running Kali on the second system.

The victim system will be running the `openssh-server` package. It can be installed using the following command:

```
    apt-get install openssh-server
```

We will then configure this system as a vulnerable SSH server to show how it can be exploited using the Shellshock bug.

# How to do it...

To see how the Shellshock bug can be used to exploit a SSH server, we need to first configure our SSH server as a vulnerable system. To do so, we will follow these steps:

1.  The first step is to add a new user account called `user1` on the SSH server system. We must also add `/home/user1` as its home directory and `/bin/bash` as its shell:

![](img/d8ffdbb1-f272-45a0-bb5b-4e0ea955681d.png)

Once the account is added, we cross check it by checking the `/etc/passwd` file.

2.  Next, we create a directory for `user1` in `/home` and grant the ownership of this directory to the `user1` account:

![](img/6bdf8939-acfd-4a4c-8890-aa89f7073eea.png)

3.  Now, we need to authenticate the attacker to login to the SSH server using the authorization keys. For doing this, we will first generate the authorization keys on the attacker's system, using the following command:

![](img/613941c5-d68d-4da7-8e6d-8845e60685b0.png)

We can see that the public/private keys have been generated.

4.  After generating the authorization keys, we will send the public key to the remote SSH server over SFTP. First, we have copied the public key file `id_rsa.pub` to the Desktop and then we run the following command to connect to the SSH server using SFTP:

![](img/99639c9e-9619-4137-86a5-cc6ff7cb0da1.png)

When connected, we transfer the file using the `put` command.

5.  Now, on the victim SSH server system, we create a directory called `.ssh` inside `/home/user1/` and then we write the content of the `id_rsa.pub` file to `authorized_keys` inside the `/home/user1/.ssh/` directory:

![](img/8a68697b-c19a-41f1-8067-adcbe0479359.png)

6.  After this, we edit the configuration file of SSH, `etc/ssh/sshd_config`, and enable the `PublicKeyAuthentication` variable. We also check that `AuthorizedKeysFile` is specified correctly:

![](img/c8758e43-2a47-4527-836f-661e04872047.png)

7.  Once the preceding steps are successfully completed, we can try to log in to the SSH server from the attacker system to see if we are prompted for a password or not:

![](img/5fa3f36c-3c7e-49db-8971-a7e8b93a16c0.png)

8.  Now, we will create a basic script which will display the message `restricted` if the user tries to pass the `date` command as an argument. However, if anything other than `date` is passed, it will get executed. We will name this script `sample.sh`:

![](img/b52db8bf-24e6-49cb-a256-eebe1e144f57.png)

9.  Once the script is created, we can run the given command to give executable permissions to it:

```
    chmod +x sample.sh
```

10.  After this, we use the `command` option in the `authorized_keys` file to run our `sample.sh` script by adding the path of the script:

![](img/621a0c58-a97f-4413-880c-d1a20bf7c0e9.png)

Marking the precedings changes in the `authorized_keys` file, to restrict a user from executing a predefined set of commands, will make the public key authentication vulnerable.

11.  Now, from the attacker's system, try connecting to the victim's system over SSH, while passing `date` as an argument:

![](img/7b14e2ee-368a-4759-bd06-20ecf821ad39.png)

We can see the message `restricted` is displayed due to the script that we have added to the `authorized_keys` file.

11.  Next, we try to pass our Shellshock exploit as an argument, as follows:

![](img/2842e182-0fc1-41e5-a34a-61661b83e3d2.png)

We can see that even though we have restricted the `date` command in this script, it gets executed this time and we get the output of the `date` command.

12.  Now, let's see how we can use Shellshock vulnerability to compromise an Apache server that is running any script that can trigger the bash shell with environment variables.

13.  If Apache is not already installed on the victim's system, we must install it by running the following command:

```
       apt-get install apache2
```

Once installed, we launch the Apache server using the following command:

```
        service apache2 start
```

14.  Next, we move to the `/usr/lib/cgi-bin/` path and create a script called `example.sh` with the following code in it, to display some HTML output:

![](img/bbf585ee-625e-4651-b3c8-4b95bb34ffbc.png)

We then make it executable by running the following command:

```
          chmod +x example.sh
```

Now from the attacker's system, we try to access `example.sh` file remotely using command line tool called `curl -`.

We get the output of the script as expected: `Example Page`.

![](img/8d58573d-be6f-4aa7-87f2-7295c2d34e32.png)

15.  Now, let's send a malicious request to the server, using curl, to print the content of the `/etc/passwd` file of the victim's system:

```
    curl -A '() { :;}; echo "Content-type: text/plain"; echo; /bin/cat /etc/passwd' http://192.168.1.104/cgi-bin/example.sh
```

![](img/286f6f8f-0d86-412b-8f02-2b5c9b1b92f7.png)

Here is the output, but truncated:

![](img/f03f7090-e5cf-4b3b-9342-e28e6ab23255.png)

We can see the output on the attacker's system, showing us how the victim's system can be remotely accessed using Shellshock vulnerability. In the preceding command, `() { :;} ;` signifies a variable that looks like a function. In this code, the function is a single `:`, which is defined as doing nothing and is only a simple command.

16.  We will try another command to see the content of the current directory of the victim's system, as follows:

![](img/503b15b4-3af1-4bdc-9bf5-6930547ef025.png)

We can see the content of the `root` directory of the victim's system in the preceding output.

# How it works...

On our SSH server system, we create a new user account and assign a bash shell to it as its default shell. We also create a directory for this new user account in `/home` and assign its ownership to this account.

Next, we configure our SSH server system to authenticate another system connecting to it using authorization keys.

We then create a bash script to restrict particular commands such as `date` and add this script path to `authorized_keys` using the command option.

After this, when we try to connect to the SSH server from the other system, whose authorization keys were configured earlier, if we pass the `date` command as an argument when connecting, we can see that the command gets restricted.

However, when the same `date` command is passed with the Shellshock exploit, we can see the output of it, thereby showing us how Shellshock can be used to exploit the SSH server.

Similarly, we exploit the Apache server by creating a sample script and placing it in the `/usr/lib/cgi-bin` directory of the Apache system.

Then, we try to access this script from the other system using the curl tool.

We can see that if we pass `shellshock exploit` when accessing the script through curl, we are able to run our commands on the Apache server remotely.

# Linux patch management system

In present computing scenarios, vulnerability and patch management is a never ending cycle. When an attack happens on a computer due to a known vulnerability being exploited, we can see that the patch for such a vulnerability already exists, but has not been implemented properly on the system, which causes the attack to happen.

As a system administrator, we have to know which patch needs to be installed and which one should be ignored.

# Getting ready

Since patch management can be done using the built-in tools of Linux, no specific settings need to be configured before performing these steps.

# How to do it...

The easiest and most efficient way to keep our system updated is to the use the Update Manager, which is built into the Linux system. In this recipe, we will explore how the Update Manager works on the Ubuntu system:

1.  To open the graphical version of Update Manager in Ubuntu, click the **Superkey**, which is on the top in the toolbar on the left-hand side, and then type `update`. In the following screenshot, we can see the Update Manager:

![](img/aad20015-ff77-4c65-8783-f87d9ca7c3b2.png)

2.  When we open Update Manager, we will see the following pop-up box, showing different security updates available for installation:

![](img/e411f634-72f5-4265-ae94-34d58ee27db4.png)

Select the updates to install and click on Install Updates to proceed.

3.  On the same window, we have the Settings button on the bottom-left. When we click that, we get a new window called Software Sources, which has more options for configuring the Update Manager.
4.  The first tab reads Ubuntu Software, and it displays a list of repositories for downloading the updates. We choose the options from the list as per our requirements:

![](img/1d0769f9-7d05-4ea1-a8d1-0a8366e08e1f.png)

5.  If we click on the option Download from, we get the option to change the repository server to be used for downloading. This option is useful if we have any problems with connecting to the currently selected server or if the server is slow:

![](img/59c824e6-78c1-4dbe-a14f-82d143e9a952.png)

6.  From the dropdown, when we select the Other option, we get a list to select the server, as follows:

![](img/50c39792-90da-4fdc-8ccd-5e7eaa3d15c1.png)

7.  The next tab, Other Software, is used to add partner repositories of Canonical:

![](img/a5b7f2c9-73f5-4394-a887-5dd8e23d08d9.png)

8.  We can choose any option from the list shown in the preceding screenshot and click on Edit to make changes to the repository details:

![](img/69cc8958-3cde-4ec4-95bd-11c42b96260f.png)

9.  The Updates tab is used to define how and when the Ubuntu system will receive updates:

![](img/27191cc7-4d61-4f80-ba64-ec2d8482587f.png)

10.  The next tab, Authentication, contains details about the authentication keys of the software providers, as obtained from the maintainer of the software repositories:

![](img/ced82557-0fa3-42a7-8b2d-bfe199b157f7.png)

11.  The last tab is called Statistics, and is available for users who would like to provide data to the Ubuntu developer project anonymously. This information helps the developer to increase the performance and experience of the software:

![](img/ffce7bad-4833-415c-b049-4f754809c656.png)

12.  After making any changes under any of the tabs, when we click on close, it gives us a prompt to confirm if the new updates should be shown in the list or not. Click Reload or Close:

![](img/5f8c1455-5b53-4ad5-b8c8-42c1fef79ae3.png)

13.  If we want to check the list of locations from which the Update Manager retrieves all the packages, we can check the content of the `/etc/apt/sources.list` file. We get the following result:

![](img/102d5896-0d73-4ba3-b67f-7b63b351bc12.png)

# How it works...

To update our Linux system, we use the built-in Update Manager as per the Linux distribution.

In the update manager, either we install all the updates that are available, or we configure it as per our requirements using the Settings window.

In the Settings window, we have the option to display the list of repositories from where the updates can be downloaded.

The second tab in the Settings window lets us add third-party partner repositories of Canonical.

Using the next tab, we can specify when and what kind of updates should be downloaded.

We also check the authentication keys of the software providers using the Settings window.

The last tab, called Statistics, helps us send data to Ubuntu project developers for increasing the performance of the software.

# Applying patches in Linux

Whenever a security vulnerability is found in any software, a security patch is released for the software to fix the bug. Normally, we use the Update Manager that's built into Linux to apply the security updates. However, for software that we install by compiling the source code, Update Manager may not be helpful.

For such situations, we can apply the patch file to the original software's source code and then recompile the software.

# Getting ready

Since we will use the built-in commands of Linux to create and apply a patch, nothing needs to be done before starting the following steps. We will be creating a sample program in C for understanding the process of creating a patch file.

# How to do it...

In this section, we will see how to create a patch for a program, using the `diff` command, and then apply the patch using the `patch` command:

1.  Our first step will be to create a simple C program called `example.c` to print `This is an example`, as follows:

![](img/cbdb0c9c-f48b-4055-b39e-68209f3e6760.png)

2.  Now, we will create a copy of `example.c` and name it `example_new.c`
3.  Next, we will edit the new file `example_new.c` and add a few extra lines of code in it, as follows:

![](img/77494098-5bbb-417f-bc9b-31214330bd7d.png)

4.  Now, `example_new.c` can be considered as the updated version of `example.c`

5.  We will now create a patch file and name it `example.patch` by using the `diff` command, as follows:

![](img/88c29b27-0f0a-407b-86e2-b6fba615bc3a.png)

6.  If we check the content of the patch file, we get the following output:

![](img/23493436-1fb7-4a5b-9a80-6cf9e2f17f53.png)

7.  Now, before applying the patch, we can take a backup of the original file by using the `-b` option, as follows:

![](img/2f3ffbc6-636a-4df2-b7f0-e3ec747b3286.png)

We can see that a new file called `example.c.orig` has been created, which is the backup file.

8.  Before doing the actual patching, we can dry run the patch file to check whether we are getting any errors or not. To do this, we run the following command:

![](img/5a60562f-ab8a-4457-adaa-b044f59a6a86.png)

If we get no error message, it means that the patch file can be now run on the original file.

9.  Now, we run the following command to apply the patch to the original file:

```
    patch < example.patch
```

10.  After applying the patch, if we now check the content of the `example.c` program, we will see that it has been updated with the extra lines of code, as written in `example_new.c`:

![](img/0bc098fc-dbac-45d2-982c-c3984f5705a4.png)

11.  Once the patch has been applied on the original file, if we wish to reverse the patch, we can use the `-R` option, as follows:

![](img/e9695a65-d798-4038-99a3-b53213cca9e7.png)

We can see the difference in the size of the file after patching and then after reversing.

# How it works...

We first create a sample C program. Then, we create a copy of it and add few more lines of code to make it an updated version. After this, we create a patch file using the `diff` command. Before applying the patch, we check it for any errors by doing a dry run.

If we get no errors, we apply the patch using the `patch` command. Now, the original file has the same content as the updated version file.

We can also reverse the patch using the `-R` option.

# Other well-known Linux vulnerabilities

With time, Linux has gained a lot of popularity due to its open source nature. However, it has also resulted into increased security concerns. Linux systems tend to be as vulnerable as other operating systems, such as Windows. These vulnerabilities may be due to faults in the OS, or due to oversight by the Linux administrators.

# How to do it...

In this section, we will see discuss about few of the most common Linux vulnerabilities, as follows:

1.  **Linux Kernel netfilter: xt_TCPMSS**: Even though it's an old vulnerability, affecting Linux kernels before 4.11, and 4.9.x before 4.9.36, it still exists in many systems of organizations that have failed to attend to this vulnerability and are still using older versions of the Linux kernel. It has CVE ID: CVE-2017-18017 and a critical vulnerability score of 9.8.
2.  If exploited successfully, the aforementioned vulnerability can help hackers send through a flood of communications and cause a **denial-of-service** (**DoS**) attack.
3.  **Dirty Cow Bug**: CVE-2016-5195 is the official reference to this bug. It was discovered that a race condition existed in the memory manager of the Linux kernel when handling copy-on-write breakage of private, read-only memory mappings.

The flaw is located in a section of the Linux kernel that's a part of virtually every distribution of the open source OS that has been around for almost a decade:

![](img/8e46120c-c0ed-44d3-a488-81589e25a33c.png)

4.  Exploitation of this bug does not leave any traces of anything abnormal happening to the logs. Any local users can write to any file they can read, and has been present since at least Linux kernel version 2.6.22.
5.  If you want more information about the exploit available for this bug, you can check: [https://www.exploit-db.com/exploits/40839/](https://www.exploit-db.com/exploits/40839/).
6.  The Metasploit framework, the most popular framework for penetration testing, also includes an exploit module for the Dirty Cow bug. More information regarding the same can be found here: [https://github.com/rapid7/metasploit-framework/pull/7476](https://github.com/rapid7/metasploit-framework/pull/7476).

# How it works...

Time and again, many vulnerabilities have been detected in Linux, whether it's related to the kernel or the OS-level code.

A few vulnerabilities, such as Dirty Cow, have existed for a long time, allowing attackers to exploit them easily.

Most of these vulnerabilities have the exploits available, hence it's necessary to keep our system patched and updated in order to remain secure.