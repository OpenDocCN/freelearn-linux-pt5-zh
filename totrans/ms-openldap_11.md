# Appendix C. Useful LDAP Commands

In the course of this book we looked at all the command line tools that come in the OpenLDAP distribution. But the scope of this book requires the discussion of each of these tools briefly. There are some advanced uses of these tools that can come in handy at times. In this appendix I have provided examples of such uses.

In this appendix, we will cover

*   Getting information about the directory using `ldapsearch`
*   Creating backups of the directory using two different strategies
*   Rebuilding a BDB/HDB database

# Getting Information about the Directory

Many LDAP servers provide information about their configuration and functional abilities. This information is stored in such a way that LDAP clients can directly access it using a search operation. For example, a client can fetch the **root DSE** record to find out the basic capabilities of the server. It can also access the **subschema** of the server and find out what object classes, syntaxes, matching rules, and attributes are supported.

## The Root DSE

The **root DSE (DSA-Specific Entry**, where **DSA** stands for **Directory Service Agent**) is a special entry that provides information about the server itself. The DN of the root DSE is an empty string (""). To retrieve it we need a carefully-crafted LDAP search that will set an empty search base and then retrieve that root entry:

```
 $ ldapsearch -x -LLL -b '' -s base -W -D \
 'cn=Manager,dc=example,dc=com' '+'

```

Note that the base is set to an empty string, and the search scope is limited to the base record. These parameters combined have the effect of requesting only the record that has an empty DN. Also, since most of the attributes in the root DSE are operational attributes, we need to specify `'+'` at the end of the search.

The results of running this search look something like this:

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

Among other things, this record gives us information about what controls, features, and extensions are understood by and enabled on the server. For example, there is a `supportedFeature` line that reads:

```
supportedExtension: 1.3.6.1.4.1.4203.1.11.1
```

