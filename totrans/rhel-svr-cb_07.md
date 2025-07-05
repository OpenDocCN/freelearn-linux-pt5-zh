# Chapter 7. Puppet Configuration Management

The recipes that are covered in this chapter are:

*   Installing and configuring Puppet Master
*   Installing and configuring Puppet agent
*   Defining a simple module to configure time
*   Defining nodes and node grouping
*   Deploying modules to single nodes and node groups

# Introduction

Puppet is an "old school" configuration management tool. It helps you enforce configurations with great ease although it is more complex than Ansible to use. Puppet's declarative language can be compared to a programming language and is difficult to master. However, once you understand how it works, it's fairly easy to use.

Puppet is very good at maintaining a strict set of configurations, but if you aim at verifying the configurations before applying them, you'll find that Puppet is not the sharpest tool in the shed. Puppet does have the `audit` metaparameter that you can use in your resources to track changes, but it doesn't let you display where it differs from your manifest. In fact it doesn't allow you to add the `audit` metaparameter to your "active" module or manifests. It sits in a separate manifest that audits the requested resources.

The version of Puppet used in these recipes is v3.8 and covers the community edition.

# Installing and configuring Puppet Master

The people at Puppet Labs have their own repository servers for puppet, which is very easy when it comes down to installing and maintaining the server and agent. Although the EPEL repository also provides puppet packages, they tend to be old or not up to date. Hence, I recommend using the Puppet Labs' yum repositories.

## How to do it…

This recipe covers a monolithic install. Perform the following steps:

1.  Enable the optional channel via the following command; you'll need this to install the Puppet Server component:

    ```
    ~]# subscription-manager repos --enable rhel-6-server-optional-rpms

    ```

2.  Download the `puppetlabs` repository installer, as follows:

    ```
    ~]# curl -Lo /tmp/puppetlabs-release-el-7.noarch.rpm https://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm

    ```

3.  Now, install the `puppetlabs` repository by executing the following:

    ```
    ~]# yum install -y /tmp/puppetlabs-release-el-7.noarch.rpm

    ```

4.  Install `puppet-server` by typing out this command:

    ```
    ~]# yum install -y puppet-server

    ```

5.  Set up Puppet Master by adding the following to the `[main]` section of `/etc/puppet/puppet.conf`:

    ```
    dns_alt_names = puppetmaster.critter.be,rhel7.critter.be
    always_cache_features = true
    ```

6.  Next, verify the generation of a CA certificate for the `puppet` environment through this command line:

    ```
    ~]# puppet master --verbose --no-daemonize

    ```

7.  Press *CTRL* + *C* when it displays the following information:

    ```
    Notice: Starting Puppet master version <version number>

    ```

8.  Now, allow traffic to the Puppet Master port (`8140/tcp`) via the following commands:

    ```
    ~]# firewall-cmd --permanent –add-port=8140/tcp
    ~]# firewall-cmd --reload

    ```

9.  Start Puppet Master by executing the following:

    ```
    ~]# systemctl start puppetmaster

    ```

10.  Finally, enable Puppet Master at boot, as follows:

    ```
    ~]# systemctl enable puppetmaster

    ```

## There's more…

The basic HTTP daemon that Puppet Master uses is not made to provide service for an enterprise. Puppet Labs recommends using Apache with Passenger to provide the same service as Puppet Master for a bigger range of systems (more than 10).

You can either compile the Passenger module yourself, or you can just use `EPEL` (for the `rubygem(rack)` package) and the Passenger repository. I choose the latter. Here are the steps that you need to perform:

1.  Install the Passenger repository by running the following command:

    ```
    curl -Lo /etc/yum.repos.d/passenger.repo https://oss-binaries.phusionpassenger.com/yum/definitions/el-passenger.repo

    ```

2.  Now, download the EPEL repository installer, as follows:

    ```
    ~]# curl -Lo /tmp/epel-release-latest-7.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

    ```

3.  Install the `rpm` EPEL repository (with `yum`) via the following command:

    ```
    ~]# yum install -y /tmp/epel-release-latest-7.noarch.rpm

    ```

4.  Next, install the necessary packages for the Puppet web interface. For this, you can execute the following command line:

    ```
    ~]# yum install -y httpd mod_ssl mod_passenger

    ```

5.  Set up Puppet Master's virtual host directories and ownership, as follows:

    ```
    ~]# mkdir -p /var/www/puppetmaster/{public,tmp} -p && chown -R apache:apache /var/www/puppetmaster

    ```

6.  Copy the `rack` configuration file to Puppet Master's virtual host root using the following command:

    ```
    ~]# cp /usr/share/puppet/ext/rack/config.ru /var/www/puppetmaster/.

    ```

