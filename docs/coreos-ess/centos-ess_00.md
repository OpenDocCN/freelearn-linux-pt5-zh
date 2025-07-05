# 前言

CoreOS 是一种新型的 Linux 操作系统，经过优化可运行 Linux 容器，如 Docker 和 rkt。它具有完全自动化的更新系统、没有包管理器，并且拥有完全集群化的架构。

无论你是 Linux 专家还是刚入门并对 Linux 有一些了解的初学者，本书将为你提供逐步的安装和配置 CoreOS 服务器的说明，同时构建开发和生产环境。你将接触到新的 CoreOS rkt 应用容器运行时引擎和谷歌的 Kubernetes 系统，它能让你将一个 Linux 容器集群作为一个单独的系统进行管理。

# 本书内容概述

第一章, *CoreOS – 概述与安装*, 包含了关于 CoreOS 的简要概述，讲解了 CoreOS 的相关内容。

第二章, *使用 etcd 入门*, 解释了什么是 etcd 以及它的用途。

第三章, *使用 systemd 和 fleet 入门*, 介绍了 systemd 的概述。本章告诉你什么是 fleet 以及如何使用它来部署 Docker 容器。

第四章, *管理集群*, 是关于设置和管理集群的指南。

第五章, *构建开发环境*, 向你展示如何设置 CoreOS 开发环境来测试你的应用容器。

第六章, *构建部署设置*, 帮助你设置代码部署、Docker 镜像构建器和私有 Docker 注册表。

第七章, *构建生产集群*, 讲解了如何在云端设置 CoreOS 生产集群。

第八章, *介绍 CoreUpdate 和容器/企业注册表*, 概述了免费和付费的 CoreOS 服务。

第九章, *CoreOS rkt 入门*, 告诉你什么是 rkt 以及如何使用它。

第十章, *Kubernetes 入门*, 教你如何设置和使用 Kubernetes。

# 本书所需条件

对于本书，你需要一个 Linux 系统或一台 Apple Mac 电脑，以及一个 Google Cloud 账户来运行示例。同时，你还需要最新版本的 VirtualBox 和 Vagrant 来运行脚本。

# 本书适用对象

本书适用于任何 Linux/Unix 系统管理员。即使是对 Linux/Unix 有基本了解的人，在使用本书时也会有一定的优势。

本书同样适用于已经熟悉网络虚拟化的系统工程师和系统管理员，他们希望了解如何利用 CoreOS 开发计算网络，以便部署应用程序和服务器。他们必须具备扎实的 Linux 操作系统和应用容器知识，最好曾经使用 Linux 发行版进行开发或管理工作。

# 约定

本书中，你将会看到多种不同风格的文本，区分不同类型的信息。以下是这些风格的一些示例，并对其含义进行解释。

文本中的代码词汇如下所示：“我们可以通过使用`include`指令包含其他上下文。”

代码块如下所示：

```
  etcd2:
    name: core-01
    initial-advertise-peer-urls: http://$private_ipv4:2380
    listen-peer-urls: http://$private_ipv4:2380,http://$private_ipv4:7001
    initial-cluster-token: core-01_etcd
    initial-cluster: core-01=http://$private_ipv4:2380
    initial-cluster-state: new
    advertise-client-urls: http://$public_ipv4:2379,http://$public_ipv4:4001
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
  fleet:
```

任何命令行输入或输出都如下所示：

```
$ git clone https://github.com/coreos/coreos-vagrant/

```

**新术语**和**重要词汇**以粗体显示。例如，你在屏幕上、菜单或对话框中看到的词汇，文本中会像这样出现：“我们应该在**终端**窗口中看到这个输出。”

### 注意

警告或重要提示以框的形式出现，如下所示。

### 提示

小贴士和技巧以如下方式出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对本书的看法——你喜欢或不喜欢什么。读者的反馈对我们非常重要，因为它帮助我们开发出更符合你需求的书籍。

若要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在某个领域有专长，并且对写作或参与书籍的编写感兴趣，可以查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经成为一本 Packt 图书的自豪拥有者，我们提供了一些工具帮助你从购买中获得最大收益。

## 下载示例代码

你可以从你的账户中下载示例代码文件，访问 [`www.packtpub.com`](http://www.packtpub.com) 获取你所购买的所有 Packt 出版的书籍的示例代码。如果你是在其他地方购买的本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以便直接将文件通过电子邮件发送给你。

## 勘误

尽管我们已尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。这样，您不仅能帮助其他读者避免困扰，还能帮助我们改进后续版本的书籍。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详细信息。待勘误确认后，您的提交将被接受，勘误将上传至我们的网站，或添加到该书标题的现有勘误列表中。

要查看之前提交的勘误，访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书名。所需信息将在**勘误**部分显示。

## 盗版

互联网上的版权素材盗版问题在所有媒体中都是一个持续存在的问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们的作品的任何非法复制，请立即向我们提供其位置地址或网站名称，以便我们采取措施。

我们感谢您帮助保护我们的作者，并支持我们为您提供有价值的内容。

## 问题

如果您在书中的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决您的问题。
