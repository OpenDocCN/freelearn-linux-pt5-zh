# Shell 做一些事

在本章中，我们将涵盖以下内容：

+   在终端中显示输出

+   使用变量和环境变量

+   向环境变量添加前缀的函数

+   使用 shell 进行数学运算

+   玩转文件描述符与重定向

+   数组和关联数组

+   访问别名

+   获取有关终端的信息

+   获取和设置日期与延迟

+   调试脚本

+   函数与参数

+   将一个命令的输出发送到另一个命令

+   在不按回车键的情况下读取 `n` 个字符

+   一直运行命令直到成功

+   字段分隔符和迭代器

+   比较与测试

+   通过配置文件自定义 bash

# 简介

起初，计算机通过读取卡片或磁带中的程序并生成单个报告来工作。当时没有操作系统、图形显示器，甚至没有交互式提示符。

到了 1960 年代，计算机支持交互式终端（通常是电传打字机或升级版打字机）来调用命令。

当贝尔实验室为全新的 Unix 操作系统创建交互式用户界面时，它具有一个独特的功能。它能够读取和评估来自文本文件（称为 shell 脚本）的相同命令，就像它接受从终端键入的命令一样。

这个功能在生产力上是一次巨大的飞跃。程序员无需输入多个命令来执行一组操作，而是可以将命令保存在文件中，并只需通过几次按键就能运行它们。Shell 脚本不仅节省了时间，还记录了你所做的操作。

最初，Unix 只支持一个交互式 shell，由 Stephen Bourne 编写，并命名为 **Bourne Shell**（**sh**）。

1989 年，GNU 项目的 Brian Fox 从许多用户界面中借鉴了特性，并创建了一个新的 shell —— **Bourne Again Shell**（**bash**）。bash shell 理解所有 Bourne shell 的构造，并添加了来自 csh、ksh 和其他 shell 的特性。

随着 Linux 成为最流行的类 Unix 操作系统实现，bash shell 已成为 Unix 和 Linux 的事实标准 shell。

本书聚焦于 Linux 和 bash。尽管如此，大多数脚本可以在 Linux 和 Unix 上运行，使用 bash、sh、ash、dash、ksh 或其他 sh 风格的 shell。

本章将为读者提供关于 shell 环境的洞察，并演示一些基本的 shell 特性。

# 在终端中显示输出

用户通过终端会话与 shell 环境进行交互。如果你使用的是基于 GUI 的系统，那么显示的将是一个终端窗口。如果你在没有 GUI 的环境下工作（例如生产服务器或 ssh 会话），你一登录就会看到 shell 提示符。

在终端中显示文本是大多数脚本和实用工具需要定期执行的任务。Shell 支持多种方法和不同格式来显示文本。

# 准备工作

命令在终端会话中输入并执行。当终端被打开时，会显示一个提示符。提示符可以通过多种方式进行配置，但通常如下所示：

```
username@hostname$

```

另外，也可以将其配置为 `root@hostname #`，或者仅仅是 `$` 或 `#`。

`$` 字符代表普通用户，`#` 代表管理员用户 root。Root 是 Linux 系统中权限最高的用户。

直接以根用户（管理员）身份使用 shell 执行任务是一个不好的主意。因为当你的 shell 拥有更多权限时，输入错误可能会造成更大的损害。建议以普通用户身份登录（你的 shell 提示符可能会显示为 `$`），并使用像 `sudo` 这样的工具来执行特权命令。使用 `sudo <command> <arguments>` 以 root 身份运行命令。

一个 shell 脚本通常以 shebang 开头：

```
#!/bin/bash

```

Shebang 是一行，在该行前缀加上 `#!`，后面跟着解释器路径。`/bin/bash` 是 Bash 的解释器命令路径。以 `#` 符号开头的行被 bash 解释器视为注释。只有脚本的第一行可以包含 shebang 来定义用于评估脚本的解释器。

脚本可以通过两种方式执行：

1.  将脚本名称作为命令行参数传递：

```
 bash myScript.sh

```

1.  设置脚本文件的执行权限，使其可以执行：

```
 chmod 755 myScript.sh ./myScript.sh.

```

如果脚本作为 `bash` 的命令行参数运行，则不需要 shebang。shebang 使得脚本可以独立运行。可执行脚本使用 shebang 后面的解释器路径来解释脚本。

脚本通过 `chmod` 命令使其可执行：

```
$ chmod a+x sample.sh

```

这个命令使得脚本对所有用户可执行。脚本可以按如下方式执行：

```
$ ./sample.sh #./ represents the current directory

```

另外，脚本也可以像这样执行：

```
$ /home/path/sample.sh # Full path of the script is used

```

内核会读取第一行，看到 shebang 为 `#!/bin/bash`，它会识别 `/bin/bash` 并按如下方式执行脚本：

```
$ /bin/bash sample.sh

```

当交互式 shell 启动时，它会执行一组命令来初始化设置，例如提示文本、颜色等。这些命令从位于用户家目录中的 `~/.bashrc`（或登录 shell 时的 `~/.bash_profile`）脚本中读取。Bash shell 会在 `~/.bash_history` 文件中保存用户运行的命令历史记录。

`~` 符号表示你的家目录，通常是 `/home/user`，其中 `user` 是你的用户名，或者是 `/root`，对于 root 用户。登录 shell 在你登录计算机时会创建。然而，在图形化环境中登录后创建的终端会话（如 GNOME、KDE 等）并不是登录 shell。通过显示管理器（如 GDM 或 KDM）登录时，可能不会读取 `.profile` 或 `.bash_profile`（大多数不会），但通过 ssh 登录远程系统时会读取 `.profile`。shell 通过分号或新的一行来分隔每个命令或命令序列。请看这个例子：`$ cmd1 ; cmd2`

这等同于以下内容：

`$ cmd1`

`$ cmd2`

注释以`#`开始，并持续到行尾。注释行通常用于描述代码，或在调试时禁用某行代码的执行：

```
# sample.sh - echoes "hello world" echo "hello world"

```

现在让我们继续学习本章的基本示例。

# 如何做到...

`echo`命令是终端中最简单的打印命令。

默认情况下，`echo`在每次调用后会添加换行符：

```
$ echo "Welcome to Bash" Welcome to Bash

```

简单来说，使用双引号括起来的文本和`echo`命令一起打印文本到终端。类似地，未使用双引号的文本也会输出相同的结果：

```
$ echo Welcome to Bash Welcome to Bash

```

另一种完成相同任务的方法是使用单引号：

```
$ echo 'text in quotes'

```

这些方法看起来相似，但每个方法都有特定的用途和副作用。双引号允许 shell 解释字符串中的特殊字符。单引号则禁用这种解释。

请考虑以下命令：

```
$ echo "cannot include exclamation - ! within double quotes"

```

这将返回以下输出：

```
bash: !: event not found error

```

如果你需要打印像`!`这样的特殊字符，必须要么不使用任何引号，要么使用单引号，或者用反斜杠（`\`）转义特殊字符：

```
$ echo Hello world ! 

```

或者，使用这个：

```
$ echo 'Hello world !'

```

或者，可以这样使用：

```
$ echo "Hello World\!" #Escape character \ prefixed.

```

当使用不带引号的`echo`时，不能使用分号，因为分号是 Bash shell 中命令之间的分隔符：

```
echo hello; hello 

```

从前面的行来看，Bash 将`echo hello`视为一个命令，第二个`hello`视为另一个命令。

在下一个示例中讨论的变量替换，在单引号内将不起作用。

另一个在终端中打印的命令是`printf`。它使用与 C 库中的`printf`函数相同的参数。请考虑以下示例：

```
$ printf "Hello world"

```

`printf`命令接受用引号括起来的文本或由空格分隔的参数。它支持格式化字符串。格式字符串指定了字符串宽度、左对齐或右对齐等。默认情况下，`printf`不会附加换行符。当需要时，我们必须显式地指定换行符，如以下脚本所示：

```
#!/bin/bash #Filename: printf.sh printf  "%-5s %-10s %-4s\n" No Name  Mark printf  "%-5s %-10s %-4.2f\n" 1 Sarath 80.3456 printf  "%-5s %-10s %-4.2f\n" 2 James 90.9989 printf  "%-5s %-10s %-4.2f\n" 3 Jeff 77.564

```

我们将收到以下格式化输出：

```
No    Name       Mark 1     Sarath     80.35 2     James      91.00 3     Jeff       77.56

