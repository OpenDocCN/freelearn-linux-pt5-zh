# 前言

说实话，如果几年前有人让我写一本书，我肯定会回答说这是不可能的，原因有很多，既有个人的，也有职业的。从 2001 年初我在 Sun Microsystems 教授我的第一门课程以来，发生了许多事情（那时，我正在使用 Sun Solaris 7）。如今，我很感激能够继续从世界各地的优秀专业人士那里学习更多关于这个出色操作系统的知识，他们本可以写这本书。

我不得不承认，我是 Oracle Solaris 的忠实粉丝，我多年的实践经验让我看出，它依然是世界上最好的操作系统，并且在一段时间里，它也一直无与伦比。当有人谈到性能、安全性、一致性、功能和可用性时，我总是会想到同一个问题：Oracle Solaris。

可能会有人不同意我的观点，我可以尝试解释我的立场，批评其他优秀的操作系统，如 Linux、AIX、HP-UX，甚至 Windows，但这样做既不高效，也不礼貌。相反，我认为更适合的方式是教你 Oracle Solaris 的高级功能及其使用案例，你可以根据自己的理解做出结论。

Oracle 在 Oracle Solaris 上投入了大量资金，自那时以来，许多优秀且先进的功能被引入，使得 Oracle Solaris 得到了极大的改善，而本书也正是从这一点开始的。

*Oracle Solaris 11 高级管理烹饪书*旨在逐步展示和解释如何在 Oracle Solaris 11 系统上执行日常任务的专门程序，其中每个命令都经过测试，并展示其输出。此外，本书还将致力于回顾一些 Oracle Solaris 11 中级管理的关键主题，所有基础和高级管理概念将根据需要介绍，帮助读者理解那些晦涩的内容。

在写这本书的过程中，我学到了很多，并测试了不同的场景和方法，只为将最核心的概念和程序带给你，所有的命令和输出都来自我自己的实验室。顺便说一下，整本书是在 x64 机器上编写的，因为大多数人很难访问基于 SPARC 的系统。

最后，我希望你在阅读这本书时能像我写它时一样享受其中。我希望你喜欢它！

# 本书涵盖的内容

第一章，*IPS 和启动环境*，涵盖了 IPS 和启动环境管理的各个方面，解释了如何管理软件包、配置 IP 仓库以及创建你自己的软件包。此外，本章还讨论了 BE 管理及其相关操作。

第二章, *ZFS*，解释了 ZFS 的卓越世界。本章重点介绍了 ZFS 池和文件系统的管理，以及如何处理快照、克隆和备份。此外，还将讨论使用 ZFS 影子、与 SMB 共享的 ZFS 共享以及日志。最后，将详细解释如何镜像根池以及如何使用 ZFS 交换空间。

第三章, *网络*，介绍了反应式网络配置、链路聚合设置和 IPMP 管理。还将详细解释其他复杂主题，如网络桥接、链路保护和集成负载均衡器。

第四章, *Zones*，展示了如何管理虚拟网络并在区域上部署资源管理器。还将讨论诸如流量控制和区域迁移等补充和有趣的主题。

第五章, *使用 Oracle Solaris 11 服务*，帮助您了解所有 SMF 操作，并回顾有关如何管理服务的基本概念。此外，本章还逐步解释并展示了创建 SMF 服务、处理清单和配置文件、管理网络服务以及排除 Oracle Solaris 11 服务故障的方法。

第六章 服务器"), *配置和使用自动安装程序 (AI) 服务器*，带您了解端到端的自动安装程序 (AI) 配置步骤，并提供如何从 AI 服务器安装 x86 客户端的所有信息。

第七章, *配置和管理 RBAC 和最小特权*，解释了如何配置和管理 RBAC 和最小特权。重点是保持 Oracle Solaris 安装的安全性。

第八章, *管理和监控进程*，提供了一个有趣的方法来处理进程及其优先级。

第九章, *配置 Syslog 和监控性能*，提供了逐步配置 Syslog 服务的方法，并且对 Oracle Solaris 11 中的性能监控进行了良好的介绍。

# 本书所需内容

我相信你非常清楚如何安装 Oracle Solaris 11。然而，仍然有必要向你展示如何配置一个简单的环境，以执行本书中的每个步骤。一个做得好的环境将帮助我们通过执行所有命令、示例和步骤来理解本书中的每一个概念。最终，你应该记住这是一本实践性很强的书！

要按照本教程操作，必须有一台物理机器，至少配备 8 GB RAM 和约 80 GB 的硬盘空间。此外，这台主机应运行与 VMware 或 VirtualBox 虚拟化软件兼容并受支持的操作系统，包括支持硬件虚拟化的 Intel 或 AMD 处理器。你还需要一台运行中的 Solaris 11，该系统将作为虚拟机（VMware 或 VirtualBox）安装和配置。

为了准备好你的环境，你需要执行以下步骤：

1.  首先，你应该从 Oracle 官网下载 Oracle Solaris 11 ([`www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html`](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html))。建议选择*Oracle Solaris 11 Live Media for x86*方法，因为它比*文本安装器*方法更简单，并且允许我们在实际安装之前从 DVD 启动 Oracle Solaris 11。例如，如果我们使用的是物理机器（而不是通常使用的虚拟机），它会提供一个名为*设备驱动程序实用程序*的工具，用来检查 Oracle Solaris 11 是否拥有物理硬件的所有驱动程序软件。不过，如果我们想在 SPARC 机器上安装 Oracle Solaris 11，那么应选择*文本安装器*方法。

