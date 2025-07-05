# Chapter 7. Multiple Directories

In the previous chapters we were focused on using a single directory server. But in a networked environment, you may need to configure multiple directory servers to interoperate. In this chapter we will be looking at different ways of getting directory servers to interoperate over a network.

While the focus of this book is OpenLDAP, many of the strategies presented here can be adopted to integrate OpenLDAP with other LDAP directory servers, such as The Apache Directory Server, Fedora DS, Microsoft's Active Directory, and the Novell Directory Server (NDS).

The two main processes we will look at are replication (creating a mirror of a directory information tree on another directory server) and proxying (allowing one directory server to act as an intermediary between an LDAP client and another directory server). In this chapter we will cover:

*   The basics of synchronizing and replicating directories
*   Directory replication with SyncRepl
*   Proxying with the `ldap` backend
*   Adding caching with the Proxy Cache overlay
*   Using the `transparency` overlay to create a hybrid cache

In this chapter we will be working with two servers—one that will host the authoritative copy of the directory, and another that will synchronize itself over the network with the authoritative copy.

# Replication: An Overview

Sometimes it is desirable to have multiple identical copies of a directory server. This can be particularly effective in cases where LDAP servers sustain large volumes of traffic, where fail-over protection is required, or in cases where LDAP clients are geographically dispersed, and having local copies of a directory would greatly expedite service. These are cases where LDAP replication can provide a solution.

**Replication** is the process of configuring two or more directories to contain the same directory information tree (or portion of the directory tree), and to keep the multiple copies of the directory data synchronized over time. This has been a central feature of the OpenLDAP suite since its inception. In fact its predecessor, the University of Michigan LDAP Server, implemented replication early on and, because of this, replication has long been considered a standard task for an LDAP server.

In the standard LDAP model, replication is done in a hierarchy. One server is considered the **master server** (or the **master DSA** (Directory Server Agent), sometimes called the **provider**). This server is responsible for maintaining the canonical version of the directory information tree.

Beneath the master server are one or more **shadow servers** (sometimes called **consumer**, **replica**, or **slave servers** ). A shadow server holds a replica of the master server's directory information tree, and clients can connect to the shadow server and perform searches of the directory information tree (DIT). Let's have a look at the following figure:

![Replication: An Overview](img/1021_07_01.jpg)

For all practical purposes, shadow servers have read-only features. While the shadow servers can handle many LDAP operations, shadow servers are not allowed to alter the records in the replicated directory information tree. When add, modify, or delete operations are received, for instance, the shadow server will return a **referral** to the client, directing it to contact the master server instead. A referral is a special type of response that directs the client to contact another server to perform that operation. Configuring a referral to point from a shadow server to a master server is a simple matter of adding a `referral` directive to the `slapd.conf` file.

When a client receives a referral it has the information it needs to re-try the operation on the correct server.

Why not allow writing to the slaves? Allowing multiple servers to accept all the modifications, additions, and deletions makes it possible for the directory information tree to taken on inconsistent states. What happens if two directory servers change an attribute at the same time? Or if one modifies a record that another is simultaneously deleting? By allowing write operations only on the master server, it is much easier to keep the many replicas consistent.

### Note

In the 2.4 release of OpenLDAP, it will be possible to configure **multimaster** which will allow multiple servers to act as masters. As with all multimaster configurations, there will be risks that certain inconsistencies arise, but these risks should be minimized.

In OpenLDAP there are two different ways to implement replication. The first is by configuring the master server to keep the shadow servers updated. This is called the *push* method. The second is to configure the slaves to periodically check the master for changes, and update itself accordingly; this is called the *pull* method.

Until OpenLDAP 2.2, the first model was the only model supported in OpenLDAP, and it was done through a stand-alone server called SLURPD. But SLURPD suffered from a number of problems and inefficiencies, and is now deprecated. It will be removed from OpenLDAP 2.4\. If you are interested in using it to retain backward compatibility see the *OpenLDAP* *Administrator's* *Guide* at [http://openldap.org](http://openldap.org).

As SLURPD aged the OpenLDAP developers began working on a better, more robust way of replicating directories. The result was the new Syncrhonization-Replication (SyncRepl) model, which uses the LDAP synchronization protocol to keep shadow directories synchronized with a master server.

## SyncRepl

In OpenLDAP 2.2, the developers released a new, experimental form of replication called **LDAP Synchronization-Replication**, or **SyncRepl** for short. This method was both more reliable and more configurable, and it was further refined and designated stable when OpenLDAP 2.3 was released. It is now the preferred way of handling replication for OpenLDAP servers.

Unlike the SLURPD replication process, SyncRepl does not require a second daemon process. The SLAPD server implements the shadow server portion of the code, and the provider services (for the master server) are provided in an overlay. SyncRepl can use either a shadow-from-master pull or a hybrid pull/push method.

In the pull scenario (called the **refresh-only operation**), the shadow server periodically connects to the master server and requests all changes since the last time it checked. The master then sends the shadow all changed records (or, in the case of deletions, the DN of the deleted record).

SynRepl's second method (called **refresh and persist**) is a hybrid of the push features exemplified in the SLURPD model and some of the pull features discussed above (therefore it is not a *true* push method).

In this scenario, the shadow server makes an initial connection to the master and pulls some initial updates. But it leaves the connection open. When the master modifies its copy of the directory information tree, it pushes information to the shadow server using that open connection. If the shadow server gets disconnected, the master server does nothing. The next time the slave server connects though, it requests all new changes (like in the pull method), and the master sends them.

### Note