```

# 它是如何工作的...

`%s`、`%c`、`%d`和`%f`字符是格式替换字符，用于定义后续参数的打印方式。`%-5s`字符串定义了一个左对齐的字符串替换（`-`表示左对齐），并且字符宽度为`5`。如果没有指定`-`，字符串将会右对齐。宽度指定了为字符串保留的字符数。对于`Name`，保留的宽度是`10`。因此，任何名称都会位于为其保留的 10 个字符宽度内，剩余部分将填充空格，直到总共有 10 个字符。

对于浮点数，我们可以传递附加参数来四舍五入小数位数。

对于 Mark 部分，我们将字符串格式化为`%-4.2f`，其中`.2`表示四舍五入到小数点后两位。请注意，对于每一行的格式化字符串，都会发出一个换行符（`\n`）。

# 还有更多...

在使用`echo`和`printf`的标志时，应将标志放在命令中的任何字符串之前，否则 Bash 会将标志视为另一个字符串。

# 在 echo 中转义换行符

默认情况下，`echo`会在输出文本的末尾附加一个换行符。使用`-n`标志可以禁用换行符。`echo`命令接受双引号字符串作为参数，并可以处理转义序列。使用转义序列时，使用`echo`命令为`echo -e "包含转义序列的字符串"`。考虑以下示例：

```
echo -e "1\t2\t3" 1  2  3

```

# 打印彩色输出

脚本可以使用转义序列在终端上生成彩色文本。

文本的颜色由颜色代码表示，包括 reset = 0，black = 30，red = 31，green = 32，yellow = 33，blue = 34，magenta = 35，cyan = 36，white = 37。

要打印彩色文本，请输入以下命令：

```
echo -e "\e[1;31m This is red text \e[0m"

