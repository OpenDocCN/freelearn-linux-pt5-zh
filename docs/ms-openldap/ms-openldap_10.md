# 附录 B. LDAP URLs

要查询目录，客户端必须向服务器发送几种不同的信息。为了将所有这些信息组合成一个符合标准的字符串格式，LDAP 开发者提出了一种标准的 LDAP URL 语法，它遵循 URL 标准（RFC 3986）。在本附录中，我们将了解 LDAP URL 的格式。

# LDAP URL

LDAP URL 由八个不同的部分组成：

1.  **协议**，通常是 LDAP（`ldap://`），尽管也使用非标准的 LDAPS 协议（`ldaps://`）。

1.  服务器的**域名**（或 IP 地址）。默认值是 `localhost`。

1.  服务器的**端口号**。默认值是标准 LDAP 端口 `389`。

1.  搜索的**基础 DN**。

1.  要返回的**属性**列表。默认情况下返回所有属性。

1.  **范围**指定符。默认使用 `base` 范围。

1.  **搜索过滤器**。默认值是 `(objectclass=*)`。

1.  **扩展**字段。如果服务器支持扩展，可以在最后的字段中传递这些扩展的参数。

结合这八个部分中的七个（我们将跳过扩展字段），我们可以创建一个类似于以下的 URL：

```
ldap://example.com:389/ou=Users,dc=example,dc=com?mail?sub?(uid=matt)
```

该 URL 由以下七个部分组成：

```
<protocol>://<domain>:<port>/<basedn>?<attrs>?<scope>?<filter>
```

当我们需要使用扩展时，我们只需在给定的 URL 末尾附加一个问号 `(?` 和扩展信息。

使用此 URL 执行 LDAP 搜索，结果如下所示：

+   客户端将使用 LDAP 协议通过端口 389 连接到 Example.Com。

+   基础 DN 将设置为 `ou=Users,dc=example,dc=com`。

+   客户端将请求 `ou=Users,dc=example,dc=com` 子树中 UID 为 `matt` 的所有条目的**mail**属性。

要使用 LDAPS（在专用 SSL/TLS 端口上使用 LDAP 的非标准做法），请使用 `ldaps://` 而不是 `ldap://`。

在许多情况下，简化 URL 并接受默认选项非常方便。例如，默认域是`localhost`（或 IP 地址 `127.0.0.1`），即 URL 执行的服务器的地址。默认端口是 389（除非协议为 `ldaps://`，而不是 `ldap://`，在这种情况下，默认端口是 LDAP 的端口 636）。

在大多数情况下，端口可以省略。但是，URL 中的域名部分也可以省略：

```
ldap:///ou=Users,dc=example,dc=com?mail?sub?(uid=matt)
```

请注意，开头现在有三个斜杠 `ldap:///`。通常出现在第二个和第三个斜杠之间的域名未指定。如果使用此 URL，LDAP 应用程序将连接到 localhost（默认主机），端口为 389（默认的 LDAP 端口），然后执行搜索。

假设现在我们不仅想让 LDAP 服务器返回 `mail` 属性，而是让它返回所有标准（非操作）属性。为此，我们只需将属性规范留空：

```
ldap:///ou=Users,dc=example,dc=com??sub?(uid=matt)
```

现在，属性位置没有值，尽管两个相邻的问号（`??`）指示空属性位置。

在前两个示例中，当我们省略了特定字段值时，必须在 URL 中保留设计符。因此，我们将 URL 的域部分写作`ldap:///`，并且在属性规范中留有没有值的`?`（在给定示例中为`??`）。

但当我们从 URL 的*末尾*去除值时，不需要保留空位置标识符。例如，如果我们去除末尾的过滤器，就不需要在 URL 末尾留下多余的`?`。以下是一个示例：

```
ldap:///ou=Users,dc=example,dc=com?mail?sub
```

在此示例中，返回了`ou=Users,dc=example,dc=com`下每个条目的`mail`属性。

# LDAP URL 的常见用途

本书中，LDAP URL 被用于各种不同的目的。

在第四章中，我们使用 LDAP URL 在`slapd.conf`中的`authz-regexp`指令中执行搜索。

正如我们所探讨的，完整的 LDAP URL 可以作为制定搜索的有用方式，但这可能并不是 LDAP URL 的主要用途。更常见的是，LDAP URL 语法被简化，只用于捕获基本信息。

## 并非所有 LDAP URL 都用于搜索

在第三章中，我们使用 LDAP URL 通过`ldapsearch`工具连接到 SLAPD，但当时我们并没有使用 LDAP URL 来指定搜索字符串。事实上，在许多情况下，LDAP URL 可能仅用于在一个方便的字符串中提供协议、主机和端口信息：

```
ldap://example.com:646
```

在此示例中，LDAP URL 提供了足够的信息，以便客户端在连接到`Example.Com`服务器时使用普通的 LDAP 协议，并且连接端口为非标准端口 646。

目录引用，在`slapd.conf`文件中由引用指令处理，也使用 LDAP URL 语法，但只使用协议、域和端口设置。

因此，LDAP URL 有两个主要用途，每个用途决定了其格式：

+   LDAP 搜索 URL 遵循复杂的八字段格式，能够传递 LDAP 代理进行搜索所需的所有信息。

+   LDAP 连接 URL 仅使用协议、主机和端口信息，主要用于传递如何连接到目录的信息。

当前没有用于修改或删除 LDAP 记录的 LDAP URL 形式。

# 有关 LDAP URL 的更多信息...

LDAP URL 格式在标准化的 RFC 4516 中进行了描述。该 RFC 充满了示例，涵盖了扩展的使用和特殊字符的编码。RFC 可以在线访问：[`rfc-editor.org/rfc/rfc4516.txt`](http://rfc-editor.org/rfc/rfc4516.txt)。

# 总结

本简短指南概述了 LDAP URL 语法。LDAP URL 被用于各种场合，提供连接信息，并且有时（在更复杂的形式下）提供执行 LDAP 搜索所需的信息。
