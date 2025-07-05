# 第五章：监控方法

本章中，我们将涵盖以下内容：

+   监控任何主机的 PING

+   监控任何主机的 SSH

+   检查一个备用 SSH 端口

+   监控邮件服务

+   监控网页服务

+   检查网站是否返回特定字符串

+   监控数据库服务

+   监控 SNMP 查询的输出

+   监控 RAID 或其他硬件设备

+   创建一个 SNMP OID 进行监控

# 介绍

Nagios Core 最好被视为一个监控框架，利用插件对主机和服务进行适当的检查，并以其理解的格式返回状态结果，用于发送通知并长期跟踪状态。

设计相当灵活。如第二章《使用命令和插件》中所述，*命令和插件使用*，Nagios Core 可以将任何命令行应用程序用作插件，只要它返回的值符合 Nagios Core 头文件、Perl 库或 Shell 脚本中定义的适当返回值。反过来，Nagios Core 可以通过配置以多种方式使用相同的插件，利用插件提供的任何开关调整其行为，包括通过 Nagios Core 宏的值（如`$HOSTADDRESS$`）向其提供元数据。

Nagios Exchange 网站上可用的插件集合相当庞大，详细记录所有插件超出了本书的范围。然而，一些最有用的插件作为 Nagios 插件集的一部分被包含在内，并作为 Nagios Core 推荐快速入门指南的一部分进行安装，地址为[`nagios.sourceforge.net/docs/3_0/quickstart.html`](http://nagios.sourceforge.net/docs/3_0/quickstart.html)。这些插件包括用于以常见方式监控典型网络特征的程序，例如监控基础网络连接、网页服务、邮件服务器等。插件网站本身的网址是[`nagiosplugins.org/`](http://nagiosplugins.org/)。

本章将演示此插件集一些最有用组件的使用方法，假设你已经安装了这些插件。重点将放在监控任务上，这些任务与大多数或甚至所有不同规模的网络相关，旨在帮助读者不再仅仅将 Nagios Core 视为一个发送 PING 请求的过程。最后几节将展示如何使用**简单网络管理协议**（**SNMP**）作为检查任何通用网络服务或系统属性的方法，这些可能并不在标准的 Nagios 插件集中涵盖。

# 监控任何主机的 PING

在本教程中，我们将学习如何为主机设置 PING 监控。我们将使用`check_ping`插件及其同名命令，向主机发送`ICMP` `ECHO`请求。我们将这作为一个简单的诊断检查，确保主机的网络栈以一致和及时的方式响应，就像管理员可能会使用`ping`命令来检查相同的属性一样。

## 准备工作

你应该已经有一个 Nagios Core 3.0 或更高版本的服务器，并且至少已经配置了一个主机。我们将使用`corinth.naginet`的例子，它是一个在独立文件中定义的主机。你还应该理解主机和服务之间的基本关系，这在第一章，*理解主机、服务和联系人*一节中有详细介绍。

## 如何操作...

我们可以按照如下方式，为现有主机添加一个新的 PING 服务检查：

1.  更改为 Nagios Core 的对象配置目录。默认路径是`/usr/local/nagios/etc/objects`。如果你将主机的定义放在了不同的文件中，请转到该目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        windows-server
        host_name  corinth.naginet
        alias      corinth
        address    10.128.0.27
    }
    ```

1.  在主机定义下方，放置一个引用`check_ping`的服务定义。你可以使用`generic-service`模板，如下所示：

    ```
    define service {
        use                  generic-service
        host_name            corinth.naginet
        service_description  PING
        check_command        check_ping!100,20%!200,40%
    }
    ```

1.  验证配置并重启 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成后，新的服务检查将开始运行，并且当发生以下情况时，相关联系人和联系人组将收到通知：

+   如果请求及其响应的**往返时间**（**RTT**）超过了 200ms，或者在检查过程中丢失了超过 40%的数据包；无论哪种情况，都会为该服务触发一个`CRITICAL`通知。

+   如果没有触发`CRITICAL`通知，并且请求及其响应的 RTT 超过了 100ms，或者在检查过程中丢失了超过 20%的数据包；此时，会为该服务触发一个`WARNING`通知。

有关阈值的信息将在`check_ping`命令的`check_command`指令定义中作为参数给出。

关于该服务的更多信息也会在 Web 界面的**服务**部分显示。

## 它是如何工作的...

前一节中添加的配置定义了一个新的服务检查，用于检查现有的`corinth.naginet`主机，确保`ICMP` `ECHO`请求及其响应的 RTT 和丢包率在可接受的范围内。

对于大多数网络配置，主机本身很可能也会通过命令`check-host-alive`被`check_ping`检查。不同之处在于，这个命令的 RTT 和丢包的阈值通常设置得非常高，因为它的目的是判断主机是否在线，而不是主机的响应速度。

## 还有更多...

在大多数主机都配置为响应`ICMP` `ECHO`请求的网络中，可能值得对配置中的所有主机配置服务检查。可以使用`*`通配符在定义`host_name`时实现这一点：

```
define service {
    use                  generic-service
    host_name            *
    service_description  PING
    check_command        check_ping!100,20%!200,40%
}
```

这将使用`PING`的`service_description`指令，应用相同的检查到数据库中配置的所有主机。这种方法可以避免为所有主机单独配置服务的麻烦。

如果网络中的某些主机未响应 PING，则可能更合适将响应的主机放入一个主机组中，主机组的名称可以是`icmp`之类的：

```
define hostgroup {
    hostgroup_name  icmp
    alias           ICMP enabled hosts
    members         sparta.naginet,corinth.naginet
}
```

然后，单个服务可以应用于该组中的所有主机，使用`service`定义中的`hostgroup_name`指令：

```
define service {
    use                  generic-service
    hostgroup_name       icmp
    service_description  PING
    check_command        check_ping!100,20%!200,40%
}
```

通常来说，尽可能让网络主机响应 ICMP 消息是一个好主意，这样可以遵循 RFC1122 中的建议并简化调试过程。

最后，注意 RTT 和丢包的阈值不是固定的；事实上，它们是在服务定义中的`check_command`行中定义的。对于那些由于网络负载或拓扑导致延迟较高的主机，可能需要调整这些阈值，具体内容可以参考第三章中的*更改 ping RTT 和丢包阈值*的食谱，*工作与检查和状态*。

## 另见

+   在第一章中，*创建新主机*、*创建新服务*和*创建新主机组*的示例，*理解主机、服务和联系人*。

+   在第二章中，*使用替代检查命令*的食谱，*工作与命令和插件*。

+   在第三章中，*更改 ping RTT 和丢包阈值*的食谱，*工作与检查和状态*。

# 监控任意主机上的 SSH

在本食谱中，我们将学习如何使用`check_ssh`插件及其同名命令检查远程主机上的 SSH 守护进程是否响应请求。这将使我们在连接到 SSH 服务时遇到问题时，能够及时收到通知。

## 准备工作

您应该有一个 Nagios Core 3.0 或更新版本的服务器，并且已经配置了至少一个主机。我们将使用` troy.naginet`这个例子，主机定义在其自己的文件中。您还应该了解主机和服务之间的基本关系，具体内容可以参考第一章中的食谱，*理解主机、服务和联系人*。

首先验证您要添加监控的主机当前是否正在运行需要检查的 SSH 服务可能是个好主意。可以通过运行`ssh`客户端连接到主机来进行验证：

```
$ ssh troy.naginet

