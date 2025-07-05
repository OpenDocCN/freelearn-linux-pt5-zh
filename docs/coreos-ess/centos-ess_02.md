# 第二章：开始使用 etcd

在本章中，我们将介绍 `etcd`，CoreOS 的服务中央枢纽，它提供了一种可靠的方式来跨集群机器存储共享数据并进行监控。

为了进行测试，我们将使用上一章中已经安装的 CoreOS 虚拟机。在本章中，我们将覆盖以下主题：

+   介绍 `etcd`

+   从主机机器读取和写入到 `etcd`

+   从应用容器读取和写入

+   观察 `etcd` 中的变化

+   TTL（生存时间）示例

+   `etcd` 的使用案例

# 介绍 etcd

`etcd` 功能是一个开源的分布式键值存储系统，运行在计算机网络上，其中信息存储在多个节点上，并通过 Raft 共识算法进行数据复制。`etcd` 功能用于存储 CoreOS 集群的服务发现和共享配置。

配置信息存储在预写日志中，包括集群成员 ID、集群 ID 和集群配置，并且可以被所有集群成员访问。

`etcd` 功能在每个集群的中央服务角色机器上运行，并在网络分区或当前主节点丢失时优雅地处理主节点选举。

# 从主机机器读取和写入到 etcd

你将学习如何从主机机器读取和写入 `etcd`。我们将在这里使用 `etcdctl` 和 `curl` 示例。

## 登录到主机

要登录到 CoreOS 虚拟机，请按以下步骤操作：

1.  启动第一章中安装的 CoreOS 虚拟机。在你的终端中输入以下命令：

    ```
    $ cdcoreos-vagrant
    $ vagrant up

    ```

1.  我们需要通过 `ssh` 登录到主机：

    ```
    $ vagrant ssh

    ```

## 读取和写入到 etcd

让我们使用 `etcdctl` 读取和写入 `etcd`。所以，请执行以下步骤：

1.  使用 `etcdctl` 设置一个名为 `message1` 的键，其值为 `Book1`：

    ```
    $ etcdctl set /message1 Book1
    Book1 (we got respond for our successful write to etcd)

    ```

1.  现在，让我们读取键值，确认一切是否正常：

    ```
    $ etcdctl get /message1
    Book1
    Perfect!

    ```

1.  接下来，让我们通过基于 HTTP 的 API 使用 `curl` 尝试做同样的操作。`curl` 功能非常方便，可以从任何可以访问 `etcd` 集群的地方访问 `etcd`，但你不想/不需要使用 `etcdctl` 客户端：

    ```
    $ curl -L -X PUT http://127.0.0.1:2379/v2/keys/message2 -d value="Book2"
    {"action":"set","key":"/message2","prevValue":"Book1","value":"Book2","index":13371}

    ```

    让我们来读取它：

    ```
    $ curl -L http://127.0.0.1:2379/v2/keys/message2
    {"action":"get","node":{"key":"/message2","value":"Book2","modifiedIndex":13371,"createdIndex":13371}}

    ```

    使用基于 HTTP 的 `etcd` API，意味着客户端应用程序可以直接读取和写入 `etcd`，无需与命令行交互。

1.  现在，如果我们想删除键值对，我们输入以下命令：

    ```
    $ etcdctl rm /message1
    $ curl -L -X DELETE http://127.0.0.1:2379/v2/keys/message2

    ```

1.  此外，我们可以向目录中添加键值对，因为当将键放入目录中时，目录会自动创建。我们只需要一个命令就能将键放入目录：

    ```
    $ etcdctl set /foo-directory/foo-key somekey

    ```

1.  现在让我们检查目录的内容：

    ```
    $ etcdctl ls /foo-directory –recursive
    /foo-directory/foo-key

    ```

1.  最后，我们可以通过输入以下命令从目录中获取键值：

    ```
    $ etcdctl get /foo-directory/foo-key
    somekey

    ```

# 从应用容器读取和写入

通常，应用容器（这是 `docker`、`rkt` 和其他类型容器的通用术语）默认情况下没有安装 `etcdctl`，甚至没有安装 `curl`。安装 `curl` 比安装 `etcdctl` 要容易得多。

