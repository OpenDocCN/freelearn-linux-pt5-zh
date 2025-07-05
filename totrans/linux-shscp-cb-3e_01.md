# Shell Something Out

In this chapter, we will cover the following recipes:

*   Displaying output in a terminal
*   Using variables and environment variables
*   Function to prepend to environment variables
*   Math with the shell
*   Playing with file descriptors and redirection
*   Arrays and associative arrays
*   Visiting aliases
*   Grabbing information about the terminal
*   Getting and setting dates and delays
*   Debugging the script
*   Functions and arguments
*   Sending output from one command to another
*   Reading `n` characters without pressing the return key
*   Running a command until it succeeds
*   Field separators and iterators
*   Comparisons and tests
*   Customizing bash with configuration files

# Introduction

In the beginning, computers read a program from cards or tape and generated a single report. There was no operating system, no graphics monitors, not even an interactive prompt.

By the 1960s, computers supported interactive terminals (frequently a teletype or glorified typewriter) to invoke commands.

When Bell Labs created an interactive user interface for the brand new Unix operating system, it had a unique feature. It could read and evaluate the same commands from a text file (called a shell script), as it accepted being typed on a terminal.

This facility was a huge leap forward in productivity. Instead of typing several commands to perform a set of operations, programmers could save the commands in a file and run them later with just a few keystrokes. Not only does a shell script save time, it also documents what you did.

Initially, Unix supported one interactive shell, written by Stephen Bourne, and named it the **Bourne Shell** (**sh**).

In 1989, Brian Fox of the GNU Project took features from many user interfaces and created a new shell—the **Bourne Again Shell** (**bash**). The bash shell understands all of the Bourne shell constructs and adds features from csh, ksh, and others.

As Linux has become the most popular implementation of Unix like operating systems, the bash shell has become the de-facto standard shell on Unix and Linux.

This book focuses on Linux and bash. Even so, most of these scripts will run on both Linux and Unix, using bash, sh, ash, dash, ksh, or other sh style shells.

This chapter will give readers an insight into the shell environment and demonstrate some basic shell features.

# Displaying output in a terminal

Users interact with the shell environment via a terminal session. If you are running a GUI-based system, this will be a terminal window. If you are running with no GUI, (a production server or ssh session), you will see the shell prompt as soon as you log in.

Displaying text in the terminal is a task most scripts and utilities need to perform regularly. The shell supports several methods and different formats for displaying text.

# Getting ready

Commands are typed and executed in a terminal session. When a terminal is opened, a prompt is displayed. The prompt can be configured in many ways, but frequently resembles this:

```
username@hostname$

```

Alternatively, it can also be configured as `root@hostname #` or simply as `$` or `#`.

The `$` character represents regular users and `#` represents the administrative user root. Root is the most privileged user in a Linux system.

It is a bad idea to directly use the shell as the root user (administrator) to perform tasks. Typing errors have the potential to do more damage when your shell has more privileges. It is recommended that you log in as a regular user (your shell may denote this as `$` in the prompt), and use tools such as `sudo` to run privileged commands. Running a command as `sudo <command> <arguments>` will run it as root.

A shell script typically begins with a shebang:

```
#!/bin/bash

```

Shebang is a line on which `#!` is prefixed to the interpreter path. `/bin/bash` is the interpreter command path for Bash. A line starting with a `#` symbol is treated by the bash interpreter as a comment. Only the first line of a script can have a shebang to define the interpreter to be used to evaluate the script.

A script can be executed in two ways:

1.  Pass the name of the script as a command-line argument:

```
 bash myScript.sh

```

2.  Set the execution permission on a script file to make it executable:

```
 chmod 755 myScript.sh ./myScript.sh.

```

If a script is run as a command-line argument for `bash`, the shebang is not required. The shebang facilitates running the script on its own. Executable scripts use the interpreter path that follows the shebang to interpret a script.

Scripts are made executable with the `chmod` command:

```
$ chmod a+x sample.sh

```

This command makes a script executable by all users. The script can be executed as follows:

```
$ ./sample.sh #./ represents the current directory

```

Alternatively, the script can be executed like this:

```
$ /home/path/sample.sh # Full path of the script is used

```

The kernel will read the first line and see that the shebang is `#!/bin/bash`. It will identify `/bin/bash` and execute the script as follows:

```
$ /bin/bash sample.sh

```

When an interactive shell starts, it executes a set of commands to initialize settings, such as the prompt text, colors, and so on. These commands are read from a shell script at `~/.bashrc` (or `~/.bash_profile` for login shells), located in the home directory of the user. The Bash shell maintains a history of commands run by the user in the `~/.bash_history` file.

