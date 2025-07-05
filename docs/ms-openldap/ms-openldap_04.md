# 第四章。保护 OpenLDAP

在第二章中，我们安装了 OpenLDAP 并创建了 SLAPD 服务器的基本配置文件。然后，在上一章中，我们将注意力转向了 LDAP 操作和 LDAP 客户端。现在，我们将返回到 SLAPD 服务器，但重点将有所不同：**安全性**。我们将重点关注 OpenLDAP 的三个主要安全考虑因素：保护服务器和客户端之间的连接、验证目录用户的身份，以及指定特定用户可以访问哪些数据（以及他们可以以何种方式访问）。我们将从实际角度来审视这些安全考虑因素，并在此过程中涵盖以下内容：

+   配置 SSL 和 TLS 以保护网络数据

+   使用简单绑定来验证 DNS（域名系统）以使用目录

+   使用 SASL 提供更强大的身份验证服务

+   集成 SASL 和客户端 SSL/TLS 证书进行身份验证

+   配置访问控制列表（ACL）以建立有关用户可以访问哪些数据的规则

# LDAP 安全性：三大方面

正如我们已经看到的，目录包含敏感信息。一个例子就是 `userPassword` 属性。但目录中也可能包含其他被认为是敏感的信息，如个人信息或关于组织的机密信息。这些信息需要得到保护。

我们可能会问，这里所说的*保护*是什么意思。因为我们显然并不是要阻止*所有*客户端查看*所有*内容。我们真正想要的，是允许人们访问特定的目录信息。然而，另一方面，也有些情况是我们希望拒绝某些用户访问特定的目录信息。所以，保护我们的数据成为在某些情况下提供信息，而在其他情况下拒绝信息的一个问题。

虽然可以进行更精细的区分，但在这里我们将考虑保护目录及其信息的三个广泛安全方面。这三个方面如下：

+   **连接安全性**：这是保护目录信息（以及客户端信息）在客户端和目录服务器之间传递过程的过程。我们将在网络安全的背景下讨论这一点，涉及 SSL 和 TLS。

+   **身份验证**：这是确保尝试访问目录信息的用户确实是其声称身份的过程。在本章中，我们将介绍两种身份验证类型：简单绑定和 SASL 绑定。SASL 代表**简单身份验证和安全层**。

+   **授权**：这是确保已识别或已验证的用户被允许访问目录中某些信息的过程。OpenLDAP 的 ACL 用于指定授权规则。

在本章中，我们将探讨这三个安全方面。通过将这三者结合起来，我们可以为我们的目录信息提供适当细粒度的保护。

# 使用 SSL/TLS 保护基于网络的目录连接

我们将要研究的第一个安全要素是网络安全。大多数客户端通过网络接口连接到 OpenLDAP，客户端请求和服务器响应都通过网络传输。

LDAP 协议默认情况下以明文形式发送和接收消息。在这种情况下，数据在通过网络传输时不会进行任何隐藏处理。明文传输有几个优点：

+   它更容易配置和维护。

+   LDAP 服务可以更快速地运行。加密和解密消息的过程可能会占用大量处理器资源，去除这些处理过程可以加速操作。

但这些优点是以安全性为代价的。网络上的其他设备可能能够拦截这些未加密的传输并读取其内容，从而可能获取敏感信息。在小型局域网（LAN）中，风险可能较小（尽管依然存在）。而在大规模网络中，例如互联网，风险则要大得多。

在本节中，我们将介绍配置**安全套接字层**（**SSL**）和**传输层安全性**（**TLS**）加密的过程，以保护数据在网络上传输时的安全。SSL 和 TLS 非常相似，以至于这两个术语经常被作为同义词使用（通常是可以接受的）。不过，TLS 是 SSL 的改进版本，实施方式比典型的 SSL 实现更加灵活。StartTLS 方法就是一种保护连接的例子。

## SSL 和 TLS 的基础

OpenLDAP 提供了两种加密网络流量的方法。第一种是让 OpenLDAP 在一个特殊端口上监听请求（默认使用端口 636，即 LDAPS 端口）。这个端口上的传输会自动进行加密。这种方法较旧，是作为 LDAP v2 的一个附加功能引入的，但现在已不再是首选方法。

第二种方法是 LDAP v3 标准的一部分，允许客户端在标准端口（通常是端口 389）上连接时，申请从明文传输切换到加密传输。我将在这里介绍这两种配置。

**安全套接字层**（**SSL**）是一种安全过程，最初由 Netscape 通信公司为其网页浏览器开发，旨在提供一种安全的方式，在服务器和任何客户端之间交换可信的信息。SSL 过程的两个主要特性是：建立身份验证和进行安全加密交易。

随着 SSL 的发展和演变，它被移交给了一个标准化组织——**互联网工程任务组（IETF）**，进行标准化和持续开发。IETF 将其重新命名为**传输层安全（TLS）**，并发布了 1.0 版本（作为 RFC 2246）。SSL 3.0 和 TLS 1.0 没有显著差异，大多数支持其中一种的服务器也支持另一种。由于它们的相似性和共同的历史，我通常将它们合并称为 SSL/TLS。

### 真实性

证明真实性和提供加密是 SSL/TLS 的两个主要特性。关于第一个，SSL/TLS 提供了一种建立服务器真实性的方法（如果需要，也可以验证客户端的真实性）。这意味着 SSL/TLS 使客户端能够合理地确认服务器确实属于它声称的所有者。

考虑一下在线银行的情况。如果我使用浏览器登录到银行的网站并进行一些交易，我希望确保我连接到的网站确实是我的银行网站，而不是一个冒充我银行的网站。SSL/TLS 提供了使用**X.509 证书**来验证服务器真实性的工具。X.509 证书包含三个重要信息：

+   有关证书所有者的个人或组织的信息

+   一个公钥（我们将在下一部分讨论）

+   **数字签名**由证书颁发机构（CA）提供

证书被设计为一种保证，确保某个服务器与特定的个人或组织相关联。当我联系我认为是我的银行的服务器时，我希望得到一些保证，确认它确实是我银行的服务器。因此，证书中包含的一项信息是关于谁拥有该证书的信息。我们可以自己检查这些信息，但由于证书包含数字签名，软件也可以以一种比简单读取证书并信任证书更可靠的方式来验证这些信息。

数字签名是加密的信息块。它使用证书颁发机构（CA）拥有的特殊“私钥”进行加密。然后，CA 可以发布一个公钥，客户端软件可以使用它来验证证书确实是由 CA 签署的。CA 在建立信任方面起着非常重要的作用。我们将在*加密*部分讨论公钥和私钥。

证书颁发机构（CA）负责颁发证书。理想情况下，CA 是一个受信任的来源，可以验证证书的真实性，并提供保证，确保证书确实由声明拥有该证书的组织或个人所有。

有许多商业 CA 提供收费的证书生成服务。为了通过这些服务获取证书，组织或个人必须提供一些信息，用以验证申请证书的人或组织是否合法。一旦对这些信息进行了调查，并且申请人或组织支付了相应的费用，CA 就会签发一个数字签名的证书。

大型 CA 的证书默认包含在大多数支持 SSL 的应用程序中，如流行的网页浏览器（如 Mozilla Firefox）和 SSL 库（如 OpenSSL）。这些证书包含了验证数字签名所需的公钥。因此，当客户端获取到由这些 CA 签名的 X.509 证书时，它就具备了验证证书真实性所需的所有工具。

但是，对于一个组织或个人来说，创建一个本地使用的证书颁发机构（CA）并使用该 CA 为内部应用生成证书是可行的，而且通常是有用的。这就是我们在为 OpenLDAP 创建证书时所做的事情。

当然，这种方式生成的证书可能不会被组织外的用户认为是可靠的，但托管一个个人或组织级别的 CA 可以有效地为自己的网络增加安全性，而无需购买来自商业供应商的证书。

### 注意