对于我们的示例，我们将使用 Alpine Linux 的 docker 镜像，它非常小，不会花费太多时间从`docker`注册表中拉取：

1.  首先，我们需要检查`docker0`接口的 IP 地址，稍后我们将使用该地址与`curl`配合使用：

    ```
    $ echo"$(ifconfig docker0 | awk'/\<inet\>/ { print $2}'):2379"
    10.1.42.1:2379

    ```

1.  让我们使用`bash` shell 运行`docker`容器：

    ```
    $ docker run -it alpine ash

    ```

    在命令提示符中我们应该看到类似于`/ #`的内容：

1.  由于 Alpine Linux 默认没有安装`curl`，我们需要安装它：

    ```
    $ apk update&&apk add curl
    $ curl -L http://10.1.42.1:2379/v2/keys/
    {"action":"get","node":{"key":"/","dir":true,"nodes":[{"key":"/coreos.com","dir":true,"modifiedIndex":3,"createdIndex":3}]}}

    ```

1.  重复上一个小节中的第 3 步和第 4 步，这样你就能明白无论你从哪里连接到`etcd`，`curl`的工作方式始终是一样的。

1.  按*Ctrl* +*D*退出`docker`容器。

# 监视`etcd`中的变化

这次，让我们监视`etcd`中键值的变化。监视键值变化在以下情况下非常有用：例如，有一个`fleet`单元，它将`nginx`的端口写入`etcd`，而另一个反向代理应用程序则监视变化并更新其配置：

1.  我们需要先在`etcd`中创建一个目录：

    ```
    $ etcdctlmkdir /foo-data

    ```

1.  接下来，我们监视此目录中的变化：

    ```
    $ etcdctl watch /foo-data--recursive

    ```

1.  现在，在新的终端窗口中打开另一个 CoreOS shell：

    ```
    $ cdcoreos-vagrant
    $ vagrantssh

    ```

1.  我们在`/foo-data`目录下添加一个新键：

    ```
    $ etcdctl set /foo-data/Book is_cool

    ```

1.  在第一个终端中，我们应该看到一条通知，提示键值已被更改：

    ```
    is_cool

    ```

# TTL（生存时间）示例

有时，为键设置**生存时间**（**TTL**）以便在一定时间后过期是非常方便的。例如，当监视一个 TTL 为 60 秒的键时，反向代理就会用到 TTL。因此，如果`nginx fleet`服务没有更新该键，它将在 60 秒内过期并从`etcd`中删除。然后，反向代理检查时未找到该键，因此会将`nginx`服务从`config`中移除。

让我们在这个示例中设置 TTL 为 30 秒：

1.  在终端中输入以下内容：

    ```
    $ etcdctl set /foo "I'm Expiring in 30 sec" --ttl 30
    I'm Expiring in 30 sec

    ```

1.  验证该键是否仍然存在：

    ```
    $ etcdctl get /foo
    I'm Expiring in 30 sec

    ```

1.  30 秒后再次检查：

    ```
    $ etcdctl get /foo

    ```

1.  如果您请求的键已经过期，您将收到`错误`：`100`：

    ```
    Error: 100: Key not found (/foo) [17053]

    ```

这次，`etcd`删除了该键，因为我们为它设置了 30 秒的 TTL。

### 注意

TTL 在使用`etcd`作为检查点进行不同服务之间的通信时非常方便。

# `etcd`的使用案例

运行在工作节点上的应用程序容器，可以在代理模式下读取和写入`etcd`集群。

`etcd`的常见使用案例如下：存储数据库连接设置、缓存设置和共享设置。例如，Vulcand 代理服务器([`vulcanproxy.com/`](http://vulcanproxy.com/))使用`etcd`存储 Web 主机连接详细信息，这些信息对所有集群连接的工作节点机器可用。另一个例子是存储 MySQL 的数据库密码，并在运行应用程序容器时检索它。

在接下来的章节中，我们将详细讨论集群设置、中央服务和工作节点角色机器。

# 总结

在这一简短的章节中，我们介绍了`etcd`的基础知识，以及如何读取和写入`etcd`，监视`etcd`中的变化，并使用 TTL 设置`etcd`键的生存时间。

在下一章中，你将学习如何使用`systemd`和`fleet`单元。