7.  Next, change the ownership of the `config.ru` file. This is very important! You can do this through the following command:

    ```
    ~#] chown -R puppet:puppet /var/www/puppetmaster/config.ru

    ```

8.  Then, create an Apache virtual host configuration file at `/etc/httpd/conf.d/puppetmaster.conf` containing the following:

    ```
    # passenger performance tuning settings:
    # Set this to about 1.5 times the number of CPU cores in your master:
    PassengerMaxPoolSize 3
    # Recycle master processes after they service 1000 requests
    PassengerMaxRequests 1000
    # Stop processes if they sit idle for 10 minutes
    PassengerPoolIdleTime 600

    Listen 8140
    <VirtualHost *:8140>
        # Make Apache hand off HTTP requests to Puppet earlier, at the cost of
        # interfering with mod_proxy, mod_rewrite, etc. See note below.
        PassengerHighPerformance On

        SSLEngine On

        # Only allow high security cryptography. Alter if needed for compatibility.
        SSLProtocol ALL -SSLv2 -SSLv3
        SSLCipherSuite EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH:+CAMELLIA256:+AES256:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA:kEDH:CAMELLIA256-SHA:AES256-SHA:CAMELLIA128-SHA:AES128-SHA
        SSLHonorCipherOrder     on

        SSLCertificateFile      /var/lib/puppet/ssl/certs/rhel7.critter.be.pem
        SSLCertificateKeyFile   /var/lib/puppet/ssl/private_keys/rhel7.critter.be.pem
        SSLCertificateChainFile /var/lib/puppet/ssl/ca/ca_crt.pem
        SSLCACertificateFile    /var/lib/puppet/ssl/ca/ca_crt.pem
        SSLCARevocationFile     /var/lib/puppet/ssl/ca/ca_crl.pem
        SSLCARevocationCheck   chain
        SSLVerifyClient         optional
        SSLVerifyDepth          1
        SSLOptions              +StdEnvVars +ExportCertData

        # Apache 2.4 introduces the SSLCARevocationCheck directive and sets it to none
        # which effectively disables CRL checking. If you are using Apache 2.4+ you must
        # specify 'SSLCARevocationCheck chain' to actually use the CRL.

        # These request headers are used to pass the client certificate
        # authentication information on to the Puppet master process
        RequestHeader set X-SSL-Subject %{SSL_CLIENT_S_DN}e
        RequestHeader set X-Client-DN %{SSL_CLIENT_S_DN}e
        RequestHeader set X-Client-Verify %{SSL_CLIENT_VERIFY}e

        DocumentRoot /var/www/puppetmaster/public

        <Directory /var/www/puppetmaster/>
          Options None
          AllowOverride None
          # Apply the right behavior depending on Apache version.
          <IfVersion < 2.4>
            Order allow,deny
            Allow from all
          </IfVersion>
          <IfVersion >= 2.4>
            Require all granted
          </IfVersion>
        </Directory>

        ErrorLog /var/log/httpd/puppetmaster_ssl_error.log
        CustomLog /var/log/httpd/puppetmaster_ssl_access.log combined
    </VirtualHost>
    ```

    ### Tip

    Make sure that you replace the certificate directives with the certificate file paths of your own system.

9.  Disable the `puppetmaster` service via the following:

    ```
    ~]# systemctl disable puppetmaster

    ```

10.  Use the following command line to stop the `puppetmaster` service:

    ```
    ~]# systemctl stop puppetmaster

    ```

11.  Now, start Apache, as follows:

    ```
    ~]# systemctl start httpd

    ```

12.  Enable Apache on boot through the following command line:

    ```
    ~]# systemctl enable httpd

    ```

