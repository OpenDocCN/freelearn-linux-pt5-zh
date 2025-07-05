# Chapter 5. Advanced Configuration

In the last chapter, we looked at securing our OpenLDAP server with SSL/TLS, simple and SASL authentication, and ACL-based authorization rules. All of these measures were implemented through configuration files for SLAPD. In this chapter, we will look at some other advanced features of SLAPD, including:

*   Configuring multiple database backends
*   Tuning directory performance
*   Working with directory overlays
*   Adding integrity checks
*   Adding uniqueness constraints

# Multiple Database Backends

As we have worked on OpenLDAP so far we have been using only one directory tree (`dc=example,dc=com`) and one backend database (an HDB database configured in `slapd.conf`). This works well for most of the small directory servers. It is simple to configure and all of the data is stored in the same place.

But there are a number of more complex-use cases where it makes sense to have one directory server that handles multiple directory trees, where each tree is stored in its own backend database. Here are some situations in which this sort of configuration might make sense:

*   One directory server hosts the directory information trees for multiple organizations
*   One large directory server is broken up into multiple smaller trees and subtrees for performance and replication reasons
*   Two or more previously existing directory information trees are being gradually consolidated (as in the case of a corporate merger)

Of course, there are other scenarios that might require an LDAP server with multiple backends. These are just a few examples of common situations.

How does a SLAPD with multiple backends works? Let's examine a simple example. Say we have two directory information trees, the `dc=example,dc=com` tree that we have used in previous chapters, and `dc=demo,dc=net`.

We want to host both of these on the same SLAPD server. But we don't want the data for `dc=example,dc=com` to be stored in the same database files as `dc=demo,dc=net` (that could present problems later on if we ever had to split up the databases). And, of course, we don't want searches for records in one directory tree to return entries from the other.

Configuring a new database is primarily a matter of defining the new database in `slapd.conf`. After that is done we just need to create some data and load it into the new database.

## The slapd.conf File

We created the `slapd.conf` file in Chapter 2\. In previous chapters we have modified small sections of `slapd.conf`, but now we are going to step back and take a look at the overall structure of the `slapd.conf` file.

As mentioned in Chapter 2, the `slapd.conf` file can be broken into component pieces. Initially we created three sections, which we called *Basics*, *Database*, and*ACLs*. In the last chapter we looked extensively at ACLs, as well as the security directives which (for the most part) are defined in the first, *Basics* section. Let's see how the structure of our `slapd.conf` file looks:

![The slapd.conf File](img/1021_05_01.jpg)

Now it is time to refine the model a little bit. The *Basics* section contains global configuration parameters. That is, the parameters defined there are effective for the entire SLAPD server, regardless of how many database backends it has.

The *Database* section contains directives that pertain to a specific database backend, where each backend often hosts only one directory information tree. Parameters in this section define which backend (such as BDB, HDB, LDIF, SQL) is used, what the specific parameters and overlays are for that backend, which DN will be the manager for that database, and so on. There can be multiple *Database* sections in one `slapd.conf` file. In fact, configuring multiple database sections is how we accomplish hosting multiple database backends on one SLAPD server.

Finally, the *ACL* section is really a subsection of the *Database* section (though, as we saw in the last chapter, ACLs can be used at the global level, as well). Each database can have its own set of access controls. So a more accurate picture of the `slapd.conf` file would look more like this:

![The slapd.conf File](img/1021_05_02.jpg)

This figure is more representative of how the `slapd.conf` file is composed. The previous example shows two separate databases (though the number of databases is certainly not limited to two), each of which has its own directives, and its own ACLs.

While global ACLs are mentioned in the *Basic* *Configuration* section, they are not visually separated into their own section in part because their role there is not as significant as the use of ACLs in the context of a backend. Global ACLs should be used primarily to protect the root DSE, `cn=Config`, and `cn=Subschema` portions of the tree (see [Appendix C](apc.html "Appendix C. Useful LDAP Commands")), but not much more. Most ACLs should be placed in the appropriate *Database* *Configuration* section.

Now we are ready to turn to the configuration file itself and see how the previous diagram is put into practice.

A basic multiple database setup can be done easily by adding just over a dozen lines to our `slapd.conf` file. We will begin with the existing backend configuration that we created in Chapter 2 and add a new database backend beneath it:

```
##############################
# BDB Database Configuration #
##############################
# Database 1: Example.Com

database        hdb
suffix          "dc=example,dc=com" "o=My Company,c=US"
rootdn          "cn=Manager,dc=example,dc=com"
rootpw          secret
directory      /var/lib/ldap
#directory       /usr/local/var/openldap-data
index   objectClass     eq
index   cn      eq,sub,pres,approx
index   uid     eq,sub,pres

########
# ACLs #
########
include /etc/ldap/acl.conf

##############################
# Database 2:  Demo.Net

database        hdb
suffix          "dc=demo,dc=net"
rootdn          "cn=Manager,dc=demo,dc=net"
rootpw          secret
directory      /var/lib/ldap/demo.net
#directory       /usr/local/var/openldap-data/demo.net
index   objectClass     eq
index   cn      eq,sub,pres,approx
index   uid     eq,sub,pres

########
# ACLs #
########
access to attrs=userPassword
        by anonymous auth
        by self write

access to dn.sub="dc=demo,dc=net" by users read
```

We have just configured two databases:

*   The `Example.Com` directory is handled by the first database
*   The `Demo.Net` directory is handled by the second database

There are a few important things to note about this configuration:

*   Each directory has a separate manager account. This is useful when each directory is managed by a different individual or group.
*   The directory for the second database is different than that of the first. Remember that the directory is the location where the database files are stored. Each backend must have its own storage directory.
*   As we discussed earlier, each database section can (and should) have its own ACLs and a different set of ACLs can be specified for each database defined in `slapd.conf`. The ACLs in the previous example are minimal.

## Creating and Importing a Second Directory

Before we can import data, we need to create the location where the data will be stored. In the `slapd.conf` file fragment, the `directory` directive points to `/var/lib/ldap/demo.net`. However, this directory does not yet exist. We need to create it:

```
 $ sudo mkdir /var/lib/ldap/demo.net

```

### Note

If SLAPD is run as a user other than `root`, make sure to change the ownership on the `demo.net/` directory. The SLAPD user ought to own the directory. For example, if the user `ldap` runs `slapd`, do this:

```
chown ldap /var/lib/ldap/demo.net

```

Next, we need to create an LDIF file that contains the basic records for our new directory. In Chapter 3, we created an LDIF file with the main tree structures for the `dc=example,dc=com` directory information tree. Here we will create just a minimal directory structure in a file called `demo.net.ldif`:

```
# This is the root of the directory tree
dn: dc=demo,dc=net
description: Demo.Net
dc: demo
o: Demo.Net
objectClass: top
objectClass: dcObject
objectClass: organization
# Subtree for users
dn: ou=Users,dc=demo,dc=net
ou: Users
description: Demo.Net Users
objectClass: organizationalUnit

# George Berkeley
dn: uid=george,ou=Users,dc=demo,dc=net
ou: Users
uid: george
sn: Berkeley
cn: George Berkeley
givenName: George
displayName: George Berkeley
mail: george@demo.net
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

This file creates the top-level entry—a single subtree branch (for users) and a single user.

Now that we have an LDIF file, we can import it with `slapadd`. If you have not already done so, stop SLAPD while running `slapadd`. We run the following command to import:

```
 $ sudo slapadd -b 'dc=demo,dc=net' -l demo.net.ldif
```

By default, `slapadd` tries to import the data into the first directory specified in `slapd.conf`. However, in our case we want the data to be stored in the second directory. Thus, in the previous example, we used the `-b` flag to specify the base DN of the second directory. Instead of doing `-b` `'dc=demo,dc=net'`, we could have done `-n` `2`, which instructs `slapadd` to put the records in database two.

Now we have a second database with a handful of entries. We can start up the server and test it with `ldapsearch`:

```
  $ ldapsearch -LLL -x -W -D 'cn=Manager,dc=demo,dc=net' -b \ 
               'dc=demo,dc=net' '(objectclass=*)' description
```

This is what we will get:

```
Enter LDAP Password: 

dn: dc=demo,dc=net
description: Demo.Net
dn: ou=Users,dc=demo,dc=net
description: Demo.Net Users

dn: uid=george,ou=Users,dc=demo,dc=net
```

Binding to the `dc=demo,dc=net` directory tree as the manager of that directory, we can verify that the three records we added exist. Note that only the description attribute is to be returned. That is why only `dn` and `description` are displayed.

No ACLs are in place in the `demo.net` portion of `slapd.conf` that would prevent users of the `example.com` database from seeing information in the `demo.net` directory. For example, the user `uid=matt,ou=users,dc=example,dc=com` can retrieve information from the `demo.net` directory:

```
 $ ldapsearch -LLL -U matt -b 'dc=demo,dc=net' '(uid=george)' mail
```

This is the output:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=george,ou=Users,dc=demo,dc=net
mail: george@demo.net
```

If we want to prevent this behavior, we can do so with ACLs. For example, we could replace the rule that reads `access` `to` `dn`.`sub="dc=demo,dc=net"` `by` `users` `read` to a rule that restricts reading to entries within the `dc=demo,dc=net` tree:

```
access to dn.sub="dc=demo,dc=net"
       by dn.sub="dc=demo,dc=net" read
```