```

我们还应检查插件本身在作为`nagios`用户运行时是否返回所需的结果。

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_ssh troy.naginet

```

如果你无法从目标机器上的 SSH 服务获得正面响应，即使你确定它正在运行，这可能是与连接性或过滤问题无关的症状。例如，我们可能需要将运行 Nagios Core 的监控服务器添加到任何适用防火墙或路由器的 SSH（通常是 TCP 目标端口`22`）白名单中。

## 如何操作...

我们可以按照以下方式将新的 SSH 服务检查添加到现有主机：

1.  切换到 Nagios Core 的`objects`配置目录。默认路径是`/usr/local/nagios/etc/objects`。如果你将主机定义放在了其他文件中，则应转到该文件所在目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  troy.naginet
        alias      troy
        address    10.128.0.25
    }
    ```

1.  在主机定义下方，放置一个服务定义，引用`check_ssh`。使用`generic-service`模板或其他合适的模板可能会有所帮助，如下所示：

    ```
    define service {
        use                  generic-service
        host_name            troy.naginet
        service_description  SSH
        check_command        check_ssh
    }
    ```

1.  验证配置并重新启动 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成此步骤后，将开始进行新的服务检查，当尝试连接到 SSH 服务器失败时，相关联系人和联系人组将收到通知。服务检查将在 Web 界面上的**服务**页面中显示。

## 它是如何工作的...

上述配置定义了一个新的服务，其`service_description`为`SSH`，适用于现有的`troy.naginet`主机，使用`generic-service`模板中的值，并另外定义了`check_command`指令`check_ssh`。

这意味着除了之前检查主机是否启动的`check-host-alive`，Nagios Core 还将检查运行在主机上的 SSH 服务是否正常工作，方法是尝试与其建立连接。如果在进行适当次数的测试后发现服务有问题，它还会通知相关联系人。

例如，如果插件发现主机可以访问但未响应客户端测试，则可能会通知如下文本：

```
Subject: ** PROBLEM Service Alert: troy.naginet/SSH is CRITICAL **

***** Nagios *****

Notification Type: PROBLEM

Service: SSH
Host: troy.naginet
Address: troy.naginet
State: CRITICAL

Date/Time: Wed May 23 13:35:21 NZST 2012

Additional Info:

CRITICAL - Socket timeout after 10 seconds

```

请注意，我们不需要为 SSH 检查提供凭证；插件仅确保服务正在运行并响应连接尝试。

## 还有更多内容...

如果我们对插件如何实际作为命令应用感到好奇，应该检查`check_ssh`命令的定义，这在`/usr/local/nagios/etc/objects/commands.cfg`的 QuickStart 配置中有定义：

```
define command {
    command_name  check_ssh
    command_line  $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
}
```

这表明 `check_ssh` 命令已配置为在 `$USER1$` 中运行 `check_ssh` 二进制文件，该宏通常扩展为 `/usr/local/nagios/libexec`，并针对适用服务器的主机地址执行。它在之前添加任何其他参数。在本教程中我们没有使用任何参数，因为我们只是想对其默认端口上的 SSH 服务进行常规检查。

此检查应适用于大多数符合 SSH2 标准的服务器，尤其是流行的 **OpenSSH** 服务器。

检查 SSH 可访问性是服务器上常见的任务，您可能希望设置一个 SSH 服务检查，应用于一个主机组，而不仅仅是单个主机。例如，如果您有一个名为 `ssh-servers` 的组，其中包含多个服务器，这些服务器应使用 `check_ssh` 进行检查，那么您可以通过 `hostgroup_name` 指令配置它们，以便通过一个服务定义对它们进行检查：

```
define service {
    use                  generic-service
    hostgroup_name       ssh-servers
    service_description  SSH
    check_command        check_ssh
}
```

这会将相同的服务检查应用于组中的每个主机，这样，如果以后需要更改或删除检查，定义将更容易更新。

请注意，`check_ssh` 插件与 `check_by_ssh` 插件不同，后者用于在远程机器上执行检查，类似于 NRPE。

## 另请参阅

+   本章中的 *检查替代 SSH 端口* 教程

+   在第一章中，*创建新主机*、*创建新服务* 和 *创建新主机组* 的教程，*理解主机、服务和联系人*

+   在第六章中，*使用 key 认证的 check_by_ssh 替代 NRPE* 和 *在远程机器上使用 NRPE 监控本地服务* 的教程，*启用远程执行*

# 检查替代 SSH 端口

在本教程中，我们将学习如何处理机器运行 SSH 守护进程且监听替代端口的常见情况。因此，使用 `check_ssh` 的服务定义，如 *监控任何主机的 SSH* 教程中所示，由于插件默认使用标准 SSH TCP 端口号 22，因此会失败。

这种设置在 SSH 服务器不应向公众开放的情况下非常常见，通常作为一种“通过模糊性提高安全性”的方法，用于减少对服务器的自动化攻击。因此，SSH 守护进程配置为监听其他端口，通常是更高的端口号；需要使用它的管理员会被告知端口号。

我们将处理这种情况，并在 Nagios Core 中监控该服务，即使它运行在非标准端口上。我们将通过定义一个新命令来检查指定端口上的 SSH，并创建一个使用该命令的服务定义。该命令将接受要检查的端口号作为参数。

这里的原则应当很好地适用于任何需要检查备用端口的情况，前提是用于执行检查的 Nagios Core 插件支持在替代端口上执行此操作。

## 准备工作

你应该已经配置好一个 Nagios Core 3.0 或更新版本的服务器，并且至少配置了一个主机。我们将以 `troy.naginet` 为例，它是一个在自己文件中定义的主机，监听非标准的 SSH 端口 `5022`。你还应该了解主机和服务的基本关系，这部分内容在第一章中有介绍，*理解主机、服务和联系人*。

一个好的第一步是验证我们是否能够从监控服务器访问指定端口上的 SSH 守护进程。我们可以使用 `ssh` 客户端从命令行进行验证，并通过 `-p` 选项指定端口号：

```
$ ssh troy.naginet -p 5022

```

或者，你也可以直接从命令行运行 `check_ssh` 插件：

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_ssh -p 5022 troy.naginet

```

## 如何操作……

我们可以按照以下方式设置一个针对非标准端口的 SSH 服务检查：