并非所有 CA 都采用相同形式的权威签名（也并非所有 CA 都收取证书费用）。有些 CA，如 Cacert.org，采用一种称为 **信任网** 的技术来建立身份认证。在信任网中，证书的真实性由同伴验证，他们可以担任确保证书归属所声明的个人或组织的角色。欲了解更多信息，请访问 [`www.cacert.org/`](http://www.cacert.org/)。

我们已经讨论了 SSL/TLS 的第一个功能：建立身份认证。接下来，我们将讨论 SSL/TLS 的第二个功能：提供加密服务。

### 加密

SSL/TLS 提供了客户端和服务器之间发送加密消息所需的功能。简而言之，过程如下：服务器将证书发送给客户端，在证书中（以及其他内容中）包含了服务器的**公钥**。公钥是一对密钥中的第一部分。公钥可用于加密消息，但不能解密消息。第二个密钥，**私钥**，则用于解密消息。服务器将私钥保密，但会将公钥提供给任何请求的客户端。客户端可以将消息发送给服务器，只有服务器可以解密并解读这些消息。

根据配置，客户端还会向服务器发送其公钥，服务器可以利用这个公钥发送只有客户端能够解密的消息。此时，双方可以互相发送加密消息。

使用公钥/私钥的缺点是：它们速度较慢且资源消耗大。与其通过这些公钥/私钥组合交换所有信息，客户端和服务器会协商一组临时对称加密密钥（使用相同的密钥来加密和解密消息），它们将在会话期间共同使用。客户端之间的所有流量都使用这些密钥进行加密。一旦会话完成，客户端和服务器都会丢弃临时密钥。

### 注意

关于 SSL 和 TLS 的更详细介绍，以及进一步信息的链接，请参阅维基百科上的传输层安全条目：[`en.wikipedia.org/wiki/Transport_Layer_Security`](http://en.wikipedia.org/wiki/Transport_Layer_Security)。

### StartTLS

按照通常的实现方式，SSL 要求服务器在与非加密流量不同的端口上监听加密流量。所有通过 SSL 端口的流量都被认为是 SSL 加密的流量。这意味着，任何需要同时提供明文和加密服务的服务器都必须至少在两个不同的端口上监听。

多端口的需求对一些人来说似乎是多余的、不优雅的且浪费资源。没有理由客户端不能在明文（非 SSL）连接上请求客户端和服务器之间的进一步通信加密。然后，客户端和服务器可以在同一连接上完成所有 SSL/TLS 协商，而不必切换到另一个仅支持 SSL/TLS 的端口。这个建议在 RFC2487 中被标准化为**StartTLS**。

### 提示

**选择哪个：StartTLS 还是 LDAPS？**

在 LDAP v.3 中实现 SSL/TLS 的标准方法是使用 StartTLS 方法。此方法应该尽可能实现。然而，外部因素（例如网络防火墙或不支持 StartTLS 的客户端）可能要求您使用 LDAPS 和专用的 SSL/TLS 保护端口。目前，LDAPS 支持已被列为过时，但尚未从 OpenLDAP 中移除。两种方法可以在同一服务器上同时使用。

在支持 StartTLS 的服务器中，如果客户端向服务器发送命令`STARTTLS`，则服务器将开始 TLS 加密过程。如果 TLS 协商成功，客户端和服务器将继续使用加密流量进行通信。

StartTLS 显然的优势是每个服务器只需要一个监听端口。而且，它使得客户端和服务器可以在处理不重要数据时进行明文通信，当安全性变得重要时再切换到 TLS。由于加密是资源密集型的，需要额外的处理能力来加密和解密消息，通过 StartTLS 方式简化服务可以提高性能，并释放资源用于其他任务。

然而，StartTLS 有一个缺点。由于加密流量和明文流量通过同一个端口发送，单纯通过阻止端口来防止不安全的数据传输（例如使用防火墙）在 StartTLS 中并不有效。安全措施必须能够在协议级别检查传输。

为了提高此类情况下的安全服务，OpenLDAP 提供了测试连接**安全强度因子（SSF）**的方法，以检查连接是否加密（如果是，加密方案是否足够强大）。我们将在本章后面的*使用* *安全* *强度* *因子*部分更详细地讨论 SSF。

到目前为止，你应该对 SSL 和 TLS 的工作原理有了相当清晰的理解。现在我们将进入更实际的部分。我们将创建我们自己的证书颁发机构（CA），并生成我们自己的证书，然后配置 OpenLDAP 以支持 SSL/TLS 和 StartTLS。

## 创建 SSL/TLS 证书颁发机构（CA）

为了创建证书颁发机构并生成证书，你需要安装 OpenSSL。由于许多 Ubuntu 包（包括 OpenLDAP 包）都需要 OpenSSL，它应该已经安装好了。

如果你是从源代码构建的，如附录 A 中详细描述的那样，你也可以通过 OpenSSL 库启用 SSL/TLS 支持。

### 提示

如果你已经有了证书，可以跳过本节并转到*配置* *StartTLS* 部分。OpenLDAP 使用 PEM 格式的证书。

我们需要做的第一件事是创建新的证书颁发机构（CA）。

虽然可以使用 `openssl` 命令行工具手动配置 CA，但使用随 OpenSSL 附带的 `CA.pl` Perl 脚本要简单得多。这个脚本简化了 OpenSSL 的许多配置选项，我们首先使用它来创建我们新的 CA 环境。

### 注意

Ubuntu 维护了关于以“长方式”创建新的证书颁发机构（手动创建所有文件）的文档。这份文档详细且值得阅读。虽然我会遵循那里建立的惯例，但我将使用 `CA.pl` 脚本来完成大部分繁重的工作（[`help.ubuntu.com/community/OpenSSL`](https://help.ubuntu.com/community/OpenSSL)）。

你可以将 CA 环境放在系统的任何位置。有些人喜欢将 CA 文件与其他 SSL 配置一起保存在 `/etc/ssl/` 目录下。也有一些人喜欢将证书颁发机构保存在用户目录中，以避免在系统升级时被覆盖（虽然这不太可能，但也有这种可能）。根据 Ubuntu 的建议，我将把它保存在我的用户主目录下，路径为 `/home/mbutcher/`：

```
 $ cd ~
 $ /usr/lib/ssl/misc/CA.pl -newca

```

请注意，`CA.pl` 脚本不在 `$PATH` 中，因此你需要输入脚本的完整路径。

### 提示

**查找 CA.pl**

不同的操作系统发行版会将 `CA.pl` 放在不同的位置。如果运行 `which CA.pl` 没有返回任何结果，你可能需要查阅 SSL 的手册页（`man config` 或 `man CA.pl`），或者使用 `find` 或 `slocate` 工具来查找 `CA.pl` 文件。

参数 `-newca` 指示 `CA.pl` 设置一个新的证书颁发机构环境。这将生成一个目录结构，并包含多个文件。

`CA.pl` 首先会提示你输入一个 CA 文件：

```
$ /usr/lib/ssl/misc/CA.pl -newca
CA certificate filename (or enter to create)
```

按下 *Enter* 创建一个新的 CA 证书。然后，`CA.pl` 会生成一个新的密钥，并提示你输入密码：

```
CA certificate filename (or enter to create)

Making CA certificate 
Generating a 1024 bit RSA private key
....++++++
...................................++++++
unable to write 'random state'
writing new private key to './demoCA/private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
```

一旦你输入并重新输入了密码，`CA.pl` 将收集一些关于你所在组织的信息：

```
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or 
    a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) [Internet Widgits]:Example.Com
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:Matt Butcher
Email Address []:matt@example.com 

Please enter the following 'extra' attributes
    to be sent with your certificate request
A challenge password []:mypassword
An optional company name []:Example.Com

```

`CA.pl` 会引导你完成创建主证书的过程。代码列表中高亮的行是你在交互式提示中需要提供信息的地方。设置好国家、州和城市名称后，我们将 **组织名称** 设置为 **Example.Com**。虽然我们将 **组织单位** 字段留空，但你可以利用这个字段进一步指定这个 CA 属于组织的哪个部分。

### 注意

你应该考虑在证书中使用与你在前两章设置目录信息树时使用的根 DN 相同的字段。

通常，**公共名称** 和 **电子邮件地址** 字段应包含有关组织的信息。有时，**公共名称** 用于服务器名称（如我们创建证书时）。有时，它也用于联系信息。在接下来的例子中，我们使用了我的名字和电子邮件。如果 CA 是你组织的“官方” CA，你应该将其设置为证书查询的官方联系人。

接下来，`CA.pl` 将开始生成 CA 证书的证书请求。换句话说，`CA.pl` 将创建一个新的证书，作为 CA 自己的证书。第一步是创建证书请求。我们需要为证书请求设置一个复杂的密码，也可以设置公司名称。有了上述信息，`CA.pl` 将继续生成新证书的过程：

```
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
      Serial Number:
        bf:2f:58:47:b1:6d:31:4d
      Validity
        Not Before: Oct 10 21:34:28 2006 GMT
        Not After : Oct  9 21:34:28 2009 GMT
      Subject:
        countryName               = US
        stateOrProvinceName       = Illinois
        organizationName          = Example.Com
        commonName                = Matt Butcher
        emailAddress              = matt@example.com
      X509v3 extensions:
        X509v3 Basic Constraints: 
            CA:FALSE
          Netscape Comment: 
            OpenSSL Generated Certificate
          X509v3 Subject Key Identifier: 

07:92:9B:35:CB:B7:EE:92:A8:33:61:B0:DC:F7:88:E9:4F:06:9F:7F
    X509v3 Authority Key Identifier: 

keyid:07:92:9B:35:CB:B7:EE:92:A8:33:61:B0:DC:F7:88:E9:4F:06:9F:7F

Certificate is to be certified until 
    Oct 9 21:34:28 2009 GMT (1095 days)

Write out database with 1 new entries
Data Base Updated

```

我们将被提示输入密码短语。这是我们首先创建的密码短语（当提示输入 **PEM 密码短语** 时）。如果我们正确输入密码短语，`CA.pl` 将生成新的证书，并在屏幕上显示其内容。

我们现在已经创建了一个证书颁发机构。现在，我们准备开始生成一个供 SLAPD 使用的证书。

### 注意

由于某些版本的`CA.pl`存在 bug，您可能需要`cd`进入`./demoCA`目录（`CA.pl -newca`创建的目录），并为其添加一个符号链接：`ln -s ./demoCA`。这是因为`CA.pl`有时会期望在当前目录（`./`）中找到文件，它假设该目录是`demoCA/`，有时它期望在`./demoCA`中找到文件（这当然等同于`demoCA/demoCA/`）。您也可以通过简单地编辑`/etc/ssl/openssl.cnf`文件中`[CA_default]`下的`dir=`行，并将其设置为绝对路径来解决此问题。

## 创建证书

创建证书是一个两步过程：

1.  我们需要生成证书请求。

1.  我们需要用 CA 的签名来签署请求。

让我们详细看看这些步骤。

### 创建新的证书请求

创建有效的 SSL 证书的第一步是创建证书请求。在这个过程中，我们将指定我们希望在证书上显示的信息。

有几种方式可以生成证书请求。例如，您可以使用`openssl`命令行工具并指定一系列命令行参数。但是，按照我们之前的示例，我们将使用`CA.pl`并让应用程序在需要时提示我们输入信息。

要生成新的请求，我们将运行`CA.pl -newreq`。在下一个示例中，突出显示的行是需要我们输入信息的行：

```
$ /usr/lib/ssl/misc/CA.pl -newreq
Generating a 1024 bit RSA private key
.....++++++
.....................++++++
unable to write 'random state'
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or 
    a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) [Internet Widgits]:Example.Com
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:example.com
Email Address []:matt@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

这应该看起来很熟悉。在大多数方面，它与生成证书颁发机构的过程相似。

首先，我们将被提示输入一个密码短语。我们将在稍后使用这个密码短语。

接下来，我们需要提供有关该证书所代表的组织的信息。和之前一样，字段包括国家名、州/省名、城市、组织名、组织单位、联系人常用名和联系人的电子邮件地址。同样，像之前一样，我们输入了 Example.Com 的相关信息。

然而，这次我们将“常用名称”字段设置为证书所对应的服务器域名——`example.com`。正确使用服务器的域名非常重要。在证书协商过程中，客户端会检查“常用名称”字段，看它是否与服务器的域名匹配。如果域名不匹配，用户可能会收到错误信息，或者客户端应用程序可能会直接终止连接。

额外的*密码*和*可选* *公司* *名称*有时会在证书请求过程中使用。由于我们自己在请求和签署过程中进行操作，因此不需要填写这些字段。

现在我们应该在 CA 目录中有两个文件：

+   第一步是创建名为`newreq.pem`的文件，里面包含我们证书请求的 base-64 编码表示。

+   第二步是创建名为`newkey.pem`的文件，里面包含 base-64 编码的私钥。

我们现在可以继续进行第二步了。

### 签署证书请求

证书请求包含了证书所需的所有信息，但仍然缺少 CA 的数字签名。那么，下一步就是使用我们之前创建的 CA 来签署这个新证书。为此，我们将运行 `CA.pl -signreq`：

```
$ /usr/lib/ssl/misc/CA.pl -signreq
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
      Serial Number:
        ba:49:df:f5:8e:7e:77:c2
      Validity
        Not Before: Oct 12 21:23:49 2006 GMT
        Not After : Oct 12 21:23:49 2007 GMT
      Subject:
        countryName               = US
        stateOrProvinceName       = Illinois
        localityName              = Chicago
        organizationName          = Example.Com
        commonName                = example.com
        emailAddress              = matt@example.com
      X509v3 extensions:
        X509v3 Basic Constraints: 
          CA:FALSE
        Netscape Comment: 
          OpenSSL Generated Certificate
        X509v3 Subject Key Identifier: 

47:DD:90:8F:79:90:2E:C0:CC:B3:95:62:35:C4:D8:6C:5D:A2:EE:88
     X509v3 Authority Key Identifier: 
                keyid:6B:FB:66:33:5D:DB:CC:40:42:D7:71:F7:F0:D0:7C:94:3E:8F:CD:58

Certificate is to be certified until 
    Oct 12 21:23:49 2007 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
Signed certificate is in newcert.pem

```

`CA.pl -signcert` 命令会查找 `newreq.pem` 文件，然后开始签名过程。首先，我们需要输入 CA 的密码短语。如果密码正确，`CA.pl` 会显示 `newreq.pem` 中的证书，并询问是否要签署它。最后，它会要求我们提交这些更改。

一旦更改被提交，将会创建一个名为 `newcert.pem` 的新文件。

我们现在有两个重要的文件：

+   `newkey.pem`，它包含私钥。

+   `newcert.pem`，它包含签名后的证书。

我们只需要处理几个小问题，然后就可以继续配置 SLAPD 以使用 SSL/TLS。

### 配置和安装证书

我们只需要完成剩下的三个步骤。第一个步骤与我们在证书中设置的密码短语有关。

#### 移除密钥的密码短语

在这里需要非常小心！在生成证书请求时，我们为证书设置了一个密码短语。这使得 `newkey.pem` 文件使用密码短语进行了加密。

如果你使用的是带有密码短语加密的密钥文件，那么每次使用这个证书时，你都必须输入密码。这意味着，在我们的例子中，每次启动 OpenLDAP 时，我们都需要输入密码短语。除非我们有严格的安全要求（并且愿意忍受每次启动或重启服务器时输入密码短语的麻烦），否则我们可能不希望密钥文件被加密。

因此，我们需要使用 `openssl` 命令创建一个未加密版本的密钥文件：

```
 $ openssl rsa < newkey.pem > clearkey.pem

```

这就是我们得到的结果：

```
Enter pass phrase:
writing RSA key
```

在这个例子中，命令 `openssl rsa` 执行了 OpenSSL RSA 工具，它将解密密钥。通过 `< newkey.pem`，我们将 `newkey.pem` 文件的内容传递给 `openssl` 进行解密。然后，通过 `> clearkey.pem`，我们指示 `openssl` 将明文密钥文件写入 `clearkey.pem` 文件。为了完成此操作，`openssl` 会提示输入密码短语。现在，`clearkey.pem` 文件包含了我们证书的未加密私钥。

### 注意

`clearkey.pem` 文件现在包含了一个未加密的私钥。此文件应该受到保护，以防止滥用。你应该为此文件设置严格的权限，以确保系统中的其他用户无法访问它。

### 提示

**OpenSSL 程序**

`openssl` 程序执行了几十个与 SSL 相关的功能，从生成证书到模拟基于网络的 SSL 客户端。其语法通常比较复杂。这也是为什么我们使用 `CA.pl` 包装脚本来执行常见任务的原因。但有些任务只能通过 `openssl` 命令来完成。如果需要，`openssl` 有非常出色的手册页面：`man openssl`。

#### 移动证书

第二个任务是将我们的新证书和密钥移动到服务器上的一个有用位置，并给 PEM 文件命名。如果这个证书需要被多个不同的服务使用，可能可以考虑将其放在共享目录中。但在我们的情况下，我们只会将 SSL 证书用于 LDAP，所以我们可以将文件放在`/etc/ldap/`（如果您是从源代码构建的，可以放在`/usr/local/etc/openldap/`）。

我们关注的两个文件是`newcert.pem`和`clearkey.pem`。我们需要重命名并移动这两个密钥：

```
 $ sudo mv cacert.pem /etc/ldap/example.com.cert.pem
 $ sudo mv clearkey.pem /etc/ldap/example.com.key.pem

```

现在，我们需要为证书文件设置权限和所有权。由于我们没有给密钥添加密码短语，我们还应确保只有 OpenLDAP 用户能够读取该密钥文件：

```
 $ sudo chown root:root /etc/ldap/example.com.*.pem
 $ sudo chmod 400 /etc/ldap/example.com.key.pem

```

第一行将两个 PEM 文件的所有者和组更改为`root`用户和`root`组。第二行设置权限，使得只有文件所有者能够读取该文件，其他人无法访问。

如果您以非 root 用户运行 OpenLDAP（这样做是个好主意），那么这些文件应该由该用户而非 root 用户拥有；例如`chown oenldap example.com.*.pem`。

#### 安装 CA 证书

第三个任务是安装 CA 的公用证书，以便系统上的其他应用程序可以使用该证书来验证我们刚生成的证书的真实性。首先，我们需要将 CA 证书复制到 Ubuntu 本地的证书数据库。在此过程中，我们将为它提供一个用户友好的名称：

```
 $ sudo cp cacert.pem /usr/share/ca-certificates/Example.Com-CA.crt

```

然后，编辑`/etc/ca-certificates.conf`文件，并在文件末尾添加`Example.Com.crt`。

最后，运行`update-ca-certificates`：

```
$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs....done.
```

CA 证书现在已经安装。`/etc/ssl/certs`目录现在是 CA 证书的权威来源。

### 注意

除了 Ubuntu 和 Debian 之外的 UNIX 和 Linux 系统可能没有`update-ca-certificates`脚本。请查阅系统文档，了解如何在这些系统中更新证书数据库。

#### 可选：清理

如果需要，您可以在 CA 目录中做一些清理。删除加密的密钥文件和证书请求文件，这两个文件都位于`demoCA/`目录中：

```
 $ rm newkey.pem newreq.pem

```

此外，确保`clearkey.pem`不再出现在`demoCA/`目录中。

现在我们准备好配置 OpenLDAP 以使用我们的新证书了。首先，我们将配置 StartTLS 支持，这是最简单的，然后我们将在 LDAPS 端口 636 上配置 SSL/TLS 支持。

## 配置 StartTLS

在前面的部分中，我们创建了新证书和密钥，并将这两个文件放置在`/etc/ldap`目录中。在本节中，我们将设置 StartTLS（我们在本章的 StartTLS 部分已经介绍过）。配置 StartTLS 只需要在`slapd.conf`文件中增加几行。

再次强调，StartTLS 是为 OpenLDAP 提供 SSL/TLS 安全性的一种标准方式（根据 RFC 4511）。出于安全原因，应在实际操作中提供对 StartTLS 的支持。

在 `slapd.conf` 文件中，在 `BDB 数据库配置` 部分之前，插入 SSL/TLS 选项：

```
###########
# SSL/TLS #
###########
TLSCACertificatePath    /etc/ssl/certs/
TLSCertificateFile      /etc/ldap/example.com.cert.pem
TLSCertificateKeyFile   /etc/ldap/example.com.key.pem
```

基本上，我们只需要指定三个指令就可以使 StartTLS 工作：

+   第一个指令 `TLSCACertificatePath` 告诉 SLAPD 在哪里找到所有它需要用于验证证书的 CA 证书。绝对位置是，正如我们之前看到的，`/etc/ssl/certs/` 目录。

+   第二个指令 `TLSCertificateFile` 指定了已签名证书的位置。

+   第三个指令 `TLSCertificateKeyFile` 指定了相应密钥文件的位置，该文件具有证书的私有加密密钥。

### 注意

还有一些其他特定于 TLS 的指令，允许您对 TLS 连接提供详细的约束（例如可以使用哪些密码套件，以及是否需要客户端向服务器提供证书）。关于这些的完整文档可以在 `slapd.conf` 手册的 TLS 部分找到：`man slapd.conf`。

这就是我们使 SLAPD 执行 StartTLS 所需的所有内容。重新启动 SLAPD 以使更改生效。

## 配置客户端 TLS

我们确实需要在 `ldap.conf` 中添加一两个指令——这是 OpenLDAP 客户端使用的配置文件。与 SLAPD 类似，我们需要将客户端指向新的 CA 证书的正确位置，以便它们可以验证服务器证书。

在 `ldap.conf` 文件的底部，我们可以添加适当的指令：

```
TLS_CACERTDIR /etc/ssl/certs
```

客户端将使用此指令来定位 CA 证书，以便检查从服务器获取的证书的数字签名。如果您知道您只会使用由特定 CA 签名的证书，可以使用 `TLS_CACERT` 指令指向特定的 CA 证书文件，而不是包含一个或多个证书的目录。

默认情况下，OpenLDAP 客户端始终对数字签名进行检查。如果服务器发送的证书由 `/etc/ssl/certs/`（或 `TLS_CACERTDIR` 指向的任何目录）之外的 CA 签名，那么客户端将关闭连接并在屏幕上打印错误消息。

但有时，即使无法验证服务器的身份，也值得获得 TLS 的加密支持，因此正确的 CA 证书不可用。

在这种情况下，您可能需要更改 OpenLDAP 客户端执行身份验证检查的方式。例如，可能希望尝试验证证书，但即使没有适当的本地 CA，也希望继续连接。为此，在 `slapd.conf` 中使用以下指令：

```
TLS_REQCERT allow
```

在这种情况下，如果没有 CA 证书或者发送的证书无法验证，会话将继续，而不会显示错误消息。`TLS_REQCERT` 有几个不同的检查级别，从 `strict`（始终验证）到 `never`（甚至不尝试验证证书）不等。

此时，我们可以使用 `ldapsearch` 来测试连接。要指示客户端使用 StartTLS，我们需要使用 `-Z` 标志。但是，如果仅指定 `-Z`，当客户端与服务器的 TLS 协商失败时，它将继续以明文进行事务。换句话说，使用 `-Z` 时，TLS 是优先的，但不是强制的。要使 TLS 成为必需，我们将向标志中添加一个额外的 z，使其变为 `-ZZ`：

```
 $ ldapsearch -LLL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
 ldap://example.com -ZZ '(uid=manny)'

```

这应该会提示输入密码，然后返回一个结果：

```
Enter LDAP Password: 
dn: uid=manny,ou=Users,dc=example,dc=com
sn: Kant
uid: immanuel
uid: manny
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
givenName: Manny
cn: Manny Kant
```

如果结果返回如下，则表示 TLS 配置成功。但由于 TLS 在设计上是严格的，配置可能会很困难。配置中的小错误（例如使用与证书中的 CN 字段不同的域名）可能会导致 TLS 无法正常工作。考虑以下示例：

```
$ ldapsearch -LL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
    ldap://localhost -ZZ '(uid=manny)'

ldap_start_tls: Connect error (-11)
    additional info: TLS: hostname does not match CN in peer 
    certificate

```

在这种情况下，命令行中指定的主机名（`localhost`）与证书中的 CN 字段（`example.com`）中的主机名不同。尽管在此情况下，这两个域名托管在同一系统上，TLS 仍然不接受不匹配。

TLS 中的其他常见错误包括：

+   反转 `TLSCertificateFile` 和 `TLSCertificateKeyFile` 指令的值

+   忘记安装 CA 证书（导致出现错误，表示无法验证服务器证书）

+   忘记在 `ldap.conf` 中正确设置客户端 CA 路径

+   在密钥文件（或证书文件）上设置读/写权限（或所有权），使得 SLAPD 服务器无法读取它

虽然 OpenLDAP 在许多方面可以宽容，但 TLS 配置并不是其中之一。在配置 TLS 和 SSL 时需要特别小心。

## 配置 LDAPS

现在我们已经配置了 TLS，接下来只需执行几个额外的步骤，以便在其专用端口上启用 SSL/TLS。运行专用的 TLS/SSL 保护的 LDAP 流量的传统端口是 636，LDAPS 端口。

大多数时候，使用 StartTLS 更好。然而，网络因素（如不支持 StartTLS 的客户端或要求强制阻塞允许非加密文本的端口的政策）可能需要使用 LDAPS。

请记住，LDAPS 和 StartTLS 都可以用于同一服务器。SLAPD 可以在专用端口上接受 LDAPS 流量，并继续在 LDAP 端口提供 StartTLS 功能。

### 注意

与 StartTLS 配置类似，此配置要求 `slapd.conf` 文件中设置 `TLSCertificateFile`、`TLSCertificateKeyFile` 和 `TLSCACertificateDir` 指令。

要使 SLAPD 监听此端口，需要在启动 `slapd` 时传递一个额外的参数。在 Ubuntu 中，与其他基于 Debian 的发行版一样，配置参数可以在 `/etc/defaults/slapd` 文件中设置。在该文件中，我们只需要设置 `SLAPD_SERVICES`。当启动脚本执行时，SLAPD 会启动此处列出的所有服务。

```
SLAPD_SERVICES="ldap:/// ldaps:///"
```

给定的代码指示 SLAPD 在所有可用的 IP 地址上监听默认的 LDAP（端口 389）和默认的 LDAPS（端口 636）。如果我们希望 SLAPD 只在一个地址上监听 LDAP 流量，但在所有地址上监听 LDAPS 流量，可以将上面的配置替换为：

```
SLAPD_SERVICES="ldap://127.0.0.1/ ldaps:///"
```

在这里，`ldap://127.0.0.1/` 告诉 SLAPD 仅在回环地址上监听 LDAP 流量，而 `ldaps:///` 则表示 SLAPD 应该在为该主机配置的所有 IP 地址上监听 LDAPS 流量。你需要重启 SLAPD，才能使这些更改生效。

类似地，如果你是从源代码构建并且想直接启动 `slapd`，`-h` 命令行参数可以让你指定启动哪些服务：

```
/usr/local/libexec/slapd -h "ldap:/// ldaps:///"
```

配置 LDAPS 就这么简单。我们现在可以用 `ldapsearch` 来测试它：

```
 ldapsearch -LL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
 ldaps://example.com '(uid=manny)'

```

这个 `ldapsearch` 和我们在测试 StartTLS 时使用的有两个关键区别：

+   `-H` 参数后指定的 URL 协议是 `ldaps://`，而不是 `ldap://`。

+   这里没有 `-Z` 或 `-ZZ` 参数。这些参数告诉客户端发送 StartTLS 命令，而 SSL/TLS 通过专用端口时，不会识别 StartTLS 命令。

如果在进行给定的搜索时遇到错误，但 StartTLS 工作正常，首先要检查的是防火墙设置。通常，防火墙会允许通过 389 端口的流量，但会阻塞 636 端口。还可以确保服务器确实在监听 636 端口。你可以通过命令行使用 `netstat –tcp -l` 来检查，这会显示正在使用的端口列表。如果 LDAPS（636）没有出现，检查 `/etc/defaults/slapd` 配置，确保 `SLAPD_SERVICES` 指令设置正确。

### 使用 OpenSSL 客户端调试

在某些情况下，能够通过 LDAPS 连接到 SLAPD 并观察证书处理过程是有用的。`openssl` 程序可以通过其内置的 `s_client` 客户端应用程序实现这一点：

```
 $ openssl s_client -connect example.com:636

```

`-connect` 参数接受主机名，后面跟一个冒号和端口号。当运行此命令时，`openssl` 将通过 SSL 连接到远程服务器，并执行证书协商。整个协商过程将显示在屏幕上。如果证书协商成功，`openssl` 将保持连接打开，你可以在命令行输入原始协议命令。要退出，只需按 *CTRL*+*C*。

现在我们已经使 StartTLS 和 TLS/SSL 都能正常工作。我们在这一节中只剩下一个简短的内容要讲解，之后我们将进入认证部分。

## 使用安全强度因子

运行 StartTLS 有其优势。它更容易配置，在许多方面更容易调试，且复杂的事务可以根据需要在明文和加密之间切换。

但有一个明显的缺点：当所有明文流量通过一个端口而所有加密流量通过另一个端口时，我们可以使用标准防火墙来阻止未加密的流量。但当两者都通过同一端口时，许多防火墙无法验证流量是否安全。

但 OpenLDAP 确实提供了一些工具来在 SLAPD 中实现这种安全性，而不是在防火墙中实现。

OpenLDAP 可以检查连接的完整性和加密状态，并根据这些特性为该连接分配一个**安全强度因子（SSF）**。SSF 是用于表示保护措施强度的数字表示。

大多数 SSF 数字只是反映了加密算法的密钥长度。例如，由于**DES**的最大密钥长度为 56，当使用 DES 保护连接时，SSF 为 56。**Triple-DES (3DES)**，这是 Ubuntu 的 OpenSSL 配置中默认使用的加密算法，密钥长度为 112。因此，它的 SSF 也是 112。**AES**加密算法既强大又能快速计算，可以使用不同的密钥大小。AES-128 使用 128 位密钥，而 AES-256 使用 256 位密钥。因此，AES 的 SSF 将反映密钥大小。

有两个特殊的 SSF 数字：0 和 1。SSF 为 0 表示（正如预期的那样）没有实施任何安全措施。SSF 为 1 表示仅对连接进行完整性检查。

OpenLDAP 可以使用 SSF 信息来确定客户端是否可以连接到目录。SSF 信息还可以在 ACL 和 SASL 配置中使用，有效地允许构建复杂的规则，以确定客户端连接必须满足哪些条件，才能在目录上执行某些操作。

我们将在本章后面讨论 SASL 身份验证和 ACL，但现在我们将看看如何在`slapd.conf`中的`security`指令中使用 SSF，作为指定连接必须多安全才能访问数据库的一种方式。

### 安全性指令

`security`指令可以在`slapd.conf`中以两种不同的方式使用。如果它放在文件的顶部，在任何后端数据库定义之前，则它被放置在*全局*上下文中，并将应用于所有连接。另一方面，如果`security`指令放在某个后端定义内，则它只会应用于该特定数据库。例如，考虑一个有两个后端的情况：

```
include /etc/ldap/schema/core.schema

modulepath /usr/local/libexec/openldap
moduleload back_hdb
# Other configuration directives ...

# DB 1:
database hdb
suffix "ou=Users,dc=example,dc=com"
# More directives for DB 1...
# DB 2:
database bdb
suffix "ou=System,dc=example,dc=com"
# More directives for DB 2...
```

这个`slapd.conf`文件的部分示例定义了两个目录后端。现在，如果`security`指令在第一个数据库定义之前使用（即在`database hdb`那行之前），则它将全局应用于所有连接。

但是，如果我们想允许对数据库 2 的未加密连接，但只允许对数据库 1（它包含所有用户条目）进行加密连接，我们可以使用不同的`security`指令：

```
include /etc/ldap/schema/core.schema

modulepath /usr/local/libexec/openldap
moduleload back_hdb
loglevel stats
# Other configuration directives ...

# DB 1:
database hdb
suffix "ou=Users,dc=example,dc=com"
security ssf=112
# More directives for DB 1...

# DB 2:
database bdb
suffix "ou=System,dc=example,dc=com"
security ssf=0
# More directives for DB 2...
```

请注意两个突出显示的行的添加——两个单独的 `security` 指令，每个后端数据库一个。

现在，重新启动目录（请注意 `loglevel` 设置为 `stats`），我们可以使用 `ldapsearch` 测试安全参数。首先，我们将尝试使用非 TLS 连接搜索 `Users` OU：

```
 $ ldapsearch -x -W -D 'uid=matt,ou=Users,dc=example,dc=com' -b \ 
 'ou=Users,dc=example,dc=com' '(uid=david)' uid

```

在日志中，我们看到如下条目：

```
conn=0 fd=12 ACCEPT from IP=127.0.0.1:48758 (IP=0.0.0.0:389)
conn=0 op=0 BIND dn="uid=matt,ou=Users,dc=example,dc=com" method=128
conn=0 op=0 RESULT tag=97 err=13 text=confidentiality required
conn=0 fd=12 closed (connection lost)
connection_read(12): no connection!
```

第三行指示服务器返回错误编号 13：`confidentiality required`。这是因为我们没有采取任何措施来保护连接。使用简单的认证（未加密）并未使用 TLS/SSL 导致客户端连接的有效 SSF 为 0。

接下来，让我们使用启用 TLS 的相同搜索：

```
 $ ldapsearch -x -W -D 'uid=matt,ou=Users,dc=example,dc=com' -b \ 
 'ou=Users,dc=example,dc=com' -Z '(uid=david)' uid

```

请注意，在此示例中，包含 `-Z` 标志以发送 StartTLS 命令。现在，服务器日志显示：

```
conn=1 fd=12 ACCEPT from IP=127.0.0.1:44684 (IP=0.0.0.0:389)
conn=1 op=0 STARTTLS
conn=1 op=0 RESULT oid= err=0 text=
conn=1 fd=12 TLS established tls_ssf=256 ssf=256
conn=1 op=1 BIND dn="uid=matt,ou=Users,dc=example,dc=com" method=128
conn=1 op=1 BIND dn="uid=matt,ou=Users,dc=example,dc=com" mech=SIMPLE ssf=0
conn=1 op=1 RESULT tag=97 err=0 text=
```

关于此结果有几点需要注意。在第二行，OpenLDAP 报告正在执行 StartTLS。两行后，它报告：`TLS established tls_ssf=256 ssf=256`。此行指示 TLS 连接的 SSF 为 256（因为连接正在使用 AES-256），连接的总 SSF 为 256。

如果您向下看几行，可以看到开始 `BIND` 的第二行，您会注意到另一个报告的 SSF：`ssf=0`。为什么呢？

OpenLDAP 在连接的各个方面上测量 SSF。首先，正如我们上面所看到的，它检查网络连接的 SSF。基于它们的密码强度，TLS/SSL 连接被分配一个 SSF。

但在客户端对目录进行身份验证的绑定阶段期间，OpenLDAP 也测量认证机制的 SSF。简单（`mech=SIMPLE`）认证机制不会加密密码，因此始终给予 SSF 为 0。

然而，连接的总 SSF 仍为 256，其中 TLS SSF 为 256，SASL SSF 为 0。

#### 一个精细化的安全指令

到目前为止，我们查看的 `security` 指令很基础。它只要求总 SSF 至少为 112（3DES 加密），但我们可以使其更具体。

例如，我们可以简单地要求任何 TLS 连接至少有一个 128 位的密钥：

```
security tls=128
```

这将要求所有传入的连接使用具有强大（128 位或更高）的 TLS 密码。

### 注意

在某些情况下，定义将使用哪些 TLS/SSL 密码或密码族是可取的。这不能通过 `security` 指令完成。相反，您将需要使用 `TLSCipherSuite` 指令，允许您为 TLS/SSL 连接指定可接受的密码详细规范。

或者，如果我们只想为试图执行简单绑定（而不是 SASL 绑定）的连接定义一个强 SSF，那么我们可以为简单绑定指定一个 SSF：

```
security simple_bind=128
```

这将要求使用一些强大的 TLS 密码来保护认证信息。

### 提示

如果你计划允许简单绑定，并且你正在一个不安全的网络上运行，强烈建议你配置 TLS/SSL 并在绑定操作期间通过 `security` 指令要求 TLS 加密。

你还可以在 `security` 指令中使用 `update_ssf` 关键字来设置更新操作所需的 SSF。因此，你可以指定对于读取目录只需要低级加密，但在执行目录信息更新时必须使用高级加密：

```
security ssf=56 update_ssf=256
```

在接下来的章节中，我们将讨论 SASL 配置。你也可以使用 `security` 指令通过 `sasl=` 和 `update_sasl=` 关键字来为 SASL 绑定设置 SSF。

最后，在极少数情况下，当 OpenLDAP 监听本地套接字（即 `ldapi://`）时，你可以使用 `security transport=112`（或任何你需要的加密强度）来确保通过该套接字传输的流量是加密的。

到此为止，我们已完成对 SSL 和 TLS 的讨论。接下来，我们将继续研究安全性三大方面中的第二个：认证。

# 将用户认证到目录

正如我们在本书前面看到的，OpenLDAP 支持两种不同的绑定（或认证）方式。第一种是使用简单绑定。第二种是使用 SASL 绑定。在本部分中，我们将分别介绍这两种认证方法。

并不需要选择其中一个。OpenLDAP 可以配置同时支持这两种方式，这时由客户端决定使用哪种方法。简单绑定更容易配置（需要的配置非常少）。但 SASL 更安全且更灵活，尽管这些优点伴随着额外的复杂性。

绑定操作和认证过程的基础知识已在第三章的早期部分介绍。虽然我们将在这里回顾一些相关内容，但你可能会觉得回头看看该部分的内容很有帮助。

## 简单绑定

我们将首先看看的认证形式是简单绑定。从用户的角度来看，它不一定简单，但它肯定更容易配置，而且绑定过程对服务器来说也更简单，因为需要的处理较少。

要执行简单绑定，服务器需要两个信息：一个 DN 和一个密码。如果 DN 和密码字段都为空，则服务器会尝试以匿名用户身份绑定。

在简单绑定过程中，客户端连接到服务器并将 DN 和密码信息发送给服务器，而不会添加任何额外的安全措施。例如，密码并没有特别加密。

如果客户端通过 TLS/SSL 进行通信，那么整个事务将被加密，密码也会因此得到保护。如果客户端没有使用 TLS/SSL，则密码将以明文形式通过网络发送。这当然是一个安全问题，应该避免（可以通过使用上一节中讨论的 `security` 指令，或者使用 SASL 绑定代替简单绑定）。

客户端应用程序执行简单绑定有两种常见方法。第一种方法有时被称为 **快速绑定**。在快速绑定中，客户端提供完整的 DN（`uid=matt,ou=users,dc=example,dc=com`）以及密码（`myPassword`）。它比常见的替代方法（以匿名身份绑定并搜索所需的 DN）更快。

### 注释

**Cyrus SASLAuthd** 提供给其他应用程序 SASL 认证服务，它是第一个使用“快速绑定”术语的应用程序。SASLAuthd 是一个为应用程序提供 SASL 认证服务的有用工具。我们将在下一节中再次讨论它。在 OpenLDAP 文档中，根本没有使用“快速绑定”这一术语。

目录首先作为匿名用户，对客户端提供的 DN 的 `userPassword` 属性执行 **auth** 访问。在 auth 访问中，服务器会将提供的密码值与存储在目录中的 `userPassword` 值进行比较。如果 `userPassword` 值已加密（例如，使用 SSHA 或 SMD5），那么 SLAPD 会将用户提供的密码进行哈希处理，然后比较哈希值。如果值匹配，OpenLDAP 将绑定用户并允许其执行其他 LDAP 操作。

### 注释

当使用 `-x` 选项时，OpenLDAP 命令行客户端执行简单绑定。客户端要求你指定完整的用户 DN 和密码，然后它们会执行快速绑定。

这就是快速绑定。但是，还有第二种常见的简单绑定方法——一种旨在消除用户需要知道完整 DN 的要求的方法。

在第二种方法中（顺便说一下，这并不叫“慢绑定”），客户端应用程序要求用户只知道某个特定的唯一标识符——通常是 `uid` 或 `cn` 的值。客户端应用程序然后作为匿名用户（或另一个预配置的用户）绑定到服务器，并执行搜索，寻找包含匹配属性值的 DN。如果找到一个（且只有一个）匹配的 DN，它就会重新绑定，使用检索到的 DN 和用户提供的密码。

通常，使用简单绑定的客户端应用程序需要一个基础 DN。执行简单绑定的第二种方法需要一个附加信息，这在快速绑定中是不需要的：一个搜索过滤器。过滤器通常像这样 `(&(uid=?)(objectclass=inetOrgPerson))`，其中问号（`?`）由用户提供的值替换。

### 使用认证用户进行简单绑定

虽然当只需要用户 ID 或 CN 时用户更为便捷，但我们所看到的第二种方法可能会引发一个额外的担忧：为了执行搜索，匿名用户必须具有*读取*目录中所有用户记录的权限。这意味着任何人都可以连接到目录（记住，匿名用户没有密码）并执行搜索。

在许多情况下，这并不是问题。允许某人查看目录中所有用户的列表可能根本不会构成安全隐患。但在其他情况下，这种访问是不可接受的。

解决这个问题的一种方法是使用不同的用户（而不是匿名用户）来执行用户 DN 的查找。在上一章中，我们创建了这样一个账户。以下是我们使用的 LDIF 记录：

```
# Special Account for Authentication:
dn: uid=authenticate,ou=System,dc=example,dc=com
uid: authenticate
ou: System
description: Special account for authenticating users
userPassword: secret
objectClass: account
objectClass: simpleSecurityObject
```

这个账户的目的是登录到服务器并执行 DN 查找。换句话说，它执行与匿名用户相同的工作，但它增加了一些安全性，因为使用`uid=authenticate`账户的客户端也必须具有相应的密码。

为了明确这一点，我们来看一个案例：一个配置为使用 Authenticate 账户的客户端，将一个自我标识为`matt`，密码为`myPassword`的用户进行绑定。

下面是逐步分析当以这种方式执行绑定操作时发生的情况：

1.  客户端连接到服务器，并开始使用 DN `uid=authenticate,ou=system,dc=example,dc=com`和密码`secret`执行绑定操作。

1.  服务器作为匿名用户，将 Authenticate 密码`secret`与`uid=authenticate,ou=system,dc=example,dc=com`记录中的`userPassword`属性值进行比较。

1.  如果上述步骤成功，那么客户端（现在已登录为 Authenticate 用户）将使用过滤器`(&(uid=matt)(objectclass=inetOrgPerson))`执行搜索。由于`uid`是唯一的，搜索应该返回 0 或 1 条记录。

1.  如果找到匹配的 DN（在我们这个例子中是`uid=matt,ou=user,dc=example,dc=com`），那么客户端将尝试以该 DN 重新绑定，并使用用户最初提供给客户端的密码（`myPassword`）。

1.  服务器作为匿名用户，将用户提供的密码`myPassword`与`uid=matt,ou=user,dc=example,dc=com`的`userPassword`属性值进行比较。

1.  如果密码比较成功，那么客户端应用程序可以继续以`uid=matt,ou=user,dc=example,dc=com`身份执行 LDAP 操作。

这个过程较为繁琐，并且要求客户端应用程序配置绑定 DN 和 Authenticate 用户的密码信息，但它为匿名绑定和搜索增加了额外的安全层。

在本节中，我们探讨了三种执行简单绑定的不同方式。这些方法在特定情况下各有其用，且当与 SSL/TLS 一起使用时，简单绑定在密码通过网络传输时不会构成显著的安全威胁。

### 提示

**slapd.conf 中的简单绑定指令**

在 `slapd.conf` 中只有少数指令与简单绑定相关。默认情况下允许简单绑定。为了防止 SLAPD 接受简单绑定操作，你可以使用 `require SASL` 指令，这将要求所有绑定操作都为 SASL 绑定操作。此外，`security` 指令提供了 `simple_bind=` SSF 检查，可用于要求对简单绑定操作设置最小的 SSF。这在 *安全* *指令* 部分有更详细的说明。

本书后续章节将介绍几个使用简单绑定连接到目录的第三方应用程序。

但有时我们需要更安全的认证过程，或者简单绑定的绑定-查询-重新绑定方法对客户端来说过于复杂。在这种情况下，使用 SASL 绑定可能更好。

## SASL 绑定

SASL 提供了第二种认证 OpenLDAP 目录的方法。SASL 通过用更强大的认证过程取代上述简单绑定方法来工作。

### 注意

SASL 标准在 RFC 2222 中定义（[`www.rfc-editor.org/rfc/rfc2222.txt`](http://www.rfc-editor.org/rfc/rfc2222.txt)）。

SASL 支持多种不同的底层认证机制，从登录/密码组合到更复杂的配置，如**一次性密码（OTP）**，甚至**Kerberos**票证认证。

虽然 SASL 提供了几十种不同的配置选项，但我们只介绍其中的一种。我们将配置 SASL 来进行 **DIGEST-MD5** 认证。它的设置稍微比某些 SASL 机制复杂，但不需要像 **GSSAPI** 或 Kerberos 那样详细的配置。

本章后面我们将把 SASL 工作与 SSL/TLS 工作结合起来，并使用**SASL EXTERNAL 机制**通过客户端 SSL 证书进行目录认证。

### 注意

Cyrus SASL 文档（位于 `/usr/share/doc/libsasl2` 或在线查看 [`asg.web.cmu.edu/sasl/`](http://asg.web.cmu.edu/sasl/)）提供了实现其他机制的信息。

在 DIGEST-MD5 认证中，用户的密码将由 SASL 客户端加密，只有加密后的密码会通过网络传输，然后由服务器解密并与明文密码进行比较。

使用 DIGEST-MD5 的优点是密码在网络上传输时得到保护。然而，缺点是密码必须以明文形式存储在服务器上。

与简单绑定的工作方式对比。在简单绑定中，密码本身在网络上传输时并未加密，但存储在数据库中的密码副本是以加密格式存储的（除非你对 OpenLDAP 进行了其他配置）。

请记住，当使用 SSL/TLS 时，所有通过连接传输的数据都将被加密，包括密码。

配置 SASL 比配置简单的绑定操作要复杂。配置 SASL 支持有两个部分：

+   Cyrus SASL 配置

+   配置 OpenLDAP

### 配置 Cyrus SASL

当我们在第二章安装 OpenLDAP 时，我们安装的其中一个软件包是 Cyrus SASL（该库名为 `libsasl2`）。我们还需要 SASL 命令行工具，这些工具包含在 `sasl2-bin` 软件包中：

```
 $ sudo apt-get install sasl2-bin

```

本软件包中包含了 `saslpasswd2` 程序以及 SASL 测试客户端和服务器应用程序。

现在我们准备开始配置了。

#### SASL 配置文件

SASL 库可以被多个应用程序使用，每个应用程序都可以拥有自己的 SASL 配置文件。SASL 配置文件存储在 `/usr/lib/sasl2` 目录中。在该目录下，我们将为 OpenLDAP 创建一个配置文件。文件 `slapd.conf` 看起来是这样的：

```
# SASL Configuration
pwcheck_method: auxprop
sasldb_path: /etc/sasldb2
```

### 注意

不要将位于 `/usr/lib/sasl2` 的 `slapd.conf` 与位于 `/etc/ldap/` 的主 `slapd.conf` 文件混淆。这是两个不同的文件。

和往常一样，所有以 `#` 开头的行都是注释。第二行决定了 SASL 如何检查密码。例如，SASL 配备了一个独立的服务器 **saslauthd**，它将处理密码检查。然而，在我们的情况下，我们希望使用 `auxprop` 插件，它自己进行密码检查，而不是查询 `saslauthd` 服务器。

最后一行告诉 SASL 密码数据库的位置（该数据库存储所有密码的明文版本）。此数据库的标准位置是 `/etc/sasldb2`。

#### 设置用户密码

在开始时，我们将把 SASL 密码存储在 `/etc/sasldb2` 数据库中。要向数据库添加密码，我们使用 `saslpasswd2` 程序：

```
 $ sudo saslpasswd2 -c -u example.com matt 

```

请注意，我们必须使用 `sudo` 来运行上述命令，因为密码文件属于 root 用户。`sudo` 和 `saslpasswd2` 都会提示你输入密码。

`saslpasswd2` 的 `-c` 参数表示如果用户 ID 尚未存在，则希望创建该用户 ID。`-u example.com` 设置 **SASL 域**。SASL 使用域作为划分认证名称空间的一种方式。客户端应用程序通常会向 SASL 提供三项信息：用户名、密码和域。默认情况下，客户端将发送它们的域名作为域。

通过使用域，可以为不同的应用程序或应用程序上下文提供相同用户名的不同密码。例如，`example.com` 域中的 `matt` 可以有一个密码，而 `testing.example.com` 域中的 `matt` 可以有不同的密码。

对于我们的目的，我们只需要一个域，并将其命名为 `example.com`。运行给定的命令时，它将提示输入用户 `matt` 的密码，然后提示输入密码确认。如果密码匹配，它将把密码以明文形式存储在 SASL 密码数据库中。

现在我们准备配置 OpenLDAP。

### 为 SASL 支持配置 SLAPD

OpenLDAP 的 SASL 配置在服务器的 `slapd.conf` 文件中完成，在客户端的 `ldap.conf` 文件中完成。 本节中，我们将重点关注 SLAPD 服务器。

当 OpenLDAP 接收到 SASL 认证请求时，它会从客户端接收四个信息字段。这四个信息字段是：

+   用户名：此字段包含用户在认证时提供的 ID。

+   领域：此字段包含用户进行身份验证时使用的 SASL 领域。

+   SASL 机制：此字段指示使用了哪种认证系统（机制）。根据我们的 SASL 配置，应该是 DIGEST-MD5。

+   认证信息：此字段始终设置为 `auth`，表示用户需要认证。

所有这些信息都被压缩成一个类似 DN 的字符串，看起来像这样：

```
uid=matt,cn=example.com,cn=DIGEST-MD5,cn=auth
```

上面字段的顺序与项目符号列表中的顺序相同：用户名、领域、SASL 机制和认证信息。但请注意，领域字段不是必须的，可能并不总是存在。如果 SASL 不使用任何领域信息，领域字段将被省略。

当然，我们的 LDAP 中没有像上面的 SASL 字符串那样的 DN 记录。因此，为了将经过身份验证的 SASL 用户与 LDAP 中的用户关联起来，我们需要设置一种方法，将上述类似 DN 的字符串转换为像目录中那些 DN 一样的结构化 DN。因此，我们希望将给定的字符串变成类似下面这样的格式：

```
uid=matt,ou=Users,dc=example,dc=com
```

进行映射有两种方式。我们可以配置一个简单的字符串替换规则，将 SASL 信息字符串转换为像最后一个那样的 DN，或者我们可以在目录中查找一个 `uid` 为 `matt` 的条目，然后如果找到匹配项，使用该匹配条目的 DN。

这两种方法各有优缺点。使用字符串替换更快，但不够灵活，可能不足以应对复杂的目录信息树。使用字符串替换时，可能需要连续使用多个 `authz-regexp` 指令，每个指令有不同的正则表达式和替换字符串。

另一方面，在一个拥有大量子树的目录中，查找用户可以更加灵活。但这会增加额外的 LDAP 树搜索开销，并且可能需要调整 ACL 以允许进行预认证搜索。

两种方法都使用 `slapd.conf` 中的相同指令：`authz-regexp` 指令。让我们从字符串替换方法开始，查看每种方法的示例。

#### 在 `authz-regexp` 中使用替换字符串

`authz-regexp` 指令接受两个参数：一个正则表达式，用于从 SASL DN 类似字符串中提取信息，和一个替换函数（根据我们是使用字符串替换还是搜索而有所不同）。

对于我们的正则表达式，我们想从 SASL 信息中提取用户名，并将其映射到 DN 中的 `uid` 字段。我们不需要其他三个 SASL 字段中的任何信息，因此我们的正则表达式非常简单：

```
"^uid=([^,]+).*,cn=auth$"
```

这个规则从行的开头 (`^`) 开始，查找以 `uid=` 开头的条目。接下来的部分 `([^,]+)` 会将 `uid=` 后和逗号（`,`）前的字符存储在一个名为 `$1` 的特殊变量中。这个规则的意思是“匹配尽可能多的字符（至少一个字符），这些字符不是逗号，并将它们存储在第一个变量（`$1`）中。”

此后，规则（使用 `.*` 匹配任何字符）跳过领域（如果有的话）和机制，然后寻找行尾的匹配项：`cn=auth$`（其中美元符号 (`$`) 表示行结束）。

一旦正则表达式执行完成，我们应该有一个变量 `$1`，它包含用户的名称。现在，我们可以在替换规则中使用该值，将 `uid` 的值设置为 `$1` 的值。整个 `authz-regexp` 行如下所示：

```
authz-regexp "^uid=([^,]+).*,cn=auth$"
             "uid=$1,ou=Users,dc=example,dc=com"
```

在 `authz-regexp` 指令之后，我插入了我们刚才查看的正则表达式。正则表达式之后是替换规则，指示 SLAPD 在该模板 DN 的 `uid` 字段中插入 `$1` 的值。

`authz-regexp` 指令可以放在 `slapd.conf` 文件中任何位置，只要它出现在第一个 `database` 指令之前。

由于 `authz-regexp` 是配置 SASL 所需的唯一指令，现在我们可以在命令行上测试 SLAPD，而不需要对 `slapd.conf` 做任何其他更改：

```
$ ldapsearch -LLL -U matt@example.com -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt@example.com
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

之前，我们已经使用了 `-x` 标志，结合 `-W` 和 `-D`，执行了一个简单的绑定操作，使用了完整的 DN 和密码。

然而，使用 SASL 时，我们不需要完整的 DN。我们只需要一个简化的连接字符串。因此，我们不再使用 `-x`、`-W` 和 `-D` 标志，而是使用 `-U matt@example.com`。`-U` 标志接受一个 SASL 用户名和（可选的）领域。领域与用户名通过 *@* 符号连接。所以，在给定的示例中，我们使用用户名 `matt` 和领域 `example.com` 进行连接。

接下来，`ldapsearch` 会提示输入密码（请参见示例中的高亮行）。这不是我们的 LDAP 密码，而是我们的 SASL 密码——也就是在运行 `saslpasswd2` 时创建的账户密码。

回顾一下，之前命令中发生的事情如下：

+   客户端正在连接到 SLAPD，请求进行 SASL 绑定。

+   SLAPD 使用 SASL 子系统（该子系统检查 `/usr/lib/sasl/slapd.conf` 文件中的设置）来告知客户端如何进行身份验证。在此案例中，它告诉客户端使用 DIGEST-MD5。

+   客户端将身份验证信息发送到 SLAPD。

+   SLAPD 执行 `authz-regexp` 中指定的转换。

+   然后，SLAPD 使用 SASL 子系统检查客户端的响应，并与 `/etc/sasldb2` 中的信息进行匹配。

+   当客户端身份验证成功时，OpenLDAP 执行搜索并将结果返回给客户端。

现在我们准备好使用 `authz-regexp` 来用特定的过滤器搜索目录了。

#### 在 authz-regexp 中使用搜索过滤器

在这种情况下，我们希望搜索目录中与 SASL 绑定期间接收到的用户名（`uid`）匹配的条目。回想一下，SASL 认证信息是以如下字符串的形式传入的：

```
uid=matt,cn=example.com,cn=DIGEST-MD5,cn=auth
```

在上一个例子中，我们直接将给定的映射到如下形式的 DN：

```
uid=<username>,ou=users,dc=example,dc=com.
```

但如果我们不知道，比如说，用户 `matt` 是否在 Users OU 或 System OU 中，该怎么办？一个简单的映射函数是行不通的。我们需要搜索目录。我们将通过修改 `authz-regexp` 指令中的最后一个参数来实现这一点。

我们新的 `authz-regexp` 指令如下所示：

```
authz-regexp "^uid=([^,]+).*,cn=auth$"
             "ldap:///dc=example,dc=com??sub?(uid=$1)"
```

这个正则表达式与前一个示例中的相同。但是 `authz-regexp` 的第二个参数是一个 LDAP URL。

### 注意

有关编写和使用 LDAP URL 的概述，请参见附录 B。

这个 LDAP URL 指示 SLAPD 在 `dc=example,dc=com` 基础上进行搜索（使用子树（`sub`）搜索），查找 `uid` 等于 `$1` 的条目，`$1` 被替换为从正则表达式第一参数中获取的值。如果用户 `matt` 尝试进行身份验证，例如，URL 将如下所示：

```
ldap:///dc=example,dc=com??sub?(uid=matt)
```

当 SLAPD 对我们的目录信息树进行搜索时，它将返回一个记录——即 DN 为 `uid=matt,ou=Users,dc=example,dc=com` 的记录。

这是使用 `ldapsearch` 的一个示例。它与前一节中使用的示例相同，即使我们使用 LDAP 搜索方法，它应该也会得到相同的结果：

```
$ ldapsearch -LLL -U matt@example.com -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt@example.com
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

#### 关于 ACL 和搜索过滤器的说明

当 SLAPD 读取搜索过滤器时，它会执行对目录的搜索。但搜索是以匿名用户的身份进行的。这意味着我们需要确保匿名用户拥有使用该过滤器进行目录搜索所需的权限。

根据我们之前的示例，匿名用户需要能够在 `dc=example,dc=com` 子树中搜索 `uid` 值。我们在第二章中创建的 ACL 并未授予匿名用户此类权限。为了使搜索操作成功，我们需要向 ACL 中添加一条规则：

```
access to attrs=uid
       by anonymous read
       by users read
```

这条规则应该出现在 ACL 列表的顶部，它授予 `anonymous` 以及系统中任何已认证用户对 `uid` 属性的读取访问权限。在这个示例中，重要的是匿名用户获得了读取权限。

请记住，通过添加这条规则，我们使得未经身份验证的用户能够看到数据库中存在的用户 ID。根据目录数据的性质，这可能会带来安全问题。如果这是一个问题，您可以使用字符串替换方法（记住，您可以连续使用多个 `authz-regexp` 表达式来处理更复杂的模式匹配），或者通过构建更严格的 ACL 来减少 `uid` 字段的暴露。

在本章后续部分，我们将更详细地讨论 ACL。

#### 映射失败

在某些情况下，`authz-regexp`的映射可能会失败。也就是说，SLAPD 会使用搜索过滤器在目录中查找，但没有找到匹配项。然而，用户已通过身份验证，SLAPD 不会失败并停止绑定。

相反，发生的情况是用户将以 SASL DN 进行绑定。因此，有效的 DN 可能类似于：

```
uid=matt,cn=example.com,cn=digest-md5,cn=auth
```

即使目录中没有与该用户名对应的实际记录，也没有关系。客户端仍然可以访问该目录。

但这个 DN 也受到 ACL 的控制，因此你可以针对那些通过 SASL 认证但没有相应目录记录的用户编写访问控制。

#### 去除指定领域的需要

在我们的配置中，所有用户都位于相同的领域`example.com`。为了避免每次都输入用户名和领域，我们可以通过在`slapd.conf`中添加以下指令来配置默认领域：

```
sasl-realm  example.com
```

如果我们用这个新的修改重启服务器，我们现在可以运行`ldapsearch`而不需要指定领域：

```
$ ldapsearch -LLL -U matt -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

这一次，传递`-U matt`就足以进行身份验证。SLAPD 自动将默认领域插入到 SASL 信息中。

#### 调试 SASL 配置

获取正确的 SASL 配置可能令人沮丧。提高调试能力的一种方法是配置日志记录，以便你可以看到 SASL 事务期间发生的情况。`trace`调试级别（`1`）可以用来观察 SASL 中的活动。你可以在`slapd.conf`中设置调试级别为 trace（或直接设置为数字`1`），或者你可以在命令行上将`slapd`运行在前台：

```
$ sudo slapd -d trace
# some of the voluminous output removed...
slap_sasl_getdn: u:id converted to uid=matt,cn=DIGEST-MD5,cn=auth
>>> dnNormalize: <uid=matt,cn=DIGEST-MD5,cn=auth>
<<< dnNormalize: <uid=matt,cn=digest-md5,cn=auth>
==>slap_sasl2dn: converting SASL name uid=matt,cn=digest-md5,cn=auth 
                 to a DN
slap_authz_regexp: converting SASL name 
                   uid=matt,cn=digest-md5,cn=auth
slap_authz_regexp: converted SASL name to 
                   uid=matt,ou=Users,dc=example,dc=com
slap_parseURI: parsing uid=matt,ou=Users,dc=example,dc=com
ldap_url_parse_ext(uid=matt,ou=Users,dc=example,dc=com)
>>> dnNormalize: <uid=matt,ou=Users,dc=example,dc=com>
<<< dnNormalize: <uid=matt,ou=users,dc=example,dc=com>
<==slap_sasl2dn: Converted SASL name to 
                 uid=matt,ou=users,dc=example,dc=com
slap_sasl_getdn: dn:id converted to 
                 uid=matt,ou=users,dc=example,dc=com
```

通过这个日志，我们可以看到初始的 SASL 字符串`uid=matt,cn=DIGEST-MD5,cn=auth`，并观察它是如何被标准化、运行正则表达式并转换为`uid=matt,ou=users,dc=example,dc=com`的。

`ldapwhoami`客户端和`slapauth`工具在调试 SASL 时也非常有用。下一节将给出使用`ldapwhoami`评估`authz-regexp`结果的示例。

## 使用客户端 SSL/TLS 证书进行身份验证

SASL 和 SSL/TLS 可以结合使用来执行**SASL EXTERNAL 认证**。在 SASL EXTERNAL 认证中，SASL 模块依赖于外部来源，在这种情况下是客户端的 X.509 证书，作为身份来源。

使用此配置，拥有适当签名证书的客户端可以绑定到 SLAPD，而无需输入用户名和密码，但这种方式仍然是安全的。

这是如何工作的？就像可以为服务器颁发 SSL/TLS 通信证书一样，也可以为用户或客户端颁发证书。我们已经讨论过，证书可以以安全的方式提供关于服务器的身份信息。客户端证书也可以发挥相同的作用。

认证，使用 SASL EXTERNAL 工作方式如下：

+   客户端和服务器使用 SSL/TLS 保护进行通信，可以使用 LDAPS 或使用 StartTLS

+   当服务器发送其证书时，请求客户端也提供一个证书

+   客户端发送自己的证书，其中包括以下内容

    +   身份信息

    +   一个公钥

    +   服务器将识别的证书颁发机构签名

+   服务器在验证证书后，通过 SASL 子系统将身份信息传递给 SLAPD

+   SLAPD 然后使用该信息进行绑定

由于客户端发送的证书包含了验证客户端身份所需的所有信息，因此不需要登录/密码组合。

配置 SASL EXTERNAL 机制需要以下步骤：

1.  创建一个新的客户端证书

1.  配置客户端以发送证书

1.  配置 SLAPD 以正确处理客户端证书

1.  配置 SLAPD 以正确转换客户端证书中提供的身份信息

### 创建一个新的客户端证书

创建新的客户端证书与创建服务器证书没有显著区别。我们将使用本章早期创建的同一证书颁发机构。

首先，我们需要创建一个新的证书请求：

```
$ /usr/lib/ssl/misc/CA.pl -newreq
Generating a 1024 bit RSA private key
............++++++
..++++++
unable to write 'random state'
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished 
    Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) 
    [Internet Widgits Pty Ltd]:Example.Com
Organizational Unit Name (eg, section) []: 
Common Name (eg, YOUR name) []:matt
Email Address []:matt@example.com

Please enter the following 'extra' attributes
    to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

这个过程就像之前的过程一样，只是字段是专门为代表此证书的用户填写的。例如，如果我们为芭芭拉生成这个证书，我们会用她的信息填写**通用名称**和**电子邮件地址**字段。

### 提示

**通用名称字段应填写什么？**

早些时候，我们使用 CN 字段存储域名。个人的 CN 字段应填写什么？一个选择是使用用户的全名。更实用的选择是使用用户 LDAP DN 中使用的标识符（如用户的`uid`属性的值）。这样做可以更轻松地从证书映射到 LDAP 记录。

现在，我们有了新请求（`newreq.pem`）和密钥（`newkey.pem`）。接下来要做的是使用我们 CA 的数字签名对证书进行签名：

```
$ /usr/lib/ssl/misc/CA.pl -signreq
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
    Serial Number:
      ba:49:df:f5:8e:7e:77:c6
    Validity
      Not Before: Jul  4 03:28:28 2007 GMT
      Not After : Jul  3 03:28:28 2008 GMT
    Subject:
      countryName               = US
      stateOrProvinceName       = Illinois
      localityName              = Chicago
      organizationName          = Example.Com
      commonName                = matt
      emailAddress              = matt@example.com
    X509v3 extensions:
      X509v3 Basic Constraints: 
        CA:FALSE
      Netscape Comment: 
        OpenSSL Generated Certificate
      X509v3 Subject Key Identifier: 

9A:97:8F:8C:95:1F:E0:6E:50:BD:DF:F4:C5:71:68:92:3F:A0:30:DD
      X509v3 Authority Key Identifier: 

keyid:6B:FB:66:33:5D:DB:32:40:42:D7:71:F7:F0:D0:7C:94:3E:8F:CD:58

Certificate is to be certified until 
    Jul 3 03:28:28 2008 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
unable to write 'random state'
Signed certificate is in newcert.pem
```

现在，我们将签名的证书存储在文件`newcert.pem`中。

接下来要做的是将这些文件移动到用户方便的位置。在这种情况下，我们将在用户的主目录中创建一个新目录，并将文件移动到该目录中：

```
 $ sudo mkdir /home/mbutcher/certs
 $ sudo mv new*.pem /home/mbutcher/certs
 $ sudo chown -R mbutcher:mbutcher /home/mbutcher/certs

```

在这三行中，我们为证书创建一个新目录。在这种情况下，新的`certs/`目录位于用户的主目录中。

然后，我们将新创建的证书文件移动到新目录中。我们可以重命名这些文件，但目前使用通用名称即可。

最后，我们需要确保用户能够访问自己的证书。这可以通过`chown`命令来完成。

证书已准备就绪。

### 配置客户端

接下来我们需要做的是配置客户端使用证书和密钥。这通过在用户的主目录下创建`.ldaprc`文件来完成。

### 注意

**.ldaprc 文件**是`ldap.conf`文件的“个人”版本。它支持`ldap.conf`中通常包含的所有指令，此外还有几个特殊指令，比如`TLS_CERT`和`TLS_KEY`指令。

由于我是用户`mbutcher`，我将在自己的主目录下创建这个文件：

```
$ cd /home/mbutcher
$ touch .ldaprc
```

现在我们可以编辑`.ldaprc`文件。这个文件需要指明客户端正在使用 SASL EXTERNAL 机制。同时，它必须包含关于证书和密钥文件的指令。此外，指定 CA 证书的位置（甚至指定签署服务器证书的 CA 的特定证书）也是一个好主意，尽管通常这在全局层面通过`ldap.conf`文件来完成。

`.ldaprc`文件如下所示：

```
SASL_MECH EXTERNAL
TLS_CERT /home/mbutcher/certs/newcert.pem
TLS_KEY /home/mbutcher/certs/newkey.pem
TLS_CACERT /etc/ssl/certs/Example.Com-CA.pem
```

第一个指令`SASL_MECH`指示客户端使用的 SASL 机制。在我们的例子中，客户端使用的是`EXTERNAL` SASL 机制。

`TLS_CERT`指令指向客户端签名的 X.509 证书的位置，`TLS_KEY`指令指示客户端私钥文件的位置。

`TLS_CACERT`指令指向用于签署服务器证书的特定证书。客户端库将使用该证书在 SSL/TLS 协商期间验证服务器的身份。

目前客户端已经准备好。接下来我们需要配置 SLAPD。

### 配置服务器

SLAPD 需要做一些工作，才能使 SASL EXTERNAL 机制生效：

+   它必须请求客户端的证书（否则客户端将不会提供证书）

+   它需要将客户端证书中给出的身份信息转换为在我们环境中有意义的 DN。

要让服务器请求客户端证书，只需要添加一条指令。在`slapd.conf`文件的全局部分，在任何数据库指令指定之前，应添加`TLSVerifyClient`指令：

```
TLSCACertificateFile    /etc/ssl/certs/Example.Com-CA.pem
TLSCertificateFile      /etc/ldap/example.com.cert.pem
TLSCertificateKeyFile   /etc/ldap/example.com.key.pem
TLSVerifyClient         try

```

只有高亮的那一行是新的。其他的行是我们在本章早些时候添加的。

`TLSVerifyClient`决定 SLAPD 是否采取步骤请求和验证客户端证书。它有四个可能的值：

+   `never`：永远不请求客户端证书。这是*默认*设置。如果没有请求证书，客户端就不会提供证书。因此，当`TLSVerifyClient`设置为`never`时，无法使用 SASL EXTERNAL 认证。

+   `allow`：这将导致 SLAPD 请求客户端证书，但如果客户端没有提供证书，或者提供的证书无效（例如，无法验证签名），会话将继续进行。

+   `try`：在这种情况下，SLAPD 将请求客户端提供证书。如果客户端未提供证书，连接将继续。然而，如果客户端提供的证书无效，连接将终止。

+   `demand`：这将导致 SLAPD 要求客户端提供证书。如果客户端没有提供证书，或者提供的证书无效，连接会终止。

在最后的示例中，我们将`TLSVerifyClient`设置为`try`。这意味着，如果客户端提交了证书，它必须是有效的证书（并且有一个已知的 CA 签名），否则 SLAPD 将不允许连接。但它也允许客户端在没有证书的情况下连接（尽管这些客户端无法使用 SASL EXTERNAL 认证）。

如果我们希望强制客户端提供证书，那么我们应该使用`demand`关键字而不是`try`。

此时，我们已经正确配置了 SSL/TLS。接下来，我们需要添加一个额外的步骤：我们需要将证书提供的身份（即 DN）映射到目录用户的 DN。

### 注意

将证书 DN 转换为另一个 DN 不是严格必要的。用户可以使用证书 DN 进行绑定，即使它不在目录中。ACL 可以写成目标这些 DN。

我们创建的客户端证书中的 DN 看起来像这样：

```
dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,\
    st=illinois,c=us
```

请注意，这是一个长行。

证书中的 DN 包含我们在运行`CA.pl -newreq`时输入的信息。我们要做的是将这个 DN 转换为对应的 LDAP 记录的 DN：`uid=matt,ou=users,dc=example,dc=com`。

这个翻译是如何完成的？使用我们在 SASL 认证部分中研究过的`authz-regexp`指令。

证书的身份字符串中有两个特别有用的字段来识别用户：`email`和`cn`。因此，简单的正则表达式可以捕获这两个字段：

```
^email=([^,]+),cn=([^,]+).*,c=us$
```

这将把电子邮件地址分配给`$1`，将 CN 分配给`$2`。

从这里开始，我们可以指定一个 LDAP URL，使用邮箱地址过滤器来查找 DN，或者可以将 CN 替换为 LDAP DN 中使用的 UID 字段（因为 CN 可以干净地映射到 UID）。

我们将使用第二种方法，并创建如下的`authz-regexp`：

```
authz-regexp "^email=([^,]+),cn=([^,]+).*,c=us$"
             "uid=$2,ou=Users,dc=example,dc=com"
```

这个指令将证书 DN 的 CN 值映射到 LDAP 授权 DN 中的 UID 属性。因此，当客户端连接并提供一个证书，证书的 DN 是`dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,c=us`时，SLAPD 会将其转换为 DN `uid=matt,ou=users,dc=example,dc=com`。

现在我们已经准备好测试了。

### 使用 ldapwhoami 进行测试

测试此过程的理想客户端是`ldapwhoami`。它将允许我们使用 SASL EXTERNAL 连接和绑定。此外，它将指示`authz-regexp`是否将证书 DN 映射到我们的 LDAP DN。

重启 SLAPD 以加载更改后，我们可以测试服务器：

```
$ ldapwhoami -ZZ -H 'ldap://example.com'
Enter PEM pass phrase:
SASL/EXTERNAL authentication started
SASL username: emailAddress=matt@example.com,CN=Matt, \O=Example.Com,L=Chicago,ST=Illinois,C=US
SASL SSF: 0
dn:uid=matt,ou=users,dc=example,dc=com
Result: Success (0)
```

首先，让我们仔细看看输入的命令：

```
ldapwhoami -ZZ -H 'ldap://example.com'
```

`-ZZ`标志要求 StartTLS 协商必须成功完成。仅使用一个`Z`将尝试 StartTLS，但如果协商失败不会关闭连接。使用`-ZZ`在尝试使用 SASL EXTERNAL 机制进行身份验证时总是一个好主意。

接下来，`-H 'ldap://example.com'`参数提供了 SLAPD 服务器的 URL。记住，StartTLS 协商要正常工作，这里 LDAP URL 中的域名必须与服务器证书中的域名匹配。

当执行这个命令时会发生什么呢？首先，系统会提示用户输入密码短语：

```
Enter PEM pass phrase:
```

这个提示实际上是由 SSL/TLS 子系统（OpenSSL）生成的。回想一下，我们生成的密钥是由密码短语保护的。为了读取密钥文件，OpenSSL 子系统需要密码短语。

但我不是说过 SASL EXTERNAL 方法可以避免输入密码吗？是的，的确可以——但要实现这一点，我们需要像生成服务器证书时那样移除密钥中的密码短语：

```
openssl rsa < newkey.pem > clearkey.pem
```

然后，`.ldaprc`中的`TLS_KEY`指令需要调整，指向`clearkey.pem`文件。

在某些情况下，移除密码短语可能是需要的，而在其他情况下则不推荐。请记住，移除密钥中的密码短语将使证书更容易被他人劫持。没有密码短语的密钥应该通过权限和其他手段进行严格保护。

一旦用户输入了密码短语，SASL 身份验证开始：

```
SASL/EXTERNAL authentication started
SASL username: emailAddress=matt@example.com,CN=Matt, \O=Example.Com,L=Chicago,ST=Illinois,C=US
SASL SSF: 0
```

如这里所见，使用了 SASL EXTERNAL 机制，并且 SASL 用户名被设置为`emailAddress=matt@example.com,CN=Matt,O=Example.Com,L=Chicago,ST=Illinois,C=US`。最后，SASL 安全强度因子被设置为 0，因为没有使用 SASL 安全机制。相反，安全机制是*外部*的，位于 SASL 之外。由于我们使用带有 AES-256 加密证书的 SSL/TLS，整体 SSF 仍然是 256。

一个重要的细节是，SLAPD 会标准化 DN。在标准化形式下，DN 看起来像这样：

```
email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,\
    c=us
```

`emailAddress`属性已经转换为`email`，所有大写字符串已经转换为小写。我们上面看到的`authz-regexp`在这个标准化的 DN 版本上进行操作。

最后，输出的最后几行是 LDAP Who Am I?操作的结果：

```
dn:uid=matt,ou=users,dc=example,dc=com
Result: Success (0)
```

根据 SLAPD，客户端当前正在使用有效 DN 为`uid=matt,ou=users,dc=example,dc=com`执行目录操作。这意味着我们的映射成功。

如果`authz-regexp`映射没有成功，输出会是什么样的呢？它可能看起来像这样：

```
$ ldapwhoami -ZZ -H 'ldap://example.com'
Enter PEM pass phrase:
SASL/EXTERNAL authentication started
SASL username: 
emailAddress=matt@example.com,CN=Matt,O=Example.Com,L=Chicago,
    ST=Illinois,C=US
SASL SSF: 0
dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,c=us
Result: Success (0)

```

高亮显示的部分展示了“我是谁？”操作的结果。返回的 DN 只是证书 DN 的标准化形式——而不是期望的 LDAP DN。

### 进一步了解 SASL

SASL 是一个灵活的身份验证处理工具。在这里我们只讨论了两种 SASL 机制：DIGEST-MD5 和 EXTERNAL。但还有许多其他的可能性。它可以与像 Kerberos 这样的强大网络身份验证系统一起使用。它可以利用像 Opiekeys 这样的安全一次性密码系统。还可以作为与更标准的密码存储系统（如 PAM（可插拔身份验证模块））的接口。

尽管这种配置超出了本书的范围，但有很多可用资源。SASL 文档（在 Ubuntu 上本地安装在 `/usr/local/doc/libsasl/index.html`），以及 OpenLDAP 管理员指南（[`openldap.org`](http://openldap.org)），都提供了有关不同 SASL 配置的更多信息。

现在我们将从身份验证转向授权，并关注 ACL（访问控制列表）。

# 使用 ACL 控制授权

我们已经讨论了连接安全性和身份验证。现在我们准备讨论安全性的最后一个方面：授权。我们特别感兴趣的是控制对目录树中信息的访问。谁应该能够访问记录？在什么条件下？他们应该能看到该记录的多少内容？这些就是我们将在本节中解决的问题。

## ACL 基础知识

OpenLDAP 控制目录数据访问的主要方式是通过访问控制列表（ACL）。当 SLAPD 服务器处理来自客户端的请求时，它会评估客户端是否具有访问所请求信息的权限。为进行此评估，SLAPD 会顺序评估配置文件中的每个 ACL，按照适当的规则应用到传入的请求。

### 注意

在本章之前，我们已经讨论了使用简单绑定和 SASL 绑定的*身份验证*。ACL 提供*授权*服务，决定给定 DN 可以访问哪些信息。

在第二章的*ACLs*部分中介绍了 ACL。本节将扩展那里讨论的基本示例。

ACL 只是 SLAPD 的一个特殊配置指令（`access` 指令）。像某些其他指令一样，`access` 指令可以多次使用。SLAPD 配置中有两个不同的地方可以放置 ACL。首先，它们可以放置在数据库部分之外的全局配置中（即，配置文件的顶部附近）。放置在此级别的规则将全局应用于所有后端。在下一章中，我们将讨论一个目录有多个后端的情况。

其次，ACL 可以放置在后端部分（`database` 指令下的某个位置）。在这种情况下，ACL 仅在处理数据库内的信息请求时使用。在第二章中，我们将 ACL 放在了后端部分，并且没有创建任何全局 `access` 指令。

这一切在实践中是如何运作的？何时使用全局规则，何时使用特定后端规则？如果一个后端没有特定的 ACL，则将应用全局规则。如果后端确实有 ACL，则只有在没有任何后端特定规则应用的情况下，才会应用全局规则。如果请求是针对存储在任何后端中的记录（例如根 DSE 或 `cn=subschema` 条目）时，则仅会应用全局规则。

在其上下文中，ACLs 是自上而下进行评估的，从配置文件中的第一条指令开始，直到最后一条。所以，当测试特定后端规则时，SLAPD 会从列表中的第一条规则开始测试，并按顺序继续，直到找到匹配的停止规则，或者 SLAPD 到达列表的末尾。

在第二章中，我们将 ACL 直接放入 `slapd.conf` 配置文件中。在本节中，我们将把它们放在自己的文件中，并在 `slapd.conf` 中使用 `include` 指令，指示 SLAPD 加载 ACL 文件。这将允许我们将可能很长的 ACL 与配置文件的其他部分分开。

让我们快速浏览一下 ACL 的格式，然后我们将继续一些示例，帮助澄清 ACL 方法的复杂性。

一个访问指令如下所示：

`访问` [*资源*]

`by` [*谁*] [*授予的* *访问类型*] [*控制*]

`by` [*谁*] [*授予的* *访问类型*] [*控制*]

```
# More 'by' clauses, if necessary....

```

一个 `访问` 指令可以有一个 `to` 短语，并且可以有任意数量的 `by` 短语。我们将首先查看 `访问` 部分，然后是 `by` 部分。

## 访问 [资源]

在 `访问` 部分，ACL 指定了此规则在目录树中要限制的内容。在给定的规则中，我们使用了 `[resources]` 作为此部分的占位符。ACL 可以通过 DN、属性、过滤器或它们的组合来进行限制。我们将首先查看如何通过 DN 限制访问。

### 使用 DN 的访问

要限制对特定 DN 的访问，我们可以使用如下内容：

```
access to dn="uid=matt,ou=Users,dc=example,dc=com"
       by * none
```

### 注意

`by * none` 短语简单地拒绝任何人的访问权限。我们将在本章稍后讨论 `by` 短语时详细介绍这一点及其他规则。

该规则将限制对特定 DN 的访问。每当收到需要访问 DN `uid=matt,ou=Users,dc=example,dc=com` 的请求时，SLAPD 会评估此规则，以确定该请求是否被授权访问该记录。

限制对特定 DN 的访问有时是有用的，但除了 DN 访问限定符外，还有其他几个支持的选项对于更通用的规则制定非常有用。

可以限制对 DN 子树的访问，甚至通过 DN 模式进行限制。例如，如果我们想编写一个规则，限制对 Users OU 下条目的访问，我们可以使用如下的 `访问` 子句：

```
access to dn.subtree="ou=Users,dc=example,dc=com"
       by * none

```

在这个示例中，规则限制了对 OU 及其下属记录的访问。这是通过使用`dn.subtree`（或同义词`dn.sub`）来实现的。在我们的目录信息树中，Users OU 子树下有多个用户记录。这些记录是 Users OU 的子级。例如，`uid=matt,ou=Users,dc=example,dc=com`的 DN 就在子树中，尝试访问该记录将触发此规则。

除了`dn.subtree`，还有三个其他关键字，用于为 DN 访问修饰符添加结构性限制：

+   `dn.base`：限制对此特定 DN 的访问。这是默认值，`dn.exact`和`dn.baselevel`是`dn.base`的同义词。

+   `dn.one`：限制对此 DN 下方的任何条目的访问。`dn.onelevel`是其同义词。

+   `dn.children`：限制对此 DN 的子级（下属）条目的访问。这与子树相似，唯一不同的是，规则不限制给定的 DN 本身。

`dn`子句接受另一个修饰符，可以用于进行复杂的模式匹配：`dn.regex`。`dn.regex`访问修饰符可以处理 POSIX 扩展正则表达式。以下是一个`dn.regex`中简单正则表达式的示例：

```
access to dn.regex="uid=[^,]+,ou=Users,dc=example,dc=com"
       by * none
```

这个示例将限制对具有`uid=SOMETHING,ou=Users,dc=example,dc=com`模式的任何 DN 的访问，其中`SOMETHING`可以是任何至少有一个字符长且不包含逗号（`,`）的字符串。正则表达式是编写 ACL 的强大工具。在我们讨论`by`短语之后，我们将在*获取* *更多* *正则表达式*一节中深入讨论它们。

### 使用 attrs 访问

除了通过 DN 限制对记录的访问，我们还可以限制对记录中一个或多个属性的访问。这是通过使用`attrs`访问修饰符来实现的。

在我们看到的示例中，当我们限制访问时，我们是在限制记录级别的访问。`attrs`限制作用于更细粒度的级别：它限制对记录中特定属性的访问。

例如，假设我们希望限制对目录信息树中所有记录的`homePhone`属性的访问。可以使用以下访问短语来实现：

```
access to attrs=homePhone
       by * none
```

`attrs`修饰符接受一个或多个属性的列表。在给定的示例中，我们仅限制了对`homePhone`属性的访问。如果我们还想阻止对`homePostalAddress`的访问，可以相应地修改`attrs`列表：

```
access to attrs=homePhone,homePostalAddress
       by * none
```

假设我们想要限制对`organizationalPerson`对象类中所有属性的访问。一种方法是创建一个长长的列表：`attrs`=`title`，`x121Address`，`registeredAddress`，`destinationIndicator`，....但这种方法既费时、难以阅读，又显得笨重。

相反，有一种方便的简写符号表示法：

```
access to attrs=@organizationalPerson
       by * none

```

然而，应该小心使用这种符号。此代码不仅限制对`organizationalPerson`中明确定义的属性的访问，还限制对`person`对象类中已经定义的所有属性的访问。为什么？因为`organizationalPerson`对象类是`person`的子类。因此，`person`的所有属性都是`organizationalPerson`的属性。

有时，限制对所有*未*被特定对象类要求或允许的属性的访问是有用的。例如，考虑一个情况，我们只想限制那些在`organizationalPerson`对象类中未指定的属性。我们可以通过将*at*符号（`@`）替换为感叹号（`!`）来实现：

```
access to attrs=!organizationalPerson
       by * none
```

这将限制对任何属性的访问，除非它们被`organizationalPerson`对象类允许或要求。

有两个特殊的名称可以在属性列表中指定，但它们并不真正匹配任何属性。这两个名称是`entry`和`children`。所以我们有两种情况：

+   如果指定了`attrs=entry`，则限制该记录本身。

+   如果`attrs=children`，则限制此记录的子记录。

当只使用`attrs`指定符时，这两个关键字并不特别有用，但当`attrs`和`dn`指定符一起使用时，它们会变得更加有用。

有时候，通过属性的值（而不是属性名）来限制访问会很有用。例如，我们可能希望限制对任何`givenName`属性值为`Matt`的访问。可以通过使用`val`（值）指定符来实现：

```
access to attrs=givenName val="Matt"
       by * none

```

和`dn`指定符一样，`val`指定符也具有`regex`、`subtree`、`base`、`one`、`exact`和`children`风格。

### 注意

使用`val`指定符时，`attrs`列表中最多只能有一个属性。`val`指定符也无法对对象类列表起作用。

使用`val.regex`，你可以使用正则表达式进行匹配。我们可以修改最后一个例子，限制对任何以字母`M`开头的`givenName`的访问：

```
access to attrs=givenName val.regex="M.*"
       by * none
```

在属性值是 DN（例如`groupOfNames`对象的`member`属性）时，可以使用`regex`、`subtree`、`base`、`one`、`exact`和`children`风格，根据属性值中的 DN 来限制访问。

```
access to attrs=member val.children="ou=Users,dc=example,dc=com"
       by * none
```

### 提示

**指定替代匹配规则**

默认情况下，`val`比较使用的是相等匹配规则。然而，你可以选择不同的匹配规则，通过在`val`后插入斜杠（`/`），然后输入匹配规则的名称或 OID：`access to attrs=givenName val/caseIgnoreMatch="matt"`。

### 使用过滤器进行访问

`access` 短语的一个较少使用但非常强大的功能是支持使用 LDAP 搜索筛选器来限制对记录的访问。我们在第三章开始时讨论搜索操作时已经看过 LDAP 筛选器的语法。在这里，我们将使用筛选器来限制对记录部分的访问。

筛选器提供了一种支持对整个记录进行值匹配的方法（而不仅仅是像 `attrs` 中那样匹配属性值）。例如，使用筛选器，我们可以限制访问所有包含 `simpleSecurityObject` 对象类的记录：

```
access to filter="(objectClass=simpleSecurityObject)"
       by * none
```

这将限制对目录信息树中所有具有 `simpleSecurityObject` 对象类的记录的访问。可以在筛选器指定符中使用任何合法的 LDAP 筛选器。例如，我们可以限制对所有具有名字 Matt、Barbara 或姓氏 Kant 的记录的访问：

```
access to 
    filter="(|(|(givenName=Matt)(givenName=Barbara))(sn=Kant))"
       by * none
```

这段代码使用了“或”（析取）操作符，表示如果请求需要访问名字为 Matt 或 Barbara 的记录，或者请求需要访问姓氏为 Kant 的记录，则应应用此规则。

### 组合访问指定符

我们已经看过三种不同的访问指定符：`dn`、`attrs` 和 `filter`。在前面的章节中，我们已使用过每种指定符。现在，我们将它们组合起来创建更具体的访问规则。

组合的顺序如下：

`access to` [*dn*] [*filter*] [*attrs*] [*val*]

`dn` 和 `filter` 指定符排在前面，因为它们处理的是整个记录。接着是 `attrs`（和 `val`），它们在属性级别起作用。假设我们希望限制仅在记录具有 `employeeNumber` 属性的情况下才能访问 Users 组织单位中的记录。为此，我们可以使用 DN 指定符和筛选器的组合：

```
access to dn.subtree="ou=Users,dc=example,dc=com"
    filter="(employeeNumber=*)"
       by * none
```

此 ACL 仅在请求的是 `ou=Users,dc=example,dc=com` 子树中的记录，并且 `employeeNumber` 字段存在且有值时，才会限制访问。

类似地，我们可以限制对某个子树中记录的属性的访问。例如，假设我们希望限制对 `description` 属性的访问，但仅限于 System 组织单位中的记录。我们可以通过组合 DN 和属性指定符来实现：

```
access to dn.subtree="ou=System,dc=example,dc=com" 
    attrs=description
       by * none
```

根据此规则，客户端可以访问 DN 为 `uid=authenticate,ou=System,dc=example,dc=com` 的记录，但无法访问该记录的 `description` 属性。

通过仔细组合这些访问指定符，可以精确地表述访问限制。我们将在继续研究 `by` 短语时看到更多应用实例。

## 通过 [who] [授予的访问类型] [控制]

`by` 短语包含三个部分：

+   **who 字段**表示允许访问访问短语中标识的资源的实体

+   **访问字段**（授予的访问类型）表示可以对资源执行的操作

+   第三个可选部分，通常被省略，是 **控制字段**。

为了理解这种区别，考虑一下我们在前面部分中使用的 `by` 子句：`by * none`。在这个 `by` 子句中，`who` 字段是 `*`（星号字符），访问字段是 `none`。这个示例中省略了控制字段。

`*` 是通用通配符。它匹配任何实体，包括匿名和所有 DN。`none` 访问类型表示不应授予任何权限给 `who` 指定的实体。换句话说，`by * none` 表示不应授予任何人访问权限。

### 注意

在 `slapd.conf` 文件中通过 `rootdn` 指令指定的目录管理员（`cn=Manager,dc=example,dc=com`）是一个例外。它不能被任何访问控制限制。因此，`by * none` 不适用于管理员。

我们将详细探讨 `who` 字段，但在此之前，让我们先来看看访问字段。

### 访问字段

客户端在访问某个条目或属性时，可以拥有六种不同的权限。此外，还有第七种权限，它表示移除所有权限：

1.  `w`：写入访问记录或属性。

1.  `r`：读取记录或属性的访问权限。

1.  `s`：搜索记录或属性的访问权限。

1.  `c`：访问以在记录或属性上执行比较操作。

1.  `x`：执行服务器端身份验证操作来访问记录或属性。

1.  `d`：访问有关记录或属性是否存在的信息（'d' 代表 'disclose'）。

1.  `0`：不允许访问记录或属性。这相当于 `-wrscxd`。

这七种权限可以在 `by` 子句中指定。要设置一个或多个访问权限，可以使用 `=`（等号）。

例如，为了让服务器将记录的 `givenName` 字段与客户端指定的 `givenName` 进行比较，我们可以使用以下 ACL：

```
access to attrs=givenName 
       by * =c
```

这将允许任何客户端尝试执行比较操作。但这就是它唯一允许的操作。根据这个规则，没有人可以读取或写入此属性。实际操作中是怎样的呢？当我们使用 `ldapsearch` 客户端尝试读取 `givenName` 属性的值时，无法获取任何有关 `givenName` 的信息：

```
$ ldapsearch -LLL -U matt "(uid=matt)" givenName
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
dn: uid=matt,ou=Users,dc=example,dc=com

```

服务器返回的唯一信息是与过滤器匹配的记录的 DN。不返回 `givenName` 属性。

然而，如果我们使用 `ldapcompare` 客户端，我们可以请求服务器告诉我们 DN 是否有一个值为 'Matt' 的 `givenName` 字段：

```
$ ldapcompare -U matt uid=matt,ou=Users,dc=example,dc=com \ 
 "givenName: Matt"
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
TRUE

```

`ldapcompare` 客户端将一个 DN 和一个属性/值对发送到服务器，并请求服务器将提供的属性值与服务器中该 DN 记录的属性值进行比较。

在这里，`ldapcompare`客户端将请求 SLAPD 服务器查找`uid=matt,ou=Users,dc=example,dc=com`的记录，并检查`givenName`属性是否为'Matt'。服务器将返回`TRUE`、`FALSE`，或者（如果出现错误）`UNDEFINED`。

在这种情况下，服务器回应了`TRUE`。这表明服务器执行了比较，并且值匹配。`ldapsearch`和`ldapcompare`的组合示例应当能够说明 ACL 的工作原理：虽然服务器端允许进行比较操作，但客户端没有权限读取属性值。

可以在一个`by`子句中授予多个访问权限。为了修改并允许对`givenName`属性进行读取（`r`）、比较（`c`）和披露（`d`），我们可以使用以下 ACL：

```
access to attrs=givenName 
       by * =rcd
```

现在，我们之前运行的`ldapsearch`和`ldapcompare`命令应该成功。

在某些情况下，权限是从其他 ACL 继承的（我们稍后会讨论）。在这种情况下，我们可以通过使用`+`（加号）来添加，使用`–`（减号）来移除特定权限，进行有选择的添加或移除。

例如，如果我们知道所有用户已经对所有属性拥有比较（`c`）和披露（`d`）权限，但我们只想为`givenName`属性添加*读取*权限，我们可以使用以下 ACL：

```
access to attrs=givenName 
       by * +r
```

### 注意

一个授予比较和披露权限，并继续处理的访问控制可能类似于这样：`access to attrs=givenName,sn by * =cd break`。这使用了`break`控制指令，告知 SLAPD 继续处理 ACL。如果这个规则出现在 SLAPD 配置文件中，位于规则`access to attrs=giveName by * +r`之上，那么对`givenName`属性的请求将会有有效权限`=rcd`。

同样，如果我们需要仅为`givenName`属性移除比较操作，可以使用类似`by * -c`的`by`子句。

`0`访问权限移除所有权限。它不能与`+`或`–`操作符一起使用，只能与`=`操作符一起使用。以下 ACL 移除所有用户对`givenName`属性的所有权限：

```
access to attrs=givenName 
       by * =0
```

这与`by`子句相同：`by * -wrscdx`。

这些访问控制适用于细粒度的权限控制，但有时使用快捷方式会更方便。OpenLDAP 提供了七个快捷方式，用于处理常见的访问控制配置：

| 关键字 | 权限 |
| --- | --- |
| `none` | `0` |
| `disclose` | `d` |
| `auth` | `xd` |
| `compare` | `cxd` |
| `search` | `scxd` |
| `read` | `rscxd` |
| `write` | `wrscxd` |

我们之前见过的`none`关键字与`=0`相同。通过查看其他关键字及其相关权限，可以看出一种模式：每个关键字会在前一个关键字的权限基础上添加一个新权限。因此，`auth`拥有来自`disclose`的`=d`权限，加上`x`权限，而`compare`拥有来自`auth`的`=xd`权限，并添加了`c`权限。底部的`write`关键字具有所有权限。

因为这种通用的权限累积方式既能捕获常见的使用案例，又保持了较高的可读性，所以关键词比权限字符串更常用。在接下来的例子中，除非有特别的原因使用权限字符串，否则我们将使用关键词。

### 注意

在七个关键词中，`disclose`、`auth`、`compare`、`search`、`read` 和 `write` 可以加上两种前缀之一：`self` 和 `realself`。`self` 前缀表示如果相关值指的是用户的 DN，那么用户可能会拥有某些权限。因此，`selfwrite` 表示只有当属性的值为用户的 DN 时，用户才拥有 `=wrscxd` 权限。

`realself` 前缀与 `self` 类似，但它附带了额外的规定，即 DN 不能被代理。这些前缀在处理组和其他基于成员的记录时特别有用。

例如，以下 ACL 只允许用户在 `uniqueMember` 属性包含该用户的 DN 时对其进行 `write` 操作：`access to attrs=uniqueMember by users selfwrite`。

现在我们已经讲解了访问字段，接下来我们将讨论 `who` 字段。

### `who` 字段

我们一直在 `who` 字段中使用 `*`。然而，`who` 字段是 ACL 字段中最丰富的，提供了二十三种不同的形式，其中大部分可以组合使用。为了高效覆盖这些内容，我们将单独讲解主要形式，然后将相似的形式组合在一起作为一个整体来处理。

五种最常用的形式是 `*`、`anonymous`、`self`、`users` 和 `dn`。

#### `*` 和 `anonymous` 修饰符

`*` 修饰符，如我们所见，是一个全局匹配符。它匹配任何客户端，包括匿名用户。

`anonymous` 修饰符只匹配那些以匿名用户身份绑定到目录的客户端（有关匿名用户的详细信息，请参见第三章）。这指的是那些没有对目录进行身份验证的客户端。由于认证过程要求客户端以匿名身份连接，然后尝试作为 DN 绑定并使用特定密码，因此匿名用户几乎总是需要权限来执行 `auth` 操作，即客户端将 DN 和密码发送到目录，并要求目录验证这些信息是否正确。因此，你可能需要一个像这样的 ACL：

```
access to attrs=userPassword
       by anonymous auth
```

这授予匿名用户执行认证操作的权限。请注意，每个 ACL 以隐式的短语结尾：`by * none`。换句话说，如果权限没有明确指定，则不授予任何权限。

请注意，上述 ACL 不允许用户修改自己的密码。这正是 `self` 修饰符的作用所在。

#### `self` 修饰符

`self` 修饰符用于指定对一个 DN 自身记录的访问控制。因此，我们可以使用 `self` 修饰符来允许用户修改她或他的 `userPassword` 值：

```
access to attrs=userPassword
       by anonymous auth
       by self write
```

如果我们以 `uid=matt,ou=Users,dc=example,dc=com` 登录并尝试修改自己记录的 `userPassword` 值（`dn: uid=matt,ou=Users,dc=example,dc=com`），SLAPD 将允许我们更改密码。但它不会（根据上述规则）允许我们修改其他人的 `userPassword` 值。

### 注意

`self` 指定符可以进一步使用 `level` 样式进行修饰。`level` 样式表示是否（以及多少个）父记录或子记录应被视为 `self` 的一部分。`level` 样式采用整数索引。正整数表示父记录，而负整数表示子记录。

因此，`access to` `ou` `by` `self.level{1}` `write` 表示当前的 DN 对其父级的 `ou` 拥有写权限。同样，`access` `to` `ou` `by` `self.level{-1}` `write` 表示当前的 DN 对其任何直接子级的 `ou` 拥有写权限。

#### `users` 指定符

`users` 指定符表示任何已认证的客户端。匿名用户不包括在 `users` 中，因为它表示尚未进行身份验证的客户端。

当需要允许任何已认证的用户访问某些资源时，dn 指定符非常有用。例如，在企业目录中，我们可能希望允许所有用户查看彼此的姓名、电话号码和电子邮件地址：

```
access to attrs=sn,givenName,displayName,telephoneNumber,mail
       by self write
       by users read
```

#### dn 指定符

`dn` 指定符在 `by` 语句中与其在 `access` `to` 语句中的作用类似。它指定一个或多个 DN。`dn` 拥有 `regex`、`base`、`one`、`subtree` 和 `children` 修饰符，它们在这里的作用与在 `access` `to` 语句中的作用相同。以下是使用几个不同 DN 模式的示例：

```
access to dn.subtree="ou=System,dc=example,dc=com" attrs=description
       by dn="uid=barbara,ou=Users,dc=example,dc=com" write
       by dn.children="ou=System,dc=example,dc=com" read
       by dn.regex="uid=[^,]+,ou=Users,dc=example,dc=com" read
```

该规则限制对系统 OU 子树中任何对象的描述属性的访问。用户 `uid=barbara,ou=Users,dc=example,dc=com` 拥有对描述的写权限，而系统 OU 的任何子级用户则拥有*读取*权限。DN 形式为 `uid=SOMETHING,ou=Users,dc=example,dc=com` 的用户也拥有对描述的*读取*权限。

除了常规的 DN 修饰符外，`by` 语句中的 `dn` 还可以带有 `level` 修饰符。Level 允许 ACL 编写者精确指定 `by` 语句应该下降多少级。回想一下，`dn.one` 指定符表示任何直接位于指定 DN 下的记录将获得指定权限。例如，`by` `dn.one="ou=Users,dc=example,dc=com"` `read` 会授予 `Users` OU 的任何直接后代读取权限。因此，`uid=matt,ou=Users,dc=example,dc=com` 会被授予读取权限，但 `uid=jake,ou=Temp,ou=Users,dc=example,dc=com` 不会被授予此权限，因为他位于第二级。`dn.level` 指定符让我们可以任意指定下降多少级。例如，`by` `dn.level{2}="ou=Users,dc=example,dc=com"` `read` 将允许 `matt` 和 `jake` 都获得读取权限。

### 注意

**代理认证与真实 DN**

如果 SLAPD 被设置为允许代理认证，其中一个 DN 用于认证，然后另一个 DN 用于执行其他目录操作，那么有时根据用于认证的 DN（即真实的 DN）编写 ACL 是很有用的。可以使用`realdn`指定符来实现这一点。它的功能与`dn`指定符相同，唯一的区别是它作用于真实的 DN。此外，`realanonymous`、`realusers`、`realdnattr`和`realself`可以用来基于真实的 DN 进行限制。详情请参阅`slapd.access`手册页：`man` `slapd.access`。

#### 组和成员

有时，授权组成员访问某个对象是很有用的。例如，如果你有一个管理员组，你可能希望授予该组的任何成员对系统 OU 中所有记录的写访问权限。

可能会有人认为，为组成员设置权限的方式就是在 ACL 中使用该组作为`dn`指定符的值。但事实并非如此，因为`dn`指定符是指整个组记录，且与组成员无关，每个组成员在目录中都有自己的记录。

但我们真正需要的是一种方法，可以搜索特定组记录中的成员属性，然后授予记录中列出的 DN 访问权限。`group`指定符正好提供了这种能力。

组评估可以使用`group`指定符来完成。其最简单的形式如下所示：

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group="cn=Admins,ou=Groups,dc=example,dc=com" write
       by users read
```

此 ACL 将授予`cn=Admins,ou=Groups,dc=example,dc=com`组的成员对系统 OU 中任何内容的写权限，同时授予所有其他用户只读权限。

### 提示

**顺序很重要**

通过短语的 ACL 按顺序进行评估，默认情况下，当 SLAPD 找到匹配项时，它会停止处理`by`短语。换句话说，如果上述规则中的`by`短语被反转，LDAP 管理员的成员将永远无法获得写权限，因为他们总是会匹配到`by` `users` `read`短语。在检查组成员资格之前，ACL 的评估就会停止。

但是上面的 ACL 只会对那些对象类为`groupOfNames`，并且成员属性为`member`的组起作用。这是因为`groupOfNames`是默认的分组对象类，而`member`是默认的成员属性。

当我们在第三章创建 LDAP 管理员组时，它不是`groupOfNames`，也没有使用`member`属性来表示成员关系。我们的记录如下所示：

```
dn: cn=LDAP Admins,ou=Groups,dc=example,dc=com
cn: LDAP Admins
ou: Groups
description: Users who are LDAP administrators
uniqueMember: uid=barbara,ou=Users,dc=example,dc=com
uniqueMember: uid=matt,ou=Users,dc=example,dc=com
objectClass: groupOfUniqueNames
```

我们使用了`groupOfUniqueNames`对象类和`uniqueMember`成员属性。为了让 ACL 匹配这些约束，我们需要在`group`指定符中指定对象类和成员属性：

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group/groupOfUniqueNames/uniqueMember=
 "cn=LDAP Admins,ou=Groups,dc=example,dc=com" write
       by users read
```

请注意高亮行中的更改。通过使用斜杠（`/`），我们首先指定了对象类，然后是应该用来确定条目代表哪些成员的成员属性。当评估此`by`短语时，SLAPD 将找到 DN `cn=LDAP` `Admins,ou=Groups,dc=example,dc=com`，检查它是否具有`groupOfUniqueMembers`对象类，然后如果在`uniqueMember`属性中指定了该 DN，则授予写权限。

使用这种扩展的表示法，你可以将其他基于成员的记录作为组来使用。例如，你可以使用`organizationalRole`对象类和`roleOccupant`成员属性。

与许多其他说明符一样，组说明符也支持`regex`样式的正则表达式。因此，我们可以创建一个规则，允许任何 OU Groups 组的成员对系统 OU 拥有写权限，方法是扩展我们最后的示例：

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group/groupOfUniqueNames/uniqueMember.regex=
 "cn=[^,]+,ou=Groups,dc=example,dc=com" write
       by users read
```

第二行和第三行应该合并为`slapd.conf`中的一长行。组说明符中的正则表达式将匹配所有具有 CN 组件的 DN。对于所有此类条目，如果对象类是`groupOfUniqueMembers`，则 SLAPD 将为该组中的`uniqueMember`用户授予成员资格。

#### 基于成员的记录访问

如果一个组成员需要修改他或她所属组的记录，该怎么办？允许这种操作的一种方法是使用`dnattr`说明符。`dnattr`说明符仅在客户端的 DN 出现在记录的某个属性中时授予访问权限。例如，以下示例允许一个组（`groupOfUniqueNames`对象）的组成员（`uniqueMember`）访问该组记录：

```
access to dn.exact="cn=LDAP Admins,ou=Groups,dc=example,dc=com"
       by dnattr=uniqueMember write
       by users read
```

第二行指定，如果客户端的 DN 出现在`uniqueMember`属性的值列表中，则该客户端应获得对整个组记录的写入权限。根据第三行，其他用户将只能读取访问权限。

#### 网络、连接与安全

SLAPD 可以在访问控制列表中使用客户端连接的信息（包括网络和安全信息）。该功能提供了一个额外的网络安全层，补充了 SSL/TLS 和 SASL。

以下是网络或连接级别的说明符：

+   `peername`：用于指定 IP 地址范围（用于`ldap://`和`ldaps://`）。

+   `sockname`：用于指定 LDAPI 监听器（`ldapi://`）的套接字文件。

+   `domain`：用于指定`ldap://`和`ldaps://`监听器的域名。

+   `sockurl`：用于指定 LDAPI 监听器的套接字文件的 URL 格式（`ldapi://var/run/ldapi`）。

+   `ssf`：连接的整体安全强度因子（SSF）。

+   `transport_ssf`：网络传输层的安全强度因子（SSF）。

+   `tls_ssf`：SSL/TLS 连接的安全强度因子（SSF）。它适用于 LDAPS 监听器上的 SSL/TLS 连接，以及 LDAP 监听器上的 Start TLS。

+   `sasl_ssf`：SASL 连接的安全强度因子（SSF）。

SSF 限定符（`ssf`，`transport_ssf`，`tls_ssf`，`sasl_ssf`）执行与 SSF 参数对 SLAPD `security` 指令相同的检查（在本章的第一部分中讨论）。然而，在这种情况下，SSF 可用于有选择地限制（或授予）对目录信息树部分的访问。SSF 限定符需要一个整数值来指定所需的安全级别。例如，使用 `ssf=256` 将要求连接的整体 SSF 为 256。 但 `tls_ssf=56` 将要求 TLS/SSL 层的 SSF 至少为 56，无论 SASL 配置的 SSF 是多少。有关 SSF 的更多信息，请参阅本章前面的标题为 *使用* *安全* *强度* *因子* 的部分。

例如，以下 ACL 仅在客户端使用强大的 SASL 密码连接时，才会授予指定 DN 的 *写入* 权限：

```
access to dn.subtree="ou=users,dc=example,dc=com"
 by self sasl_ssf=128 write
       by users read
```

此规则仅允许用户在通过 SASL 认证并使用强度为 128（DIGEST-MD5）或更高的安全机制时修改自己的记录。所有其他用户只能获得读取权限。

### 提示

**在 by 短语中组合限定符**

如上规则所示，多个限定符可以在同一个 by 短语中使用。当这种情况发生时，所有限定符必须匹配，才能授予（或拒绝）指示的权限。

`peername` 限定符用于根据 IP 连接信息设置限制。它可以与网络安全中的其他组件（如 SSL/TLS）配合使用。`peername` 限定符可以接受一个 IP 地址或一系列 IP 地址（使用子网掩码），还可以指定源端口。

以下规则授予本地连接写入权限，授予本地局域网（地址从 10.40.0.0 到 10.40.0.255）上的连接读取权限，并拒绝所有其他客户端的访问。请记住，每条规则以隐式的 `by` `*` `none` 结尾。

```
access to *
       by peername.ip=127.0.0.1 write
       by peername.ip=10.40.0.0%255.255.255.0 read
```

请注意，`peername` 限定符要求使用 IP 风格来指定 IP 地址。它还支持 `regex` 风格（`access` `to` `*` `by` `peername.regex="^IP=10\`.`40\`.`0\`.`[0-9]+:[0-9]+$"` `write`）以及 `path` 限定符来复制 `sockname` 的行为。

### 提示

**IP 地址的正则表达式**

对于 IP 地址，在正则表达式评估中使用的字符串格式如下：`IP=<address>:<port>`。如果您正在创建精确的正则表达式，请确保处理 `IP=` 前缀和端口信息。像这样的正则表达式会失败：`peername.regex="¹⁰.40.12[0-9]$"`。为什么？因为它缺少 `IP=` 和端口信息。

上述规则的一个更有用的版本是，如果连接不在特定范围内，则拒绝访问目录中的所有内容，但会将进一步的访问控制留给 ACL 列表中后面的规则。这可以通过使用下节中描述的特殊`break`控制来实现。我们还可以添加 SSF 信息，这样通过非本地连接来的连接也必须使用强 SSL/TLS 加密。以下是规则：

```
access to * 
       by peername.ip=127.0.0.1 break
       by peername.ip=10.40.0.0%255.255.255.0 tls_ssf=128 break
```

上述规则可能看起来难以阅读，但它的作用如下：

+   如果连接是本地的（来自 127.0.0.1 或`localhost`），则 SLAPD 允许进一步处理 ACL 列表（这就是`break`的作用）。用户是否能够访问资源则依赖于其他规则。

+   如果连接来自局域网中的地址并且使用强 SSL/TLS 加密，那么 SLAPD 将继续处理 ACL 列表。

+   在任何其他连接情况下，连接都会被拒绝。例如，如果连接来自局域网，但未使用足够强的 SSL/TLS 加密，连接将被关闭。此行为是由隐式的`by` `*` `none`短语引起的。

有关`break`控制的更多信息，请参见名为*控制字段*的章节。

有时，能够指定哪些域名（而不是哪些 IP 地址）应被授予访问权限更加有用。这可以通过使用`domain`指定符来完成：

```
access to * 
       by domain.exact="main.example.com" write
       by domain.sub="example.com" read
```

在上面的示例中，第二行为来自域名`main.example.com`的任何客户端连接提供写入权限。第三行为`example.com`及其任何子域名提供读取权限。所以，如果域名为`test2.example.com`的服务器发起请求，它将在第三条规则下获得访问权限。然而，`testexample.com`则不匹配，因为它不是`example.com`的子域名——它是一个完全不同的域名。

当 SLAPD 在 ACL 中遇到域名指定符时，它会获取客户端连接的 IP 地址并进行反向 DNS 查找以获取主机名。鉴于此，在使用域名指定符时需要记住两点。

首先，反向 DNS 查找返回的名称可能与正向 DNS 查找返回的结果不同。例如，对`ldap.example.com`进行 DNS 查找返回地址 10.40.0.23，而对 10.40.0.23 进行反向 DNS 查找返回`mercury.example.com`。为什么会这样？

这是因为`ldap.example.com`在 DNS 术语中是一个**CNAME 记录**，而`mercury.example.com`是一个**A 记录**。实际上，这意味着`ldap.example.com`是服务器真实（**规范**）名称`mercury.example.com`的别名。实际结果是：当你使用`domain`指定符编写 ACL 时，确保使用 A 记录域名，而不是 CNAME 记录名。否则，SLAPD 会将规则应用到错误的域名。

### 提示

**查找 DNS 信息**

有许多工具可以查找 DNS 信息。大多数 Linux 发行版，包括 Ubuntu Linux，都提供了用于命令行 DNS 查找的 `host` 和 `dig` 命令。`host` 命令提供简短的类似句子的信息，例如：`ldap.example.com` `is` `an` `alias` `for` `mercury.example.com`。与此相对，`dig` 命令提供详细的技术信息。

在考虑域名指定符时，第二点需要记住的是，它的可靠性低于使用 IP 地址信息。DNS 地址可以被伪造，这意味着网络上的另一台服务器可能会假冒 `ldap.example.com`，并发送看起来像是来自真实 `ldap.example.com` 的流量给 SLAPD。

降低这种风险的一种方法是使用客户端 SSL/TLS 证书，并配置 SLAPD 要求客户端发送签名证书进行身份验证，然后才能执行任何其他目录操作。不幸的是，客户端证书不能通过 ACL 有选择地强制使用。相反，你需要在 `slapd.conf` 文件中使用 `TLSVerifyClient` `demand` 指令。

`sockname` 和 `sockurl` 指定符用于使用 UNIX 本地套接字进程间通信（IPC）而不是网络套接字运行的服务器。这些指令可以用于限制使用 IPC 层而非通过 IP 网络连接的本地连接。

### 注意

运行 LDAPI 并不常见。通常只在不能或不应使用 IP 网络连接的情况下使用。在典型情况下，本地客户端通过 LDAP 连接到 SLAPD，使用 `ldap://localhost/` URL，而不是使用 LDAPI。

例如，我们可以使用以下 ACL 仅允许本地（LDAPI）连接写入记录，而通过其他机制连接的用户只能读取记录：

```
access to dn.exact="uid=matt,ou=Users,dc=example,dc=com"	
       by sockurl="ldapi://var/run/ldapi" write
       by users read
```

第二行表示只有通过特定 LDAPI 套接字文件连接的 LDAPI 连接才应获得对 DN 的写访问权限。所有其他客户端（`用户`）将获得读取权限。

#### 高级步骤：使用 set 指定符

除了我们刚才检查过的语法外，还有一种实验性的 `by` 短语类型——**set** 语法。`set` 语法可以用于创建一个简洁且强大的访问条件集。由于它允许使用布尔运算符，并且具有访问属性值的方法，单个 `set` 语法规则可以完成原本需要非常复杂的 ACL 才能实现的任务。

`set` 语法的基本思想是这样的。通过使用由条件组成的规则，SLAPD 创建了一个对象集，这些对象可以访问相关记录。如果对 `set` 指定符的评估结果是一个包含一个或多个成员的集合，则 `by` 短语被视为匹配，权限将被应用。另一方面，如果集合为空，SLAPD 将继续评估该规则的 `by` 短语，以查看是否能找到另一个匹配项。

### 提示

`set`指定符使用与集合论中类似的操作。当使用集合指定符时，您可能会发现以集合论的角度思考非常有帮助，思考集合（项目的列表）和集合操作，如并集（`&`）和交集（`|`）。

这是一个简单的 ACL，使用`set`指定符来复制`group`指定符的行为。它只为 LDAP Admins 组中的客户端提供对系统 OU 中记录的写访问权限，其他所有客户端只能获得读访问权限：

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by set="[cn=ldap admins,ou=groups,dc=example,dc=com]/
 uniqueMember & user" write
       by users none
```

上面突出显示的第二行包含`set`指定符，其中包含`set`语句。方括号中的文本指定了一个 DN，即 LDAP Admins 组的 DN。为了访问`uniqueMember`属性的值，我们将`/uniqueMember`附加到 DN 上。当 SLAPD 展开时，它将包含 LDAP `Admins` `group` 中所有`uniqueMembers`的集合。在集合论表示法中（OpenLDAP 未使用此表示法，但有助于理解发生了什么），组成员的集合将如下所示：

```
{ uid=matt,ou=users,dc=example,dc=com ; 
    uid=barbara,ou=users,dc=example,dc=com }
```

LDAP Admins 组中有两个成员（两个`uniqueMembers`）。

`&`（与符号）运算符对两个集合执行并集操作。**user 关键字**展开为包含一个成员的集合：当前客户端的 DN。因此，如果我执行搜索，绑定为`uid=matt,ou=users,dc=example,dc=com`，那么用户集合将只包含一条记录：

```
{ uid=matt,ou=users,dc=example,dc=com }
```

当`&`运算符被应用时，它将生成两个集合的交集。也就是说，结果集合只会包含同时出现在原始两个集合中的成员。由于只有 UID 为`matt`的记录同时存在于两个集合中，因此结果集合只会包含`matt`的 DN：

```
{ uid=matt,ou=users,dc=example,dc=com }
```

结果集合非空，因此被视为匹配。集合评估的结果是，`uid=matt,ou=users,dc=example,dc=com`将基于`set`指定符获得访问权限。

### 注意

集合是区分大小写的，并且始终使用标准化的 DN 形式。这意味着集合中的 DN 应该始终是小写的。

然而，考虑一个情况，假设用户不是 LDAP Admins 组的成员。如果`uid=david,ou=users,dc=example,dc=com`绑定，是否可以执行读写操作？当集合指定符运行时，第一个集合（组成员）将与上述相同：

```
{ uid=matt,ou=users,dc=example,dc=com ; 
    uid=barbara,ou=users,dc=example,dc=com }
```

但是，user 关键字会展开为这个：

```
{ uid=david,ou=users,dc=example,dc=com }
```

这两个集合的交集为空集，因此，在应用`&`运算符后，结果集合是一个空集：

```
{ }
```

没有匹配项，因此这个`by`子句不会应用。我们 ACL 中的最后一行（`by` `users` `none`）将会应用，`uid=david`将不会获得任何访问权限。

让我们来看另一个例子。我们将使用集合指定符来实现一条规则，即当客户端 DN 尝试访问记录 DN 时，只有当两个 DN 相同，才赋予写访问权限；否则，如果它们在同一 OU 中，则赋予读访问权限。否则，客户端 DN 将被拒绝访问记录 DN。以下是 ACL：

```
access to dn.subtree="dc=example,dc=com"
       by set="this & user" write
       by set="this/ou & user/ou" read
```

第一行表明这条规则将应用于`dc=example,dc=com`的记录以及其下的所有内容。

第二行取自两个关键词生成的集合的交集：`this`和`user`。`this`关键词扩展为包含请求记录 DN 的集合。`user`关键词，如我们所见，扩展为客户端的 DN。

因此，如果客户端`uid=david,ou=users,dc=example,dc=com`请求访问其自己的记录，结果集合操作如下：

```
{ uid=david,ou=users,dc=exampls,dc=com } & 
    { uid=david,ou=users,dc=example,dc=com }
```

由于两个集合包含相同的成员，结果集合（两者的交集）为`{` `uid=david,ou=users,dc=example,dc=com` `}`。最终集合非空，因此该用户将被授予写访问权限。

现在让我们来看一下给定 ACL 的第三行。只要请求的 DN 和客户端的 DN 在`ou`属性上具有相同的值，这条规则就会返回一个非空集合。如果`uid=david,ou=users,dc=example,dc=com`请求`uid=matt,ou=users,dc=example,dc=com`的记录，SLAPD 将检查它们各自的 OU 属性值。

`this/ou`标识的集合将被扩展，包含请求记录中所有 OU 属性的值（`uid=matt,ou=users,dc=example,dc=com`的记录）。该集合为：

```
{ 'Users' }
```

请注意，在这种情况下，值不是 DN，而是字符串。集合可以对字符串以及 DN 执行匹配操作。

`user/ou`标识的集合将被扩展，包含客户端记录中所有 OU 属性的值。`uid=david,ou=users,dc=example,dc=com`的记录包含一个`ou`属性值，结果集合将包含该一个属性值：

```
{ 'Users' }

```

SLAPD 将计算`{` `'Users'` `}`与`{` `'Users'` `}`的交集，结果为`{` `'Users'` `}`。由于集合非空，`uid=david,ou=users,dc=example,dc=com`将被授予访问`uid=matt,ou=users,dc=example,dc=com`记录的权限。

`set`指定符提供了一种在记录包含特定属性时*仅*授予访问权限的方法。如果我们只想对具有 title 属性的记录授予写访问权限，可以使用以下规则：

```
access to dn.child="ou=Users,dc=example,dc=com"
       by set="this/title" write
```

在这个 ACL 中，如果请求的记录具有一个`title`属性，那么上述规则的评估结果将是一个包含一个元素的集合。然而，如果记录没有 title 属性，那么结果集合将为空，写访问权限将不会被授予。

在我们的目录中，`uid=matt,ou=users,dc=example,dc=com`的记录有以下 title 属性：

```
title: Systems Integrator
```

但`uid=barbara,ou=users,dc=example,dc=com`的记录根本没有 title 属性。因此，如果请求的是`uid=matt`的记录，基于上述 ACL，结果集合将是：

```
{ 'Systems Integrator' }
```

所以，如果一个经过认证的用户尝试访问`uid=matt`的记录，SLAPD 将授予访问权限。相反，`uid=barbara`的集合将是`{}`，即空集合。因此，尝试访问`uid=barbara`记录的用户将被拒绝访问。

使用类似的集合指定符，我们可以根据属性的存在性以及其值来授予对记录的访问权限：

```
access to dn.child="ou=Users,dc=example,dc=com"
       by set="this/objectclass & [person]" write
```

根据上述规则，只有当条目具有 `objectclass` 属性且其值为 `person` 时，才会授予对 Users OU 中任何内容的写访问权限。请注意，在这种情况下，方括号用于定义字符串文字。

如果客户端尝试访问记录 `uid=barbara,ou=users,dc=example,dc=com`，我们 `set` 语句的第一部分将计算出以下集合：

```
{ 'person' ; 'organizationalPerson' ; 'inetOrgPerson' }
```

这些是 `uid=barbara` 记录的三个对象类。另一部分 `[person]` 将扩展为以下集合：

```
{ 'person' }
```

当计算联合时，结果将是集合 `{'person'}`，因此会授予写访问权限。

这些只是使用 `set` 指定符可以执行的一些基本操作。不幸的是，`set` 在 `slapd.access` 手册页中没有文档记录。然而，OpenLDAP 官方 FAQ-O-Matic 上有一篇详细且内容丰富的文章介绍了如何使用 `set`：[`www.openldap.org/faq/data/cache/1133.html`](http://www.openldap.org/faq/data/cache/1133.html)。

### 控制字段

`by` 短语中的最后一个字段是控制字段。控制字段只有三种可能的值：`stop`、`break` 和 `continue`。如果未指定控制字段，则默认为 `stop`。例如，`by` `*` `none` 与 `by` `*` `none` `stop` 是相同的。

第一个值 `stop` 表示如果该特定的 `by` 条件匹配，则不应继续检查与之匹配的其他 ACL。考虑以下（虽然是人为构造的）情况：

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users=cd
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r

access to attr=employeeNumber
       by users +w
```

如果我以 `uid=matt,ou=Users,dc=example,dc=com` 身份绑定并尝试修改我的 `employeeNumber`，我会被允许吗？不会，我不会被允许。

我无法修改记录的原因是因为我只会拥有第一个 `by` 短语所授予的权限：`by` `users` `=cd`（记住，`by` `users` `=cd` 与 `by` `users=cd` `stop` 是相同的）。一旦 SLAPD 看到我匹配第一个 ACL 的第一个 `by` 短语，它就会停止测试 ACL。因此，它永远不会执行授予我 DN `+r` 访问权限的规则，也不会执行授予所有用户对 `employeeNumber` 属性 `+w` 权限的规则。

这是 `stop` 控制的一个例子，它被所有三个规则隐式使用。

现在，如果我想确保在第一个 `by` 短语之后，SLAPD 继续评估 ACL 内的短语，我可以使用 `continue` 控制重新编写 ACL：

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users-=cd continue
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r

access to attr=employeeNumber
       by users +w
```

在对这些规则进行相同测试后，DN `uid=matt,ou=Users,dc=example,dc=com` 将拥有 `=cdr` 权限。

`continue` 控制指示 SLAPD 继续处理 *当前* *ACL* 中的所有 `by` 短语。然而，一旦它完成了对该 ACL 的评估，它将不再继续查找其他 ACL 中的匹配项。

为了告诉 SLAPD 查看不同的规则来进行匹配，我们必须使用`break`控制。当 SLAPD 遇到以`break`控制结尾的适用子句时，它会停止处理当前的 ACL，但会继续查看其他 ACL，看看它们是否适用。

因此，为了通过 ACL 获得写权限，我们希望使用以下 ACL：

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users=cd continue
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r break

access to attr=employeeNumber
       by users +w stop
```

现在，当 UID 为`matt`的用户尝试访问`employeeNumber`时会发生什么呢？

首先，第一条 ACL 的`by`短语将被评估，`matt`将被授予`=cd`权限。由于`continue`控制，SLAPD 接着会检查第二个`by`子句，这个子句也会匹配到`matt`用户。因此，当第一条 ACL 处理完成时，`matt`将拥有`=rcd`权限。

由于`break`控制，第二个 ACL 也会被评估，并且`matt`将被授予`+w`权限，因此他的最终权限将是`=wrcd`。

使用`continue`和`break`控制语句是逐步处理权限的一种方法。在复杂的配置中，合理使用`continue`和`break`可以让维护 ACL 变得更加容易，并且减少 ACL 的总数。

## 从正则表达式中获取更多信息

在前面的章节中，我们已经看过了如何在`access` `to` 短语和`by` 短语中使用正则表达式。但我们也可以将它们结合使用。我们可以在`access` `to` 短语中存储匹配到的信息，然后在`by` 短语中使用这些信息。

为了临时存储`access` `to`短语中的匹配信息，我们可以用括号将正则表达式包裹起来。以下是一个示例：

```
access to dn.regex="ou=([^,]+),dc=example,dc=com"
       by dn.children,expand="ou=$1,dc=example,dc=com" read
```

这个 ACL 只会在客户端的 DN 和记录的 DN 位于同一个目录树部分（即它们位于同一个 OU 中）时，才授予客户端读取记录 DN 的权限。

在给定的 ACL 的第一行中，我们使用括号捕获了正则表达式`[^,]+`的匹配结果，它将成为 DN 的`ou=`部分的值。再说一次，`[^,]+`的意思是“匹配所有不是`,`的字符”。

在第二行中，我们使用了`dn.children`指定符，但加上了一个额外的关键字：`expand`。`expand`关键字告诉 SLAPD 将`access` `to`子句中的匹配项替换到这个短语中。

由于`expand`关键字，变量`$1`将被替换为第一行匹配的值。正则表达式中`(`和`)`之间捕获的所有内容将存储在`$1`中。

变量名称按顺序分配。正则`access` `to`短语中的第一组括号将存储在`$1`中。如果有第二组括号，里面的匹配信息将存储在`$2`中，以此类推，对于每一组括号都如此。

例如，我们可能想要像这样的 ACL：

```
access to dn.regex="uid=([^,]+),ou=([^,]+),dc=example,dc=com"
       by dn.children,expand="uid=$1,ou=$2,dc=example,dc=com" write
```

这条规则将授予客户端 DN 读取和写入其自己记录下属条目的权限，但禁止其他用户读取这些条目。

### 注意

地址簿有时通过将用户的地址存储为用户条目下的从属条目来在 OpenLDAP 中实现。在 OpenLDAP FAQ-O-Matic 中有一个示例：[`www.openldap.org/faq/data/cache/1005.html`](http://www.openldap.org/faq/data/cache/1005.html)

请注意，第一行存储了两个变量。UID 存储在 `$1` 中，OU 存储在 `$2` 中。这些变量在第二行中被展开。

也可以在 `by` 短语中使用来自 `access` `to` 短语的匹配项，作为正则表达式的一部分：

```
access to dn.regex="uid=[^,]+,ou=([^,]+),dc=example,dc=com"
       by dn.regex="uid=[^,]+,ou=$1,dc=example,dc=com" write
```

在第一行中，仅捕获并存储第二个正则表达式的结果，并将其存储在变量中。第二行也包含一个正则表达式，并利用 `$1` 变量从第一行中获取 OU 的值。请注意，`dn.children,expand` 已被 `dn.regex` 替代。正则表达式不需要添加 `expand` 关键字。

该规则授予客户端 DN 对目录树中同一 OU 下的任何用户记录的写入访问权限。

我们已经在这些 ACL 中看了一些简单但有用的正则表达式。但还可以构建更复杂的正则表达式，使 ACL 更加强大。当你编写更高级的正则表达式时，你可能会发现一些其他的信息源很有帮助。除了 `slapd.access` 手册页外，POSIX 扩展正则表达式手册页（`man` `regex`）也可能很有用。

## 调试 ACL

调试 ACL 可能令人沮丧。它们复杂、安全敏感，并且需要详细的测试。但有三种工具可以使调试和测试过程变得更容易。

第一个工具就是 `ldapsearch` 命令行客户端。它可以用于精心编写过滤器，专门用于测试 ACL 的处理。`ldapcompare` 工具在需要测试比较操作时也很有用。

但充分利用 LDAP 的日志指令也是很有帮助的。`trace` 和 `acl` 调试级别都提供有关 ACL 处理的详细信息。例如，`acl` 级别会记录每次 ACL 评估。这对于确定哪些规则被执行以及何时执行非常有用。我们发现 `trace` 调试级别也很有用，因为它提供了每次评估是如何执行的的信息，包括正则表达式是如何展开的。

### 提示

**在前台运行 SLAPD**

有时通过将 SLAPD 以前台模式运行，而不是作为守护进程运行，并将调试和日志信息打印到标准输出上来测试 ACL 会更容易。例如，我们可以通过这种方式打印 ACL 和 trace 调试信息：`slapd` `-d` `"acl,trace"`。请注意，你需要以合适的用户身份（如 `openldap`）运行此命令。要终止该进程，可以使用 *Ctrl*-*C* 键盘组合。

最后，`slapacl`命令行工具提供了一个注重细节的工具，用于直接评估 ACL。由于它不通过 LDAP 协议连接到 SLAPD 服务器，因此它允许直接测试 ACL。

例如，我们可以检查特定的 SASL 用户`matt`是否可以访问记录`cn=LDAP` `Admins,ou=Groups,dc=example,dc=com`并*读取*`description`属性的值：

```
 $ slapacl -U matt -b "cn=LDAP Admins,ou=Groups,dc=example,dc=com" \ 
 "description/read"

```

`-U` `matt` 参数指定了 SASL 用户名。`-b` `"cn=LDAP` `Admins,ou=Groups,` `dc=example,dc=com"` 参数指示我们希望测试的记录，最后一个字段`"description/read"`指示属性和访问级别。如果 ACL 允许读取访问，它将返回`ALLOWED`，否则返回`DENIED`。

同样，我们可以测试其他 LDAP 操作。例如，我们可以测试用户是否有权限进行`compare`操作：

```
$ slapacl -U matt -b "uid=matt,ou=Users,dc=example,dc=com" 
    "uid/compare"
authcDN: "uid=matt,ou=users,dc=example,dc=com"
compare access to uid: ALLOWED
```

在这个例子中，我们已经包含了响应内容。第一行响应显示了 SASL DN 是如何解析的，第二行显示了对`uid`的比较访问被允许。

`slapacl`程序本质上运行自己的 SLAPD，因此可以设置为将完整的处理日志打印到屏幕上。例如，要启用跟踪调试，我们只需将`-d` `trace`参数添加到给定命令中：

```
$ slapacl -U matt -b "uid=matt,ou=Users,dc=example,dc=com" -d trace 
    "uid/compare"
slapacl init: initiated tool.
slap_sasl_init: initialized!
hdb_back_initialize: initialize HDB backend
hdb_back_initialize: Sleepycat Software: Berkeley DB 4.3.29: 
    (September  6, 2005)
bdb_db_init: Initializing HDB database
>>> dnPrettyNormal: <dc=example,dc=com>
# LOTS of lines deleted...
<<< dnPrettyNormal: <uid=matt,ou=Users,dc=example,dc=com>, 
    <uid=matt,ou=users,dc=example,dc=com>
entry_decode: ""
<= entry_decode()
compare access to uid: ALLOWED
slapacl shutdown: initiated
====> bdb_cache_release_all
slapacl destroy: freeing system resources.
```

如您所见，`slapacl`在这种情况下提供了详细的评估信息。

使用 LDAP 命令行客户端、详细日志记录和`slapacl`命令，可以有效地调试和测试 ACL。

## 一个实际的例子

在本章的这一部分，我们对 OpenLDAP 中的 ACL 进行了低级别的研究。我们已经覆盖了 ACL 系统的许多细节。现在是时候将我们所学的内容应用到实践中，为我们的目录信息树创建一组通用的 ACL 了。

在第二章中，我们在`slapd.conf`文件中创建了一组基础的 ACL。以下是我们当时创建的内容：

```
########
# ACLs #
########
access to attrs=userPassword
       by anonymous auth
       by self write
       by * none

access to *
       by self write
       by * none
```

现在，我们将创建一组新的、更实际的 ACL。

我们首先要做的是将 ACL 从`slapd.conf`中移出来，放到一个单独的文件`acl.conf`中。这将使得 ACL 的长列表与我们其余的配置分开。为此，我们将用`include`指令替换上面的 ACL：

```
########
# ACLs #
########
include /etc/ldap/acl.conf
```

当 SLAPD 启动时，它会在`include`语句出现的位置包含`/etc/ldap/acl.conf`的内容。请记住，ACL 是与后端特定的。每个不同的数据库可以有自己的 ACL（并且多个数据库可以在同一个`slapd.conf`文件中定义）。因此，将`include`指令放在`slapd.conf`文件中的数据库`directive`之后是很重要的。

现在我们将开始编辑`acl.conf`文件。我们将编写的规则将是简单的，适用于大多数目录用户都被允许查看目录中大多数信息的目录。一个更高安全性的目录可能会有一份更复杂的 ACL 列表。

由于 ACL 是按从上到下的顺序进行评估的，我们需要仔细制定规则，以确保重要的限制能够立即生效。

如果存在基于网络的访问规则，它们通常应该出现在 ACL 列表的顶部，以便首先评估。例如，如果我们希望限制当主机不在我们的局域网内时对整个数据库的访问，我们可以使用以下规则：

```
access to * 
       by peername.ip=127.0.0.1 none break
       by peername.ip=10.40.0.0%255.255.255.0 none break
```

根据此规则，只有来自本地主机（127.0.0.1）和我们 10.40.0.0 子网内部的访问将被允许访问目录。由于指定了`break`控制，后续规则可能会修改`none`权限，从而授予客户端更多权限。所有其他连接将立即关闭。

接下来，我们希望授予 LDAP 管理员组成员对`dc=example,dc=com`树中所有内容的写访问权限：

```
access to dn.subtree="dc=example,dc=com"
       by group/groupOfUniqueNames/uniqueMember=
           "cn=LDAP Admins,ou=Groups,dc=example,dc=com" write
       by * none break
```

这立即授予 LDAP 管理员组成员写访问权限。然而，对于所有其他客户端，SLAPD 将继续处理。

### 注意

目录管理器不需要编写 ACL，`slapd.conf`指令`rootdn`中指定的 DN 始终具有对目录信息树的完全访问权限，ACL 对该用户没有任何作用。

接下来，我们希望确保`userPassword`字段对匿名用户可用，以便进行身份验证。我们还希望允许用户修改自己的密码，但除此之外，我们希望`userPassword`对其他人不可读写。请注意，根据前面的规则，LDAP 管理员也能够修改用户的密码。

```
access to attrs=userPassword
       by anonymous auth
       by self write
```

在某些情况下，其他用户可能也需要对密码进行`auth`访问，这时您可能需要将`by` `users` `auth`添加到给定的列表中。

如果我们在`authz-regexp`指令中使用`ldap://` URL 形式进行 SASL 绑定，我们还需要授予`uid`属性的访问权限。这是因为 LDAP URL 中的过滤器是以匿名身份运行的（参见*为 SASL 支持配置 SLAPD*小节的讨论）。

此外，我们不希望允许用户尝试修改他们自己的`uid`，因为`uid`在 DN 中被使用：

```
access to attrs=uid
        by anonymous read
        by users read
```

现在，匿名用户和所有经过身份验证的用户将能够访问他们有权限访问的目录中任何记录的`uid`属性。

还有一些其他属性，我们不希望用户能够修改——即使是在他们自己的记录中。

我们不希望用户尝试修改他们的 OU 属性，因为 OU 属性也用于 DN 中。我们也不希望他们能够修改他们的`employeeNumber`或`employeeType`：

```
access to attrs=ou,employeeNumber,employeeType by users read
```

我们有一个特殊账户`uid=Authenticate,ou=System,dc=example,dc=com`，该账户偶尔用于帮助进行绑定请求。此用户不应有权访问除了我们指定的内容之外的任何其他内容：

```
access to * 
       by dn.exact="uid=Authenticate,ou=System,dc=example,dc=com" 
           none
       by users none break
```

同样，最后一行指示 SLAPD 继续处理没有认证账户的用户的 ACL。这一行还将阻止匿名用户浏览树中的其他部分，因为末尾的隐式规则 `by` `*` `none` 会拦截匿名用户。

### 注意

`uid=Authenticate` 用户在之前的规则中已被授权访问 `uid` 属性，这个属性是该账户用于查找绑定所需的用户信息的。

假设我们不希望普通用户（位于 Users OU 中的 DNs）能够访问目录中的 System OU 记录（通常用于系统账户）。我们可以通过以下规则实现这一点：

```
access to dn.subtree="ou=System,dc=example,dc=com"
       by dn.subtree="ou=Users,dc=example,dc=com" none
       by users read
```

这拒绝了用户 OU 中的用户访问权限，但允许其他用户（如系统账户）访问这些记录。

我们还希望给予每个用户读取和写入自己记录的权限，但限制其他人访问这些记录。这使得用户可以在目录中存储自己的信息（如通讯录）：

```
access to dn.regex="^.*,uid=([^,]+),ou=Users,dc=example,dc=com$"
       by dn.exact,expand="uid=$1,ou=Users,dc=example,dc=com write
```

最后，我们需要的最后一条规则是默认规则。这个规则应该回答“当没有其他规则匹配时，我们希望发生什么？”的问题。我们希望用户能够修改自己的记录并查看他人的记录：

```
access to *
       by self write
       by users read
```

现在我们的 ACL 列表已完整。总体来看，它们是这样的：

```
#################################################
# ACLs
# These are ACLs for the first database section
# of the slapd.conf file found in this directory
#################################################
##
## Restrict by IP address:
access to *
       by peername.ip=127.0.0.1 none break
       by peername.ip=10.40.0.0%255.255.255.0 none break

## Give Admins immediate write access:
access to dn.subtree="dc=example,dc=com"
       by group/groupOfUniqueNames/uniqueMember="cn=LDAP 
           Admins,ou=Groups,dc=example,dc=com" write
       by * none break

## Grant access to passwords for auth, but allow users to change 
## their own.
access to attrs=userPassword
       by anonymous auth
       by self write

## This rule is needed by authz-regexp
## (Note: Since uid is used in DN, user cannot change its own uid.)
access to attrs=uid
       by anonymous read
       by users read
## Don't let anyone modify OUs, employee num or employee type.
access to attrs=ou,employeeNumber,employeeType by users read

## Stop authentication account from reading anything else. This also 
## stops anonymous.
access to *
       by dn.exact="uid=Authenticate,ou=System,dc=example,dc=com" 
           none
       by users none break

## Prevent DNs in ou=Users from seeing system accounts
access to dn.subtree="ou=System,dc=example,dc=com"
       by dn.subtree="ou=Users,dc=example,dc=com" none
       by users read

## Allow user to add subentries beneath its own record.
access to dn.regex="^.*,uid=([^,]+),ou=Users,dc=example,dc=com$"
       by dn.exact,expand="uid=$1,ou=Users,dc=example,dc=com" write

## The default rule: Allow DNs to modify their own records. Give 
## read access to everyone else.
access to *
       by self write
       by users read
```

尽管这些规则肯定无法满足所有需求，但它们为平衡目录的安全性和可用性提供了一个良好的起点。此外，它们为本书后续内容的操作奠定了基础。

在本书的后续章节中，我们将再次讨论并微调这些 ACLs，以支持更多功能，如目录复制。

# 总结

本章的重点是 OpenLDAP 的安全性，我们已经覆盖了很多内容。我们从连接级别的安全性开始，配置了我们的目录服务器的 SSL/TLS 加密。我们使用了标准 LDAP 端口上的 StartTLS，并且在端口 636 上配置了较旧的（LDAP v2）LDAPS 协议。接着，我们探讨了 LDAP 身份验证的过程。在这一部分，我们讨论了简单绑定和 SASL 绑定。最后，我们详细审视了访问控制列表（ACLs），并以一组基本的 ACLs 结束了本章。

在下一章中，我们将深入探讨 OpenLDAP 的 SLAPD 服务器的高级配置。我们将配置服务器以托管多个后端数据库，并通过目录覆盖来为我们的 SLAPD 服务器添加强大的附加功能。