This would deny entries outside of the `dc=demo,dc=net` tree from accessing these records. A similar rule would have to be added to the ACLs in the `dc=example,dc=com` section to block access from users in the `dc=demo,dc=net` tree.

Now we have a directory with two different databases. In later parts of this book, we will examine other aspects of using multiple databases. For example, later in this chapter we will look at using the `glue` overlay to connect two databases together in one search. In Chapter 7 we will look at doing replication with multiple databases. But next we will look at some performance tuning options for SLAPD.

# Performance Tuning

In Chapter 2 we created a basic `slapd.conf` file. Our focus there, though, was on getting a basic server running. In the last chapter, we took a close look at the directives that had to do with security. While creating a second database backend just now, we took a higher-level look at the `slapd.conf` file.

In this part, we will continue working on `slapd.conf`, but here we will focus on parameters that help you tailor the server to the performance needs of your organization. Later in this part we will look at the `DB_CONFIG` file that the Berkeley DB backends (BDB and HDB) use. The optimizations made in that file can lead to significant performance improvements in OpenLDAP.

### Tip

**Terminology: Databases and Backends**

The distinction between databases and backends is a fine-grained one, and often the terms are used interchangeably. Here is the difference.

*   A **database** is a location (a file, a relational database, a network resource) where a directory information tree is stored.
*   A **backend** is a particular mechanism that is used to store databases (or, in some cases, to direct SLAPD to a remote database). Backends are coded as modules, which means they can be loaded dynamically at startup.

## Performance Directives

We have already created a `slapd.conf` file that SLAPD uses for managing the directory server. We will continue building on this configuration file as we look at the next batch of directives.

We will break the directives into two different classes:

*   Those that are global, and should be placed in the basic configuration section at the top of the `slapd.conf` file
*   Those that apply only to individual database backends

Of those that apply to database backends, some are available to all backend types (such as BDB, SQL, Shell, LDIF, and so on), and there are some that apply only to a specific backend type. Since we are using an HDB backend (the default), we will focus on directives that can be used by that backend.

### Global Directives

The global directives must be placed at the top part of the `slapd.conf` file, before any database sections are defined. These directives apply to the entire SLAPD server, not just a particular directory information tree within that server.

The first three directives we will see are used to optimize the interaction between the client and the LDAP server. These directives are the `timelimit`, `sizelimit`, and `idletimeout` directives. After that, we will look at the `threads` directive, which is used to tune SLAPD's threading.

### Note

Fine-grained limits on size and time can be set per-database using the `limit` directive, discussed later in this section. For example, this directive can be used to set time and size limits based per user or group.

#### Time Limits

The `timelimit` directive is used to specify the maximum amount of time SLAPD will work on a particular operation before stopping the operation and returning a message to the client.

Some operations, like searching a large directory for an attribute that is not indexed, can take a long time. Other times, clients connecting over slow network links and requesting large amounts of data can also take up significant time. Such lengthy searches can slow the entire server down, and on a busy server it can also prevent other clients from connecting and getting timely responses. And, of course, not all client applications deal well with lengthy waiting periods.

In order to avoid these problems, there is a `timelimit` directive, which gives you the ability to set the maximum amount of time the server will wait for an operation to complete before ending the operation and returning a message to the client.

The default time limit is 3600 seconds. In this example, we will lower it to only five minutes:

```
timelimit 300
```

Remember, this directive is a *global* directive, and must be placed in the configuration file before any `database` directives.

Sometimes it is useful to eliminate all time limits. This does have the disadvantage of allowing a connection to occupy resources for an unspecified amount of time and, if too many connections did this, the result would be lengthy delays (and perhaps, in extreme cases, denial of service) for clients. But, in controlled environments, this might be a risk that can be taken. To turn off time limits use the keyword `unlimited`:

```
timelimit unlimited
```

With this setting the server will not return a message to the client until the operation is complete.

These examples represent the basic use of time limits, but sometimes a more sophisticated time limit configuration is desirable. The OpenLDAP developers created a more advanced form of the `timelimit` directive to handle such complex time limit settings. In this form, the `timelimit` directive can set two different sorts of time limits:

*   **Soft limit**: The soft limit is the default time limit that the server uses if the client does not include a desired time limit in its request.
*   **Hard limit**: The hard limit is the absolute longest time that the server will spend processing a request.

Understanding this difference will help to know how the client and the server handle timing issues.

When a client connects to the directory and performs a search, it might send its own time limit request, which instructs the server to take no longer than that amount of time to do the search. For example, if a client sends a time limit of 30 seconds, it will expect the server to take no longer than 30 seconds to respond. If the server's hard time limit is higher than the time limit sent by the client, then the server will set the limit for that request to the client's requested time limit. However, if the server's hard limit is lower than the client's then it will use its own hard limit for that request.

So, if the server's hard time limit is 60 seconds, and the client requests a 30 second time limit, the server will use the 30 second limit. If, however, the server's hard time limit is 10 seconds and the client requests a 30 second limit, the server will use its hard 10-second limit, since it is lower.

### Tip

**Setting the Client Time Limit**

For OpenLDAP clients like `ldapsearch`, you can set the client time limit by editing `ldap.conf` (or your `.ldaprc` file) and adding the `TIMELIMIT` directive. In the `ldap.conf` file, `TIMELIMIT` takes only one parameter: time limit in seconds. For example, to set the time limit to 30 seconds: `TIMELIMIT` `30`.

Where does the soft limit come in? The client does not always supply a time limit and, in these cases, you may want to set a limit that is lower than the hard limit. That is, if the default hard limit is an hour, that may be a perfectly legitimate limit to set as a maximum limit, but a default of a minute or two is a better limit for those clients that don't need the longer limit.

### Note

If you set a soft limit higher than the hard limit, the hard limit will be used.

Now we can look at the expanded form of the `timelimit` directive to see an example of setting the hard and soft limits. Typically, both are set in the same command (though you can set one without setting the other):

```
timelimit time.soft=30 time.hard=300
```

In this example the soft time limit is 30 seconds, while the hard time limit is 300 seconds. This allows clients that request longer limits to get longer processing time, while setting a lower default for clients that do not provide time limits when making requests.

What does the client get if the time limit is reached? The server will return as much of the processing as it could complete, but it will also include a warning that the time limit was exceeded.

Note that on a busy server a request may get queued, but not actually be executed until a thread becomes available to do the processing. In such cases, the time that the request waits for a thread is not counted against the time limit. The timer for the time limit begins when the worker thread begins processing the request, not when the server receives the request.

### Note

The backend-specific limits directive discussed later in this chapter provides fine-grained time and size-limit support. For example, you can set time limits on particular users or group members.

#### Idle Timeouts

Along with limiting the amount of time SLAPD spends processing a request you can also limit how long SLAPD should allow a client to remain connected, but idle. A connection is idle if it is connected to SLAPD but is not performing any operations. For example, a client may connect to SLAPD, perform a bind, and then keep the connection open, perhaps waiting for input from a user.

In many cases, there is no harm in allowing clients to remain connected, but idle. Idle clients do not require attention by one of the server's threads, so they do not use up valuable resources. Because of this, the default behavior of the server is to simply allow idle connections to remain connected indefinitely.

But on occasion (sometimes because of limitations in another part of the system), it is desirable to prevent clients from connecting and remaining idle. Use the `idletimeout` directive to set a timeout. Like the simple form of `timelimit`, `idletimeout` takes just one argument, the number of seconds a connection can be idle before SLAPD closes the connection:

```
idletimeout 3600
```

#### Size Limits

Along with setting limits on the amount of time that an operation can take, it is also possible to set limits on the number of records a search operation can return. Clients can easily perform broad searches that will return many records. Without a size limit in place a search with the filter `(objectclass=*)` would, if not restricted by ACLs, return every record in the search base. And if such a search was performed on a database that held millions of records, SLAPD would send all of those records back to the client.

In most cases it makes sense to set an upper limit on the number of records that can be returned in any one search. By default, SLAPD will only return the first 500 records. But that number can be changed with the `sizelimit` directive.

In its simple form the `sizelimit` directive takes only one parameter, the maximum number of records to return:

```
sizelimit 1000
```

As with `timelimit` though, there is an expanded form of the `sizelimit` directive, and like `timelimit`, `sizelimit` has both soft and hard limits. The expanded `sizelimit` directive also has a third property that can be set, and this property is called `unchecked`.

Hard and soft limits function similarly in `sizelimit` as they do in `timelimit`. The hard limit determines the maximum number of search results that will be returned in any search. Just as is the case with time limits, clients can also send information telling the server the maximum number of entries the client wants back. If no such information is set the value of the soft limit will be used.

If the server finds more records than the `sizelimit` allows, it will return the maximum number of records as well as an error message: `Size limit exceeded`.

The `unchecked` condition is a little bit more complex. In cases where a search requests matches for an attribute that is not indexed, SLAPD may find a large number of records that it needs to test to see if they match the client's filter. Sometimes the number of candidate records is quite large. The `unchecked` property can be used to set a limit on the maximum number of records that can be selected as candidates for matching. This can prevent poorly-tuned databases from consuming lots of time and resources searching through huge potential records for those that match.

### Note

Indexing attributes that are commonly searched is the best way to avoid this situation. Indexing is discussed later in this chapter.

If a client's request produces more candidates than allowed by the unchecked property, the server will return an error (`Administrative limit exceeded`) and will not do the search at all.

