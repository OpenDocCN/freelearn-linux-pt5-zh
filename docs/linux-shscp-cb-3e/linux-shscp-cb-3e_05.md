# 错综复杂的 Web？一点也不！

在本章中，我们将介绍以下方法：

+   从网页下载

+   将网页下载为纯文本

+   cURL 入门

+   从命令行访问未读的 Gmail 邮件

+   从网站解析数据

+   图片爬虫和下载器

+   Web 照片相册生成器

+   Twitter 命令行客户端

+   通过 Web 服务器访问单词定义

+   查找网站中的断开链接

+   跟踪网站的变化

+   向网页发布数据并读取响应

+   从互联网下载视频

+   使用 OTS 摘要文本

+   从命令行翻译文本

# 介绍

Web 已经成为技术的面貌，也是数据处理的中央访问点。Shell 脚本无法完成 PHP 等语言在 Web 上能够实现的所有功能，但对于许多任务，Shell 脚本是理想的选择。我们将探讨一些方法，下载和解析网站数据，向表单发送数据，自动化网站使用任务以及类似的活动。通过几行脚本，我们可以自动化许多通过浏览器交互执行的活动。HTTP 协议和命令行工具提供的功能使我们能够编写脚本来解决许多 Web 自动化需求。

# 从网页下载

下载文件或网页非常简单。有一些命令行下载工具可以完成这个任务。

# 准备就绪

`wget` 是一个灵活的文件下载命令行工具，可以通过许多选项进行配置。

# 如何操作...

可以使用 `wget` 下载网页或远程文件：

```
$ wget URL

```

例如：

```
$ wget knopper.net
--2016-11-02 21:41:23--  http://knopper.net/
Resolving knopper.net... 85.214.68.145
Connecting to knopper.net|85.214.68.145|:80...
connected.
HTTP request sent, awaiting response... 200 OK
Length: 6899 (6.7K) [text/html]
Saving to: "index.html.1"

100% [=============================]45.5K=0.1s

2016-11-02 21:41:23 (45.5 KB/s) - "index.html.1" saved
[6899/6899]

```

也可以指定多个下载 URL：

```
$ wget URL1 URL2 URL3 ..

```

# 它是如何工作的...

默认情况下，下载的文件名称与 URL 相同，下载信息和进度将写入 `stdout`。

`-O` 选项指定输出文件名。如果已存在该名称的文件，它将被下载的文件替换：

```
$ wget http://www.knopper.net -O knopper.html.

```

`-o` 选项指定一个 `logfile`，而不是将日志打印到 `stdout`：

```
$ wget ftp://ftp.example.com/somefile.img -O dloaded_file.img -o log

```

使用上述命令不会在屏幕上打印任何内容。日志或进度将写入日志，输出文件将是 `dloaded_file.img`。

由于互联网连接不稳定，下载可能会中断。`-t` 选项指定工具在放弃之前重试的次数：

```
$ wget -t 5 URL

```

使用 `0` 值强制 `wget` 无限次尝试：

```
$ wget -t 0 URL

```

# 还有更多...

`wget` 工具提供了选项来微调行为并解决问题。

# 限制下载速度

当带宽有限且许多应用程序共享带宽时，一个大文件可能会占用所有带宽，导致其他进程（可能是交互式用户）无法访问。`wget` 选项 `-limit-rate` 将指定下载任务的最大带宽，允许所有应用程序公平地访问互联网：

```
$ wget  --limit-rate 20k http://example.com/file.iso

```

在此命令中，`k`（千字节）指定了速度限制。你也可以使用 `m` 表示兆字节。

`-quota`（或`-Q`）选项指定下载的最大大小。当超过配额时，`wget`会停止。这在下载多个文件到空间有限的系统时很有用：

```
$ wget -Q 100m http://example.com/file1 http://example.com/file2

```

# 重新开始下载并继续

如果`wget`在下载完成之前被中断，可以使用`-c`选项从中断处恢复下载：

```
$ wget -c URL

```

# 复制整个网站（镜像）

`wget`可以通过递归地收集 URL 链接并像爬虫一样下载它们，来下载完整的网站。要下载页面，请使用`--mirror`选项：

```
$ wget --mirror --convert-links exampledomain.com

```

或者，使用以下命令：

```
$ wget -r -N -l -k DEPTH URL

```

`-l`选项指定网页的深度，以级别表示。这意味着它只会遍历该级别的数量。它与`-r`（递归）一起使用。`-N`参数用于启用文件的时间戳功能。`URL`是需要发起下载的站点的基本 URL。`-k`或`--convert-links`选项指示`wget`将其他页面的链接转换为本地副本。

在镜像其他网站时要谨慎。除非你获得许可，否则仅为个人使用并避免频繁进行此操作。

# 使用 HTTP 或 FTP 身份验证访问页面

`--user`和`--password`参数提供需要身份验证的网站的用户名和密码。

```
$ wget --user username --password pass URL

```

也可以在不直接指定密码的情况下请求密码。为此，请使用`--ask-password`而不是`--password`参数。

# 将网页下载为纯文本

网页只是包含 HTML 标签、JavaScript 和 CSS 的文本。HTML 标签定义了网页的内容，我们可以解析这些内容以提取特定信息。Bash 脚本可以解析网页。HTML 文件可以在网页浏览器中查看以正确显示格式，或使用前一章中描述的工具进行处理。

解析文本文档比解析 HTML 数据更简单，因为我们不需要去除 HTML 标签。**Lynx**是一个命令行网页浏览器，可以将网页下载为纯文本。

# 准备工作

Lynx 并非所有发行版都预装，但可以通过包管理器安装。

```
# yum install lynx

```

或者，你可以执行以下命令：

```
 apt-get install lynx

```

# 如何做...