1.  我们应该从 Oracle Solaris 仓库镜像中下载所有文件，并将它们连接成一个单一文件（`# cat part1 part2 part3 … > sol-11-repo-full.iso`）。这个最终的镜像将在第一章中使用，*IPS 和启动环境*，我们会讨论如何配置 IPS 本地仓库。

1.  在本书后续章节中，将会讲解如何配置*Oracle Solaris 11 自动安装*，因此建议你花时间下载*Oracle Solaris 11 自动化*安装镜像 DVD 版本，适用于 x86，从[`www.oracle.com/technetwork/server-storage/solaris11/downloads/install-2245079.html`](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/install-2245079.html)。

1.  必须获取一些虚拟化工具来创建虚拟机进行 Oracle Solaris 11 安装，比如 VMware Workstation ([`www.vmware.com/products/workstation/workstation-evaluation`](http://www.vmware.com/products/workstation/workstation-evaluation)) 或可以从 [`www.virtualbox.org/`](https://www.virtualbox.org/) 下载的 Oracle VirtualBox。

1.  不幸的是，本书中无法详细介绍如何安装 Oracle Solaris 11。然而，Oracle 技术网络（OTN）上有一篇很好的文章，解释并展示了详细的逐步安装过程，网址是 [`www.oracle.com/technetwork/articles/servers-storage-admin/solaris-install-borges-1989211.html`](http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris-install-borges-1989211.html)。

1.  记住，在 LiveCD GUI 安装方法中，root 用户始终被配置为角色，而这一操作与 *文本安装程序* 方法不同，后者允许我们选择是否将 root 用户配置为角色。

1.  以防读者忘记如何将 root 角色恢复为用户角色，我们可以执行以下命令：

    ```
    root@solaris11:/#  su - root
    root@solaris11:/#  rolemod  -K  type=normal  root

    ```

    之后，需要注销并重新登录系统才能使用 root 用户。

1.  最后，我们建议你通过运行以下命令来验证 Oracle Solaris 11 是否正常工作：

    ```
    root@solaris11:/# svcs  network/physical
    STATE          STIME    FMRI
    online         13:43:02 svc:/network/physical:upgrade
    online         13:43:18 svc:/network/physical:default

    root@solaris11:~# ipadm show-addr
    ADDROBJ       TYPE       STATE          ADDR
    lo0/v4      static        ok       127.0.0.1/8
    net0/v4     dhcp          ok       192.168.1.111/24
    lo0/v6      static        ok       ::1/128
    net0/v6     addrconf      ok       fe80::a00:27ff:fe56:85b8/10
    ```

我们已经完成了环境设置。现在是时候开始学习了！

# 本书的读者对象

如果你是 IT 专业人士、IT 分析师，或拥有 Oracle Solaris 11 中级管理知识，并希望学习和部署 Oracle Solaris 11 的高级功能，那么这本书适合你。此外，这是一本实用的书籍，需要运行 Oracle Solaris 11 虚拟机的系统。

# 约定

本书中包含多种文本风格，用于区分不同类型的信息。以下是一些风格示例及其含义说明。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账户名等均如下所示：“用于检测 `nmap` 包损坏的命令成功识别了确切问题。”

任何命令行输入或输出都如下所示：

```
root@solaris11:~# beadm list
BE                Active Mountpoint Space    Policy  Created  
-------------     ------ ---------- -------  ------  ---------- 
solaris           NR     /          4.99G    static  2013-10-05 20:44
solaris-backup-1  -      -          163.0K   static  2013-10-10 19:57
solaris-backup-b  -      -          173.0K   static  2013-10-12 22:47
```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词汇，例如在菜单或对话框中，通常以这样的方式呈现：“要启动包管理器界面，请进入**系统** | **管理员** | **包管理器**。”

### 注意

警告或重要提示通常以框的形式呈现。

### 提示

提示和技巧通常以这种方式呈现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们你对本书的看法——你喜欢或不喜欢的部分。读者反馈对我们开发能够帮助你充分利用的书籍非常重要。

若要向我们发送一般反馈，请发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果你在某个主题上有专业知识，并且有兴趣为书籍写作或贡献内容，请查看我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 上的作者指南。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们为您提供了一些帮助，让您从购买中获得最大收益。

## 勘误

尽管我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码的错误——我们将非常感激您能报告给我们。通过这样做，您可以帮助其他读者避免困惑，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)进行报告，选择您的书籍，点击**勘误提交表单**链接，并输入勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该书籍现有勘误的列表中，位于该标题的勘误部分。您可以通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有的勘误。

## 盗版

互联网上的版权资料盗版是一个在所有媒体中持续存在的问题。我们在 Packt 非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，无论以何种形式，请立即提供该地址或网站名称，以便我们采取措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版内容的链接。

我们感谢您帮助保护我们的作者，并支持我们为您提供有价值的内容。

## 问题

如果您在书籍的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
