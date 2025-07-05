# Chapter 6. Orchestrating with Ansible

In this chapter, the following recipes will be addressed:

*   Installing Ansible
*   Configuring the Ansible inventory
*   Creating the template for a kickstart file
*   Creating a playbook to deploy a new VM with kickstart
*   Creating a playbook to perform system configuration tasks
*   Troubleshooting Ansible

# Introduction

Ansible is an easy-to-use agentless system configuration management tool. It allows us to deploy complex configurations without the hassle of a complex interface or language.

Ansible uses playbooks, which are collections of tasks to deploy configurations and applications to multiple nodes over SSH in a controlled way. However, it doesn't stop there.

Ansible's modules, which are used to execute tasks, are all built to be idempotent in their execution.

The definition of Idempotence, according to Wikipedia, is as follows:

> *Idempotence (/ˌaɪdɨmˈpoʊtəns/ eye-dəm-poh-təns [citation needed]) is the property of certain operations in mathematics and computer science that can be applied multiple times without changing the result beyond the initial application.*

In short, any module will detect the changes to be applied and perform them. If it doesn't need to change anything, it will not reapply the requested changes or interfere with file metadata.

The Ansible company also provides Tower, a paid subscription with extra features, as an add-on to Ansible. Tower provides a graphical interface to control your Ansible orchestration tool. However, this is out of the scope of this chapter.

# Install Ansible

Ansible is not in the default RHEL 7 repositories, but in this recipe, I will show you how to install it in several ways.

## Getting ready

Ansible needs the following packages installed:

