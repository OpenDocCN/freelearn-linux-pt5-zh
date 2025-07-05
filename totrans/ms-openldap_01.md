# Chapter 1. Directory Servers and LDAP

In this first chapter, we will cover the basics of LDAP. While most of the chapters in this book take a practical hands-on approach, this first chapter is higher-level and introductory in nature. We will get introduced to directory servers and LDAP, including commonly-used directory terminology. We will also see how the OpenLDAP server fits into the directory landscape, where it came from, and how it works. Here are the main topics covered in this chapter:

*   The basics of LDAP directories
*   The history of LDAP and the OpenLDAP server
*   A technical overview of the OpenLDAP server

# LDAP Basics

The term **LDAP** stands for **Lightweight Directory Access Protocol**. As the name indicates, LDAP was originally designed to be a network protocol that provided an alternative form of access to existing directory servers, but as the idea of LDAP—and the technologies surrounding it—matured, the term LDAP became synonymous with a specific type of directory architecture. We use the term LDAP when referring to directory services that comply with that architecture, as defined in the LDAP specifications.

### Note

LDAP is standardized. The body of LDAP standards, including the network protocols, the directory structure, and the services provided by an LDAP server, are all available in the form of RFCs (Requests For Comments). Throughout this book, I will reference specific LDAP RFCs as authoritative sources of information about LDAP.

The current version of LDAP is LDAP v.3 (version 3), a standard developed in 1997 as RFC 2251, and widely implemented throughout the industry. The original specification has recently (June 2006) been updated, and RFCs 4510-4519 provide a clarified and much more cohesive specification for LDAP.

While directories in general, and LDAP directories in particular, are by no means novel or rare in the information technology world, the driving technologies are certainly not as well understood as near relatives like the relational database. One of the goals of this chapter (and of this book in general) is to introduce and clarify the function and use of an LDAP directory.

In this section, we will introduce some of the concepts that are important for understanding LDAP. The best place to start is with the idea of the directory.

## What is a Directory?

When we think of a directory, we conjure images of telephone directories or address books. We use such directories to find information about individuals or organizations. For instance, I might thumb through my address book to find the phone number of my friend Jack, or skim through the telephone directory looking for the address of Acme Services.

A **directory server** is used this way, too. It maintains information about some set of entities (entities like people or organizations), and it provides services for accessing that information.

Of course, a directory server must also have means for adding, modifying, and deleting information, as well. But, even as a telephone directory is assumed to be primarily a resource for reading, a directory server's information is assumed to be read more often than written. This assumption about the use of a directory server is codified, or summarized, in the phrase "high-read, low-write". Consequently, many applications of LDAP technology are geared toward reading and searching for information.

### Note

While many directory servers have been optimized for fast reading at the expense of fast modification, this is not necessarily the case with OpenLDAP. OpenLDAP is efficient on both counts, and it can be used for applications that require frequent writing of data.

Some sorts of directory servers (envision a simple server-based implementation of an address book) simply provide a narrow and specific service. A single-purpose directory server, such as an online address book, might store only a very specific type of data, like phone numbers, addresses, and email information for a set of people. Such directories are not extensible. Instead, they are *single-purpose*.

But LDAP (and its X.500 predecessor) was designed to be a *general*-*purpose* directory server. It has not been designed with the purpose of capturing a specific type of data (like telephone numbers or email addresses). Instead, it was designed to give implementers the ability to define—clearly and carefully—what data the directory should store.

Such a generic directory server ought to be able to store many different kinds of information. For that matter, it should be able to store different kinds of information about different kinds of entities. For example, a general purpose directory should be able to store information about entities as diverse as people and igneous rock samples. But we don't want to store the same information about people as we do about rocks.

A person might have a surname, a phone number, and an email address, as shown in the following figure:

![What is a Directory?](img/1021_01_01.jpg)

A rock sample might have an identification number, information about its geographical origin, and a hardness classification.

LDAP makes it possible to define what a person's entry would look like, and what a rock's entry would look like. Its general architecture provides the capabilities needed for managing large amounts of diverse directory entries.