1.  切换到 Nagios Core 的 `objects` 配置目录。默认路径是 `/usr/local/nagios/etc/objects`。如果你将主机定义放在其他文件中，则需要转到该文件所在目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑一个包含命令定义的合适文件，找到 `check_ssh` 命令的定义。在默认安装中，该文件是 `commands.cfg`。`check_ssh` 的定义类似于以下代码片段：

    ```
    define command {
        command_name  check_ssh
        command_line  $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
    }
    ```

1.  在 `check_ssh` 定义下方，添加一个新的命令定义，如下所示：

    ```
    define command {
        command_name  check_ssh_altport
     command_line  $USER1$/check_ssh -p $ARG1$ $HOSTADDRESS$
    }
    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  troy.naginet
        alias      troy
        address    10.128.0.25
    }
    ```

1.  在主机定义下方，使用我们的新命令添加一个新的服务定义：

    ```
    define service {
        use                  generic-service
        host_name            troy.naginet
        service_description  SSH_5022
        check_command        check_ssh_altport!5022
    }
    ```

1.  验证配置并重启 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成这些步骤后，Nagios Core 将开始使用 `check_ssh` 插件进行服务检查，但会在连接尝试时使用备用的目标端口 `5022`，其 `service_description` 为 `SSH_5022`。

## 工作原理……

前面部分添加的配置几乎和添加一个默认的 `check_ssh` 服务完全相同，唯一的区别是检查的端口不同，以便建立连接。我们使用 `check_ssh_altport` 命令来实现这一点，其语法与 `check_ssh` 定义非常相似。

区别在于，该命令接受一个参数，该参数作为 `-p` 选项的值传递给 `check_ssh` 插件，用于检查指定的端口号；在此情况下是 TCP 端口 `5022`，而不是默认的端口 `22`。

## 还有更多……

由于 Nagios Core 中的参数可以包含空格，我们也可以将服务检查定义如下，而无需额外定义命令：

```
define service {
    use                  generic-service
    host_name            troy.naginet
    service_description  SSH_5022
    check_command        check_ssh!-p 5022
}
```

这是因为 `$ARG1$` 宏表示的参数仍然在原始的 `check_ssh` 命令中使用，但它需要包括选项及其值。两者的主要区别在于偏好问题，取决于我们认为哪种方式更清晰和更易于维护。可以考虑一下是否使用一个命名明确的命令有助于其他人理解我们配置的含义。

## 另请参见

+   本章中的 *监控任何主机的 SSH* 配方

+   第一章中的 *创建新主机* 和 *创建新服务* 配方，*理解主机、服务和联系人*

+   第二章中的 *创建新命令* 配方，*与命令和插件协作*

# 监控邮件服务

在本配方中，我们将学习如何监控指定主机的三项常见邮件服务：**SMTP**、**POP** 和 **IMAP**。我们还将了解如何使用相同的结构，为这些服务的加密版本添加额外的检查：**SMTPS**、**POPS** 和 **IMAPS**。

为了简化起见，我们假设在本示例中这三项服务都运行在同一主机上，但这个过程很容易推广到常见的情况，即为一个或多个先前提到的功能指定专用服务器。

## 准备工作

你应该已经配置了至少一台主机的 Nagios Core 3.0 或更新版本的服务器。我们将使用 `troy.naginet` 作为示例，这是在独立文件中定义的主机。你还应该了解主机和服务之间的基本关系，这在第一章，*理解主机、服务和联系人*中有涉及。

检查目标服务器上所需服务的连接性也是个好主意，以确保监控服务器将在适当的协议和端口上建立的自动连接能够按预期工作。对于未加密的邮件服务，可以通过 **Telnet** 连接到相应的端口来完成检查。对于 SMTP：

```
$ telnet marathon.naginet 25
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
220 marathon.naginet ESMTP Postfix

```

对于 POP：

```
$ telnet marathon.naginet 110
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
+OK Hello there.

```

对于 IMAP：

```
$ telnet marathon.naginet 143
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
* OK CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE...

```

对于安全服务，检查的一种可能方式是使用 `openssl` 客户端。对于 SMTPS，其“经典”端口号为 465：

```
$ openssl s_client -host marathon.naginet -port 465
CONNECTED(00000003)
...
220 marathon.naginet ESMTP Postfix

```

对于 POPS：

```
$ openssl s_client -host marathon.naginet -port 995
CONNECTED(00000003)
...
+OK Hello there.

```

对于 IMAPS：

```
$ openssl s_client -host marathon.naginet -port 993
CONNECTED(00000003)
...
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE...

```

如果你愿意，也可以使用网络扫描工具如 `nmap` 来测试端口是否开放并响应。

一旦我们验证了所需邮件服务的连接性，并且确认主机本身是否已在 Nagios Core 中配置并检查过，我们就可以添加适当的服务检查。

## 如何操作...

我们可以为主机添加未加密邮件服务的检查，包括 SMTP、POP 和 IMAP 服务，方法如下：

1.  切换到 Nagios Core 的 `objects` 配置目录。默认路径是 `/usr/local/nagios/etc/objects`。如果你将主机定义放在了其他文件中，请转到该文件所在目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  marathon.naginet
        alias      marathon
        address    10.128.0.34
    }
    ```

1.  在主机定义下，放置三个新的服务定义，每个适用于一个相应的邮件服务：

    ```
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  SMTP
        check_command        check_smtp
    }
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  POP
        check_command        check_pop
    }
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  IMAP
        check_command        check_imap
    }
    ```

1.  验证配置并重新启动 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart
    ```

完成此步骤后，将开始进行三项新的服务检查，适当的联系人和联系组会在尝试连接任何服务失败时收到通知。这些服务的详细信息也将在网页界面的**服务**部分中显示。

## 它是如何工作的...

上一节中添加的配置将三项新的服务检查添加到现有的 `marathon.naginet` 主机：

+   `SMTP`，使用 `check_smtp` 命令打开 SMTP 会话

+   `POP`，使用 `check_pop` 命令打开 POP 会话

+   `IMAP`，使用 `check_imap` 命令打开 IMAP 会话

在这三种情况下，都会检查服务的连接性和响应性，并在它返回适当值且在可接受时间范围内时判定为 `OK`。

需要注意的是，此处定义的配置并不实际发送或接收任何电子邮件；它仅检查服务的基本连接性，以及是否能响应简单的请求。因此，仅仅因为状态为 `OK` 并不意味着电子邮件消息被正确传递；它可能仅意味着服务在响应。

## 还有更多...

如果需要检查这些服务的安全 SSL/TLS 版本，则配置非常相似，但需要事先进行一些额外设置。这是因为尽管 Nagios 插件中包含用于检查它们的插件，但它们并未被配置为作为命令使用。请注意，这一点在未来的 Nagios Core 版本中可能会发生变化。

为了添加适当的命令，可以将以下段落添加到命令配置文件中，通常是 `/usr/local/nagios/etc/objects/commands.cfg`：