*   Python v2.7 (Ansible doesn't support v3 yet)
*   `python-httplib2`
*   `python-jinja2`
*   `python-paramiko`
*   `python-setuptools`
*   `PyYAML`

So, in order to achieve this, execute the following command:

```
~]# yum install -y python-httplib2 python-jinja2 python-keyczar python-paramiko python-setuptools PyYAML

```

As RHEL 7 and some other major distributions come preinstalled with Python (yum requires it, as do most of the Red Hat tools), we don't have to include it in the preceding command.

## How to do it…

In this recipe, I will cover the three most used methods of installing Ansible.

### Installing the latest tarball

This method is quite simple as you just download the tarball and extract it in a location of your choosing. Perform the following steps:

1.  Grab the latest tarball located at [http://releases.ansible.com/ansible/](http://releases.ansible.com/ansible/) via the following command:

    ```
    ~]$ curl -o /tmp/ansible-latest.tar.gz http://releases.ansible.com/ansible/ansible-latest.tar.gz
     % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
     Dload  Upload   Total   Spent    Left  Speed
    100  905k  100  905k    0     0   870k      0  0:00:01  0:00:01 --:--:--  870k
    ~]$

    ```

2.  Extract the tarball to `/opt`, as follows:

    ```
    ~]# tar zxf /tmp/ansible-latest.tar.gz -C /opt/

    ```

3.  Now, create a symbolic link for easy access using this command:

    ```
    ~]# ln -s /opt/ansible-1.9.2 /opt/ansible

    ```

4.  Add the Ansible binaries and man pages to your environment's path by executing the following:

    ```
    ~]# cat << EOF > /etc/profile.d/ansible.sh
    # Ansible-related stuff
    export ANSIBLE_HOME=/opt/ansible
    export PATH=\${PATH-""}:${ANSIBLE_HOME}/bin
    export MANPATH=\${MANPATH-""}:${ANSIBLE_HOME}/docs/man
    export PYTHONPATH=\${PYTHONPATH-""}:${ ANSIBLE_HOME}/lib
    EOF
    ~]#

    ```

5.  Next, source the Ansible PATH and MANPATH by running this command line:

    ```
    ~]# . /etc/profile.d/ansible.sh

    ```

6.  Finally, use the following command to regenerate the man pages:

    ```
    ~]# /etc/cron.daily/man-db.cron

    ```

### Installing cutting edge from Git

Git makes keeping your local copy of Ansible up to date quite simple.

It automatically updates/removes files where needed. Perform the following steps:

1.  Make sure `git` is installed using this command:

    ```
    ~]# yum install -y git

    ```

2.  Clone the Ansible `git` repository to `/opt`, as follows:

    ```
    ~]# cd /opt
    ~]# git clone git://github.com/ansible/ansible.git --recursive

    ```

    ![Installing cutting edge from Git](img/00050.jpeg)
3.  Add the Ansible binaries and man pages to your environment's path, through the following command:

    ```
    ~]# cat << EOF > /etc/profile.d/ansible.sh
    # Ansible-related stuff
    export ANSIBLE_HOME=/opt/ansible
    export PATH=\${PATH-""}:${ANSIBLE_HOME}/bin
    export MANPATH=\${MANPATH-""}:${ANSIBLE_HOME}/docs/man
    export PYTHONPATH=\${PYTHONPATH-""}:${ ANSIBLE_HOME}/lib
    EOF
    ~]#

    ```

4.  Now, source the Ansible PATH and MANPATH via this command:

    ```
    ~]# . /etc/profile.d/ansible.sh

    ```

5.  Finally, using the following line, regenerate the man pages:

    ```
    ~]# /etc/cron.daily/man-db.cron

    ```

### Installing Ansible from the EPEL repository

Installing from a repository has the advantage that you can keep your version of Ansible up to date along with your system. Here are the steps you need to perform:

1.  Install the extra packages for the **Enterprise Linux** (**EPEL**) repository from [https://fedoraproject.org/wiki/EPEL](https://fedoraproject.org/wiki/EPEL) via this command:

    ```
    ~]# yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

    ```

2.  Now, install Ansible using yum, as follows:

    ```
    ~]# yum install -y ansible

    ```

## There's more…

If you want to keep your Git clone up to date, remember that the sources tree also contains two subtrees. You'll have to execute the following:

```
~]# git pull --release
~]# git submodule update --init --recursive

```

# Configuring the Ansible inventory

The Ansible inventory is the heart of the product as it provides a lot of variables about your environment to the deployment mechanism. These variables are known as `facts` and serve Ansible to make decisions, template text-based files, and so on.

## How to do it…

There are several ways of adding information about your environment to your inventory.

### The static inventory file

The static inventory is basically a mini-formatted file containing the definitions for hosts and groups. Here's what you need to do:

1.  Create `/etc/ansible/hosts` with the following contents:

    ```
    ~]# cat << EOF >> /etc/ansible/hosts
    localhost         ansible_connection=local
    srv1.domain.tld   ansible_connection=ssh ansible_ssh_user=root

    [mail]
    mail[01..50].domain.tld

    [mail:vars]
    dns_servers=[ '8.8.8.8', '8.8.4.4' ]
    mail_port=25
    EOF
    ~]#

    ```

### The dynamic inventory file

The dynamic inventory file has to be an executable file, generating a JSON string containing information about your hosts and groups. Follow these steps::

1.  Create an `~/inventory.py` script with the following contents:

    ```
    -]# cat << EOF >> ~/inventory.py
    #!/usr/bin/python -tt
    # -*- coding: utf-8 -*-
    # vim: tabstop=8 expandtab shiftwidth=4 softtabstop=4
    import json

    def main():
     inventory = {
     '_meta': {
     'hostvars': {
     'localhost': {
     'ansible_connection': 'local' },
     'srv1.domain.tld': {
     'ansible_connection': 'ssh',
     'ansible_ssh_user': 'root' },
     }
     },
     'all': {
     'hosts': [
     'localhost',
     'srv1.domain.tld' ] },
     'mail': {
     'hosts': [],
     'vars': {
     'dns_servers': [ '8.8.8.8', '8.8.4.4' ],
     'mail_port': 25} }
     }

     for x in range(1,50):
     hostname = 'mail' + ('00%d' % x)[-2:] + '.domain.tld'
     inventory['_meta']['hostvars'].update({ hostname: {} })
     inventory['mail']['hosts'].append(hostname)

     print json.dumps(inventory, sort_keys=True, indent=4, separators=(',',': '))

    if __name__ == '__main__':
     main()
    ~]#

    ```

2.  Now, make the script executable, as follows:

    ```
    ~]# chmod +x ~/inventory.py

    ```

### host_vars files

A `host_vars` file is a `yml`-formatted one containing extra facts, which will only be applied to the host with the same name as the file. Simply do the following:

1.  Create a `host_vars` file for `srv1.domain.tld` through this command:

    ```
    ~]# cat << EOF >> ~/host_vars/srv1.domain.tld.yml
    ansible_connection: ssh
    ansible_ssh_user: root
    EOF
    ~]#

    ```

### group_vars files

Like `host_vars`, `group_vars` files are `yml`-formatted ones containing extra facts. These will be applied to the group with the same name as the file. Perform the following:

1.  Create a `group_vars` file for mail via the following command:

    ```
    ~]# cat << EOF >> ~/group_vars/mail.yml
    dns_servers: [ '8.8.8.8', '8.8.4.4' ]
    mail_port: 25
    EOF
    ~]#

    ```

## How it works…

The inventory file location is set in the Ansible configuration file—look for the line starting with `hostfile` within the `defaults` section. This file is either a static file, or a script returning a JSON-formatted list of hosts and groups, as shown in the preceding recipe. Ansible automatically detects whether a file is a script and treats it this way to import information.

There is one caveat, however: the script needs to show the JSON-formatted information by specifying `--list`.

Ansible can automatically combine the inventory with the `host_vars` and `group_vars` files if the latter two directories are in the same directory as the inventory file / script. Take a look at the following:

```
/etc/ansible/hosts
/etc/ansible/host_vars
/etc/ansible/host_vars/srv1.domain.tld.yml
/etc/ansible/host_vars/...
/etc/ansible/group_vars
/etc/ansible/group_vars/mail.yml
/etc/ansible/group_vars/...
```

The same can be achieved by putting the `host_vars` and `group_vars` directories in the same directory as the playbook you are executing.

### Tip

The facts in `host_vars` and `group_vars` take priority over the variables returned through the inventory.

## There's more…

Ansible already seeds the inventory with the facts that it retrieves from the host itself. You can easily find out which facts Ansible prepares for your use by executing the following command:

```
~]# ansible -m setup <hostname>

```

This will produce a lengthy JSON-formatted output with all the facts Ansible knows about your destination host.

If you want even more information, on RHEL systems, you can install `redhat-lsb-core` to have access to LSB-specific facts.

Enterprises tend to have databases containing information regarding all their systems for change management. This is an excellent source for the inventory script to get its information.

## See also

If you want more detailed information about the Ansible inventory, go to [http://docs.ansible.com/ansible/intro_inventory.html](http://docs.ansible.com/ansible/intro_inventory.html).

Shameless self-promotion for a personal project and a tool to automate the inventory calls for a mention of [https://github.com/bushvin/inventoryd/](https://github.com/bushvin/inventoryd/).

# Creating a template for a kickstart file

A `template` is one of the core modules of Ansible. It is used to easily generate files (for example, configuration files) based on a common set of facts. It uses the Jinja2 template engine to interpret template files.

For this recipe, we'll use a simple `kickstart` script that is generic enough to deploy any host. Refer to [Chapter 2](part0025_split_000.html#NQU21-501a83dd54944cb1bf060a2ce9fab11f "Chapter 2. Deploying RHEL "En Masse""), *Deploying RHEL "En Masse"*, to find out about `kickstart` files.

## Getting ready

The facts that we need for this host are `repo_url`, `root_password_hash`, `ntp_servers`, `timezone`, `ipv4_address`, `ipv4_netmask`, `ipv4_gateway`, and `dns_servers`.

## How to do it…

Create the `kickstart` file in your playbook's template folder (`~/playbooks/templates/kickstart/rhel7.ks`) with the following content:

```
install
url --url={{ repo_url }}
skipx
text
reboot
lang en_US.UTF-8
keyboard us
selinux --enforcing
firewall --enabled --ssh
rootpw –iscrypted {{ root_password_hash }}
authconfig --enableshadow --passalgo=sha512
timezone --utc --ntpservers {{ ntp_servers|join(',') }} {{ timezone }}
zerombr
clearpart --all
bootloader --location=mbr --timeout=5
part /boot --asprimary --fstype="xfs" --size=1024 --ondisk=sda
part pv.1   --size=1 --grow --ondisk=sda
volgroup {{ hostname }}_system pv.1
logvol / --vgname={{ inventory_hostname }}_system --size=2048 --name=root --fstype=xfs
logvol /usr --vgname={{ inventory_hostname }}_system --size=2048 --name=usr --fstype=xfs
logvol /var --vgname={{ inventory_hostname }}_system --size=2048 --name=var --fstype=xfs
logvol /var/log --vgname={{ inventory_hostname }}_system --size=2048 --name=varlog --fstype=xfs
logvol swap --vgname={{ inventory_hostname }}_system --recommended --name=swap --fstype=swap
network --device=eth0 --bootproto=static --onboot=yes --activate --ip={{ ipv4_address }} --netmask={{ ipv4_netmask }} --gateway={{ ipv4_gateway }} --nameserver={{ dns_servers|join(',') }}
%packages --excludedocs
@Core
vim-enhanced
%end
```

## How it works…

The Jinja2 engine replaces all the variables enclosed by `{{ }}` with whichever facts are available for the specified host in the inventory, resulting in a correct `kickstart` file, assuming all variables have been correctly set.

## There's more…

Jinja2 can do more than just replace variables with whatever is in the inventory. It was originally developed as a rich templating language for web pages and supports major features such as conditions, loops, and so on.

Using Jinja, you can easily loop over a list or array within the inventory and use the resultant variable or even dictionaries and objects. For example, consider that your host has the following fact:

```
{ 'nics': [
    { 'device': 'eth0', 'ipv4': { 'address':'192.168.0.100', 'netmask':'255.255.255.0','gateway':'192.168.0.1'} },
    { 'device': 'eth1', 'ipv4': { 'address':'192.168.1.100', 'netmask':'255.255.255.0','gateway':'192.168.1.1'} } ] }
```

This would allow you to replace the network portion of your `kickstart` script with the following:

```
{% for nic in nics %}
network –device={{ nic.device }} --bootproto=static --onboot=yes --activate --ip={{ nic.ipv4.address }} --netmask={{ nic.ipv4.netmask }} --gateway={{ nic.ipv4.gateway }}
{% endfor %}
```

There is one consideration with provisioning new systems such as this and the inventory: you can only use the facts that you have introduced yourself, not those that Ansible gets from the system. This is because firstly, they don't exist yet, and secondly, the task is executed on a different host.

## See also

For more information about templating with Ansible, read the Jinja2 Template Designer documentation at [http://jinja.pocoo.org/docs/dev/templates/](http://jinja.pocoo.org/docs/dev/templates/).

For more information on the Ansible template module, go to [http://docs.ansible.com/ansible/template_module.html](http://docs.ansible.com/ansible/template_module.html).

# Creating a playbook to deploy a new VM with kickstart

Creating playbooks for Ansible is a relatively easy task as most considerations are handled by the modules. All modules are made as "idempotently" as possible, meaning that a module first checks what it is supposed to do with what has been done on the system and only then applies the changes if they are different.

## Getting ready

We don't need any additional facts for this recipe.

For this to work, we need to have a web server and a location to store the `kickstart` files, which will be served by the web server.

For the sake of convenience, our web server is called `web.domain.tld`, the location on this web server is `/var/www/html/kickstart`, and this directory can be accessed through `http://web.domain.tld/kickstart`.

We also need a KVM host (refer to [Chapter 1](part0015_split_000.html#E9OE1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 1. Working with KVM Guests"), *Working with KVM Guests*, on how to set up a KVM server). In this case, we'll call our KVM server `kvm.domain.tld`.

## How to do it…

Let's create the playbook that will provision new systems via the following steps:

1.  Create a `~/playbooks/provisioning.yml` playbook with the following contents:

    ```
    - name: Provision new machines
      hosts: all
      gather_facts: no
      tasks:
      - name: Publish kickstart template as new file to webserver
        action: template src=templates/kickstart/rhel7.ks dest=/var/www/html/kickstart/{{ inventory_hostname }}.ks
                         owner=apache group=apache mode=0644
                         seuser=system_u serole=object_r setype=httpd_sys_content_t selevel=s0
        delegate_to: web.domain.tld

      - name: Create new isolinux file to contain reference to the kickstart file
        action: template src=templates/isolinux/isolinux.cfg.el7 dest=/root/iso/isolinux/isolinux.cfg
        delegate_to: kvm.domain.tld

      - name: Create new iso boot media
        action: shell cd /root/iso; mkisofs -o /tmp/{{ inventory_hostname }}.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -J -r .
        delegate_to: kvm.domain.tld

      - name: Create disk for the new kvm guest
        action: virsh vol-create-as --pool localfs-vm --name {{ hostname }}-vda.qcows2 --format qcows2 --capacity 15G
        delegate_to: kvm.domain.tld

      - name: Create new vm on KVM
        action: shell virt-install --hvm --name {{ inventory_hostname }} --ram 2048 --vcpus 2 --os-type linux  --boot hd,cdrom,network,menu=on --controller type=scsi,model=virtio-scsi --disk device=cdrom,path=/tmp/{{ inventory_hostname }}.iso,readonly=on,bus=scsi --disk device=disk,vol=localfs-vm/{{ inventory_hostname }}-vda.qcows2,cache=none,bus=scsi --network network=bridge-eth0,model=virtio --graphics vnc --graphics spice --noautoconsole --memballoon virtio
        delegate_to: kvm.domain.tld
    ```

2.  You'll also need to create the template for the `~/templates/isolinux/isolinux.cfg.el7` file; you can do this by executing the following:

    ```
    default vesamenu.c32
    timeout 600
    display boot.msg
    menu clear
    menu background splash.png
    menu title Red Hat Enterprise Linux 7.0
    menu vshift 8
    menu rows 18
    menu margin 8
    menu helpmsgrow 15
    menu tabmsgrow 13
    menu color sel 0 #ffffffff #00000000 none
    menu color title 0 #ffcc000000 #00000000 none
    menu color tabmsg 0 #84cc0000 #00000000 none
    menu color hotsel 0 #84cc0000 #00000000 none
    menu color hotkey 0 #ffffffff #00000000 none
    menu color cmdmark 0 #84b8ffff #00000000 none
    menu color cmdline 0 #ffffffff #00000000 none
    label linux
      menu label ^Install Red Hat Enterprise Linux 7.0
      kernel vmlinuz
      append initrd=initrd.img ks=http://web.domain.tld/kickstart/{{ inventory_hostname }}.ks text

    label local
      menu label Boot from ^local drive
      localboot 0xffff

    menu end
    ```

3.  Now, use the following command to execute the playbook:

    ```
    ~]# ansible-playbook --limit newhost ~/playbooks/provisioning.yml

    PLAY [Provision new machines] ********************************

    TASK: [Publish kickstart template as new file to webserver] **
    changed: [newhost -> web.domain.tld]

    TASK: [Create new isolinux file to contain reference to the kickstart file] ***
    changed: [newhost -> kvm.domain.tld]

    TASK: [Create new iso boot media] ****************************
    changed: [newhost -> kvm.domain.tld]

    TASK: [Create disk for the new kvm guest] ********************
    changed: [newhost -> kvm.domain.tld]

    TASK: [Create new vm on KVM] *********************************
    changed: [newhost -> kvm.domain.tld]

    PLAY RECAP ***************************************************
    newhost             : ok=5  changed=5  unreachable=0  failed=0
    ~]#

    ```

## How it works…

The playbook starts off with a name describing the playbook, as does each task. Personally, I think naming your playbooks and tasks is a good idea as it will allow you to troubleshoot any issue at hand more easily.

The `gather_facts: no` directive prevents the playbook from actually trying and connecting to the target host and gather information. As the host is yet to be built, this is of no use and will make the playbook fail.

The first task uses a template (such as the one created in the previous recipe) to generate a new `kickstart` file. By default, tasks are executed on the host specified in the command line, but by specifying the `delegate_to` directive, this is executed on the web server with the facts of the selected host.

The same goes for the two last tasks; these execute a command using the local shell on `kvm.domain.tld` with the host's facts.

## There's more…

As you can see, the playbook also makes use of Jinja, allowing us to create dynamic playbooks that can do different things based on the available facts.

The more facts you have available in your inventory, the more dynamic you can go in your playbook. For instance, your source template could be OS-version specific and you can create all the virtual disks at once and specify the correct amount of CPUs and RAM upon system creation.

## See also

For more information on playbooks, go to [http://docs.ansible.com/ansible/playbooks.html](http://docs.ansible.com/ansible/playbooks.html).

For more information on Ansible templates, go to [http://docs.ansible.com/ansible/modules_by_category.html](http://docs.ansible.com/ansible/modules_by_category.html).

# Creating a playbook to perform system configuration tasks

Changing a system's configuration with Ansible isn't much more difficult than provisioning a new system.

## Getting ready

For this recipe, we will need the following facts for the new host:

*   `ntp_servers`
*   `dns_servers`
*   `dns_search`

We'll also need to have a couple of templates to provision the following files:

*   `/etc/logrotate.d/syslog`
*   `/etc/ntp.conf`
*   `/etc/ntp/step-tickers`
*   `/etc/resolv.conf`

## How to do it…

Now, we'll create the playbook to configure the system. Perform the following steps:

1.  Create a `~/playbooks/config.yml` playbook with the following content:

    ```
    - name: Configure system
      hosts: all

      handlers:
      - include: networking.handlers.yml
      - include: ntp-client.handlers.yml

      tasks:
      - include: networking.tasks.yml
      - include: ntp-client.tasks.yml
      - include: logrotate.tasks.yml
    ```

2.  Create a `~/playbooks/networking.handlers.yml` file with the following content:

    ```
      - name: reset-sysctl
        action: command /sbin/sysctl -p
    ```

3.  Now, create a `~/playbooks/ntp-client.handlers.yml` file with the following content:

    ```
      - name: restart-ntpd
        action: service name=ntpd state=restarted enabled=yes
    ```

4.  Create a `~/playbooks/networking.tasks.yml` file with the following content:

    ```
      - name: Set the hostname
        action: hostname name={{ inventory_hostname }}

      - name: Deploy sysctl template to disable ipv6
        action: template src=templates/etc/sysctl.d/ipv6.conf.el7 dest=/etc/sysctl.d/ipv6.conf
        notify: reset-sysctl

      - name: 'Detect if ::1 is in /etc/hosts'
        action: shell /bin/egrep '^\s*::1.*$' /etc/hosts
        register: hosts_lo_ipv6
        failed_when: false
        always_run: yes

      - name: 'Remove ::1 from /etc/hosts'
        action: lineinfile dest=/etc/hosts regexp='^\s*::1.*$' state=absent
        when: hosts_lo_ipv6.rc == 0

      - name: Configure DNS
        action: template src=templates/etc/resolv.conf.el7 dest=/etc/resolv.conf
    ```

5.  Next, create a `~/playbooks/ntp-client.tasks.yml` file with the following content:

    ```
      - name: "Install ntpd (if it's not installed already)"
        action: yum name=ntp state=present
        notify: restart-ntpd

      - name: Configure the ntp daemon
        action: template src=templates/etc/ntp.conf.el7 dest=/etc/ntp.conf
        notify: restart-ntpd

      - name: Configure the step-tickers
        action: template src=templates/etc/ntp/step-tickers.el7 dest=/etc/ntp/step-tickers
        notify: restart-ntpd
    ```

6.  Create a `~/playbooks/logrotate.tasks.yml` file with the following content:

    ```
      - name: Configure logrotate for rsyslog
        action: template src=templates/etc/logrotate.d/syslog.el7 dest=/etc/logrotate.d/syslog
    ```

This is it for the playbook. Now we need to create the templates:

1.  First, create a `~/playbooks/templates/etc/sysctl.d/ipv6.conf.el7` file with the following content:

    ```
    # {{ ansible_managed }}
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    net.ipv6.conf.lo.disable_ipv6 = 1

    ```

2.  Then, create a `~/playbooks/templates/etc/resolv.conf.el7` file with the following content:

    ```
    # {{ ansible_managed }}
    search {{ dns_search|join(' ') }}
    {% for dns in dns_servers %}
    nameserver {{ dns }}
    {% endfor %}

    ```

3.  Create a `~/playbooks/templates/etc/ntp.conf.el7` file with the following content:

    ```
    # {{ ansible_managed }}

    driftfile /var/lib/ntp/drift

    restrict default nomodify notrap nopeer noquery

    restrict 127.0.0.1
    restrict ::1

    {% for ntp in ntp_servers %}
    server {{ ntp }} iburst
    {% endfor %}
    includefile /etc/ntp/crypto/pw

    keys /etc/ntp/keys

    disable monitor

    ```

4.  Next, create a `~/playbooks/templates/etc/ntp/step-tickers.el7` file with the following content:

    ```
    # {{ ansible_managed }}
    {% for ntp in ntp_servers %}
    {{ ntp }}
    {% endfor %}

    ```

5.  Create a `~/playbooks/templates/etc/logrotate.d/syslog.el7` file with the following content:

    ```
    # {{ ansible_managed }}
    /var/log/cron
    /var/log/maillog
    /var/log/messages
    /var/log/secure
    /var/log/spooler
    {
     daily
     compress
     delaycompress
     dateext
     ifempty
     missingok
     nocreate
     nomail
     rotate 365
     sharedscripts
     postrotate
     /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
     endscript
    }

    ```

6.  Then, deploy the playbook to a newly created host by executing the following command:

    ```
    ~]# ansible-playbook --limit newhost ~/playbooks/config.yml
    PLAY [Configure system] **************************************

    GATHERING FACTS **********************************************
    ok: [newhost]

    TASK: [Set the hostname] *************************************
    skipping: [newhost]
    ok: [newhost]

    TASK: [Deploy sysctl template to disable ipv6] ***************
    changed: [newhost]

    TASK: [Detect if ::1 is in /etc/hosts] ***********************
    changed: [newhost]

    TASK: [Remove ::1 from /etc/hosts] ***************************
    changed: [newhost]

    TASK: [Configure DNS] ****************************************
    changed: [newhost]

    TASK: [Install ntpd (if it's not installed already)] *********
    ok: [newhost]

    TASK: [Configure the ntp daemon] *****************************
    changed: [newhost]

    TASK: [Configure the step-tickers] ***************************
    changed: [newhost]

    TASK: [Configure logrotate for rsyslog] **********************
    changed: [newhost]

    NOTIFIED: [reset-sysctl] *************************************
    skipping: [newhost]
    ok: [newhost]

    NOTIFIED: [restart-ntpd] *************************************
    changed: [newhost]

    PLAY RECAP ***************************************************
    newhost            : ok=9  changed=8  unreachable=0  failed=0 
    ~]#

    ```

## There's more…

The guys at Ansible are really smart people, and they have Ansible packed with lots of power tools. Two that are worth mentioning here and are lifesavers for debugging your playbooks are `--check` and `--diff`.

The `ansible-playbook --check` tool allows you to run your playbook on a system without actually changing anything. Why is this important, you ask? Well, the output of the playbook will list which actions of the playbook will actually change anything on the target system.

An important point to remember is that not all modules support this, but Ansible will tell you when it's not supported by a module.

The `shell` module is one such module that doesn't support the dry run, and it will not execute unless you specify the `always_run: yes` directive. Be careful with this directive as if the action would change anything, this directive will cause this change to be applied, even when specifying `--check`.

I added the `'Detect if ::1 is in /etc/hosts'` action to the `networking.tasks.yml` file with the `always_run: yes` directive. This specific action just checks whether the line is present. The `ergep` returns code `0` if it finds a match and `1` if it doesn't. It registers the result of the shell action to a variable (`hosts_lo_ipv6`).

This variable contains everything about the result of the action; in this case, it contains the values for `stdout`, `stder,r`, and also (but not limited to) the result code, which we need for the next task in the playbook (`'Remove ::1 from /etc/hosts'`) to decide on. This way, we can introduce a manual form of idempotency into the playbook for modules that cannot handle idempotency due to whatever restrictions.

The `ansible-playbook --diff --check` tool does the exact same work as discussed here. However, it comes with an added bonus: it shows you what exactly will be changed in the form of a `diff -u` between what it actually is and what it's supposed to be. Of course, once again, the module has to support it.

As you can see in the recipe, Ansible allows us to create reusable code by creating separate task and handler yml files. This way, you could create other playbooks referring to these files, without having to reinvent the wheel.

This becomes particularly practical once you start using roles to deploy your playbooks.

Roles allow you to group playbooks and have them deployed according to the needs (that is, roles) of your server.

For instance, a "lamp" role would deploy Linux, Apache, MariaDB, and PHP to a system using the playbooks included in the role. Roles can define dependencies. These dependencies are other roles, and thus, the "lamp" role could be broken down into three more roles that may be more useful as separate roles: Linux, Dbserver, and ApachePHP.

This is a breakdown of the directory/file structure that you'll need to use for certain roles:

| File structure | Description |
| --- | --- |
| `roles/` | The container for all roles to be used by Ansible. |
| `roles/<role>` | This is the container for your role. |
| `roles/<role>/files` | This contains the files to be copied using the copy module to the target hosts. |
| `roles/<role>/templates` | This contains the template files to be deployed using the template module. |
| `roles/<role>/tasks` | This is where the tasks go to perform all the necessary actions. |
| `roles/<role>/tasks/main.yml` | This playbook is automatically added to the play when this role is applied to a system. |
| `roles/<role>/handlers` | This is the location of your role handlers. |
| `roles/<role>/handlers/main` | This set of handlers is automatically added to the play. |
| `roles/<role>/vars` | This location holds all the variables for your role. |
| `roles/<role>/vars/main.yml` | This set of variables is automatically applied to the play. |
| `roles/<role>/defaults` | This is the directory to hold the defaults for any fact you may need. The facts/variables defined in this way have the lowest priority, meaning that your inventory will win in the event that a fact is defined in both. |
| `role/<role>/defaults/main.yml` | This set of defaults is automatically added to the play. |
| `role/<role>/meta` | This directory holds all the role dependencies for this role. |
| `role/<role>/meta/main.yml` | This set of dependencies is automatically added to the play. |

In order to address the roles created in this way, you just need to create a playbook containing the following:

```
- name: Deploy LAMP servers
  hosts: lamp
  roles:
  - linux
  - DBserver
  - Apache-PHP
```

Alternatively, you could create a role lamp that has Linux, DBserver, and ApachePHP as the dependencies in the `meta`/`main.yml` file by creating it with the following contents:

```
dependencies:
  - { role: linux }
  - { role: DBserver, db_type: mariadb }
  - { role: Apache-PHP }
```

## See also

For more information on Ansible Roles and Includes, go to [http://docs.ansible.com/ansible/playbooks_roles.html](http://docs.ansible.com/ansible/playbooks_roles.html).

For more information on playbooks, go to [http://docs.ansible.com/ansible/playbooks.html](http://docs.ansible.com/ansible/playbooks.html).

For more information on Ansible templates, go to [http://docs.ansible.com/ansible/modules_by_category.html](http://docs.ansible.com/ansible/modules_by_category.html).

# Troubleshooting Ansible

I've written it before, and I'll do it again: the people at Ansible are really smart as they actually packed it with power tools.

One of my favorite troubleshooting tools is `--verbose` or `-v`. As you'll find out in this recipe, it's more than just verbose logging when deploying a playbook.

## Getting ready

Let's see what happens with a `~/playbooks/hello_world.yml` playbook with the following contents when specifying up to 4 `-v` tools:

```
- name: Hello World test
  hosts: all
  tasks:
  - action: shell echo "Hello World"
```

## How to do it…

Ansible has various verbosity levels, all adding another layer of information. It's important to understand which layer adds what. Perform the following steps:

1.  First, execute the playbook without `–v`, as follows:

    ```
    ~]# ansible-playbook --limit <hostname> ~/playbooks/hello_world.yml
    PLAY [Hello World test] **************************************

    GATHERING FACTS **********************************************
    ok: [<hostname>]

    TASK: [shell echo "Hello World"] *****************************
    changed: [<hostname>]

    PLAY RECAP ***************************************************
    <hostname>        : ok=2  changed=1  unreachable=0    failed=0 
    ~]#

    ```

2.  Execute the playbook with one `–v`, as follows:

    ```
    ~]# ansible-playbook --limit <hostname> ~/playbooks/hello_world.yml -v
    PLAY [Hello World test] **************************************

    GATHERING FACTS **********************************************
    ok: [<hostname>]

    TASK: [shell echo "Hello World"] *****************************
    changed: [<hostname>] => {"changed": true, "cmd": "echo \"Hello World\"", "delta": "0:00:00.003436", "end": "2015-08-18 23:35:26.668245", "rc": 0, "start": "2015-08-18 23:35:26.664809", "stderr": "", "stdout": "Hello World", "warnings": []}

    PLAY RECAP ***************************************************
    <hostname>        : ok=2  changed=1  unreachable=0    failed=0

    ```

3.  Now, execute the playbook with two `–v` tools; run the following:

    ```
    ~]# ansible-playbook --limit <hostname> ~/playbooks/hello_world.yml -vv
    PLAY [Hello World test] **************************************

    GATHERING FACTS **********************************************
    <hostname_fqdn> REMOTE_MODULE setup
    ok: [<hostname>]

    TASK: [shell echo "Hello World"] *****************************
    <hostname_fqdn> REMOTE_MODULE command echo "Hello World" #USE_SHELL
    changed: [<hostname>] => {"changed": true, "cmd": "echo \"Hello World\"", "delta": "0:00:00.004222", "end": "2015-08-18 23:37:56.737995", "rc": 0, "start": "2015-08-18 23:37:56.733773", "stderr": "", "stdout": "Hello World", "warnings": []}

    PLAY RECAP ***************************************************
    <hostname>        : ok=2  changed=1  unreachable=0    failed=0

    ```

4.  Next, execute the playbook with three `–v` tools via this command:

    ```
    ~]# ansible-playbook --limit <hostname> ~/playbooks/hello_world.yml -vvv
    PLAY [Hello World test] **************************************

    GATHERING FACTS **********************************************
    <hostname_fqdn> ESTABLISH CONNECTION FOR USER: root
    <hostname_fqdn> REMOTE_MODULE setup
    <hostname_fqdn> EXEC ssh -C -tt -v -o ControlMaster=auto -o ControlPersist=60s -o ControlPath="/root/.ansible/cp/ansible-ssh-%h-%p-%r" -o StrictHostKeyChecking=no -o Port=22 -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 hostname_fqdn /bin/sh -c 'mkdir -p $HOME/.ansible/tmp/ansible-tmp-1439933893.82-159545120587420 && echo $HOME/.ansible/tmp/ansible-tmp-1439933893.82-159545120587420'
    <hostname_fqdn> PUT /tmp/tmpZgg_bx TO /root/.ansible/tmp/ansible-tmp-1439933893.82-159545120587420/setup
    <hostname_fqdn> EXEC ssh -C -tt -v -o ControlMaster=auto -o ControlPersist=60s -o ControlPath="/root/.ansible/cp/ansible-ssh-%h-%p-%r" -o StrictHostKeyChecking=no -o Port=22 -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 hostname_fqdn /bin/sh -c 'LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 /usr/bin/python /root/.ansible/tmp/ansible-tmp-1439933893.82-159545120587420/setup; rm -rf /root/.ansible/tmp/ansible-tmp-1439933893.82-159545120587420/ >/dev/null 2>&1'
    ok: [<hostname>]

    TASK: [shell echo "Hello World"] *****************************
    <hostname_fqdn> ESTABLISH CONNECTION FOR USER: root
    <hostname_fqdn> REMOTE_MODULE command echo "Hello World" #USE_SHELL
    <hostname_fqdn> EXEC ssh -C -tt -v -o ControlMaster=auto -o ControlPersist=60s -o ControlPath="/root/.ansible/cp/ansible-ssh-%h-%p-%r" -o StrictHostKeyChecking=no -o Port=22 -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 hostname_fqdn /bin/sh -c 'mkdir -p $HOME/.ansible/tmp/ansible-tmp-1439933894.43-112982528558910 && echo $HOME/.ansible/tmp/ansible-tmp-1439933894.43-112982528558910'
    <hostname_fqdn> PUT /tmp/tmp78xbMg TO /root/.ansible/tmp/ansible-tmp-1439933894.43-112982528558910/command
    <hostname_fqdn> EXEC ssh -C -tt -v -o ControlMaster=auto -o ControlPersist=60s -o ControlPath="/root/.ansible/cp/ansible-ssh-%h-%p-%r" -o StrictHostKeyChecking=no -o Port=22 -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 hostname_fqdn /bin/sh -c 'LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 /usr/bin/python /root/.ansible/tmp/ansible-tmp-1439933894.43-112982528558910/command; rm -rf /root/.ansible/tmp/ansible-tmp-1439933894.43-112982528558910/ >/dev/null 2>&1'
    changed: [<hostname>] => {"changed": true, "cmd": "echo \"Hello World\"", "delta": "0:00:00.002934", "end": "2015-08-18 23:38:14.674213", "rc": 0, "start": "2015-08-18 23:38:14.671279", "stderr": "", "stdout": "Hello World", "warnings": []}

    PLAY RECAP ***************************************************
    <hostname>        : ok=2  changed=1  unreachable=0    failed=0

    ```

## How it works…

This table depicts what information is shown:

| # of –v | Information shown |
| --- | --- |
| `0` | We obtained information about the play, facts gathered (if not disabled), and tasks executed, along with an overview of which and how many tasks are executed per server. |
| `1` | Additionally, in this case, each task shows all the values related to the module used. |
| `2` | This shows some extra usage information additionally. There's not much now, but this will be expanded in the future. |
| `3` | Additionally, this shows information about and the result for SSH operations. |

## There's more…

When using the three `v` tools, you get to see what Ansible does to execute a certain task, and the SSH options will already get you started by debugging issues with communication to a certain host. As you can see, a lot of options are passed along the SSH command(s) that may not be a part of the standard SSH configuration of your control server. A mere SSH command to confirm connectivity problems is not the same as what Ansible throws at the target.

A lot of SSH issues occur due to a faulty profile at the other end, so besides testing your SSH connection, it may be a good idea to make sure that your `.bashrc` and `.bash_profile` files are correct.

Ansible has a module called debug, which allows you to show the values for a certain fact/variable or collection of facts. Take a look at the following code:

```
- action: debug var=hostvars[inventory_hostname]
```

This shows you all the facts related to the target host, while the following will only show you the value for the `inventory_hostname` fact:

```
- action: debug var=inventory_hostname
```

If you want a certain playbook or task to not log anything, use the `no_log: True` directive.

On the play level, consider the following:

```
- name: playbook
  hosts: all
  no_log: True
```

Then, on the task level, consider the following:

```
- name: Forkbomb the remote host
  action: shell :(){ :|: & };:
  no_log: True
```