In the remainder of this section we will examine how information in an LDAP directory is structured. We will start by looking at the idea of a **directory entry**, with a **distinguished name** and **attributes**. Then, we will look at how entries are organized within the **directory information tree**. By the end of this section, you should understand the basic structure of information within an LDAP directory.

## The Structure of a Directory Entry

Let's continue with our comparison of a directory server and a phone book. A phone book contains a very specific type of information, organized in a very specific way, and designed to fulfil a very specific purpose. Here's an example phone book entry:

```
Acme Services
123 W. First St.
Chicago, IL 60616-1234
(773) 555-8943 or (800) 555 9834
```

As mentioned earlier, this sort of directory has specific information, organized in a specific way, designed to fulfill a specific purpose: it is information about how to contact a specific organization (Acme Services) organized in a familiar pattern (address and phone number). And it is designed so that a person, having a particular name in mind, can quickly scan through the directory (which is ordered alphabetically by organization name), and find the desired contact information.

But there are a few things to note about the phone book entry:

*   The data is arranged for searching by only one value: the name of the organization. If you should happen to have the phone number of the organization, but not the name, searching the phone book for the matching telephone number in order to ascertain the name would be a taxing, and probably futile task.
*   The format of the entry is sparse, and requires that the reader will be able to recognize the format and supply auxiliary information required for making sense of the data. One accustomed to reading phone book entries will be able to extrapolate from the previous entry, and identify the information this way:

    ```
    Organization Name: Acme Services
    Street Address: 123 West First Street
    City: Chicago
    State: Illinois
    Postal Code: 60616-1234
    Country: USA
    Phone Number: +1 773 555 8943
    Phone Number: +1 800 555 9834
    ```

In this example, the meaning of the information is made more explicit. Each value is preceded by a name that identifies the type of information given. Acme Services is now identified as the name of an organization. Information is also broken up into smaller chunks (city and state on separate lines), and some information which was implicit in the previous entry (such as the country) has been made explicit. And where two pieces of information (the two phone numbers) were initially compressed onto one line, they have now been separated, making the information more explicit.

This form of entry is closer to the way a record would look in an LDAP directory. But there is still another issue to address. How can we distinguish between two very similar records?

For example, say we have a telephone directory for the entire state of Illinois. And in Illinois, we have a company called Acme Services located in the city of Chicago, and another company named Acme Services located in the city of Springfield.

Simply knowing the company name alone is not sufficient information to isolate just one entry in the phone book. To do that, we would need some sort of unique name—a name that exists only once in the entire directory, and which can be used to refer to *one specific entry*.

## A Unique Name: The DN

One way of distinguishing between two very similar records is to create a unique name for each record in the directory. This is the strategy adopted by LDAP; each record in the directory has a **distinguished name**. The distinguished name is an important LDAP term; usually it is abbreviated as **DN**.

In an LDAP directory, the directory designer is the one who decides what components will make up a DN, but typically the DN reflects where the record is in the directory (a concept we will examine in the next part), as well as some information that distinguishes this record from other near records.

A DN then, is composed of a combination of directory information, and looks something like this:

```
dn: o=Acme Services, l=Chicago, st=Illinois, c=US
```

This single identifier is sufficient to pick it out from the Springfield company by the same name. The DN of the Springfield company named Acme Services would, according to the previous scheme, look something like this:

```
dn: o=Acme Services, l=Springfield, st=Illinois, c=US
```

As may be evident from this example, when defining what fields will compose a DN, it is necessary to make sure that these fields will be fine-grained enough to distinguish between two different entries. In other words, all it takes to break the DN syntax is for another Acme Services to appear in Chicago.

### Tip

**DNs are not case sensitive**

Some parts of LDAP records are case sensitive, and others are not. DNs, for example, are not case sensitive.

The DN is one important element in an LDAP entry. Next, we will take a closer look at the idea of an LDAP entry, and the components that make up an entry.

## An Example LDAP Entry

Let's take a specific look at what an **LDAP entry** looks like.