```
define command {
    command_name  check_ssmtp
    command_line  $USER1$/check_ssmtp $ARG1$ -H $HOSTADDRESS$
}
define command {
    command_name  check_spop
    command_line  $USER1$/check_spop $ARG1$ -H $HOSTADDRESS$
}
define command {
    command_name  check_simap
    command_line  $USER1$/check_simap $ARG1$ -H $HOSTADDRESS$
}
```

完成此步骤后，可以将以下服务定义添加到适当的主机中，既可以替换也可以补充未加密服务的检查。它们与未加密版本相同，只是将 `service_description` 和 `check_command` 中添加了 `s`：

```
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SSMTP
 check_command        check_ssmtp
}
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SPOP
 check_command        check_spop
}
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SIMAP
 check_command        check_simap
}
```

最后，注意如果你管理着多台邮件服务器，并且这些服务器运行着上述服务中的一个或多个，那么最好将服务应用到包含所有适用主机的主机组，而不是为每个服务创建新的服务定义。请参阅[第一章中的*在组内所有主机上运行服务*部分，了解如何操作。

## 另见

+   在第一章的*创建新主机*、*创建新服务*、*在组内的所有主机上运行服务*和*创建新主机组*等配方中，有涉及到这些内容。

+   第二章的*创建新命令*配方中介绍了如何操作。

# 监控网络服务

在这个配方中，我们将设置一个服务检查，用来监控 HTTP 和 HTTPS 服务器的响应性。我们将使用 `check_http` 命令及其在 Nagios 插件集中提供的同名插件，向 web 服务器发起 HTTP 和 HTTPS 请求，以确保它返回适当且及时的响应。这在需要检查网站是否仍在正常运行的情况下非常有用，特别是当网站在承受高负载或遭遇拒绝服务攻击时。

## 准备工作

你应该已经有一个 Nagios Core 3.0 或更新版本的服务器，并且至少配置了一个主机。我们将使用 `sparta.naginet` 作为示例，这是一个在其自身文件中定义的主机。你还应该理解主机和服务之间的基本关系，这在第一章的*理解主机、服务和联系人*中有介绍。

一个合适的第一步是确保我们打算检查的服务可以从运行 Nagios Core 的监控服务器访问。这可以通过命令行完成，使用如 `curl` 或 `wget` 这样的 HTTP 客户端：

```
$ wget http://sparta.naginet/
$ curl http://sparta.naginet/

```

`check_http` 插件二进制文件也可以直接调用来测试连接性；我们期望收到一个 `HTTP` `OK` 响应，状态码为 `200`：

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_http -I sparta.naginet
HTTP OK: HTTP/1.1 200 OK - 453 bytes in 0.004 second response time |time=0.004264s;;;0.000000 size=453B;;;0

```

可选地，我们也可以以相同的方式检查 HTTPS，只需要为插件添加 `-S` 选项：

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_http -S -I sparta.naginet
HTTP OK: HTTP/1.1 200 OK - 453 bytes in 0.058 second response time |time=0.057836s;;;0.000000 size=453B;;;0

```

这两者可能需要安装一个默认页面由主机提供服务，通常是像 `index.html` 或 `default.asp` 这样的文件，具体取决于使用的 web 服务器软件。

一旦验证了从监控服务器到主机的 HTTP 连接正常，并且收到了合适的响应，我们就可以继续添加我们的服务检查。

## 如何操作...

我们可以按以下方式为主机添加 web 服务检查：

1.  切换到 Nagios Core 的 `objects` 配置目录。默认路径为 `/usr/local/nagios/etc/objects`。如果你将主机定义放在了其他文件中，请转到相应的目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  sparta.naginet
        alias      sparta
        address    10.128.0.21
    }
    ```

1.  在主机定义下方，放置一个新的服务定义来进行 HTTP 检查：

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTP
        check_command        check_http
    }
    ```

1.  如果还需要进行 HTTPS 检查，可以添加一个可选的第二个服务定义：

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTPS
        check_command        check_http!-S
    }
    ```

1.  验证配置并重启 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成此操作后，一个名为 HTTP 的新服务，以及一个可选的 HTTPS 服务，将会被添加到`sparta.naginet`主机中，并且服务器将定期发起 HTTP 请求，报告连接失败或返回意外状态的情况。这些服务将在 Web 界面的**服务**部分中可见。

## 它是如何工作的...

前面部分中添加的配置使用`check_http`插件对`sparta.naginet`服务器进行定期请求。默认情况下，请求的是首页，因此请求的形式如下：

```
GET / HTTP/1.0
User-Agent: check_http/v1.4.15 (nagios-plugins 1.4.15)
Connection: close

```

插件等待响应，然后根据以下标准返回状态：

+   是否在可接受的时间范围内收到了格式良好的 HTTP 响应。如果响应过慢，可能会在插件超时时引发`CRITICAL`状态。

+   是否收到 HTTP 响应的响应码为`200` `Found`，表示已找到并返回了一个文档。默认情况下，如果收到`404` `Not Found`的响应码，则会触发`CRITICAL`状态。

查看`/usr/local/nagios/etc/objects/commands.cfg`中默认`check_http`命令的定义，可以更好地了解它如何使用同名插件：

```
define command {
    command_name  check_http
    command_line  $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
}
```

这使用了三个 Nagios Core 宏：

+   `$USER1$`：这表示插件脚本和二进制文件所在的目录；通常是`/usr/local/nagios/libexec`。

+   `$HOSTADDRESS$`：这是服务关联主机中定义的`address`指令的值；在本例中为`10.0.128.21`。

+   `$ARG1$`：这是一个额外的参数，如果命令中定义了该参数；它允许我们在`check_http`调用中添加`-S`选项，以便进行 HTTPS 检查。

## 还有更多...

`check_http`插件有很多其他非常有用的选项；通过输入没有参数的命令可以查看它们的列表：

```
# ./check_http
check_http: Could not parse arguments
Usage:
 check_http -H <vhost> | -I <IP-address> [-u <uri>] [-p <port>]
 [-w <warn time>] [-c <critical time>] [-t <timeout>] [-L] [-a auth]
 [-b proxy_auth] [-f <ok|warning|critcal|follow|sticky|stickyport>]
 [-e <expect>] [-s string] [-l] [-r <regex> | -R <case-insensitive regex>]
 [-P string] [-m <min_pg_size>:<max_pg_size>] [-4|-6] [-N] [-M <age>]
 [-A string] [-k string] [-S] [--sni] [-C <age>] [-T <content-type>]
 [-j method]