The `~` symbol denotes your home directory, which is usually `/home/user`, where user is your username or `/root` for the root user. A login shell is created when you log in to a machine. However, terminal sessions you create while logged in to a graphical environment (such as GNOME, KDE, and so on), are not login shells. Logging in with a display manager such as GDM or KDM may not read a `.profile` or `.bash_profile` (most don't), but logging in to a remote system with ssh will read the `.profile`. The shell delimits each command or command sequence with a semicolon or a new line. Consider this example: `$ cmd1 ; cmd2`
This is equivalent to these: 
`$ cmd1`
`$ cmd2`

A comment starts with `#` and proceeds up to the end of the line. The comment lines are most often used to describe the code, or to disable execution of a line of code during debugging:

```
# sample.sh - echoes "hello world" echo "hello world"

```

Now let's move on to the basic recipes in this chapter.

# How to do it...

The `echo` command is the simplest command for printing in the terminal.

By default, `echo` adds a newline at the end of every echo invocation:

```
$ echo "Welcome to Bash" Welcome to Bash

```

Simply, using double-quoted text with the `echo` command prints the text in the terminal. Similarly, text without double quotes also gives the same output:

```
$ echo Welcome to Bash Welcome to Bash

```

Another way to do the same task is with single quotes:

```
$ echo 'text in quotes'

```

These methods appear similar, but each has a specific purpose and side effects. Double quotes allow the shell to interpret special characters within the string. Single quotes disable this interpretation.

Consider the following command:

```
$ echo "cannot include exclamation - ! within double quotes"

```

This returns the following output:

```
bash: !: event not found error

```

If you need to print special characters such as `!`, you must either not use any quotes, use single quotes, or escape the special characters with a backslash (`\`):

```
$ echo Hello world ! 

```

Alternatively, use this:

```
$ echo 'Hello world !'

```

Alternatively, it can be used like this:

```
$ echo "Hello World\!" #Escape character \ prefixed.

```

When using `echo` without quotes, we cannot use a semicolon, as a semicolon is the delimiter between commands in the Bash shell:

```
echo hello; hello 

```

From the preceding line, Bash takes `echo hello` as one command and the second `hello` as the second command.

Variable substitution, which is discussed in the next recipe, will not work within single quotes.

Another command for printing in the terminal is `printf`. It uses the same arguments as the C library `printf` function. Consider this example:

```
$ printf "Hello world"

```

The `printf` command takes quoted text or arguments delimited by spaces. It supports formatted strings. The format string specifies string width, left or right alignment, and so on. By default, `printf` does not append a newline. We have to specify a newline when required, as shown in the following script:

```
#!/bin/bash #Filename: printf.sh printf  "%-5s %-10s %-4s\n" No Name  Mark printf  "%-5s %-10s %-4.2f\n" 1 Sarath 80.3456 printf  "%-5s %-10s %-4.2f\n" 2 James 90.9989 printf  "%-5s %-10s %-4.2f\n" 3 Jeff 77.564

```

We will receive the following formatted output:

```
No    Name       Mark 1     Sarath     80.35 2     James      91.00 3     Jeff       77.56

```

# How it works...

The `%s`, `%c`, `%d`, and `%f` characters are format substitution characters, which define how the following argument will be printed. The `%-5s` string defines a string substitution with left alignment (`-` represents left alignment) and a `5` character width. If `-` was not specified, the string would have been aligned to the right. The width specifies the number of characters reserved for the string. For `Name`, the width reserved is `10`. Hence, any name will reside within the 10-character width reserved for it and the rest of the line will be filled with spaces up to 10 characters total.

For floating point numbers, we can pass additional parameters to round off the decimal places.

For the Mark section, we have formatted the string as `%-4.2f`, where `.2` specifies rounding off to two decimal places. Note that for every line of the format string, a newline (`\n`) is issued.

# There's more...

While using flags for `echo` and `printf`, place the flags before any strings in the command, otherwise Bash will consider the flags as another string.

# Escaping newline in echo

By default, `echo` appends a newline to the end of its output text. Disable the newline with the `-n` flag. The `echo` command accepts escape sequences in double-quoted strings as an argument. When using escape sequences, use `echo` as `echo -e "string containing escape sequences"`. Consider the following example:

```
echo -e "1\t2\t3" 1  2  3

```

# Printing a colored output

A script can use escape sequences to produce colored text on the terminal.

Colors for text are represented by color codes, including, reset = 0, black = 30, red = 31, green = 32, yellow = 33, blue = 34, magenta = 35, cyan = 36, and white = 37.

To print colored text, enter the following command:

```
echo -e "\e[1;31m This is red text \e[0m"

```

Here, `\e[1;31m` is the escape string to set the color to red and `\e[0m` resets the color back. Replace `31` with the required color code.

For a colored background, reset = 0, black = 40, red = 41, green = 42, yellow = 43, blue = 44, magenta = 45, cyan = 46, and white=47, are the commonly used color codes.

To print a colored background, enter the following command:

```
echo -e "\e[1;42m Green Background \e[0m"

```

These examples cover a subset of escape sequences. The documentation can be viewed with `man console_codes`.

# Using variables and environment variables

All programming languages use variables to retain data for later use or modification. Unlike compiled languages, most scripting languages do not require a type declaration before a variable is created. The type is determined by usage. The value of a variable is accessed by preceding the variable name with a dollar sign. The shell defines several variables it uses for configuration and information like available printers, search paths, and so on. These are called **environment variables**.

# Getting ready

Variables are named as a sequence of letters, numbers, and underscores with no whitespace. Common conventions are to use UPPER_CASE for environment variables and camelCase or lower_case for variables used within a script.

All applications and scripts can access the environment variables. To view all the environment variables defined in your current shell, issue the `env` or `printenv` command:

```
$> env 
PWD=/home/clif/ShellCookBook 
HOME=/home/clif 
SHELL=/bin/bash 
# ... And many more lines

```

To view the environment of other processes, use the following command:

```
cat /proc/$PID/environ

```

Set `PID` with a process ID of the process (`PID` is an integer value).

Assume an application called `gedit` is running. We obtain the process ID of `gedit` with the `pgrep` command:

```
$ pgrep gedit 12501

```

We view the environment variables associated with the process by executing the following command:

```
$ cat /proc/12501/environ GDM_KEYBOARD_LAYOUT=usGNOME_KEYRING_PID=1560USER=slynuxHOME=/home/slynux

```

Note that the previous output has many lines stripped for convenience. The actual output contains more variables.
The `/proc/PID/environ` special file contains a list of environment variables and their values. Each variable is represented as a name=value pair, separated by a null character (`\0`). This is not easily human readable.

To make a human-friendly report, pipe the output of the `cat` command to `tr`, to substitute the `\0` character with `\n`:

```
$ cat /proc/12501/environ  | tr '\0' '\n'

```

# How to do it...

Assign a value to a variable with the equal sign operator:

```
varName=value

```

The name of the variable is `varName` and `value` is the value to be assigned to it. If `value` does not contain any space character (such as space), it need not be enclosed in quotes, otherwise it must be enclosed in single or double quotes.

Note that `var = value` and `var=value` are different. It is a usual mistake to write `var = value` instead of `var=value`. An equal sign without spaces is an assignment operation, whereas using spaces creates an equality test.

Access the contents of a variable by prefixing the variable name with a dollar sign (`$`).

```
var="value" #Assign "value" to var echo $var

```

You may also use it like this:

```
echo ${var}

```

This output will be displayed:

```
value

```

Variable values within double quotes can be used with `printf`, `echo`, and other shell commands:

```
#!/bin/bash #Filename :variables.sh fruit=apple count=5 echo "We have $count ${fruit}(s)"

```

The output will be as follows:

```
We have 5 apple(s)

```

Because the shell uses a space to delimit words, we need to add curly braces to let the shell know that the variable name is `fruit`, not `fruit(s)`.

Environment variables are inherited from the parent processes. For example, `HTTP_PROXY` is an environment variable that defines which proxy server to use for an Internet connection.

Usually, it is set as follows:

```
HTTP_PROXY=192.168.1.23:3128 export HTTP_PROXY

```

The `export` command declares one or more variables that will be inherited by child tasks. After variables are exported, any application executed from the current shell script, receives this variable. There are many standard environment variables created and used by the shell, and we can export our own variables.

For example, the `PATH` variable lists the folders, which the shell will search for an application. A typical `PATH` variable will contain the following:

```
$ echo $PATH 
/home/slynux/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

```

Directory paths are delimited by the `:` character. Usually, `$PATH` is defined in `/etc/environment`, `/etc/profile` or `~/.bashrc`.

To add a new path to the `PATH` environment, use the following command:

```
export PATH="$PATH:/home/user/bin"

```

Alternatively, use these commands:

```
$ PATH="$PATH:/home/user/bin" 
$ export PATH 
$ echo $PATH 
/home/slynux/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/home/user/bin

```

Here we have added `/home/user/bin` to `PATH`.

Some of the well-known environment variables are `HOME`, `PWD`, `USER`, `UID`, and `SHELL`.

When using single quotes, variables will not be expanded and will be displayed as it is. This means, `$ echo '$var'` will display `$var`.

Whereas, `$ echo "$var"` will display the value of the `$var` variable if it is defined, or nothing if it is not defined.

# There's more...

The shell has many more built-in features. Here are a few more:

# Finding the length of a string

Get the length of a variable's value with the following command:

```
length=${#var}

```

Consider this example:

```
$ var=12345678901234567890$ echo ${#var} 20

```

The `length` parameter is the number of characters in the string.

# Identifying the current shell

To identify the shell which is currently being used, use the `SHELL environment` variable.

```
echo $SHELL

```

Alternatively, use this command:

```
echo $0

```

Consider this example:

```
$ echo $SHELL /bin/bash

```

Also, by executing the `echo $0` command, we will get the same output:

```
$ echo $0 /bin/bash

```

# Checking for super user

The `UID` environment variable holds the User ID. Use this value to check whether the current script is being run as a root user or regular user. Consider this example:

```
If [ $UID -ne 0 ]; then 
  echo Non root user. Please run as root. 
else 
  echo Root user 
fi

```

Note that `[` is actually a command and must be separated from the rest of the string with spaces. We can also write the preceding script as follows:

```
if test $UID -ne 0:1 
  then 
    echo Non root user. Please run as root 
  else 
    echo Root User 
fi

```

The `UID` value for the root user is `0`.

# Modifying the Bash prompt string (username@hostname:~$)

When we open a terminal or run a shell, we see a prompt such as `user@hostname: /home/$`. Different GNU/Linux distributions have different prompts and different colors. The `PS1` environment variable defines the primary prompt. The default prompt is defined by a line in the `~/.bashrc` file.

*   View the line used to set the `PS1` variable:

```
        $ cat ~/.bashrc | grep PS1 
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '

```

*   To modify the prompt, enter the following command:

```
        slynux@localhost: ~$ PS1="PROMPT> " # Prompt string changed 
        PROMPT> Type commands here.

```

*   We can use colored text using the special escape sequences such as `\e[1;31` (refer to the *Displaying output in a terminal *recipe of this chapter).

Certain special characters expand to system parameters. For example, `\u` expands to username, `\h` expands to hostname, and `\w` expands to the current working directory.

# Function to prepend to environment variables

Environment variables are often used to store a list of paths of where to search for executables, libraries, and so on. Examples are `$PATH` and  `$LD_LIBRARY_PATH`, which will typically resemble this:

```
PATH=/usr/bin;/bin 
LD_LIBRARY_PATH=/usr/lib;/lib

```

This means that whenever the shell has to execute an application (binary or script), it will first look in `/usr/bin` and then search `/bin`.

When building and installing a program from source, we often need to add custom paths for the new executable and libraries. For example, we might install `myapp` in `/opt/myapp`, with binaries in a `/opt/myapp/bin` folder and libraries in `/opt/myapp/lib`.

# How to do it...

This example shows how to add new paths to the beginning of an environment variable. The first example shows how to do this with what's been covered so far, the second demonstrates creating a function to simplify modifying the variable. Functions are covered later in this chapter.

```
export PATH=/opt/myapp/bin:$PATH 
export LD_LIBRARY_PATH=/opt/myapp/lib;$LD_LIBRARY_PATH

```

The `PATH` and `LD_LIBRARY_PATH` variables should now look something like this:

```
PATH=/opt/myapp/bin:/usr/bin:/bin 
LD_LIBRARY_PATH=/opt/myapp/lib:/usr/lib;/lib

```

We can make adding a new path easier by defining a prepend function in the `.bashrc` file.

```
prepend() { [ -d "$2" ] && eval $1=\"$2':'\$$1\" && export $1; }

```

This can be used in the following way:

```
prepend PATH /opt/myapp/bin 
prepend LD_LIBRARY_PATH /opt/myapp/lib

```

# How it works...

The `prepend()` function first confirms that the directory specified by the second parameter to the function exists. If it does, the `eval` expression sets the variable, with the name in the first parameter equal to the second parameter string, followed by `:` (the path separator), and then the original value for the variable.

If the variable is empty when we try to prepend, there will be a trailing `:` at the end. To fix this, modify the function to this:

```
prepend() { [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1 ; }

```

In this form of the function, we introduce a shell parameter expansion of the form:
`${parameter:+expression}`
This expands to expression if parameter is set and is not null.
With this change, we take care to try to append `:` and the old value if, and only if, the old value existed when trying to prepend.

# Math with the shell

The Bash shell performs basic arithmetic operations using the `let`, `(( ))`, and `[]` commands. The `expr` and `bc` utilities are used to perform advanced operations.

# How to do it...

1.  A numeric value is assigned to a variable the same way strings are assigned. The value will be treated as a number by the methods that access it:

```
 #!/bin/bash no1=4; no2=5;

```

2.  The `let` command is used to perform basic operations directly. Within a `let` command, we use variable names without the `$` prefix. Consider this example:

```
 let result=no1+no2 echo $result 

```

                  Other uses of `let` command are as follows:

*   Use this for increment: 

```
 $ let no1++

```

*   For decrement, use this:

```
 $ let no1--

```

*   Use these for shorthands:

```
 let no+=6 let no-=6

```

                These are equal to `let no=no+6` and `let no=no-6`, respectively.

*   Alternate methods are as follows:

              The `[]` operator is used in the same way as the `let` command:

```
 result=$[ no1 + no2 ]

```

              Using the $ prefix inside the [] operator is legal; consider this example:

```
 result=$[ $no1 + 5 ]

```

      The `(( ))` operator can also be used. The prefix variable names                        with a `$` within the `(( ))` operator:

```
 result=$(( no1 + 50 ))

```

             The `expr` expression can be used for basic operations:

```
 result=`expr 3 + 4` result=$(expr $no1 + 5)

```

      The preceding methods do not support floating point numbers,
      and operate on integers only.

3.  The `bc` application, the precision calculator, is an advanced utility for mathematical operations. It has a wide range of options. We can perform floating point arithmetic and use advanced functions:

```
 echo "4 * 0.56" | bc 2.24 no=54; result=`echo "$no * 1.5" | bc` echo $result 81.0

```

The `bc` application accepts prefixes to control the operation. These are separated from each other with a semicolon.

*   **Decimal places scale with bc**: In the following example, the `scale=2` parameter sets the number of decimal places to `2`. Hence, the output of `bc` will contain a number with two decimal places:

```
 echo "scale=2;22/7" | bc 3.14

```

*   **Base conversion with bc**: We can convert from one base number system to another one. This code converts numbers from decimal to binary and binary to decimal:

```
 #!/bin/bash Desc: Number conversion no=100 echo "obase=2;$no" | bc 1100100 no=1100100 echo "obase=10;ibase=2;$no" | bc 100

```

*   The following examples demonstrate calculating squares and square roots:

```
 echo "sqrt(100)" | bc #Square root echo "10^10" | bc #Square

```

# Playing with file descriptors and redirection

File descriptors are integers associated with the input and output streams. The best-known file descriptors are `stdin`, `stdout`, and `stderr`. The contents of one stream can be redirected to another. This recipe shows examples on how to manipulate and redirect with file descriptors.

# Getting ready

Shell scripts frequently use standard input (`stdin`), standard output (`stdout`), and standard error (`stderr`). A script can redirect output to a file with the greater-than symbol. Text generated by a command may be normal output or an error message. By default, both normal output (`stdout`) and error messages (`stderr`) are sent to the display. The two streams can be separated by specifying a specific descriptor for each stream.

File descriptors are integers associated with an opened file or data stream. File descriptors 0, 1, and 2 are reserved, as given here:

*   0: `stdin`
*   1: `stdout`
*   2: `stderr`

# How to do it...

1.  Use the greater-than symbol to append text to a file:

```
 $ echo "This is a sample text 1" > temp.txt

```

This stores the echoed text in `temp.txt`. If `temp.txt` already exists, the single greater-than sign will delete any previous contents.

2.  Use double-greater-than to append text to a file:

```
 $ echo "This is sample text 2" >> temp.txt

```

3.  Use `cat` to view the contents of the file:

```
 $ cat temp.txt This is sample text 1 This is sample text 2

```

The next recipes demonstrate redirecting `stderr`. A message is printed to the `stderr` stream when a command generates an error message. Consider the following example:

```
$ ls + ls: cannot access +: No such file or directory

```

Here `+` is an invalid argument and hence an error is returned.

Successful and unsuccessful commands
When a command exits because of an error, it returns a nonzero exit status. The command returns zero when it terminates after successful completion. The return status is available in the special variable `$?` (run echo `$?` immediately after the command execution statement to print the exit status).

The following command prints the `stderr` text to the screen rather than to a file (and because there is no `stdout` output, `out.txt` will be empty):

```
$ ls + > out.txt ls: cannot access +: No such file or directory 

```

In the following command, we redirect `stderr` to `out.txt` with `2>` (two greater-than):

```
$ ls + 2> out.txt # works

```

You can redirect `stderr` to one file and `stdout` to another file.

```
$ cmd 2>stderr.txt 1>stdout.txt

```

It is also possible to redirect `stderr` and `stdout` to a single file by converting `stderr` to `stdout` using this preferred method:

```
$ cmd 2>&1 allOutput.txt

```

This can be done even using an alternate approach:

```
$ cmd &> output.txt 

```

If you don't want to see or save any error messages, you can redirect the stderr output to `/dev/null`, which removes it completely. For example, consider that we have three files `a1`, `a2`, and `a3`. However, `a1` does not have the read-write-execute permission for the user. To print the contents of all files starting with the letter `a`, we use the `cat` command. Set up the test files as follows:

```
$ echo A1 > a1 $ echo A2 > a2 $ echo A3 > a3 $ chmod 000 a1  #Deny all permissions

```

Displaying the contents of the files using wildcards (`a*`), will generate an error message for the `a1` file because that file does not have the proper read permission:

```
$ cat a* cat: a1: Permission denied A2 A3

```

Here, `cat: a1: Permission denied` belongs to the `stderr` data. We can redirect the `stderr` data into a file, while sending `stdout` to the terminal.

```
$ cat a* 2> err.txt #stderr is redirected to err.txt A2 A3 $ cat err.txt cat: a1: Permission denied

```

Some commands generate output that we want to process and also save for future reference or other processing. The `stdout` stream is a single stream that we can redirect to a file or pipe to another program. You might think there is no way for us to have our cake and eat it too.

However, there is a way to redirect data to a file, while providing a copy of redirected data as `stdin` to the next command in a pipe. The `tee` command reads from `stdin` and redirects the input data to `stdout` and one or more files.

```
command | tee FILE1 FILE2 | otherCommand

```

In the following code, the `stdin` data is received by the `tee` command. It writes a copy of `stdout` to the `out.txt` file and sends another copy as `stdin` for the next command. The `cat -n` command puts a line number for each line received from `stdin` and writes it into `stdout`:

```
$ cat a* | tee out.txt | cat -n cat: a1: Permission denied
 1 A2 2 A3

```

Use `cat` to examine the contents of `out.txt`:

```
$ cat out.txt A2 A3

```

Observe that `cat: a1: Permission denied` does not appear, because it was sent to `stderr`. The `tee` command reads only from `stdin`.

By default, the `tee` command overwrites the file. Including the `-a` option will force it to append the new data.

```
$ cat a* | tee -a out.txt | cat -n

```

Commands with arguments follow the format: `command FILE1 FILE2 ...` or simply `command FILE`.

To send two copies of the input to `stdout`, use `-` for the filename argument:

```
$ cmd1 | cmd2 | cmd -

```

Consider this example:

```
$ echo who is this | tee - who is this who is this

```

Alternately, we can use `/dev/stdin` as the output filename to use `stdin`.
Similarly, use `/dev/stderr` for standard error and `/dev/stdout` for standard output. These are special device files that correspond to `stdin`, `stderr`, and `stdout`.

# How it works...

The redirection operators (`>` and `>>`) send output to a file instead of the terminal. The `>` and `>>` operators behave slightly differently. Both redirect output to a file, but the single greater-than symbol (`>`) empties the file and then writes to it, whereas the double greater-than symbol (`>>`) adds the output to the end of the existing file.

By default, the redirection operates on standard output. To explicitly take a specific file descriptor, you must prefix the descriptor number to the operator.

The `>` operator is equivalent to `1>` and similarly it applies for `>>` (equivalent to `1>>`).

When working with errors, the `stderr` output is dumped to the `/dev/null` file. The `./dev/null` file is a special device file where any data received by the file is discarded. The null device is often known as a **black hole**, as all the data that goes into it is lost forever.

# There's more...

Commands that read input from `stdin` can receive data in multiple ways. It is possible to specify file descriptors of our own, using `cat` and pipes. Consider this example:

```
$ cat file | cmd $ cmd1 | cmd2

```

# Redirection from a file to a command

We can read data from a file as `stdin` with the less-than symbol (`<`):

```
$ cmd < file

```

# Redirecting from a text block enclosed within a script

Text can be redirected from a script into a file. To add a warning to the top of an automatically generated file, use the following code:

```
#!/bin/bash 
cat<<EOF>log.txt 
This is a generated file. Do not edit. Changes will be overwritten. 
EOF

```

The lines that appear between `cat <<EOF >log.txt` and the next `EOF` line will appear as the `stdin` data. The contents of `log.txt` are shown here:

```
$ cat log.txt 
This is a generated file. Do not edit. Changes will be overwritten. 

```

# Custom file descriptors

A file descriptor is an abstract indicator for accessing a file. Each file access is associated with a special number called a file descriptor. 0, 1, and 2 are reserved descriptor numbers for `stdin`, `stdout`, and `stderr`.

The `exec` command can create new file descriptors. If you are familiar with file access in other programming languages, you may be familiar with the modes for opening files. These three modes are commonly used:

*   Read mode
*   Write with append mode
*   Write with truncate mode

The `<` operator reads from the file to `stdin`. The `>` operator writes to a file with truncation (data is written to the target file after truncating the contents). The `>>` operator writes to a file by appending (data is appended to the existing file contents and the contents of the target file will not be lost). File descriptors are created with one of the three modes.

Create a file descriptor for reading a file:

```
$ exec 3<input.txt # open for reading with descriptor number 3

```

We can use it in the following way:

```
$ echo this is a test line > input.txt $ exec 3<input.txt

```

Now you can use file descriptor `3` with commands. For example, we will use `cat<&3`:

```
$ cat<&3 this is a test line

```

If a second read is required, we cannot reuse the file descriptor `3`. We must create a new file descriptor (perhaps 4) with `exec` to read from another file or re-read from the first file.

Create a file descriptor for writing (truncate mode):

```
$ exec 4>output.txt # open for writing

```

Consider this example:

```
$ exec 4>output.txt $ echo newline >&4 $ cat output.txt newline

```

Now create a file descriptor for writing (append mode):

```
$ exec 5>>input.txt

```

Consider the following example:

```
$ exec 5>>input.txt $ echo appended line >&5 $ cat input.txt newline appended line

```

# Arrays and associative arrays

Arrays allow a script to store a collection of data as separate entities using indices. Bash supports both regular arrays that use integers as the array index, and associative arrays, which use a string as the array index. Regular arrays should be used when the data is organized numerically, for example, a set of successive iterations. Associative arrays can be used when the data is organized by a string, for example, host names. In this recipe, we will see how to use both of these.

# Getting ready

To use associate arrays, you must have Bash Version 4 or higher.

# How to do it...

Arrays can be defined using different techniques:

1.  Define an array using a list of values in a single line:

```
 array_var=(test1 test2 test3 test4) #Values will be stored in consecutive locations starting 
        from index 0.

```

Alternately, define an array as a set of index-value pairs:

```
 array_var[0]="test1" array_var[1]="test2" array_var[2]="test3" array_var[3]="test4" array_var[4]="test5" array_var[5]="test6"

```

2.  Print the contents of an array at a given index using the following commands:

```
 echo ${array_var[0]} test1 index=5 echo ${array_var[$index]} test6

```

3.  Print all of the values in an array as a list, using the following commands:

```
 $ echo ${array_var[*]} test1 test2 test3 test4 test5 test6

```

  Alternately, you can use the following command:

```
 $ echo ${array_var[@]} test1 test2 test3 test4 test5 test6

```

4.  Print the length of an array (the number of elements in an array):

```
 $ echo ${#array_var[*]}6

```

# There's more...

Associative arrays have been introduced to Bash from Version 4.0\. When the indices are a string (site names, user names, nonsequential numbers, and so on), an associative array is easier to work with than a numerically indexed array.

# Defining associative arrays

An associative array can use any text data as an array index. A declaration statement is required to define a variable name as an associative array:

```
$ declare -A ass_array

```

After the declaration, elements are added to the associative array using either of these two methods:

*   Inline index-value list method:

```
 $ ass_array=([index1]=val1 [index2]=val2)

```

*   Separate index-value assignments:

```
 $ ass_array[index1]=val1 $ ass_array'index2]=val2

```

For example, consider the assignment of prices for fruits, using an associative array:

```
$ declare -A fruits_value $ fruits_value=([apple]='100 dollars' [orange]='150 dollars')

```

Display the contents of an array:

```
$ echo "Apple costs ${fruits_value[apple]}" Apple costs 100 dollars

```

# Listing of array indexes

Arrays have indexes for indexing each of the elements. Ordinary and associative arrays differ in terms of index type.

Obtain the list of indexes in an array.

```
$ echo ${!array_var[*]}

```

Alternatively, we can also use the following command:

```
$ echo ${!array_var[@]}

```

In the previous `fruits_value` array example, consider the following command:

```
$ echo ${!fruits_value[*]} orange apple

```

This will work for ordinary arrays too.

# Visiting aliases

An **alias** is a shortcut to replace typing a long-command sequence. In this recipe, we will see how to create aliases using the `alias` command.

# How to do it...

These are the operations you can perform on aliases:

1.  Create an alias:

```
 $ alias new_command='command sequence'

```

This example creates a shortcut for the `apt-get install` command:

```
 $ alias install='sudo apt-get install'

```

Once the alias is defined, we can type `install` instead of `sudo apt-get install`.

2.  The `alias` command is temporary: aliases exist until we close the current terminal. To make an alias available to all shells, add this statement to the `~/.bashrc` file. Commands in `~/.bashrc` are always executed when a new interactive shell process is spawned:

```
 $ echo 'alias cmd="command seq"' >> ~/.bashrc

```

3.  To remove an alias, remove its entry from `~/.bashrc` (if any) or use the `unalias` command. Alternatively, `alias example=` should unset the alias named `example`.
4.  This example creates an alias for `rm` that will delete the original and keep a copy in a backup directory:

```
 alias rm='cp $@ ~/backup && rm $@'

```

When you create an alias, if the item being aliased already exists, it will be replaced by this newly aliased command for that user.

# There's more...

When running as a privileged user, aliases can be a security breach. To avoid compromising your system, you should escape commands.

# Escaping aliases

Given how easy it is to create an alias to masquerade as a native command, you should not run aliased commands as a privileged user. We can ignore any aliases currently defined, by escaping the command we want to run. Consider this example:

```
$ \command

```

The `\` character escapes the command, running it without any aliased changes. When running privileged commands on an untrusted environment, it is always a good security practice to ignore aliases by prefixing the command with `\`. The attacker might have aliased the privileged command with his/her own custom command, to steal critical information that is provided by the user to the command.

# Listing aliases

The `alias` command lists the currently defined aliases:

```
$ aliasalias lc='ls -color=auto' alias ll='ls -l' alias vi='vim'

```

# Grabbing information about the terminal

While writing command-line shell scripts, we often need to manipulate information about the current terminal, such as the number of columns, rows, cursor positions, masked password fields, and so on. This recipe helps in collecting and manipulating terminal settings.

# Getting ready

The `tput` and `stty` commands are utilities used for terminal manipulations.

# How to do it...

Here are some capabilities of the `tput` command:

*   Return the number of columns and rows in a terminal:

```
 tput cols tput lines

```

*   Return the current terminal name:

```
 tput longname

```

*   Move the cursor to a 100,100 position:

```
 tput cup 100 100

```

*   Set the terminal background color:

```
 tput setb n

```

The value of `n` can be a value in the range of 0 to 7

*   Set the terminal foreground color:

```
 tput setf n

```

The value of `n` can be a value in the range of 0 to 7

Some commands including the common `color ls` may reset the foreground and background color.

*   Make text bold, using this command:

```
 tput bold

```

*   Perform start and end underlining:

```
 tput smul tput rmul

```

*   To delete from the cursor to the end of the line, use the following command:

```
 tput ed

```

*   A script should not display the characters while entering a password. The following example demonstrates disabling character echo with the `stty` command:

```
 #!/bin/sh #Filename: password.sh echo -e "Enter password: " # disable echo before reading password stty -echo read password # re-enable echo stty echo echo echo Password read.

```

The `-echo` option in the preceding command disables the output to the terminal, whereas `echo` enables output.

# Getting and setting dates and delays

A time delay is used to wait a set amount of time(such as 1 second) during the program execution, or to monitor a task every few seconds (or every few months). Working with times and dates requires an understanding of how time and date are represented and manipulated. This recipe will show you how to work with dates and time delays.

# Getting ready

Dates can be printed in a variety of formats. Internally, dates are stored as an integer number of seconds since 00:00:00 1970-01-01\. This is called **epoch** or **Unix time**.

The system's date can be set from the command line. The next recipes demonstrate how to read and set dates.

# How to do it...

It is possible to read the dates in different formats and also to set the date.

1.  Read the date:

```
 $ date Thu May 20 23:09:04 IST 2010

```

2.  Print the epoch time:

```
 $ date +%s 1290047248

```

The date command can convert many formatted date strings into the epoch time. This lets you use dates in multiple date formats as input. Usually, you don't need to bother about the date string format you use if you are collecting the date from a system log or any standard application generated output.
Convert the date string into epoch:

```
 $ date --date "Wed mar 15 08:09:16 EDT 2017" +%s 1489579718

```

The `--date` option defines a date string as input. We can use any date formatting options to print the output. The date command can be used to find the day of the week given a date string:

```
 $ date --date "Jan 20 2001" +%A Saturday

```

The date format strings are listed in the table mentioned in the *How it works...* section

3.  Use a combination of format strings prefixed with `+` as an argument for the `date` command, to print the date in the format of your choice. Consider this example:

```
 $ date "+%d %B %Y" 20 May 2010

```

4.  Set the date and time:                                                          

```
 # date -s "Formatted date string" # date -s "21 June 2009 11:01:22"

```

On a system connected to a network, you'll want to use `ntpdate` to set the date and time:
`/usr/sbin/ntpdate -s time-b.nist.gov`

5.  The rule for optimizing your code is to measure first. The date command can be used to time how long it takes a set of commands to execute:

```
 #!/bin/bash #Filename: time_take.sh start=$(date +%s) commands; statements; end=$(date +%s) difference=$(( end - start)) echo Time taken to execute commands is $difference seconds.

```

The date command's minimum resolution is one second. A better method for timing commands is the `time` command:
`time commandOrScriptName`.

# How it works...

The Unix epoch is defined as the number of seconds that have elapsed since midnight proleptic **Coordinated Universal Time** (**UTC**) of January 1, 1970, not counting leap seconds. Epoch time is useful when you need to calculate the difference between two dates or times. Convert the two date strings to epoch and take the difference between the epoch values. This recipe calculates the number of seconds between two dates:

```
secs1=`date -d "Jan 2 1970" 
secs2=`date -d "Jan 3 1970" 
echo "There are `expr $secs2 - $secs1` seconds between Jan 2 and Jan 3" 
There are 86400 seconds between Jan 2 and Jan 3 

```

Displaying a time in seconds since midnight of January 1, 1970, is not easily read by humans. The date command supports output in human readable formats.

The following table lists the format options that the date command supports.

| **Date component** | **Format** |
| --- | --- |
| Weekday | `%a` (for example, Sat)`%A` (for example, Saturday) |
| Month | `%b` (for example, Nov)`%B` (for example, November) |
| Day | `%d` (for example, 31) |
| Date in format (mm/dd/yy) | `%D` (for example, 10/18/10) |
| Year | `%y` (for example, 10)`%Y` (for example, 2010) |
| Hour | `%I` or `%H` (For example, 08) |
| Minute | `%M` (for example, 33) |
| Second | `%S` (for example, 10) |
| Nano second | `%N` (for example, 695208515) |
| Epoch Unix time in seconds | `%s` (for example, 1290049486) |

# There's more...

Producing time intervals is essential when writing monitoring scripts that execute in a loop. The following examples show how to generate time delays.

# Producing delays in a script

The sleep command will delay a script's execution period of time given in `seconds`. The following script counts from 0 to 40 seconds using `tput` and `sleep`:

```
#!/bin/bash 
#Filename: sleep.sh 
echo Count: 
tput sc 

# Loop for 40 seconds 
for count in `seq 0 40` 
do 
  tput rc 
  tput ed 
  echo -n $count 
  sleep 1 
done

```

In the preceding example, a variable steps through the list of numbers generated by the `seq` command. We use `tput sc` to store the cursor position. On every loop execution, we write the new count in the terminal by restoring the cursor position using `tput rc`, and then clearing to the end of the line with `tputs ed`. After the line is cleared, the script echoes the new value. The sleep command causes the script to delay for 1 second between each iteration of the loop.

# Debugging the script

Debugging frequently takes longer than writing code. A feature every programming language should implement is to produce trace information when something unexpected happens. Debugging information can be read to understand what caused the program to behave in an unexpected fashion. Bash provides debugging options every developer should know. This recipe shows how to use these options.

# How to do it...

We can either use Bash's inbuilt debugging tools or write our scripts in such a manner that they become easy to debug; here's how:

1.  Add the `-x` option to enable debug tracing of a shell script.

```
 $ bash -x script.sh

```

Running the script with the `-x` flag will print each source line with the current status.

You can also use `sh -x script`.

2.  Debug only portions of the script using `set -x` and `set +x`. Consider this example:

```
        #!/bin/bash 
        #Filename: debug.sh 
        for i in {1..6}; 
        do 
            set -x 
            echo $i 
            set +x 
        done 
        echo "Script executed"

```

In the preceding script, the debug information for `echo $i` will only be printed, as debugging is restricted to that section using `-x` and `+x`.
The script uses the `{start..end}` construct to iterate from a start to end value, instead of the `seq` command used in the previous example. This construct is slightly faster than invoking the `seq` command.

3.  The aforementioned debugging methods are provided by Bash built-ins. They produce debugging information in a fixed format. In many cases, we need debugging information in our own format. We can define a _DEBUG environment variable to enable and disable debugging and generate messages in our own debugging style.

Look at the following example code:

```
        #!/bin/bash 
        function DEBUG() 
        { 
            [ "$_DEBUG" == "on" ] && $@ || : 
        } 
        for i in {1..10} 
        do 
          DEBUG echo "I is $i" 
        done

```

Run the preceding script with debugging set to "on":

```
 $ _DEBUG=on ./script.sh

```

We prefix `DEBUG` before every statement where debug information is to be printed. If `_DEBUG=on` is not passed to the script, debug information will not be printed. In Bash, the command `:` tells the shell to do nothing.

# How it works...

The `-x` flag outputs every line of script as it is executed. However, we may require only some portions of the source lines to be observed. Bash uses a `set builtin` to enable and disable debug printing within the script:

*   `set -x`: This displays arguments and commands upon their execution
*   `set +x`: This disables debugging
*   `set -v`: This displays input when they are read
*   `set +v`: This disables printing input

# There's more...

We can also use other convenient ways to debug scripts. We can make use of shebang in a trickier way to debug scripts.

# Shebang hack

The shebang can be changed from `#!/bin/bash` to `#!/bin/bash -xv` to enable debugging without any additional flags (`-xv` flags themselves).

It can be hard to track execution flow in the default output when each line is preceded by `+`. Set the PS4 environment variable to `'$LINENO:'` to display actual line numbers:

```
PS4='$LINENO: ' 

```

The debugging output may be long. When using `-x` or set `-x`, the debugging output is sent to `stderr`. It can be redirected to a file with the following command:

```
sh -x testScript.sh 2> debugout.txt

```

Bash 4.0 and later support using a numbered stream for debugging output:

```
exec 6> /tmp/debugout.txt 
BASH_XTRACEFD=6

```

# Functions and arguments

Functions and aliases appear similar at a casual glance, but behave slightly differently. The big difference is that function arguments can be used anywhere within the body of the function, while an alias simply appends arguments to the end of the command.

# How to do it...

A function is defined with the function command, a function name, open/close parentheses, and a function body enclosed in curly brackets:

1.  A function is defined as follows:

```
        function fname() 
        { 
            statements; 
        }  

```

Alternatively, it can be defined as:

```
        fname() 
        { 
            statements; 
        } 

```

It can even be defined as follows (for simple functions):

```
        fname() { statement; }

```

2.  A function is invoked using its name:

```
 $ fname ; # executes function

```

3.  Arguments passed to functions are accessed positionally, `$1` is the first argument, `$2` is the second, and so on:

```
 fname arg1 arg2 ; # passing args

```

The following is the definition of the function `fname`. In the `fname` function, we have included various ways of accessing the function arguments.

```
        fname() 
        { 
           echo $1, $2; #Accessing arg1 and arg2 
           echo "$@"; # Printing all arguments as list at once 
           echo "$*"; # Similar to $@, but arguments taken as single  
           entity 
           return 0; # Return value 
         }

```

Arguments passed to scripts can be accessed as `$0` (the name of the script):

*   *   `$1` is the first argument
    *   `$2` is the second argument
    *   `$n` is the *n*th argument
    *   `"$@"` expands as `"$1" "$2" "$3"` and so on
    *   `"$*"` expands as `"$1c$2c$3"`, where `c` is the first character of IFS
    *   `"$@"` is used more often than `$*`, since the former provides all arguments as a single string
*   **Compare alias to function**
*   Here's an alias to display a subset of files by piping `ls` output to `grep`. The argument is attached to the end of the command, so `lsg txt` is expanded to `ls | grep txt`:

```
 $> alias lsg='ls | grep' 
 $> lsg txt 
 file1.txt 
 file2.txt 
 file3.txt 

```

*   If we wanted to expand that to get the IP address for a device in `/sbin/ifconfig`, we might try the following:

```
 $> alias wontWork='/sbin/ifconfig | grep' 
 $> wontWork eth0 
 eth0  Link  encap:Ethernet  HWaddr 00:11::22::33::44:55 

```

*   The `grep` command found the `eth0` string, not the IP address. If we use a function instead of an alias, we can pass the argument to the `ifconfig`, instead of appending it to the `grep`:

```
 $> function getIP() { /sbin/ifconfig $1 | grep 'inet ';  } 
 $> getIP eth0 
 inet addr:192.168.1.2 Bcast:192.168.255.255 Mask:255.255.0.0

```

# There's more...

Let's explore more tips on Bash functions.

# The recursive function

Functions in Bash also support recursion (the function can call itself). For example, `F() { echo $1; F hello; sleep 1; }`.

Fork bomb

A recursive function is a function that calls itself: recursive functions must have an exit condition, or they will spawn until the system exhausts a resource and crashes.

This function: `:(){ :|:& };:` spawns processes forever and ends up in a denial-of-service attack.

The `&` character is postfixed with the function call to bring the subprocess into the background. This dangerous code forks processes forever and is called a fork bomb.

You may find it difficult to interpret the preceding code. Refer to the Wikipedia page [h t t p ://e n . w i k i p e d i a . o r g /w i k i /F o r k _ b o m b](https://en.wikipedia.org/wiki/Fork_bomb) for more details and interpretation of the fork bomb.
Prevent this attack by restricting the maximum number of processes that can be spawned by defining the `nproc` value in `/etc/security/limits.conf`.

This line will limit all users to 100 processes:

 `hard nproc 100`

**Exporting functions**
Functions can be exported, just like environment variables, using the `export` command. Exporting extends the scope of the function to subprocesses:

```
export -f fname $> function getIP() { /sbin/ifconfig $1 | grep 'inet '; } $> echo "getIP eth0" >test.sh $> sh test.sh
 sh: getIP: No such file or directory $> export -f getIP $> sh test.sh
 inet addr: 192.168.1.2 Bcast: 192.168.255.255 Mask:255.255.0.0

```

# Reading the return value (status) of a command

The return value of a command is stored in the `$?` variable.

```
cmd; echo $?;

```

The return value is called **exit status**. This value can be used to determine whether a command completed successfully or unsuccessfully. If the command exits successfully, the exit status will be zero, otherwise it will be a nonzero value.

The following script reports the success/failure status of a command:

```
#!/bin/bash 
#Filename: success_test.sh 
# Evaluate the arguments on the command line - ie success_test.sh 'ls | grep txt' 
eval $@ 
if [ $? -eq 0 ]; 
then 
 echo "$CMD executed successfully" 
else 
 echo "$CMD terminated unsuccessfully" 
fi

```

# Passing arguments to commands

Most applications accept arguments in different formats. Suppose `-p` and `-v` are the options available, and `-k N` is another option that takes a number. Also, the command requires a filename as argument. This application can be executed in multiple ways:

*   `$ command -p -v -k 1 file`
*   `$ command -pv -k 1 file`
*   `$ command -vpk 1 file`
*   `$ command file -pvk 1` 

Within a script, the command-line arguments can be accessed by their position in the command line. The first argument will be `$1`, the second `$2`, and so on.
This script will display the first three command line arguments:

```
echo $1 $2 $3

```

It's more common to iterate through the command arguments one at a time. The `shift` command shifts eachh argument one space to the left, to let a script access each argument as `$1`. The following code displays all the command-line values:

```
$ cat showArgs.sh
for i in `seq 1 $#`
do
echo $i is $1
shift
done
$ sh showArgs.sh a b c
1 is a
2 is b
3 is c

```

# Sending output from one command to another

One of the best features of the Unix shells is the ease of combining many commands to produce a report. The output of one command can appear as the input to another, which passes its output to another command, and so on. The output of this sequence can be assigned to a variable. This recipe illustrates how to combine multiple commands and how the output can be read.

# Getting ready

The input is usually fed into a command through `stdin` or arguments. The output is sent to `stdout` or `stderr`. When we combine multiple commands, we usually supply input via `stdin` and generate output to `stdout`.

In this context, the commands are called **filters**. We connect each filter using pipes, sympolized by the piping operator (`|`), like this:

```
$ cmd1 | cmd2 | cmd3 

```

Here, we combine three commands. The output of `cmd1` goes to `cmd2`, the output of `cmd2` goes to `cmd3`, and the final output (which comes out of `cmd3`) will be displayed on the monitor, or directed to a file.

# How to do it...

Pipes can be used with the subshell method for combining outputs of multiple commands.

1.  Let's start with combining two commands:

```
 $ ls | cat -n > out.txt

```

The output of `ls` (the listing of the current directory) is passed to `cat -n`, which in turn prepends line numbers to the input received through `stdin`. The output is redirected to `out.txt`.

2.  Assign the output of a sequence of commands to a variable:

```
 cmd_output=$(COMMANDS)

```

This is called the **subshell method**. Consider this example:

```
 cmd_output=$(ls | cat -n) echo $cmd_output

```

Another method, called **back quotes** (some people also refer to it as **back tick**) can also be used to store the command output:

```
 cmd_output=`COMMANDS`

```

Consider this example:

```
 cmd_output=`ls | cat -n`
 echo $cmd_output

```

Back quote is different from the single-quote character. It is the character on the *~* button on the keyboard.

# There's more...

There are multiple ways of grouping commands.

# Spawning a separate process with subshell

Subshells are separate processes. A subshell is defined using the `( )` operators:

*   The `pwd` command prints the path of the working directory
*   The `cd` command changes the current directory to the given directory path:

```
        $> pwd 
        / 
        $> (cd /bin; ls) 
        awk bash cat... 
        $> pwd 
        /

```

When commands are executed in a subshell, none of the changes occur in the current shell; changes are restricted to the subshell. For example, when the current directory in a subshell is changed using the `cd` command, the directory change is not reflected in the main shell environment.

# Subshell quoting to preserve spacing and the newline character

Suppose we are assigning the output of a command to a variable using a subshell or the back quotes method, we must use double quotes to preserve the spacing and the newline character (`\n`). Consider this example:

```
$ cat text.txt 1 2 3 $ out=$(cat text.txt) $ echo $out 1 2 3 # Lost \n spacing in 1,2,3 $ out="$(cat text.txt)" $ echo $out 1 2 3

```

# Reading n characters without pressing the return key

The bash command `read` inputs text from the keyboard or standard input. We can use `read` to acquire input from the user interactively, but `read` is capable of more. Most input libraries in any programming language read the input from the keyboard and terminate the string when return is pressed. There are certain situations when return cannot be pressed and string termination is done based on a number of characters received (perhaps a single character). For example, in an interactive game, a ball is moved upward when *+* is pressed. Pressing *+* and then pressing *return* to acknowledge the *+* press is not efficient.

This recipe uses the `read` command to accomplish this task without having to press *return*.

# How to do it...

You can use various options of the `read` command to obtain different results, as shown in the following steps:

1.  The following statement will read *n* characters from input into the `variable_name` variable:

```
 read -n number_of_chars variable_name

```

Consider this example:

```
 $ read -n 2 var $ echo $var

```

2.  Read a password in the non-echoed mode:

```
 read -s var

```

3.  Display a message with `read` using the following command:

```
 read -p "Enter input:"  var

```

4.  Read the input after a timeout:

```
 read -t timeout var

```

Consider the following example:

```
 $ read -t 2 var # Read the string that is typed within 2 seconds into
        variable var.

```

5.  Use a delimiter character to end the input line:

```
 read -d delim_char var

```

 Consider this example:

```
 $ read -d ":" var hello:#var is set to hello

```

# Running a command until it succeeds

Sometimes a command can only succeed when certain conditions are met. For example, you can only download a file after the file is created. In such cases, one might want to run a command repeatedly until it succeeds.

# How to do it...

Define a function in the following way:

```
repeat() 
{ 
  while true 
  do 
    $@ && return 
  done 
}

```

Alternatively, add this to your shell's `rc` file for ease of use:

```
repeat() { while true; do $@ && return; done }

```

# How it works...

This repeat function has an infinite `while` loop, which attempts to run the command passed as a parameter (accessed by `$@`) to the function. It returns if the command was successful, thereby exiting the loop.

# There's more...

We saw a basic way to run commands until they succeed. Let's make things more efficient.

# A faster approach

On most modern systems, true is implemented as a binary in `/bin`. This means that each time the aforementioned `while` loop runs, the shell has to spawn a process. To avoid this, we can use the shell built-in `:` command, which always returns an exit code 0:

```
repeat() { while :; do $@ && return; done }

```

Though not as readable, this is faster than the first approach.

# Adding a delay

Let's say you are using `repeat()` to download a file from the Internet which is not available right now, but will be after some time. An example would be as follows:

```
repeat wget -c http://www.example.com/software-0.1.tar.gz

```

This script will send too much traffic to the web server at `www.example.com`, which causes problems for the server (and maybe for you, if the server blacklists your IP as an attacker). To solve this, we modify the function and add a delay, as follows:

```
repeat() { while :; do $@ && return; sleep 30; done }

```

This will cause the command to run every 30 seconds.

# Field separators and iterators

The **internal field separator** (**IFS**) is an important concept in shell scripting. It is useful for manipulating text data.

An IFS is a delimiter for a special purpose. It is an environment variable that stores delimiting characters. It is the default delimiter string used by a running shell environment.

Consider the case where we need to iterate through words in a string or **comma separated values** (**CSV**). In the first case, we will use `IFS=" "` and in the second, `IFS=","`.

# Getting ready

Consider the case of CSV data:

```
data="name,gender,rollno,location" 
To read each of the item in a variable, we can use IFS. 
oldIFS=$IFS 
IFS=, # IFS is now a , 
for item in $data; 
do 
    echo Item: $item 
done 

IFS=$oldIFS

```

This generates the following output:

```
Item: name Item: gender Item: rollno Item: location

```

The default value of IFS is a white-space (newline, tab, or a space character).

When IFS is set as `,` the shell interprets the comma as a delimiter character, therefore, the `$item` variable takes substrings separated by a comma as its value during the iteration.

If IFS is not set as `,` then it will print the entire data as a single string.

# How to do it...

Let's go through another example usage of IFS to parse the `/etc/passwd` file. In the `/etc/passwd` file, every line contains items delimited by `:`. Each line in the file corresponds to an attribute related to a user.

Consider the input: `root:x:0:0:root:/root:/bin/bash`. The last entry on each line specifies the default shell for the user.

Print users and their default shells using the IFS hack:

```
#!/bin/bash 
#Desc: Illustration of IFS 
line="root:x:0:0:root:/root:/bin/bash"  
oldIFS=$IFS; 
IFS=":" 
count=0 
for item in $line; 
do 

     [ $count -eq 0 ]  && user=$item; 
     [ $count -eq 6 ]  && shell=$item; 
    let count++ 
done; 
IFS=$oldIFS 
echo $user's shell is $shell;

```

The output will be as follows:

```
root's shell is /bin/bash

```

Loops are very useful in iterating through a sequence of values. Bash provides many types of loops.

*   **List-oriented `for` loop**:

```
        for var in list; 
        do 
            commands; # use $var 
        done 

```

A list can be a string or a sequence of values.

We can generate sequences with the `echo` command:

```
echo {1..50} ;# Generate a list of numbers from 1 to 50.
echo {a..z} {A..Z} ;# List of lower and upper case letters. 

```

We can combine these to concatenate data.
In the following code, in each iteration, the variable i will hold a character in the a to z range:

```
      for i in {a..z}; do actions; done;

```

*   **Iterate through a range of numbers**:

```
        for((i=0;i<10;i++)) 
        { 
           commands; # Use $i 
        }

```

*   **Loop until a condition is met**:

The while loop continues while a condition is true, the until loop runs until a condition is true:

```
        while condition 
        do 
            commands; 
        done

```

For an infinite loop, use `true` as the condition:

*   **Use a `until` loop**:

A special loop called `until` is available with Bash. This executes the loop until the given condition becomes true. Consider this example:

```
        x=0; 
        until [ $x -eq 9 ]; # [ $x -eq 9 ] is the condition 
        do 
            let x++; echo $x; 
        done

```

# Comparisons and tests

Flow control in a program is handled by comparison and test statements. Bash comes with several options to perform tests. We can use `if`, `if else`, and logical operators to perform tests and comparison operators to compare data items. There is also a command called `test`, which performs tests.

# How to do it...

Here are some methods used for comparisons and performing tests:

*   Use an `if` condition:

```
        if condition; 
        then 
            commands; 
        fi

```

*   Use `else if` and `else`:

```
        if condition;  
        then 
            commands; 
        else if condition; then 
            commands; 
        else 
            commands; 
        fi 

```

Nesting is possible with if and else. The if conditions can be lengthy; to make them shorter we can use logical operators:

`[ condition ] && action;` # action executes if the condition is true

`[ condition ] || action;` # action executes if the condition is false

`&&` is the logical AND operation and `||` is the logical OR operation. This is a very helpful trick while writing Bash scripts.
Performing mathematical comparisons: usually, conditions are enclosed in square brackets `[]`. Note that there is a space between `[` or `]` and operands. It will show an error if no space is provided.

```
[$var -eq 0 ] or [ $var -eq 0]

```

Perform mathematical tests on variables and values, like this:

```
[ $var -eq 0 ]  # It returns true when $var equal to 0\. 
[ $var -ne 0 ] # It returns true when $var is not equal to 0

```

Other important operators include the following:

*   `-gt`: Greater than
*   `-lt`: Less than
*   `-ge`: Greater than or equal to
*   `-le`: Less than or equal to

The `-a` operator is a logical AND and the `-o` operator is the logical OR. Multiple test conditions can be combined:

```
[ $var1 -ne 0 -a $var2 -gt 2 ]  # using and -a 
[ $var1 -ne 0 -o var2 -gt 2 ] # OR -o

```

Filesystem-related tests are as follows:

Test different filesystem-related attributes using different condition flags

*   `[ -f $file_var ]`: This returns true if the given variable holds a regular file path or filename
*   `[ -x $var ]`: This returns true if the given variable holds a file path or filename that is executable
*   `[ -d $var ]`: This returns true if the given variable holds a directory path or directory name
*   `[ -e $var ]`: This returns true if the given variable holds an existing file
*   `[ -c $var ]`: This returns true if the given variable holds the path of a character device file
*   `[ -b $var ]`: This returns true if the given variable holds the path of a block device file
*   `[ -w $var ]`: This returns true if the given variable holds the path of a file that is writable
*   `[ -r $var ]`: This returns true if the given variable holds the path of a file that is readable
*   `[ -L $var ]`: This returns true if the given variable holds the path of
    a symlink

Consider this example:

```
fpath="/etc/passwd" 
if [ -e $fpath ]; then 
    echo File exists;  
else 
    echo Does not exist;  
fi

```

String comparisons: When using string comparison, it is best to use double square brackets, since the use of single brackets can sometimes lead to errors

Note that the double square bracket is a Bash extension. If the script will be run using ash or dash (for better performance), you cannot use the double square.

**Test if two strings are identical**:

*   `[[ $str1 = $str2 ]]`: This returns true when `str1` equals `str2`, that is, the text contents of `str1` and `str2` are the same
*   `[[ $str1 == $str2 ]]`: It is an alternative method for string
    equality check

**Test if two strings are not identical**:

*   `[[ $str1 != $str2 ]]`: This returns true when `str1` and `str2` mismatch

Find alphabetically larger string:
Strings are compared alphabetically by comparing the ASCII value of the characters. For example, "A" is 0x41 and "a" is 0x61\. Thus "A" is less than "a", and "AAa" is less than "Aaa".

*   `[[ $str1 > $str2 ]]`: This returns true when `str1` is alphabetically greater than `str2`
*   `[[ $str1 < $str2 ]]`: This returns true when `str1` is alphabetically lesser than `str2`

A space is required after and before `=`; if it is not provided, it is not a comparison, but it becomes an assignment statement.

**Test for an empty string**:

*   `[[ -z $str1 ]]`: This returns true if `str1` holds an empty string
*   `[[ -n $str1 ]]`: This returns true if `str1` holds a nonempty string

It is easier to combine multiple conditions using logical operators such as `&&` and `||`, as in the following code:

```
if [[ -n $str1 ]] && [[ -z $str2 ]] ;
   then
       commands;
   fi

```

Consider this example:

```
str1="Not empty " 
str2="" 
if [[ -n $str1 ]] && [[ -z $str2 ]]; 
then 
    echo str1 is nonempty and str2 is empty string. 
fi

```

This will be the output:

```
str1 is nonempty and str2 is empty string.

```

The test command can be used for performing condition checks. This reduces the number of braces used and can make your code more readable. The same test conditions enclosed within `[]` can be used with the test command.

Note that test is an external program which must be forked, while [ is an internal function in Bash and thus more efficient. The test program is compatible with Bourne shell, ash, dash, and others.

Consider this example:

```
if  [ $var -eq 0 ]; then echo "True"; fi 
can be written as 
if  test $var -eq 0 ; then echo "True"; fi

```

# Customizing bash with configuration files

Most commands you type on the command line can be placed in a special file, to be evaluated when you log in or start a new bash session. It's common to customize your shell by putting function definitions, aliases, and environment variable settings in one of these files.

Common commands to put into a configuration file include the following:

```
# Define my colors for ls 
LS_COLORS='no=00:di=01;46:ln=00;36:pi=40;33:so=00;35:bd=40;33;01' 
export LS_COLORS 
# My primary prompt 
PS1='Hello $USER'; export PS1 
# Applications I install outside the normal distro paths 
PATH=$PATH:/opt/MySpecialApplication/bin; export PATH 
# Shorthand for commands I use frequently 
function lc () {/bin/ls -C $* ; }

```

**What customization file should I use?**

Linux and Unix have several files that might hold customization scripts. These configuration files are divided into three camps—those sourced on login, those evaluated when an interactive shell is invoked, and files evaluated whenever a shell is invoked to process a script file.

# How to do it...

These files are evaluated when a user logs into a shell:

```
/etc/profile, $HOME/.profile, $HOME/.bash_login, $HOME/.bash_profile /

```

Note that `/etc/profile`, `$HOME/.profile` and `$HOME/.bash_profile` may not be sourced if you log in via a graphical login manager. That's because the graphical window manager doesn't start a shell. When you open a terminal window, a shell is created, but it's not a login shell.

If a `.bash_profile` or `.bash_login` file is present, a `.profile` file will not be read.

These files will be read by an interactive shell such as a X11 terminal session or using `ssh` to run a single command like: `ssh 192.168.1.1 ls /tmp`.

```
/etc/bash.bashrc $HOME/.bashrc

```

Run a shell script like this:

```
$> cat myscript.sh 
#!/bin/bash 
echo "Running"

```

None of these files will be sourced unless you have defined the `BASH_ENV` environment variable:

```
$> export BASH_ENV=~/.bashrc 
$> ./myscript.sh

```

Use `ssh` to run a single command, as with the following:

```
ssh 192.168.1.100 ls /tmp

```

This will start a bash shell which will evaluate `/etc/bash.bashrc` and `$HOME/.bashrc`, but not `/etc/profile` or `.profile`.

Invoke a ssh login session, like this:

```
ssh 192.168.1.100

```

This creates a new login bash shell, which will evaluate the following:

```
/etc/profile 
/etc/bash.bashrc 
$HOME/.profile or .bashrc_profile

```

DANGER: Other shells, such as the traditional Bourne shell, ash, dash, and ksh, also read this file. Linear arrays (lists) and associative arrays, are not supported in all shells. Avoid using these in `/etc/profile` or `$HOME/.profile`.

Use these files to define non-exported items such as aliases desired by all users. Consider this example:

```
alias l "ls -l"
/etc/bash.bashrc /etc/bashrc

```

Use these files to hold personal settings. They are useful for setting paths that must be inherited by other bash instances. They might include lines like these:

```
CLASSPATH=$CLASSPATH:$HOME/MyJavaProject; export CLASSPATH
$HOME/.bash_login $HOME/.bash_profile $HOME/.profile

```

If `.bash_login` or `.bash_profile` are present, `.profile` will not be read. A `.profile` file may be read by other shells.

Use these files to hold your personal values that need to be defined whenever a new shell is created. Define aliases and functions here if you want them available in an X11 terminal session:

```
$HOME/.bashrc, /etc/bash.bashrc

```

Exported variables and functions are propagated to subordinate shells, but aliases are not. You must define `BASH_ENV` to be the `.bashrc` or `.profile`, where aliases are defined in order to use them in a shell script.

This file is evaluated when a user logs out of a session:

```
$HOME/.bash_logout

```

For example, if the user logs in remotely they should clear the screen when they log out.

```
$> cat ~/.bash_logout 
# Clear the screen after a remote login/logout. 
clear

```