```

这里，`\e[1;31m`是设置颜色为红色的转义字符串，而`\e[0m`将颜色重置。将`31`替换为所需的颜色代码。

对于彩色背景，reset = 0，black = 40，red = 41，green = 42，yellow = 43，blue = 44，magenta = 45，cyan = 46，white = 47，是常用的颜色代码。

要打印彩色背景，请输入以下命令：

```
echo -e "\e[1;42m Green Background \e[0m"

```

这些示例涵盖了部分转义序列。文档可以通过`man console_codes`查看。

# 使用变量和环境变量

所有编程语言都使用变量来保存数据，以便稍后使用或修改。与编译语言不同，大多数脚本语言在创建变量之前不需要声明类型。类型由使用情况决定。通过在变量名前加上美元符号来访问变量的值。Shell 定义了多个它用于配置和信息的变量，例如可用的打印机、搜索路径等。这些被称为**环境变量**。

# 准备中

变量命名规则是字母、数字和下划线的组合，不允许有空格。常见的约定是环境变量使用大写字母（UPPER_CASE），而在脚本中使用的变量使用驼峰命名法或小写字母（camelCase 或 lower_case）。

所有应用程序和脚本都可以访问环境变量。要查看当前 Shell 中定义的所有环境变量，请执行`env`或`printenv`命令：

```
$> env 
PWD=/home/clif/ShellCookBook 
HOME=/home/clif 
SHELL=/bin/bash 
# ... And many more lines

```

要查看其他进程的环境，请使用以下命令：

```
cat /proc/$PID/environ

```

设置`PID`为进程的进程 ID（`PID`是一个整数值）。

假设一个名为`gedit`的应用程序正在运行。我们通过`pgrep`命令获取`gedit`的进程 ID：

```
$ pgrep gedit 12501

```

我们通过执行以下命令查看与进程相关的环境变量：

```
$ cat /proc/12501/environ GDM_KEYBOARD_LAYOUT=usGNOME_KEYRING_PID=1560USER=slynuxHOME=/home/slynux

```

注意，前面的输出有许多行被去除以方便阅读。实际输出包含更多的变量。

`/proc/PID/environ`特殊文件包含环境变量及其值的列表。每个变量以 name=value 对的形式表示，两个部分之间由空字符（`\0`）分隔。这对于人类来说并不容易阅读。

要生成易于阅读的报告，可以将`cat`命令的输出通过管道传递给`tr`，将`\0`字符替换为`\n`：

```
$ cat /proc/12501/environ  | tr '\0' '\n'

```

# 如何做到…

使用等号操作符为变量赋值：

```
varName=value

```

变量的名称是`varName`，`value`是要赋给它的值。如果`value`中不包含空格字符（如空格），则不需要用引号括起来；否则，必须用单引号或双引号括起来。

请注意，`var = value`和`var=value`是不同的。将`var = value`写成`var=value`是常见错误。没有空格的等号是赋值操作，而使用空格会创建一个相等性测试。

通过在变量名前加上美元符号（`$`）来访问变量的内容。

```
var="value" #Assign "value" to var echo $var

```

你也可以这样使用：

```
echo ${var}

```

该输出将显示如下：

```
value

```

双引号中的变量值可以与`printf`、`echo`和其他 Shell 命令一起使用：

```
#!/bin/bash #Filename :variables.sh fruit=apple count=5 echo "We have $count ${fruit}(s)"

```

输出将如下所示：

```
We have 5 apple(s)

```

由于 Shell 使用空格来分隔单词，我们需要添加花括号，以让 Shell 知道变量名是`fruit`，而不是`fruit(s)`。

环境变量是从父进程继承的。例如，`HTTP_PROXY`是一个环境变量，定义了用于 Internet 连接的代理服务器。

通常，它被设置为如下：

```
HTTP_PROXY=192.168.1.23:3128 export HTTP_PROXY

```

`export`命令声明一个或多个将由子任务继承的变量。变量被导出后，当前 Shell 脚本中执行的任何应用程序都会接收到这些变量。Shell 创建并使用了许多标准环境变量，我们也可以导出自己的变量。

例如，`PATH`变量列出了 Shell 搜索应用程序的文件夹。一个典型的`PATH`变量包含以下内容：

```
$ echo $PATH 
/home/slynux/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

```

目录路径由`:`字符分隔。通常，`$PATH`在`/etc/environment`、`/etc/profile`或`~/.bashrc`中定义。

要将新路径添加到`PATH`环境中，使用以下命令：

```
export PATH="$PATH:/home/user/bin"

```

或者，使用这些命令：

```
$ PATH="$PATH:/home/user/bin" 
$ export PATH 
$ echo $PATH 
/home/slynux/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/home/user/bin

```

这里我们将`/home/user/bin`添加到`PATH`中。

一些著名的环境变量有`HOME`、`PWD`、`USER`、`UID`和`SHELL`。

使用单引号时，变量不会展开，显示的就是原样内容。这意味着，`$ echo '$var'`将显示`$var`。

而`$ echo "$var"`将在`$var`变量已定义时显示其值，未定义时显示空值。

# 还有更多...

Shell 还有许多内置功能。以下是一些额外功能：

# 查找字符串的长度

使用以下命令获取变量值的长度：

```
length=${#var}

```

考虑这个例子：

```
$ var=12345678901234567890$ echo ${#var} 20

```

`length`参数表示字符串中的字符数。

# 识别当前 Shell

要识别当前使用的 Shell，可以使用`SHELL environment`变量。

```
echo $SHELL

```

或者，使用以下命令：

```
echo $0

```

考虑这个例子：

```
$ echo $SHELL /bin/bash

```

同样，通过执行`echo $0`命令，我们将得到相同的输出：

```
$ echo $0 /bin/bash

```

# 检查超级用户

`UID`环境变量保存用户 ID。使用此值可以检查当前脚本是以 root 用户还是普通用户身份运行。考虑这个例子：

```
If [ $UID -ne 0 ]; then 
  echo Non root user. Please run as root. 
else 
  echo Root user 
fi

```

请注意，`[` 实际上是一个命令，必须与其余字符串用空格分开。我们也可以将前面的脚本写成如下形式：

```
if test $UID -ne 0:1 
  then 
    echo Non root user. Please run as root 
  else 
    echo Root User 
fi

```

root 用户的 `UID` 值为 `0`。

# 修改 Bash 提示符字符串（username@hostname:~$）

当我们打开终端或运行 shell 时，会看到类似 `user@hostname: /home/$` 的提示符。不同的 GNU/Linux 发行版有不同的提示符和颜色。`PS1` 环境变量定义了主提示符。默认的提示符由 `~/.bashrc` 文件中的一行定义。

+   查看设置 `PS1` 变量的行：

```
        $ cat ~/.bashrc | grep PS1 
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '

```

+   要修改提示符，可以输入以下命令：

```
        slynux@localhost: ~$ PS1="PROMPT> " # Prompt string changed 
        PROMPT> Type commands here.

```

+   我们可以使用特殊的转义序列来显示彩色文本，例如 `\e[1;31`（参见本章的 *在终端中显示输出* 这一小节）。

某些特殊字符会扩展为系统参数。例如，`\u` 扩展为用户名，`\h` 扩展为主机名，`\w` 扩展为当前工作目录。

# 用于向环境变量添加前缀的函数

环境变量通常用于存储一系列搜索可执行文件、库等的路径。例如，`$PATH` 和 `$LD_LIBRARY_PATH` 通常类似于下面的形式：

```
PATH=/usr/bin;/bin 
LD_LIBRARY_PATH=/usr/lib;/lib

```

这意味着，每当 shell 需要执行一个应用程序（二进制文件或脚本）时，它会首先在 `/usr/bin` 中查找，然后再搜索 `/bin`。

在从源代码构建并安装程序时，我们经常需要为新的可执行文件和库添加自定义路径。例如，我们可能会将 `myapp` 安装在 `/opt/myapp` 中，其中二进制文件在 `/opt/myapp/bin` 文件夹，库文件在 `/opt/myapp/lib` 文件夹。

# 如何做到这一点...

这个示例展示了如何将新路径添加到环境变量的开头。第一个示例展示了如何使用目前为止讲解的内容来实现，第二个示例展示了如何创建一个函数来简化修改变量的过程。函数的内容将在本章后面介绍。

```
export PATH=/opt/myapp/bin:$PATH 
export LD_LIBRARY_PATH=/opt/myapp/lib;$LD_LIBRARY_PATH

```

`PATH` 和 `LD_LIBRARY_PATH` 变量现在应该类似于以下内容：

```
PATH=/opt/myapp/bin:/usr/bin:/bin 
LD_LIBRARY_PATH=/opt/myapp/lib:/usr/lib;/lib

```

我们可以通过在 `.bashrc` 文件中定义一个 prepend 函数，使得添加新路径更加简单。

```
prepend() { [ -d "$2" ] && eval $1=\"$2':'\$$1\" && export $1; }

```

可以通过以下方式使用：

```
prepend PATH /opt/myapp/bin 
prepend LD_LIBRARY_PATH /opt/myapp/lib

```

# 它是如何工作的...

`prepend()` 函数首先确认由函数第二个参数指定的目录是否存在。如果存在，`eval` 表达式会设置变量，变量名为第一个参数的值，等于第二个参数字符串，后跟 `:`（路径分隔符），然后是变量的原始值。

如果在尝试添加前缀时变量为空，末尾会有一个多余的 `:`。为了解决这个问题，可以将函数修改为如下形式：

```
prepend() { [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1 ; }

```

在这种形式的函数中，我们引入了一种 shell 参数扩展，形式如下：

`${parameter:+expression}`

如果参数已设置且不为 null，则扩展为表达式。

通过此更改，我们确保在尝试添加前缀时，如果旧值存在，则仅在旧值存在的情况下追加 `:` 和旧值。

# 与 shell 进行数学运算

Bash shell 使用`let`、`(( ))`和`[]`命令执行基本的算术操作。`expr`和`bc`工具用于执行高级操作。

# 如何做...

1.  数值的赋值与字符串赋值相同。访问它的方法会将其作为数字处理：

```
 #!/bin/bash no1=4; no2=5;

```

1.  `let`命令用于直接执行基本操作。在`let`命令内，我们使用没有`$`前缀的变量名。请参考这个例子：

```
 let result=no1+no2 echo $result 

```

`let`命令的其他用法如下：

+   使用此命令进行递增：

```
 $ let no1++

```

+   递减操作使用此命令：

```
 $ let no1--

```

+   使用这些来表示简写：

```
 let no+=6 let no-=6

```

这些等同于`let no=no+6`和`let no=no-6`。

+   替代方法如下：

`[]`操作符的使用方式与`let`命令相同：

```
 result=$[ no1 + no2 ]

```

在`[]`操作符内部使用`$`前缀是合法的；请参考这个例子：

```
 result=$[ $no1 + 5 ]

```

`(( ))`操作符也可以使用。变量名前缀带有`$`的方式在`(( ))`操作符内使用：

```
 result=$(( no1 + 50 ))

```

`expr`表达式可用于基本操作：

```
 result=`expr 3 + 4` result=$(expr $no1 + 5)

```

前述方法不支持浮动点数，

仅对整数操作。

1.  `bc`应用程序，精度计算器，是一个用于数学运算的高级工具。它有广泛的选项，我们可以进行浮点运算并使用高级函数：

```
 echo "4 * 0.56" | bc 2.24 no=54; result=`echo "$no * 1.5" | bc` echo $result 81.0

```

`bc`应用程序接受前缀来控制操作。这些前缀通过分号分隔。

+   **小数位数使用 bc 调整**：在下面的例子中，`scale=2`参数将小数位数设置为`2`。因此，`bc`的输出将包含一个两位小数的数字：

```
 echo "scale=2;22/7" | bc 3.14

```

+   **使用 bc 进行进制转换**：我们可以将一个进制的数字转换为另一个进制。以下代码将数字从十进制转换为二进制，并将二进制转换为十进制：

```
 #!/bin/bash Desc: Number conversion no=100 echo "obase=2;$no" | bc 1100100 no=1100100 echo "obase=10;ibase=2;$no" | bc 100

```

+   以下示例演示了如何计算平方和平方根：

```
 echo "sqrt(100)" | bc #Square root echo "10¹⁰" | bc #Square

```

# 操作文件描述符和重定向

文件描述符是与输入输出流相关联的整数。最常见的文件描述符是`stdin`、`stdout`和`stderr`。一个流的内容可以被重定向到另一个流。这个示例展示了如何操作和重定向文件描述符。

# 准备就绪

Shell 脚本常常使用标准输入（`stdin`）、标准输出（`stdout`）和标准错误（`stderr`）。脚本可以使用大于号符号将输出重定向到文件。命令生成的文本可以是正常输出或错误消息。默认情况下，正常输出（`stdout`）和错误消息（`stderr`）都会显示在屏幕上。通过为每个流指定特定的描述符，两个流可以被分开。

文件描述符是与打开的文件或数据流相关联的整数。文件描述符 0、1 和 2 是保留的，如下所示：

+   0: `stdin`

+   1: `stdout`

+   2: `stderr`

# 如何做...

1.  使用大于号符号将文本追加到文件：

```
 $ echo "This is a sample text 1" > temp.txt

```

这会将回显的文本存储到`temp.txt`中。如果`temp.txt`已存在，单个大于符号会删除之前的内容。

1.  使用双大于符号将文本追加到文件中：

```
 $ echo "This is sample text 2" >> temp.txt

```

1.  使用`cat`命令查看文件内容：

```
 $ cat temp.txt This is sample text 1 This is sample text 2

```

接下来的示例演示了如何重定向`stderr`。当命令生成错误信息时，会将一条消息打印到`stderr`流中。考虑以下示例：

```
$ ls + ls: cannot access +: No such file or directory

```

这里`+`是一个无效参数，因此返回错误。

成功与失败的命令

当一个命令因错误退出时，它会返回一个非零的退出状态。命令在成功完成后退出时返回零。返回状态可以在特殊变量`$?`中获得（在执行命令后立即运行`echo $?`来打印退出状态）。

以下命令将`stderr`文本打印到屏幕上，而不是文件中（由于没有`stdout`输出，`out.txt`将为空）：

```
$ ls + > out.txt ls: cannot access +: No such file or directory 

```

在以下命令中，我们使用`2>`（两个大于符号）将`stderr`重定向到`out.txt`：

```
$ ls + 2> out.txt # works

```

你可以将`stderr`重定向到一个文件，将`stdout`重定向到另一个文件。

```
$ cmd 2>stderr.txt 1>stdout.txt

```

也可以通过将`stderr`转换为`stdout`来将`stderr`和`stdout`重定向到同一个文件，使用这种首选方法：

```
$ cmd 2>&1 allOutput.txt

```

也可以通过另一种方法来完成此操作：

```
$ cmd &> output.txt 

```

如果你不想看到或保存任何错误消息，可以将`stderr`输出重定向到`/dev/null`，这样可以完全去除它。例如，假设我们有三个文件`a1`、`a2`和`a3`，但是`a1`对用户没有读写执行权限。为了打印所有以字母`a`开头的文件内容，我们使用`cat`命令。可以按如下方式设置测试文件：

```
$ echo A1 > a1 $ echo A2 > a2 $ echo A3 > a3 $ chmod 000 a1  #Deny all permissions

```

使用通配符(`a*`)显示文件内容时，将会因为`a1`文件没有适当的读权限而生成错误消息：

```
$ cat a* cat: a1: Permission denied A2 A3

```

这里，`cat: a1: Permission denied`属于`stderr`数据。我们可以将`stderr`数据重定向到一个文件，同时将`stdout`输出到终端。

```
$ cat a* 2> err.txt #stderr is redirected to err.txt A2 A3 $ cat err.txt cat: a1: Permission denied

```

有些命令生成我们想要处理的输出，并且还希望将其保存以供将来参考或进一步处理。`stdout`流是一个单一的流，我们可以将其重定向到文件或通过管道传递给另一个程序。你可能认为我们无法做到既得其所，又能两全其美。

然而，也有一种方法可以将数据重定向到文件的同时，将重定向的数据作为`stdin`提供给管道中的下一个命令。`tee`命令从`stdin`读取并将输入数据重定向到`stdout`和一个或多个文件。

```
command | tee FILE1 FILE2 | otherCommand

```

在以下代码中，`stdin`数据由`tee`命令接收。它将`stdout`的副本写入`out.txt`文件，并将另一副本作为`stdin`传递给下一个命令。`cat -n`命令为每一行从`stdin`接收的内容加上行号，并写入`stdout`：

```
$ cat a* | tee out.txt | cat -n cat: a1: Permission denied
 1 A2 2 A3

```

使用`cat`命令查看`out.txt`的内容：

```
$ cat out.txt A2 A3

```

注意到 `cat: a1: Permission denied` 不会显示，因为它被发送到了 `stderr`。`tee` 命令仅从 `stdin` 读取。

默认情况下，`tee` 命令会覆盖文件。加入 `-a` 选项会强制它将新数据附加到文件末尾。

```
$ cat a* | tee -a out.txt | cat -n

```

带有参数的命令遵循格式：`command FILE1 FILE2 ...` 或简单的 `command FILE`。

要将输入的两份副本发送到 `stdout`，请使用 `-` 作为文件名参数：

```
$ cmd1 | cmd2 | cmd -

```

参考此示例：

```
$ echo who is this | tee - who is this who is this

```

或者，我们可以使用 `/dev/stdin` 作为输出文件名来使用 `stdin`。

同样，使用 `/dev/stderr` 表示标准错误，`/dev/stdout` 表示标准输出。这些是特殊的设备文件，分别对应 `stdin`、`stderr` 和 `stdout`。

# 它是如何工作的…

重定向操作符（`>` 和 `>>`）将输出发送到文件，而不是终端。`>` 和 `>>` 操作符的行为略有不同。两者都将输出重定向到文件，但单个大于号符号（`>`）会清空文件并写入新内容，而双大于号符号（`>>`）会将输出添加到现有文件的末尾。

默认情况下，重定向作用于标准输出。要明确使用特定的文件描述符，必须将描述符数字作为前缀添加到操作符。

`>` 操作符等同于 `1>`，类似地，`>>` 也等同于 `1>>`。

在处理错误时，`stderr` 输出会被转储到 `/dev/null` 文件中。`./dev/null` 文件是一个特殊的设备文件，接收到的数据会被丢弃。这个空设备通常被称为 **黑洞**，因为所有进入它的数据都会永远丢失。

# 还有更多内容...

从 `stdin` 读取输入的命令可以通过多种方式接收数据。我们可以使用 `cat` 和管道来指定我们自己的文件描述符。参考此示例：

```
$ cat file | cmd $ cmd1 | cmd2

```

# 从文件重定向到命令

我们可以通过小于符号（`<`）从文件中读取数据作为 `stdin`：

```
$ cmd < file

```

# 从脚本中封闭的文本块进行重定向

文本可以从脚本重定向到文件。要在自动生成的文件顶部添加警告，请使用以下代码：

```
#!/bin/bash 
cat<<EOF>log.txt 
This is a generated file. Do not edit. Changes will be overwritten. 
EOF

```

出现于 `cat <<EOF >log.txt` 和下一个 `EOF` 行之间的内容将作为 `stdin` 数据显示。`log.txt` 的内容如下：

```
$ cat log.txt 
This is a generated file. Do not edit. Changes will be overwritten. 

```

# 自定义文件描述符

文件描述符是访问文件的抽象标识符。每次文件访问都会关联一个称为文件描述符的特殊数字。0、1 和 2 是保留的描述符数字，分别代表 `stdin`、`stdout` 和 `stderr`。

`exec` 命令可以创建新的文件描述符。如果你熟悉其他编程语言中的文件访问，你可能已经了解了打开文件的模式。这三种模式通常使用：

+   读模式

+   使用追加模式写入

+   使用截断模式写入

`<` 运算符将文件内容读取到 `stdin`。`>` 运算符将数据写入文件并截断（数据写入目标文件，内容会被截断）。`>>` 运算符将数据追加到文件中（数据附加到现有文件内容，不会丢失目标文件的内容）。文件描述符是通过这三种模式之一创建的。

创建一个用于读取文件的文件描述符：

```
$ exec 3<input.txt # open for reading with descriptor number 3

```

我们可以这样使用：

```
$ echo this is a test line > input.txt $ exec 3<input.txt

```

现在，你可以使用文件描述符`3`与命令一起操作。例如，我们将使用`cat<&3`：

```
$ cat<&3 this is a test line

```

如果需要第二次读取，不能重用文件描述符`3`。我们必须使用`exec`创建一个新的文件描述符（可能是 4），以从另一个文件读取或重新从第一个文件读取。

创建用于写入的文件描述符（截断模式）：

```
$ exec 4>output.txt # open for writing

```

考虑以下示例：

```
$ exec 4>output.txt $ echo newline >&4 $ cat output.txt newline

```

现在创建一个用于写入的文件描述符（追加模式）：

```
$ exec 5>>input.txt

```

考虑以下示例：

```
$ exec 5>>input.txt $ echo appended line >&5 $ cat input.txt newline appended line

```

# 数组和关联数组

数组允许脚本通过索引将一组数据存储为独立的实体。Bash 支持两种类型的数组：普通数组（使用整数作为数组索引）和关联数组（使用字符串作为数组索引）。当数据按数字顺序组织时，应使用普通数组，例如一系列连续的迭代。关联数组可以用于数据按字符串组织时，例如主机名。在本例中，我们将看到如何使用这两种数组。

# 准备工作

要使用关联数组，必须使用 Bash 版本 4 或更高版本。

# 如何做到...

数组可以使用不同的技术来定义：

1.  使用一行值的列表定义数组：

```
 array_var=(test1 test2 test3 test4) #Values will be stored in consecutive locations starting 
        from index 0.

```

或者，可以将数组定义为一组索引-值对：

```
 array_var[0]="test1" array_var[1]="test2" array_var[2]="test3" array_var[3]="test4" array_var[4]="test5" array_var[5]="test6"

```

1.  使用以下命令打印数组中给定索引的内容：

```
 echo ${array_var[0]} test1 index=5 echo ${array_var[$index]} test6

```

1.  使用以下命令打印数组中的所有值作为列表：

```
 $ echo ${array_var[*]} test1 test2 test3 test4 test5 test6

```

或者，你可以使用以下命令：

```
 $ echo ${array_var[@]} test1 test2 test3 test4 test5 test6

```

1.  打印数组的长度（数组中元素的数量）：

```
 $ echo ${#array_var[*]}6

```

# 还有更多…

关联数组从 Bash 版本 4.0 开始引入。当索引是字符串时（例如站点名称、用户名、非顺序数字等），关联数组比数字索引数组更容易使用。

# 定义关联数组

关联数组可以使用任何文本数据作为数组索引。需要声明语句来定义变量名为关联数组：

```
$ declare -A ass_array

```

在声明后，使用以下两种方法之一向关联数组添加元素：

+   内联索引-值列表方法：

```
 $ ass_array=([index1]=val1 [index2]=val2)

```

+   分离的索引-值赋值：

```
 $ ass_array[index1]=val1 $ ass_array'index2]=val2

```

例如，考虑为水果分配价格，使用关联数组：

```
$ declare -A fruits_value $ fruits_value=([apple]='100 dollars' [orange]='150 dollars')

```

显示数组的内容：

```
$ echo "Apple costs ${fruits_value[apple]}" Apple costs 100 dollars

```

# 列出数组索引

数组通过索引对每个元素进行索引。普通数组和关联数组在索引类型上有所不同。

获取数组中的索引列表。

```
$ echo ${!array_var[*]}

```

另外，我们也可以使用以下命令：

```
$ echo ${!array_var[@]}

```

在之前的`fruits_value`数组示例中，考虑以下命令：

```
$ echo ${!fruits_value[*]} orange apple

```

这同样适用于普通数组。

# 访问别名

**别名**是一个快捷方式，用来替代输入长命令序列。在本教程中，我们将看到如何使用`alias`命令创建别名。

# 如何操作...

以下是你可以对别名执行的操作：

1.  创建别名：

```
 $ alias new_command='command sequence'

```

这个示例为`apt-get install`命令创建了一个快捷方式：

```
 $ alias install='sudo apt-get install'

```

一旦定义了别名，我们可以输入`install`代替`sudo apt-get install`。

1.  `alias`命令是临时的：别名存在直到我们关闭当前终端。要使别名在所有 Shell 中有效，可以将此语句添加到`~/.bashrc`文件中。`~/.bashrc`中的命令总是在启动新的交互式 Shell 进程时执行：

```
 $ echo 'alias cmd="command seq"' >> ~/.bashrc

```

1.  要删除别名，删除`~/.bashrc`文件中的相关条目（如果有），或者使用`unalias`命令。或者，`alias example=`可以取消名为`example`的别名。

1.  这个示例创建了一个`rm`的别名，它会删除原始文件并在备份目录中保留一份副本：

```
 alias rm='cp $@ ~/backup && rm $@'

```

当你创建一个别名时，如果被别名的项已经存在，它将会被新创建的别名命令替代，仅对该用户有效。

# 还有更多...

当以特权用户身份运行时，别名可能会成为安全隐患。为了避免危及系统安全，你应该转义命令。

# 转义别名

考虑到创建别名伪装成原生命令是多么简单，你不应该以特权用户身份运行有别名的命令。我们可以通过转义我们要运行的命令来忽略当前定义的任何别名。考虑以下示例：

```
$ \command

```

`\`字符用于转义命令，执行时不进行任何别名替换。当在不受信任的环境中运行特权命令时，通过在命令前加`\`来忽略别名总是一种良好的安全实践。攻击者可能已经将特权命令与其自定义命令别名，借此窃取用户提供给命令的关键信息。

# 列出别名

`alias`命令列出当前定义的别名：

```
$ aliasalias lc='ls -color=auto' alias ll='ls -l' alias vi='vim'

```

# 获取终端信息

在编写命令行脚本时，我们经常需要操作关于当前终端的信息，比如列数、行数、光标位置、被隐藏的密码字段等。这个教程帮助收集和操作终端设置。

# 准备工作

`tput`和`stty`命令是用于终端操作的工具。

# 如何操作...

以下是`tput`命令的一些功能：

+   返回终端中的列数和行数：

```
 tput cols tput lines

```

+   返回当前终端名称：

```
 tput longname

```

+   将光标移动到 100,100 的位置：

```
 tput cup 100 100

```

+   设置终端背景颜色：

```
 tput setb n

```

`n`的值可以是 0 到 7 之间的一个值

+   设置终端前景颜色：

```
 tput setf n

```

`n`的值可以是 0 到 7 之间的一个值

一些命令，包括常用的`color ls`，可能会重置前景和背景颜色。

+   使用此命令使文本加粗：

```
 tput bold

```

+   执行起始和结束下划线操作：

```
 tput smul tput rmul

```

+   要删除从光标到行尾的内容，可以使用以下命令：

```
 tput ed

```

+   在输入密码时，脚本不应显示字符。以下示例展示了使用`stty`命令禁用字符回显：

```
 #!/bin/sh #Filename: password.sh echo -e "Enter password: " # disable echo before reading password stty -echo read password # re-enable echo stty echo echo echo Password read.

```

上述命令中的`-echo`选项禁用终端输出，而`echo`选项则启用输出。

# 获取和设置日期以及延迟

时间延迟用于在程序执行过程中等待设定的时间（例如 1 秒），或每隔几秒钟（或几个月）监控一个任务。处理时间和日期时需要了解时间和日期的表示方式以及如何操作它们。本篇食谱将展示如何处理日期和时间延迟。

# 准备工作

日期可以以多种格式打印。内部上，日期是以自 1970 年 1 月 1 日 00:00:00 以来的秒数存储的，这被称为**纪元时间**或**Unix 时间**。

系统的日期可以通过命令行设置。接下来的食谱展示了如何读取和设置日期。

# 它是如何做的...

可以以不同的格式读取日期，也可以设置日期。

1.  读取日期：

```
 $ date Thu May 20 23:09:04 IST 2010

```

1.  打印纪元时间：

```
 $ date +%s 1290047248

```

`date`命令可以将许多格式化的日期字符串转换为纪元时间。这使得你可以使用多种日期格式作为输入。通常，如果你从系统日志或任何标准应用程序生成的输出中收集日期，格式无需过多关注。

将日期字符串转换为纪元时间：

```
 $ date --date "Wed mar 15 08:09:16 EDT 2017" +%s 1489579718

```

`--date`选项定义了一个日期字符串作为输入。我们可以使用任何日期格式选项来打印输出。`date`命令可以用来根据日期字符串查找星期几：

```
 $ date --date "Jan 20 2001" +%A Saturday

```

日期格式字符串在*它是如何工作的...*部分中列出。

1.  使用以`+`为前缀的格式字符串组合作为`date`命令的参数，可以将日期按你选择的格式打印。考虑以下示例：

```
 $ date "+%d %B %Y" 20 May 2010

```

1.  设置日期和时间：

```
 # date -s "Formatted date string" # date -s "21 June 2009 11:01:22"

```

在连接到网络的系统上，您可能需要使用`ntpdate`来设置日期和时间：

`/usr/sbin/ntpdate -s time-b.nist.gov`

1.  优化代码的规则是先进行测量。`date`命令可以用来计算一组命令执行的时间：

```
 #!/bin/bash #Filename: time_take.sh start=$(date +%s) commands; statements; end=$(date +%s) difference=$(( end - start)) echo Time taken to execute commands is $difference seconds.

```

`date`命令的最小分辨率为一秒。对命令进行计时的更好方法是`time`命令：

`time commandOrScriptName`。

# 它是如何工作的...

Unix 纪元定义为自 1970 年 1 月 1 日 00:00:00（不包括闰秒）以来的秒数，采用**协调世界时**（**UTC**）。纪元时间在需要计算两个日期或时间之间的差异时非常有用。将两个日期字符串转换为纪元时间并取其差值。此食谱计算两个日期之间的秒数：

```
secs1=`date -d "Jan 2 1970" 
secs2=`date -d "Jan 3 1970" 
echo "There are `expr $secs2 - $secs1` seconds between Jan 2 and Jan 3" 
There are 86400 seconds between Jan 2 and Jan 3 

```

显示自 1970 年 1 月 1 日午夜以来的秒数对人类来说不易阅读。`date`命令支持以人类可读的格式输出。

下表列出了`date`命令支持的格式选项。

| **日期组件** | **格式** |
| --- | --- |
| 星期几 | `%a`（例如，Sat）`%A`（例如，Saturday） |
| 月份 | `%b`（例如，Nov）`%B`（例如，November） |
| 日 | `%d`（例如，31） |
| 日期格式（mm/dd/yy） | `%D`（例如，10/18/10） |
| 年 | `%y`（例如，10）`%Y`（例如，2010） |
| 小时 | `%I` 或 `%H`（例如，08） |
| 分钟 | `%M`（例如，33） |
| 秒 | `%S`（例如，10） |
| 纳秒 | `%N`（例如，695208515） |
| Unix 时间戳（秒） | `%s`（例如，1290049486） |

# 还有更多...

生成时间间隔在编写循环执行的监控脚本时非常重要。以下示例展示了如何生成时间延迟。

# 在脚本中产生延迟

`sleep`命令将根据给定的`秒`数延迟脚本的执行时间。以下脚本使用`tput`和`sleep`从 0 计数到 40 秒：

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

在前面的示例中，一个变量遍历由`seq`命令生成的数字列表。我们使用`tput sc`来存储光标位置。每次循环执行时，我们通过恢复光标位置（使用`tput rc`）并使用`tputs ed`清除到行尾，将新计数值写入终端。清除行后，脚本会输出新值。`sleep`命令会使脚本在每次循环迭代之间延迟 1 秒。

# 调试脚本

调试通常比编写代码所需时间更长。每种编程语言都应实现一个功能，在发生意外时产生追踪信息。通过读取调试信息，可以了解程序为何会出现意外的行为。Bash 提供了每个开发者都应了解的调试选项。本食谱展示了如何使用这些选项。

# 如何做...

我们可以使用 Bash 内建的调试工具，或者编写便于调试的脚本；方法如下：

1.  添加`-x`选项以启用 shell 脚本的调试跟踪。

```
 $ bash -x script.sh

```

使用`-x`标志运行脚本会打印每行源代码及其当前状态。

你也可以使用`sh -x script`。

1.  仅调试脚本的某些部分，使用`set -x`和`set +x`。考虑以下示例：

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

在前面的脚本中，只有`echo $i`的调试信息会被打印，因为调试仅限于该部分，使用了`-x`和`+x`来控制。

脚本使用`{start..end}`构造从开始到结束值进行迭代，而不是前面示例中使用的`seq`命令。这个构造比调用`seq`命令稍快。

1.  上述调试方法由 Bash 内建提供。它们以固定格式输出调试信息。在许多情况下，我们需要以自定义格式输出调试信息。我们可以定义一个 _DEBUG 环境变量来启用和禁用调试，并按照自己的调试风格生成消息。

看一下以下示例代码：

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

使用调试模式“开启”运行前面的脚本：

```
 $ _DEBUG=on ./script.sh

```

我们在每个需要打印调试信息的语句前加上 `DEBUG` 前缀。如果没有将 `_DEBUG=on` 传递给脚本，调试信息将不会被打印。在 Bash 中，命令 `:` 会告诉 shell 什么都不做。

# 它是如何工作的...

`-x` 标志会输出每一行脚本的执行情况。然而，我们可能只需要观察源代码的某些部分。Bash 使用 `set builtin` 来在脚本中启用和禁用调试打印：

+   `set -x`：这会在命令执行时显示参数和命令。

+   `set +x`：这会禁用调试功能。

+   `set -v`：这会在读取时显示输入。

+   `set +v`：这会禁用打印输入。

# 还有更多内容...

我们也可以使用其他方便的方式来调试脚本。我们可以通过更复杂的方式使用 shebang 来调试脚本。

# Shebang 黑客技巧

Shebang 可以从 `#!/bin/bash` 改为 `#!/bin/bash -xv`，以启用调试功能而无需额外的标志（即 `-xv` 标志本身）。

在默认输出中，每行前都有 `+`，这可能使得跟踪执行流程变得困难。可以将 PS4 环境变量设置为 `'$LINENO:'`，以显示实际的行号：

```
PS4='$LINENO: ' 

```

调试输出可能很长。当使用 `-x` 或设置 `-x` 时，调试输出会被发送到 `stderr`。可以使用以下命令将其重定向到文件：

```
sh -x testScript.sh 2> debugout.txt

```

Bash 4.0 及更高版本支持使用编号流来调试输出：

```
exec 6> /tmp/debugout.txt 
BASH_XTRACEFD=6

```

# 函数与参数

函数与别名乍一看相似，但行为略有不同。主要区别在于，函数参数可以在函数体内的任何地方使用，而别名仅将参数附加到命令末尾。

# 如何做...

函数通过 `function` 命令定义，函数名称、圆括号以及用大括号括起来的函数体：

1.  函数的定义如下：

```
        function fname() 
        { 
            statements; 
        }  

```

或者，它也可以按如下方式定义：

```
        fname() 
        { 
            statements; 
        } 

```

它甚至可以按如下方式定义（适用于简单函数）：

```
        fname() { statement; }

```

1.  使用函数名称调用一个函数：

```
 $ fname ; # executes function

```

1.  传递给函数的参数是按位置访问的，`$1` 是第一个参数，`$2` 是第二个，以此类推：

```
 fname arg1 arg2 ; # passing args

```

以下是 `fname` 函数的定义。在 `fname` 函数中，我们包含了多种访问函数参数的方式。

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

传递给脚本的参数可以通过 `$0` 访问（脚本的名称）：

+   +   `$1` 是第一个参数

    +   `$2` 是第二个参数

    +   `$n` 是第 *n* 个参数。

    +   `"$@"` 扩展为 `"$1" "$2" "$3"`，依此类推。

    +   `"$*"` 扩展为 `"$1c$2c$3"`，其中 `c` 是 IFS 的第一个字符。

    +   `"$@"` 比 `$*` 使用得更频繁，因为前者将所有参数作为一个单独的字符串提供。

+   **比较别名与函数**

+   下面是一个别名，通过将 `ls` 输出传递给 `grep` 来显示文件的子集。参数附加在命令的末尾，因此 `lsg txt` 会展开为 `ls | grep txt`：

```
 $> alias lsg='ls | grep' 
 $> lsg txt 
 file1.txt 
 file2.txt 
 file3.txt 

```

+   如果我们想要扩展它来获取 `/sbin/ifconfig` 中设备的 IP 地址，可能会尝试如下操作：

```
 $> alias wontWork='/sbin/ifconfig | grep' 
 $> wontWork eth0 
 eth0  Link  encap:Ethernet  HWaddr 00:11::22::33::44:55 

```

+   `grep`命令找到的是`eth0`字符串，而不是 IP 地址。如果我们使用函数而不是别名，我们可以将参数传递给`ifconfig`，而不是将其附加到`grep`上：

```
 $> function getIP() { /sbin/ifconfig $1 | grep 'inet ';  } 
 $> getIP eth0 
 inet addr:192.168.1.2 Bcast:192.168.255.255 Mask:255.255.0.0

```

# 还有更多...

让我们探索更多 Bash 函数的技巧。

# 递归函数

Bash 中的函数也支持递归（函数可以调用自身）。例如，`F() { echo $1; F hello; sleep 1; }`。

Fork bomb

递归函数是指调用自身的函数：递归函数必须有一个退出条件，否则它们会一直生成进程，直到系统耗尽资源并崩溃。

这个函数：`:(){ :|:& };:` 会永远生成进程，最终导致拒绝服务攻击。

`&`字符被附加到函数调用后面，将子进程带入后台。这个危险的代码会无限生成进程，称为 fork bomb。

你可能会觉得很难理解前面的代码。请参考 Wikipedia 页面 [h t t p ://e n . w i k i p e d i a . o r g /w i k i /F o r k _ b o m b](https://en.wikipedia.org/wiki/Fork_bomb) 了解更多细节和对 fork bomb 的解释。

通过在`/etc/security/limits.conf`中定义`nproc`值来限制能够生成的最大进程数，从而防止此类攻击。

这一行将限制所有用户最多只能启动 100 个进程：

`hard nproc 100`

**导出函数**

函数可以像环境变量一样通过`export`命令导出。导出扩展了函数的作用域到子进程：

```
export -f fname $> function getIP() { /sbin/ifconfig $1 | grep 'inet '; } $> echo "getIP eth0" >test.sh $> sh test.sh
 sh: getIP: No such file or directory $> export -f getIP $> sh test.sh
 inet addr: 192.168.1.2 Bcast: 192.168.255.255 Mask:255.255.0.0

```

# 读取命令的返回值（状态）

一个命令的返回值存储在`$?`变量中。

```
cmd; echo $?;

```

返回值被称为**退出状态**。这个值可以用来判断一个命令是否成功执行。如果命令成功退出，退出状态将为零，否则为非零值。

以下脚本报告命令的成功/失败状态：

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

# 向命令传递参数

大多数应用程序接受不同格式的参数。假设`-p`和`-v`是可用的选项，`-k N`是另一个需要数字的选项。此外，命令需要一个文件名作为参数。这个应用程序可以通过多种方式执行：

+   `$ command -p -v -k 1 file`

+   `$ command -pv -k 1 file`

+   `$ command -vpk 1 file`

+   `$ command file -pvk 1`

在脚本中，命令行参数可以通过它们在命令行中的位置进行访问。第一个参数是`$1`，第二个是`$2`，依此类推。

这个脚本会显示前三个命令行参数：

```
echo $1 $2 $3

```

通常情况下，逐个迭代命令参数。`shift`命令将每个参数向左移动一个位置，使脚本可以将每个参数作为`$1`来访问。以下代码显示所有命令行的值：

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

# 将一个命令的输出发送给另一个命令

Unix shell 的一个最好的特点是可以轻松地组合多个命令来生成报告。一个命令的输出可以作为另一个命令的输入，而另一个命令则将其输出传递给下一个命令，依此类推。这个命令序列的输出可以分配给一个变量。这个示例展示了如何组合多个命令以及如何读取输出。

# 准备好

输入通常通过`stdin`或参数传递给命令。输出发送到`stdout`或`stderr`。当我们组合多个命令时，通常通过`stdin`提供输入，并生成输出到`stdout`。

在这种情况下，命令被称为**过滤器**。我们使用管道（由管道操作符`|`表示）将每个过滤器连接起来，如下所示：

```
$ cmd1 | cmd2 | cmd3 

```

在这里，我们组合了三个命令。`cmd1`的输出传递给`cmd2`，`cmd2`的输出传递给`cmd3`，最终输出（来自`cmd3`）将显示在屏幕上，或者被重定向到一个文件。

# 如何做到这一点...

管道可以与子壳方法一起使用，用于组合多个命令的输出。

1.  让我们从组合两个命令开始：

```
 $ ls | cat -n > out.txt

```

`ls`（当前目录的列表）的输出被传递给`cat -n`，后者在通过`stdin`接收到的输入前加上行号。输出被重定向到`out.txt`。

1.  将一系列命令的输出分配给一个变量：

```
 cmd_output=$(COMMANDS)

```

这被称为**子壳方法**。考虑以下这个例子：

```
 cmd_output=$(ls | cat -n) echo $cmd_output

```

另一种方法，称为**反引号**（有些人也称之为**反勾引号**），也可以用来存储命令输出：

```
 cmd_output=`COMMANDS`

```

考虑以下这个例子：

```
 cmd_output=`ls | cat -n`
 echo $cmd_output

```

反引号与单引号字符不同。它是键盘上*~*键上的字符。

# 还有更多...

有多种方法可以对命令进行分组。

# 使用子壳生成一个独立的进程

子壳是独立的进程。子壳使用`( )`运算符来定义：

+   `pwd`命令打印工作目录的路径

+   `cd`命令将当前目录更改为给定的目录路径：

```
        $> pwd 
        / 
        $> (cd /bin; ls) 
        awk bash cat... 
        $> pwd 
        /

```

当命令在子壳中执行时，当前 shell 不会发生任何变化；更改仅限于子壳。例如，当使用`cd`命令在子壳中更改当前目录时，目录更改不会反映在主 shell 环境中。

# 使用子壳引用来保留空格和换行符

假设我们使用子壳或反引号方法将命令的输出分配给一个变量，我们必须使用双引号来保留空格和换行符（`\n`）。考虑这个例子：

```
$ cat text.txt 1 2 3 $ out=$(cat text.txt) $ echo $out 1 2 3 # Lost \n spacing in 1,2,3 $ out="$(cat text.txt)" $ echo $out 1 2 3

```

# 读取 n 个字符而无需按回车键

bash 命令 `read` 从键盘或标准输入中读取文本。我们可以使用 `read` 来交互式地获取用户输入，但 `read` 还能做更多事情。任何编程语言中的大多数输入库都会从键盘读取输入，并在按下回车时终止字符串。在某些情况下，无法按下回车键，而字符串的终止是基于接收到的字符数（可能是单个字符）。例如，在一个互动游戏中，当按下 *+* 时，球会向上移动。按下 *+* 然后再按 *return* 来确认 *+* 的按下是不高效的。

这个方案使用 `read` 命令来完成此任务，无需按 *return* 键。

# 如何实现...

你可以使用 `read` 命令的各种选项来获取不同的结果，如下所示：

1.  以下语句将从输入中读取 *n* 个字符到 `variable_name` 变量中：

```
 read -n number_of_chars variable_name

```

请考虑这个示例：

```
 $ read -n 2 var $ echo $var

```

1.  以不回显的模式读取密码：

```
 read -s var

```

1.  使用以下命令通过 `read` 显示一条消息：

```
 read -p "Enter input:"  var

```

1.  在超时后读取输入：

```
 read -t timeout var

```

请考虑以下示例：

```
 $ read -t 2 var # Read the string that is typed within 2 seconds into
        variable var.

```

1.  使用分隔符字符来结束输入行：

```
 read -d delim_char var

```

请考虑以下示例：

```
 $ read -d ":" var hello:#var is set to hello

```

# 执行命令直到它成功

有时，命令只能在满足特定条件时成功。例如，只有在文件创建后才能下载该文件。在这种情况下，你可能希望重复执行命令直到它成功。

# 如何实现...

以以下方式定义一个函数：

```
repeat() 
{ 
  while true 
  do 
    $@ && return 
  done 
}

```

或者，可以将此添加到 shell 的 `rc` 文件中以便于使用：

```
repeat() { while true; do $@ && return; done }

```

# 它是如何工作的...

这个重复函数有一个无限的 `while` 循环，尝试执行作为参数传递给函数的命令（通过 `$@` 访问）。如果命令执行成功，则返回并退出循环。

# 还有更多...

我们看到了一种基本的执行命令直到成功的方法。让我们使其更加高效。

# 更快速的方法

在大多数现代系统中，`true` 被实现为 `/bin` 中的一个二进制文件。这意味着每次上述的 `while` 循环运行时，shell 都需要启动一个进程。为了避免这种情况，我们可以使用 shell 内建的 `:` 命令，它始终返回退出码 0：

```
repeat() { while :; do $@ && return; done }

```

尽管这种方法不如第一种方法可读，但它比第一种方法更快。

# 添加延迟

假设你正在使用 `repeat()` 从互联网上下载一个当前不可用，但稍后会可用的文件。示例如下：

```
repeat wget -c http://www.example.com/software-0.1.tar.gz

```

这个脚本将向 `www.example.com` 发送过多的流量，导致服务器出现问题（如果服务器将你的 IP 列入黑名单，可能也会对你造成麻烦）。为了解决这个问题，我们修改了函数并添加了延迟，具体如下：

```
repeat() { while :; do $@ && return; sleep 30; done }

```

这将使命令每 30 秒运行一次。

# 字段分隔符和迭代器

**内部字段分隔符**（**IFS**）是 shell 脚本中的一个重要概念。它对文本数据的处理非常有用。

IFS 是一个特殊目的的定界符。它是一个存储分隔字符的环境变量，是运行的 shell 环境使用的默认定界符字符串。

考虑需要遍历字符串中的单词或**以逗号分隔的值**（**CSV**）的情况。在第一种情况下，我们将使用`IFS=" "`，在第二种情况下，使用`IFS=","`。

# 准备工作

考虑 CSV 数据的情况：

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

这会生成以下输出：

```
Item: name Item: gender Item: rollno Item: location

```

IFS 的默认值是空白字符（换行符、制表符或空格字符）。

当 IFS 设置为`,`时，shell 将逗号解释为分隔符字符，因此在迭代过程中，`$item`变量将获取由逗号分隔的子字符串作为其值。

如果 IFS 没有设置为`,`，则会将整个数据作为一个字符串打印出来。

# 如何操作...

让我们通过另一个使用 IFS 的例子来解析`/etc/passwd`文件。在`/etc/passwd`文件中，每行包含用`:`分隔的项。文件中的每一行对应于与用户相关的一个属性。

考虑输入：`root:x:0:0:root:/root:/bin/bash`。每行的最后一项指定了该用户的默认 shell。

使用 IFS 技巧打印用户及其默认 shell：

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

输出将如下所示：

```
root's shell is /bin/bash

```

循环在遍历一系列值时非常有用。Bash 提供了多种类型的循环。

+   **面向列表的`for`循环**：

```
        for var in list; 
        do 
            commands; # use $var 
        done 

```

列表可以是一个字符串或一系列值。

我们可以使用`echo`命令生成序列：

```
echo {1..50} ;# Generate a list of numbers from 1 to 50.
echo {a..z} {A..Z} ;# List of lower and upper case letters. 

```

我们可以将这些结合起来以连接数据。

在以下代码中，每次迭代中，变量 i 将包含 a 到 z 范围内的字符：

```
      for i in {a..z}; do actions; done;

```

+   **遍历一系列数字**：

```
        for((i=0;i<10;i++)) 
        { 
           commands; # Use $i 
        }

```

+   **循环直到满足条件**：

while 循环在条件为真时继续运行，而 until 循环在条件为真之前执行：

```
        while condition 
        do 
            commands; 
        done

```

对于无限循环，使用`true`作为条件：

+   **使用`until`循环**：

Bash 中有一个特殊的循环叫做`until`。这个循环会一直执行直到给定条件变为真。请考虑这个例子：

```
        x=0; 
        until [ $x -eq 9 ]; # [ $x -eq 9 ] is the condition 
        do 
            let x++; echo $x; 
        done

```

# 比较和测试

程序中的流程控制通过比较和测试语句来处理。Bash 提供了几种执行测试的选项。我们可以使用`if`、`if else`和逻辑运算符来执行测试，并使用比较运算符来比较数据项。还有一个叫做`test`的命令，它也用于执行测试。

# 如何操作...

这里是一些用于比较和执行测试的方法：

+   使用`if`条件：

```
        if condition; 
        then 
            commands; 
        fi

```

+   使用`else if`和`else`：

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

if 和 else 可以嵌套。if 条件可能很长；为了简化它们，我们可以使用逻辑运算符：

`[ condition ] && action;` # 如果条件为真，执行动作

`[ condition ] || action;` # 如果条件为假，执行动作

`&&`是逻辑与操作，`||`是逻辑或操作。这在编写 Bash 脚本时非常有用。

执行数学比较：通常，条件包含在方括号 `[]` 中。注意，`[` 或 `]` 和操作数之间有空格。如果没有空格，将会出现错误。

```
[$var -eq 0 ] or [ $var -eq 0]

```

对变量和值进行数学测试，如下所示：

```
[ $var -eq 0 ]  # It returns true when $var equal to 0\. 
[ $var -ne 0 ] # It returns true when $var is not equal to 0

```

其他重要的运算符包括以下内容：

+   `-gt`：大于

+   `-lt`：小于

+   `-ge`：大于或等于

+   `-le`：小于或等于

`-a` 运算符是逻辑与，`-o` 运算符是逻辑或。多个测试条件可以组合使用：

```
[ $var1 -ne 0 -a $var2 -gt 2 ]  # using and -a 
[ $var1 -ne 0 -o var2 -gt 2 ] # OR -o

```

与文件系统相关的测试如下：

使用不同的条件标志测试不同的文件系统属性。

+   `[ -f $file_var ]`：如果给定的变量保存的是一个常规文件路径或文件名，返回 true。

+   `[ -x $var ]`：如果给定的变量保存的是一个可执行的文件路径或文件名，返回 true。

+   `[ -d $var ]`：如果给定的变量保存的是目录路径或目录名称，返回 true。

+   `[ -e $var ]`：如果给定的变量保存的是一个存在的文件，返回 true。

+   `[ -c $var ]`：如果给定的变量保存的是一个字符设备文件的路径，返回 true。

+   `[ -b $var ]`：如果给定的变量保存的是一个块设备文件路径，返回 true。

+   `[ -w $var ]`：如果给定的变量保存的是一个可写的文件路径，返回 true。

+   `[ -r $var ]`：如果给定的变量保存的是一个可读的文件路径，返回 true。

+   `[ -L $var ]`：如果给定的变量保存的是路径，返回 true。

    符号链接

考虑以下示例：

```
fpath="/etc/passwd" 
if [ -e $fpath ]; then 
    echo File exists;  
else 
    echo Does not exist;  
fi

```

字符串比较：在使用字符串比较时，最好使用双中括号，因为使用单中括号有时可能会导致错误。

注意，双中括号是 Bash 扩展。如果脚本将在 ash 或 dash 中运行（为了提高性能），则不能使用双中括号。

**测试两个字符串是否相同**：

+   `[[ $str1 = $str2 ]]`：当 `str1` 等于 `str2` 时返回 true，即 `str1` 和 `str2` 的文本内容相同。

+   `[[ $str1 == $str2 ]]`：这是另一种字符串比较的方法。

    相等性检查

**测试两个字符串是否不相同**：

+   `[[ $str1 != $str2 ]]`：当 `str1` 和 `str2` 不匹配时返回 true。

查找字母顺序上较大的字符串：

字符串通过比较字符的 ASCII 值按字母顺序进行比较。例如，"A" 是 0x41，而 "a" 是 0x61。因此，"A" 小于 "a"，而 "AAa" 小于 "Aaa"。

+   `[[ $str1 > $str2 ]]`：当 `str1` 在字母顺序上大于 `str2` 时返回 true。

+   `[[ $str1 < $str2 ]]`：当 `str1` 在字母顺序上小于 `str2` 时返回 true。

在 `=` 前后需要空格；如果没有提供空格，它就不是比较，而是赋值语句。

**测试空字符串**：

+   `[[ -z $str1 ]]`：如果 `str1` 保存的是一个空字符串，返回 true。

+   `[[ -n $str1 ]]`：如果 `str1` 保存的是一个非空字符串，返回 true。

使用逻辑运算符 `&&` 和 `||` 来组合多个条件更为简便，如以下代码所示：

```
if [[ -n $str1 ]] && [[ -z $str2 ]] ;
   then
       commands;
   fi

```

考虑这个例子：

```
str1="Not empty " 
str2="" 
if [[ -n $str1 ]] && [[ -z $str2 ]]; 
then 
    echo str1 is nonempty and str2 is empty string. 
fi

```

这将是输出：

```
str1 is nonempty and str2 is empty string.

```

`test` 命令可以用来进行条件检查。这减少了使用大括号的数量，并且可以使代码更具可读性。`[]` 中的相同测试条件也可以与 `test` 命令一起使用。

注意，`test` 是一个外部程序，必须被 fork，而 `[` 是 Bash 中的内部函数，因此效率更高。`test` 程序与 Bourne shell、ash、dash 等兼容。

考虑这个例子：

```
if  [ $var -eq 0 ]; then echo "True"; fi 
can be written as 
if  test $var -eq 0 ; then echo "True"; fi

```

# 使用配置文件定制 bash

你在命令行中输入的大多数命令都可以放入一个特殊文件中，在登录或启动新的 bash 会话时进行评估。通过将函数定义、别名和环境变量设置放入这些文件之一来定制你的 shell 是很常见的做法。

常见的命令可以放入配置文件中，包括以下内容：

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

**我应该使用哪个定制文件？**

Linux 和 Unix 有几个文件可能包含自定义脚本。这些配置文件分为三类——登录时加载的文件、交互式 shell 调用时评估的文件，以及每次 shell 被调用以处理脚本文件时评估的文件。

# 如何操作...

这些文件会在用户登录到 shell 时被评估：

```
/etc/profile, $HOME/.profile, $HOME/.bash_login, $HOME/.bash_profile /

```

注意，如果通过图形登录管理器登录，则可能不会加载 `/etc/profile`、`$HOME/.profile` 和 `$HOME/.bash_profile`。这是因为图形窗口管理器不会启动 shell。当你打开一个终端窗口时，会创建一个 shell，但这不是登录 shell。

如果存在 `.bash_profile` 或 `.bash_login` 文件，则 `.profile` 文件不会被读取。

这些文件将被交互式 shell 读取，如 X11 终端会话，或使用 `ssh` 执行类似命令 `ssh 192.168.1.1 ls /tmp`。

```
/etc/bash.bashrc $HOME/.bashrc

```

如下所示运行 shell 脚本：

```
$> cat myscript.sh 
#!/bin/bash 
echo "Running"

```

除非你已定义了 `BASH_ENV` 环境变量，否则这些文件都不会被加载：

```
$> export BASH_ENV=~/.bashrc 
$> ./myscript.sh

```

使用 `ssh` 执行单个命令，如下所示：

```
ssh 192.168.1.100 ls /tmp

```

这将启动一个 bash shell，评估 `/etc/bash.bashrc` 和 `$HOME/.bashrc`，但不会评估 `/etc/profile` 或 `.profile`。

调用 ssh 登录会话，如下所示：

```
ssh 192.168.1.100

```

这将创建一个新的登录 bash shell，系统将评估以下内容：

```
/etc/profile 
/etc/bash.bashrc 
$HOME/.profile or .bashrc_profile

```

危险：其他外壳，如传统的 Bourne shell、ash、dash 和 ksh，也会读取此文件。线性数组（列表）和关联数组，并非所有外壳都支持。在 `/etc/profile` 或 `$HOME/.profile` 中避免使用它们。

使用这些文件来定义所有用户都希望拥有的非导出项，如别名。考虑这个例子：

```
alias l "ls -l"
/etc/bash.bashrc /etc/bashrc

```

使用这些文件来存储个人设置。它们对于设置必须由其他 bash 实例继承的路径很有用。它们可能包含如下行：

```
CLASSPATH=$CLASSPATH:$HOME/MyJavaProject; export CLASSPATH
$HOME/.bash_login $HOME/.bash_profile $HOME/.profile

```

如果 `.bash_login` 或 `.bash_profile` 存在，`.profile` 文件将不会被读取。`.profile` 文件可能会被其他 shell 读取。

使用这些文件来保存每次创建新 Shell 时需要定义的个人值。如果你希望它们在 X11 终端会话中可用，可以在这里定义别名和函数：

```
$HOME/.bashrc, /etc/bash.bashrc

```

导出的变量和函数会传播到下级 Shell，但别名不会。你必须将 `BASH_ENV` 定义为 `.bashrc` 或 `.profile`，在这些文件中定义别名，才能在 Shell 脚本中使用它们。

当用户登出会话时，此文件会被执行：

```
$HOME/.bash_logout

```

例如，如果用户远程登录，他们在登出时应清除屏幕。

```
$> cat ~/.bash_logout 
# Clear the screen after a remote login/logout. 
clear

```