The `unchecked` property will keep the server from spending too much time on such tasks, but at the expense of the client's ability to run queries against the database. Perfectly legitimate searches can be blocked this way. For that reason, the `unchecked` property should be used with care. The default is to not limit the number of candidates. This is equivalent to specifying `size.unchecked=unlimited`.

Here's an example of setting all three in one directive:

```
sizelimit size.soft=500 size.hard=1000 size.unchecked=2000
```

In this example, the soft size limit is set to 500, while the hard limit is set to 1000, and the maximum number of unchecked records to be analyzed is 2000\. Note that the unchecked size limit should, as a matter of practice, be set to a value larger than the hard limit.

#### Threads

The last few directives have dealt with setting limits on the server's performing requested operations. These can prove valuable ways of preventing resources from being wasted or misused. Now however, I want to turn to a directive that governs the server's ability to handle requests.

SLAPD is a multi-threaded application. Unlike other servers, SLAPD does not start subprocesses to handle searching. Instead, the SLAPD server is a single process that has many different threads executing concurrently within that processes.

Each thread can perform its own task. So, if a server has sixteen threads (the default for OpenLDAP's SLAPD server), then it can perform sixteen different tasks at the same time. Roughly speaking, threads perform operations. A single client can make a single connection, and then request several different operations, each of which may be done by a different thread (although no more than half of the total threads will be dedicated to a single client).

Sixteen threads, the default, is excessive. Recent performance testing has shown that running a busy server at eight threads performs better than running sixteen, even at high loads. Why? The answer, in a nutshell, is that more threads introduce more competition for the same resources. SLAPD is efficient enough that delegating work to a smaller thread pool is typically faster than using a large thread pool, and incurring thread scheduling overhead.

Lowering the thread count has additional benefits. It is estimated that each thread costs at least 13MB (and perhaps quite a bit more, depending on the configuration of SLAPD and the hardware on the machine). Enterprise LDAP directories can certainly handle this sort of overhead, but on a host that runs LDAP along with many other services, reducing the number of threads might boost the server's performance in other areas, and still perform at the same speeds (or better) as it would when running sixteen threads.

### Note

In future versions of OpenLDAP, the default thread count will very likely be reduced from sixteen to eight.

The `threads` directive is used to set the maximum number of threads that SLAPD will create. It takes an integer:

```
threads 8
```

In typical OpenLDAP configurations, this setting is optimal, though small servers with little traffic may benefit by dropping the thread pool to as low as four.

### Tip

**Proxies and Threads**

If you are running busy SLAPD proxy server (with a `proxy` or `ldap` backend, covered in Chapter 7) that queries remote directory servers, you may benefit by having much larger thread pools. Since the worker thread is occupied until the remote LDAP server responds, a thread can remain occupied for long periods of time. In order to keep clients from being denied service you may want to add threads.

Note that the lowest number of threads allowed is 2\. This is the minimum number of threads OpenLDAP needs to provide basic service.

### Directives in the Database Section

Some directives go in the database section instead of the main portion of the configuration file. And of these, some database directives are specific to the particular backend being used. Along with the backend-neutral directives, we will see a few directives that can be used in BDB/HDB backends.

#### Limits

We looked at the `sizelimit` and `timelimit` directives, both of which are used in the global section. But in the Database section, there is another directive used for setting limits, and this directive provides finer-grained control over who is limited. You can, for example, set lower or higher limits for individual DNs, subtrees, or for members of a group. The directive for doing these things is the `limit` directive.

A `limit` directive is similar to an ACL. It has three parts: the directive itself, the who-phrase, and one or more limit-phrases. Here's an example:

```
limits users size=20
```

This directive sets a limit for all authenticated users (using the `users` keyword). Only twenty records will be returned before SLAPD will return the message: `Size limit exceeded`.

The `limit` directive supports two limit-phrases: `size` and `time`. As with the `sizelimit` directive discussed above, `size` can use the `soft`, `hard`, and `unchecked` styles. Similarly, `time` can use the `soft` and `hard` styles. Since more than one limit-phrase can be used, we can create a more robust set of limits. Here's an example limiting the anonymous user to only short result sets, and only if the operation can be done quickly:

```
limits anonymous 
  size.soft=5 size.hard=15 size.unchecked=100 
  time.soft=5 time.hard=30
```

This sets all three size limits, as well as both time limits, for the anonymous user. This would keep the anonymous user from running lengthy searches.

As we have seen, the `anonymous` and `users` keywords can be used in the who-phrase. But just as in ACLs, the `dn` specifier, along with its modifiers (`exact`, `base`, `onelevel`, `subtree`, `children`, and `regex`) can also be used.

### Note

The `dn` field and its modifiers were covered in detail in the section on *Access Control Lists* in the previous chapter.

Using the `dn` field we can create limits for particular DNs, DN patterns, or subtrees. For example, we can set a size limit for a particular user:

```
limits dn="uid=matt,ou=Users,dc=example,dc=com" size=50
```

This will set the size limit to 50 for this particular user only. If this is the only limits statement, then SLAPD will apply the size limit set in `sizelimit` to all other DNs.

Similarly, we can set a size limit for all DNs in a subtree with a directive like this:

```
limits dn.sub="ou=Users,dc=example,dc=com" size=50
```

The above limit will apply to `uid=matt,ou=Users,dc=example,dc=com` as well as all other users in that same branch of the directory information tree.

Finally, limits can also be set by group. In this case the limits will apply to any member of the group. As with ACLs, the limits directive's who-phrase uses the `group` field to indicate that SLAPD should base restrictions on group membership:

```
limits group="cn=Admins,ou=Groups,dc=example,dc=com" size=unlimited
```

This directive sets the limit for members of the `Admins` group to `unlimited`, which means that no limiting will be enforced on these group members.

Just as with ACLs, only records with the object class `groupOfNames` are automatically considered to be groups. But other object classes function as groups, as well. For example, in Chapter 3 we created a group with the object class `groupOfUniqueNames`. That group's DN was `cn=LDAP` `Admins`, `ou=Groups,dc=example,dc=com`.

In order to use that record as a group we need to specify more information in the `limits` clause:

```
limits group/groupOfUniqueNames/uniqueMember="cn=LDAPAdmins,ou=Groups,dc=example,dc=com" size=unlimited
```

When putting a directive, such as the given one into a `slapd.conf` file note that the entire group field (from `group` to the end of the DN) must be on one line.

This `limits` directive will allow search results of unlimited size for members of the group `cn=LDAP` `Admins`, `ou=Groups,dc=example,dc=com`. The group type explicitly indicates the object class of the record (`groupOfUniqueNames`) and the field that is to be treated as the membership field for that group (`uniqueMember`). Thus, when

SLAPD checks limits, it will look at the `LDAP` `Admins` record, check to see if it has the `groupOfUniqueNames` object class, and then evaluate whether the user who connected is listed in one of the `uniqueMember` values in the record. If so, then that user's size limit will be set to `unlimited`.

#### Read-only and Restrict Directives

One way to improve performance on a busy server is to limit what clients can do on the server. For example, if the information in a directory is static (that is, no users ought to be able to change data), then it may be best to put the directory server into a read-only mode. Or perhaps limiting just specific operations (such as adding new records or deleting records) is sufficient.

There are two directives that can be placed in the `slapd.conf` file for achieving these results: `readonly` and `restrict`.

The `readonly` directive is simple. It is either `on` or `off`. By default it is `off`, so the directory allows writing operations (add, modify, delete, and so on). Here's how it is used to configure SLAPD as a read-only directory server:

```
readonly on
```

When this directive is set, a client that attempts to modify information in the directory information tree will get an error message from the server:

```
  ldap_modify: Server is unwilling to perform (53)
  additional info: operation restricted
```

### Note

Not even the manager can perform modifications to the directory when `readonly` is turned on.

Binding, searching, and other operations that do not involve changing directory information can continue to function as normal though.

### Note

Extended operations, such as the **Password Modify extended operation**, are not affected by the `readonly` directive. For that reason the `ldappasswd` client, for example, will still change a password in the directory even if `readonly` is turned on.

To prevent this, use the `restrict` operation to restrict one or all extensions. The Password Modify extended operation is defined in RFC 3062 ([http://www.ietf.org/rfc/rfc3062.txt](http://www.ietf.org/rfc/rfc3062.txt)).

Sometimes setting the server to read-only mode is too stringent. It may be desirable to just prevent certain operations. This can be accomplished with the `restrict` directive.

The `restrict` directive takes a list of one or more LDAP operations that should be disallowed. These are the operations that `restrict` understands:

*   `add`
*   `bind`
*   `compare`
*   `delete`
*   `modify`
*   `rename`
*   `read` (a special pseudonym that prevents all reading operations like search, compare, and bind)
*   `search`
*   `write` (a special pseudonym that prevents all writing operations and is equivalent to setting `readonly` `on`)

In addition to these nine, there is one special type for handling extension: `extended=<OID>`. In the extended type, `<OID>` should be replaced with the Object Identifier (OID) for the extended operation that you want to restrict.

For example, we can prevent users from adding, renaming, and deleting entire entries with the following directive:

```
restrict add delete rename
```

This will prevent a user from adding new entries, renaming existing entries (that is, changing the DN), or deleting entries. With the above configuration in the database section of `slapd.conf`, we cannot add or remove entries with the command-line tools:

```
  $ ldapadd -U matt -f john_locke.ldif 
```

Here is what we get:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
adding new entry "cn=John Locke, ou=users,dc=example,dc=com"
ldap_add: Server is unwilling to perform (53)
          additional info: operation restricted

$ ldapdelete -U matt "uid=manny,ou=users,dc=example,dc=com"
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
ldap_delete: Server is unwilling to perform (53)
             additional info: operation restricted
```

Notice that in both cases the server responded: `Server unwilling to perform`. However, modifying an attribute in the record is still allowed, as are searching, comparing, and binding.

As we noted before, extended operations can be restricted using the `extended` type with the `restrict` directive. Unlike the other types though, `extended` takes a value—we can specify which extended operation we want to restrict. Unfortunately, the value must be in the unfriendly OID format. To find out the correct OID you can either check your server's Root DSE entry (see [Appendix C](apc.html "Appendix C. Useful LDAP Commands")), or you can read the RFC for the desired extended operation.

Once you have the OID number it is easy to set a restriction. For example, to prevent clients from performing the *Password* *Modify* extended operation use the following:

```
restrict extended=1.3.6.1.4.1.4203.1.11.1
```

Attempting to use the `ldappasswd` client to modify a password will result in an error:

```
  $ ldappasswd -x -W -D 'cn=Manager,dc=example,dc=com' -S 
       'uid=barbara,ou=users,dc=example,dc=com'
```

Here is the error:

```
New password: 
Re-enter new password: 
Enter LDAP Password: 
Result: Server is unwilling to perform (53)
Additional info: extended operation restricted
```

The `restrict` directive provides a convenient way of limiting what operations clients can perform.

#### Index (BDB/HDB Backends Only)

If you are running a SLAPD server with the BDB or HDB backends (the most commonly-used backends), then the `index` directive is the single most important performance-related directive.

The `index` directive, which is specified in the database section for each BDB or HDB database, indicates which fields SLAPD should build and maintain an index for. An index is a separate database file that is optimized for speedy access during LDAP read operations.

When a client uses a search filter with an attribute that is not indexed, SLAPD searches through every record in the directory for the desired attribute, then checks the value of that attribute against the attribute value or filter supplied by the client.

If the attribute is indexed on the other hand, the SLAPD server simply searches the appropriate attribute index for the value, and quickly returns a list of matching records.

An index search is much faster than a full directory search and, the larger the directory, the more noticeable the difference.

The task of determining which attributes must be indexed is left up to you, and the attributes that you ought to index should be determined by which object classes are used in your directory information tree, and which reading operations (searches, binds, compares) are run against your directory server. Directories primarily oriented around information about people (using `person`, `organizationalPerson`, and `inetOrgPerson` object classes) should probably have indexes for commonly used attributes such as `cn`, `sn`, and `uid`.

When we created our basic `slapd.conf` file in Chapter 2, we configured the following indexes:

```
index  objectClass  eq
index  cn  eq,sub,pres,approx
index  uid  eq,sub,pres
```

There are three indexes specified above: one for `objectClass`, one for `cn`, and one for `uid`.

The first line creates an index for the `objectClass` attribute. The index is optimized for equality (`eq`) matches (that is, searches like `objectclass=person`, but not searches such as `objectclass=*son`). This index should always be included, as the vast majority of reading operations will use the `objectClass` attribute.

The second line is the index for the `cn` attribute. Along with configuring this index to efficiently handle equality (`eq`) matches, it is also configured to efficiently perform substring (`sub`) and approximation (`approx`) matches, as well as doing quick tests to see if the attribute is present (`pres`). Here is a brief explanation of each of the index optimization types:

*   `approx`: This optimizes searches for approximation matching. If a search operation is made request approximate matches (`cn~=mat`), this index may be used to speed up the approximation matching.
*   `eq`: This optimizes for equality matching. Filters that request an exact match, such as (`uid=matt`) or (`objectclass=person`) make use of the `eq` optimization. It is very important that the `objectclass` attribute have an index optimized for equality. When using directory replication or other overlays, you may also need to index other frequently used attributes.
*   `sub`: This optimizes substring matching. A substring search occurs when a search request sends a part of a string and asks that attribute value that contains that part be returned. For example, the filter (`uid=*ar*`) should match any UIDs that contain the string `ar`. Users `mark` and `karen` would both match this filter.
*   `subinitial`: This is a special type of `sub` optimization that optimizes matching the first part of the string only. It is good for handling filters like (`uid=mar*`), but not filters like (`uid=*ark`).
*   `subfinal`: This is also a special type of `sub` optimization. This one optimizes matching the last part of the string, and performs well for filters like (`uid=*ark`).
*   `pres`: The `pres` type optimizes the index for cases where a search merely needs to see if an attribute is present.

Not all attributes though, support all of the index options. For example, the `objectclass` attribute does not support `approx`, `sub`, or any of the `sub` variants, and does not benefit from having a `pres` index.

### Tip

**Indexes and Schemas**

An object class' schema defines what matching rules an attribute supports, and the type of matching rule determines whether or not it can support a particular type of index. See Chapter 6.

In general, adding indexes for commonly-used attributes is a good thing. It speeds up searching and other reading operations, and since the majority of LDAP operations are reading operations, this can be a boon to performance.

But maintaining indexes does slow down writing operations that involve indexed attributes, since those attributes have to be maintained not only in the main database, but also in the index database files. Also, each index requires additional cache space for efficiently searching, which means adding more indexes will consume more memory. For those reasons, it is best to index only the attributes that are frequently used in searching operations, rather than indexing everything.

When an `index` directive is added or modified though, SLAPD does not automatically re-index all of the entries in the directory. You will need to do it by hand. For example, after looking at common searches on our system, we determine that it would be good to add indexes for the `sn` and `member` attributes. Other applications often run searches to find out what groups particular DNs are members of, and an index of this attribute would expedite those searches.

To service these needs we will add the following new `index` directives:

```
index sn eq,sub,approx
index member eq
```

But once we have added these to `slapd.conf`, we will need to stop SLAPD and run the `slapindex` program to rebuild the index files:

```
 $ sudo slapindex -q
```

This will rebuild all of the indexes. The `-q` (quick) flag will greatly speed up the process, as it skips performing consistency checks of the database.

### Tip

**Avoid Rebuilding Indexes**

A `slapindex` procedure will rebuild all indexes. When adding an index to a large directory, you may want to avoid rebuilding all of the other indexes. One way to do this is to comment out the existing indexes in `slapd.conf` (leaving only the new index lines uncommented), run `slapdindex`, and then remove the comments from the existing indexes. The next version of OpenLDAP will support a more convenient way of adding indexes.

The `slapindex` program will print error messages if any of the optimizing types are not allowed for an attribute (like if one tries to add substring indexing to `objectclass`). But when it runs successfully, it will quietly exit without printing any messages.

Once `slapindex` is finished, SLAPD can be restarted.

#### Controlling the Cache (BDB/HDB Only)

With BDB and HDB backends, SLAPD stores frequently-accessed records in a cache so that it doesn't have to read directory information from disk with every request. By default, SLAPD retains one-thousand records in the cache. But busy directory servers with a few thousand entries or more will benefit from having a larger cache. This can be done with the `cachesize` directive:

```
cachesize 2000
```

The above directive doubles the default cache size, instructing SLAPD to keep 2000 records in memory.

What happens when the cache is full? By default, SLAPD simply drops the last item in the cache (leaving a cache of 2000 with 1999 full slots). On a busy server, emptying just one cache entry at a time can have slight negative impacts on performance, since it is possible that if a number of searches are executed in rapid succession, each one missing a cache hit, the last entry of the cache would be freed up and filled with every request. This scenario is more likely to happen with the cache is disproportionately small, when compared to the number of entries in the database.

The `cachefree` directive can be used to instruct SLAPD to drop more than one item from the cache when it fills:

```
cachefree 5
```

This example instructs SLAPD to drop the last five entries in the cache.

Ideally, the cache size should remain as close to the actual number of entries in the database as memory constraints will permit. At least, though, the cache should be large enough that frequently requested records can remain in memory. For example, if your directory server functions as an address book, then the cache should be large enough that the user records, as well as their ancestor records, can all be kept in cache at the same time.

### Note

These caching directives are not the only ones of importance for SLAPD. See also the `set_cachesize` directive in the `DB_CONFIG` file section.

The third cache directive is `idlcachesize`. The `idlcachesize` directive is used for caching the results of frequently performed searches, and a large cache here will make searches of often used searches much faster. With the HDB database, it is suggested that this be *three* *times* the `cachesize` value:

```
cachesize 2000
idlcachesize 6000
```

We have now finished taking a look at the `slapd.conf` configuration options. Now we will turn to another configuration file that can be used to tailor the performance of SLAPD.

#### Reducing Disk I/O Latency (BDB/HDB Only)

When LDAP operations write new data to the directory and SLAPD is using the BDB or HDB backends the data is stored in memory first, and then flushed to the database files stored in the operating system.

On a very busy directory server (or a server with really slow disk I/O), it is sometimes desirable to trade off data security for speed. There are two directives in particular that instruct SLAPD to make this trade off:

*   The first, and the less risky of the two, is the `dirtyread` directive which takes no parameters.

    Consider the case where one client performs a write operation to modify a record, and then another client performs a read operation on that same record before SLAPD has written the first client's changes to disk. Should the server return the unmodified data stored on disk, or the modified data that has not yet been committed? Usually it does the first, sending the *clean*, but soon-to-be out-of-date record to the client.

    The term "dirty read" describes the second case, where the server sends the client information that has not been committed. While returning this data may be faster, it might possibly be inaccurate; the server may reject or abort the modification request of the first client even after having sent the modified data to the second client.

    The `dirtyread` directive only increases the risks that a client may get inaccurate data.

*   The second directive, `dbnosync`, carries a higher risk.

    Normally, when an operation changes directory information, the changes are written to disk as soon as possible. Data stored in memory is flushed to the files in the Berkeley DB subsystem. But performing disk I/O can slow the server. One way to speed this up is to instruct SLAPD to delay writing the information to the log file on the disk, and this is done with the `dbnosync` directive.

    The risk in running with `dbnosync` though, is that in the event that the server should die without a clean shutdown, modifications made to the directory, but not yet written to disk, will be lost. However, there is no greater risk of corrupting the database—the database will still be recoverable, though the most recent changes may be lost.

    You can reduce (though not eliminate) the risks of running with `dbnosync` by also using the `checkpoint` directive. Setting a checkpoint causes SLAPD to periodically write data to the disk. The `checkpoint` directive takes two parameters: a maximum size (in kilobytes), and a time limit (in minutes). SLAPD will checkpoint the database anytime the amount of data written is greater than the maximum size or after the specified interval of time has passed. Here is an example of the `checkpoint` directive:

    ```
    	  checkpoint 1024 30
    ```

    This instructs SLAPD to checkpoint the database (flushing any new data from memory to the file system) whenever more than one megabyte of data has been written to the database and every 30 minutes.

Due to the increased risks with these directives, it is generally better to try other means of improving performance (such as altering the cache or tuning the `DB_CONFIG` file) before implementing these directives.

## The DB_CONFIG File

The `DB_CONFIG` file is technically not an OpenLDAP configuration file at all. It is a Berkeley DB configuration file, and is specific to the BDB and HDB backends only. It provides a series of settings for the Berkeley database engine.

### Note

Berkeley DB is an Open Source embedded database, now maintained by Oracle. Because it is robust and reliable, actively maintained, and widely support, it is a popular product in both open source and proprietary applications. For more information about Berkeley DB, see Oracle's website: [http://www.oracle.com/database/berkeley-db/index.html](http://www.oracle.com/database/berkeley-db/index.html)

Since the entire directory information tree, as well as the indexes, for a BDB/HDB backend is stored in a Berkeley DB database, a properly configured `DB_CONFIG` file is the most important facet of directory performance.

When experimenting with the `DB_CONFIG` file and trying out new configurations, it is best to use a non-production server, and to use `slapcat` to make a full backup of the directory data before you make any changes.

The `DB_CONFIG` file is not stored with the OpenLDAP configuration files. Instead, it is stored alongside the database files at `/var/lib/ldap` (or `/usr/local/var/openldap-data`). Unlike the other configuration files, it is read only when the database is created or recovered. As of OpenLDAP 2.3, if SLAPD detects changes in `DB_CONFIG` when it is starting up, it will attempt to perform a database recovery in order to incorporate the changes, and you may see an entry like this in your log file:

```
bdb_db_open: DB_CONFIG for suffix dc=example,dc=com has changed
Performing database recovery to activate new settings
```

Likewise, when you create a new directory, the Berkeley DB subsystem will read the `DB_CONFIG` file and create the databases according to the directives therein.

### Tip

Make sure your database has a `DB_CONFIG` file. If your database directory does not have a `DB_CONFIG` file present, you will be using the factory defaults for Berkeley DB, which are very conservative. On anything but a small (<100 entries) directory server, the defaults will be insufficient, and result in poor performance.

OpenLDAP distributions include a default `DB_CONFIG` file tuned for general use. It should be located at `/var/lib/ldap` already (though it is sometimes labeled `DB_CONFIG.example`, in which case you will need to rename it to just `DB_CONFIG`). In Ubuntu Linux, an Ubuntu-customized `DB_CONFIG` file is located at `/usr/share/doc/slapd/examples/DB_CONFIG`. We will start by using the version included with the OpenLDAP source distribution (which is configured for enterprise use). The default version looks something like this:

```
# one 0.25 GB cache
set_cachesize 0 268435456 1
# Data Directory
#set_data_dir db

# Transaction Log settings
set_lg_regionmax 262144
set_lg_bsize 2097152
#set_lg_dir logs
```

We have removed some of the comments from the header and footer of the file, but preserved all of the settings.

For standard usage on a medium-sized directory, these settings are good. If your directory is performing sufficiently fast and your system is not strapped for resources, you need not feel compelled to change the default settings.

The `DB_CONFIG` file contains directives that directly pertain to the performance of the underlying Berkeley DB files. We will go through these six settings in order. The most important directive is the first.

At the end of this section we will also look at three additional directives used for tuning Berkeley DB lock handling.

### Note

Some of the directives we examined earlier are synonyms for `DB_CONFIG` directives. For example, `dbnosync` does the same thing as the `DB_CONFIG` directive `set_flags` `DB_TXN_NOSYNC`.

### Setting the Cache Size

The BDB/HDB backend attempts to keep as much of the directory as possible in memory in the form of a cache. This keeps directory reading quick since SLAPD does not have to read information from the disk.

While it might not be possible (on a system with other services, a good economic trade-off) to keep the entire directory in the cache, the server will run faster if at least the most frequently used entries are kept in the cache.

The `set_cachesize` directive determines how much memory SLAPD will allocate for a directory cache. The directive takes three arguments:

*   The number of gigabytes of space to allocate for the cache
*   The number of bytes of space to allocate for the cache
*   The number of segments to use for the the cache

The first and second are added together and should not, when combined, be larger than 4 GB. The third determines how many data segments the Berkeley DB backend will break the cache into. The values 1 and 0 both result in a single cache segment (which is usually desired).

In the default OpenLDAP `DB_CONFIG` file, the `set_cachesize` directive looks like this:

```
set_cachesize 0 268435456 1
```

The total size of the cache is 256 megabytes (268435456/1024/1024), and the entire cache is stored in one segment. For our tiny directory, this is far more than we need. It is a safe setting, though the full 256 megabytes will not be allocated.

A good rule of thumb for estimating the minimum amount of cache you will need in a small or medium-sized directory is to allocate two megabytes of cache for every 100 megabytes of LDIF data, plus one megabyte of cache per index. Larger directories will definitely benefit though, from carefully-tuned caches. For a finer-grained calculation, see the OpenLDAP FAQ-O-Matic entry on setting cache sizes: [http://www.openldap.org/faq/data/cache/1075.html](http://www.openldap.org/faq/data/cache/1075.html).

### Configuring the Data Directory

The `set_data_dir` directive takes one parameter, which is the path to the directory that contains the database files. In the previous example this directive is commented out. Since the `DB_CONFIG` file is stored in the same directory as the BDB files themselves you should not need to set this directive. It only needs to be set when the `DB_CONFIG` file is loaded from a location outside of the database directory.

### Optimizing BDB/HDB Transaction Logging

The last three directives relate to transaction logging. As modifications are made to the Berkeley DB, the complete details of the transaction are written to log files, named `log.XXXXXXXXXX`, where the ten `X`'s are replaced by digits from 0-9\. The first log file is `log.0000000001`, and once it grows too large, a new log file is created by incrementing the number: `log.0000000002`.

The log files comprise a record of all that has happened in a database. In fact, they are so complete that they can be used to rebuild a corrupt database. The log file format is not plain text, and cannot be read using the usual tools (like `cat`, `more`, or `less`). To read it you will need to use the `db_printlog` command (or `dbX.Y_printlog`, where `X.Y` is replaced by the major and minor version numbers of the database, such as `db4.2_printlog`). This will display a record for each transaction made to the databases.

### Tip

**Recovering a Corrupt BDB/HDB Database**

The log files written by the Berkeley DB subsystem can be used to recover a corrupt database. The Berkeley DB distribution includes a tool called `db_recover` (or `dbX.Y_recover`, where `X.Y` is the major and minor version number, such as `db4.3_recover`). The `db_recover` tool uses the log files to fix corrupted databases. For more information view the man page for `db_recover`.

At startup SLAPD automatically performs a recovery on the BDB directory to ensure that the database is in a stable state. It is only in rare cases that a system administrator will have to manually work with the log files.

Since these transaction log files play such an important role in the safety of SLAPD's data, it is good to ensure that the environment is properly tuned.

The `set_lg_regionmax` directive controls the amount of memory allocated to storing the names of Berkeley DB files. It takes one argument: the amount of space to be allocated (in bytes). The file above allocates 256 KB for storing names, and this should be fine for almost all applications. Only in rare cases where there are many index files would it be necessary to raise this limit (I have never yet encountered such a situation).

The next directive, `set_lg_bsize`, is used to allocate the amount of memory used to buffer data before it is written to the transaction log. It too takes one argument: the amount of space (in bytes) to be used for a buffer. The setting in our file allocates two megabytes of space. When a modification is made to the BDB/HDB database, information about the modification is not written to the log until the transaction is complete. Until it is written it is temporarily stored in an in-memory buffer, whose size is no bigger than the value of `set_lg_bsize`.

Since most LDAP data is relatively short, two megabytes is usually sufficient. But if your particular directory frequently stores large chunks of data (such as image files), you may consider increasing the buffer size for the transaction log to accommodate the largest chunks of data. For example, if the directory stores images as large as ten megabytes, `set_lg_bsize` should be set at `10485760` (which is 10 * 1024 *1024).

Howard Chu, one of the OpenLDAP developers, points out that when increasing the `set_lg_bsize` flag to a value this large, you will also have to raise the maximum size limit for the log file using the `set_lg_max` flag. The maximum size for the log file must be *at* *least* four times the value of `set_lg_bsize`.

```
set_lg_max 41943040
```

Finally, the last directive, `set_lg_dir`, points to the log file for BDB. By default, these log files are stored in the same directory as the rest of the database files (`/var/lib/ldap/` or `/usr/local/var/openldap-data/` if you compiled from source ). However, since logs are crucial in recovery of the database, it is not a bad idea to store the log files in a different location than the databases. For example, you might want to store the logs on a different hard disk than the database files. To do so, uncomment the `set_lg_dir` directive and set it to the absolute path of the desired destination directory:

```
set_lg_dir /usr/local/var/ldap/
```

This directive will instruct the Berkeley DB subsystem to write the log files to `/usr/local/var/ldap` instead of the same directory that the BDB files are located.

### Note

Regularly backing up the Berkeley DB files (including the log files) is a good idea. A more portable way of backing up the data is to dump a copy of the directory using the `slapcat` tool. This will export the database into LDIF format, which can be easily imported into a SLAPD server, regardless of the backend format.

### Tuning Lock Files

There are three additional parameters that should be included in the `DB_CONFIG` file. These are the three directives that tune the locking mechanisms in Berkeley DB.

Certain operations on the database will require that the data be locked to prevent the introduction of data inconsistency. For example, it is not good to allow two different threads to modify the same record at the same time. Berkeley DB uses a locking mechanism to prevent this from happening.

There are three directives that are used to tune the locking subsystem. These are:

*   `set_lk_max_objects`: The maximum number of objects that can be locked at a given time
*   `set_lk_max_locks`: The maximum number of locks that can be requested at a time
*   `set_lk_max_lockers`: The maximum number of simultaneous lock requests

In the default Ubuntu `DB_CONFIG` file these are all set to 5000, but lower values (between 1500 and 3000) may be more desirable:

```
# Number of objects that can be locked at the same time.
set_lk_max_objects      5000
# Number of locks (both requested and granted)
set_lk_max_locks        5000
# Number of lockers
set_lk_max_lockers  5000
```

Setting these values at a sufficiently high value will prevent the database from running out of locks, and thus denying database access.

### Note

To see if your Berkeley DB lock settings are adequate, you can use the following command, which prints detailed information about locks and lockers:

**db4.2_stat -c**

### More about Berkeley DB

The directives we have covered in this section are those that get the most attention for OpenLDAP. However, there are other directives, and judicious use of such settings can also improve the performance and reliability of the BDB and HDB backends.

Some information about these parameters can be found in OpenLDAP's FAQ-O-Matic ([http://www.openldap.org/faq/data/cache/1072.html](http://www.openldap.org/faq/data/cache/1072.html)). For a thorough understanding, though the best resource is the *Berkeley* *DB* *Reference* *Guide.* The newest version can be found here: [http://www.oracle.com/technology/documentation/berkeley-db/db/ref/toc.html](http://www.oracle.com/technology/documentation/berkeley-db/db/ref/toc.html)

At this point we have looked at the `slapd.conf` and `DB_CONFIG` files, examining some of the ways that these files can be modified to improve the performance of SLAPD. Next, we will turn to a different topic: extending the functionality of SLAPD using directory overlays.

# Directory Overlays

As the OpenLDAP project has grown, more and more features have been added. Initially, these features were added directly to the SLAPD server's code base. But as features were rolled into OpenLDAP, both the code and the configuration became increasingly complex.

To address this problem, OpenLDAP developers introduced a new concept in OpenLDAP 2.2 that made it easier to introduce new features while reducing the complexity of the underlying code. The developers introduced a modular system called **overlays**. An overlay is a chunk of code that can modify the behavior of the SLAPD.

When SLAPD receives a request for a database configured to use an overlay, the overlay is given an opportunity to perform processing on the request before any information is retrieved from the underlying database. As a result overlays can be used to perform additional processing of requests.

How is an overlay added to the directory server? It is through special directives in the `slapd.conf` file. The `overlay` directive is placed in the database configuration section, though an overlay sometimes intercepts operations that are not backend-specific.

More than one overlay can be used in a database. When overlays are used this way, they are said to be **stacked**. As we will see later in this chapter the order of overlay directives is very important because SLAPD sequentially goes through the overlay stack, calling the overlays one at a time.

## A Brief Tour of the Official Overlays

In OpenLDAP 2.3 there are sixteen *official* overlays included with the OpenLDAP distribution, and a handful of contributed and unofficial overlays. Almost all of the official overlays are described in the man pages. Here we have a brief description of each of the sixteen; we will also see a few useful overlays in more detail. In the later chapters we will also make use of overlays.

The *official* overlays are as follows:

1.  `accesslog`: The access logging overlay is used to record information about directory access and utilization. Rather than recording the data in the file system, information is stored as records inside a special log directory. Logs can then be retrieved through LDAP clients, or by using a tool such as `slapcat` to dump the logs into a flat (LDIF) file. We will implement this overlay in the next chapter, and use it again in Chapter 7 to improve replication.
2.  `auditlog`: The audit logging overlay records information on changes to the directory. Unlike the more powerful access logging overlay, audit log stores information in a file in the file system.
3.  `chain`: In complex directory environments, one directory may have information that another directory does not have. That second directory may be configured to *refer* clients of the first directory. Typically, a referral involves sending the client information about redirecting its query, and then the client is left to chase the referral. The chain overlay handles referral chasing on the server side; the server will follow the referral itself and return the complete information to the client.
4.  `denyop`: The deny operation overlay performs the same sort of function as the restrict directive discussed earlier in this chapter. It disallows clients from performing certain LDAP operations. In the next section we will use this overlay.
5.  `dyngroup`: The `dyngroup` overlays provide ways of creating dynamic groups based on specific attributes in an object. This provides a powerful method of grouping records.
6.  `dynlist`: It is similar to the `dyngroup` overlay.
7.  `glue`: The glue overlay, which is built-in and loaded by default, makes it possible to link two databases together so they appear as if they were one large directory information tree. For example, if one database contains `dc=example,dc=com`, and a second database holds `ou=Users,dc=example,dc=com`, the glue overlay makes it possible for searches of `dc=example, dc=com` to return entries from the `ou=Users,dc=example,dc=com` database. The `subordinate` directive must be used in the database section of `slapd.conf` to indicate which databases should be glued.
8.  `lastmod`: The last modification overlay creates a special record in the directory information tree that contains information about what the most recently modified record is and when it was modified.
9.  `pcache`: The proxy cache overlay caches the results of an LDAP search. This overlay is mainly used with the `ldap` backend. With this combination, SLAPD can be configured to use another LDAP server as its backend, but speed up client requests by keeping a cached copy of the data in a special database.
10.  `ppolicy`: The password policy overlay allows you to enforce certain restrictions, such as password expiration dates and password length. The password policy overlay is described in the next chapter.
11.  `refint`: The referential integrity overlay is used to keep directory entries consistent when records are deleted or DNs are modified. For example, if a DN is deleted from the directory and the `refint` overlay is used, SLAPD will search the directory for other references to this DN (such as group memberships) and remove those references as well. We will look at this later in the chapter.
12.  `retcode`: This overlay is designed to help LDAP client implementors test how their code responds to abnormal server responses.
13.  `rwm`: The rewriting and mapping overlay provides a facility for taking a client request and re-writing or mapping parts of the request to other values. This can be used in conjunction with a proxying LDAP server to re-write attribute names and DNs.
14.  `syncprov`: The synchronization provider overlay is used by SLAPD servers that act as providers from which other SLAPD servers replicate data. We will discuss this more in Chapter 7.
15.  `translucent`: The `translucent` overlay is similar to the proxy overlay. When a client requests a record, it retrieves the record from a remote server. But, it can do more—it can store a local copy of the record that can override portions of the remote record.
16.  `unique`: The `unique` overlay enforces attribute uniqueness. It is used to ensure that, for specified attributes, a given attribute value exists in only one record in the directory. This is useful to keep multiple users from having the same email address (`mail`) or user ID (`uid`) attribute values.

Each of the overlays documented here (except for `denyop`) has a corresponding man page that can be accessed using the command `man` `slapo-<name` `of` `overlay>`, where `<name` `of` `overlay>` is replaced with the abbreviated name of the overlay. For example, to get the man page for the `translucent` overlay, run the command: `man` `slapo-translucent`.

In the remainder of this chapter we will cover a few simple overlays in detail. In the next few chapters we will cover several sophisticated overlays, using them to address common directory server needs.

## Configuring an Overlay: denyop

Since we have covered the basic concepts behind the `denyop` overlay when we looked at the `restrict` directive, and since `denyop` is simple to implement, we will look at it as an example for how to use an overlay.

### Note

The `restrict` directive is actually the preferred method of restricting operations. The `denyop` overlay was intended primarily as an example for other overlay authors.

Overlays are configured in the `slapd.conf` file. Typically there are three steps to configuring an overlay:

1.  Load the dynamic object with the moduleload directive
2.  Add the overlay to the *database* *section* with the `overlay` directive
3.  Add any overlay-specific directives to the database section

Let's look at each step in detail.

### Loading the module

The first task is to load the module containing the overlay. This part is not always necessary. Some versions of OpenLDAP have all of the modules statically compiled in, which means they are loaded along with the server. More often though, SLAPD is compiled to dynamically load modules that are loaded when SLAPD starts, and almost all overlays are implemented as modules.

### Note

See [Appendix A](apa.html "Appendix A. Building OpenLDAP from Source") for a further discussion of the difference between these two ways of building OpenLDAP.

The `moduleload` directive should go near the top of the configuration file, before the first `database` directive. To load the `denyop` dynamic object we need to add the following highlighted line:

```
modulepath /usr/lib/ldap
moduleload back_hdb
moduleload denyop

```

When SLAPD starts it will search for the `denyop` object in its module path, and load it if it finds it.

### Note

If you need to load a module not in the module path you can specify the full path to the module. For example `/usr/local/libexec/openldap/my_module`.

If SLAPD fails to find the module on startup it will fail to start, exiting with an error like this:

```
lt_dlopenext failed: (/tmp/lastmod) /tmp/lastmod.so: cannot open 
                   shared object file: No such file or directory
```

This indicates that the module, `lastmod`, was not found in the given module path, which in this case was erroneously set to `/tmp`.

Make sure that the module is in one of the directories listed in `modulepath`, or that the full path to the module is correct.

### Adding the Overlay

The next step is to add the overlay to the overlay stack. Since there are no overlays already specified, this will be the first of three items on the stack. The `glue` overlay is automatically applied, though it does nothing unless a `subordinate` directive is present. The backend processing of the operation (the actual directory lookup) is always the last item on the stack.

To add our overlay we need to put the directive in the appropriate database section of the `slapd.conf` file. In a situation where there are multiple backends, the same overlay directive can be repeated in each database section to load the overlay for each database. The new directive is highlighted in the following example:

```
database hdb
suffix "dc=example,dc=com" "o=My Company,c=US"
rootdn "cn=Manager,dc=example,dc=com"
rootpw secret
directory /var/lib/ldap
overlay denyop

```

Now, we are ready for the third step.

### Adding Overlay-Specific Directives

An overlay may have its own special directives. These directives are usually documented in the man page for that overlay.

There is only one directive supported by the `denyop` overlay, and it is the eponymous `denyop` directive. Like the `restrict` directive that we looked at earlier, the `denyop` directive takes a list of operations. Clients will be disallowed from performing any operation in this list.

Earlier in this chapter we used the `restrict` directive to prevent clients from performing `add`, `delete`, and `rename` operations:

```
restrict add delete rename
```

We can implement the same thing with the `denyop` directive:

```
denyop add,delete,modrdn
```

There are a few minor differences between the two directives:

*   `denyop` takes a comma-separated list of operations
*   `denyop` uses the `modrdn` name instead of using the term `rename`

If a client attempts to perform one of the disallowed operations `denyop` will stop SLAPD from performing the operation, and the client will be returned an `Unwilling` `to` `perform` error.

The `denyop` overlay is simple and, due to the restrict directive, not likely to enjoy much use in a production server. But the next overlay that we will look at provides useful features, though the accompanying directives are slightly more complex.

## Referential Integrity Overlay

The second overlay we will examine is the RefInt (Referential Integrity) overlay. RefInt is designed to handle cases where the modification or deletion of a record may render attribute values in other records inaccurate.

LDAP groups provide a good context for illustrating the problem that the RefInt overlay is designed to address. In Chapter 3 we created an LDAP group that looked like this:

```
dn: cn=Admins,ou=Groups,dc=example,dc=com
objectClass: groupOfNames
cn: Admins
ou: Groups
member: uid=matt,ou=users,dc=example,dc=com
member: uid=david,ou=users,dc=example,dc=com
```

This group has two members, `uid=matt`, and `uid=david`. These two member attributes refer to other records (identified by their respective DNs) that are also located in the directory. For example, here is the record for `uid=david`:

```
dn: uid=david,ou=Users,dc=example,dc=com
cn: David Hume
sn: Hume
uid: david
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

What would happen to the `cn=Admins` group if we deleted this record for `uid=david` from the directory information tree? Nothing! The `cn=Admins` group would still contain a member attribute with the DN for `uid=david`. By default, SLAPD does not do any searching for references to a modified or removed DN. Why? The assumption has generally been that such tasks are the responsibility of the applications that access and modify the directory.

But keeping a directory free of invalid references is not a job that everyone wants to leave to external applications. For that reason the OpenLDAP developers created the RefInt overlay, which makes the task of maintaining referential integrity the responsibility of SLAPD.

There are two cases when the RefInt overlay kicks into action:

*   When a DN is modified (via a `modrdn` operation): The RefInt overlay does a search of the directory (searching only the values of the attributes specified in the configuration), and replacing any occurrences of the old DN with the newly modified DN
*   When a record is removed (with a `delete` operation): The RefInt overlay searches the directory (looking for the specified attributes only), and deletes any references to the DN that it finds

We will look at examples of these, but first let's configure the overlay.

### Configuring the Overlay

The first step to configuring the overlay is to make sure the module is loaded. This is done (as always) by adding a `moduleload` directive in the basic section of the `slapd.conf` file, before the first database section:

```
modulepath /usr/lib/ldap
moduleload back_hdb
moduleload denyop
moduleload refint

```

This example builds on our earlier `moduleload` example. Only the highlighted line has been added.

Next, we want to add the overlay to the stack, and configure it for operation. These configuration directives go in each database section for which we want to use the overlay:

```
overlay refint
refint_attributes member uniqueMember seeAlso
refint_nothing cn=EMPTY
```

The first line, the `overlay` directive, adds RefInt to the overlay stack. Remember, it's position relative to other `overlay` directives will determine its position on the overlay stack.

On the next line is the `refint_attributes` directive. This directive takes a list of attributes (separated by whitespace characters) that will be searched whenever a `modrdn` or `delete` operation is performed. We want to include all of the attributes that we want SLAPD to maintain referential integrity for.

Since we have records that are `groupOfNames` and `groupOfUniqueNames` object classes, we want the RefInt overlay to check the `member` and `uniqueMember` attributes. The `seeAlso` attribute, which is an attribute allowed for `organization`, `organizationalUnit`, and `person` objects (all of which are used in our directory information tree), takes a DN for a value, so we want RefInt to check it as well.

### Tip

**The seeAlso Attribute**

The `seeAlso` attribute, which allows only values that are DNs, is used to indicate a connection between the record that contains the `seeAlso` attribute, and the record or records that the `seeAlso` attribute points to. There are other attributes, such as the `manager` attribute for `inetOrgPerson` objects, which also take DN values.

The last directive, `refint_nothing`, is used in special cases when RefInt is responding to a `delete` operation.

Sometimes it is not possible for RefInt to delete an attribute value. This happens when a record must (according to the schema) have at least one such attribute value. For example, any `groupOfNames` object must have at least one `member` attribute value. The schema does not allow groups with no members.

But what if deleting an entry would require RefInt to remove the only `member` attribute from a group? We wouldn't want RefInt to try to violate the server's schema constraints.

RefInt avoids the problem this way: RefInt adds the DN in `refint_nothing` as a value for that attribute, and then deletes the other attribute. Effectively, it replaces the deleted value with a known placeholder value.

In the previous example we have set the `refint_nothing` DN to be `cn=EMPTY`. There is no entry in our directory information tree named `cn=EMPTY` (though if there were, it would cause no problems).

### Modifying the Records

Now, we will add two records to our directory:

```
dn: uid=marcus,ou=users,dc=example,dc=com
uid: marcus
sn: Tullius
cn: Marcus Tullius
givenName: Marcus
ou: users
objectclass: person
objectclass: organizationalperson
objectclass: inetOrgPerson

dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectclass: groupOfNames
cn: Public Relations
ou: Groups
member: uid=marcus,ou=users,dc=example,dc=com
```

The first record is for a new `inetOrgPerson` with the UID `marcus`. The second record defines the `cn=Public` `Relations` group which currently has one member, `uid=marcus`. What happens to the `member` attribute of `cn=Public` `Relations` if we delete the record for `uid=marcus` by using the following command?

```
  $ ldapdelete -U matt uid=marcus,ou=users,dc=example,dc=com
```

Now, we search for the `cn=Public` `Relations` group:

```
  $ ldapsearch -U matt -LLL '(cn=Public Relations)'
```

The record looks like this:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectClass: groupOfNames
cn: Public Relations
ou: Groups
member: cn=EMPTY
```

As the last line of the code illustrates, there is still one member (the `groupOfNames` schema requires that there be one) but, thanks to the RefInt overlay, it no longer points to the deleted `uid=marcus` record. Instead it points to the DN we specified in `refint_nothing`.

Usually, though, the record will have more than one member attribute, like the `cn=Admins` example earlier. In such a case when one of those DNs is deleted, the attribute value is completely removed. Consider a modified version of our `cn=Public` `Relations` group:

```
dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectclass: groupofnames
cn: Public Relations
ou: Groups
member: uid=david,ou=users,dc=example,dc=com
member: uid=marcus,ou=users,dc=example,dc=com

```

If the record for `uid=marcus` was deleted in this case, then the RefInt overlay would simply delete the second member attribute value, leaving the group looking like this:

```
dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectclass: groupofnames
cn: Public Relations
ou: Groups
member: uid=david,ou=users,dc=example,dc=com

```

The value of `refint_nothing` is used only when required.

These last two examples have dealt with cases where the `delete` operation is used. But the RefInt overlay also handles changes to DNs made with the `modrdn` operation. For example, what if instead of deleting the record for `uid=marcus` we changed the DN? Using the previous example let's begin with the same two records:

```
dn: uid=marcus,ou=users,dc=example,dc=com
uid: marcus
sn: Tullius
cn: Marcus Tullius
givenName: Marcus
ou: users
objectclass: person
objectclass: organizationalperson
objectclass: inetOrgPerson

dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectclass: groupofnames
cn: Public Relations
ou: Groups
member: uid=marcus,ou=users,dc=example,dc=com

```

Let's change the DN of the first record to use Marcus Tullius's better-known name:

```
 $ ldapmodrdn -U matt uid=marcus,ou=users,dc=example,dc=com
      uid=cicero
```

In the previous example, we are changing the DN `uid=marcus,ou=users,dc=example,dc=com`, replacing the relative DN portion (`uid=marcus`) with a new relative DN: `uid=cicero`. Now the first record looks like this:

```
dn: uid=cicero,ou=users,dc=example,dc=com
uid: marcus
uid: cicero
sn: Tullius
cn: Marcus Tullius
givenName: Marcus
ou: users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

The `ldapmodrdn` client added a new `uid` attribute value (`cicero`), and then changed the DN of the entry from `uid=marcus,ou=users,dc=example,dc=com` to `uid=cicero,ou=users,dc=example,dc=com`. And what about the `cn=Public` `Relations` group? It now looks like this:

```
dn: cn=Public Relations,ou=Groups,dc=example,dc=com
objectClass: groupOfNames
cn: Public Relations
ou: Groups
member: uid=cicero,ou=users,dc=example,dc=com

```

The RefInt attribute changed the value of the `member` attribute to point to the newly modified DN. Remember, without the RefInt overlay, the `cn=Public` `Relations` group would point to the now non-existent DN `uid=marcus,ou=users,dc=example,dc=com`.

### Drawbacks

Are there any drawbacks of using the RefInt overlay? Performance is one issue. For every deletion or DN modification, the RefInt overlay will check all the values for all of the attributes listed in the `refint_attributes` directive. A large number of deletions or DN modifications can have an impact on system performance. But in most situations, large-scale `delete` and `modrdn` operations are not the norm (and the overlay can always be turned off when doing such operations).

There is one other drawback worthy of consideration. Some applications do handle their own reference checking. It is possible that a poorly-written client might try to delete attribute values that do not exist, generating spurious error messages in the process. Of course, this would not have any negative effect on the directory information tree, but it might alarm the user. However, the vast majority of clients, including many that perform their own integrity checking, should not be hampered by the RefInt overlay.

### A Useful Note

When starting up SLAPD after installing a new overlay, it is not uncommon to get the following warning message:

```
WARNING: No dynamic config support for overlay refint.
```

What does this message mean? And is the problem serious?

This warning message can be ignored when configuring OpenLDAP with a `slapd.conf` file. It is simply a notice that the configuration options for this overlay cannot be changed once the server starts. But this is, of course, how all directives in the `slapd.conf` file work.

This warning message applies only to installations that load their configuration into the directory as an LDIF file, and then manage their configuration inside of the directory server (using the `cn=Config` record). This feature is fairly new, and since it does not support all of the OpenLDAP features (such as many overlays), it is not the recommended configuration for most clients.

# The Uniqueness Overlay

The last overlay that we will examine in this section is the uniqueness overlay. The uniqueness overlay enforces uniqueness for a configurable set of attributes in the directory. It prevents attributes in different records from containing the same values. This is desirable, for example, when working with the `uid` attribute, where we would clearly not want to have the same UID for multiple users in the system. By default, SLAPD only enforces uniqueness when it comes to DNs—no two DNs may be the same. But other attribute values are unchecked. Using the uniqueness overlay, we can specify which attributes we want SLAPD to ensure uniqueness for.

The first step in configuring the uniqueness overlay is to load the module:

```
modulepath      /usr/local/libexec/openldap
moduleload      back_hdb
moduleload      denyop
moduleload      refint
moduleload      unique

```

In the *Basics* section of `slapd.conf`, we add one more `moduleload` directive. The module we want to load is named `unique`.

Next we want to add this overlay, along with a few specific directives, to the relevant database sections:

```
overlay unique
unique_base dc=example,dc=com
unique_attributes uid
```

This is a very basic configuration for the uniqueness overlay. The `unique_base` directive indicates which parts of the directory information tree we want to enforce uniqueness in. For our exercise we want to enforce uniqueness across our entire directory tree, `dc=example,dc=com`.

The `unique_attributes` directive takes a whitespace-separated list of attributes that the uniqueness overlay will enforce uniqueness constraints. In this example we just want to enforce uniqueness on the UID attribute.

### Note

The behavior of the uniqueness overlay is expected to change in the next version of OpenLDAP (version 2.4). In particular, it will support multiple bases inside a single database.

Thus, according to our configuration, no two UID values for any records in the `dc=example,dc=com` directory information tree should be identical.

Now let's see how this overlay works in practice.

In the discussion of the RefInt overlay, we created the following record:

```
dn: uid=cicero,ou=users,dc=example,dc=com
uid: marcus
uid: cicero
sn: Tullius
cn: Marcus Tullius
givenName: Marcus
ou: users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

Note that this record has the UID `marcus`, even though this attribute is not used in the DN. Now let's try to add the following record:

```
dn: uid=marcus,ou=users,dc=example,dc=com
uid: marcus
sn: Aurelius
cn: Marcus Aurelius
givenName: Marcus
ou: users
objectclass: person
objectclass: organizationalperson
objectclass: inetOrgPerson
```

This record also uses the UID `marcus`. Without the uniqueness overlay, SLAPD would allow both records to have the same UID. This, of course, will cause problems for applications that assume that a Unique ID is really unique—only zero or one results will be returned for a search on the UID attribute.

But with the uniqueness overlay, as we have configured it, SLAPD will prevent clients from adding a UID value that matches an existing UID value. The uniqueness overlay does this by checking the attributes in `add`, `modify`, or `modrdn` operations. If we try to add the given record for `uid=marcus`, we get an error:

```
$ ldapadd -U matt -f unique-example.ldif
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

adding new entry "uid=marcus,ou=users,dc=example,dc=com"
ldap_add: Constraint violation (19)
 additional info: some attributes not unique

```

SLAPD sends back a **Constraint violation** error because the uniqueness overlay will not allow a duplicate UID attribute value. To work around this, we would have to either delete the extra UID attribute from the `uid=cicero` record or use a different UID for Marcus Aurelius's record.

The example configuration we have just seen represents the most typical use of the uniqueness overlay. There are two additional uniqueness directives that provide more complex configurations:

The first is the `unique_ignore` directive. Typically, this is used *instead* of `unique_attributes`.

### Tip

While you can use both `unique_attributes` and `unique_ignore`, it is not recommended because it can cause unexpected behavior. See the man page for more detail: `man` `slapo-unique`.

The `unique_ignore` directive takes a whitespace-separated list of attributes that *should* *not* be tested for uniqueness. There are attributes, such as `ou`, `sn`, and `objectclass`, that are likely to be legitimately used more than once in a directory. For example, it is perfectly possible for multiple people in an organization to have the same surname, and thus have identical `sn` attribute values.

But when `unique_attributes` is not specified, then by default all *non-operational* *attributes* are assumed to require uniqueness. Consider this example configuration:

```
  overlay unique
  unique_base dc=example,dc=com
  unique_ignore objectclass sn ou description
```

According to this configuration, all of the attribute values in the directory information tree except `objectclass`, `sn`, `ou`, and `description` will be required to have unique values. Obviously, this configuration is far more restrictive than our first example and it should be used with care.

### Note

Operational parameters—those intended for internal SLAPD use—are not automatically added to the uniqueness list under any circumstances. Doing so might cause hard-to-debug errors that would prevent SLAPD from functioning properly.

Finally, there is one additional directive for the unique overlay. The `unique_strict` directive, which takes no parameters, can be used to turn on "strict" uniqueness enforcement.

By default, the uniqueness overlay allows multiple attributes to have empty (null) values. For example, if we enforce uniqueness on the `uid` attribute, SLAPD would still allow multiple records to have a UID attribute with an empty value. But this is not always desirable. Under some circumstances, it might be necessary to ensure that no more than one attribute has an empty value. The `unique_strict` directive is used for this purpose.

When the `unique_strict` directive is present, the uniqueness overlay will not allow a client to set an attribute value to empty (null) if another instance of the same attribute already exists and already has an empty value.

At this point, you should have a good idea of how to use overlays. We have looked at three different overlays but in the coming chapters we will look at several more.

# Summary

The focus of this chapter has been on advanced configuration of the SLAPD server. We began by taking a second look at the `slapd.conf` file. Then we added an additional database to our directory server, supporting a second directory information tree. From there we looked at some ways of improving SLAPD's performance using directives in the `slapd.conf` file, and also tuning the Berkeley DB's `DB_CONFIG` file. In the last section we looked at SLAPD's overlay engine, touring three specific overlays.

By now you should feel comfortable working with the `slapd.conf` file as well as using overlays.

In the next chapter we will examine LDAP schemas, adding a few schemas for new overlays, and then creating our own schema. Later, in Chapter 7, we will expand upon some of the themes in this chapter when we look at the ways to configure multiple OpenLDAP servers to work together.