`-dump`选项将网页下载为纯 ASCII 文本。下一个示例展示了如何将该 ASCII 版本的页面发送到文件中：

```
$ lynx URL -dump > webpage_as_text.txt

```

此命令将在`References`标题下单独列出所有超链接（`<a href="link">`），作为文本输出的页脚。这使我们可以使用正则表达式单独解析链接。

考虑以下示例：

```
$lynx -dump http://google.com > plain_text_page.txt

```

你可以使用`cat`命令查看`text`的纯文本版本：

```
    $ cat plain_text_page.txt
 Search [1]Images [2]Maps [3]Play [4]YouTube [5]News [6]Gmail   
    [7]Drive
 [8]More »
 [9]Web History | [10]Settings | [11]Sign in

 [12]St. Patrick's Day 2017

 _______________________________________________________
 Google Search  I'm Feeling Lucky    [13]Advanced search
 [14]Language tools

 [15]Advertising Programs     [16]Business Solutions     [17]+Google
 [18]About Google

 © 2017 - [19]Privacy - [20]Terms

References
...

```

# cURL 基础

**cURL** 使用 HTTP、HTTPS 或 FTP 协议在服务器与本地之间传输数据。它支持 `POST`、cookies、身份验证、从指定偏移位置下载部分文件、referer、用户代理字符串、额外的头部、限速、最大文件大小、进度条等功能。cURL 对于维护网站、检索数据以及检查服务器配置非常有用。

# 准备工作

与 `wget` 不同，cURL 并不是所有 Linux 发行版都自带的；你可能需要使用包管理器进行安装。

默认情况下，cURL 会将下载的文件输出到 `stdout`，并将进度信息输出到 `stderr`。若要禁用显示进度信息，请使用 `--silent` 选项。

# 如何操作...

`curl` 命令执行多种功能，包括下载、发送不同的 HTTP 请求和指定 HTTP 头部。

+   要将下载的文件输出到 `stdout`，请使用以下命令：

```
        $ curl URL

```

+   `-O` 选项指定将下载的数据发送到一个文件，该文件名是从 URL 解析出来的。注意，URL 必须是完整的页面 URL，而不仅仅是站点名称。

```
        $ curl www.knopper.net/index.htm --silent -O

```

+   `-o` 选项指定输出文件的名称。使用此选项，你可以仅指定站点名称以获取主页。

```
        $curl www.knopper.net -o knoppix_index.html
 % Total % Received % Xferd  Avg  Speed Time   Time  Time  
        Current
 Dload Upload Total Spent Left  Speed
 100 6889 100 6889  0 0     10902  0     --:-- --:-- --:-- 26033

```

+   `-silent` 选项可以防止 `curl` 命令显示进度信息：

```
        $ curl URL --silent

```

+   `-progress` 选项在下载时显示进度条：

```
        $ curl http://knopper.net -o index.html --progress
 ################################## 100.0% 

```

# 它是如何工作的...

cURL 下载网页或远程文件到本地系统。你可以使用 `-O` 和 `-o` 选项控制目标文件名，使用 `-silent` 和 `-progress` 选项控制冗余信息的显示。

# 还有更多...

在前面的章节中，你学习了如何下载文件。cURL 支持更多的选项，可以细化其行为。

# 继续和恢复下载

cURL 可以从指定的偏移位置恢复下载。如果你有每日数据限制并且需要下载大文件，这个功能非常有用。

```
$ curl URL/file -C offset

```

偏移量是一个字节数的整数值。

如果我们想恢复下载一个文件，cURL 不要求我们知道确切的字节偏移位置。如果你希望 cURL 自动计算正确的恢复点，可以使用 `-C -` 选项，如下所示：

```
$ curl -C - URL

```

cURL 会自动计算从指定文件的哪里重新开始下载。

# 使用 cURL 设置 referer 字符串

**Referer** 字段在 HTTP 头部中标识了引导当前网页的页面。当用户点击网页 A 上的链接跳转到网页 B 时，页面 B 的 referer 头部字符串将包含页面 A 的 URL。

一些动态页面在返回 HTML 数据之前会检查 referer 字符串。例如，一个网页可能会在用户从 Google 导航到某个网站时显示 Google 的徽标，而当用户直接输入 URL 时，则显示不同的页面。

一个 web 开发者可以编写一个条件语句，如果 referer 是 www.google.com，就返回一个 Google 页面；如果不是，则返回不同的页面。

你可以使用 `--referer` 与 `curl` 命令来指定 referer 字符串，如下所示：

```
$ curl --referer Referer_URL target_URL

```

考虑以下示例：

```
$ curl --referer http://google.com http://knopper.org

```

# 使用 cURL 处理 cookies

`curl` 可以指定并存储在 HTTP 操作过程中遇到的 cookies。

`-cookie``COOKIE_IDENTIFER` 选项指定要提供的 Cookies。Cookies 的格式为 `name=value`。多个 Cookies 应该用分号（`;`）分隔：

```
$ curl http://example.com --cookie "user=username;pass=hack"

```

`-cookie-jar` 选项指定用于存储 Cookies 的文件：

```
$ curl URL --cookie-jar cookie_file

```

# 使用 cURL 设置用户代理字符串

一些检查用户代理的网页，如果没有指定用户代理，可能无法正常工作。例如，某些旧网站要求使用 **Internet Explorer**（**IE**）。如果使用其他浏览器，它们会显示一个消息，提示该网站必须在 IE 中查看。这是因为该网站检查用户代理。你可以通过 `curl` 设置用户代理。

`--user-agent` 或 `-A` 选项设置用户代理：

```
$ curl URL --user-agent "Mozilla/5.0"

```

可以通过 cURL 传递额外的头部信息。使用 `-H "Header"` 传递额外的头部：

```
$ curl -H "Host: www.knopper.net" -H "Accept-language: en" URL

```