An LDAP **entry**, or **record**, is the directory unit that stores information about an individual item in the directory. Again, drawing on ideas found in other directories is useful: an entry in a telephone directory describes a specific unit of information in that directory. Likewise, a record in an LDAP directory contains information about a specific unit, though (since LDAP is generic) the exact target of that unit is unspecified. It might be a person, or a company, or a rock, or some virtual entity like a Java object.

### Note

Originally, the LDAP specification stated that an entry had to have a correlate in the real world. While this may have been the intention of early directory server developers, there is no reason why, in practice, a directory server entry must correlate with anything external to the directory—real or virtual.

An entry is composed of a DN and one or more **attributes**. The DN serves as a unique identifier within an LDAP directory information tree. Attributes provide information about that entry. Let's convert our previous telephone directory entry into an LDAP record:

```
dn: o=Acme Services, l=Chicago, st=Illinois, c=US
o: Acme Services
postalAddress: 123 West First Street
l: Chicago
st: Illinois
postalCode: 60616-1234
c: US
telephoneNumber: +1 773 555 8943
telephoneNumber: +1 800 555 9834
objectclass: organization
```

The first line is the DN. All other lines in this record represent attributes.

Note that the main difference between this example and the previous telephone directory examples we have examined is the names of each field in the entry; these are now compacted into a form that the directory can easily interpret.

### Tip

These attribute names, like `o` and `postalAddress`, refer to well-defined attribute definitions contained in an LDAP schema. They cannot be "invented" on the fly, or made up as you go. Creating new attributes requires writing a schema. Schemas are covered in Chapter 6 of this book.

An attribute describes a specific type of information. There are eight attributes here in our example, representing the following:

1.  Organization Name (`o`)
2.  Mailing address (`postalAddress`)
3.  Locality (`l`), which may be the name of a city, town, village, and so forth
4.  State or Province (`st`)
5.  Postal Code or ZIP Code (`postalCode`)
6.  Country (`c`)
7.  Telephone Number (`telephoneNumber`)
8.  Object Class (`objectclass`), which specifies what type (or types) of record this entry is

An attribute may have one or more attribute names, where these names are synonyms. For example `c` and `countryName` are both names for the attribute type that identify a country. Both identify the same information, and LDAP will treat the two names as describing the same type of information.

