# 附录 C. 有用的 LDAP 命令

在本书过程中，我们查看了 OpenLDAP 发布版中所有命令行工具。但本书的范围要求我们简要讨论每个工具。也有一些高级用法，在某些时候可能会很有用。在本附录中，我提供了这些用法的示例。

在本附录中，我们将介绍

+   使用 `ldapsearch` 获取目录信息

+   使用两种不同策略创建目录备份

+   重建 BDB/HDB 数据库

# 获取目录信息

许多 LDAP 服务器提供有关其配置和功能的详细信息。这些信息以一种 LDAP 客户端可以直接访问的方式存储，客户端可以通过搜索操作来获取。例如，客户端可以获取 **根 DSE** 记录，以了解服务器的基本功能。它还可以访问服务器的 **子模式**，并了解支持哪些对象类、语法、匹配规则和属性。

## 根 DSE

**根 DSE（DSA 特定条目）**（其中 **DSA** 代表 **目录服务代理**）是一个特殊条目，提供关于服务器自身的信息。根 DSE 的 DN 是一个空字符串（""）。为了获取它，我们需要一个精心设计的 LDAP 查询，该查询将设置空的搜索基准并检索该根条目：

```
 $ ldapsearch -x -LLL -b '' -s base -W -D \
 'cn=Manager,dc=example,dc=com' '+'

```

请注意，基准被设置为空字符串，搜索范围仅限于基准记录。这些参数结合起来的效果是仅请求 DN 为空的记录。此外，由于根 DSE 中大多数属性是操作属性，因此我们需要在搜索的末尾指定 `'+'`。

运行此查询的结果大致如下：

```
dn:
structuralObjectClass: OpenLDAProotDSE
configContext: cn=config
namingContexts: dc=example,dc=com
supportedControl: 2.16.840.1.113730.3.4.18
supportedControl: 2.16.840.1.113730.3.4.2
supportedControl: 1.3.6.1.4.1.4203.1.10.1
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.2.826.0.1.334810.2.3
supportedControl: 1.2.826.0.1.3344810.2.3
supportedControl: 1.3.6.1.1.13.2
supportedControl: 1.3.6.1.1.13.1
supportedControl: 1.3.6.1.1.12
supportedExtension: 1.3.6.1.4.1.1466.20037
supportedExtension: 1.3.6.1.4.1.4203.1.11.1
supportedExtension: 1.3.6.1.4.1.4203.1.11.3
supportedFeatures: 1.3.6.1.1.14
supportedFeatures: 1.3.6.1.4.1.4203.1.5.1
supportedFeatures: 1.3.6.1.4.1.4203.1.5.2
supportedFeatures: 1.3.6.1.4.1.4203.1.5.3
supportedFeatures: 1.3.6.1.4.1.4203.1.5.4
supportedFeatures: 1.3.6.1.4.1.4203.1.5.5
supportedLDAPVersion: 3
supportedSASLMechanisms: NTLM
supportedSASLMechanisms: DIGEST-MD5
supportedSASLMechanisms: CRAM-MD5
entryDN:
subschemaSubentry: cn=Subschema

```

除其他事项外，该记录为我们提供了有关服务器理解并启用哪些控件、功能和扩展的信息。例如，记录中有一行 `supportedFeature`，内容如下：

```
supportedExtension: 1.3.6.1.4.1.4203.1.11.1
```