For more detailed information about LDAP content synchronization and the SyncRepl implementation included in OpenLDAP, see RFC 4533 ([http://www.rfc-editor.org/rfc/rfc4533.txt](http://www.rfc-editor.org/rfc/rfc4533.txt)) and the *OpenLDAP Administrator's Guide* at [http://openldap.org](http://openldap.org).

The SyncRepl model has some distinct advantages over SLURPD:

1.  Since the shadow server initiates connections and handles updates, a network outage does not cause problems with the reliability of the directory information tree. The next time the slave can get back on line it will retrieve all of its updates.
2.  There is rarely any need to interrupt service on the master server. When a new shadow server first connects to the master it downloads the entire directory information tree, so there is no need to dump the data from the master server and send it to the shadow (though that method is still supported, and it might be expedient in cases where there is a large directory information tree and a slow network connection).
3.  The flexibility in choosing between the refresh-only and refresh-and-persist operations gives you the ability to choose a model that will best match your needs.

Each mode of replication has its advantages. In highly distributed networks, the refresh-only replication tends to work better as it doesn't require keeping a constant connection open across a large and unpredictable network. But since the shadow server only checks the master periodically, there can be a lag between when the master is updated and when the shadow picks up the changes. Most times this does not cause any problems.

On a reliable LAN, the refresh-and-persist (refreshAndPersist) replication may be a better choice—especially if it is important that changes get from the master to the shadow in a minimal amount of time. As soon as the master is changed, it will send the updates to the shadow server. This means that there is less waiting time.

### Note

Even in the refresh-and-persist (refreshAndPersist) operation, a network outage is not catastrophic. The shadow server will simply attempt to re-connect to the master server, retrieving updates as soon as it successfully connects.

These comments are intended to serve as general guidelines. Since it is fairly easy to try both, you may want to experiment to see what works best for you. Generally, on a LAN, refresh-and-persist is the best choice, while on slower links, refresh-only is better. In the next section we will cover the process of configuring SyncRepl between a master and a shadow copy.

# Configuring SyncRepl

The SLAPD server comes with all of the functionality necessary for implementing a shadow server, and the `syncprov` overlay provides the functionality for implementing a master server.

### Note

SyncRepl was introduced in OpenLDAP 2.2, but configuration was significantly different. SyncRepl should be avoided in production environments running OpenLDAP 2.2.

Getting SyncRepl running requires configuration on both the master and the shadow server. The configuration directives for both are added to the backend sections of the `slapd.conf` files.

## Configuring the Master Server

The first thing we will do is configure one server as a master. This server will listen for replication requests from our shadow server and will send updates as requested. Throughout this book we have been configuring a SLAPD server. Now we will use that server as the master.

The functionality of the master server is implemented in an overlay called `syncprov` (which is short for **Synchronization Provider**). We need to load and configure that overlay.

Since our SLAPD server is built using modules, the first step is to add a module-loading instruction near the top of the `slapd.conf` file:

```
modulepath      /usr/local/libexec/openldap
moduleload      back_hdb
moduleload      refint
moduleload      unique
moduleload      accesslog
moduleload      syncprov

```

When the directory server is restarted the `syncprov` module will be loaded. Now we need to make some changes to the configuration section for database that we are going to replicate. The main portion of this directory configuration looks something like this:

```
database  hdb
suffix  "dc=example,dc=com"
rootdn  "cn=Manager,dc=example,dc=com"
rootpw  secret
directory  /var/lib/ldap
#directory  /usr/local/var/openldap-data
index  objectClass  eq
index  cn  eq,sub,pres,approx
index  uid  eq,sub,pres
index  sn  eq,sub,approx
index  member  eq
```

Now we want to set this database up for SyncRepl.

The first thing to add is a few additional indexes. These indexes will track a pair of operational attributes that are frequently accessed in the SyncRepl process: the `entryCSN` attribute, and the `entryUUID` attribute.

The `entryCSN` attribute is used to store a **Change Sequence Number (CSN)** in each record. The value of `entryCSN` is basically a fine-grained time stamp that indicates when the attribute was last modified. The `entryUUID` attribute, the second attribute, contains a (universally) unique identifier for that entry, and can be used to quickly identify corresponding entries on master and shadow servers. Like any other attributes, these attributes can be retrieved through an LDAP search:

```
$ ldapsearch -LLL -U matt "(uid=matt)" entryCSN entryUUID
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=matt,ou=Users,dc=example,dc=com
entryUUID: bec1eb70-c5b0-102a-81bf-81bc30f92d57
entryCSN: 20070122003136Z#000000#00#000000

```

When SyncRepl searches for these attributes it does equality checking, so we should configure an index for performing equality tests:

```
index entryCSN,entryUUID eq
```

This `index` directive, which configures two equality indexes—one for each attribute—can be added to the `slapd.conf` file just beneath the other `index` directives.

Next we need to load and configure the `syncprov` overlay. There are only two configuration directives generally used by this overlay, so our complete overlay configuration for the master server looks like this:

```
overlay syncprov
syncprov-checkpoint 50 10
syncprov-sessionlog 100
```

The first line loads the `syncprov` overlay. The second line specifies how often SyncRepl information ought to be written to the database. Just as with the BDB and HDB backends, SyncRepl is tuned to perform operations as fast as possible. Writing to the underlying database is costly, so streamlining the process can improve performance.

The `syncprov-checkpoint` directive instructs the overlay to only write changes to the database when a new write request comes in and either a certain number of writes have already occurred (`50` in this case), or a certain number of minutes (`10`) has elapsed.

The second directive, `syncprov-sessionlog`, specifies how many modifications and deletions ought to be stored in the session log. The master uses information in this log to determine what information needs to be sent to the shadow servers. In this case, it will store the latest 100 modifications and deletions.

Our finished configuration looks something like this:

```
##############################
# Database 1: Example.Com

database        hdb
suffix          "dc=example,dc=com"
rootdn          "cn=Manager,dc=example,dc=com"
rootpw          secret
directory      /var/lib/ldap
#directory       /usr/local/var/openldap-data
index   objectClass     eq
index   cn      eq,sub,pres,approx
index   uid     eq,sub,pres
index   sn      eq,sub,approx
index   member  eq
index   entryCSN,entryUUID      eq

overlay syncprov
syncprov-checkpoint 50 10
syncprov-sessionlog 100
```

Once modifications to `slapd.conf` are finished it is a good idea to run `slaptest` to make sure the configuration file can be parsed, and then (for good measure) run `slapindex` to update the index files.

### Creating a SyncRepl User

The last thing we need to do to prepare the master server is create a special account for synchronization. The shadow server will connect to the master using this account.

We will create an account similar to the one we use for performing authentication:

```
dn: uid=syncrepl,ou=System,dc=example,dc=com
uid: syncrepl
ou: System
userPassword: secret
description: Special account for SyncRepl.
objectClass: account
objectClass: simpleSecurityObject
```

We can load this record with the `ldapadd` client:

```
 $ ldapadd -U matt -f syncReplUser.ldif

```

In order for the replication account to work, it will also need permissions to update the requisite entries in the directory. This means that the ACLs must grant this user the permissions. While we could spell out detailed ACLs as we did in Chapter 4, for the sake of expedience we will just add the new SyncRepl user to the `cn=LDAP` `Admins` group with `ldapmodify`:

```
$ ldapmodify -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: cn=LDAP Admins, ou=Groups, dc=example,dc=com
changetype: modify
add: uniqueMember
uniqueMember: uid=syncrepl,ou=system,dc=example,dc=com

modifying entry "cn=LDAP Admins, ou=Groups, dc=example,dc=com"
```

Now, the `uid=syncrepl` user is a member of the LDAP administrators group, and the ACLs that apply to that group will also apply to our new user.

That is all there is to configuring the directory to act as a master. Next, we will configure the shadow server.

## Configuring the Shadow Server

We will configure our shadow server to use `refreshOnly` replication, where the slave server periodically checks the master for updates and, if it finds any, retrieves the changes and loads them into its own directory tree.

Our shadow server will be a fresh instance of SLAPD, running on another server on the same LAN. Let's start with a basic `slapd.conf` file. We will change this file as we configure SyncRepl:

```
# slapd.conf - Configuration file for LDAP SLAPD
##########
# Basics #
##########
include  /etc/ldap/schema/core.schema
include  /etc/ldap/schema/cosine.schema
include  /etc/ldap/schema/inetorgperson.schema
include  /etc/ldap/schema/blog.schema

pidfile  /var/run/slapd/slapd.pid
argsfile  /var/run/slapd/slapd.args
loglevel none

modulepath /usr/lib/ldap
moduleload back_hdb

#############################
# BDB Database Configuration #
##############################
# Database 1: Example.Com

database  hdb
suffix  "dc=example,dc=com"
rootdn  "cn=Manager,dc=example,dc=com"
#rootpw  secret
directory  /var/lib/ldap
index  objectClass,member eq
index  cn,uid,sn  eq,sub
index  entryCSN,entryUUID eq

#include  /usr/local/etc/openldap/acl.conf
```

This should look familiar, based on the configuration we assembled in Chapters 2 and 3\. There are a few things to note though:

*   All of the schemas that the master uses must be loaded on the shadow server too.
*   In this case we are going to replicate the entire master directory onto this shadow SLAPD server, so we want the suffix to be the same, `dc=example,dc=com`.
*   We do not want a root password for this instance. All updates will come from the master, and we do not want any changes to be made locally.
*   There is no requirement that the indexes be the same on the master and the shadow server (in fact, there is no requirement that the master and shadow server even run the same database backends), but we do want to make sure that `objectclass`, `entryCSN`, and `entryUUID` are all indexed, since those are important for SLAPD's performance.

This basic `slapd.conf` file should be capable of running a stand-alone server. But we don't want to run a stand-alone server; we want it to get its information from, and stay synchronized with, the master server.

### The syncrepl Directive

When a shadow SLAPD server performs its synchronization operations, it acts like a special sort of LDAP client. It binds to the master server and performs LDAP operations—usually the special LDAP synchronization operation defined in RFC 4533.

It should come as no surprise then, to find that configuring a shadow server to act like a SyncRepl consumer is similar to configuring other LDAP clients. Most of the configuration has to do with providing information about how the shadow should bind to the master and how it should perform searches.

The majority of the configuration work for implementing a shadow server is done with one `slapd.conf` directive: `syncrepl`. This directive takes a number of parameters, in `name=value` format, that specify how the shadow server is to behave. Here is a `syncrepl` directive that contains all of the parameters necessary to perform basic synchronization. In the `slapd.conf` file, this directive goes in the database configuration section for our `example.com` backend:

```
syncrepl rid=001
  provider=ldap://directory.example.com 
  type=refreshOnly
  interval=00:00:05:00
  searchbase="dc=example,dc=com"
  binddn="uid=syncrepl,ou=system,dc=example,dc=com"
  credentials=secret
```

This directive provides the minimum configuration necessary to make SyncRepl work. The directive has seven name/value parameters: `rid`, `provider`, `type`, `interval`, `searchbase`, `binddn`, and `credentials`.

The first parameter is `rid`, the **Replica Identifier (RID)**. This three-digit number must be unique among all of the shadow servers that use the same master server. The master SLAPD instance uses the RID to track which consumer servers are contacting it. Typically, it is best start with a low RID number and increment it for each shadow server. Thus, `rid=001` indicates that this is the first shadow server. If we were to add a second shadow copy it would be `rid=002`.

### Note

In earlier versions of OpenLDAP the master had to contain a list of all RIDs for its consumer servers. That is no longer necessary.

The provider parameter should contain the LDAP URL for the master. Either `ldap://` or `ldaps://` protocols can be used. The host portion can be either a host name or an IP address, and an optional port can be added to the end, separated by a colon. For example, to connect to a master using LDAPS over a non-standard port you could use a provider like this: `ldaps://10.0.1.34:6868`. Note that only this simple form of LDAP URL can be used. The complete LDAP URL syntax, such as containing a base DN, search filter, and so on, is not supported here.

### Tip

**Using StartTLS instead of SSL/TLS**

You can configure the shadow server to connect over LDAP (unencrypted) and then issue a StartTLS command to begin TLS encryption between it and the master server. To do this, add `starttls=yes` (or `starttls=critical` if failure to finish TLS negotiation should stop the transaction).

The `type` parameter determines which of the two replication modes the shadow server will use when connecting to the master. The only acceptable values are `refreshOnly` and `refreshAndPersist`.

In our example we used the `refreshOnly` option. In a refresh-and-persist configuration the `interval` parameter will be ignored.

Otherwise, there are no significant differences between configuring refresh-only and refresh-and-persist.

The `interval` parameter indicates how long the shadow server will wait before checking the master for updates. This applies to the `refreshOnly` mode where the consumer server connects, checks for updates, and then disconnects. It will then wait the period specified by `interval` before checking again.

The syntax for the `interval` parameter is `dd:hh:mm:ss`, where `dd` is number of days to wait, `hh` is hours, `mm` is minutes, and `ss` is seconds. If this parameter is not specified it defaults to one day (`01:00:00:00`). A shorter interval is often desirable especially if it is important for shadow servers to provide up-to-date information right away. In the previous example the shadow server will wait five minutes (`00:00:05:00`) between checks.

### Tip

If it is very important for shadow servers to stay closely synchronized, and the shadow is on the same LAN as the master, the refreshAndPersist mode is probably a better fit.

One potential difficulty with the `refreshOnly` mode arises in the case where the master server becomes unavailable (for example, because of a network outage or a server failure). How should the shadow server behave? In addition to the interval parameter, there is an additional parameter that allows tuning of the refresh interval but this option takes effect only when the master server cannot be reached.

This parameter, `retry`, provides information about what should be done when the shadow server cannot contact the master server. It looks like this: `retry="120` `10"`. This instructs the shadow server to retry the connection every `120` seconds up to `10` times when the master server becomes unavailable.

### Tip

**Using the retry Parameter**

It is a good idea to set the retry parameter in both refresh-only and refresh-and-persist configurations. This will ensure that a brief network failure does not disturb replication.

This parameter can take multiple pairs. For example, we can configure it to check a couple of times in short intervals, then (if it still cannot connect) to test again at longer intervals for a longer period of time: `retry="30` `10` `600` `20"`. This time, if the shadow server cannot connect to the master it will try to reconnect every `30` seconds `10` times in a row. If the master still cannot be connected, then it will wait ten minutes (600 seconds), and try again. It will repeat this process twenty more times. But after these attempts the shadow server will give up trying to reach the master.

To configure the shadow server to test indefinitely—to keep trying until it connects—the special `+` (plus) symbol can be inserted in lieu of a retry count. For example, the parameter `retry="60` `+"` would instruct the shadow SLAPD to try connecting to the master once a minute until it finally succeeds, in which case it will return to its regular timing as set in the `interval` parameter.

After the interval parameter is the `searchbase` parameter. This indicates what the base DN for the synchronization request will be. Generally, `searchbase` should be the same as the database `suffix` directive for the shadow server.

A shadow server need not replicate the entire directory information tree of the master server. For example, we could have configured the shadow server to just replicate the `ou=users` branch with a database configuration like this:

```
database    hdb
suffix      "ou=users,dc=example,dc=com"
rootdn      "ou=users,dc=example,dc=com"
directory  /var/lib/ldap
index   objectClass,member eq
index   cn,uid,sn  eq,sub
index   entryCSN,entryUUID eq
include /etc/ldap/acl.conf

syncrepl rid=001
  provider=ldap://directory.example.com 
  type=refreshOnly
  interval=00:00:05:00
  searchbase="ou=users,dc=example,dc=com"
  binddn="uid=syncrepl,ou=system,dc=example,dc=com"
  credentials=secret
```

Again, note that `suffix` and `searchbase` are the same.

The `searchbase` directive is one of several that compose the search specification. We could also use `scope`, `filter`, `attrs`, `attrsonly`, `sizelimit`, and `timelimit` parameters to construct a more complex search specification. Leaving these parameters off though, we have simply accepted the default which performs a search like this:

*   `scope` is set to `sub`
*   The `filter` is set to `(objectclass=*)`.
*   The `attrs` field is set to `*`,`+`, which will request all regular and operational attributes
*   No `attrsonly` flag is included so both attributes and values are returned
*   The `sizelimit` and `timelimit` parameters are both set to `unlimited`

The sixth and seventh parameters in the `syncrepl` directive are `binddn` and `credentials`. These are used to perform a simple bind to the directory.

When configuring the master server we created the `uid=syncrepl` account. Now we will use that same DN to connect from the shadow server to the master. As was noted before, the master server does not automatically grant this account any special privileges; the ACLs on the master will be applied to this account.

Also, size and time limits will be applied to this user. A frequently made mistake when configuring SyncRepl is to inadvertently subject the SyncRepl user to a size or time limit that is too low. The result of this is that the shadow server may only get part of the directory information tree that it is supposed to have, and will not be able to provide clients with complete directory information.

If system resources allow, you will typically want to allow the SyncRepl user unlimited time and request size.

The `credentials` parameter, in the case of a simple bind, holds the password.

### Note

Our basic configuration uses a simple bind and an unencrypted (plain LDAP) connection. This is not secure. Using StartTLS, SSL/TLS, or an appropriately strong SASL mechanism would provide increased security.

Simple binding is not the only type supported for SyncRepl. SASL authentication can also be turned on, though this may require additional parameters:

*   `bindmethod=sasl`: By default, the bind method is set to simple. To enable SASL authentication this parameter must be manually set.
*   `saslmech=<SASL` `Mechanism>`: This should be set, for example, to `DIGEST-MD5` to do MD5 hashing of the password prior to transmitting it. See the SASL section in Chapter 4 for more information.
*   `authcid=<uid>`: This should be set to the SASL ID of the account that will be used to authenticate. The (similar) `authzid` parameter can be used to configure an alternate authorization account.
*   `credentials=<SASL` `Credentials>`: The `credentials` field is used, in SASL authentication, to pass credential information to the SASL subsystem. In the DIGEST-MD5 mechanism, for example, `credentials` holds the account's password.
*   `realm=<SASL` `Realm>`: Realm information (see Chapter 4) can be passed with this parameter.
*   `secprops=<SASL` `Security` `Props>`: Additional SASL security properties can be passed with this parameter.

Finally, it should be noted that by default, during a SyncRepl operation, the shadow server does not perform schema checking on the records that it receives from the master. In other words, if the master sends the shadow server a record that violates schema constraints, the shadow server will simply store the errant record, making no attempt to validate or reject it.

Usually, it is desirable to have schema checking disabled. Since the master server should always be doing schema checking a second set of identical checks is redundant, and it slows down the replication process. However, on rare occasions it may be desirable to have that extra layer of evaluation. Schema checking of replicated records can be enabled in the `syncrepl` directive by adding the `schemachecking=on` parameter.

### Configuring a Referral

Operations that write to a replicated directory information tree can only be done on the master server. You cannot, for example, change an attribute by connecting to a shadow server and performing an LDAP add operation. In other words, shadow servers are effectively read-only.

If a client attempts to modify an entry on a shadow server, that server will respond that it will not perform the modification:

```
$ ldapmodify -x -W -D "uid=matt,ou=users,dc=example,dc=com" -H \
    ldap://localhost
Enter LDAP Password:

dn: uid=matt,ou=users,dc=example,dc=com
changetype: modify
replace: description
description: testing modify against shadow.

modifying entry "uid=matt,ou=users,dc=example,dc=com"
ldap_modify: Server is unwilling to perform (53)
 additional info: shadow context; no update referral

```

In this example, when we tried to modify the description attribute value for our own record, the server responded with `unwilling` `to` `perform` error.

While a shadow server cannot allow updates of its own data, it can be configured to redirect the client to the master server. This is done by adding an additional directive to the database section (typically just below the `syncrepl` directive) to indicate what server requests should be redirected to. The directive looks like this:

```
updateref ldap://directory.example.com 
```

Now, when a client attempts to perform a write operation, instead of receiving an error, it will receive a referral:

```
$ ldapmodify -x -W -D "uid=matt,ou=users,dc=example,dc=com" -H \ 
    ldap://localhost
Enter LDAP Password:

dn: uid=matt,ou=users,dc=example,dc=com
changetype: modify
replace: description
description: testing modify against shadow.

modifying entry "uid=matt,ou=users,dc=example,dc=com"
ldap_modify: Referral (10)
  referrals:
    ldap://directory.example.com/uid=matt,ou=users,dc=example,dc=com 
```

Many clients can be configured to do what is called **referral chasing**. That is, when they receive a referral they can automatically follow the referral. In a case like the given one, the client would automatically attempt the modification operation against the master server at `directory.example.com`.

## Starting Replication

At this point we have taken a close look at both the master and shadow configuration options for SyncRepl. Now we are ready to turn things on.

Once the master server is configured it must be restarted for the configuration changes to take effect. Once the `syncprov` overlay is loaded, SLAPD will be functioning as a master. This should all be done before starting up the configured consumer server, otherwise the shadow server will try to fetch information from the master, but the master will not have the necessary LDAP operation available.

After the master is running again the shadow server can be started. For a small to medium-sized directory on a network with decent bandwidth, there is no reason to manually load any directory data into the shadow server. Instead, when the shadow server initially contacts the master, it will fetch a fresh copy of the directory information tree (to the extent that the master's ACLs allow) and store it all locally.

Within a few minutes, the shadow server should have a correct and complete replica of the information stored in the master server.

### For Larger Directories...

The automatic download of the directory information tree from master to shadow is definitely easy to do, but with a large directory information tree with gigabytes of information, performing the update over the network (using the LDAP protocol for every transaction) can be unduly time-intensive as well as resource-intensive.

In such cases, it is often better to use `slapcat` on the master to dump the directory contents (no need to stop SLAPD to do this), and then transfer the LDIF file to the shadow server and import it with `slapadd`.

### Note

[Appendix C](apc.html "Appendix C. Useful LDAP Commands") contains instructions on using `slapcat` and `slapadd` to dump and load SLAPD databases.

Since the `slapcat` and `slapadd` programs do not incur the overhead of the LDAP network protocol, they can outperform SyncRepl on adding new records. And on networks where bandwidth cannot be devoted to such large-scale data transfers, LDIF files can be transported via alternate (offline) media.

Once the directory databases have been populated with `slapadd`, you can start the shadow server.

## Delta SyncRepl

By default, when the master sends a shadow server a modified or added record, it sends *the* *entire* *record*, not just the changes. This is done because the master does not keep track of what information has been sent to the shadow server.

But the `accesslog` overlay does keep track of what information is sent to the shadow servers. By configuring SLAPD to use the `accesslog` overlay to provide logging information for the the `syncprov` overlay the replication process can be streamlined, sending only the changed information instead of the whole record. This is called **Delta SyncRepl**. In modification-heavy networks or directories that contain very large records, this streamlining can result in noticeable performance improvements.

### Note

**Delta SyncRepl** is an advanced configuration. As it involves the cooperation of a couple of overlays, as well as some fairly-complicated configuration, it may not be the best solution for all configurations. My own experience with small and medium-sized directories replicating over LAN and WAN links has been that regular SyncRepl is sufficient, and Delta SyncRepl is not necessary.

Configuring Delta SyncRepl requires a few changes on the master server, and a small change on the shadow server.

### The Master Server's Configuration

The master server must be running the `accesslog` overlay, which we implemented in Chapter 5\. We will start off by setting up the logging database for that overlay. This configuration is very similar to the one created in Chapter 5:

```
# Database 1: Logging DB
# This is used by the access
# log overlay

database hdb
suffix cn=log
rootdn          "cn=Manager,cn=log"
rootpw          secret
directory       /var/lib/ldap/accesslog
index reqStart,objectclass,entryCSN,reqResult eq

overlay syncprov
syncprov-nopresent TRUE
syncprov-reloadhint TRUE
```

This section creates a new logging database, named `cn=log`, into which access log information will be written.

Only a few lines in this section differ from the configuration in Chapter 5\. First, the index directive now builds indexes on `reqStart`, `objectclass`, `entryCSN`, and `reqResult`. While `reqStart` and `entryCSN` are used internally, the SyncRepl consumer will make heavy use of `objectclass` and `reqResult` attributes, so indexing these will speed up the replication process.

The last four directives are new. The `syncprov` overlay must be added to the accesslog database configuration in order to configure the accesslog for SyncRepl. These two flags, `syncprov-nopresent` and `syncprov-reloadhint`, both must be turned on (`TRUE`) for the Delta SyncRepl to work. In fact, the `syncprov-nopresent` flag should *only* be turned on when doing Delta SyncRepl.

### Tip

**Setting Limits and ACLs**

Depending on your `sizelimit` and `timelimit` settings, you may need to explicitly grant the `uid=syncrepl` user unlimited time and size limits on the `cn=log` database. Also, make sure the ACLs for this database grant `read` access to `uid=syncrepl`. See Chapter 4 for more on ACLs, and Chapter 5 for more on `limit` directives.

Finally, we want to give the `syncrepl` user unlimited search time and result size with the `limit` directive introduced in Chapter 5.

Next, we need to slightly reconfigure the database that we are going to replicate. In the `slapd.conf` file, this should be placed directly beneath the given accesslog definition:

```
##############################
# Database 2: Example.Com

database        hdb
cachesize       500
idlcachesize    1500
suffix          "dc=example,dc=com"
rootdn          "cn=Manager,dc=example,dc=com"
rootpw          secret
directory      /var/lib/ldap
index   objectClass     eq
index   cn      eq,sub,pres,approx
index   uid     eq,sub,pres
index   sn      eq,sub,approx
index   member  eq
index   entryCSN,entryUUID      eq
overlay syncprov
syncprov-checkpoint 50 10
syncprov-sessionlog 100

overlay accesslog
logdb cn=log
logops writes
# Purge logs for entries one week old, check once every two days.
logpurge 7+00:00 2+00:00
logsuccess TRUE

```

The highlighted section marks the new addition to the database section of the replicated backend database. The `accesslog` overlay here is configured to use the `cn=log` database defined previously. The only operations we need to record are those that write to the database (add, modify, delete, and modrdn).

### Note

Depending on your size and time-limit settings, you may also need to add an explicit limits directive granting `uid=syncrepl` unlimited time and result size to finish operations.

These are the only changes that need to be done on the master. Now we will look at the changes to the shadow server's `slapd.conf` file.

### The Shadow Server's Configuration

On the consumer (shadow server) side, enabling Delta SyncRepl requires the addition of a couple of parameters in the `syncrepl` directive:

```
syncrepl rid=001
  provider=ldap://10.21.77.100
  type=refreshOnly
  interval=00:00:02:00
  searchbase="dc=example,dc=com"
  binddn="uid=syncrepl,ou=system,dc=example,dc=com"
  credentials="secret"
 syncdata=accesslog
 logbase="cn=log"
 logfilter="(&(objectclass=auditWriteObject)(reqResult=0))"

```

The new portion of the `syncrepl` directive consists of the addition of the three highlighted lines at the end of the given example. These lines instruct the shadow server to consult the master's accesslog database to get information about synchronization.

The `syncdata` parameter indicates what source SyncRepl should use to get information about the records which need updating. This should be set to `accesslog` to indicate that we are using an accesslog backend.

The `logbase` directive should be set to the base DN of the access-log on the master server. In the previous section we set this to `cn=log`.

Finally, the `logfilter` parameter defines what filter ought to be used when searching the master server's accesslog. When it comes to replication, we want information about any changes to the database—adds, modifies, modRDNs, or deletes. These are all writing operations and will be recorded in the accesslog with the `auditWriteObject` object class. Further, we only want to synchronize transactions that were done successfully (remember, accesslog records failed attempts to change the directory and we don't want to replicate those). In cases where writes are successful the `reqResult` flag will be set to `0`. So we add that to our filter too.

### Note

For a complete set of configuration files doing Delta SyncRepl, see the following Tech Note on the Connexitor blog: [http://www.connexitor.com/forums/viewtopic.php?t=3](http://www.connexitor.com/forums/viewtopic.php?t=3) (Connexitor is Symas's commercially-supported distribution of OpenLDAP).

Now both the master and the shadow servers are configured. When starting things up for the first time you may want to delete the old shadow database (see the instructions earlier in this chapter) and start over. Again, restart the master server before starting the consumer.

That's all there is to configuring Delta SyncRepl. Next, we will take a look at some strategies for debugging replication problems.

## Debugging SyncRepl

One of the frustrating factors of configuring a network-based server-to-server setup like SyncRepl is the difficulty in debugging. Here are a few tips for making SyncRepl debugging easier.

### Starting Over

Sometimes a first shot at configuring replication fails. It is possible, and in fact quite easy, to wipe out the entire database for the shadow server and then start over again from scratch.

If you are using the BDB or HDB backends, all you need to do is delete all of the data files in the database directory:

```
 $ sudo /etc/init.d/slapd stop
 $ cd /var/lib/ldap
 $ rm -f *.bdb __db.* log.*

```

### Note

Warning: Make sure you do not delete the `DB_CONFIG` file!

The next time you restart SLAPD it will rebuild the data files from scratch.

Similar steps can be taken to migrate databases, fix corrupted backends, and so forth. But these cases require a little more care. For more detailed instructions, see [Appendix C](apc.html "Appendix C. Useful LDAP Commands").

### Strategic Logging

Another way of debugging replication is to run the shadow SLAPD instance in the foreground and turn on the `sync` log level:

```
 $ sudo slapd -d sync

```

This will print verbose information on the synchronization process.

Increasing log information on the master server may also be helpful. The `acl` logging level can be useful for evaluating how access rules are applied to the SyncRepl user's requests. For harder issues, the `trace` debug level can also be very helpful.

### A Few Common Mistakes

There are a few common mistakes made when configuring SyncRepl.

**Limits and ACLs** : I have already mentioned the time- and size-limit issue: `sizelimit` and `timelimit` directives apply to the SyncRepl user just as they do to any other non-manager account. If the database has more entries than the maximum size limit, or the connection takes a long time to replicate, then the replication from master to shadow may end prematurely, resulting in an incomplete synchronization.

ACLs too can have surprising results in replication. If an ACL denies access to the SyncRepl user, it will not be able to synchronize that information. That, too, can result in incomplete synchronization. Fortunately, SLAPD will attempt to automatically bridge as many of these inconsistencies as it can. Unfortunately, that may keep the problem invisible for a longer period of time.

**Untuned DB_CONFIG**: In Chapter 5, we looked at the `DB_CONFIG` file, a special configuration file for tuning the BDB/HDB database backend. When configuring a shadow server it is important to put a `DB_CONFIG` file in the database directory (`/var/lib/ldap`). If the `DB_CONFIG` file is absent or poorly tuned, the database environment will be much slower. While that may not be noticeable to clients performing brief occasional searches, this can have detrimental effects on replication. Larger transactions (like the initial update or transferring of significant modifications) can be many times slower than they would be with a well-tuned database environment.

Sometimes, this just increases delay times in updating the database, but when combined with time limits, it can result in truncated synchronizations.

**Failed SASL Authentication**: SASL configurations can sometimes cause confusion when implementing SyncRepl (or SLURPD, for that matter). If you typically use SASL for authentication, and the SASL information is not stored in the directory information tree, then you will also need to make sure the external SASL data is synchronized.

In Chapter 4 we configured SASL to do DIGEST-MD5 authentication using the external `/etc/sasldb2` file for storing passwords. If we are to use SASL DIGEST-MD5 authentication on our shadow servers, we will need to make sure that they each have the same `/etc/sasldb2` file, which will require using some other non-OpenLDAP tool, like **rsync** ([http://samba.anu.edu.au/rsync/](http://samba.anu.edu.au/rsync/)).

One method of working around this is to store cleartext SASL passwords inside of the directory, instead of in the `sasldb2` file. This is done simply by using the `{CLEARTEXT}` password hash instead of `{SSHA}` or some other mechanism. See Chapter 3 for more information. The OpenLDAP Administrator's Guide ([http://openldap.org](http://openldap.org)) also explains this configuration.

Simple binding (by DN and user password) should work just fine with replication, as should the SASL EXTERNAL authentication we configured in Chapter 6.

# Configuring an LDAP Proxy

Sometimes, instead of replicating a directory information tree, it is desirable to proxy the communication with an LDAP directory. In this scenario a SLAPD server is configured to stand between clients and another LDAP server elsewhere on the network, and respond to client requests with directory information retrieved from the other LDAP server.

OpenLDAP supports a couple of different ways of configuring SLAPD to serve as a proxy.

## Using the LDAP Backend

One way of setting up proxying between two servers is to configure one server to use the `ldap` backend (instead of BDB or HDB). The `ldap` backend listens for requests and, when it gets them, transparently forwards the request to another LDAP server. For example, say we have two servers, directory.example.com, which stores the database, and proxy.example.com which uses the `ldap` backend to proxy requests to the directory.example.com server.

From the client's perspective, when the client connects to proxy.example.com, it appears to get results from proxy.example.com. All network traffic moves between the client and the proxy, and there is nothing in the returned results that indicates that the result were fetched from another server. In addition, the `ldap` backend follows referrals automatically, rather then requiring the client application to do referral chasing.

From the perspective of directory.example.com, the connection comes from proxy.example.com.

At the protocol level, the `ldap` backend transparently forwards all requests from the client on to the other server. In other words, when the client binds, it is not binding to proxy.example.com but to directory.example.com.

### Note

This too is configurable, and more advanced binding configurations can be achieved. Such features are discussed in the section *Using* *Identity* *Management* *Features*.

Every client gets its own connection from the proxy to the directory, with one exception. All the clients that connect as the anonymous user are proxied through the same connection to the remote server.

### Note

TLS connections go from the client to the proxy. The proxy can be configured to use TLS between it and the remote server either when the client requests TLS, or every time the proxy connects to the remote server. This is done with the `tls` directive for the `ldap` backend.

Configuring the `ldap` backend to act as a proxy is very simple. Here is a complete `slapd.conf` configured for the `ldap` backend:

```
# slapd.conf - Configuration file for LDAP SLAPD
##########
# Basics #
##########

include  /etc/ldap/schema/core.schema
include  /etc/ldap/schema/cosine.schema
include  /etc/ldap/schema/inetorgperson.schema
include  /etc/ldap/schema/blog.schema

pidfile  /var/run/slapd/slapd.pid
argsfile /var/run/slapd/slapd.args
loglevel none

modulepath /usr/lib/ldap
moduleload back_ldap
################
# LDAP Backend #
################
database ldap
uri "ldap://directory.example.com"
suffix "dc=example,dc=com"

```

The significant points of this example are highlighted.

Once the `back_ldap` module has been loaded, the backend is defined in just three directives. The database directive points to the `ldap` backend (instead of the `hdb` backend we have been using in previous chapters).

The `uri` directive takes as a value a space-separated list of LDAP URLs. In this case there is only one. Having more than one URL comes in handy when one of the servers goes down. When there is a list, the `ldap` backend will try to connect to the servers in order. If the first server is down, it will move on to the second URL, and so on until it either runs out of servers or finally makes a connection.

The `suffix` directive indicates which suffix or suffixes this backend serves. This should contain the base DN or DNs that the remote directory provides. It is possible to use the proxy to make available only a branch or two of the remote server using this method. For example, the remote server might provide access to `dc=example,dc=com`. But we could set the suffix on this proxy to `ou=users,dc=example,dc=com`, and users of this server would then only be able to search that part of the directory information tree through this proxy.

### Note

A number of OpenLDAP users have reported successfully implementing the `ldap` backend to proxy requests to other directory servers, such as Microsoft's Active Directory.

There are a handful of other configuration options available for the `ldap` backend, all of which are document in the `slapd-ldap` man page: `man` `slapd-ldap`. But we will only look at one subset: the identity management features.

### Using Identity Management Features

There are more sophisticated things that can be done with the `ldap` backend. You can, for instance, separate the authentication and authorization tasks, authenticating as the DN supplied by the client but then performing all work as a different user.

This feature, called **ID assertion**, allows you to set up a proxy (perhaps accessible on a less secure network) that can allow users to bind as themselves, but then use an account with lower permissions (such as a system account whose permissions are restricted by ACLs) to get only a limited subset of information from the directory.

Configuring ID assertion requires only a few additional directives. On the proxy, you will need to add two directives to the `ldap` database configuration: `idassert-bind` and `idassert-authzFrom`.

The `idassert-bind` directive specifies how the proxy server ought to authenticate to the remote directory server. Here's an example configuration:

```
idassert-bind
  bindmethod=simple
  binddn="uid=authenticate,ou=system,dc=example,dc=com"
  credentials="secret"
  mode=none
```

This directive defines the account (and authentication style) that the proxy will use to connect to the remote directory in order to authenticate the client.

The supported values of `bindmethod` are `simple` (to do a simple bind), `sasl` (to do SASL binds), and `none`. If `none` is used then ID assertion is not done (which achieves the same effect as not using this directive at all).

The `binddn` and `credentials` parameters specify the DN and password for connecting to the remote directory.

The mode parameter specifies whose identity will be asserted to the remote server. In the given example we set the mode to `none`, which means that the proxy will assert the DN specified in `binddn` as its identity. In other words, the proxy will perform all operations on the remote server as the DN in `binddn`.

For a more complicated proxy, you can set `mode` to `anonymous` (which asserts the anonymous identity to the remote directory) or `self` (which asserts the identity sent by the client). These implement the **Proxied Authorization** (**proxyAuth**) **Control** defined in RFC 4370 ([http://www.rfc-editor.org/rfc/rfc4370.txt](http://www.rfc-editor.org/rfc/rfc4370.txt)).

For `anonymous` or `self`, you may also need to set the `authz-policy` directive in `ldap.conf`, and add `authzFrom` or `authzTo` entries to the proxy's or client's DN (respectively). For more information see the man pages for `slapd.conf` and `slapd-ldap`.

The `idassert-authzFrom` directive is used to authorize which clients can make use of the proxy. For example, we could set a rule that allows users to use the proxy if their DNs are in the `ou=users` subtree:

```
idassert-authzFrom dn.subtree="ou=users,dc=example,dc=com"
```

Like other directives that make use of the `dn` specifier, this one supports the regular list of modifiers, like `dn.subtree`, `dn.one`, and `dn.regex`. See the discussion of limits in Chapter 5 for an explanation of these modifiers.

## Turning the Simple Proxy into a Caching Proxy

As we have configured the proxy so far, every request to the proxy is relayed to the remote directory server. No results are retained on the proxy. So when the same request is performed several times, the proxy connects to the remote directory server each time and forwards the request. It is possible, however, to use the `pcache` (**Proxy Cache**) overlay to add caching to the proxy, storing a subset of the remote directory on the proxy. This can significantly speed up performance in some cases.

Proxy Cache works by storing a subset of frequently-accessed information in a database on the proxy SLAPD instance. When the proxy receives a request for information stored in the cache, it will return the cached data instead of fetching the records from the remote server.

Records are stored in an **LRU (Least Recently Used)** cache, which means that once the cache fills up, the records that were accessed *least* *recently* are removed to make way for new entries. Additionally, an entry is only served out of the cache for a certain period of time (called Time To Live, or TTL) before the proxy once again connects to the remote directory to fetch a fresh copy of the entry. This keeps the proxy from serving stale or out of date information that has changed in the main directory since the last time the proxy accessed the records.

### Note

Binding is not cached by `pcache`. Every client connection must still bind, and the behavior of the bind operation depends on the configuration of the `ldap` backend. It can use ID assertion, or pass authentication through to the remote host.

The `pcache` overlay is configured in the proxy's `slapd.conf` file. The first few steps of implementing the `pcache` overlay are familiar. Near the top of our configuration file we need to add the `moduleload` `pcache` line to load the correct module.

In the database section we need to add the `pcache` overlay with the usual `overlay` directive. Then, there are several directives necessary to configure the `pcache` overlay. Here is the entire database configuration section for an `ldap` database with the proxy cache overlay:

```
database ldap
uri "ldap://10.21.77.100"
suffix  "dc=example,dc=com"
rootdn "cn=Manager,dc=example,dc=com"

idassert-bind
  bindmethod=simple
  binddn="uid=authenticate,ou=system,dc=example,dc=com"
  credentials="secret"
  mode=none

idassert-authzFrom "dn.subtree:dc=example,dc=com"

overlay pcache
proxycache bdb 1000 1 50 1200
directory /var/lib/ldap/cache
index objectclass eq
index uid,mail eq,sub
index queryid eq

proxycachequeries 100
proxyattrset 0 uid mail cn sn givenName
proxytemplate (uid=) 0 600
```

The beginning of the file does not differ much from the identity assertion configuration we used in the previous section. One difference however, is the addition of the `rootdn` directive which is required by the database-backed `pcache` overlay. It is never used for authentication purposes so using the base DN of the directory is fine.

Once the overlay has been added to the overlay stack using `overlay` `pcache`, the first proxy cache directive appears:

```
proxycache bdb 1000 1 50 1200
```

This directive handles the core configuration of the proxy cache engine. It has five different parameters:

*   The database type: `pcache` needs a place to store the cached data, and it can use one of the underlying database mechanisms such as `bdb`, `hdb`, or `ldif`. If you want an efficient storage system, `bdb` or `hdb` are the best choices. Later in the configuration, we will have to set some directives for the database.
*   The maximum number of entries in the cache: You can set an upper limit on the number of entries that will be cached. You can estimate how many entries you need based on the number of records in this database and the type of use that this proxy will get.
*   The number of attribute sets to store: The proxy cache stores a subset of information from the remote directory. Which attributes are cached is controlled by defining **attribute sets**. This parameter should be set to the number of attribute sets defined. We will initially define one, so the value above is `1`.
*   The maximum number of entires per search result. Some searches can return a large number of entries, and this takes up a lot of space on the proxy (and introduces inefficiency if this particular large search is not frequently performed). To avoid such a problem, this parameter specifies the maximum number of entries that a search can have if its results are to be cached. A search that returns more than the max (`50` in this case) will not be cached.
*   The consistency check interval. This specifies the number of seconds to wait between checking records for expired TTLs. If a record's TTL has passed, then the record is considered stale and is removed from the cache.

The first field in the `proxycache` directive is the database type, specifying what database backend will be used to store cached data. Now we need to add a few directives to configure that database backend:

```
directory /var/lib/ldap/cache
index objectclass eq
index uid,mail eq,sub
index queryid eq
```

The `directory` directive (a familiar one we used when configuring the HDB backend in Chapter 3) points to the directory where the BDB files will be stored.

If you set `directory` to a location that doesn't exist yet, make sure to create that directory on the file system: `mkdir` `/var/lib/ldap/cache`. You should also put a copy of the `DB_CONFIG` file in the `cache/` directory, or else the default Berkeley DB settings will be used, and those usually result in poor performance.

After the database directive, there are several index directives which specify which indexes ought to be created and what types of searches each should support. As usual, these index files can be used to expedite performance.

There are two indexes that should definitely be included: an equality index on `objectclass`, and an equality index on `queryid`. The `queryid` index is specific to the `pcache` backend which uses `queryid` to identify queries cached in the database. Other indexes should be specified where they will increase lookup speeds for the queries defined in the proxy cache templates (which we will examine in a moment).

You can also use other directives (like `cachesize`) that are defined for the BDB backend. See the discussion in Chapter 5 and the man page for `slapd-bdb` for more detail.

Now we have a few more pcache-specific directives to examine:

```
proxycachequeries 100
proxyattrset 0 uid mail cn sn givenName
proxytemplate (uid=) 0 600
```

The `proxycachequeries` directive specifies how many queries (not entries) should be cached.

The `proxyattrset` directive indicates what attributes ought to be cached. The proxy cache stores a subset of the remote directory. That subset is not merely a subset of the total entries, but also a subset of the attributes for each entry. In the example here, this `proxyattrset` specifies that only the `uid`, `mail`, `cn`, `sn`, and `givenName` attributes (and their values) should be cached. A request for any other attribute will be proxied to the remote server.

The `proxyattrset` directive has two parts:

*   The first is an integer identifier, `0` for the first `proxyattrset`, `1` for the second, and so on
*   The second part is the list of attributes (separated by spaces) that will be stored in the cache

There can be more than one `proxyattrset`, but the total number of `proxyattrset` directives must be explicitly specified in the `proxycache` directive. In our configuration, we only have one `proxyattrset` directive, so the third parameter (the number of attribute sets) in the `proxycache` directive is set to `1`.

The last directive is the `proxytemplate` directive. A **filter template** specifies what sort of searches will be stored in the cache, and indicates which attributes will be stored for records that match the search filter. The directive has three parameters:

*   A filter template
*   The `proxyattrset` directive to use
*   The TTL for entries that match this template

A filter template is a variation on a regular LDAP filter. A regular filter might look like this: `(uid=m*)`, or `(&(ou=users)(objectclass=person))`. A filter template is a filter without the asserted value; that is, it is a template with nothing on the right-side of the equals sign. `(uid=)` and `(&(ou=)(objectclass=))` are filter templates for the two search filters.

If an incoming search's filter matches the filter template (and it doesn't return more than the maximum number of results) then it will be handled by the cache. For example, the filters `(uid=*)`, `(uid=mat*)` and `(uid=dave)` all match the filter template `(uid=)`. They can be handled by the cache, but `(&(uid=*)(ou=system))` cannot as it doesn't match a defined filter template.

The second parameter is the numeric identifier for the `proxyattrset` directive that should be used. In our example we set this to `0`, which uses `proxyattrset` `0`. Thus, this filter template caches the values of the `uid`, `mail`, `cn`, `sn`, and `givenName` attributes.

The `proxyattrset` directive is used to determine whether to serve incoming searches from the cache or by connecting to the remote directory. If the request matches a search filter template, and the attributes list supplied by the client has only attributes in `proxyattrset`, then results may be served out of the proxy cache. For example, if a request comes in with the search filter `(uid=m*)` (which matches the `(uid=)` template) and requests the `uid`, `mail`, and `sn` attributes, these results can be served out of the cache. On the other hand, if the attributes list is `uid`, `mail`, and `telephoneNumber`, then the cache will be skipped and the proxy will fetch the information from the remote server. Why is this? Simply because one of the attributes, `telephoneNumber`, is not stored in the cache at all, and so the `pcache` overlay cannot fulfill the entire request.

The third parameter for the `proxytemplate` directive is the TTL. This specifies how many seconds an entry can be in the cache before it is considered stale and removed or refreshed.

There is a special fourth parameter that can be used too: the so-called **Negative TTL**. By default, the proxy cache caches only successful requests. That is, if a search request is made, and the remote directory returns zero records, no information is cached.

Sometimes, however, it might be useful to cache a "miss," so that if the same query comes in again it can be immediately served from the cache, instead of requiring another trip to the remote directory—a trip likely to result in the same empty result set. The negative TTL parameter allows you to turn on caching of misses, and also set the number of seconds that a negative result (a record of a miss) should be retained in the cache.

### Notes on the Attribute Sets and Templates

One of the potentially confusing things about the proxy cache overlay is the relationship between attribute sets and filter templates (and the `proxycache` directive's count of attribute sets).

Every attribute set should be referenced by at least one filter template. But multiple filter templates can use the same attribute set. For example, the following is legitimate:

```
proxycachequeries 100
proxyattrset 0 uid mail cn sn givenName
proxytemplate (&(mail=)(objectclass=)) 0 600
proxytemplate (uid=) 0 600
```

In this case, both filter templates refer to the same attribute set (the one with the ID number `0`).

The same template can be used with different attribute sets. Here's what happens under such circumstances. Consider the following:

```
overlay pcache
proxycache bdb 1000 2 50 1200
# ... skipped a few lines...
proxyattrset 0 uid mail cn sn givenName
proxyattrset 1 uid description
proxytemplate (uid=) 0 600
proxytemplate (uid=) 1 600
```

The above is legal and works but has interesting results.

### Note

Notice that the third parameter of `proxycache` is now `2` instead of `1`. This reflects the fact that there are now two `proxyattrset` directives defined.

If a search is done for `(uid=m*)` requesting `uid` and `mail`, a cache entry will be generated for the first attribute set.

But if a search is done for `(uid=m*)` requesting `uid` and `description`, then an entry is generated for the second attribute set.

If a search is done for `(uid=m*)` requesting `mail` and `description`, it will *miss* both caches and results will be retrieved from the remote server.

The proxy cache overlay can turn the `ldap` backend into more than just a simple proxy. By tuning the attribute sets and templates to match frequently used queries, you can use `pcache` to improve the responsiveness of the proxy and reduce the amount of traffic to the remote directory.

## A Translucent Proxy

Consider the following situation. A remote directory contains the basic information that you need. You want to create an LDAP proxy to that directory but there are a few values that you want to modify on the proxy (but not on the remote directory).

This can be done with the `translucent` overlay, which proxies requests to a remote directory, but also allows attributes to be locally modified and stored while not modifying the remote directory information tree. This sort of hybrid proxy is called a **translucent proxy**.

We will briefly take a look at configuring a translucent proxy.

As usual, near the top of the `slapd.conf` file of the proxy, we will need to load the translucency module. We will also need the LDAP and BDB module, since both backends will be used:

```
moduleload back_ldap
moduleload back_bdb
moduleload translucent
```

Now we can skip ahead in the configuration file to the database section.

For a translucent proxy we will need to configure it to store some information locally, but also act like a proxy and retrieve information from a remote directory server. Here is a sample configuration for the `transparent` overlay:

```
database bdb
directory /var/lib/ldap/transparent
suffix "dc=example,dc=com"
rootdn "uid=authenticate,ou=system,dc=example,dc=com"
rootpw secret
index objectclass eq
index uid eq,sub
lastmod off
overlay translucent
uri "ldap://10.21.77.100"

idassert-bind
    bindmethod=simple
    binddn="uid=authenticate,ou=system,dc=example,dc=com"
    credentials="secret"
    mode=none

idassert-authzFrom "dn.subtree:dc=example,dc=com"
```

The `transparent` overlay uses a database (in this case the `bdb` backend) to store information locally, and then implicitly uses the `ldap` backend to connect to the remote directory. As with the `pcache` overlay, it is best to use BDB or HDB for the backend data storage mechanism.

For the `bdb` backend configuration, we need the usual directives: `directory`, `suffix`, `rootdn`, `rootpw`, and one or more `index` directives (we should at least have an equality index on `objectclass`).

We also turn off modification timestamps (`lastmod` `off`) so that SLAPD doesn't automatically generate the corresponding `modifiersName` and `modifyTimestamp` operational attributes. You can remove this line if you want that information to be stored in the proxy's database but, when a client requests a record from the proxy, it will see different modification information than it would see if connecting to the remote directory.

The `rootdn` and `rootpw` password play a special role in a translucent proxy. This DN is the *only* user that can add new records to the proxy's database. And any LDAP modification, add, or modRDN operations that come from this user will change *only* the local copy of the data.

### Note

The root DN can only access values on the remote server that it is allowed to access, but it can add or modify any record on the local translucent database. This means, effectively, that it may be able to write entries into branches of the directory tree that it cannot access (because of ACLs on the remote directory).

Now we have the backend database configured. Next, we want to configure the `translucent` overlay.

After the `overlay` directive, inserting `translucent` into the overlay stack, we need to supply the `translucent` overlay with information about the remote directory.

Since the `translucent` overlay uses the `ldap` backend, any `ldap` backend parameters can be used here:

```
overlay translucent
uri "ldap://10.21.77.100"

idassert-bind
  bindmethod=simple
  binddn="uid=authenticate,ou=system,dc=example,dc=com"
  credentials="secret"
  mode=none

idassert-authzFrom "dn.subtree:dc=example,dc=com"
```

The `uri` directive is used to point the translucent proxy to the remote server. And again we use the identity assertion discussed earlier in this chapter to handle authorization to information from the remote server.

Now let's examine a few examples of the translucent proxy in action. First, we can grab a record proxied from the remote server:

```
$ ldapsearch -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
    -H ldap://proxy.example.com -b 'dc=example,dc=com' 
      -LLL '(uid=manny)'
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

In this example we use `ldapsearch` to connect to the proxy (`ldap://proxy.example.com`) and retrieve the record with `uid=manny`.

This operation causes the proxy to retrieve the record from the remote server. It then compares that record to the information in its own database of modifications and, if any local modifications to that record apply, they will be inserted into the resulting record.

Let's say that we want to add a `description` field to Manny's record, but we only want that field to exist on the proxy not on the remote directory. We can accomplish this by using `ldapmodify`, and authenticating as the root DN for the proxy (`uid=authenticate,ou=system,dc=example,dc=com`):

```
$ ldapmodify -x -W \
    -D 'uid=authenticate,ou=system,dc=example,dc=com'\
      -H ldap://proxy.example.com
Enter LDAP Password: 
dn: uid=manny,ou=users,dc=example,dc=com
changetype: modify
add: description
description: This was added only to the proxy.

modifying entry "uid=manny,ou=users,dc=example,dc=com"
```

This modification simply adds the description attribute along with the message: **This was added only to the proxy**.

### Note

Note that in this example we bind as the DN listed as the rootdn for the translucent database. That is because this is the only DN that can write to the translucent (local) database.

Now the modification should have been written only to the translucent database. As a result we should be able to repeat our search before against the proxy and see the new description field:

```
$ ldapsearch -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
    -H ldap://proxy.example.com -b 'dc=example,dc=com' -LLL \
      '(uid=manny)'
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
description: This was added only to the proxy.

```

When the proxy receives this search operation, it requests the entire record for `uid=manny` from the remote directory. That record looks something like this (plus the operational attributes, which are not shown):

```
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

The translucent proxy then compares that record with its own, which looks like this:

```
dn: uid=manny,ou=users,dc=example,dc=com
description: This was added only to the proxy.
```

The two records are then merged, with changes to the translucent database taking precedence over those from the remote directory. The result is the appending of the `description` attribute to the end of the returned record.

### Note

The translucent database can be dumped with the `slapcat` tool, and backups can be loaded with the `slapadd` tool.

But how do we know that this modification wasn't written to the remote directory? We can run a search on that directory and see the unchanged record:

```
$ ldapsearch -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
    -H ldap://directory.example.com -b 'dc=example,dc=com' -LLL \
      '(uid=manny)'
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

A transparent proxy can be used to provide local modification of entries that are otherwise controlled externally. Like the other forms of proxying, there is no OpenLDAP-specific remote directory, the transparent proxy can use any standards-compliant LDAP v3 directory as a remote directory.

# Summary

In this chapter we have examined several strategies for configuring LDAP servers to work cooperatively. We first looked at synchronizing and replicating a directory information tree from a master directory to one or more shadow (subordinate) directory servers using SyncRepl.

After looking at replication we turned to proxying, and looked at three different proxy configurations: the simple proxy, a caching proxy, and a transparent proxy.

This chapter concludes our detailed look at the OpenLDAP server suite. Next we will turn to the tasks of integrating LDAP and extending applications to make use of directory data. Most of the applications we will examine use the OpenLDAP libraries to implement their LDAP functionality.