```

一个特别有用的选项是`-u`选项，它允许我们从服务器请求除默认索引文档以外的特定 URL。如果我们需要为站点上的多个页面设置检查，这个选项就非常有用，也可以作为代码单元测试的一个很好的补充，尤其是在站点部署或更新时。

例如，如果我们想检查三个页面是否返回`200` `Found`响应：`about.php`、`products.php`和`contact.php`，那么我们可以设置一个类似以下的命令来检查特定页面：

```
define command {
    command_name  check_http_page
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -u $ARG1$
}
```

这将允许我们使用新命令进行三个类似的服务检查，如下所示：

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-about
 check_command        check_http_page!/about.php
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-products
 check_command        check_http_page!/products.php
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-contact
 check_command        check_http_page!/contact.php
}
```

这些服务检查的运行方式与食谱中展示的相同，唯一不同的是它们每个都会请求一个特定的页面。请注意，URL 前面的斜杠是必需的。

类似地，`-H`选项允许您指定主机名，这在托管多个站点的服务器上非常有用。可以通过如下配置命令来实现：

```
define command {
    command_name  check_http_host
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -H $ARG1$
}
```

这将允许你检查同一主机上的两个站点，[`www.naginet/`](http://www.naginet/) 和 [`dev.naginet/`](http://dev.naginet/)，分别进行服务检查：

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-www
 check_command        check_http_host!www.naginet
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-dev
 check_command        check_http_host!dev.naginet
}
```

值得注意的是，`check_http`请求会与常规请求一起出现在服务器日志中。如果你担心这些扭曲的统计数据或报告中出现不需要的值，那么使用其`User-Agent`头部值过滤这些请求可能是最简单的办法，`User-Agent`中包含字符串`check_http`。

## 另见

+   本章中的*检查网站返回指定字符串*

+   第一章中的*创建新主机*和*创建新服务*教程，*理解主机、服务和联系人*

+   第二章中的*创建新命令*和*定制现有插件*教程，*工作与命令和插件*

# 检查网站是否返回指定字符串

在本教程中，我们将基于本章中*监控网页服务*教程中建立的基本网页服务监控，学习如何创建一个使用`check_http`插件的命令，以确保 HTTP 响应中包含特定字符串。

默认情况下，Nagios Core 没有定义使用插件的命令，所以这个教程将包括在作为服务检查的一部分之前定义命令。

如果我们正在监控的服务器上的网站可能不会返回`404` `Not` `Found`或类似错误，这些错误会在 Nagios 中标记为`WARNING`或`CRITICAL`状态，那么这样做可能是必要的；与其单纯检查文档是否存在，我们可以检查其是否匹配某个字符串，以确认它是否符合我们预期的特定文档。

这种设置是网站或 Web 应用程序代码单元测试套件的一个很好的补充。

## 准备工作

你应该已经有一个 Nagios Core 3.0 或更高版本的服务器，并且至少已经配置了一个主机。我们将使用`sparta.naginet`作为示例，该主机定义在自己的文件中，并且我们将检查它是否在响应中返回简单字符串`naginet`。你还应该理解主机和服务之间的基本关系，这在第一章，*理解主机、服务和联系人*的教程中有所涵盖。

你应该先为主机设置基本的 HTTP 监控，如本章中的*检查网页服务*教程所述，以确保监控服务器与主机之间有连接，并且请求和响应都正常工作，并返回适当的错误码。

## 如何操作...

我们可以设置一个包含 HTTP 响应内容检查的服务检查，如下所示：

1.  切换到 Nagios Core 的`objects`配置目录。默认路径是`/usr/local/nagios/etc/objects`。如果你将主机定义放在了其他文件中，则转到相应的目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含命令定义的适当文件，并找到`check_http`命令的定义。在 QuickStart 安装中，该文件是`commands.cfg`。`check_http`定义类似于以下代码片段：

    ```
    define command {
        command_name  check_http
        command_line  $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
    }
    ```

1.  在`check_http`定义下方，添加一个新的命令定义，如下所示：

    ```
    define command {
        command_name  check_http_content
     command_line  $USER1$/check_http -I $HOSTADDRESS$ -s $ARG1$
    }
    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  sparta.naginet
        alias      sparta
        address    10.128.0.21
    }
    ```

1.  在主机的定义下方以及任何可能使用`check_http`的检查下方，使用我们的新命令放置一个新的服务定义：

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTP-content
        check_command        check_http_content!naginet
    }
    ```

1.  验证配置并重启 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成此操作后，Nagios Core 应开始像`check_http`一样发出 HTTP 请求以监控该服务，唯一不同的是，只有在网站内容包含`naginet`字符串时，才会返回`OK`状态。否则，它将生成警报，并将该服务标记为`CRITICAL`，消息类似如下：

```
HTTP CRITICAL: HTTP/1.1 200 OK - string 'naginet' not found on 'http://10.128.0.21:80/' - 453 bytes in 0.006 second response time

```

## 它是如何工作的……

`check_http`的众多选项之一是`-s`，即`--string`的缩写，它接受一个指定的字符串作为参数，该字符串必须出现在内容中，才能使服务检查返回`OK`状态。

当收到 HTTP 响应时，`check_http`会检查响应中的文本，看看它是否与指定的字符串匹配，并且除了标记`WARNING`或`CRITICAL`状态用于连接或超时问题外，还会执行通常的行为。

请注意，为了使这个功能工作，有必要定义一个新的命令，使用第一个参数（在此例中是字符串`naginet`）作为`-s`选项传递给`check_http`。执行的完整命令行类似于以下命令：

```
$ /usr/local/nagios/libexec/check_http -H sparta.naginet -s naginet

```

## 还有更多……

`check_http`插件提供的功能远远超出了单个字符串检查，如果需要测试内容中是否存在正则表达式，可以使用`-r`或`--regex`选项。我们可以定义一个命令来检查正则表达式，如下所示：

```
define command {
    command_name  check_http_regex
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -r $ARG1$
}
```

如果需要检查某个特定的正则表达式是否不匹配内容，可以通过添加`--invert-regex`标志来实现：

```
define command {
    command_name  check_http_noregex
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -r $ARG1$ --invert-regex
}
```

使用此命令进行的服务检查如果响应匹配作为`check_command`指令第一个参数提供的模式，将返回`CRITICAL`。

其他类似的选项包括`-e`或`--expect`，它允许指定一个由逗号分隔的字符串集，至少其中一个字符串必须与检查通过的头部的第一行匹配。

## 另请参见

+   本章中的*监控 Web 服务*示例

+   第一章中的*创建新主机*和*创建新服务*示例，*了解主机、服务和联系人*

+   第二章中的*创建新命令*和*自定义现有插件*食谱，*命令和插件的使用*。

# 监控数据库服务

在本食谱中，我们将学习如何使用 Nagios Core 监控数据库服务器的状态。我们将以流行的 MySQL 为例，使用`check_mysql`插件，并讨论在本食谱的*更多内容*部分中运行实际的测试查询和为 PostgreSQL 指定类似的检查。

## 准备工作

你应该已经拥有一个 Nagios Core 3.0 或更高版本的服务器，并且至少配置了一个主机。我们将以`delphi.naginet`为例，这是一个在自己文件中定义的主机。你还应该理解主机和服务之间的基本关系，这在第一章，*理解主机、服务和联系人*的食谱中有介绍。

