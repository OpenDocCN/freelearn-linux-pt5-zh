# Tangled Web? Not At All!

In this chapter, we will cover the following recipes:

*   Downloading from a web page
*   Downloading a web page as plain text
*   A primer on cURL
*   Accessing unread Gmail e-mails from the command line
*   Parsing data from a website
*   Image crawler and downloader
*   Web photo album generator
*   Twitter command-line client
*   Accessing word definitions via a web server
*   Finding broken links in a website
*   Tracking changes to a website
*   Posting to a web page and reading the response
*   Downloading a video from the Internet
*   Summarizing text with OTS
*   Translating text from the command line

# Introduction

The Web has become the face of technology and the central access point for data processing. Shell scripts cannot do everything that languages such as PHP can do on the Web, but there are many tasks for which shell scripts are ideally suited. We will explore recipes to download and parse website data, send data to forms, and automate website-usage tasks and similar activities. We can automate many activities that we perform interactively through a browser with a few lines of scripting. The functionality provided by the HTTP protocol and command-line utilities enables us to write scripts to solve many web-automation needs.

# Downloading from a web page

Downloading a file or a web page is simple. A few command-line download utilities are available to perform this task.

# Getting ready

`wget` is a flexible file download command-line utility that can be configured with many options.

# How to do it...

A web page or a remote file can be downloaded using `wget`:

```
$ wget URL

```

For example:

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

It is also possible to specify multiple download URLs:

```
$ wget URL1 URL2 URL3 ..

```

# How it works...

By default, the downloaded files are named the same as the URL, and the download information and progress is written to `stdout`.

The `-O` option specifies the output filename. If a file with that name already exists, it will be replaced by the downloaded file:

```
$ wget http://www.knopper.net -O knopper.html.

```

The `-o` option specifies a `logfile` instead of printing logs to `stdout`:

```
$ wget ftp://ftp.example.com/somefile.img -O dloaded_file.img -o log

```

Using the preceding command will print nothing on the screen. The log or progress will be written to the log and the output file will be `dloaded_file.img`.

There is a chance that downloads might break due to unstable Internet connections. The `-t` option specifies how many times the utility will retry before giving up:

```
$ wget -t 5 URL

```

Use a value of `0` to force `wget` to keep trying infinitely:

```
$ wget -t 0 URL

```

# There's more...

The `wget` utility has options to fine-tune behavior and solve problems.

# Restricting the download speed

When there is limited bandwidth with many applications sharing it, a large file can devour all the bandwidth and starve other processes (perhaps interactive users). The `wget` option `-limit-rate` will specify the maximum bandwidth for the download job, allowing all applications fair access to the Internet:

```
$ wget  --limit-rate 20k http://example.com/file.iso

```

In this command, `k` (kilobyte) specifies the speed limit. You can also use `m` for megabyte.

The `-quota` (or `-Q`) option specifies the maximum size of the download. `wget` will stop when the quota is exceeded. This is useful when downloading multiple files to a system with limited space:

```
$ wget -Q 100m http://example.com/file1 http://example.com/file2

```

# Resume downloading and continue

If `wget` gets interrupted before the download is complete, it can be resumed where it left off with the `-c` option:

```
$ wget -c URL

```

# Copying a complete website (mirroring)

`wget` can download a complete website by recursively collecting the URL links and downloading them like a crawler. To download the pages, use the `--mirror` option:

```
$ wget --mirror --convert-links exampledomain.com

```

Alternatively, use the following command:

```
$ wget -r -N -l -k DEPTH URL

```

The `-l` option specifies the depth of web pages as levels. This means that it will traverse only that number of levels. It is used along with `-r` (recursive). The `-N` argument is used to enable time stamping for the file. `URL` is the base URL for a website for which the download needs to be initiated. The `-k` or `--convert-links` option instructs `wget` to convert the links to other pages to the local copy.

Exercise discretion when mirroring other websites. Unless you have permission, only perform this for your personal use and don't do it too frequently.

# Accessing pages with HTTP or FTP authentication

The `--user` and `--password` arguments provide the username and password to websites that require authentication.

```
$ wget --user username --password pass URL

```