网上有许多不同的用户代理字符串，涵盖多个浏览器和爬虫。你可以在 [`www.useragentstring.com/pages/useragentstring.php`](http://www.useragentstring.com/pages/useragentstring.php) 找到一些常见的列表。

# 在 cURL 中指定带宽限制

当带宽被多个用户共享时，我们可以使用 `--limit-rate` 选项来限制下载速率：

```
$ curl URL --limit-rate 20k

```

可以使用 `k`（千字节）或 `m`（兆字节）来指定速率。

# 指定最大下载大小

`--max-filesize` 选项指定最大文件大小：

```
$ curl URL --max-filesize bytes

```

如果文件大小超过限制，`curl` 命令将返回一个非零退出代码；如果下载成功，则返回零。

# 使用 cURL 进行身份验证

`curl` 命令的 `-u` 选项执行 HTTP 或 FTP 身份验证。

用户名和密码可以使用 `-u username:password` 进行指定：

```
$ curl -u user:pass http://test_auth.com

```

如果你希望系统提示输入密码，只提供用户名即可：

```
$ curl -u user http://test_auth.com 

```

# 打印响应头部，排除数据部分

检查头部信息对于许多检查和统计已足够。例如，我们不需要下载整个页面来确认它是否可达。只读取 HTTP 响应即可。

另一个检查 HTTP 头部的使用场景是检查 `Content-Length` 字段以确定文件大小，或检查 `Last-Modified` 字段来查看文件是否比当前副本更新，然后再进行下载。

`-I` 或 `-head` 选项仅输出 HTTP 头部，不下载远程文件：

```
$ curl -I http://knopper.net 
HTTP/1.1 200 OK
Date: Tue, 08 Nov 2016 17:15:21 GMT
Server: Apache
Last-Modified: Wed, 26 Oct 2016 23:29:56 GMT
ETag: "1d3c8-1af3-b10500"
Accept-Ranges: bytes
Content-Length: 6899
Content-Type: text/html; charset=ISO-8859-1

```

# 另见

+   本章的 *发布到网页并读取响应* 示例

# 从命令行访问未读 Gmail 邮件

Gmail 是 Google 提供的一款广泛使用的免费电子邮件服务：[`mail.google.com/`](http://mail.google.com/)。它允许你通过浏览器或经过身份验证的 RSS 提要来读取邮件。我们可以解析这些 RSS 提要，报告发件人名称和主题。这是一种快速扫描未读邮件而无需打开网页浏览器的方法。

# 如何做到...

让我们通过一个 shell 脚本解析 Gmail 的 RSS 提要，来显示未读邮件：

```
#!/bin/bash 
#Desc: Fetch gmail tool 

username='PUT_USERNAME_HERE' 
password='PUT_PASSWORD_HERE' 

SHOW_COUNT=5 # No of recent unread mails to be shown 

echo 
curl -u $username:$password --silent \
    "https://mail.google.com/mail/feed/atom" | \
     tr -d '\n' | sed 's:</entry>:\n:g' |\ 
     sed -n 's/.*<title>\(.*\)<\/title.*<author><name>\([^<]*\)<\/name><email>
 \([^<]*\).*/From: \2 [\3] \nSubject: \1\n/p' | \ 
head -n $(( $SHOW_COUNT * 3 ))

```

输出结果如下：

```
$ ./fetch_gmail.sh
From: SLYNUX [ slynux@slynux.com ]
Subject: Book release - 2

From: SLYNUX [ slynux@slynux.com ]
Subject: Book release - 1 
.
... 5 entries

```

如果你使用带有两步验证的 Gmail 账户，你需要为此脚本生成一个新密钥并使用它。你的常规密码将无法使用。

# 它是如何工作的…

脚本使用 cURL 下载 RSS 订阅源。你可以通过登录 Gmail 帐户并查看[`mail.google.com/mail/feed/atom`](https://mail.google.com/mail/feed/atom)来查看传入数据的格式。

cURL 通过`-u user:pass`参数提供的用户认证读取 RSS 订阅源。当你仅使用`-u user`而不提供密码时，cURL 会交互式地询问密码。

+   `tr -d '\n'`：这会删除换行符

+   `sed 's:</entry>:\n:g'`：这将每个`</entry>`元素替换为换行符，因此每个电子邮件条目都由新行分隔，从而使邮件可以逐一解析。

需要作为一个单一表达式执行的下一个脚本块使用`sed`提取相关字段：

```
 sed 's/.*<title>\(.*\)<\/title.*<author><name>\([^<]*\)<\/name><email>
 \([^<]*\).*/Author: \2 [\3] \nSubject: \1\n/' 

```

该脚本使用`<title>\(.*\)<\/title`正则表达式匹配标题，使用`<author><name>\([^<]*\)<\/name>`正则表达式匹配发送者名称，使用`<email>\([^<]*\)`匹配电子邮件。Sed 使用反向引用将作者、标题和邮件主题显示为易于阅读的格式：

```
Author: \2 [\3] \nSubject: \1\n 

```

`\1`对应第一个子字符串匹配（标题），`\2`对应第二个子字符串匹配（姓名），依此类推。

`SHOW_COUNT=5`变量用于获取要在终端上打印的未读邮件条目数量。

`head`用于显示从第一行开始的`SHOW_COUNT*3`行。`SHOW_COUNT`乘以三，以便显示三行输出。

# 另见

+   本章中的*cURL 入门*配方解释了`curl`命令

+   第四章中的*使用 sed 进行文本替换*配方解释了`sed`命令

# 从网站解析数据

`lynx`、`sed`和`awk`命令可以用于从网站提取数据。你可能在第四章的*使用 grep 搜索和提取文件中的文本*配方中看到过一个女演员排名列表；它是通过解析[`www.johntorres.net/BoxOfficefemaleList.html`](http://www.johntorres.net/BoxOfficefemaleList.html)网页生成的。

# 如何实现…

让我们来看看用于解析网站上女演员详细信息的命令：

```
$ lynx -dump -nolist \
    http://www.johntorres.net/BoxOfficefemaleList.html
    grep -o "Rank-.*" | \
    sed -e 's/ *Rank-\([0-9]*\) *\(.*\)/\1\t\2/' | \
    sort -nk 1 > actresslist.txt 

```

输出如下：

```
# Only 3 entries shown. All others omitted due to space limits
1   Keira Knightley 
2   Natalie Portman 
3   Monica Bellucci 

```

# 它是如何工作的…

Lynx 是一个命令行网页浏览器；它可以像在网页浏览器中看到的那样，转储网站的文本版本，而不是返回像`wget`或 cURL 那样的原始 HTML。这省去了去除 HTML 标签的步骤。`-nolist`选项显示没有编号的链接。使用`sed`对包含排名的行进行解析和格式化：

```
sed -e 's/ *Rank-\([0-9]*\) *\(.*\)/\1\t\2/'

```

这些行随后会根据排名进行排序。

# 另见

+   第四章中的*使用 sed 进行文本替换*配方解释了`sed`命令

+   本章中的*下载网页为纯文本*配方解释了`lynx`命令

# 图像爬虫和下载器

**图片爬虫** 会下载网页中出现的所有图片。我们可以通过脚本自动识别图片并下载，而不必手动从 HTML 页面中挑选图片。

# 如何实现...

这个 Bash 脚本将识别并下载网页中的所有图片：

```
#!/bin/bash 
#Desc: Images downloader 
#Filename: img_downloader.sh 

if [ $# -ne 3 ];
then
  echo "Usage: $0 URL -d DIRECTORY"
  exit -1
fi
while [ $# -gt 0 ]
do
  case $1 in
  -d) shift; directory=$1; shift ;;
  *) url=$1; shift;;
  esac
done

mkdir -p $directory;
baseurl=$(echo $url | egrep -o "https?://[a-z.\-]+")

echo Downloading $url 
curl -s $url | egrep -o "<img[^>]*src=[^>]*>" | \
  sed 's/<img[^>]*src=\"\([^"]*\).*/\1/g' | \
  sed "s,^/,$baseurl/," > /tmp/$$.list

cd $directory;

while read filename;
do
  echo Downloading $filename
  curl -s -O "$filename" --silent
done < /tmp/$$.list

```

示例用法如下：

```
$ url=https://commons.wikimedia.org/wiki/Main_Page
$ ./img_downloader.sh $url -d images

```

# 它是如何工作的...

图片下载脚本读取 HTML 页面，去除所有标签（除了 `<img>` 标签），解析 `<img>` 标签中的 `src="img/URL"` 并将其下载到指定目录。此脚本接受网页 URL 和目标目录作为命令行参数。

`[ $# -ne 3 ]` 语句检查脚本的参数总数是否为三，如果不是，则退出并返回使用示例。否则，这段代码会解析 URL 和目标目录：

```
while [ -n "$1" ] 
do  
  case $1 in 
  -d) shift; directory=$1; shift ;; 
   *) url=${url:-$1}; shift;; 
esac 
done 

```

`while` 循环会一直执行，直到所有参数处理完成。`shift` 命令会将参数向左移动，这样 `$1` 就会获取下一个参数的值，即 `$2`，以此类推。因此，我们可以通过 `$1` 本身评估所有参数。

`case` 语句检查第一个参数（`$1`）。如果它匹配 `-d`，那么下一个参数必须是目录名，因此参数会被移位，目录名将被保存。如果该参数是其他任何字符串，则它是 URL。

这种解析参数的优势在于，我们可以在命令行中的任何位置放置 `-d` 参数：

```
$ ./img_downloader.sh -d DIR URL

```

或：

```
$ ./img_downloader.sh URL -d DIR

```

`egrep -o "<img src=[^>]*>"` 只会打印匹配的字符串，即包括其属性的 `<img>` 标签。`[^>]*` 这个短语匹配所有除关闭 `>` 之外的字符，即 `<img src="img/image.jpg">`。

`sed's/<img src=\"\([^"]*\).*/\1/g'` 从 `src="img/url"` 字符串中提取 `url`。

图像源路径有两种类型：绝对路径和相对路径。**绝对路径** 包含以 `http://` 或 `https://` 开头的完整 URL。相对 URL 以 `/` 或 `image_name` 本身开头。一个绝对 URL 的例子是 `http://example.com/image.jpg`。一个相对 URL 的例子是 `/image.jpg`。

对于相对 URL，起始的 `/` 应该被替换为基础 URL，从而将其转换为 `http://example.com/image.jpg`。该脚本通过以下命令提取初始 URL 来初始化 `baseurl`：

```
baseurl=$(echo $url | egrep -o "https?://[a-z.\-]+") 

```

前述 `sed` 命令的输出被传递到另一个 sed 命令，替换掉开头的 `/` 为 `baseurl`，并将结果保存在以脚本的 PID 命名的文件中：（`/tmp/$$.list`）。

```
sed "s,^/,$baseurl/," > /tmp/$$.list 

```

最终的 `while` 循环会遍历列表中的每一行，并使用 curl 下载图片。使用 `--silent` 参数来避免 `curl` 输出多余的进度信息。

# 另见

+   本章中的*cURL 入门* 食谱解释了 `curl` 命令。

+   第四章中的*使用 sed 执行文本替换*食谱解释了 `sed` 命令。

+   第四章中的 *使用 grep 搜索和提取文件中的文本* 食谱，*文字和驾驶*，解释了`grep`命令

# 网络照片相册生成器

Web 开发人员常常创建全尺寸和缩略图照片相册。当点击缩略图时，会显示该图片的放大版。这需要调整大小并放置许多图像。这些操作可以通过简单的 Bash 脚本来自动化。该脚本创建缩略图，将其放置在正确的目录中，并自动生成`<img>`标签的代码片段。

# 准备工作

这个脚本使用`for`循环来遍历当前目录中的每个图像。它使用了常见的 Bash 工具，如`cat`和`convert`（来自 Image Magick 包）。这些工具将生成一个 HTML 相册，并将所有图像放入`index.html`中。

# 如何做...

这个 Bash 脚本将生成一个 HTML 相册页面：

```
#!/bin/bash 
#Filename: generate_album.sh 
#Description: Create a photo album using images in current directory 

echo "Creating album.." 
mkdir -p thumbs 
cat <<EOF1 > index.html 
<html> 
<head> 
<style> 

body  
{  
  width:470px; 
  margin:auto; 
  border: 1px dashed grey; 
  padding:10px;  
}  

img 
{  
  margin:5px; 
  border: 1px solid black; 

}  
</style> 
</head> 
<body> 
<center><h1> #Album title </h1></center> 
<p> 
EOF1 

for img in *.jpg; 
do  
  convert "$img" -resize "100x" "thumbs/$img" 
  echo "<a href=\"$img\" >" >>index.html 
  echo "<img src=\"thumbs/$img\" title=\"$img\" /></a>" >> index.html 
done 

cat <<EOF2 >> index.html 

</p> 
</body> 
</html> 
EOF2  

echo Album generated to index.html 

```

按如下方式运行脚本：

```
$ ./generate_album.sh
Creating album..
Album generated to index.html

```

# 它是如何工作的...

脚本的初始部分用于编写 HTML 页面头部部分。

以下脚本将所有内容重定向到 `EOF1` 之前的部分，并写入`index.html`：

```
cat <<EOF1 > index.html 
contents... 
EOF1 

```

头部包括 HTML 和 CSS 样式。

`for img in *.jpg *.JPG;` 会遍历文件名并执行循环体的内容。

`convert "$img" -resize "100x" "thumbs/$img"`将创建宽度为 100px 的缩略图。

以下语句生成所需的`<img>`标签并将其附加到`index.html`中：

```
echo "<a href=\"$img\" >" 
echo "<img src=\"thumbs/$img\" title=\"$img\" /></a>" >> index.html 

```

最后，使用`cat`将页脚的 HTML 标签附加到`index.html`中，就像脚本的第一部分那样。

# 另请参见

+   本章中的 *Web 照片相册生成器* 食谱解释了`EOF`和`stdin`重定向

# Twitter 命令行客户端

**Twitter** 是最热门的微博平台，也是目前在线社交媒体的流行词汇。我们可以使用 Twitter API 从命令行读取我们时间线上的推文！

让我们看看怎么做。

# 准备工作

最近，Twitter 停止允许人们使用纯 HTTP 身份验证登录，因此我们必须使用 OAuth 来进行身份验证。本书不涉及 OAuth 的完整解释，因此我们将使用一个库，使得在 Bash 脚本中使用 OAuth 变得更加简单。请按照以下步骤进行操作：

1.  从 [`github.com/livibetter/bash-oauth/archive/master.zip`](https://github.com/livibetter/bash-oauth/archive/master.zip) 下载 `bash-oauth` 库，并将其解压到任何目录。

1.  进入该目录，然后在子目录`bash-oauth-master`内，以 root 用户身份运行`make install-all`。

1.  访问 [`apps.twitter.com/`](https://apps.twitter.com/) 并注册一个新应用。这将使得能够使用 OAuth。

1.  注册新应用程序后，进入应用程序设置并将访问类型更改为“读取和写入”。

1.  现在，进入应用程序的详情部分，记录下两个信息，Consumer Key 和 Consumer Secret，以便在我们将要编写的脚本中替换这些内容。

太好了，现在让我们编写一个使用这个功能的脚本。

# 如何做...

这个 Bash 脚本使用 OAuth 库来读取推文或发送自己的更新：

```
#!/bin/bash 
#Filename: twitter.sh 
#Description: Basic twitter client 

oauth_consumer_key=YOUR_CONSUMER_KEY 
oauth_consumer_scret=YOUR_CONSUMER_SECRET 

config_file=~/.$oauth_consumer_key-$oauth_consumer_secret-rc  

if [[ "$1" != "read" ]] && [[ "$1" != "tweet" ]]; 
then  
  echo -e "Usage: $0 tweet status_message\n   OR\n      $0 read\n" 
  exit -1; 
fi 

#source /usr/local/bin/TwitterOAuth.sh 
source bash-oauth-master/TwitterOAuth.sh 
TO_init 

if [ ! -e $config_file ]; then 
 TO_access_token_helper 
 if (( $? == 0 )); then 
   echo oauth_token=${TO_ret[0]} > $config_file 
   echo oauth_token_secret=${TO_ret[1]} >> $config_file 
 fi 
fi 

source $config_file 

if [[ "$1" = "read" ]]; 
then 
TO_statuses_home_timeline '' 'YOUR_TWEET_NAME' '10' 
  echo $TO_ret |  sed  's/,"/\n/g' | sed 's/":/~/' | \ 
    awk -F~ '{} \ 
      {if ($1 == "text") \ 
        {txt=$2;} \ 
       else if ($1 == "screen_name") \ 
        printf("From: %s\n Tweet: %s\n\n", $2, txt);} \ 
      {}' | tr '"' ' ' 

elif [[ "$1" = "tweet" ]]; 
then  
  shift 
  TO_statuses_update '' "$@" 
  echo 'Tweeted :)' 
fi 

```

按照以下方式运行脚本：

```
$./twitter.sh read
Please go to the following link to get the PIN:   
https://api.twitter.com/oauth/authorize?
oauth_token=LONG_TOKEN_STRING
PIN: PIN_FROM_WEBSITE
Now you can create, edit and present Slides offline.
- by A Googler 
$./twitter.sh tweet "I am reading Packt Shell Scripting Cookbook"
Tweeted :)
$./twitter.sh read | head -2
From: Clif Flynt
Tweet: I am reading Packt Shell Scripting Cookbook 

```

# 它是如何工作的……

首先，我们使用 `source` 命令来引入 `TwitterOAuth.sh` 库，以便使用其功能访问 Twitter。`TO_init` 函数初始化该库。

每个应用程序在首次使用时都需要获取 OAuth 令牌和令牌密钥。如果没有这些信息，我们将使用 `TO_access_token_helper` 库函数来获取它们。获取到令牌后，我们将它们保存到一个 `config` 文件中，这样下次运行脚本时，只需简单地加载该文件即可。

`TO_statuses_home_timeline` 库函数从 Twitter 获取推文。这些数据以 JSON 格式作为一长串字符串返回，类似这样开始：

```
{"created_at":"Thu Nov 10 14:45:20 +0000    
"016","id":7...9,"id_str":"7...9","text":"Dining...

```

每条推文都以 `"created_at"` 标签开始，并包含 `text` 和 `screen_name` 标签。脚本将提取文本和屏幕名称数据，并仅显示这些字段。

脚本将长字符串分配给 `TO_ret` 变量。

JSON 格式使用双引号括起键，并且键值可能会被引用或不被引用。键值对通过逗号分隔，键和值通过冒号（`:`）分隔。

第一个 `sed` 命令将每个 `"` 字符替换为换行符，将每个键值对分开。这些行被传送到另一个 `sed` 命令，替换每个 `":` 的出现为波浪号 (~)，从而生成类似这样的行：

```
screen_name~"Clif_Flynt" 

```

最终的 `awk` 脚本读取每一行。`-F~` 选项将每行按波浪号分成字段，因此 `$1` 是键，`$2` 是值。`if` 命令检查是否为 `text` 或 `screen_name`。文本通常出现在推文的第一部分，但如果先显示发送者会更容易阅读，因此脚本会先保存 `text` 的返回值，直到它看到 `screen_name`，然后打印出 `$2` 的当前值和保存的文本值。

`TO_statuses_update` 库函数生成一条推文。空的第一个参数将我们的消息定义为默认格式，消息是第二个参数的一部分。

# 另见

+   [第四章《发短信与开车》中的 *使用 sed 进行文本替换* 配方，解释了 `sed` 命令。

+   在第四章《发短信与开车》中的 *使用 grep 在文件中搜索和挖掘文本* 配方，解释了 `grep` 命令。

# 通过 web 服务器访问单词定义

网上有多个词典提供 API，可以通过脚本与其网站进行交互。本配方演示了如何使用一个流行的 API。

# 准备工作

我们将使用 `curl`、`sed` 和 `grep` 来定义这个实用程序。有许多词典网站，您可以注册并免费使用它们的 API 进行个人使用。在这个示例中，我们使用的是 Merriam-Webster 的字典 API。请按照以下步骤操作：

1.  访问[`www.dictionaryapi.com/register/index.htm`](http://www.dictionaryapi.com/register/index.htm)，并为自己注册一个账户。选择《大学词典》和《学习者词典》：

1.  使用新创建的账户登录，进入“我的密钥”页面，获取密钥。记下学习者词典的密钥。

# 如何操作...

此脚本将显示一个单词的定义：

```
#!/bin/bash 
#Filename: define.sh 
#Desc: A script to fetch definitions from dictionaryapi.com 

key=YOUR_API_KEY_HERE 

if  [ $# -ne 2 ]; 
then 
  echo -e "Usage: $0 WORD NUMBER" 
  exit -1; 
fi 

curl --silent \
http://www.dictionaryapi.com/api/v1/references/learners/xml/$1?key=$key | \ 
  grep -o \<dt\>.*\</dt\> | \ 
  sed 's$</*[a-z]*>$$g' | \ 
  head -n $2 | nl 

```

运行脚本的方法如下：

```
    $ ./define.sh usb 1
 1  :a system for connecting a computer to another device (such as   
    a printer, keyboard, or mouse) by using a special kind of cord a   
    USB cable/port USB is an abbreviation of "Universal Serial Bus."How 
    it works...

```

# 工作原理...

我们使用`curl`从字典 API 网页获取数据，指定我们的 API `Key ($apikey)`和我们想要查询的单词（`$1`）。结果包含在`<dt>`标签中的定义，使用`grep`提取。`sed`命令用于去除标签。脚本从定义中选择所需的行数，并使用`nl`为每行添加行号。

# 另见

+   第四章中的*使用 sed 进行文本替换*示例解释了`sed`命令。

+   第四章中的*使用 grep 在文件中搜索和挖掘文本*示例解释了`grep`命令。

# 查找网站中的失效链接

网站必须进行失效链接测试。对于大型网站，手动完成这个任务是不可行的。幸运的是，这是一个可以轻松自动化的任务。我们可以使用 HTTP 操作工具来查找失效链接。

# 准备工作

我们可以使用`lynx`和`curl`来识别链接并找出失效的链接。Lynx 具有`-traversal`选项，可以递归访问网站上的页面，并构建所有超链接的列表。cURL 用于验证每个链接。

# 如何操作...

此脚本使用`lynx`和`curl`查找网页中的失效链接：

```
#!/bin/bash  
#Filename: find_broken.sh 
#Desc: Find broken links in a website 

if [ $# -ne 1 ];  
then  
  echo -e "$Usage: $0 URL\n"  
  exit 1;  
fi  

echo Broken links:  

mkdir /tmp/$$.lynx  
cd /tmp/$$.lynx  

lynx -traversal $1 > /dev/null  
count=0;  

sort -u reject.dat > links.txt  

while read link;  
do  
  output=`curl -I $link -s \ 
| grep -e "HTTP/.*OK" -e "HTTP/.*200"` 
  if [[ -z $output ]];  
  then  
    output=`curl -I $link -s | grep -e "HTTP/.*301"` 
    if [[ -z $output ]];  
      then  
      echo "BROKEN: $link" 
      let count++  
    else  
      echo "MOVED: $link" 
    fi 
  fi  
done < links.txt  

[ $count -eq 0 ] && echo No broken links found. 

```

# 工作原理...

`lynx -traversal URL`将在工作目录中生成多个文件。其中包括一个`reject.dat`文件，里面包含该网站的所有链接。`sort -u`用于构建一个去重的列表。然后，我们遍历每个链接，使用`curl -I`检查其头部响应。如果头部的第一行包含 HTTP/和`OK`或`200`，则表示该链接有效。如果该链接无效，则重新检查并测试是否有`301`-*链接已移动*的响应。如果该测试也失败，损坏的链接将会显示在屏幕上。

从名字来看，`reject.dat`可能会让人觉得它应该包含无法访问或损坏的 URL 列表。然而，事实并非如此，`lynx`只是将所有的 URL 都添加到这个文件中。

另外需要注意的是，`lynx`会生成一个名为`traverse.errors`的文件，其中包含所有浏览时出问题的 URL。然而，`lynx`只会添加返回`HTTP 404 (未找到)`的 URL，因此我们会忽略其他错误（例如，`HTTP 403 Forbidden`）。这就是为什么我们需要手动检查状态码的原因。

# 另见

+   本章中的*将网页下载为纯文本*示例解释了`lynx`命令。

+   本章中的*cURL 入门*示例解释了`curl`命令。

# 跟踪网站变化

追踪网站变化对于 Web 开发者和用户都有用。手动检查网站是不现实的，但可以定期运行变化追踪脚本。一旦发生变化，它会生成通知。

# 准备工作

在 Bash 脚本中追踪变化意味着在不同时间获取网站，并使用`diff`命令查看差异。我们可以使用`curl`和`diff`来完成这个任务。

# 如何操作...

这个 Bash 脚本结合了不同的命令，用于追踪网页变化：

```
#!/bin/bash 
#Filename: change_track.sh 
#Desc: Script to track changes to webpage 

if [ $# -ne 1 ]; 
then  
  echo -e "$Usage: $0 URL\n" 
  exit 1; 
fi 

first_time=0 
# Not first time 

if [ ! -e "last.html" ]; 
then 
  first_time=1 
  # Set it is first time run 
fi 

curl --silent $1 -o recent.html 

if [ $first_time -ne 1 ]; 
then 
  changes=$(diff -u last.html recent.html) 
  if [ -n "$changes" ]; 
  then 
    echo -e "Changes:\n" 
    echo "$changes" 
  else 
    echo -e "\nWebsite has no changes" 
  fi 
else 
  echo "[First run] Archiving.." 

fi 

cp recent.html last.html 

```

让我们来看一下你控制的一个网站上`track_changes.sh`脚本的输出。首先我们看到的是网页没有变化时的输出，然后是做出更改后的输出。

请注意，你应该将`MyWebSite.org`替换为你自己网站的名称。

+   首先，运行以下命令：

```
        $ ./track_changes.sh http://www.MyWebSite.org
 [First run] Archiving..

```

+   其次，再次运行命令：

```
        $ ./track_changes.sh http://www.MyWebSite.org
 Website has no changes 

```

+   第三，在更改网页后，运行以下命令：

```
        $ ./track_changes.sh http://www.MyWebSite.org

 Changes: 

 --- last.html    2010-08-01 07:29:15.000000000 +0200 
 +++ recent.html    2010-08-01 07:29:43.000000000 +0200 
 @@ -1,3 +1,4 @@ 
 <html>
 +added line :)
 <p>data</p>
 </html>

```

# 它是如何工作的...

脚本通过`[ ! -e "last.html" ];`检查脚本是否是第一次运行。如果`last.html`不存在，这意味着是第一次运行，必须下载网页并将其保存为`last.html`。

如果这不是第一次，它会下载新的副本（`recent.html`），并通过 diff 工具检查差异。任何更改都会以 diff 输出的形式显示。最后，`recent.html`会被复制到`last.html`。

请注意，更改你正在检查的网站将会在第一次检查时生成一个巨大的 diff 文件。如果你需要追踪多个页面，可以为你打算观察的每个网站创建一个文件夹。

# 另见

+   本章中的*《cURL 入门》*配方解释了`curl`命令

# 发布到网页并读取响应

`POST`和`GET`是 HTTP 请求中的两种类型，用于向网站发送信息或从网站检索信息。在`GET`请求中，我们通过网页的 URL 本身发送参数（名称-值对）。`POST`命令将键值对放在消息体中，而不是 URL 中。`POST`通常用于提交较长的表单，或隐藏信息，以防止在表面上暴露。

# 准备工作

对于这个示例，我们将使用**tclhttpd**包中包含的`guestbook`网站。你可以从[`sourceforge.net/projects/tclhttpd`](http://sourceforge.net/projects/tclhttpd)下载 tclhttpd，并在本地系统上运行它来创建一个本地 Web 服务器。留言簿页面请求一个名字和 URL，并将其添加到留言簿中，显示谁在用户点击“将我添加到留言簿”按钮时访问了该网站。

这个过程可以通过一个`curl`（或`wget`）命令来自动化。

# 如何操作...

下载 tclhttpd 包并`cd`到`bin`文件夹。使用以下命令启动 tclhttpd 守护进程：

```
    tclsh httpd.tcl

```

发布和读取通用网站 HTML 响应的格式如下：

```
    $ curl URL -d "postvar=postdata2&postvar2=postdata2"

```

考虑以下示例：

```
    $ curl http://127.0.0.1:8015/guestbook/newguest.html \
 -d "name=Clif&url=www.noucorp.com&http=www.noucorp.com"

```

`curl`命令打印出类似这样的响应页面：

```
<HTML> 
<Head> 
<title>Guestbook Registration Confirmed</title> 
</Head> 
<Body BGCOLOR=white TEXT=black> 
<a href="www.noucorp.com">www.noucorp.com</a> 

<DL> 
<DT>Name 
<DD>Clif 
<DT>URL 
<DD> 
</DL> 
www.noucorp.com 

</Body> 

```

`-d`是用于发布数据的参数。`-d`的字符串参数类似于`GET`请求的语义。`var=value`对应该通过`&`分隔。

你可以使用`wget`通过`--post-data "string"`发布数据。考虑以下示例：

```
    $ wget http://127.0.0.1:8015/guestbook/newguest.cgi \
 --post-data "name=Clif&url=www.noucorp.com&http=www.noucorp.com" \
 -O output.html

```

使用与 cURL 相同的格式来传递名称-值对。`output.html`中的文本与 cURL 命令返回的文本相同。

传递给发布参数（例如`-d`或`--post-data`）的字符串应始终用引号括起来。如果不使用引号，`&`会被 shell 解释为指示该操作为后台进程。

如果你查看网页源代码（从浏览器选择查看源代码选项），你会看到定义了一个 HTML 表单，类似于以下代码：

```
<form action="newguest.cgi" " method="post" > 
<ul> 
<li> Name: <input type="text" name="name" size="40" > 
<li> Url: <input type="text" name="url" size="40" > 
<input type="submit" > 
</ul> 
</form> 

```

在这里，`newguest.cgi`是目标网址。当用户输入详细信息并点击提交按钮时，名称和网址输入会作为`POST`请求发送到`newguest.cgi`，并且响应页面会返回到浏览器。

# 另请参见

+   本章中的*cURL 入门*食谱解释了`curl`命令。

+   本章中的*从网页下载*食谱解释了`wget`命令。

# 从互联网下载视频

下载视频有很多原因。如果你使用的是按流量计费的服务，你可能想在非高峰时段下载视频，那时候费用较便宜。你可能想在带宽不足以支持流媒体播放的情况下观看视频，或者你只是想确保总能拥有那段可爱猫咪的视频给朋友们展示。

# 准备工作

一个用于下载视频的程序是`youtube-dl`。它并不包含在大多数发行版中，且软件库可能没有更新，所以最好访问`youtube-dl`的官方网站[`yt-dl.org`](http://yt-dl.org)。

你会在该页面上找到下载和安装`youtube-dl`的链接和信息。

# 如何操作...

使用`youtube-dl`很简单。打开浏览器，找到你喜欢的视频。然后复制/粘贴那个网址到`youtube-dl`命令行中：

```
    youtube-dl https://www.youtube.com/watch?v=AJrsl3fHQ74

```

当`youtube-dl`正在下载文件时，它会在你的终端上生成一个状态行。

# 它是如何工作的...

`youtube-dl`程序通过向服务器发送`GET`消息来工作，就像浏览器那样。它伪装成浏览器，以便 YouTube 或其他视频提供商下载视频，就好像设备正在进行流媒体播放。

`-list-formats`（`-F`）选项会列出视频可用的格式，而`-format`（`-f`）选项会指定下载的格式。如果你想下载一个比你的网络连接可以稳定播放的更高分辨率的视频，这个选项很有用。

# 使用 OTS 总结文本

**开放文本摘要器**（**OTS**）是一个应用程序，可以去除文本中的冗余内容，创建一个简明的摘要。

# 准备工作

`ots`包不是大多数 Linux 标准发行版的一部分，但可以通过以下命令安装：

```
    apt-get install libots-devel

```

# 如何操作...

`OTS` 应用程序使用起来非常简单。它可以从文件或 `stdin` 中读取文本，并将生成的摘要输出到 `stdout`。

```
    ots LongFile.txt | less

```

或

```
    cat LongFile.txt | ots | less

```

`OTS` 应用程序也可以与 `curl` 一起使用，来总结网站上的信息。例如，你可以使用 `ots` 来总结冗长的博客：

```
    curl http://BlogSite.org | sed -r 's/<[^>]+>//g' | ots | less

```

# 它是如何工作的...

`curl` 命令从博客站点获取页面并将页面传递给 `sed`。`sed` 命令使用正则表达式将所有 HTML 标签（即以小于符号开头，以大于符号结尾的字符串）替换为空白。去除标签后的文本会传递给 `ots`，生成的摘要会通过 `less` 显示。

# 从命令行翻译文本

Google 提供了一个在线翻译服务，你可以通过浏览器访问。Andrei Neculau 创建了一个 **awk** 脚本，能够访问该服务并从命令行进行翻译。

# 准备工作

大多数 Linux 发行版没有包含命令行翻译工具，但你可以通过以下方式从 Git 安装它：

```
    cd ~/bin
 wget git.io/trans 
 chmod 755 ./trans

```

# 如何操作...

`trans` 应用程序默认会根据你的本地环境变量翻译成该语言。

```
    $> trans "J'adore Linux"

 J'adore Linux

 I love Linux

 Translations of J'adore Linux
 French -> English

 J'adore Linux 
 I love Linux

```

你可以通过在文本之前添加一个选项来控制翻译的语言。选项的格式如下：

```
    from:to

```

要将英文翻译成法语，请使用以下命令：

```
    $> trans en:fr "I love Linux" 
 J'aime Linux

```

# 它是如何工作的...

`trans` 程序大约有 5,000 行 awk 代码，它使用 `curl` 与 Google、Bing 和 Yandex 翻译服务进行通信。