对于从监控服务器检查远程主机，数据库服务器需要在适当的网络接口上监听。还需要确保存在一个适当的数据库用户帐户，以便`check_mysql`插件进行身份验证。最好将此帐户设置为一个没有任何数据库权限的专用用户，因为凭据需要以纯文本存储，如果使用更敏感的凭据，可能会带来安全风险。

对于 MySQL，我们可以使用以下命令创建一个没有权限的新用户，假设监控服务器`olympus.naginet`的 IPv4 地址是`10.128.0.11`。这里使用了随机生成的密码：

```
mysql> CREATE USER 'nagios'@'10.128.0.11' IDENTIFIED BY 'UVocjPHoH0';

```

然后，我们可以使用监控服务器上的`mysql`客户端检查连接性：

```
$ mysql --host=delphi.naginet --user=nagios --password=UVocjPHoH0
mysql>

```

或者，直接以`nagios`用户身份从命令行运行插件：

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_mysql -H delphi.naginet -u nagios -p UVocjPHoH0
Uptime: 1631  Threads: 1  Questions: 102  Slow queries: 0  Opens: 99 
Flush tables: 1  Open tables: 23  Queries per second avg: 0.62

```

如果在构建 Nagios 插件时没有安装 MySQL 库，可能会发现`/usr/local/nagios/libexec`目录下没有`check_mysql`和`check_mysql_query`二进制文件。我们可以通过在监控系统上安装 MySQL 共享库，重新构建并重新安装 Nagios 插件包来解决此问题。

默认情况下，还需要定义新命令以实际使用这些插件，这将在本食谱中完成。

## 如何操作...

我们可以如下设置一些基本的 MySQL 服务器数据库监控：

1.  切换到 Nagios Core 的`objects`配置目录。默认路径是`/usr/local/nagios/etc/objects`。如果你将主机的定义放在其他文件中，请转到该文件所在的目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含命令定义的适当文件，可能是`commands.cfg`，并添加以下定义：

    ```
    define command {
        command_name  check_mysql
     command_line  $USER1$/check_mysql -H $HOSTADDRESS$ -u $ARG1$ -p $ARG2$
    }
    ```

1.  编辑包含主机定义的文件。主机定义可能看起来像以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  delphi.naginet
        alias      delphi
        address    10.128.0.51
    }
    ```

1.  在主机的定义下方，放置一个新的服务定义，用于 MySQL 检查，包括之前选择的用户名和密码作为参数：

    ```
    define service {
        use                  generic-service
        host_name            delphi.naginet
        service_description  MYSQL
     check_command        check_mysql!nagios!UVocjPHoH0
    }
    ```

1.  验证配置并重启 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成此操作后，将为 `delphi.naginet` 主机添加一个新的服务检查，描述为 `MYSQL`，它将使用 `check_mysql` 插件来报告 MySQL 服务器的状态。输出还将包括其正常运行时间、打开的表以及查询平均值等统计信息，和所有服务输出一样，将在 Web 界面的 **Services** 下显示。

## 它是如何工作的……

此配置定义了一个名为 `check_mysql` 的新命令，使用同名插件，接受两个参数；第一个是测试的 Nagios Core 用户名，在此为 `nagios`，第二个是该用户的密码。`check_mysql` 插件作为 MySQL 客户端，使用提供的凭据，向数据库请求诊断信息，并将其作为检查的一部分返回。

如果连接或使用 MySQL 服务器时遇到问题，它将标记为 `CRITICAL` 状态，并生成相应的通知。

## 还有更多……

我们可以通过为 `-d` 参数提供值，选择性地检查对特定数据库的访问权限。此数据库应该是 `nagios` 用户已被授予访问权限的数据库，否则检查将失败。

如果我们想检查在连接后是否能够实际运行查询，我们可以进一步扩展，使用 `check_mysql_query` 插件：

```
define command {
    command_name  check_mysql_query
 command_line  $USER1$/check_mysql_query -H $HOSTADDRESS$ -u $ARG1$ -p $ARG2$ -d $ARG3$ -q $ARG4$
}
define service {
    use                  generic-service
    host_name            delphi.naginet
    service_description  MYSQL_QUERY
 check_command        check_mysql_query!nagios!UVocjPHoH0!exampledb!"SELECT COUNT(1) FROM exampletbl"
}
```

上述代码片段将尝试在 `exampledb` 数据库上运行 `SELECT` `COUNT(1)` `FROM` `exampletbl` 查询。请注意，将查询用引号括起来很重要，以便它作为一个参数而不是多个参数进行处理。

类似于本食谱中指定的服务检查，可以使用 `check_pgsql` 插件（同样是标准 Nagios 插件集的一部分）为 PostgreSQL 数据库服务器配置。命令和服务检查定义可能类似于以下代码片段：

```
define command {
    command_name  check_pgsql
    command_line  $USER1$/check_pgsql -H $HOSTADDRESS$ -p $ARG1$
}
define service {
    use                  generic-service
    host_name            delphi.naginet
    service_description  PGSQL
    check_command        check_pgsql!N4Nw8o8X
}
```

在上述示例中，需要在 PostgreSQL 服务器的 `pg_hba.conf` 文件中授予监控服务器 IP 地址的访问权限，并访问默认的标准 `template1` 数据库。

在生产环境中，通常出于安全或编程策略的原因，数据库服务器并未配置为接受通过网络接口的直接连接，即使是在安全的接口上。许多系统上的打包 MySQL 和 PostgreSQL 服务器实际上默认仅在 `localhost` 接口 `127.0.0.1` 上监听。

这可能会稍微复杂化监控设置，但通常可以通过在数据库服务器上安装远程 Nagios 插件执行代理来解决，比如 NRPE 或 NSclient++。NRPE 的使用在第六章，*启用远程执行*中有详细讲解，使用的是这样配置的 MySQL 服务器作为示范。

## 另见

+   第一章中的*创建新主机*和*创建新服务*示例，*理解主机、服务和联系人*

+   第二章中的*创建新命令*示例，*使用命令和插件*

+   第六章中的*使用 NRPE 监控远程机器上的本地服务*示例，*启用远程执行*

# 监控 SNMP 查询的输出

在本示例中，我们将学习如何使用 `check_snmp` 插件来监控 **简单网络管理协议**（**SNMP**）请求返回的输出。

尽管名字中有“简单”二字，SNMP 实际上并不是一个非常简单的协议，但它是访问许多种类网络设备信息的常见方法，包括监控板、使用计量器、存储设备，以及工作站、服务器和路由设备。

由于 SNMP 得到了广泛的支持，并且通常能够向受信任的主机提供大量的信息，因此它是从主机收集信息的极好方式，这些信息无法通过网络服务获取。例如，虽然检查来自大型路由器的 PING 响应很简单，但可能没有简单的方法来检查它的接口状态，或者其路由表中某条路由的存在与否。