It is also possible to ask for a password without specifying the password inline. For this, use `--ask-password` instead of the `--password` argument.

# Downloading a web page as plain text

Web pages are simply text with HTML tags, JavaScript, and CSS. The HTML tags define the content of the web page, which we can parse for specific content. Bash scripts can parse web pages. An HTML file can be viewed in a web browser to see it properly formatted or processed with tools described in the previous chapter.

Parsing a text document is simpler than parsing HTML data because we aren't required to strip off the HTML tags. **Lynx** is a command-line web browser that downloads a web page as plain text.

# Getting ready

Lynx is not installed in all distributions, but is available via the package manager.

```
# yum install lynx

```

Alternatively, you can execute the following command:

```
 apt-get install lynx

```

# How to do it...

The `-dump` option downloads a web page as pure ASCII. The next recipe shows how to send that ASCII version of the page to a file:

```
$ lynx URL -dump > webpage_as_text.txt

```

This command will list all the hyperlinks (`<a href="link">`) separately under a heading `References`, as the footer of the text output. This lets us parse links separately with regular expressions.

Consider this example:

```
$lynx -dump http://google.com > plain_text_page.txt

```

You can see the plain text version of `text` using the `cat` command:

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

# A primer on cURL

**cURL** transfers data to or from a server using the HTTP, HTTPS, or FTP protocols. It supports `POST`, cookies, authentication, downloading partial files from a specified offset, referer, user agent string, extra headers, limiting speed, maximum file size, progress bar, and more. cURL is useful for maintaining a website, retrieving data, and checking server configurations.

# Getting ready

Unlike `wget`, cURL is not included in all Linux distros; you may have to install it with your package manager.

By default, cURL dumps downloaded files to `stdout`, and progress information to `stderr`. To disable displaying progress information, use the `--silent` option.

# How to do it...

The `curl` command performs many functions, including downloading, sending different HTTP requests, and specifying HTTP headers.

*   To dump the downloaded file to `stdout`, use the following command:

```
        $ curl URL

```

*   The `-O` option specifies sending the downloaded data into a file with the filename parsed from the URL. Note that the URL must be a full page URL, not just a site name.

```
        $ curl www.knopper.net/index.htm --silent -O

```

*   The `-o` option specifies the output file name. With this option you can specify only the site name to retrieve the home page.

```
        $curl www.knopper.net -o knoppix_index.html
 % Total % Received % Xferd  Avg  Speed Time   Time  Time  
        Current
 Dload Upload Total Spent Left  Speed
 100 6889 100 6889  0 0     10902  0     --:-- --:-- --:-- 26033

```

*   The `-silent` option prevents the `curl` command from displaying progress information:

```
        $ curl URL --silent

```

*   The `-progress` option displays progress bar while downloading:

```
        $ curl http://knopper.net -o index.html --progress
 ################################## 100.0% 

```

# How it works...

cURL downloads web pages or remote files to your local system. You can control the destination filename with the `-O` and `-o` options, and verbosity with the `-silent` and `-progress` options.

# There's more...

In the preceding sections, you learned how to download files. cURL supports more options to fine tune its behavior.

# Continuing and resuming downloads

cURL can resume a download from a given offset. This is useful if you have a per-day data limit and a large file to download.

```
$ curl URL/file -C offset

```

offset is an integer value in bytes.

cURL doesn't require us to know the exact byte offset, if we want to resume downloading a file. If you want cURL to figure out the correct resume point, use the `-C -` option, as follows:

```
$ curl -C - URL

```

cURL will automatically figure out where to restart the download of the specified file.

# Setting the referer string with cURL

The **Referer** field in the HTTP header identifies the page that led to the current web page. When a user clicks on a link on web page A to go to web page B, the referer header string for page B will contain the URL of page A.

Some dynamic pages check the referer string before returning the HTML data. For example, a web page may display a Google logo when a user navigates to a website from Google, and display a different page when the user types the URL.

A web developer can write a condition to return a Google page if the referer is www.google.com, or return a different page if not.

You can use `--referer` with the `curl` command to specify the referer string, as follows:

```
$ curl --referer Referer_URL target_URL

```

Consider this example:

