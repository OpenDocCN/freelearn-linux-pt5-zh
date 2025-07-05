# Chapter 8. Nginx – Deploying a Performance-centric Web Server

When it comes to web servers, it seems that Apache gets all the attention, and you may be led to believe that there is little competition; so let me introduce Nginx to you. We have seen many articles implementing the **Linux Apache MySQL and PHP** (**LAMP**) technology. We shall play this a little left field and look at **Linux Nginx MySQL and PHP** (**LEMP**); the E in LEMP comes from the phonetic version of the web server that is pronounced *engine-x*, allowing us to place a well needed vowel to create the acronym LEMP. The web server was first introduced in 2004, and Nginx is beginning to make inroads into the enterprise web space, being faster to deliver web content than equivalent Apache servers.

The following topics are going to be covered in this chapter:

*   **Installing and configuring Nginx**: We will install and configure the Nginx web server
*   **Installing PHP**: We will install PHP5 to integrate with Nginx
*   **Installing MySQL**: We will install and configure the MySQL database server
*   **Creating dynamic web content**: Using the LEMP stack, we will learn to create simple dynamic web pages

# Installing and configuring Nginx

To begin this chapter, we will need to install the web server Nginx on our CentOS system. Nginx is the new kid on the block in terms of web servers, but in recent surveys from NetCraft, [http://www.netcraft.com](http://www.netcraft.com), we have seen that the Internet has fallen a little out of love with Apache, with Nginx making steady rises since its introduction in 2004\. That said, in May 2014, Apache still had 37 percent of the web server share with Microsoft at 33 percent and Nginx at 14 percent.

## Installing Nginx

Nginx is not part of the standard repositories, but we can use the EPEL repository that we used to install the 389-ds we looked at in [Chapter 7](ch07.html "Chapter 7. LDAP – A Better Type of User"), *LDAP – A Better Type of User*. With the **Extra Packages for Enterprise Linux** (**EPEL**) repository in place, we can install using yum, and then once we have Nginx installed, we can start the service and configure it with `chkconfig` to start on the system boot:

```
# yum install nginx
# service nginx start
# chkconfig nginx on

```

There is a welcome page configured in the default site's configuration that points to `/usr/share/nginx/html/`. We will leave this in place, but will create our own document root soon. We can test the functionality of the web server by browsing to the site `http://localhost` as seen in the following screenshot:

![Installing Nginx](img/5920OS_08_01.jpg)

That was really quite easy, wasn't it! We probably need to replace this web page with one of our own and tidy up some other configurations; but this simple test is good enough to prove the site is up and running.

## Configuring Nginx

The configuration directory for Nginx, or what is referred to as `ServerRoot`, is `/etc/nginx`. The main configuration file is `/etc/nginx/nginx.conf`; however, the web server takes a very modular approach to its configuration, and there is an include statement within the main configuration file that will reference all `.conf` files in `conf.d`. The statement from the `nginx.conf` reads as follows:

```
include /etc/nginx/conf.d/*.conf
```

In this way, it is easy to add in additional configurations without having to edit existing files and risking costly errors. The default configuration that defines the initial server is `/etc/nginx/conf.d/default.conf`. To help understand a little of the anatomy of the Nginx configurations files, let's take a look at the following diagram:

![Configuring Nginx](img/5920OS_08_02.jpg)

To gain the best understanding of the configuration of Nginx, it is often better to start with your own configuration. Start with something simple and add to it. To this end, we will rename `default.conf` to something else and create our own configuration. For a simple server configuration, we need little more than five lines of code:

```
server {
    listen 80;
    root /var/www/html;
    index index.html;
}
```

The previous lines can save this configuration to the new file, `/etc/nginx/conf.d/main.conf`. We will then rename the original configuration `/etc/nginx/conf.d/default.conf` to `/etc/nginx/conf.d/default.conf.old`. It is only `.conf` files that are included so in this way, we can maintain the original configuration without effecting the operation of the web server.

The new configuration that we have is very simple and sparse, and we can explain the limited directives:

*   `listen`: We will listen on all interfaces to TCP port 80.
*   `root`: Here we set the document root to `/var/www/html`. It is better to have variable content like this in the `/var` structure rather than the default location of `/usr`.
*   `index`: We set the default page, often known as the welcome page, to `index.html`. If the URL from the client is entered using the server name only or server name with a directory path without specifying a web page, then the server will look for a page named `index.html`.

We will need to create the directory structure for the document root and fashion a simple web page:

```
# mkdir /var/www/html 
#echo '<h1>Welcome to NGINX</h1>' > /var/www/html/index.html

```

In an ideal world, before restarting the Nginx server, we should endeavor to test any configuration changes. In this way, we can avoid the embarrassing issue where the restart of the server is interrupted by configuration anomalies. When we issue a restart to the service, we will need to first stop and then start Nginx. Stopping the service will not be a problem, but the start might be if we have omitted a semi-colon or other little typo. Consequently, it may be few minutes before we can identify the problem and resume normal service. Having made any changes to the Nginx configuration, we should always test the integrity of these edits before restarting. Using the `/usr/sbin/nginx -t` command, we can perform this check and ensure that if we stop the server, we will be able to start it again. If you prefer, the same test is available by the use of the `service nginx configtest` command.

Other options to the `nginx` command include `-v` for the version of Nginx and `-V` to show the version and configuration options. If we do come across errors, we can check the logfile `/var/log/nginx/error.log`. The `tail` command is often good for this since only the last 10 lines will be shown. The path to the error log is configured in the `/etc/nginx/nginx.conf` with the following line:

```
error_log /var/log/nginx/error.log
```

If required, you could change this to a different logfile; but the default seems reasonable. For now, we can satisfy ourselves that the configuration is OK and restart the service as follows:

```
# nginx -t && service nginx restart

```

### Tip

In this command sequence, the `&&` operator ensures that the restart will only occur if the first command succeeds and the configuration check gave no errors.

We can revisit our site now. It may appear a little less glamorous, but it is all our own work as we can see in the following screenshot:

![Configuring Nginx](img/5920OS_08_03.jpg)

## Configuring a 404 Document Not Found Error page

Another small change we will implement is to control the page not found or HTTP 404 errors. If a user types a page that does not exist, then they will be displayed a very simple error page. We can customize this a little and, at least, give the user a link back to the main index page. If we reedit the configuration file, `/etc/nginx/conf.d/main.conf` it will now read:

```
server {
    listen 80;
    root /var/www/html;
    index index.html;
    error_page 404 not_found.html;
}
```

The extra line `error_page` looks for HTTP 404 errors and returns the page `not_found.html`. We, of course, need to create the page, and it could look similar to this, as a very simple example providing the error and a link to return to the index page:

```
<h2>We could not locate the document</h2>
<a href='/index.html'>Home</a>
```

Remember to test the configuration; we can restart the web server as follows:

```
# nginx -t && service nginx restart

```

Then, access a page that we know does not exist, such as `http://localhost/page1.html`. We should see the new error page, which may look like this if you used my example:

![Configuring a 404 Document Not Found Error page](img/5920OS_08_04.jpg)

Although the design of the web page is simple and bare, we are not trying to teach HTML or CSS tricks here, but more about gaining the idea of how we can use directives in Nginx to issue our own custom error pages.

# Installing PHP

Now that we have the web server up and running, we can add the PHP processor that we need to be able to add PHP elements to our page and subsequently create dynamic web content. Nginx uses the PHP FastCGI Process Manager, which is again available from the EPEL repository. We have that set up already from the Nginx install and the earlier install of 389-ds. To install PHP and the PHP-FPM, we can use `yum`:

```
# yum install php-fpm

```

Once installed, we need to edit the FPM so that it uses the correct accounts for Nginx. To do this, we can edit `/etc/php-fpm.d/www.conf`. We will need to edit the user and group lines from `apache` to `nginx`:

```
user = nginx
group = nginx
```

We also need to make sure that the Nginx web server knows to forward PHP files to the FPM service on port 9000\. We can re-edit `/etc/nginx/conf.d/main.conf` and add it to the server section:

```
server {
 listen 80;
 root /var/www/html;
 index index.html;
 error_page 404 not_found.html;
 location ~ \.php$  {
  fastcgi_pass 127.0.0.1:9000;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name
  include fastcgi_params;
 }

}
```

The code we add is all within the original server block. The file will end correctly with two right braces. We are closing the new location block and the original server code block. Taking stock of the edit, we can see that we have defined a `location` block. This is used when we access a web page that ends with `.php` or, more simply, a PHP page. The definition for the location block looks a little akin to comic book profanities, and that should identify for you that it is in fact a regular expression; the tilde (`~`) denotes that we look for a regular expression match. The expression we search for is URLs that end in `.php`. The `$` symbol denotes the end of the string. The complete expression starts with the escape character `\`; this is needed as the period (dot) has special meaning in regular expressions. To protect this, we use `\` to inform Nginx to read it as literal dot rather than as a regular expression language element. The rest of the code block then denotes that we should pass the PHP code through to the PHP interpreter by means of port 9000 on the local host. The `include` statement reads in a preconfigured file for PHP to set various parameter values.

We can now test and restart the Nginx web server and start the FPM service, configuring it for autostart:

```
# nginx -t && service nginx restart
# service php-fpm start
# chkconfig php-fpm on

```

With a little luck and a following wind, all has been successful, but of course, we do need to test a PHP page now; this can be easily achieved by calling a simple `phpinfo()` function. This is a really simple test that will prove PHP is up and running on your CentOS system. We will return to the Nginx document root directory, `/var/www/html` and create a new page, `info.php`. The PHP extension is important as this is what we look for in the location block: to direct through to the PHP interpreter. The page that we will create could not be any more simple; however, the power behind the function will display a lot in your web browser for very little typing. We can mix HTML code and PHP code in the single file, but we will just use only PHP here. The PHP block starts with `<?php` and ends with the closing tag, `?>`. Each line of PHP code ends with a semi-colon. The `/var/www/html/info.php` file will look like this when edited:

```
<?php
    phpinfo();
?>
```

When we direct the browser to the page with `http://127.0.0.1/info.php`, we should be encouraged with a comprehensive page detailing the configuration of PHP on our CentOS host. See the output from my system in the following screenshot:

![Installing PHP](img/5920OS_08_05.jpg)

Very quickly we have been able to demonstrate the power that lies behind PHP with this simple test. We also can be confident that we have configured PHP correctly with Nginx on our system. We will now add the MySQL database.

# Installing MySQL

MySQL is the open source database solution now managed by Oracle, and of course this is a pivotal component of the LEMP stack that we are implementing. The MySQL server can store data to be displayed later on our web pages. We will communicate from Nginx using PHP with the database server. We can use `yum` to install MySQL and the PHP modules:

```
# yum install php-mysql mysql-server
# service mysqld start
# chkconfig mysqld on

```

Once MySQL is installed, we need to secure the installation a little further; even if it is only to set the MySQL root password. Out of the box security in many systems tends to be a little light. Using the `mysql_secure_installation` command, we can add a little extra security. Running the program will lead you into a simple interactive prompted session:

```
# mysql_secure_installation

```

The resulting wizard will prompt you through the process described in the following:

*   **Enter current password for root**: This is currently blank, so just use the *Enter* key.
*   **Set root password**: We will choose `Y`.
*   **New Password**: Enter the new password twice.
*   **Remove anonymous users**: Choose `Y` for this. In this way, only configured accounts have access.
*   **Disallow root login remotely**: This is usually a good idea, preventing remote MySQL root access. We only need access from this host, so we will answer `Y`.
*   **Remove test database and access to it**: There is a database test that is empty. If we do not need it, we should delete it.
*   **Reload privilege table now**: We will answer `Y` to make these settings effective.

If for nothing else, this is one of the simplest ways to set the MySQL root account password and remind us to verify other settings as we run through the simple script. When we are ready, we can test the operation of the database server from the Linux command line:

```
$ mysql -u root -p -e 'show databases;'

```

We are authenticating as root and will be prompted for the password we set earlier. The `-e` option allows us to execute a MySQL query directly from the command line, and the subsequent query we issue will list all databases. Of course, these will be system databases, as we have not created our own. We can use this same query within a web page to show that we have connectivity to the database from Nginx later. From the query results, we should see two databases listed: `information_schema` and `mysql`.

# Create dynamic web content

To demonstrate how easily we can create dynamic web pages that will connect to the database using PHP from the Nginx server, we will create a new PHP page in `/var/www/html`. So fire up your favorite editor, and we will create the page within the document root, `/var/www/html/db.php`:

```
<h2>Databases</h2>
<?php
      $dbh=mysqli_connect("localhost","root","Password1");
      $result=mysqli_query($dbh, "SHOW DATABASES");
      while ($row = mysqli_fetch_assoc($result)) {
        echo $row['Database'] . "<BR>";
      }
?>
```

The code again is kept as simple as possible, and ideally we would include the connection credentials stored within another file that was not accessible to the web server, allowing access only from the PHP process; however, keeping the code to a minimum does aide the learning process at this early stage.

In this PHP file, you can see that we mix a little HTML code with the PHP code, starting with heading tags before entering into the PHP block. The PHP code first connects to the MySQL server, and then we execute the same query we demonstrated before, from the command line. Before we test this, we will need to restart Nginx and the `php-fpm` service:

```
# service nginx restart
# service php-fpm restart

```

This time, the results will show in the web browser and illustrating that we have a simple dynamic page created from database content. At this stage, we can be happy that we have a LEMP server up and running. With this proof of concept in place, we can now consider building further projects on the LEMP stack, and I certainly hope that you can take a little enthusiasm away with you to read a little more on what you can achieve with PHP and MySQL.

This is the home page that you'll be seeing:

![Create dynamic web content](img/5920OS_08_06.jpg)

# Summary

In this chapter, we have seen how we can build upon the CentOS host and the LEMP stack and implementing Nginx, MySQL, and PHP. Nginx is quite simple to configure but provides faster access to web content than Apache, but can equally be configured to communicate with PHP and MySQL in the backend. Gaining the basic knowledge of configuring PHP and MySQL to operate with the web server can build the grounding you need to develop your web application further.

In the following chapter, we will see how to implement Puppet on CentOS as a central configuration server to allow configuration changes to be made on one server to replicate to other configured clients.