在 Nagios Core 中使用`check_snmp`可以自动从设备中获取这些信息，并生成相应的警报。虽然它的设置有些复杂，但值得学习如何使用它，因为它是 Nagios Core 中最强大的插件之一，尤其对于网络管理员来说，通常可以在大型网络的配置中看到定义了几十个命令来使用它。它通常可以用来补充或甚至替代远程插件执行守护进程，如 NRPE 或 NSclient++。

## 准备工作

你应该有一个已配置至少一个主机的 Nagios Core 3.0 或更高版本的服务器。你还应该了解主机和服务之间的基本关系，这些内容在第一章的示例中有讲解，*理解主机、服务和联系人*。

本文档假设您具有关于 SNMP 的基本知识，包括其一般预期用途、SNMP 社区的概念，以及 SNMP MIB 和 OID 的含义。特别是，如果您希望监视网络设备的某个属性，该属性通过 SNMP 可用，您应该知道该数据的 OID 是什么。这些信息通常可以在网络设备的文档中找到，或者可以通过针对主机运行适当的`snmpwalk`命令来查看所有 OID 的输出来推断出来。

您应该检查目标主机上是否运行了 SNMP 守护程序，并且监控主机上是否可用`check_snmp`插件。它作为标准 Nagios 插件的一部分包含在内，因此只要在编译这些插件时系统中可用了 Net-SNMP 库，该插件应该是可用的。如果没有，请在监控系统上安装 Net-SNMP 库并重新编译插件。

我们将使用从主机名为`ithaca.naginet`的 Linux 服务器检索总进程计数的示例，并在适当的高范围内标记`WARNING`和`CRITICAL`状态。我们还将讨论如何测试字符串的存在或不存在，而不是数值阈值。

测试主机是否以预期形式响应 SNMP 查询是个好主意。我们可以使用`snmpget`进行测试。假设社区名称为`public`，我们可以写：

```
$ snmpget -v1 -c public ithaca.naginet .1.3.6.1.2.1.25.1.6.0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 81

```

我们还可以通过直接作为`nagios`用户运行插件来测试该插件：

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_snmp -H ithaca.naginet -C public -o .1.3.6.1.2.1.25.1.6.0
SNMP OK - 81 | iso.3.6.1.2.1.25.1.6.0=81