This line indicates that this LDAP server supports an LDAPv3 extension for **Change Password** operations as defined in RFC 3062 ([http://www.rfc-editor.org/rfc/rfc3062.txt](http://www.rfc-editor.org/rfc/rfc3062.txt)).

Using this information, a well-crafted LDAP client would be able to perform a server-side Change Password operation instead of changing the password on the client side and then using a Modify operation to send the change to the server.

### Note

The advantage of the Change Password operation is in the server's storage. If the client changes a password through a Modify operation it must know in advance what types of encryption are supported on the server, it must do the encrypting itself, and then submit the encrypted password to the server. Usually, it is better to have the client securely contacting the server (over TLS, for example), and then using a Change Password operation so that the server can do the storage.

The root DSE record also points to the configuration (`cn=config`) and subschema (`cn=subschema`) records.

## The Subschema Record

The subschema record is stored in `cn=subschema`. This record contains detailed information about the schemas supported by the server, including what types of matching rules it has available, what sort of syntaxes are allowed in attributes, and what attributes and object classes are recognized by the server.

This information can be used by client applications to correctly craft records or searches, and then correctly interpret the responses.

The subschema record can be retrieved with `ldapsearch` using the following command:

```
 ldapsearch -x -LLL -b 'cn=subschema' -s base -W \
 -D 'cn=Manager,dc=example,dc=com'  '+'

```

In this example we request the desired record by setting the base DN to `cn=config`, and then requesting a search type of `base` (`-b` `'cn=subschema'` `-s` `base`). This returns the exact record with the DN `cn=subschema`.

Also, most of the attributes we want are operational attributes, which means they will not be returned in a normal search, so at the end we specify `'+'` to indicate that we want the operational attributes.

The record returned looks like this:

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

A subschema record contains all of the schema information and thus, it may be well over a thousand lines.

Subschema records can be particularly useful to learn about what schemas a server supports, or when developing and debugging custom schemas, as discussed in Chapter 6.

## The Configuration Record

An experimental feature of OpenLDAP 2.3 (and one that will probably reach production quality in OpenLDAP 2.4) is the ability to store the LDAP configuration inside of the directory. To do this you must first re-create your configuration in LDIF format using a special configuration schema, and instruct SLAPD to read its configuration from this new LDIF file.

The configuration is stored inside of the directory with the DN `cn=config`. It can be accessed with a search similar to the one used in the previous section:

```
 ldapsearch -x -LLL -b 'cn=config' -s base -W \
 -D 'cn=Manager,dc=example,dc=com'  '*'

```

In OpenLDAP 2.3, not all of the overlays and features of OpenLDAP work correctly with this new configuration style, and that is a significant drawback to its use. But improving this alternate configuration mechanism is a priority for development in OpenLDAP 2.4.

What might be the advantages of storing your configuration in the directory? Here are a few:

*   Easy access to configuration information through `ldapsearch` and other LDAP clients.
*   The ability to edit configuration information through directory tools like `ldapmodify`.
*   Replication support for SLAPD configuration. You may be able to use SyncRepl to synchronize directory configurations across the network.

If you would like to implement the new LDAP-based configuration file format, you can learn about it in the LDAP Administrators Guide at the OpenLDAP site: [http://www.openldap.org/doc/admin23/slapdconf2.html](http://www.openldap.org/doc/admin23/slapdconf2.html).

# Making a Directory Backup

There are two common strategies for backing up the contents of your directory. One is to make a backup of the directory database. The other is to dump the contents of the directory into an LDIF file.

## A Backup Copy of the Directory Database

Different backends locate the contents of the directory in different locations. For example, the BDB and HDB backends store data in special Berkeley DB database files. SQL-based backends store the information in a relational database management system. Special backends like the LDAP and Perl backends may not store data at all, but might simply access other sources.

Each of these backends will require a different backup procedure. Here we will just look at backing up BDB and HDB databases—the types we've used throughout the book.

### Note

This method is not portable. BDB/HDB files are version sensitive. Each new release of OpenLDAP (or of Berkeley DB) may use different structures for these databases, so this backup method only works when the backup and the restore are done on the same software versions.

In Ubuntu these database files are located at `/var/lib/ldap`. All of the files in this directory, including the indexes (those that end with the `bdb` extension), the main database files (`__db.???`) and the log files (`log.??????????`). It is also a good idea to make a copy of the `DB_CONFIG` file, though it rarely changes and does not store any directory data.

When backing up these files it is best to stop SLAPD. Here's a very simple example using common shell tools:

```
 $ sudo invoke-rc.d slapd stop
 $ sudo cp -a /var/lib/ldap/* /usr/local/backup/ldap/
 $ sudo invoke-rc.d slapd start

```

This will stop SLAPD and copy all of the files at `/var/lib/ldap/` to `/usr/local/backup/ldap/`. Then, SLAPD will be started again.

## An LDIF Backup File

The second, and more portable, strategy for backing up the directory is to dump the contents of the directory to an LDIF file. There are several distinct advantages to this approach:

*   There is no need to stop SLAPD
*   The output is more portable, and data can be moved from one database backend to another, and from one OpenLDAP version to another

There is less redundant data, so backup files are much smaller than the BDB/HDB files.To make an LDIF backup file of the contents of a directory server with only one database (that is, it has only one directory root), the command is simple:

```
 $ sudo slapcat -l /usr/local/backup/my_directory.ldif

```

This command uses `slapcat` to dump the contents of the directory, in the LDIF format, into the file `/usr/local/backup/my_directory.ldif`. It can be loaded back into the directory using the `slapdadd` tool discussed in Chapter 3.

If your directory contains more than one directory information tree, you will need to run the `slapcat` routine once for each server, using the `-b` flag to identify the suffix (base DN) of the directory information tree you want to dump:

```
 $ cd /usr/local/backup
 $ sudo slapcat -b "dc=example,dc=com" -l example_com.ldif 
 $ sudo slapcat -b "dc=test,dc=net" -l test_net.ldif

```

In this example we backup each directory into its own LDIF file.

# Rebuilding a Database (BDB, HDB)

Sometimes it is necessary to rebuild a backend database. This process differs depending on the database backend. For instance, with a SQL backend, it might entail dumping, dropping, and re-creating tables in the database.

### Note

Moving to a new server and transferring contents to a new slave server are also processes similar to rebuilding a database, and the differences are mentioned within the text here.

The most commonly-used backends for OpenLDAP are the HDB and BDB backends (both based on the Berkeley DB lightweight database). In this section, I want to cover the process of rebuilding these databases.

This process consists of five steps:

1.  Stop SLAPD
2.  Dump the directory data into a file
3.  Delete the old directory files
4.  Create a new database
5.  Start SLAPD

None of these steps is particularly difficult. In fact, for a small to medium-sized directory, this process can be done in less than ten minutes.

### Note

**Moving from Server to Server**

Moving a directory from one server to another is done by a process very similar to that described here. Only step three, as mentioned later, differs. In this case, instead of deleting directory files, the LDIF file would be transferred from the original server to the new server. Steps one and two would be run on the original server, and steps four and five would be done on the new server.

## Step 1: Stop the Server

The purpose of stopping the server is to prevent additional changes to the directory information tree while we are working on it.

### Note

If you are just dumping the contents of a master directory to import into a shadow server that will use SyncRepl, you need not stop the server. Any changes that happen after the directory has been dumped will be retrieved by the shadow server during its first LDAP synchronization operation.

This can be done either by killing the server's process ID, or by running the startup script with the stop command:

```
 $ sudo invoke-rc.d slapd stop

```

Now that the server is stopped, we can dump the database.

## Step 2: Dump the Database

In Chapter 3 I covered the OpenLDAP utilities. One of the tools I discussed was the `slapcat` program, which is a tool for dumping the contents of the directory into an LDIF file. That is the program we will use in this step.

Why use `slapcat` instead of an `ldapsearch`? There are two reasons.

First, `slapcat` preserves all of the attributes (and records for that matter) that the LDAP server uses, including the operational attributes that are stored. (Those operational attributes that are generated at runtime are not generated by `slapcat`, and that is good. We wouldn't want to import those, anyway.)

Second, `slapcat` accesses the database directly, instead of opening an LDAP connection to the server. That means that ACLs, time and size limits, and other by products of the LDAP connection are not evaluated, and hence will not alter the data.

The BDB/HDB database is stored in a small set of files located at `/var/lib/ldap` (or `/usr/local/var/openldap-data` if you built from source). Usually access to those files is restricted to only the ID of the SLAPD user. By default this is `root` or `ldap`. In order to extract information using `slapcat`, you will need to have access to those files.

We have this command:

```
 $ sudo slapcat -l /tmp/backup.ldif

```

This command executes `slapcat` as root. The `-l` flag is used to pass in the name of the output file. In this case the file `backup.ldif` will be created in the `/tmp` directory.

### Note

You may prefer putting the LDIF file in a folder other than `/tmp`, especially if you plan on keeping the LDIF file for more than a few minutes.

In most cases the `-l` flag is the only one you will need. If you have more than one backend and you only want to dump one, you can use the `-n` flag to specify which backend to dump.

Once the `slapcat` is complete, we are done with this step.

Before continuing however, you may want to check the contents of the LDIF file to make sure that it is not corrupt. Do this before deleting the database files.

## Step 3: Delete the Old Database Files

If you are re-building a database you will want to delete the old database files before building new ones.

### Note

You do not need to do this if you are either migrating from an old server to a new server or configuring SyncRepl shadow servers.

These files are stored at `/var/lib/ldap` (or `/usr/local/var/openldap-data` if you built from source). However, not all of the files in that directory should be deleted. We only want to delete:

*   The index files: files that end in '`.bdb`'.
*   The main database files: files named `__db.???`, where the question marks are replaced by numbers in sequence (`__db.001`, `__db.002`, and so on).
*   The `alock` file: a file used internally for storing locking information. (Usually, this can be left with no negative consequences, but if SLAPD crashed, this can be left in an unstable state.)
*   The BDB log files: files named `log.??????????`, where the ten question marks are replaced by numbers in sequence: `log.0000000001`, `log.0000000002`, and so on.

There is one file we definitely do not want to delete. This is our database configuration file, `DB_CONFIG`. Deleting it would cause the BDB engine to use its default settings, which are not tuned to our needs, and generally cause OpenLDAP to perform poorly.

So, to delete the files, we can do the following:

```
$ cd /var/lib/ldap
$ sudo rm __db.* *.bdb alock log.*
```

To reduce the risk of data loss, you may want to backup the `__db.*`, `*.bdb`, and `log.*` files before removing them. Or instead of doing an `rm`, you may use `mv` to move the files to a different location:

```
$ cd /var/lib/ldap
$ sudo mkdir backup/
$ sudo mv *.bdb log.* alock __db.* backup/ 
```

Now the database directory has been cleared. We are ready to create new database files.

## Step 4: Create a New Database

The new database can be created and populated with the data all in one step, using the `slapadd` utility that we covered in Chapter 3\. Still in the OpenLDAP data directory, run the following command:

```
 $ sudo slapadd -l /tmp/backup.ldif

```

This will create all of the necessary files, import the LDIF file, and handle all of the data indexing as well.

### Note

If you are running your LDAP server as a user other than root (and it is a good idea to do so), you will also need to use `chown` to change the ownership on all of the files at `/var/lib/ldap` to be owned by the SLAPD userID: `sudo` `chown` `openldap` `*.bdb` `log.*` `__db.*`.

All we need to do now is restart the server.

## Step 5: Restart SLAPD

If you stopped the server in step 1 you will need to restart it.

Restart the server in one of the usual ways. Using the init script is usually the best way:

```
 $ sudo invoke-rc.d slapd start

```

That's all there is to it. Now you should have SLAPD running with a fresh copy of the database.

## Troubleshooting Rebuilds

As long as the LDIF file exported with `slapcat` is good, there is not much that can go wrong in this process. Even if you have to delete and recreate several times, as long as the LDIF file is safe, no important data is at risk.

If SLAPD is running as a user other than `root`, the main problem with importing is usually the permissions on the database files at `/var/lib/ldap`. Permissions on the configuration files in `/etc/ldap` directory may also be the source of SLAPD failures. Make sure they are owned by the appropriate user.

When switching versions of OpenLDAP, occasionally an old LDIF file will not be valid in the new server (this happened between OpenLDAP 2.0 and OpenLDAP 2.2, and again between 2.2 and 2.3; it could happen again in the future). While the standard schemas are fairly stable over time, operational attributes, which are not usually standardized, are more volatile, and do change from release to release.

Often, the fix will be tweaking records in the LDIF file to match the attributes used in new version. One other common issue has to do with starting up the server. Sometimes, when using the init script, you will not be able to get the server to start, but no informative message will be sent to the console or the log files. (One common reason for the failure to start is the permissions issue I noted earlier).

A good first step in solving startup problems is to run `slapd` from the command line, with debugging enabled: `sudo slapd -d trace`.

# Summary

In this appendix we looked at a couple of useful commands, including some designed to get detailed information about the directory server itself. Also, we saw two ways of making directory backups, and examined the process of rebuilding a directory database.