这一行表示该 LDAP 服务器支持 RFC 3062 中定义的 LDAPv3 扩展的 **更改密码** 操作（[`www.rfc-editor.org/rfc/rfc3062.txt`](http://www.rfc-editor.org/rfc/rfc3062.txt)）。

使用这些信息，一个精心设计的 LDAP 客户端将能够执行服务器端的更改密码操作，而不是在客户端更改密码后再使用修改操作将更改发送到服务器。

### 注意

更改密码操作的优点在于服务器的存储。如果客户端通过修改操作更改密码，它必须事先知道服务器支持哪些加密类型，它必须自己进行加密，然后将加密后的密码提交到服务器。通常，最好让客户端通过安全的方式（例如通过 TLS）与服务器进行连接，然后使用更改密码操作，这样服务器就可以进行存储。

根 DSE 记录还指向配置（`cn=config`）和子模式（`cn=subschema`）记录。

## 子模式记录

子模式记录存储在`cn=subschema`中。该记录包含有关服务器支持的模式的详细信息，包括它可用的匹配规则类型、允许在属性中使用的语法类型，以及服务器识别的属性和对象类。

客户端应用程序可以使用这些信息正确构造记录或搜索，并正确解释响应。

可以使用以下命令通过`ldapsearch`检索子模式记录：

```
 ldapsearch -x -LLL -b 'cn=subschema' -s base -W \
 -D 'cn=Manager,dc=example,dc=com'  '+'

```

在此示例中，我们通过将基础 DN 设置为`cn=config`，然后请求`base`类型的搜索（`-b` `'cn=subschema'` `-s` `base`），来请求所需的记录。这将返回具有 DN 为`cn=subschema`的确切记录。

此外，我们需要的多数属性是操作属性，这意味着它们不会在正常搜索中返回，因此最后我们指定`'+'`，表示我们需要操作属性。

返回的记录如下所示：

```
dn: cn=Subschema
structuralObjectClass: subentry
createTimestamp: 20061216235843Z
modifyTimestamp: 20061216235843Z
ldapSyntaxes: ( 1.3.6.1.1.16.1 DESC 'UUID' )
ldapSyntaxes: ( 1.3.6.1.1.1.0.1 DESC 'RFC2307 Boot Parameter' )
# ... lots of lines removed
objectClasses: ( 2.16.840.1.113730.3.2.2 NAME 'inetOrgPerson' 
  DESC 'RFC2798: Internet Organizational Person' 
  SUP organizationalPerson STRUCTURAL 
  MAY ( audio $ businessCategory $ carLicense $ departmentNumber $
  displayName $ employeeNumber $ employeeType $ givenName $
  homePhone $ homePostalAddress $ initials $ jpegPhoto $ 
  labeledURI $ mail $ manager $ mobile $ o $ pager $ photo $ 
  roomNumber $ secretary $ uid $ userCertificate $
  x500uniqueIdentifier $ preferredLanguage $ userSMIMECertificate $
  userPKCS12 ) )
objectClasses: ( 1.3.6.1.4.1.4203.666.11.1.4.2.1.2 NAME  
  'olcHdbConfig' DESC 'HDB backend configuration' 
  SUP olcDatabaseConfig STRUCTURAL 
  MUST olcDbDirectory 
  MAY ( olcDbCacheSize $ olcDbCheckpoint $ olcDbConfig $ olcDbNoSync
        $ olcDbDirtyRead $ olcDbIDLcacheSize $ olcDbIndex $
        olcDbLinearIndex $ olcDbLockDetect $ olcDbMode $ 
        olcDbSearchStack $ olcDbShmKey $ olcDbCacheFree ) )
entryDN: cn=Subschema
subschemaSubentry: cn=Subschema
```

一个子模式记录包含所有的模式信息，因此，它可能会有一千多行。

子模式记录特别有助于了解服务器支持哪些模式，或者在开发和调试自定义模式时，如第六章所讨论的那样。

## 配置记录

OpenLDAP 2.3 的一个实验性功能（并且这个功能可能在 OpenLDAP 2.4 中达到生产质量）是将 LDAP 配置存储在目录中。要实现这一点，必须首先使用特殊的配置模式将配置重新创建为 LDIF 格式，并指示 SLAPD 从这个新的 LDIF 文件中读取其配置。

配置信息存储在目录中的 DN 为`cn=config`。可以通过类似前一节使用的搜索来访问：

```
 ldapsearch -x -LLL -b 'cn=config' -s base -W \
 -D 'cn=Manager,dc=example,dc=com'  '*'

```

在 OpenLDAP 2.3 中，并非所有的叠加层和 OpenLDAP 的功能都能与这种新的配置样式正确配合工作，这对于其使用来说是一个重要的缺点。但改进这种替代配置机制是 OpenLDAP 2.4 开发中的一个优先事项。

将配置存储在目录中的优势是什么？以下是几个可能的优势：

+   通过`ldapsearch`和其他 LDAP 客户端轻松访问配置信息。

+   通过目录工具如`ldapmodify`编辑配置信息的能力。

+   SLAPD 配置的复制支持。你可以使用 SyncRepl 同步目录配置到网络上。

如果你想实现新的基于 LDAP 的配置文件格式，可以在 OpenLDAP 网站的 LDAP 管理员指南中了解：[`www.openldap.org/doc/admin23/slapdconf2.html`](http://www.openldap.org/doc/admin23/slapdconf2.html)。

# 进行目录备份

有两种常见的备份策略：一种是备份目录数据库，另一种是将目录内容导出到 LDIF 文件中。

## 目录数据库的备份副本

不同的后端将目录的内容存储在不同的位置。例如，BDB 和 HDB 后端将数据存储在特殊的 Berkeley DB 数据库文件中。基于 SQL 的后端将信息存储在关系型数据库管理系统中。像 LDAP 和 Perl 这样的特殊后端可能根本不存储数据，而只是访问其他数据源。

每个后端将需要不同的备份程序。这里我们只看如何备份 BDB 和 HDB 数据库——这是本书中使用的类型。

### 注意

这种方法不可移植。BDB/HDB 文件对版本非常敏感。每次 OpenLDAP（或 Berkeley DB）的新版本发布时，可能会使用不同的数据库结构，因此此备份方法仅在备份和恢复使用相同软件版本时有效。

在 Ubuntu 中，这些数据库文件位于 `/var/lib/ldap`。该目录中的所有文件，包括索引（以 `bdb` 扩展名结尾的文件）、主数据库文件（`__db.???`）和日志文件（`log.??????????`）。最好也备份 `DB_CONFIG` 文件，尽管它很少变化，且不存储任何目录数据。

在备份这些文件时，最好停止 SLAPD。下面是一个使用常见 shell 工具的简单示例：

```
 $ sudo invoke-rc.d slapd stop
 $ sudo cp -a /var/lib/ldap/* /usr/local/backup/ldap/
 $ sudo invoke-rc.d slapd start

```

这将停止 SLAPD 并将 `/var/lib/ldap/` 下的所有文件复制到 `/usr/local/backup/ldap/`。然后，SLAPD 会重新启动。

## 一个 LDIF 备份文件

第二种，更具可移植性的备份策略是将目录的内容导出到 LDIF 文件。此方法有几个明显的优点：

+   无需停止 SLAPD

+   输出更具可移植性，数据可以从一个数据库后端迁移到另一个，并且可以从一个 OpenLDAP 版本迁移到另一个版本。

数据冗余较少，因此备份文件比 BDB/HDB 文件要小得多。要创建一个仅包含一个数据库的目录服务器的 LDIF 备份文件（即它只有一个目录根），命令非常简单：

```
 $ sudo slapcat -l /usr/local/backup/my_directory.ldif

```

这个命令使用 `slapcat` 将目录的内容以 LDIF 格式导出到文件 `/usr/local/backup/my_directory.ldif`。可以使用第三章中讨论的 `slapdadd` 工具将其重新加载到目录中。

如果您的目录包含多个目录信息树，您需要对每个服务器运行一次 `slapcat` 程序，并使用 `-b` 标志来指定您要导出的目录信息树的后缀（基础 DN）：

```
 $ cd /usr/local/backup
 $ sudo slapcat -b "dc=example,dc=com" -l example_com.ldif 
 $ sudo slapcat -b "dc=test,dc=net" -l test_net.ldif

```

在这个示例中，我们将每个目录备份到各自的 LDIF 文件中。

# 重建数据库（BDB, HDB）

有时需要重建后端数据库。这个过程会根据数据库后端的不同而有所不同。例如，使用 SQL 后端时，可能需要转储、删除并重新创建数据库中的表。

### 注意

移动到新服务器并将内容传输到新从属服务器的过程也类似于重建数据库，文中提到了其中的区别。

OpenLDAP 最常用的后端是 HDB 和 BDB 后端（两者都基于 Berkeley DB 轻量级数据库）。在这一节中，我将讲解重建这些数据库的过程。

该过程包含五个步骤：

1.  停止 SLAPD

1.  将目录数据转储到文件中

1.  删除旧的目录文件

1.  创建新数据库

1.  启动 SLAPD

这些步骤都不特别困难。实际上，对于一个小型到中型的目录，这个过程可以在不到十分钟的时间内完成。

### 注意

**从服务器迁移到服务器**

将目录从一台服务器迁移到另一台服务器的过程与这里描述的非常相似。只有步骤三，如后文所述，有所不同。在这种情况下，LDIF 文件会从原始服务器传输到新服务器，而不是删除目录文件。步骤一和二会在原始服务器上执行，步骤四和五会在新服务器上完成。

## 步骤 1：停止服务器

停止服务器的目的是为了在我们处理目录信息树时，避免对其进行额外的修改。

### 注意

如果你只是将主目录的内容转储并导入到将使用 SyncRepl 的影子服务器中，你无需停止服务器。目录转储后发生的任何更改都会在影子服务器的第一次 LDAP 同步操作中被检索到。

这可以通过结束服务器的进程 ID 或运行启动脚本并加上停止命令来完成：

```
 $ sudo invoke-rc.d slapd stop

```

现在服务器已经停止，我们可以转储数据库。

## 步骤 2：转储数据库

在第三章中，我讲解了 OpenLDAP 的工具。我们讨论的一个工具是 `slapcat` 程序，它用于将目录内容转储到 LDIF 文件中。这正是我们在本步骤中使用的工具。

为什么使用 `slapcat` 而不是 `ldapsearch`？有两个原因。

首先，`slapcat` 会保留 LDAP 服务器使用的所有属性（以及记录），包括存储的操作属性。（那些在运行时生成的操作属性不会被 `slapcat` 生成，这很好。我们反正不想导入这些属性。）

其次，`slapcat` 直接访问数据库，而不是打开与服务器的 LDAP 连接。这意味着 ACL、时间和大小限制，以及 LDAP 连接的其他副产品不会被评估，因此不会更改数据。

BDB/HDB 数据库存储在位于 `/var/lib/ldap`（如果你是从源代码构建的，则为 `/usr/local/var/openldap-data`）的小文件集中。通常只有 SLAPD 用户的 ID 可以访问这些文件，默认情况下该用户为 `root` 或 `ldap`。为了使用 `slapcat` 提取信息，你需要访问这些文件。

我们有这个命令：

```
 $ sudo slapcat -l /tmp/backup.ldif

```

该命令以 root 用户身份执行 `slapcat`。`-l` 标志用于传入输出文件的名称。在这种情况下，文件 `backup.ldif` 将被创建在 `/tmp` 目录中。

### 注意

你可能更倾向于将 LDIF 文件放在 `/tmp` 以外的文件夹中，特别是如果你打算保留 LDIF 文件超过几分钟的话。

在大多数情况下，`-l` 标志是唯一需要使用的。如果你有多个后端，并且只想转储一个后端，可以使用 `-n` 标志指定要转储的后端。

一旦 `slapcat` 完成，我们就完成了这一步。

然而，在继续之前，你可能希望检查 LDIF 文件的内容，以确保它没有损坏。请在删除数据库文件之前执行此操作。

## 第 3 步：删除旧的数据库文件

如果你正在重建数据库，你需要在构建新数据库之前删除旧的数据库文件。

### 注意

如果你是从旧服务器迁移到新服务器，或者配置 SyncRepl 阴影服务器，则无需执行此操作。

这些文件存储在 `/var/lib/ldap`（如果你是从源代码构建的，则为 `/usr/local/var/openldap-data`）目录下。然而，并不是该目录中的所有文件都应该被删除。我们只想删除以下文件：

+   索引文件：以 '`.bdb`' 结尾的文件。

+   主数据库文件：以 `__db.???` 命名的文件，其中问号被顺序的数字替代（如 `__db.001`、`__db.002` 等）。

+   `alock` 文件：用于内部存储锁定信息的文件。（通常可以保持不变，不会产生负面后果，但如果 SLAPD 崩溃，这个文件可能会处于不稳定状态。）

+   BDB 日志文件：以 `log.??????????` 命名的文件，其中十个问号被顺序的数字替代：`log.0000000001`、`log.0000000002` 等。

有一个文件是我们绝对不想删除的，那就是我们的数据库配置文件 `DB_CONFIG`。删除它会导致 BDB 引擎使用默认设置，这些设置没有针对我们的需求进行调整，通常会导致 OpenLDAP 性能不佳。

所以，为了删除文件，我们可以执行以下操作：

```
$ cd /var/lib/ldap
$ sudo rm __db.* *.bdb alock log.*
```

为了减少数据丢失的风险，你可能希望在删除这些文件之前备份 `__db.*`、`*.bdb` 和 `log.*` 文件。或者，你可以使用 `mv` 命令将文件移动到其他位置，而不是使用 `rm` 删除它们：

```
$ cd /var/lib/ldap
$ sudo mkdir backup/
$ sudo mv *.bdb log.* alock __db.* backup/ 
```

现在数据库目录已经清理完毕，我们可以开始创建新的数据库文件了。

## 第 4 步：创建新数据库

新数据库可以通过使用我们在第三章中介绍的 `slapadd` 工具，在一个步骤中创建并填充数据。仍然在 OpenLDAP 数据目录下，运行以下命令：

```
 $ sudo slapadd -l /tmp/backup.ldif

```

这将创建所有必要的文件，导入 LDIF 文件，并处理所有的数据索引。

### 注意

如果你以 root 以外的用户运行 LDAP 服务器（并且这样做是个好主意），你还需要使用`chown`命令更改`/var/lib/ldap`下所有文件的所有权，使其归 SLAPD 用户 ID 所有：`sudo` `chown` `openldap` `*.bdb` `log.*` `__db.*`。

现在我们需要做的就是重启服务器。

## 第 5 步：重启 SLAPD

如果你在步骤 1 中停止了服务器，你需要重新启动它。

以常规方式之一重新启动服务器。使用初始化脚本通常是最好的方法：

```
 $ sudo invoke-rc.d slapd start

```

就是这样。现在你应该已经有了 SLAPD，并且数据库已更新。

## 故障排除重建

只要通过`slapcat`导出的 LDIF 文件没有问题，这个过程通常不会出错。即使需要删除并重新创建多次，只要 LDIF 文件安全，重要数据就不会受到威胁。

如果 SLAPD 是以非`root`用户身份运行的，那么导入过程中的主要问题通常是`/var/lib/ldap`下数据库文件的权限。`/etc/ldap`目录中的配置文件权限也可能是 SLAPD 失败的原因。确保这些文件归适当的用户所有。

在切换 OpenLDAP 版本时，偶尔旧的 LDIF 文件在新服务器中无效（这种情况发生在 OpenLDAP 2.0 和 OpenLDAP 2.2 之间，又发生在 2.2 和 2.3 之间；未来可能还会发生）。虽然标准模式在时间上相对稳定，但操作属性通常没有标准化，变化更为频繁，并且在不同版本之间会有所变化。

通常，修复方法是调整 LDIF 文件中的记录，以匹配新版本中使用的属性。另一个常见问题是与启动服务器有关。有时，在使用初始化脚本时，可能无法启动服务器，但不会向控制台或日志文件发送任何有用的消息。（启动失败的一个常见原因是我之前提到的权限问题）。

解决启动问题的一个好的第一步是从命令行运行`slapd`，并启用调试：`sudo slapd -d trace`。

# 摘要

在本附录中，我们介绍了一些有用的命令，包括一些用于获取目录服务器详细信息的命令。此外，我们还介绍了两种制作目录备份的方法，并详细探讨了重建目录数据库的过程。