```

## 如何做...

我们可以定义一个命令和服务检查来检查 Linux 进程计数的 OID，如下所示：

1.  切换到 Nagios Core 的`objects`配置目录。默认路径是`/usr/local/nagios/etc/objects`。如果您将主机定义放在不同的文件中，则改为移动到其目录。

    ```
    # cd /usr/local/nagios/etc/objects

    ```

1.  编辑包含命令定义的适当文件，可能是`commands.cfg`，并将以下定义添加到文件末尾。

    ```
    define command {
        command_name  check_snmp_linux_procs
     command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C $ARG1$ -o .1.3.6.1.2.1.25.1.6.0 -w 100 -c 200
    }
    ```

1.  编辑包含主机定义的文件。主机定义可能类似于以下代码片段：

    ```
    define host {
        use        linux-server
        host_name  ithaca.naginet
        alias      ithaca
        address    10.128.0.61
    }
    ```

1.  在主机定义之后，使用我们的新命令添加一个新的服务定义。如果不同，请用您的 SNMP 社区名称替换`public`：

    ```
    define service {
        use                  generic-service
        host_name            ithaca.naginet
        service_description  SNMP_PROCS
     check_command        check_snmp_linux_procs!public
    }
    ```

1.  验证配置并重新启动 Nagios Core 服务器：

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

完成此操作后，将添加一个新的服务检查，其描述为`SNMP_PROCS`，添加到`ithaca.naginet`主机中，并且`check_snmp`插件将按照其正常检查请求指定 OID 的值。如果计数大于`100`，它将标记为`WARNING`状态；如果大于`200`，则标记为`CRITICAL`状态，并相应通知。所有这些都将显示在 Web 界面中的**Services**菜单项中，与任何其他服务一样。

## 如何工作...

上述配置定义了一个新的基于 `check_snmp` 插件的命令，并依此为 `ithaca.naginet` 服务器使用该命令进行新的服务检查。SNMP 请求的社区名称 `public` 作为参数传递给命令；其他一切，包括要请求的 OID，都是固定在 `check_snmp_linux_procs` 命令定义中的。

定义的命令行的一部分包括 `-w` 和 `-c` 选项。对于像我们这样的数值输出，这些选项用于定义超出值的限制，以便分别触发`WARNING`或`CRITICAL`状态。在这种情况下，我们定义了 `100` 个进程的`WARNING`阈值和 `200` 个进程的`CRITICAL`阈值。

同样，如果由于连接问题或语法错误导致 SNMP 检查完全失败，将报告`UNKNOWN`状态。

## 还有更多...

也可以测试 SNMP 检查的输出，以查看它们是否与特定的字符串或模式匹配，从而确定检查是否成功。例如，如果我们需要检查系统的主机名是否为`ithaca.naginet`（可能是一个应该始终成功的简单测试 SNMP 查询），那么我们可以设置如下的命令定义：

```
define command {
    command_name  check_snmp_hostname
 command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C $ARG1$ -o .1.3.6.1.2.1.1.5.0 -r $ARG2$
}
```

具有以下相应服务检查：

```
define service {
    use                  generic-service    host_name            ithaca.naginet
    service_description  SNMP_HOSTNAME
 check_command        check_snmp_hostname!public!ithaca
}
```

这个特定的检查只有在 SNMP 查询成功并返回与第二个参数指定的字符串`ithaca`匹配的字符串时才会成功。

## 另见

+   本章中的*创建 SNMP OID 以进行监控*

+   本书中第一章的*创建新主机*和*创建新服务*配方，*理解主机、服务和联系人*

+   本书中第二章的*创建新命令*配方，*与命令和插件一起工作*

# 监控 RAID 或其他硬件设备

在这个配方中，我们将学习监控硬件设备属性的一般策略。由于厂商实现硬件的方式不同，这通常比监控标准网络服务更复杂。

至少有四种一般方法可以解决这个问题。

## 准备工作

你需要知道一些你想要监控的硬件的具体信息，包括型号。你最好还应该有一个带有 Net-SNMP 库编译的 Nagios Core 3.0 服务器，以便构建`check_snmp`插件，这是 Nagios 插件集的一部分。

## 如何做...

我们可以通过以下方式找到在本地或远程机器上监控任意硬件设备的方法：

1.  检查是否已经存在用于轮询特定设备的官方或非官方 Nagios Core 插件。最好的起点是 Nagios Exchange，网址为[*http://exchange.nagios.org/*](http://exchange.nagios.org/)，只需搜索硬件品牌，看看是否已有插件，按照*第二章*中的*寻找插件*食谱来进行。你可以按照同一章节中的*安装插件*食谱来安装插件。

1.  检查你从硬件中需要的任何值是否已经作为 SNMP OID 导出，或者是否可以导出，以便使用本章中的*监控 SNMP 查询的输出*食谱进行检查。

1.  如果没有，但有一个命令行诊断工具输出，或者一个可以用作检查的返回值，你可以考虑将其作为自定义 OID 导出到 SNMP 服务器中，使用本章中的*创建一个新的 SNMP OID 来监控*食谱。

1.  最后，我们可能不得不自己编写插件。实际上，这并不像看起来那么困难；这一过程在*第二章*中的*从零开始编写新插件*食谱中有所讨论。这可能是定制硬件或非常罕见硬件的唯一选择。

## 工作原理...

如果我们在网上找到适合硬件的插件，主要的难点在于我们不仅要通过在硬件的`OK`和`CRITICAL`状态下测试插件来确保其有效（这可能很难做到），还要确保插件是安全的。Nagios Exchange 上的插件在添加之前会进行审查，但你在任何其他网站上找到的插件代码可能不安全运行。

尽可能使用 SNMP 进行这些类型的检查有两个优势：一是可以通过标准的 Nagios 插件`check_snmp`来检查这些值，二是这些值可以通过网络读取，这意味着我们可能不需要依赖如 NRPE 或 NSclient++之类的远程执行守护进程来获取这些信息。

## 另见

+   本章中的*监控 SNMP 查询的输出*和*创建一个新的 SNMP OID 来监控*食谱

+   *第二章*中的*寻找插件*、*安装插件*和*从零开始编写新插件*食谱，*使用命令和插件*

# 创建一个 SNMP OID 用于监控

在本食谱中，我们将学习如何在 Linux 服务器上配置一个 Net-SNMP `snmpd`服务器，以便通过 SNMP OID 返回命令的输出。这可以作为 NRPE 监控的替代方案，用于检查网络服务中无法直接访问的信息，以便 Nagios Core 可以通过其标准的`check_snmp`方法进行检查。

例如，这可以是监控硬件设备的一个非常好的方式，例如远程服务器上的**RAID**阵列，其中提供命令行诊断工具来报告状态为数字或字符串，但这些工具仅在本地有效，并且不会在 SNMP MIB 树中包含任何信息。

## 准备就绪

我们要检查的主机应运行 Net-SNMP `snmpd`服务器，允许对指定社区字符串（如`public`）的 MIB 树进行完全的读取访问。该 SNMP 服务器应能够在配置中使用`exec`指令，在 SNMP 客户端请求时返回命令的输出作为 SNMP OID 的值。因此，您需要了解 SNMP 的基础知识。

您还需要一个启用了 SNMP 的 Nagios Core 3.0 或更新版本的服务器，才能实际监控 OID，这将在*监控 SNMP 查询输出*方法中说明。

在本示例中，我们将处理一个简单的脚本，它在目标主机`ithaca.naginet`上运行，脚本路径为`/usr/local/bin/raidstat`，它返回一个整数：零表示 RAID 状态良好，非零则表示有问题。我们假设该工具可以以任何用户身份运行，并且不需要 root 权限。

## 如何操作...

我们可以按照以下方式设置自定义 SNMP OID：

1.  将以下行添加到目标主机上的`snmpd.conf`文件中，替换生成所需输出的命令路径：

    ```
    exec raidstat /usr/local/bin/raidstat

    ```

1.  重新启动目标主机上的`snmpd`服务器，具体操作可能如下，具体取决于系统：

    ```
    # /etc/init.d/snmpd restart

    ```

1.  在监控服务器上，我们可以遍历`.1.3.6.1.4.1.2021` OID，找到我们的`raidstat` OID 及其值：

    ```
    $ snmpwalk -v1 -c public ithaca.naginet .1.3.6.1.4.1.2021 | less
    ...
    iso.3.6.1.4.1.2021.8.1.2.1 = STRING: "raidstat"
    iso.3.6.1.4.1.2021.8.1.3.1 = STRING: "/usr/local/bin/raidstat"
    iso.3.6.1.4.1.2021.8.1.100.1 = INTEGER: 0
    iso.3.6.1.4.1.2021.8.1.101.1 = STRING: "GOOD"
    ...

    ```

1.  现在我们知道可以使用本章中的*监控 SNMP 查询输出*方法来查询检查 OID，并可以直接通过`snmpget`测试获取它：

    ```
    $ snmpget -v1 -c public ithaca.naginet .1.3.6.1.4.1.2021.8.1.100.1

    ```

## 它是如何工作的...

`snmpd`配置中的`exec`调用定义了一个应该执行的程序，用于返回 OID 树中的新值。我们在输出中找到的四个 OID 如下：

+   `1.3.6.1.4.1.2021.8.1.2.1`: 这是我们分配的 OID 名称，`raidstat`

+   `1.3.6.1.4.1.2021.8.1.3.1`: 这是被调用脚本的完整路径，`/usr/local/bin/raidstat`

+   `1.3.6.1.4.1.2021.8.1.100.1`: 这是来自调用的返回值，以整数形式表示

+   `1.3.6.1.4.1.2021.8.1.101.1`: 这是来自调用的任何输出，以字符串形式表示

因此，这种设置可以用来检查任何命令的返回值和字符输出，只要该命令是`snmpd`用户能够执行的，对于像这样的特定情况，这是一个很好的临时监控方法。

## 还有更多...

如果需要，可以向服务器配置中添加多个`exec`命令。例如，如果我们需要通过命令`tempstat`检查 CPU 温度的状态，那么我们可能会定义：

```
exec raidstat /usr/local/bin/raidstat
exec tempstat /usr/local/bin/tempstat

```

这两个的返回值和输出将显示在 SNMP 输出中，并且在不同的 OID 中。

如果需要，命令定义后还可以跟随参数：

```
exec tempstat /usr/local/bin/tempstat --cpu=0

```

本书不涉及`exec`和类似的`pass`配置在 Net-SNMP 中的工作原理，但在本书编写时的文档中有广泛讨论，文档可以通过[`www.net-snmp.org/docs/readmefiles.html`](http://www.net-snmp.org/docs/readmefiles.html)访问。

请注意，如果通过 SNMP 导出值不适用，远程监控的替代方法是使用**Nagios 远程插件执行器** (**NRPE**) 在目标服务器上运行检查，并将结果返回给监控服务器。有关这方面的内容，请参见第六章，*启用远程执行*中的配方。

## 另请参见

+   本章中的*创建 SNMP OID 以进行监控*配方

+   第一章中的*创建新主机*和*创建新服务*配方，*理解主机、服务和联系人*

+   第二章中的*创建新命令*配方，*与命令和插件一起工作*

+   第六章中的*使用 NRPE 监控远程机器上的本地服务*配方，*启用远程执行*