In any given record, an attribute may have one or more values (assuming the attribute's definition allows more than one value). The record above has only one attribute that contains more than one value. The `telephoneNumber` attribute has two values, each representing a different phone number.

Attributes are defined in *attribute definitions*, which will be discussed at length in Chapter 6\. These definitions provide information about the syntax and length of the information stored in values, all of the attribute names that apply to that attribute, whether or not the attribute can have multiple values, and so on. Records stored in LDAP directories must adhere to the attribute definitions.

For example, the attribute definition for a country name gives the following information:

*   The names `c` and `countryName` can refer to this object. The default name is `c`.
*   A country name is stored as a string.
*   When doing matches on the attribute values, case can be ignored.
*   Matching can be done on either the entire string (for example `Canada`) or using substrings (`Ca*`).
*   A country name cannot be longer than 32768 characters.
*   Only one country name is allowed per record.

All of this information is packed into a compact schema definition that the directory server reads when it starts.

Attribute names are not case sensitive. The attribute name `o` is treated as synonymous with the name `O`. Likewise, `GivenName`, `givenname`, and `givenName` are all evaluated as the same attribute name.

As for the values of attributes, case sensitivity depends on the attribute definition. For example, the values of DNs and `objectclass` attributes are *not* case sensitive, but a URI (`labeledURI`) attribute value *is* case sensitive.

### The Object Class Attribute

The last attribute in the given record is the `objectclass` attribute. This is a special attribute that provides information about what type of record (or entry) this is.

An object class determines what attributes may be given to a record. The object class, `organization`, indicates that this record describes an organization. According to this object class's definition, an `organization` record can contain a locality (`l`), and a postal code (`postalCode`), and all of the other attributes present in the record.

One of the fields, the organization name (`o`), is required for any entry with an `organization` object class.

The object class also allows several other attributes that are not present in our record, like `description` and `facsimileTelephoneNumber`.

Given the object class attribute, which is required for any entry, the directory can determine what attributes must, can, and cannot be present in the entry.

As with other attributes, the `objectclass` attribute may have multiple values, though which values may be given are subject to the **object class definition** and **schema definition**—the rules about what attributes belong to what object classes, and how these object classes can be combined.

### Note

An **LDAP Schema** consists of rules that define the types of records in a directory, and how those records might relate to each other. The main two items stored in a schema (though there are others) are attribute type definitions and object class definitions. Chapter 6 of this book is devoted to schemas.

While a record may have multiple object classes, one of these object classes must be the **structural object class** for the record. A structural object class determines what type of object the record is. We will talk about structural object classes later in the book.

The LDAP record then, is composed of a single DN, and one or more attributes (remember, `objectclass` is required). The attributes contain information about the entity that is identified by the DN.

An LDAP directory contains an aggregation of entries, arranged in one or more hierarchies in a tree structure.

### Operational Attributes

In addition to regular attributes, the directory server may also attach special **operational attributes** to an entry. Operational attributes are used by the directory server itself to store information about entries. Such attributes are not designed for use by end users (though on occasion they can be useful), and are usually not returned during LDAP searches.

At various points in this book, we will make use of operational attributes. But most of the time, when we talk about attributes, we are talking about regular attributes.

## The Directory Information Tree

So far, we have been comparing an LDAP directory to an address book or a telephone directory. But now I am going to introduce one of the primary differences between the structure of the data in an LDAP directory server, and that of many other forms of directories.

The information in a telephone directory is typically stored in one long alphabetical list. But in an LDAP directory the organizational structure is more sophisticated.

Information in an LDAP directory is organized into one or more hierarchies where, at the top of the hierarchy, there is a **base entry**, and other entries are organized in tree-like structures beneath the base entry. Each node on the hierarchy is an entry, with a DN and more than one attributes.

This hierarchically organized collection of entries is called a **directory information tree**, sometimes referred to simply as a directory tree or **DIT**.

To understand this method of organizing information, consider the organizational chart of a company.

The top of the hierarchy is the company itself. Beneath this, there are a number of departments and organizational units, and beneath these are the employees, contractors, and other individuals with a formal affiliation to the company. We can draw this as a hierarchy:

![The Directory Information Tree](img/1021_01_02.jpg)

LDAP directories store data in hierarchical relationships, too. At the top of the directory information tree is the root entry. Beneath that is a **subordinate entry**, which, in turn, may have its own subordinate entries. Each of these records has its own DN, and its own attributes.

### Tip

**A File System Analogy**

Most modern file systems represent data in hierarchies too. For example, the directory `/home` may have multiple subdirectories: `/home/mbutcher`, `/home/ikant`, `/home/dhume`. We can say that `/home` has three subordinates, but that each of those has one superior (the `/home` directory). When thinking about LDAP directory trees, it may help to compare them to the layout of a file system.

Adapting this to the previous example, we could easily create an LDAP directory information tree that represented the organizational chart:

![The Directory Information Tree](img/1021_01_03.jpg)

Note that the DN of each entry contains information about its **superior entry** (the record above it). In fact, a DN is composed of two parts: the first part is the **relative DN** (**RDN**), and contains one or more attributes from the entry. The second part is the *full* DN of the superior entry. We will look at this relationship further in Chapter 3.

When we create our directory in the next few chapters, we will create a tree-like structure of records.

You should now have a basic idea of how a directory is represented in a directory information tree. Records, consisting of a DN and some attributes, are organized in a hierarchy. At the top of the hierarchy is the base entry, and beneath that entries are organized into branches.

## What to Do with an LDAP Server

I've given a description of what an LDAP directory is, but it is also helpful to look at what an LDAP directory is used for. What is the function of an LDAP server? What problem is it intended to solve?

The first, and most obvious, answer is that LDAP is designed to provide a digital directory—an online presentation equivalent to a telephone directory or address book. Of course, there is some truth to this, and LDAP servers can indeed be used in this way. But so can relational databases and even more basic data structures.

We could expand on this answer, and point out that LDAP provides a robust layer of services—searching with complex filters, representing complex entities with attributes, allowing fine-grained access to data, and so on—that provide sophisticated directory services.

A more classical explanation, one rooted in the historical development of LDAP out of the X.500 directory, would be that LDAP is designed to represent organizations, including their structure, their physical assets, and their personnel. LDAP, by this account, isn't so much a fancy telephone directory as it is an enterprise management tool. In fact, this is one of the more common ways to use LDAP directories.

The most common use of an LDAP, a use based on a conception of LDAP as a narrow type of enterprise management tool, is as a central authority on network users, groups, and accounts. An LDAP directory stores information on each user account for the network—information like username, password, full name, and email address. Other services on the network, from workstations to email servers to web applications, can use LDAP as an authoritative source of user information. Applications can authenticate users against the directory. A single user account can be shared across multiple (perhaps all) enterprise applications.

Finally, there is a more generic, or abstract, view of the function of LDAP services. LDAP is nothing other than a special sort of database that organizes data into tree structures, like a file system hierarchy. This view is more easily seen by comparing an LDAP directory to a relational database (RDB) system.

Relational databases store information in tables, and tables are composed of records. Relationships, in RDBs, are established between records in different tables, and there are numerous forms of relationship: one to many, one to one, many to one, and so on. RDBs support reading and writing operations on data, typically implemented through some version of SQL (Standard Query Language), and they typically listen on network connections, making data available to other applications on the network.

Compared to an RDB, LDAP can also be seen as a storage system. Rather than presenting data in tabular structures, though, LDAP stores entries in a hierarchy (like a file system). The basic relationships in an LDAP consist of the superior-to-subordinate relation (one to many), and the subordinate-to-superior relation (one-to-one), though other relationships can be used.

### Tip

**Other Relationships in LDAP**

While the superior/subordinate relationships are the most commonly used, they are not the only ones supported. Relationships among arbitrary entries within the database are often modeled by linking DNs together using attributes. We will examine this use in detail when talking about groups in Chapter 4.

Reading and writing to the database are supported through LDAP operations with sophisticated filters and data structures like **LDIF** (**LDAP Data Interchange Format**). And LDAP directories, like their RDB counterparts, often listen on network sockets to provide services to other applications.

I have suggested some different views of the purpose of LDAP. Is any one of these the *correct* answer? No. Each of these uses of LDAP is legitimate, and LDAP directories can be used to address a broad range of problems.

# The History of LDAP and OpenLDAP

At first glance, the term LDAP seems misleading. When we talk, for instance, about the primary protocol for the web, HTTP (HyperText Transfer Protocol), we are talking about the way that web applications transfer information across the network. We are not talking about the format of the data that is moved across the network, nor are we talking about how that data is stored on or retrieved from the server.

But when we talk about LDAP, we are usually talking not only about the network protocol, but about a particular kind of server that stores data of a well-defined format inside of a special database. There is a historical reason for this seemingly misleading name.

Originally, LDAP was just a network protocol used to get data out of an X.500 directory (a directory server architecture, designed in the 1980s and standardized in 1988). This was the intent of Yeong, Howes, and Killie when they initially drafted the LDAP specification as RFC 1487 in 1993.

### Tip

**About RFCs**

RFCs (Requests for Comments) are a series of technical documents, usually specifying standards. Each RFC is identified by number, which are organized sequentially—earlier RFCs have lower numbers. There are many websites that make the RFC database, in whole or in part, available. One exemplary source is the RFC Editor ([http://www.rfc-editor.org](http://www.rfc-editor.org)), which is used in this book.

The first LDAP servers were gateways to X.500 directories, but these servers quickly evolved into full-fledged directory servers. Tim Howes and his colleagues at the University of Michigan created the Open Source *University of Michigan LDAP Implementation*, which became the reference implementation for other LDAP servers.

### Note

Historical information on the University of Michigan LDAP project is still available online: [http://www.umich.edu/~dirsvcs/ldap/ldap.html](http://www.umich.edu/~dirsvcs/ldap/ldap.html)

As the University of Michigan's LDAP server matured, a wealth of new standards was created. LDAP picked up industry momentum. Tim Howes was hired by Netscape, and LDAP went mainstream.

By the late 1990's, Netscape, Novell, Oracle, and Microsoft (among others) all touted LDAP offerings. RFC 2251, released in 1997, standardized LDAPv3, which made vast improvements to the earlier LDAP standards.

The market for LDAP servers matured, but the University of Michigan project lost momentum. Key developers had left the university to move along to other projects.

In 1998 the OpenLDAP project was started by Kurt Zeilenga. Soon after, Howard Chu (formerly of the University of Michigan, and the current architect of the project) joined. They rescued the University of Michigan's code base, beginning development anew. The result, OpenLDAP 2.0, was highly successful, and made its way into almost every major Linux distribution.

### Note

A complete list of OpenLDAP contributors, from the project's inception to the present, can be found at [http://www.openldap.org/project/](http://www.openldap.org/project/).

Since the late '90's, OpenLDAP has continued to mature, overseen by the OpenLDAP Foundation, and supported by contributions from industry sponsors. As of this writing, version 2.3 is the stable release, and version 2.4 is in the beta stages.

As was the intent with the University of Michigan LDAP server, OpenLDAP still adheres closely to the LDAP standards. In fact, Kurt Zeilenga is responsible for many of the updates made to the LDAP standards in June 2006.

But in addition to its high degree of standards compliance, OpenLDAP is also one of the fastest directory servers in the market, far outpacing offerings from other Open Source directory server implementations.

# A Technical Overview of OpenLDAP

This book is a practically oriented technical book. It is designed to help you get OpenLDAP up and running, and to help you integrate LDAP into your own applications.

We will now begin this transition from the high-level material presented earlier to a more practical examination of the OpenLDAP suite of packages. First, let's take a brief look at the technical structure of OpenLDAP.

The OpenLDAP suite can be broken up into four components:

*   Servers: Provide LDAP services
*   Clients: Manipulate LDAP data
*   Utilities: Support LDAP servers
*   Libraries: provide programming interfaces to LDAP

In the course of this book, we will look at all four of these categories. Here, we will just get an overview:

![A Technical Overview of OpenLDAP](img/1021_01_04.jpg)

This diagram explains how these four elements relate to each other.

## The Server

The main server in the LDAP suite is **SLAPD** (the **Stand-Alone LDAP Daemon**). This server provides access to one or more directory information trees. Clients connect to the server over the LDAP protocol, usually using a network-based connection (though SLAPD provides a UNIX socket listener, too).

A server can store directory data locally, or simply access (or proxy access) to external sources. Typically, it provides authentication and searching services, and may also support adding, removing, and modifying directory data. It provides fine-grained access control to the directory.

SLAPD is a major focus of this book, and we will discuss it in detail in the chapters to come.

## Clients

Clients access LDAP servers over the LDAP network protocol. They function by requesting that the server performs operations on their behalf. Typically, a client will first connect to the directory server, then bind (authenticate), and then perform zero or more other operations (searches, modifications, additions, deletions, and so on) before finally unbinding and disconnecting.

## Utilities

Unlike clients, utilities do not perform operations using the LDAP protocol. Instead, they manipulate data at a lower level, and without mediation by the server. They are used primarily to help maintain the server.

## Libraries

There are several OpenLDAP libraries that are shared between LDAP applications. The libraries provide LDAP functions to these applications. The clients, utilities, and servers all share access to some of these libraries.

Application Programming Interfaces (APIs) are provided to allow software developers to write their own LDAP-aware applications without having to re-write fundamental LDAP code.

While the APIs provided with OpenLDAP are written in C, the OpenLDAP project also provides two Java APIs. These Java libraries are not included in the OpenLDAP suite, and are not covered in this book. Both however, can be retrieved from the OpenLDAP website: [http://openldap.org](http://openldap.org).

As we move on through this book we will examine each of these components of the LDAP architecture in detail.

# Summary

In this chapter we have covered the basics of LDAP directories in general, and of the OpenLDAP server in particular. We have covered the history of LDAP, the important terminology, and some of the high-level technical aspects of OpenLDAP. Now we are ready to start applying this knowledge.

In the next chapter we will turn our attention toward the process of installing and configuring OpenLDAP.