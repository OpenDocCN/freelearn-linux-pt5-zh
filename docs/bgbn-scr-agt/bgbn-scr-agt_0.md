# 前言

BeagleBone Black 以不同的方式激发了嵌入式黑客的兴趣。有些人把它看作是一个小型机器人控制板，而另一些人则将其视为一个低功耗网络服务器。在本书中，我们将 BeagleBone Black 视为个人安全和隐私的捍卫者。每一章将讨论一种技术，并提供一个项目来加强对该概念的理解。

让我们开始吧！

# 本书内容

第一章，*创建你的 BeagleBone Black 开发环境*，通过展示如何设置 Emacs 来最大化你的嵌入式黑客之旅，开启了这段旅程。

第二章，*使用 Tor 桥接绕过审查*，讲解了如何将 BeagleBone Black 转变为一个带有前面板界面的 Tor 服务器。

第三章，*使用 CryptoCape 添加硬件安全*，研究了生物识别认证和专用安全芯片。

第四章，*使用受信平台模块保护 GPG 密钥*，向你展示了如何利用 BeagleBone Black 来保护电子邮件加密密钥。

第五章，*离线聊天*，详细介绍了如何在 BeagleBone Black 上运行 IRC 网关，以加密即时消息。

附录，*参考书目*，列出了本书中引用的所有参考文献。

# 本书所需的工具

本书包含多个独立的项目，涉及各种软件包和硬件组件。所有软件都是开源的，安装说明贯穿于每一章。所有必要的硬件清单都列在[`www.sparkfun.com/wish_lists/93119`](https://www.sparkfun.com/wish_lists/93119)的 SparkFun Electronics 愿望清单上。每一章会列出该项目所需的组件，因此最好先阅读该章内容，再收集必要的硬件。

# 本书适合的人群

如果你对个人安全和互联网隐私感兴趣，那么你应该会喜欢这本书。如果你有安全背景，但对嵌入式计算不太熟悉，那么你会觉得这些项目具有挑战性，但也非常有收获。相反，如果你精通电子学，并且正在使用 GNU/Linux 发行版，但尚未学习本书中提到的安全技术，那么你应该会喜欢每一章中关于这些技术的讨论。

# 使用约定

在本书中，你会看到一些不同的文本样式，用于区分不同类型的信息。以下是一些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名都如下所示：“由`FrontPanelDisplay`类控制的 LCD 通过 BBB 的 UART 4 以 9600 波特率写入串口`/dev/ttyO4`。”

一段代码如下所示：

```
self.clear_screen()
up_str = '{0:<16}'.format('Up:   ' + self.block_char * up)
dn_str = '{0:<16}'.format('Down: ' + self.block_char * down)

self.port.write(up_str)
self.port.write(dn_str)
```

当我们希望引起您对代码块中特定部分的注意时，相关行或项目会以粗体显示：

```
import Adafruit_BBIO.UART as UART
import serial
class FrontPanelDisplay(object):

  def __init__(self):
    self.uart = 'UART4'
    UART.setup(self.uart)
 self.port = serial.Serial(port="/dev/ttyO4", baudrate=9600)
    self.port.open()
```

任何命令行输入或输出都写成如下格式：

```
Mar 25 21:37:43.000 [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
Mar 25 21:37:43.000 [notice] Bootstrapped 100%: Done.

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的文字，菜单或对话框中的文字例如，会这样出现在文本中：“要连接到您的桥接设备，请启动 Tor 浏览器并在启动时点击**打开设置**。”

### 注意

警告或重要说明会以类似这样的框框形式出现。

### 提示

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对本书的看法——喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能够充分利用的书籍非常重要。

若要向我们提供一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书名。如果您在某个话题上有专业知识，并且有兴趣写书或为书籍做贡献，请参阅我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

现在，您已成为 Packt 书籍的骄傲拥有者，我们为帮助您最大化购买收益，提供了若干帮助资源。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您从其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误还是可能会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)提交，选择您的书籍，点击**勘误提交表格**链接，并输入您的勘误详情。一旦您的勘误被验证，我们将接受您的提交，并将勘误上传至我们网站，或添加到该书籍现有勘误列表中的勘误部分。任何现有的勘误可以通过选择您的书籍，从[`www.packtpub.com/support`](http://www.packtpub.com/support)查看。

## 盗版

网络上的版权侵权问题是所有媒体面临的持续性问题。在 Packt，我们非常重视对版权和许可证的保护。如果您在互联网上发现我们作品的任何非法复制，无论是什么形式，请立即提供相关地址或网站名称，以便我们采取必要的措施。

如发现涉嫌盗版的资料，请通过`<copyright@packtpub.com>`与我们联系，并提供相关链接。

我们感谢您在保护我们的作者以及帮助我们提供有价值内容方面的支持。

## 问题

如果您在书籍的任何方面遇到问题，请通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