```
$ curl --referer http://google.com http://knopper.org

```

# Cookies with cURL

`curl` can specify and store the cookies encountered during HTTP operations.

The `-cookie``COOKIE_IDENTIFER` option specifies which cookies to provide. Cookies are defined as `name=value`. Multiple cookies should be delimited with a semicolon (`;`):

```
$ curl http://example.com --cookie "user=username;pass=hack"

```

The `-cookie-jar` option specifies the file to store cookies in:

```
$ curl URL --cookie-jar cookie_file

```

# Setting a user agent string with cURL

Some web pages that check the user agent won't work if there is no user agent specified. For example, some old websites require **Internet Explorer** (**IE**). If a different browser is used, they display a message that the site must be viewed with IE. This is because the website checks for a user agent. You can set the user agent with `curl`.

The `--user-agent` or `-A` option sets the user agent:

```
$ curl URL --user-agent "Mozilla/5.0"

```

Additional headers can be passed with cURL. Use `-H "Header"` to pass additional headers:

```
$ curl -H "Host: www.knopper.net" -H "Accept-language: en" URL

```

There are many different user agent strings across multiple browsers and crawlers on the Web. You can find a list of some of them at [http://www.useragentstring.com/pages/useragentstring.php](http://www.useragentstring.com/pages/useragentstring.php).

# Specifying a bandwidth limit on cURL

When bandwidth is shared among multiple users, we can limit the download rate with the `--limit-rate` option:

```
$ curl URL --limit-rate 20k

```

The rate can be specified with `k` (kilobyte) or `m` (megabyte).

# Specifying the maximum download size

The `--max-filesize` option specifies the maximum file size:

```
$ curl URL --max-filesize bytes

```

The `curl` command will return a non-zero exit code if the file size exceeds the limit or a zero if the download succeeds.

# Authenticating with cURL

The `curl` command's  `-u` option performs HTTP or FTP authentication.

The username and password can be specified using `-u username:password`:

```
$ curl -u user:pass http://test_auth.com

```

If you prefer to be prompted for the password, provide only a username:

```
$ curl -u user http://test_auth.com 

```

# Printing response headers excluding data

Examining headers is sufficient for many checks and statistics. For example, we don't need to download an entire page to confirm it is reachable. Just reading the HTTP response is sufficient.

Another use case for examining the HTTP header is to check the `Content-Length` field to determine the file size or the `Last-Modified` field to see if the file is newer than a current copy before downloading.

The `-I` or `-head` option outputs only the HTTP headers, without downloading the remote file:

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

# See also

*   The *Posting to a web page and reading the response* recipe in this chapter

# Accessing unread Gmail e-mails from the command line

Gmail is a widely-used free e-mail service from Google: [http://mail.google.com/](http://mail.google.com/). It allows you to read your mail via a browser or an authenticated RSS feeds. We can parse the RSS feeds to report the sender name and subject. This is a quick way to scan unread e-mails without opening the web browser.

# How to do it...

Let's go through a shell script to parse the RSS feeds for Gmail to display the unread mails:

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

The output resembles this:

```
$ ./fetch_gmail.sh
From: SLYNUX [ slynux@slynux.com ]
Subject: Book release - 2

From: SLYNUX [ slynux@slynux.com ]
Subject: Book release - 1 
.
... 5 entries

```

If you use a Gmail account with two-factor authentication, you will have to generate a new key for this script and use it. Your regular password won't work.

# How it works...

The script uses cURL to download the RSS feed. You can view the format of the incoming data by logging in to your Gmail account and viewing [https://mail.google.com/mail/feed/atom](https://mail.google.com/mail/feed/atom).

cURL reads the RSS feed with the user authentication provided by the `-u user:pass` argument. When you use `-u user` without the password cURL, it will interactively ask for the password.

*   `tr -d '\n'`: This removes the newline characters
*   `sed 's:</entry>:\n:g'`: This replaces every `</entry>` element with a newline, so each e-mail entry is delimited by a new line and, hence, mails can be parsed one-by-one.

The next block of script that needs to be executed as one single expression uses `sed` to extract the relevant fields:

```
 sed 's/.*<title>\(.*\)<\/title.*<author><name>\([^<]*\)<\/name><email>
 \([^<]*\).*/Author: \2 [\3] \nSubject: \1\n/' 

```

This script matches the title with the `<title>\(.*\)<\/title` regular expression, the sender name with the `<author><name>\([^<]*\)<\/name>` regular expression, and e-mail using `<email>\([^<]*\)`. Sed uses back referencing to display the author, title, and subject of the e-mail into an easy to read format:

```
Author: \2 [\3] \nSubject: \1\n 

```

`\1` corresponds to the first substring match (title), `\2` for the second substring match (name), and so on.

The `SHOW_COUNT=5` variable is used to take the number of unread mail entries to be printed on the terminal.

`head` is used to display only the `SHOW_COUNT*3` lines from the first line. `SHOW_COUNT` is multiplied by three in order to show three lines of output.

# See also

*   The *A primer on cURL* recipe in this chapter explains the `curl` command
*   The *Using sed to perform text replacement* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving,* explains the `sed` command

# Parsing data from a website

The `lynx`, `sed`, and `awk` commands can be used to mine data from websites. You might have come across a list of actress rankings in a *Searching and mining text inside a file with grep *recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*; it was generated by parsing the [http://www.johntorres.net/BoxOfficefemaleList.html](http://www.johntorres.net/BoxOfficefemaleList.html) web page.

# How to do it...

Let's go through the commands used to parse details of actresses from the website:

```
$ lynx -dump -nolist \
    http://www.johntorres.net/BoxOfficefemaleList.html
    grep -o "Rank-.*" | \
    sed -e 's/ *Rank-\([0-9]*\) *\(.*\)/\1\t\2/' | \
    sort -nk 1 > actresslist.txt 

```

The output is as follows:

```
# Only 3 entries shown. All others omitted due to space limits
1   Keira Knightley 
2   Natalie Portman 
3   Monica Bellucci 

```

# How it works...

Lynx is a command-line web browser; it can dump a text version of a website as we will see in a web browser, instead of returning the raw HTML as `wget` or cURL does. This saves the step of removing HTML tags. The `-nolist` option shows the links without numbers. Parsing and formatting the lines that contain Rank is done with `sed`:

```
sed -e 's/ *Rank-\([0-9]*\) *\(.*\)/\1\t\2/'

```

These lines are then sorted according to the ranks.

# See also

*   The *Using sed to perform text replacement* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains the `sed` command
*   The *Downloading a web page as plain text* recipe in this chapter explains the `lynx` command

# Image crawler and downloader

**Image crawlers** download all the images that appear in a web page. Instead of going through the HTML page to pick the images by hand, we can use a script to identify the images and download them automatically.

# How to do it...

This Bash script will identify and download the images from a web page:

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

An example usage is as follows:

```
$ url=https://commons.wikimedia.org/wiki/Main_Page
$ ./img_downloader.sh $url -d images

```

# How it works...

The image downloader script reads an HTML page, strips out all tags except `<img>`, parses `src="img/URL"` from the `<img>` tag, and downloads them to the specified directory. This script accepts a web page URL and the destination directory as command-line arguments.

The `[ $# -ne 3 ]` statement checks whether the total number of arguments to the script is three, otherwise it exits and returns a usage example. Otherwise, this code parses the URL and destination directory:

```
while [ -n "$1" ] 
do  
  case $1 in 
  -d) shift; directory=$1; shift ;; 
   *) url=${url:-$1}; shift;; 
esac 
done 

```

The `while` loop runs until all the arguments are processed. The `shift` command shifts arguments to the left so that `$1` will take the next argument's value; that is, `$2`, and so on. Hence, we can evaluate all arguments through `$1` itself.

The `case` statement checks the first argument (`$1`). If that matches `-d`, the next argument must be a directory name, so the arguments are shifted and the directory name is saved. If the argument is any other string it is a URL.

The advantage of parsing arguments in this way is that we can place the -d argument anywhere in the command line:

```
$ ./img_downloader.sh -d DIR URL

```

Or:

```
$ ./img_downloader.sh URL -d DIR

```

`egrep -o "<img src=[^>]*>"` will print only the matching strings, which are the `<img>` tags including their attributes. The `[^>]*` phrase matches all the characters except the closing `>`, that is, `<img src="img/image.jpg">`.

`sed's/<img src=\"\([^"]*\).*/\1/g'` extracts the `url` from the `src="img/url"` string.

There are two types of image source paths: relative and absolute. **Absolute paths** contain full URLs that start with `http://` or `https://`. Relative URLs starts with `/` or `image_name` itself. An example of an absolute URL is `http://example.com/image.jpg`. An example of a relative URL is `/image.jpg`.

For relative URLs, the starting `/` should be replaced with the base URL to transform it to `http://example.com/image.jpg`. The script initializes `baseurl` by extracting it from the initial URL with the following command:

```
baseurl=$(echo $url | egrep -o "https?://[a-z.\-]+") 

```

The output of the previously described `sed` command is piped into another sed command to replace a leading `/` with `baseurl`, and the results are saved in a file named for the script's PID: (`/tmp/$$.list`).

```
sed "s,^/,$baseurl/," > /tmp/$$.list 

```

The final `while` loop iterates through each line of the list and uses curl to download the images. The `--silent` argument is used with `curl` to avoid extra progress messages from being printed on the screen.

# See also

*   The *A primer on cURL* recipe in this chapter explains the `curl` command
*   The *Using sed to perform text replacement* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*  explains the `sed` command
*   The *Searching and mining text inside a file with grep* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains the `grep` command

# Web photo album generator

Web developers frequently create photo albums of full-size and thumbnail images. When a thumbnail is clicked, a large version of the picture is displayed. This requires resizing and placing many images. These actions can be automated with a simple Bash script. The script creates thumbnails, places them in exact directories, and generates the code fragment for `<img>` tags automatically.

# Getting ready

This script uses a `for` loop to iterate over every image in the current directory. The usual Bash utilities such as `cat` and `convert` (from the Image Magick package) are used. These will generate an HTML album, using all the images, in `index.html`.

# How to do it...

This Bash script will generate an HTML album page:

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

Run the script as follows:

```
$ ./generate_album.sh
Creating album..
Album generated to index.html

```

# How it works...

The initial part of the script is used to write the header part of the HTML page.

The following script redirects all the contents up to `EOF1` to `index.html`:

```
cat <<EOF1 > index.html 
contents... 
EOF1 

```

The header includes the HTML and CSS styling.

`for img in *.jpg *.JPG;` iterates over the filenames and evaluates the body of the loop.

`convert "$img" -resize "100x" "thumbs/$img"` creates images 100px-wide as thumbnails.

The following statement generates the required `<img>` tag and appends it to `index.html`:

```
echo "<a href=\"$img\" >" 
echo "<img src=\"thumbs/$img\" title=\"$img\" /></a>" >> index.html 

```

Finally, the footer HTML tags are appended with `cat` as  in the first part of the script.

# See also

*   The *Web photo album generator* recipe in this chapter explains `EOF` and `stdin` redirection

# Twitter command-line client

**Twitter** is the hottest micro-blogging platform, as well as the latest buzz word for online social media now. We can use Twitter API to read tweets on our timeline from the command line!

Let's see how to do it.

# Getting ready

Recently, Twitter stopped allowing people to log in using plain HTTP Authentication, so we must use OAuth to authenticate ourselves. A full explanation of OAuth is out of the scope of this book, so we will use a library which makes it easy to use OAuth from Bash scripts. Perform the following steps:

1.  Download the `bash-oauth` library from [https://github.com/livibetter/bash-oauth/archive/master.zip](https://github.com/livibetter/bash-oauth/archive/master.zip), and unzip it to any directory.
2.  Go to that directory and then inside the subdirectory `bash-oauth-master`, run `make install-all` as root.
3.  Go to [https://apps.twitter.com/](https://apps.twitter.com/) and register a new app. This will make it possible to use OAuth.
4.  After registering the new app, go to your app's settings and change Access type to Read and Write.
5.  Now, go to the Details section of the app and note two things, Consumer Key and Consumer Secret, so that you can substitute these in the script we are going to write.

Great, now let's write the script that uses this.

# How to do it...

This Bash script uses the OAuth library to read tweets or send your own updates:

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

Run the script as follows:

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

# How it works...

First of all, we use the source command to include the `TwitterOAuth.sh` library, so we can use its functions to access Twitter. The `TO_init` function initializes the library.

Every app needs to get an OAuth token and token secret the first time it is used. If these are not present, we use the `TO_access_token_helper` library function to acquire them. Once we have the tokens, we save them to a `config` file so we can simply source it the next time the script is run.

The `TO_statuses_home_timeline` library function fetches the tweets from Twitter. This data is retuned as a single long string in JSON format, which starts like this:

```
[{"created_at":"Thu Nov 10 14:45:20 +0000    
"016","id":7...9,"id_str":"7...9","text":"Dining...

```

Each tweet starts with the `"created_at"` tag and includes a `text` and a `screen_name` tag. The script will extract the text and screen name data and display only those fields.

The script assigns the long string to the `TO_ret` variable.

The JSON format uses quoted strings for the key and may or may not quote the value. The key/value pairs are separated by commas, and the key and value are separated by a colon (`:`).

The first `sed` replaces each `"` character set with a newline, making each key/value a separate line. These lines are piped to another `sed` command to replace each occurrence of `":` with a tilde (~), which creates a line like this:

```
screen_name~"Clif_Flynt" 

```

The final `awk` script reads each line. The `-F~` option splits the line into fields at the tilde, so `$1` is the key and `$2` is the value. The `if` command checks for `text` or `screen_name`. The text is first in the tweet, but it's easier to read if we report the sender first; so the script saves a `text` return until it sees a `screen_name`, then prints the current value of `$2` and the saved value of the text.

The `TO_statuses_update` library function generates a tweet. The empty first parameter defines our message as being in the default format, and the message is a part of the second parameter.

# See also

*   The *Using sed to perform text replacement* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*,  explains the `sed` command
*   The *Searching and mining text inside a file with grep* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains the `grep` command

# Accessing word definitions via a web server

Several dictionaries on the Web offer an API to interact with their website via scripts. This recipe demonstrates how to use a popular one.

# Getting ready

We are going to use `curl`, `sed`, and `grep` for this define utility. There are a lot of dictionary websites where you can register and use their APIs for personal use for free. In this example, we are using Merriam-Webster's dictionary API. Perform the following steps:

1.  Go to [http://www.dictionaryapi.com/register/index.htm](http://www.dictionaryapi.com/register/index.htm), and register an account for yourself. Select Collegiate Dictionary and Learner's Dictionary:
2.  Log in using the newly created account and go to My Keys to access the keys. Note the key for the learner's dictionary.

# How to do it...

This script will display a word definition:

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

Run the script like this:

```
    $ ./define.sh usb 1
 1  :a system for connecting a computer to another device (such as   
    a printer, keyboard, or mouse) by using a special kind of cord a   
    USB cable/port USB is an abbreviation of "Universal Serial Bus."How 
    it works...

```

# How it works...

We use `curl` to fetch the data from the dictionary API web page by specifying our API `Key ($apikey)`, and the word we want the definition for (`$1`). The result contains definitions in the `<dt>` tags, selected with `grep`. The `sed` command removes the tags. The script selects the required number of lines from the definitions and uses `nl` to add a line number to each line.

# See also

*   The *Using sed to perform text replacement* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml) explains the `sed` command
*   The *Searching and mining text inside a file with grep* recipe in [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains the `grep` command

# Finding broken links in a website

Websites must be tested for broken links. It's not feasible to do this manually for large websites. Luckily, this is an easy task to automate. We can find the broken links with HTTP manipulation tools.

# Getting ready

We can use `lynx` and `curl` to identify the links and find broken ones. Lynx has the `-traversal` option, which recursively visits pages on the website and builds a list of all hyperlinks. cURL is used to verify each of the links.

# How to do it...

This script uses `lynx` and `curl` to find the broken links on a web page:

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

# How it works...

`lynx -traversal URL` will produce a number of files in the working directory. It includes a `reject.dat` file, which will contain all the links in the website. `sort -u` is used to build a list by avoiding duplicates. Then, we iterate through each link and check the header response using `curl -I`. If the first line of the header contains HTTP/ and either `OK` or `200`, it means that the link is valid. If the link is not valid, it is rechecked and tested for a `301`-*link moved*-reply. If that test also fails, the broken link is printed on the screen.

From its name, it might seem like `reject.dat` should contain a list of URLs that were broken or unreachable. However, this is not the case, and lynx just adds all the URLs there.
Also note that `lynx` generates a file called `traverse.errors`, which contains all the URLs that had problems in browsing. However, `lynx` will only add URLs that return `HTTP 404 (not found)`, and so we will lose other errors (for instance, `HTTP 403 Forbidden`). This is why we manually check for statuses.

# See also

*   The *Downloading a web page as plain text* recipe in this chapter explains the `lynx` command
*   The *A primer on cURL* recipe in this chapter explains the `curl` command

# Tracking changes to a website

Tracking website changes is useful for both web developers and users. Checking a website manually is impractical, but a change tracking script can be run at regular intervals. When a change occurs, it generates a notification.

# Getting ready

Tracking changes in terms of Bash scripting means fetching websites at different times and taking the difference using the `diff` command. We can use `curl` and `diff` to do this.

# How to do it...

This Bash script combines different commands, to track changes in a web page:

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

Let's look at the output of the `track_changes.sh` script on a website you control. First we'll see the output when a web page is unchanged, and then after making changes.

Note that you should change `MyWebSite.org` to your website name.

*   First, run the following command:

```
        $ ./track_changes.sh http://www.MyWebSite.org
 [First run] Archiving..

```

*   Second, run the command again:

```
        $ ./track_changes.sh http://www.MyWebSite.org
 Website has no changes 

```

*   Third, run the following command after making changes to the web page:

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

# How it works...

The script checks whether the script is running for the first time using `[ ! -e "last.html" ];`. If `last.html` doesn't exist, it means that it is the first time, and the web page must be downloaded and saved as `last.html`.

If it is not the first time, it downloads the new copy (`recent.html`) and checks the difference with the diff utility. Any changes will be displayed as diff output. Finally, `recent.html` is copied to `last.html`.

Note that changing the website you are checking will generate a huge diff file the first time you examine it. If you need to track multiple pages, you can create a folder for each website you intend to watch.

# See also

*   The *A primer on cURL* recipe in this chapter explains the `curl` command

# Posting to a web page and reading the response

`POST` and `GET` are two types of request in HTTP to send information to or retrieve information from a website. In a `GET` request, we send parameters (name-value pairs) through the web page URL itself. The POST command places the key/value pairs in the message body instead of the URL. `POST` is commonly used when submitting long forms or to conceal information submitted from a casual glance.

# Getting ready

For this recipe, we will use the sample `guestbook` website included in the **tclhttpd** package. You can download tclhttpd from [http://sourceforge.net/projects/tclhttpd](http://sourceforge.net/projects/tclhttpd) and then run it on your local system to create a local web server. The guestbook page requests a name and URL which it adds to a guestbook to show who has visited a site when the user clicks on the Add me to your guestbook button.

This process can be automated with a single `curl` (or `wget`) command.

# How to do it...

Download the tclhttpd package and `cd` to the `bin` folder. Start the tclhttpd daemon with this command:

```
    tclsh httpd.tcl

```

The format to POST and read the HTML response from the generic website resembles this:

```
    $ curl URL -d "postvar=postdata2&postvar2=postdata2"

```

Consider the following example:

```
    $ curl http://127.0.0.1:8015/guestbook/newguest.html \
 -d "name=Clif&url=www.noucorp.com&http=www.noucorp.com"

```

The curl command prints a response page like this:

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

`-d` is the argument used for posting. The string argument for `-d` is similar to the `GET` request semantics. `var=value` pairs are to be delimited by `&`.

You can post the data using `wget` using `--post-data "string"`. Consider the following example:

```
    $ wget http://127.0.0.1:8015/guestbook/newguest.cgi \
 --post-data "name=Clif&url=www.noucorp.com&http=www.noucorp.com" \
 -O output.html

```

Use the same format as cURL for name-value pairs. The text in output.html is the same as that returned by the cURL command.

The string to the post arguments (for example, to `-d` or `--post-data`) should always be given in quotes. If quotes are not used, `&` is interpreted by the shell to indicate that this should be a background process.

If you look at the website source (use the View Source option from the web browser), you will see an HTML form defined, similar to the following code:

```
<form action="newguest.cgi" " method="post" > 
<ul> 
<li> Name: <input type="text" name="name" size="40" > 
<li> Url: <input type="text" name="url" size="40" > 
<input type="submit" > 
</ul> 
</form> 

```

Here, `newguest.cgi` is the target URL. When the user enters the details and clicks on the Submit button, the name and URL inputs are sent to `newguest.cgi` as a `POST` request, and the response page is returned to the browser.

# See also

*   The *A primer on cURL* recipe in this chapter explains the `curl` command
*   The  *Downloading from a web page *recipe in this chapter explains the `wget` command

# Downloading a video from the Internet

There are many reasons for downloading a video. If you are on a metered service, you might want to download videos during off-hours when the rates are cheaper. You might want to watch videos where the bandwidth doesn't support streaming, or you might just want to make certain that you always have that video of cute cats to show your friends.

# Getting ready

One program for downloading videos is `youtube-dl`. This is not included in most distributions and the repositories may not be up-to-date, so it's best to go to the `youtube-dl` main site at [http://yt-dl.org](http://yt-dl.org).

You'll find links and information on that page for downloading and installing `youtube-dl`.

# How to do it...

Using `youtube-dl` is easy. Open your browser and find a video you like. Then copy/paste that URL to the `youtube-dl` command line:

```
    youtube-dl https://www.youtube.com/watch?v=AJrsl3fHQ74

```

While `youtube-dl` is downloading the file it will generate a status line on your terminal.

# How it works...

The `youtube-dl` program works by sending a `GET` message to the server, just as a browser would do. It masquerades as a browser so that YouTube or other video providers will download a video as if the device were streaming.

The `-list-formats` (`-F`) option will list the available formats a video is available in, and the `-format` (`-f`) option will specify which format to download. This is useful if you want to download a higher-resolution video than your Internet connection can reliably stream.

# Summarizing text with OTS

The **Open Text Summarizer** (**OTS**) is an application that removes the fluff from a piece of text to create a succinct summary.

# Getting ready

The `ots` package is not part of most Linux standard distributions, but it can be installed with the following command:

```
    apt-get install libots-devel

```

# How to do it...

The `OTS` application is easy to use. It reads text from a file or from `stdin` and generates the summary to `stdout`.

```
    ots LongFile.txt | less

```

Or

```
    cat LongFile.txt | ots | less

```

The `OTS` application can also be used with `curl` to summarize information from websites. For example, you can use `ots` to summarize longwinded blogs:

```
    curl http://BlogSite.org | sed -r 's/<[^>]+>//g' | ots | less

```

# How it works...

The `curl` command retrieves the page from a blog site and passes the page to `sed`. The `sed` command uses a regular expression to replace all the HTML tags, a string that starts with a less-than symbol and ends with a greater-than symbol, with a blank. The stripped text is passed to `ots`, which generates a summary that's displayed by less.

# Translating text from the command line

Google provides an online translation service you can access via your browser. Andrei Neculau created an **awk** script that will access that service and do translations from the command line.

# Getting ready

The command line translator is not included on most Linux distributions, but it can be installed directly from Git like this:

```
    cd ~/bin
 wget git.io/trans 
 chmod 755 ./trans

```

# How to do it...

The `trans` application will translate into the language in your locale environment variable by default.

```
    $> trans "J'adore Linux"

 J'adore Linux

 I love Linux

 Translations of J'adore Linux
 French -> English

 J'adore Linux 
 I love Linux

```

You can control the language being translated from and to with an option before the text. The format for the option is as follows:

```
    from:to

```

To translate from English to French, use the following command:

```
    $> trans en:fr "I love Linux" 
 J'aime Linux

```

# How it works...

The `trans` program is about 5,000 lines of awk code that uses `curl` to communicate with the Google, Bing, and Yandex translation services.