13.  Check your HTTP daemon's status using the following:

    ```
    ~]# systemctl status httpd

    ```

    This will result in the following (similar) output:

    ![There's more…](img/00051.jpeg)

Puppet can also run in a masterless mode. In this case, you don't install a server but only the clients on all the systems that you wish to manage in this way.

## See also

For more in-depth information about installing Puppet on RHEL, refer to the following page:

[https://docs.puppetlabs.com/guides/install_puppet/install_el.html](https://docs.puppetlabs.com/guides/install_puppet/install_el.html)

# Installing and configuring the Puppet agent

Unlike Ansible, Puppet requires an agent to be able to enforce configurations. This recipe will teach you how to install and configure the puppet agent on a system. The only way to mass deploy the Puppet agent is through an orchestration tool (such as Ansible).

## How to do it…

The Puppet agent can be installed and maintained using the same repository as the Puppet server: the Puppet Labs repository. Perform the following steps:

1.  Download the Puppet Labs repository installer via the following command:

    ```
    ~]# curl -Lo /tmp/puppetlabs-release-el-7.noarch.rpm https://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm

    ```

2.  Install the Puppet Labs repository by executing the following command:

    ```
    ~]# yum install -y /tmp/puppetlabs-release-el-7.noarch.rpm

    ```

3.  Use the following command to download the EPEL repository installer:

    ```
    ~]# curl -Lo /tmp/epel-release-latest-7.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

    ```

4.  Now, install the `rpm` EPEL repository (with `yum`) through the following command line:

    ```
    ~]# yum install -y /tmp/epel-release-latest-7.noarch.rpm

    ```

5.  Install the Puppet agent; you can run the following command:

    ```
    ~]# yum install -y puppet

    ```

6.  Next, configure the agent so that it will connect to your Puppet Master.
7.  Add your Puppet Master to the `[main]` section of `/etc/puppet/puppet.conf`, as follows:

    ```
    server = rhel7.critter.be
    ```

8.  Start the Puppet agent by executing the following command:

    ```
    ~]# systemctl start puppet

    ```

9.  Then, enable the Puppet agent by running the following:

    ```
    ~]# systemctl enable puppet

    ```

10.  Finally, sign the new node's certificate on Puppet Master, as follows:

    ```
    ~]# puppet cert sign rhel7-client.critter.be

    ```

## There's more…

Instead of signing every single certificate individually, you can sign the certificate for all systems that have been registered with Puppet Master by executing the following:

```
~]# puppet cert sign –all

```

If you start looking for puppet unit files in `/lib/systemd/system`, you'll also find a `puppetagent.service` unit file. The `puppetagent.service` unit file is actually a soft link to the `puppet.service` unit file.

If you don't want to set the server property in the `/etc/puppet/puppet.conf` file, you can do this by defining a `puppet` DNS entry that points to Puppet Master in all the DNS domain zones.

The Puppet agent is known to consume memory. In order to mitigate this, the Puppet agent can be run as a cron job. This would release some memory, but you would lose the flexibility of pushing new configurations from Master.

This will create a cron job that launches the Puppet agent once every `30` minutes, as follows:

```
~]# puppet resource cron puppet-agent ensure=present user=root minute=30 command='/usr/bin/puppet agent --onetime --no-daemonize --splay'

```

The Puppet agent can also be configured to run in the `Masterless` mode. This means that you will take care of distributing your puppet modules and classes yourself instead of Puppet taking care of this. This implies that you will synchronize all modules and classes, even those that are not used by the system, which can be a security risk.

# Defining a simple module to configure time

Modules are collections of manifests and files that define how to install and configure various components. Manifests contain the instructions to apply to a system's configuration. In this recipe, we'll create a simple module to install and configure the NTP daemon.

## Getting ready

Puppet has a strict way of organizing modules. Your modules should always be stored in `/etc/puppet/modules`. Every module is a directory within this directory, containing the necessary directories that in turn contain manifests, files, templates, and so on.

## How to do it…

In this recipe, we'll create the necessary directory structure, manifests, and files to configure your system's time. Perform the following steps:

1.  Create `ntp/manifests` in `/etc/puppet/modules` via the following command:

    ```
    ~]# mkdir -p /etc/puppet/modules/ntp/manifests

    ```

2.  Create `ntp/templates` to house all the templates used by the puppet module through the following:

    ```
    ~]# mkdir -p /etc/puppet/modules/ntp/templates

    ```

3.  Now, create the `install.pp` file in `/etc/puppet/modules/ntp/manifests` with the following contents:

    ```
    class ntp::install inherits ntp {
      package { 'ntp':
        ensure => installed,
      }
    }
    ```

4.  Create the `config.pp` file in `/etc/puppet/modules/ntp/manifests` with the following contents:

    ```
    class ntp::config inherits ntp {
      file { '/etc/ntp.conf':
        ensure  => file,
        owner   => 'root',
        group   => 'root',
        mode    => 0644,
        content => template("ntp/ntp.conf.erb"),
      }
    }
    ```

5.  Next, create the `ntp.conf.erb` template file in `/etc/puppet/modules/ntp/templates` with the following contents:

    ```
    driftfile /var/lib/ntp/drift

    restrict default nomodify notrap nopeer noquery

    restrict 127.0.0.1
    restrict ::1

    server 0.be.pool.ntp.org iburst
    server 1.be.pool.ntp.org iburst
    server 2.be.pool.ntp.org iburst
    server 3.be.pool.ntp.org iburst

    includefile /etc/ntp/crypto/pw

    keys /etc/ntp/keys

    disable monitor
    ```

6.  Create the `service.pp` file in `/etc/puppet/modules/ntp/manifests` with the following contents:

    ```
    class ntp::service inherits ntp {
      service { 'ntp':
        ensure     => running,
        enable     => true,
        hasstatus  => true,
        hasrestart => true,
        require => Package['ntp'],
      }
    }
    ```

7.  Finally, create the `init.pp` file that binds them all together in `/etc/puppet/modules/ntp/manifests` with the following contents:

    ```
    class ntp {
        include ntp::install
        include ntp::config
        include ntp::service
    }
    ```

## How it works...

When applying a module to a system, it applies the directives found in the module's `init.pp` manifest.

As you can see, we created a template file that is "automagically" distributed to the clients. Puppet automatically creates a file share for the `templates` and `files` directories.

As you can see in the `config.pp` file, the template references `ntp/ntp.conf.erb`. Puppet will automatically resolve this to the correct location (`ntp/templates/ntp.conf.erb`).

## There's more...

I created four manifests to install and configure Puppet. This could be easily achieved by just creating one monolithic `init.pp` manifest with the contents of the other three files. When you start creating complex manifests, you'll be happy to have split them up.

If you want to have a single location for all the assets (templates and files) you use in your modules, you will have to define a separate file share for this location in the `/etc/puppet/fileserver.conf` file, as follows:

```
[mount_point]
    path /path/to/files
    allow *
```

## See also

Read up on Puppet Modules through the link [https://docs.puppetlabs.com/puppet/3.8/reference/modules_fundamentals.html](https://docs.puppetlabs.com/puppet/3.8/reference/modules_fundamentals.html).

# Defining nodes and node grouping

In order to push a manifest, its classes, and assets to systems, they need to be known by Puppet Master. Grouping is practical if you want to push a manifest to a number of hosts without having to modify each configuration node.

## How to do it…

In contrast to what the title wants you to believe, you cannot create a group and add nodes. However, you can group nodes and make them behave in a similar way to groups.

Nodes and node groups are defined in `/etc/puppet/manifests/site.pp` or a file at `/etc/puppet/manifests/site.pp`.

### Create the configuration node

Create a `/etc/puppet/manifests/site.pp/rhel7-client.pp` file with the following contents:

```
node 'rhel7-client.critter.be' {
}
```

### Create a node group

Create a `/etc/puppet/manifests/site.pp/rhel7-clientgroup.pp` file with the following contents:

```
node 'rhel7-client00.critter.be', 'rhel7-client01.critter.be', 'rhel7-client02.critter.be' {
}
```

## There's more…

If you have a strict naming convention, you can use `regular expressions` to define your node group. Run the following commands:

```
node /^www[0-9]+\.critter\.be$/ {
}
node /^repo[0-9]+\.critter\.be$/ {
}
```

By default, node names are defined by their certificate name, which is **FQDN** (**Fully Qualified Domain Name**) of the system we used to register with Puppet Master.

If you don't remember the names of all of your nodes, you can easily find them at `/var/lib/puppet/ssl/ca/signed/`.

# Deploying modules to single nodes and node groups

Once you define modules and nodes, you can start deploying the modules to your nodes. You can do this on various levels, which will be demonstrated in the following recipe.

## How to do it…

In order to deploy a module (or manifest) to a node, your must configure this in the node's stanza or a group of nodes that the node belongs to, or you can define it on the base level to apply it to every node.

### Configure to deploy a module or manifest to a single client

Edit the client configuration node from the previous recipe and add an include statement referring to manifest you want to be applied to the client block. You can execute the following command for this:

```
node 'rhel7-client.critter.be' {
  include ntp
}
```

### Configure to deploy a module or manifest to a node group

In the same way you edited the single node file, edit the node group configuration file and add an include statement to the node group block referring to the manifest you want applied. Take a look at the following command:

```
node 'rhel7-client0.critter.be', 'rhel7-client1.critter.be', 'rhel7-client2.critter.be' {
  include ntp
}
```

### Configure to deploy to all registered systems

One will typically have a node configuration file within `/etc/puppet/manifests/site.pp/`, or `/etc/puppet/manifests/site.pp` itself, if you work with one monolithic site definition, which affects all nodes. Edit `/etc/puppet/manifests/site.pp/default.pp` and enter the following code:

```
include ntp
```

### Deploy to a system

On the system with the Puppet Agent installed, execute the following:

```
~]# puppet agent –-test

```

When executed, the following will appear:

![Deploy to a system](img/00052.jpeg)

## There's more…

For testing purposes, there's an alternative to defining nodes and including modules.

Copy the manifest(s), files, and templates to your test machine (usually, you will develop elsewhere than the production Puppet Master anyway) and execute them in the following way:

```
~]# puppet apply /path/to/manifest.pp

```

### Tip

By default, Puppet applies all manifests found in `/etc/puppet/manifests/site.pp`. As explained in the preceding section, this doesn't need to be a single monolithic file containing all your directives. When using it as a directory, it uses all the manifests found within this directory, or if the name of a subdirectory ends with `.pp`, it interprets all of its contents as manifests as well. It interprets all files alphanumerically.