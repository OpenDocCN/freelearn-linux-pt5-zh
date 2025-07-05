# Chapter 6. LDAP Schemas

The focus of this chapter will be LDAP Schemas. Schemas are the standard way of describing the structure of objects that may be stored inside the directory. The first few sections are designed to provide foundational knowledge of what schemas do and how they work—a foundation necessary for our work, later in this chapter, using and implementing schemas. But we will continue on from there to a number of more practical topics, including adding pre-defined schemas and defining our own custom schemas.

We will begin with a general examination of schemas. From there, we will look at schema hierarchies. Like the directory information tree itself, schemas are organized into hierarchies. Next, we will examine some of the basic schemas that are included with OpenLDAP. We will also look at two overlays that require their own schemas. Finally, we will create a custom schema consisting of a pair of new object classes, each with new attributes. The main topics we will discuss in this chapter are:

*   The basics of schema definitions
*   The three types of object classes
*   Using different schemas in OpenLDAP
*   Configuring the Accesslog and Password Policy Overlays
*   Obtaining and using an Object Identifier (OID)
*   Creating new schemas by hand

# Introduction to LDAP Schemas

We have already looked at a variety of attributes and object classes used in OpenLDAP. For example, we created entries for our users using the `person`, `organizationalPerson`, and `inetOrgPerson` object classes and, in so doing, we used attributes like `cn`, `sn`, `uid`, `mail`, and `userPassword`. We also created groups using the `groupOfNames` and `groupOfUniqueNames` object classes, paying special attention to the `member` and `uniqueMember` attributes. We even looked briefly (in Chapter 3) at object classes and attributes for describing documents and collections of documents (`document` and `documentCollection` respectively).

Each of these object classes and attributes has a strict definition. The definitions of attributes and object classes are bundled together into larger collections called schemas. OpenLDAP applications use these schemas to determine how records should be structured and where (in the hierarchical structure) each entry can be located.

## Why Do They Look So Complicated?

LDAP schemas have a bad reputation. They are viewed as complex, arcane, hyper-technical, and difficult to implement. The goal of this chapter is to overcome this perception.

It is understandable why this reputation persists though. I think there are a few aspects of LDAP schemas that are daunting to the neophyte.

First, LDAP schemas are based on generations of technical specifications coming out of the complex X.500 system. Because of this heritage, LDAP schemas make frequent use of equipment that is not particularly human-friendly, such as object identifier numbers that look like this: `1.3.6.1.4.1.1466.115.121.1.25`. However, a little bit of background knowledge can overcome this hurdle.

Second, the LDAP schema definition language is notably different from the sorts of definition languages (DDL) familiar to SQL developers. This is largely due to the different nature of the backend database. LDAP is not inherently tabular as relational databases are, while it does make frequent use of concepts like inheritance (a rarity in SQL DDL languages, though some do support the idea). Finally, while SQL DDL takes the form of a SQL command, LDAP schema definitions are purely descriptive.

But the LDAP schema language is actually quite compact and typically only two directives (`attributetype` and `objectclasstype`), each with a handful of arguments, are needed in order to create custom schemas. For this reason, the learning curve is short, and by the end of this chapter you should be able to comfortably create your own schemas.

Typically schemas are written in plain-text files and stored in a subdirectory of the OpenLDAP configuration folder. In Ubuntu these files are located at `/etc/ldap/schema`. If you built from source the schema files are located by default at `/usr/local/etc/openldap/schema`.

SLAPD does not automatically use all of the schemas in the schema directory. When SLAPD starts up, it loads only the schemas specified in the `slapd.conf` file.

### Note

There is an exception to this rule: certain vital LDAP schema components, like `objectclass`, are hard-coded into OpenLDAP, since they are fundamental to the operation of the server.

Usually schemas are included using the `include` directive. In Chapter 2 we included three schema files in our `slapd.conf` file. The include section, near the top of the file, looks like this:

```
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema
```

The first line imports the core schema, which contains the schemas of attributes and object classes necessary for standard LDAP use. The second imports a number of commonly used object classes and attributes, including those used for storing document information and DNS records. The `inetorgperson.schema` file includes the inetOrgPerson object class definition and its associated attribute definitions.

In the coming sections we will look at the format of these files, implementing some existing schemas, and finally creating our own schema.

## Schema Definitions

LDAP schemas are used to formally define attributes, object classes, and various rules for structuring the directory information tree. The term **schema** refers to a collection of (conceptually related) schema definitions. The `inetOrgPerson` schema, for example, contains the definition of the `inetOrgPerson` object class, as well as all of the extra (non-core) attributes that are allowed or required by the `inetOrgPerson` object class.

A **schema definition** is a special type of directive that provides information about how a particular entity in SLAPD is to be structured. There are four different types of schema definition that can be included in `slapd.conf` (or an included schema definition):

*   **Object class definitions**: This defines an object class, including its unique identifier, its name, and the attributes it may or must have.
*   **Attribute definitions**: This defines an attribute, including its unique identifier, its name or names, the rules for what types of content will be allowed as values, and how matching operations are performed.
*   **Object identifiers**: This attaches a string name to a unique identifier. It is primarily used to expedite creating schemas.
*   **DIT content rules**: This specifies rules for what additional (auxiliary) object classes an entry with a particular structural object class may have.

In addition to these four, there are other schema definitions that are not typically placed in a schema. Instead, most of these are generated by OpenLDAP code. Here is a brief description of what each does (for more information see RFC 4512, which defines the LDAP schema language):

*   **Matching rule definitions**: These define a rule used for matching operations. Searches may use matching rules (such as equality and substring matching) to find specific attribute values. For example, the `distinguishedNameMatch` matching rule (with unique identifier `2.5.13.1`) defines the matching rule for exactly matching DNs. This rule is used by attributes like `member` (for group membership) and `seeAlso`. Searching with this rule will return successful results only if an attribute's value matches the given DN. The matching rules for any attribute determine what indexes can be created for that attribute.
*   **Matching rule uses**: These map attributes to matching rules, and is usually created dynamically by SLAPD. Based on this definition, a client can tell which attributes a particular matching rule can be applied to. It can be used, for example, to find out all of the attribute values that support exact DN matches (the `distinguishedNameMatch` matching rule). The schema definition for a matching rule use (`matchingRuleUse`) contains a unique identifier, the matching rule name, and all of the attributes that this matching rule applies to.
*   **LDAP syntaxes**: These describe the syntax allowed for the content of an attribute value. The exact type and syntax of data for an attribute value can be specified when an attribute is defined. There are a number of supported syntaxes (`ldapSyntaxes`) defined for SLAPD, including a syntax for the DN structure, one for binary data, several for kinds of plain text data, and so on. We will talk about supported syntaxes more when we look at attribute definitions.
*   **Structure rules**: These define where in a directory information tree a given entry can be located. It is based on the structural object class of the entry. Structural object classes and the object class hierarchy are discussed later in the section *The* *Object* *Class* *Hierarchy* section.
*   **Name forms**: These specify what attributes may or must be used in the RDN portion of an entry's DN (based on the entry's structural object class).

SLAPD builds this part of the schema in code. For example, matching rule uses are generated based on what matching rules exist and what attributes implement those matching rules. Like the rest of the schema, matching rules, LDAP syntaxes, structure rules, and name forms can all be accessed over the LDAP protocol. See the *Retrieving* *Schemas* *from* *SLAPD* section for more information.

For the time being though, we will focus primarily on the four schema definitions that can be included in the `slapd.conf` file. In particular, we will focus on creating new object classes and attributes.

## Object Classes and Attributes

There are two different types of schema definition that we need in order to extend the types of information that our directory server will store:

*   **Attribute type definition**: An attribute type definition defines an attribute, including what attribute names it may have (for example, `cn` and `commonName`), what sort of values an attribute may contain (numbers, string, DNs, and so on), what rules to use when trying to match values, and whether it may have more than one value.

    Any given attribute may require that its value or values be composed of certain characters or data types. For example, the `description` attribute allows long strings of characters, which makes it possible to include a sentence or two of information as a value to a description field.

*   **Object class definition**: An object class definition specifies the name of the object class, what attributes it must have, what attributes it may have, and what kind of object it is.

We will look at each of these in turn. To start, let's take another look at one of the schemas introduced in Chapter 3\. Here is a graphical representation of the `person` object class:

![Object Classes and Attributes](img/1021_06_01.jpg)

The `person` object class has two required attributes (`cn` and `sn`) and four more attributes that are allowed, but not required: `userPassword`, `telephoneNumber`, `seeAlso`, and `description`.

A new record that is of object class `person` (and has no other object classes) might look like this:

```
dn: cn=Thomas Reid, dc=example,dc=com
objectclass: person
cn: Thomas Reid
sn: Reid
userPassword:: DSFSUYJKHGH=
telephoneNumber: 555-555-5555
seeAlso: uid=david,ou=users,dc=example,dc=com
description: A basic user.
```

This record contains all, and only, the attributes in the `person` object class. Attempting to add a different attribute type not mentioned in the schema would lead to an error. Similarly, trying to remove all values for the `cn` or `sn` attributes would also lead to an error since their presence is required.

But how does OpenLDAP know which attributes are required and which are allowed? This information is stored in the schema definition for the `person` object class.

## Object Class Definitions

The schema definition is stored in the `core.schema` (and also in `core.ldif`) file at `/etc/ldap/schema` (or `/usr/local/etc/openldap/schema` if you compiled from source). Have a look at this:

```
objectclass 
  ( 
   2.5.6.6
   NAME 'person'
   DESC 'RFC4519: a person'
   SUP top STRUCTURAL
   MUST ( sn $ cn )
   MAY ( userPassword $ telephoneNumber $ seeAlso $ description ) 
  )
```

This is a simple object class definition. It begins with a descriptor, `objectclass`, which tells the schema interpreter what type of definition is being made. The rest of the definition is enclosed in parentheses. Extra whitespace characters, including line breaks, are generally ignored (unless enclosed in a quoted string), but remember that

since `objectclass` is a directive in the `slapd.conf` file format, every line other than the first must start with a whitespace character.

The first field within the definition is the numeric identifier for the object class: `2.5.6.6`. This unique identifier is called an **Object IDentifier** (**OID**). Every schema definition has a unique OID that distinguishes that definition from any other definition in the world. Because this OID is supposed to be globally unique there is an official procedure for giving a definition a unique identifier. This will be described later in the chapter. For now it is sufficient to note that these OIDs must be universally unique.

Any LDAP application may refer to a definition by its OID. Object classes, attributes, matching rules, and many other LDAP entities have OIDs.

### Note

The Root DSE record is a good example of how LDAP clients can learn important information about a server's capabilities based on the OIDs that the server presents to the client. See [Appendix C](apc.html "Appendix C. Useful LDAP Commands") for an example.

The second field in the definition is the `NAME` field. While an OID is easily used by a computer, it is not so easily used by humans. So, in addition to an OID, server-unique names (in character strings) may be specified. The above object class only has one name: `person`. Multiple names can be given to a single object class, but usually a single one will suffice.

In a schema definition, the string names should always be enclosed in single quotation marks. In a list of string values, each value must be enclosed in single quotes and the entire list must be enclosed in parentheses. For example, if the person definition specified two names, `person` and `humanBeing`, the `NAME` field would look like this:

```
NAME ( 'person' 'humanBeing' )
```

Note also that spaces are not allowed in the values of the `NAME` field, so `'human` `being'` would be an illegal name.

### Note

In attribute definitions, it is more common to give attributes a long name and an abbreviated name. For example, `cn` and `commonName` are both names for the attribute with OID `2.5.4.3`.

Most of the time object classes and attributes are referred to by the values in the `NAME` field rather than by OID.

### Note

By convention, names that consist of multiple words are concatenated by capitalizing the first letter of each word after the first. For example, `commonName` is composed of two words: `common` and `name`. Only the second word is capitalized. Underscores, dashes, and other special characters are typically not used to concatenate words. Thus, you should not use names like `common_name` or `common-name`.

The `DESC` field is a brief description of what this schema definition is to be used for. In this case the description field refers to an RFC (RFC 4519) that gives a detailed explanation of the object class. Of course it is not necessary to create an RFC to formally define your schemas, though if you plan on distributing the schema widely writing an RFC is a good idea.

The next field, `SUP`, which is short for 'superior,' indicates what the parent object class of this object class is. The parent of the `person` object class is the object class called `top`. Object classes, like directory information trees, are organized in hierarchies. The `top` object class is at the top of the object class hierarchy. The `STRUCTURAL` keyword also pertains to how this schema definition fits into the schema hierarchy. We will discuss schema hierarchies in the next part.

The last two fields are less mysterious. They define which attributes a `person` object must (`MUST`) contain, and which attributes an object may (`MAY`) contain.

The syntax for the `MUST` and `MAY` fields is straightforward. Each description takes a list of attributes:

```
MAY ( userPassword $ telephoneNumber $ seeAlso $ description )
```

The list of attribute values (designated either by OID or by an attribute name) is enclosed in parentheses. Values are separated with the dollar sign (`$`). The example above indicates that the four values, `userPassword`, `telephoneNumber`, `seeAlso`, and `description`, are all attributes that a person object is allowed to have.

An attribute should be specified in only one of the two lists. There is no need to put an attribute in both a `MAY` and a `MUST` list.

Of course, the names can be replaced with OIDs instead. Thus, the following two lines are equivalent:

```
   MUST ( sn $ cn )
```

and

```
   MUST ( sn $ 2.5.4.3 )
```

The OID for the `cn` attribute is `2.5.4.3`, and either identifier will work.

There are a few fields that may be present in an object class definition but which are not present in the previous code. The first is the `OBSOLETE` keyword, which appears after the `DESC` field. This is used to designate an object class as obsolete but still (temporarily) supported.

The second is the extensions section, which is used for providing implementation-specific extensions to a schema. At the end of the schema one or more extensions may be specified. An extension is a keyword followed by a list enclosed in parentheses. By default, none of the schemas included in OpenLDAP's `schema/` directory have any extensions.

In summary then, an object class definition begins with the `objectclasstype` directive, and can contain the following fields:

*   A unique OID to identify this object class (example: `2.5.6.6`).
*   A `NAME` field with a unique name (`NAME` `'person'`).
*   A `DESC` field with a brief description of the purpose of the object class (`DESC` `'RFC4519:` `a` `person'`).
*   Optionally, it may contain an `OBSOLETE` tag if the class is obsolete and should not be used.
*   A `SUP` line, indicating what object class is the parent (superior) of this one. Also, this line should specify the type of object class (`STRUCTURAL`, `ABSTRACT`, or `AUXILLIARY`). Example: `SUP` `top` `STRUCTURAL`. Abstract classes do not have superiors. When defining an abstract class `SUP` can be omitted.
*   A `MUST` field with a list of attributes that must be specified for an instance of this object class. Example: `MUST` `(` `sn` `$` `cn` `)`.
*   A `MAY` field with a list of attributes that can optionally be added to records of this object class. Example: `MAY` `(` `userPassword` `$` `telephoneNumber` `$` `seeAlso` `$` `description` `)`.
*   One or more extensions.

Object class definitions are an important part of schemas and we will look back at these concepts several times in this chapter. After covering other definition types, we will take a detailed look at the object class hierarchy. As we do that the role of the `SUP` line will become clearer.

Further on we will look at some specific object classes and we will also write our own custom object class. But before we move on to those things we will look at the other schema definitions. Next, we will look at attribute definitions.

## Attribute Definitions

The `person` object class that we examined now can have six different attributes—the two necessary `sn` and `cn` attributes, and the optional `userPassword`, `telephoneNumber`, `seeAlso`, and `description` attributes. Just as the object class was defined in the schema, so each attribute is also defined. The syntax for attribute definitions is similar though the fields allowed in the definition are different and more numerous.

The schema definition for the `telephoneNumber` attribute is a good example of a basic attribute definition:

```
attributetype 
  ( 
   2.5.4.20
   NAME 'telephoneNumber'
   DESC 'RFC2256: Telephone Number'
   EQUALITY telephoneNumberMatch
   SUBSTR telephoneNumberSubstringsMatch
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.50{32} 
  )
```

The attribute definition begins with an `attributetype` directive. The rest of the definition is enclosed in parentheses.

The first field in the definition is the unique OID for this attribute. As with all OIDs, this identifier must be globally unique. The OID `2.5.4.20` should only be used to refer to a `telephoneNumber` attribute. Later in this chapter, in the section *Getting* *an* *OID*, we will discuss getting and using a base OID.

After the OID comes the `NAME` field that associates one or more names with the attribute.

### Note

The names given in the `NAME` field are usually called *attribute* *descriptions* (see the discussion of the search operation in Chapter 3). This term is confusing when talking about schema definitions because the attribute schema definition has a description field, and that field is not the attribute description.

It is not uncommon for attributes to have two names—a long name (such as `commonName` or `surname`) and an abbreviated name (`cn` or `sn` respectively). When an attribute has multiple names, the list of names should be enclosed in parentheses. As an example, consider the `NAME` field for the `fax` attribute:

```
attributetype 
  ( 
   2.5.4.23
 NAME ( 'facsimileTelephoneNumber' 'fax' )
   DESC 'RFC2256: Facsimile (Fax) Telephone Number'
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.22 
  )
```

Note the syntax of the highlighted line. Each name in the list of names is enclosed in single quotes (`'`) and the entire list is enclosed in parentheses.

### Note

SLAPD will refer to attributes by the first name. Thus, if you search for the `fax` attribute SLAPD will return the matching attributes as `facsimileTelephoneNumber` not as `fax`.

The `DESC` field provides a brief description of the purpose of the attribute. In the `telephoneNumber` attribute definition, the value of this field is `'RFC2256:` `Telephone` `Number'`, indicating that the attribute is defined in RFC 2256.

One important aspect of defining an attribute is specifying how an application should test two attribute values to see if they match. Do `TEST` and `test` match? In some cases we might want them to, while in others we might not. Does `t*st` match `test`? Again, in some cases, this is desirable while in others it is not.

We can determine, in the attribute definition, which matching rules should be used to test whether one value matches another. When we discussed the search operation in Chapter 3 we saw four different comparison operators that could be used in search filters:

*   The equality operator (`=`)
*   The approximation operator (`~=`)
*   The greater-than-or-equal-to operator (`>=`)
*   The less-than-or-equal-to operator: (`<=`)

In addition to these we looked at using regular expression characters, such as the asterisk (`*`), to match portions, or substrings, of an attribute value. The behavior of each of these is determined, to a large degree, by the matching rules in the attribute definition.

When the LDAP server processes a comparison (during operations like binding, comparing, and searching) it uses the schema to determine how to handle these comparisons. The schema specifies which matching rules should be used. There are three different sorts of matching rules that can be assigned in a schema:

*   The equality rule, EQUALITY
*   The ordering rule, ORDERING
*   The substring matching rule, SUBSTRING

An attribute schema may specify rules for one, two, or all three of these. The value of each can be either the OID or the name of a matching rule. In the `telephoneNumber` schema, `EQUALITY` and `SUBSTRING` are used:

```
EQUALITY telephoneNumberMatch
SUBSTR telephoneNumberSubstringsMatch
```

When an equality test for a telephone number is requested, such as the evaluation of the filter `(telephoneNumber=+1` `234` `567` `8901)`, the `telephoneNumberMatch` rule is used. Note that the plus sign (`+`) is part of the telephone number, not part of the operator. If the filter includes a wild-card match, such as `(telephoneNumber=+1` `234` `567*)`, then the `telephoneNumberSubstringsMatch` rule is used instead.

### Note

With no `ORDERING` rule defined, SLAPD will not process matching tests for `>=` or `<=` operators. Any comparison will return false.

How do these two matching rules perform? Let's look at an example. When we defined the user with UID `matt`, we assigned that user a telephone number. Here, we will search for that entry, requesting only the `telephoneNumber` attribute:

```
 $ ldapsearch -LL -U matt '(uid=matt)' telephoneNumber

```

And the search result is as follows:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=matt,ou=Users,dc=example,dc=com
telephoneNumber: +1 555 555 4321
```

The `telephoneNumber` attribute has the value `+1` `555` `555` `4321`. Now let's perform a search using the telephone number:

```
 $ ldapsearch -LL -U matt '(telephoneNumber=+1 555 555 4321)' uid \
 telephoneNumber

```

And the search result is as follows:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
telephoneNumber: +1 555 555 4321
```

As expected a search using the exact phone number returned a result. This looks no different from what we would expect a string matching rule to do. Using the special `telephoneNumberMatch` rule in the schema has some advantages though. When using this matching rule, SLAPD will ignore certain telephone number formatting characters. Here's an example using a substring search:

```
 $ ldapsearch -LL -U matt '(telephoneNumber=+1 555-555-43*)' uid \
 telephoneNumber

```

Here is the result:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
telephoneNumber: +1 555 555 4321
```

The filter in this example uses dashes (`-`) where the previous filter used spaces. Using the `telphoneNumberSubstringMatch` rule, SLAPD ignored the dashes. With the `telephoneNumberMatch` and `telephoneNumberSubstringMatch` rules, the numbers `+15555554321`, `+1` `555` `555` `4321`, `1-5-5-55554-3-2-1`, and `+1` `555-555-4321` are all treated as identical matches.

This illustrates the virtue of being able to specify matching rules in the schema. For attributes such as `cn`, `sn`, or `mail` (email address), we certainly wouldn't want dashes to be treated the same as white space characters. We wouldn't want `Dan` `Forth` to match `Danforth`. But it is certainly a desirable feature when matching phone numbers. The LDAP answer to this problem is to assign matching rules fitting to the type of information stored in the attribute.

### Note

Other attributes, like `homePhone`, `pagerTelephoneNumber`, and `mobileTelephoneNumber` (all defined in `cosine.schema`) all use the `telephoneNumberMatch` and `telelphoneNumberSubstringMatch` matching rules too. Since they all share the same format there is no need to assign each a different specialized matching rule.

### Note

**Matching Rules and Indexes**

Some backends, such as BDB and HDB, support indexes (using the `index` directive in `slapd.conf`). The index supported is determined by the matching rules defined for an attribute. For example, an attribute with an equality matching rule can have an equality (`eq`) index. Likewise, one with a substring matching rule supports `sub` indexes.

The last field in the `telephoneNumber` matching scheme is the `SYNTAX` field. This relates to the type and structure of the data stored in values for `telephoneNumber` attributes.

```
SYNTAX 1.3.6.1.4.1.1466.115.121.1.50{32}
```

The value of the `SYNTAX` parameter has two parts. The first is the OID (or the name) of the LDAP syntax, and the second, set off in curly braces (`{` and `}`) is the maximum length (usually the number of characters) for the field. The length specifier is optional, and the server is not obligated to enforce the maximum length.

The OID mentioned earlier, `1.3.6.1.4.1.1466.115.121.1.50`, is the telephone number syntax. This indicates that attribute values for instances of the `telephoneNumber` attribute are to contain the characters (integers, dashes, spaces, and so on) that a phone number would require. SLAPD will reject attempts to add phone numbers that contain letters and other special characters. Later in this chapter, in the *Creating* *a* *Schema* part, we will look at the list of common LDAP syntaxes that OpenLDAP supports.

As far as complexity goes the `telephoneNumber` attribute is about average. However, many attribute definitions are much shorter, taking advantage of the fields set in similar attributes. Thus, there are many attributes that, because they inherit most of their features from their superior (parent) attribute, have only an OID, a `NAME` field, and a `DESC` field. The schema definition for the ever popular `cn` attribute looks like this:

```
attributetype 
  ( 
   2.5.4.3
   NAME ( 'cn' 'commonName' )
   DESC 'RFC2256: common name(s) for which the entity is known by'
   SUP name 
  )
```

In this case, the `SUP` name field indicates that the `name` attribute is the parent of the `cn` attribute. Attributes, like object classes, can be organized hierarchically. A superior attribute is the parent or prototype for this attribute and certain properties, if left unspecified in the schema definition, are inherited from the superior. Syntax and matching rules, for example, can be inherited from a parent.

In the previous example no matching rules and no LDAP syntax were specified. Therefore, the `cn` attribute type inherits these values from its superior. The `name` attribute uses the `caseIgnoreMatch` `EQUALITY` matching rule and the `caseIgnoreSubstringMatch` `SUBSTR` rule, and uses the Directory String LDAP syntax (`1.3.6.1.4.1.1466.115.121.1.15`). A directory string is a UTF-8 encoded string intended to store text.

There are a handful of other fields that the previous examples do not make use of. These are `OBSOLETE`, `SINGLE-VALUE`, `COLLECTIVE`, `NO-USER-MODIFICATION`, `USAGE`, and the extension area. Let's briefly look at those.

The `OBSOLETE` flag, which usually appears after the `DESC` field, plays the same role in attribute definitions as it does in object class definitions. It labels an attribute obsolete. While obsolete attributes are still supported and can be used for records in the directory information tree, they are to be treated as deprecated, subject to removal in future versions of the schema or software. `OBSOLETE` takes no parameters.

The `SINGLE-VALUE` flag indicates that the defined attribute can only have one attribute value. Typically, an attribute can have an arbitrary number of values. But any attribute whose schema includes the `SINGLE-VALUE` flag can have no more than one. The domain component (`dc`) attribute that we looked at in Chapters 3 and 4 is an example of this. An object that has a `dc` attribute can only assign one value to that attribute. `SINGLE-VALUE` takes no parameters.

The `COLLECTIVE` flag indicates that this attribute is a collective attribute. Entries can be grouped, with collective attributes, into an **entry collection**.

Collectives are implemented in OpenLDAP via the `collect` overlay, which is not compiled or installed by default, though it can be found in the `servers/slapd/overlays` directory of the source code. The schemas necessary for collective support are also not included by default in the OpenLDAP distribution, and must be copied from another source (such as RFC 3671).

Here's a rough idea of how entry collections work:

1.  One record is the collection record, and must use the `collectiveAttributeSubentry` object class. This becomes the authority for that collective attribute. All other subordinate records then inherit the attribute (and its value) and the attribute is visible (though read-only) as an attribute of each of these records. For more information on collectives see RFC 3671 ([http://www.ietf.org/rfc/rfc3671.txt](http://www.ietf.org/rfc/rfc3671.txt)).
2.  The `NO-USER-MODIFICATION` flag is used to indicate that the attribute is an operational attribute (used by SLAPD or an overlay), and cannot be modified by an LDAP client. This is not usually used in user-defined schemas. Use it only when writing a custom overlay that will make use of its own operational attributes.
3.  The `USAGE` field provides SLAPD with information about what will use the attribute. There are four possible values. The first three, `directoryOperation`, `distributedOperation`, and `dSAOperation`, all indicate that SLAPD itself uses the attribute. The last, `userApplication`, is the default and it indicates that the attribute is primarily intended to be used by client applications. Since most schemas are intended for client application use, the default is usually what is desired and the `USAGE` field is rarely used.
4.  Finally, `attributetype` definitions can also use extensions, though there are no extensions used in the main schemas included in OpenLDAP. The syntax for extensions is the same for attribute types as it is for object class definitions.

In summary, an attribute schema definition begins with the `attributetype` directive which is followed by a schema definition enclosed in parentheses. The following fields are allowed in attribute definitions:

*   A unique OID number, which is required. Example: `2.5.4.15`.
*   A `NAME` field, with one or more names for the attribute. Example: `NAME` `'businessCategory'`.
*   A `DESC` field containing a description of the attribute type. Example: `DESC` `'RFC2256:` `business` `category'`.
*   A `DEPRECATED` tag, if the attribute is deprecated.
*   A `SUP` field with the name or OID of the superior attribute type. Example: `SUP` `postalAddress`.
*   An `EQUALITY` matching rule OID or name. Example: `EQUALITY` `caseIgnoreMatch`.
*   An `ORDERING` matching rule OID or name.
*   A `SUBSTR` matching rule OID or name. Example: `SUBSTR` `caseIgnoreSubstringsMatch`.
*   A `SYNTAX` field with an LDAP syntax OID, and an optional length. Example: `SYNTAX` `1.3.6.1.4.1.1466.115.121.1.15{128}`.
*   The `SINGLE-VALUE` flag, if the attribute can only have one value.
*   The `COLLECTIVE` flag, if the attribute is a collective attribute.
*   The `NO-USER-MODIFICATION` flag, if the attribute is an operational attribute that client applications should not be able to modify.
*   The `USAGE` field, together with one of the four keywords (`userApplication`, `directoryOperation`, `distributedOperation`, or `dSAOperation`) used to indicate what the attribute is to be used for.
*   Any extensions that the attribute definition requires.

At this point we have looked at both object class definitions and attribute definitions. When creating your own custom schemas it is most likely that these are the only two types of schema definitions that you will need to use.

We have discussed the basics of schemas and seen a few examples in the text. Later in this chapter we will look at some other specific examples. But if you want to take a look at more examples of attribute and object class schemas peruse the files in the schema directory for OpenLDAP (`/etc/ldap/schema` or `/usr/local/etc/openldap/schema`). The best place to start is with the `core.schema` schema, which defines the standard LDAPv3 schemas.

### Note

While reading `core.schema` you might notice that several very important object classes and attribute types are commented out. Why? Because they are included in the **system schema**, which is hard-coded into OpenLDAP. This schema is found in the OpenLDAP source code in `slapd/schema_prep.c`.

The `cosine.schema` file contains many other commonly used schemas and is also a good place to look. The `inetOrgPerson.schema` schema is a good example of what a user-defined schema file ought to look like. Or, for a shorter example of a user-defined schema, see `openldap.schema`.

While `attributetype` and `objectclass` are the two primary directives used in schema creation there are a few others which we will cover, albeit more briefly, in the next two sections.

## Object Identifier Definitions

The object identifier directive (`objectidentifier`) is an extension to the standard definition language. While it doesn't provide additional functionality to the schema language, it serves as a time-saving (and human-friendly) utility.

The `objectidentifier` directive is used to assign a string alias to an OID. When SLAPD processes OID fields for `attributetype`, `objectclasstype`, and `ditcontentrule` directives, if it encounters a string instead of an OID, it will check to see if this string is an alias to an OID and, if so, it will use the value of the OID. The `telephoneNumber` schema we examined in the last section provides a good example:

```
attributetype 
  ( 
   2.5.4.20
   NAME 'telephoneNumber'
   DESC 'RFC2256: Telephone Number'
   EQUALITY telephoneNumberMatch
   SUBSTR telephoneNumberSubstringsMatch
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.50{32} 
  )
```

Instead of using the OID for the telephone number equality and substring matching rules (`1.3.6.1.4.1.1466.115.121.1.50` and `1.3.6.1.4.1.1466.115.121.1.58`, respectively), the schema refers to the names of the matching rules: `telephoneNumberMatch` and `telephoneNumberSubstringMatch`. This later form is much easier for humans to read.

The `objectidentifier` directive makes it easy to define such aliases for OID numbers, in whole or in part. Here is a simple example of assigning a name to an OID:

```
objectidentifier exampleComDemo 1.3.6.1.4.1.8254.1021.3.1
```

Using a directive like this at the top of a schema makes it possible to refer to the OID using the name `exampleComDemo` later.

### Note

The given OID is valid and is registered to the author. If you are developing your own LDAP schemas, you should register your own OID (see the *Getting* *an* *OID* section). While you are free to use this OID when recreating these examples, do not use it to write your own extensions. Otherwise, there will be no way to ensure that such OIDs are globally unique which defeats the purpose of the OID.

For example, we could create a schema like this:

```
objectclass 
  (
   exampleComDemo
   NAME 'myPersonObjectClass'
   DESC 'My Person Object Class'
   SUP inetOrgPerson STRUCTURAL
  )
```

Note that instead of using the OID number for the object, we used the `exampleComDemo` alias. But, generally, we would not assign one alias per object class. It would be more convenient to alias a common root OID and then append just the last part of the OID number. For example:

```
objectidentifier exampleComOC 1.3.6.1.4.1.8254.1021.1

objectclass 
  (
   exampleComOC:1
   NAME 'myPersonObjectClass'
   DESC 'My Person Object Class'
   SUP inetOrgPerson STRUCTURAL
  )
```

In this example we used the `objectidentifier` directive to create an alias for the OID base that will be used for all of my object class definitions. Thus, when SLAPD encounters the name `exampleComOC`, it will expand it to `1.3.6.1.4.1.8254.1021.1`. The object class definition for `myPersonObjectClass` should have the OID `1.3.6.1.4.1.8254.1021.1.1` (note the extra `.1` at the end). Rather than writing out the entire number we use the `exampleComOC` alias and append a colon (`:`) and then the numeric suffix for the object class.

When SLAPD encounters `exampleComOC:1` it will expand it to `1.3.6.1.4.1.8254.1021.1.1`. Likewise, if I were to create a second object class with the desired OID `1.3.6.1.4.1.8254.1021.1.2`, I could use `exampleComOC:2` instead of typing out the entire long OID.

### Note

Using the `objectidentifier` attribute can not only save typing, but reduce typos in an area particularly prone to typos (and with typos particularly difficult to spot).

For more examples of the `objectidentifier` directive, see `openldap.schema` in the schema directory for OpenLDAP.

## DIT Content Rules

The last schema directive we will look at is the `ditcontentrule` directive which is used for creating **DIT Content Rules**.

### Note

*DIT* stands for Directory Information Tree. This abbreviation is a frequently used bit of LDAP parlance.

A DIT content rule identifies a particular structural object class, and indicates which auxiliary object classes are allowed (or not allowed) to be included in entries that use that object class.

For an example, let's use a few of the object classes introduced in Chapter 3\. In the *Anatomy* *of* *an* *LDIF* *File* section we created an entry representing a document. It implemented the `document` object class, whose schema (located in `cosine.schema`) looks like this:

```
objectclass 
  ( 
   0.9.2342.19200300.100.4.6 
   NAME 'document'
   SUP top 
   STRUCTURAL
   MUST documentIdentifier
   MAY ( commonName $ description $ seeAlso $ localityName $
         organizationName $ organizationalUnitName $
         documentTitle $ documentVersion $ documentAuthor $
         documentLocation $ documentPublisher )
  )
```

This is a structural object class. Also in Chapter 3, in the *Adding* *System* *Records* section we added the entry for `uid=authenticate,ou=System,dc=example,dc=com`. This entry implemented the `simpleSecurityObject` object class. Here is the schema for `simpleSecurityObject`:

```
objectclass 
  ( 
   0.9.2342.19200300.100.4.19 
   NAME 'simpleSecurityObject'
   DESC 'RFC1274: simple security object'
   SUP top 
   AUXILIARY
   MUST userPassword 
  )
```

This object class is an auxiliary object class, meaning that it can be added to entries that already have a structural object class, the result being that the attributes of the auxiliary object class are now available for that entry.

### Note

For more discussion on the different sorts of object classes and how they function, see the discussion in Chapter 3 and the section in this chapter called *The* *Object* *Class* *Hierarchy*.

According to default OpenLDAP settings, if we had an entry with the `document` structural object class, we could give this document a password (for binding to the directory) by adding `objectclass:` `simpleSecurityObject` to the record, and then adding a `userPassword` attribute. This would give us a record looking something like this:

```
dn: documentIdentifier=011,uid=david,ou=Users,dc=example,dc=com
documentIdentifier: 011
documentTitle: Treatise on Human Nature
userPassword:: c2VjcmV0
objectClass: document
objectClass: simpleSecurityObject
```

This entry is essentially a document that has the ability to log in! A client that used this record's DN and the correct password could log in as this document.

Perhaps there are cases where this is desirable, but for the sake of this example, let us suppose that this is a configuration that we do not want to allow.

Normally, decisions about which entries have which object classes are left to external applications. But what if we wanted to make sure that no application could give `document` a `userPassword` attribute?

The best method for solving this problem is to create a DIT content rule that disallows adding the `userPassword` attribute to any entry that has the `document` object class. This is done with the `ditcontentrule` directive:

```
ditcontentrule 
  ( 
   0.9.2342.19200300.100.4.6 
   NAME 'noPWForDocs'
   DESC 'Do not allow passwords for documents'
   NOT userPassword
  )
```

The form of the `ditcontentrule` directive should be familiar by now. Like the `objectclass` and `attributetype` directives, this directive encloses the DIT content rule definition inside of parentheses.

The first field is an OID. But unlike the other schema definitions, this OID is not the OID for this definition. Instead, it is the OID of the structural object class that we are targeting.

In this case, the OID `0.9.2342.19200300.100.4.6` is the OID for the `document` object class. You can verify this with a glance at the document schema listed a few pages back or by browsing the cosine schema.

The `NAME` field should contain a unique name used for referencing this rule. For the most part, the value of this field is used in reporting references to this rule in the log file, and in responses to the client.

The `DESC` field contains a short-text description of what the rule does.

The `NOT` field contains a list of OIDs or names of attributes that should be disallowed. The name `userPassword` comes from the `NAME` field in the `userPassword` attribute definition.

With this content rule in place, what will happen if we try to add a `userPassword` attribute to a document? Here is an example using `ldapmodify`:

```
$ ldapmodify -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: documentIdentifier=011,uid=dave,ou=users,dc=example,dc=com
changetype: modify
add: objectclass
objectclass: simpleSecurityObject
-
add: userPassword
userPassword: secret
modifying entry 
       "documentIdentifier=011,uid=dave,ou=users,dc=example,dc=com"
ldap_modify: Object class violation (65)
        additional info: content rule 'noPWForDocs' precluded 
        attribute 'userPassword'
```

The highlighted portion in this example is the attempted modification. We attempted to add the `simpleSecurityObject` object class and the `userPassword` attribute to the record. But the server responded with an **Object class violation** error, giving the following reason:

```
content rule 'noPWForDocs' precluded attribute 'userPassword'
```

Our custom DIT content rule did its job—it prevented the addition of the `userPassword` attribute to the `document` entry.

This DIT content rule we created above is a negative rule—it defines what attributes an entry *cannot* have. But `ditcontentrule` can also be used to create positive rules: rules that specify which attributes (or auxiliary object classes) are allowed.

For example, we could write a rule that says that every entry that is an `inetOrgPerson` must have a `userPassword`:

```
ditcontentrule 
  (
   2.16.840.1.113730.3.2.2
   NAME 'reqPassword'
   DESC 'Require userPassword for inetOrgPerson'
   MUST userPassword
  )
```

The OID used in this rule is the OID for the `inetOrgPerson` object class. The `MUST` field indicates that any entry with the structural object class `inetOrgPerson` must also have the `userPassword` attribute set.

Because of this rule, an attempt to add a new `inetOrgPerson` entry without a `userPassword` would result in an error similar to the one we looked at earlier:

```
$ ldapadd -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=Johann,ou=users,dc=example,dc=com
uid: johann
ou: users
cn: Johann Fichte
cn: Johann Gottlieb Fichte
sn: Fichte
givenName: Johann
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson

adding new entry "uid=Johann,ou=users,dc=example,dc=com"
ldap_add: Object class violation (65)
        additional info: content rule 'reqPassword' requires attribute 'userPassword'
```

The record being added (highlighted) is a valid `inetOrgPerson` entry, according to the `inetOrgPerson` object class definition. But, because of the DIT content rule, adding the record failed because there is no `userPassword` attribute value specified.

Now let's expand this rule to take advantage of the `AUX` field. The `AUX` field may be used to explicitly state which auxiliary classes can be combined with this structural object class.

In our newly revised DIT content rule we will make it so that only the `pkiUser` and the `labeledURIObject` auxiliary object classes can be added to an `inetOrgUser` record.

### Note

The `pkiUser` object class is an auxiliary object class designed to indicate that an entry is capable of performing **public key infrastructure** (**PKI**) secure transactions. It has one attribute, `userCertificate`, that contains the user's cryptographic certificate. See the Wikipedia page for a quick introduction to PKI: [http://en.wikipedia.org/wiki/Public_key_infrastructure](http://en.wikipedia.org/wiki/Public_key_infrastructure).

The `labeledURIObject` object class allows an additional attribute, `labeledURI`, which takes a URI (such as a URL) and a plain text description:

```
labeledURI: http://aleph-null.tv Home Page
```

The URI is separated from the label by a white space. So the URI is [http://aleph-null.tv](http://aleph-null.tv) and the label is `Home` `Page`. The `labeledURIObject` is defined in RFC 2079 ([http://www.ietf.org/rfc/rfc2079.txt](http://www.ietf.org/rfc/rfc2079.txt)).

Also, we will change the `NAME` and `DESC` elements to reflect the fact that our rule now does more than just require a `userPassword`. The DIT content rule now looks like this:

```
ditcontentrule 
  (
   2.16.840.1.113730.3.2.2
   NAME 'inetOrgPersonRules'
   DESC 'Restrictions for entries with inetOrgPerson object class'
   MUST userPassword
   AUX ( labeledURIObject $ pkiUser )
  )
```

Note the syntax of the `AUX` field. To list multiple values in a field it is necessary to enclose the list of values, separated by a dollar sign (`$`), inside of parentheses.

Using this DIT content rule, we can successfully add a URL (using the `labeledURIObject` auxiliary object class) to my record:

```
$ ldapmodify -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=matt,ou=users,dc=example,dc=com
changetype: modify
add: objectclass
objectclass: labeledURIObject
-
add: labeledURI
labeledURI: http://aleph-null.tv Home Page

modifying entry "uid=matt,ou=users,dc=example,dc=com"
```

The entry, highlighted above, was added successfully because the `labeledURIObject` (which allows the `labeledURI` attribute) is allowed by the content rule. But if I try to add a different auxiliary object class—one not explicitly allowed in the DIT content rule – the change request will be denied:

```
$ ldapmodify -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=matt,ou=users,dc=example,dc=com
changetype: modify
add: objectclass
objectclass: userSecurityInformation

modifying entry "uid=matt,ou=users,dc=example,dc=com"
ldap_modify: Object class violation (65)
        additional info: content rule 'inetOrgPersonRules' does not 
        allow class 'userSecurityInformation'
```

The DIT content rule prevented the addition of an auxiliary object class because this class is not specified in the `AUX` field of the rule.

Like the other definitions, the `ditcontentrule` directive also allows the `OBSOLETE` flag.

In summary, the `ditcontentrule` directive takes a definition of a DIT content rule enclosed in parentheses. The following fields are supported:

*   The OID of the structural object class to which this rule applies.
*   The `NAME` field, which provides a short name used to identify the rule.
*   The `DESC` field, which contains a description of the rule.
*   The `OBSOLETE` flag to mark this rule as obsolete.
*   The `AUX` field, which contains the names or OIDs of all auxiliary classes that the entries of this object class are allowed to implement.
*   The `MUST` field, which contains a list of all of the (not already mandatory) attributes that entries of this object class must have.
*   The `MAY` field, which lists all of the fields that a member of this object class may have. As of OpenLDAP 2.3.30, this is not exclusive. Attributes not in this list but allowed by object class schema definitions are still allowed. In other words, `MAY` does not impose any restrictions.
*   The `NOT` field, which contains a list of attributes that an entry of this object class cannot have. This cannot be applied to attributes that are required by the object classes schema definition.

Now we have looked at the four different schema definition directives allowed in the `slapd.conf` file (or included files). With this information you should be able to read through and understand any of the schemas defined in OpenLDAP.

Next we will take a quick look at how to get schema information out of a SLAPD server using the LDAP protocol.

## Retrieving the Schemas from SLAPD

When SLAPD loads the schemas, it stores them in a special part of the directory information tree, along with the Root DSE record; a special entry holds schema information. Having this information is useful for debugging, but more importantly it provides a way for client applications to find out about what types of objects and attributes may be stored in this directory server.

Obtaining the information from the directory is as easy as issuing an `ldapsearch` command.

The schema information is stored in a special record called the **subschema subentry**. You can access the subschema subentry using `ldapsearch`:

```
 $ ldapsearch -U matt -b 'cn=subschema' -s base +

```

### Note

Access to the `cn=subschema` record is governed by global ACLs (ACLs that appear before the database section). For example, to grant access to the subschema to users only, you can use a rule like this: `access` `to` `dn.exact="cn=subschema"` `by` `users` `read`.

This will retrieve the entire schema specification from the server including not only the attribute and object class definitions, but also definitions of matching rules, matching rule uses, structure rules, name forms, and LDAP syntaxes.

But, as with any other record in an LDAP server, we can use search filters to get just the values of specific attributes. For example, we can find out what all of the existing DIT content rules are:

```
$ ldapsearch -LL -U matt -b 'cn=subschema' -s base ditcontentrules
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: cn=Subschema
dITContentRules: ( 0.9.2342.19200300.100.4.6 NAME 'noPWForDocs' DESC
       'Do not allow passwords for documents' NOT userPassword )
dITContentRules: ( 2.16.840.1.113730.3.2.2 NAME 'inetOrgPersonRules'
      DESC 'Restrictions for inetOrgPerson object class.'
      AUX ( labeledURIObject $ pkiUser )
      MUST userPassword )
```

This search returns all of the DIT content rules currently included in the schema definitions for this server. Of course, the only two there are the ones we created in the last section.

The following schema-related attributes are included in the `cn=Subschema` record:

*   `ldapSyntaxes`: This attribute has one value for every LDAP syntax supported in the directory. Example: `ldapSyntaxes:` `(` `1.3.6.1.1.16.1` `DESC` `'UUID'` `)`.
*   `matchingRules`: This attribute has one value for every matching rule in the directory. Example: `matchingRules:` `(` `2.5.13.14` `NAME` `'integerMatch'` `SYNTAX` `1.3.6.1.4.1.1466.115.121.1.27` `)`.
*   `matchingRuleUse`: This attribute has one value for every matching rule use, which pairs matching rule OIDs with a list of all of the attributes that implement that matching rule. Example: `matchingRuleUse:` `(` `2.5.13.27` `NAME` `'generalizedTimeMatch'` `APPLIES` `(` `createTimestamp` `$` `modifyTimestamp` `)` `)`.
*   `attributeTypes`: This attribute has one value for every attribute definition in this directory. Example: `attributeTypes:` `(` `2.5.4.3` `NAME` `(` `'cn'` `'commonName'` `)` `DESC` `'RFC2256:` `common` `name(s)` `for` `which` `the` `entity` `is` `known` `by'` `SUP` `name` `)`.
*   `objectClasses`: This attribute contains one value for every object class definition. Example: `objectClasses:` `(` `2.5.6.2` `NAME` `'country'` `DESC` `'RFC2256:` `a` `country'` `SUP` `top` `STRUCTURAL` `MUST` `c` `MAY` `(` `searchGuide` `$` `description` `)` `)`.
*   `dITContentRules`: This attribute contains one value for every DIT content rule defined.

Other standard attributes, such as `cn`, `objectclass`, and the basic operational attributes, are also part of the record.

Examining schemas this way is an alternative to simply reading the schema files. While it has less documentation (since there are no comments), using filters can be helpful. Also, information not in the standard schemas (such as schema definitions for operational attributes) is also available in this record.

Later in the chapter we will begin implementing schemas in SLAPD, first by including some already written schemas, then by writing our own. But next we will take a quick look at one more theoretical component of schemas: the schema hierarchy.

# The ObjectClass Hierarchy

Object classes and attributes in LDAP can be organized into hierarchical relationships. A hierarchical relationship is one in which one entity stands in a parent or superior relationship to one or more subordinate entities.

Attribute hierarchies tend to be simple, and require only a short explanation. Object classes, on the other hand, use a more complicated hierarchical model and will be the focus of this part of the chapter.

In the cases of both attribute and object class hierarchies, the mechanism for creating the hierarchy is the schema definition. Schema definitions for both attributes and object classes use the `SUP` field to indicate a relationship to a parent, or superior.

We will start out with a brief discussion of attribute hierarchies, and then move on to the more complicated object class hierarchies.

## Attribute Hierarchies

Attribute hierarchies are simple relationships wherein one attribute can, through its subordinate relationship to another attribute, inherit certain features, such as matching rules and LDAP syntaxes.

The simplicity of attribute hierarchies manifests itself in a few ways:

*   There is no requirement that an attribute have any relation to other attributes. In other words, there is no requirement that attributes be part of a hierarchy. Many, such as the `telephoneNumber` attribute we looked at in the previous part, stand on their own.
*   Attribute hierarchies do not play a significant role in how attributes are used. Attribute hierarchies exist primarily to keep attribute schema definitions clean and succinct, minimizing repetition.

The `name` attribute, which is conventionally not used directly in any object class, is a good example of the use of superior/subordinate relationships in attribute definitions. Thirteen attributes in the core schema list `name` as their superior. The `cn` attribute is one example.

![Attribute Hierarchies](img/1021_06_02.jpg)

The schema definition for `cn` uses only the `NAME`, `DESC`, and `SUP` fields, with `SUP` indicating that the `name` attribute is the superior of `cn`.

Since the `cn` attribute definition does not specify any matching rules or an LDAP syntax, these are inherited from the name attribute. Hence, `cn` is assigned the equality and substring matching rules defined in `name`, as well as the LDAP syntax and length.

But there is not much more that can be done with attribute hierarchies. Other than matching rules and syntax nothing else is automatically inherited from the superior, and there are no other benefits in using attribute hierarchies.

### Subordinate Attributes and Searching

There is one interesting effect that results from attribute hierarchies. A request for a superior attribute may return subordinate attributes as matches. For example, here is a search requesting just a single attribute: `name`:

```
$ ldapsearch -LL -U matt '(uid=matt)' name
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=matt,ou=Users,dc=example,dc=com
ou: Users
cn: Matt Butcher
sn: Butcher
givenName: Matt
givenName: Matthew
title: Systems Integrator
st: Illinois
l: Chicago

```

According to the search parameters, the search should return any `name` attribute values for records with `uid=matt`. But the record returned (highlighted) has more than that. In addition to the DN, which is always returned, the record also has `ou`, `cn`, `sn`, `givenName`, `title`, `st`, and `l` values.

Why is this? This happens simply because all of those attribute types have `name` as the superior.

Such behavior extends to search filter behavior, as well. For example, a search filter like (`name=Marcus`) will result in a search being performed against all attributes that use `name` as a superior:

```
$ ldapsearch -LL -U matt '(name=Marcus)'
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=cicero,ou=Users,dc=example,dc=com
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

The record for `uid=cicero` matched because the `givenName` field has the value `Marcus`. As can be seen in the attribute type definition, the `givenName` attribute has `name` as a superior attribute type:

```
attributetype ( 2.5.4.42 NAME ( 'givenName' 'gn' )
    DESC 'RFC2256: first name(s) for which the entity is known by'
    SUP name 
```

While this feature can cause some unexpected behavior for those unfamiliar with schemas, it can prove quite useful at times.

For the most part, attribute hierarchies are fairly simple. Object class hierarchies are more complex though. And we will now take a look at them.

## Object Class Types: Abstract, Structural, and Auxiliary

Like attributes, object classes can be organized into hierarchies. Typically, there is one major object class hierarchy. But while the hierarchical organization of object classes plays an important part in the structure of the directory, not all object classes are part of the hierarchy. To understand why this is we must begin by examining the different types of object classes.

There are three types of object class: abstract, structural, and auxiliary as follows:

*   An **abstract object class** holds a place at the top of the object class hierarchy. It may set required and allowed attributes for all object classes beneath it in the hierarchy, but no record can be an instance of that object class only. Further, any parent of an abstract object class must also be abstract.
*   A **structural object class** also holds a place in the hierarchy, and is a subordinate of (or, to phrase it differently, inherits from) either another structural object class or an abstract object class. An entry in the directory is an instance of a structural object class. When one structural object class subclasses another structural object class, the parent class is treated as if it were abstract. So, operationally speaking, for any given record it has only one structural object class—the structural object class is lowest on the object class hierarchy.
*   An **auxiliary object class** is not required to be part of the object class hierarchy, though it can be. An auxiliary object class is intended to allow extra attributes to be defined for a record that already has a structural object class. For example, a record that describes a system account may not be in the person part of the hierarchy but may still need a password. The `simpleSecurityObject` is an auxiliary object class that can be added to other structural object classes to allow (and, in fact, require) that a `userPassword` attribute be set.

Abstract and structural object classes are organized into a hierarchy, with abstract classes at the top, and structural object classes as subordinates of those. In core schema (`core.schema`), there is only one abstract object class: `top`. This object class marks the top of the object class hierarchy—the ancestor (the highest superior) of all object classes.

### The Object Class Hierarchy: An Overview

The hierarchy begins with the abstract object class `top`. Beneath it are any number of structural object classes, all of which are either direct or indirect subordinates. A direct subordinate is one that lists `top` as its superior object class (in the `SUP` field of the schema definition). An indirect subordinate is farther down the object class hierarchy. It lists another abstract or structural object class as its superior, but that superior either itself refers to `top` as its superior, or refers to another indirect subordinate.

### Note

In other LDAP references, a superior class is sometimes called a **superclass**, while a superior attribute (in the attribute hierarchy) is called a **supertype**. Likewise, the terms **subclass** and **subtype** can be used to indicate the subordinate relationships in classes and attributes.

Structural object classes have either an abstract object class or another structural object class as their superior.

Auxiliary object classes may or may not be in the object class hierarchy. They can have superiors, but they are not required to.

Here is a pictorial representation of a simple object class hierarchy (consisting of four object classes) and a pair of records in the directory information tree that we created in Chapter 3:

![The Object Class Hierarchy: An Overview](img/1021_06_03.jpg)

The `account` and `groupOfNames` structural object classes both have `top` listed as their superior (as indicated by the solid lines). `simpleSecurityObject`, an auxiliary object class, has no superiors.

Beneath the object class hierarchy are two records, with the DN and object class attributes displayed. The dotted lines indicate which schemas these entries implement. Each of the two records (the `uid=authenticate` user and the `cn=Admins` group) are related to a different part of the object class hierarchy. `cn=Admins` is a `groupOfNames`, while `uid=authenticate` is an account that also has the attributes of a `simpleSecurityObject`.

This representation of the object class hierarchy is designed to show how the organization of schemas is related to the entries within the directory.

It is important to keep in mind that there are two different hierarchies in play here. The two entries above are part of the directory information tree hierarchy. Their position in that hierarchy is indicated by their DNs. The `uid=authenticate` entry, for example, is a child of the `ou=System` entry, which in turn is a child of the `dc=example,dc=com` entry (the root entry for our directory information tree).

But by their object classes, the entries can also be related to the object class hierarchy, as is illustrated. For the time being it is only this second hierarchy—the object class hierarchy—that we are interested in.

Let's take a look at each of the three types of object classes. Understanding the differences between the three, and the respective role each plays, will illuminate the concepts at play in the object class hierarchy.

### Abstract Classes

The first of the three types that we will examine is the **abstract class**. While abstract classes are only rarely used, they play a major role in the development of the object class hierarchy.

We have already talked about the special `top` object class. The most commonly used LDAP schemas do not use any other abstract object classes beside `top`. The `top` object class definition looks like this:

```
objectclass 
  ( 
   2.5.6.0
   NAME 'top'
   DESC 'RFC2256: top of the superclass chain'
   ABSTRACT
   MUST objectClass
  )
```

It requires only one attribute: `objectclass`. All structural object classes should be related, either directly or indirectly, to `top`. And any abstract object class that will have structural object classes subordinate to it must also be related to `top`. While it is possible to create an abstract class without a superior class, effectively starting a new tree of object classes, this is rarely done.

### Tip

**Abstracts without Superiors**

The main circumstance for defining abstract classes without superiors is when all of the classes that inherit from that abstract class will be auxiliary object classes. Structural object classes, according to RFC 4512, must be related (directly or indirectly) to the top object class.

But `top` is not the only abstract object class in frequent use. There are a few common schemas included with OpenLDAP, notably the `java.schema` and `corba.schema`, which make use of abstract object classes whose superiors are `top`. If an abstract object class has a superior, it must be an abstract superior.

Abstract object classes can have lists of attributes in the `MUST` and `MAY` fields of their definition. The `top` object class, as we just saw, requires the `objectclass` attribute. Any entry that implements a structural object class subordinate to this abstract object class inherits the `MUST` and `MAY` constraints of the parent.

For example, in the `java.schema` the class `javaObject` is abstract. Here is its definition:

```
objectclass 
  ( 
   1.3.6.1.4.1.42.2.27.4.2.4
   NAME 'javaObject'
   DESC 'Java object representation'
   SUP top
   ABSTRACT
   MUST javaClassName
   MAY ( javaClassNames $ javaCodebase $
         javaDoc $ description ) 
  )
```

According to the `SUP` field, this object class is subordinate to `top`. It requires that any entry that implements `javaObject` has a `javaClassName` attribute. It also defines several attributes—`javaClassNames`, `javaCodebase`, `javaDoc`, and `description`—that entries may include.

### Note

The Java schema is used to store serialized Java objects in a directory server. It is defined in RFC 2713.

There are no structural object classes subordinate to `javaObject`. However, there are a couple of auxiliary object classes that are subordinate to `javaObject`: `javaSerializedObject` and `javaMarshalledObject`. Here is the `javaSerializedObject` schema definition:

```
objectclass 
  ( 
   1.3.6.1.4.1.42.2.27.4.2.5
   NAME 'javaSerializedObject'
   DESC 'Java serialized object'
   SUP javaObject
   AUXILIARY
   MUST javaSerializedData 
  )
```

Only one attribute is required in this class: `javaSerializedData`. There are no optional attributes specified in this definition.

If some entry uses the `javaSerializedData` object class what fields *must* it have? And what fields *may* it have?

It must have a `javaSerializedData` attribute. We can see that from the `javaSerializedObject` schema. But it also must have the `javaClassName` attribute because that is required in the object class of the superior `javaObject` object class. And a `javaSerializedData` entry may have any of the attributes listed in the `MAY` field of the `javaObject` schema: `javaClassNames`, `javaCodebase`, `javaDoc`, and `description`.

This example illustrates the use of the abstract object class as a way of organizing object classes into hierarchies, grouping similar object classes (here, `javaSerializedObject` and `javaMarshalledObject`) under a common (and more generic) ancestor, `javaObject`. The `javaObject` abstract object class is then used to specify the common attributes that both of the subordinate object classes need included.

Thus, one of the major uses of abstract object classes is to collect common attributes that should be (or may be) included in object classes that are defined as subordinate to it.

Abstract classes are rare. In contrast, the most commonly used object class type is the structural object class.

### Structural Object Classes

As we have seen in a number of examples, every record has a DN and one or more object classes. From there, what other attributes it has depends on the object classes. But there are constraints on which object classes an entry has. One major factor that determines what object classes an entry may have is the structural object class hierarchy.

Every record in the directory must have at least one structural object class. The structural object class determines what type of entry a record is. For example, an entry with the structural object class `organization` is an `organization` entry.

### Note

Once an entry has been created in the directory, its structural object class cannot be changed. Adding and removing auxiliary object classes is allowed, but the structural object class is unalterable (as is, *ipso* *facto*, the chain of superior object classes).

An entry may implement more than one object class, and not all the object classes that it implements need be structural. Let's take a look at the organization record we created in Chapter 3:

```
dn: dc=example,dc=com
description: Example.Com, your trusted non-existent corporation.
dc: example
o: Example.Com
objectClass: top
objectClass: dcObject
objectClass: organization
```

There are three object classes for this entry:

*   `top`—an abstract object class
*   `dcObject`—an auxiliary object class
*   `organization`—a structural object class

### Note

The `top` object class is not strictly necessary in this entry. SLAPD implicitly includes `top` in all entries, since all structural object classes derive from it.

How do we know which object classes are of which type? The schema definitions for these object classes are the primary source of such information.

The structural object class locates this entry in the hierarchy of object classes, a hierarchy composed of abstract and structural object classes.

An entry may have more than one structural object class as long as they are all related by superior/subordinate relationships.

In the case where there are multiple structural object classes in one record, the most subordinate object class (the one farthest from the root object class, `top`) will have all of the rest of the structural object classes as ancestors. That is, for the object class farthest from `top` in the object class hierarchy, all other structural object classes must be superior to it. This lowest object class is then treated as the structural object class.

For example, in Chapter 3 we created a record for the user `barbara`:

```
dn: uid=barbara,ou=Users,dc=example,dc=com
ou: Users
uid: barbara
sn: Jensen
cn: Barbara Jensen
givenName: Barbara
displayName: Barbara Jensen
mail: barbara@example.com
userPassword: secret
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

This user belongs to four object classes. Three of these were explicitly stated earlier: `person`, `organizationalPerson`, and `inetOrgPerson`. All three of these happen to be structural object classes. The fourth, the `top` object class, is implicitly included.

All four of these object classes are related in the hierarchy. The `top` abstract object class sits at the top of the object class hierarchy. The `person` object class is directly subordinate to top. That is, the `person` object class definition lists `top` as its parent:

```
 objectclass 
  ( 
   2.5.6.6
   NAME 'person'
   DESC 'RFC2256: a person'
 SUP top 
   STRUCTURAL
   MUST ( sn $ cn )
   MAY ( userPassword $ telephoneNumber $ seeAlso $ description ) 
  )
```

While `person` points to `top` as its superior, `organizationalPerson` points to `person`. And `inetOrgPerson` points to `organizationalPerson` as its superior. Thus, we get a hierarchy of object classes:

![abstract object classworking ofStructural Object Classes](img/1021_06_04.jpg)

So, according to this hierarchy, any entry that is an `inetOrgPerson` must also abide by definitions of all of its superiors: `organizationalPerson`, `person`, and `top`. Any required attributes of any of those object classes will be required for an `inetOrgPerson` entry, and any optional attributes for any of those classes is optional for an `inetOrgPerson` entry.

Thus, the required fields for `inetOrgPerson` are `sn` and `cn`, both of which it gets from the `person` object class, and the `objectclass` attribute, which it inherits from `top`.

### Note

For a complete list of fields required by and allowed by `inetOrgPerson`, see the subsection *Adding* *User* *Records* in Chapter 3.

In the previous figure, the `pilotPerson` object class is also included, which represents another branch of the hierarchy. Like `organizationalPerson` and `inetOrgPerson`, `pilotPerson` describes a person within an organization, but it includes a number of attributes not present in `organizationalPerson` and `inetOrgPerson`, including the `favouriteDrink` and `janetMailBox` attributes.

While `pilotPerson` is not officially obsolete, it is not usually used; `inetOrgPerson` is typically used instead. But like `organizationalPerson`, `pilotPerson` lists `person` as its superior. Thus, it inherits the attributes of `person` and `top`. However, it is not related, directly or indirectly, to `organizationalPerson` or `inetOrgPerson`, and thus inherits none of their attributes.

Because `pilotPerson` is not related to `organizationalPerson` or `inetOrgPerson`, and because all of these are structural object classes, SLAPD will not allow any record to implement the `pilotPerson` object class and `organizationalPerson` or its subordinates. For example, if we try to add a record with all four of the person-describing object classes, we will get an error:

```
$ ldapadd -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=charles,ou=users,dc=example,dc=com
uid: charles
ou: users
cn: Charles Sanders Peirce
sn: Peirce
gn: Charles
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
objectclass: pilotPerson

adding new entry "uid=charles,ou=users,dc=example,dc=com"
ldap_add: Object class violation (65)
        additional info: invalid structural object class chain 
        (inetOrgPerson/pilotPerson)
```

When the client requests that the record above be added, SLAPD responds with an **Object class violation** error, indicating that the chain of object classes is not correct. This is because `pilotPerson` is not related to `organizationalPerson` or `inetOrgPerson`.

Returning to our record for `uid=barbara`, that entry lists three object classes:

```
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

As we saw in the previous figure, `inetOrgPerson` is the lowest object class on the hierarchy—the far most from `top`. That means that SLAPD considers this object class to be the structural object class for the record. It even sets a special operational attribute, `structuralObjectClass`, that stores this value. Thus, you can get information on the structural object class through `ldapsearch`:

```
 $ ldapsearch -LL -U matt '(uid=barbara)' structuralObjectClass
```

Here is the information:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: uid=barbara,ou=Users,dc=example,dc=com
structuralObjectClass: inetOrgPerson
```

When handling operations and evaluating rules, such as DIT content rules, SLAPD will treat this record as an `inetOrgPerson` record.

Within this discussion of structural object classes we have covered the gist of the object class hierarchy. An entry's place in the hierarchy is determined by its structural object class. But not all object classes affect a record's placement in the object class hierarchy. Let's turn to the third type of object class: auxiliary object classes.

### Auxiliary Object Classes

Auxiliary object classes provide a mechanism for adding one or more attributes to an entry with an existing structural object class. Think of it as a modular system for defining a collection of related attributes that can be attached to otherwise (conceptually) unrelated object classes.

To get an idea of how this works let's take another look at the `uid=authenticate` entry:

```
dn: uid=authenticate,ou=System,dc=example,dc=com
uid: authenticate
ou: System
description: Special account for authenticating users
userPassword:: c2VjcmV0
objectClass: account
objectClass: simpleSecurityObject
```

The structural object class for this entry is `account`. The `simpleSecurityObject` object class is auxiliary.

The `account` schema, found in `cosine.schema`, looks like this:

```
objectclass 
  ( 
   0.9.2342.19200300.100.4.5 
   NAME 'account'
   SUP top STRUCTURAL
   MUST userid
   MAY ( description $ seeAlso $ localityName $
         organizationName $ organizationalUnitName $ host )
  )
```

This entry, according to the COSINE standard (RFC 4524), is to define a system account on a computer.

For whatever reason, the creators of the standard did not include the attributes necessary to give the account a password. This makes sense. It is probably not typical that a system account would need to authenticate against LDAP. However, the system account we have created needs to perform directory operations and so we need this account to have a `userPassword` attribute.

One way to achieve this is to create a new structural object class subordinate to account, but which requires a `userPassword` attribute. But there is also an object class in `core.schema` designed specifically for the purpose of giving non-person entries in the directory a `userPassword` to allow them to bind. In other words, there is an existing object class that provides exactly the functionality we require: the `simpleSecurityObject` object class.

### Note

The `simpleSecurityObject` is also defined in the COSINE schema.

The `simpleSecurityObject` schema looks like this:

```
objectclass 
  ( 
   0.9.2342.19200300.100.4.19 
   NAME 'simpleSecurityObject'
   DESC 'RFC1274: simple security object'
   SUP top 
   AUXILIARY
   MUST userPassword 
  )

```

This schema definition adds one required attribute to any implementing entry, `userPassword`. Effectively then, the `simpleSecurityObject` object class can be added to an entry in order to allow it to bind to the directory (assuming the ACLs allow).

Given the combination of the structural object class, `account`, and the auxiliary object class, `simpleSecurityObject`, our `uid=authenticate` record now has three required fields:

*   `objectclass`, inherited from `top`
*   `uid`, from the `account` structural object class
*   `userPassword`, from the `simpleSecurityObject` auxiliary object class.

This example illustrates how the auxiliary object class can be used to add additional attributes to an entry that already belongs to a structural object class.

Rather than creating new structural object classes for each set of attributes you want an entry to have, the auxiliary object class mechanism makes it possible to define a modular collection of add-on attributes that can be attached to entries as needed.

By default, any auxiliary object class can be added to a record regardless of the structural object class of that record.

In other words, by default it is legal to take an entry with a `person` structural object class (an entry obviously intended to represent a human being) and attach to it the `javaSerializedObject` auxiliary object class (an entry intended to describe a stored representation of a Java binary class).

Historically, the responsibility for judiciously choosing which auxiliary object classes ought to be added to an entry has been left to LDAP client applications and users. However, you can use DIT content rules (see the previous part of this chapter) to formalize which auxiliary object classes an entry of a given structural object class is allowed to have.

## Moving Onward

Up to this point, this chapter has focused on the details of the LDAP schema system, and has focused as much on theoretical points as it has on practice.

In these pages, I have tried to provide a condensed explanation of LDAP schemas, focusing on the aspects most applicable to the goals of this book. This material should provide the necessary background knowledge for reading schema definitions, wisely selecting which schemas to use for your own directory needs, and writing custom schemas.

However, if you intend to work on the OpenLDAP code, write overlays or modules, or even write schemas intended for public standardization, you ought to read the LDAP RFCs, particularly RFC 4512, which defines the LDAP schema language.

Now we are ready to move on to more practical matters. In the next section, we will implement a few overlays that require extra schemas. As we configure those overlays we will examine the schemas and the role those schemas play in the functioning of the overlay.

After that, we will create our own short schema.

# Schemas: Accesslog and Password Policy Overlays

In the last chapter we saw OpenLDAP's overlay technology, and we implemented a few simple overlays. In this chapter we have seen how LDAP schemas work. Now we are going to take a look at a few overlays that require custom schemas.

The two overlays that we will examine are the `accesslog` overlay and the `ppolicy` (Password Policy) overlay.

Because they require their own schemas, and because each provides a robust feature set, these two overlays have a more complicated configuration. However, since the basic concepts are familiar already, we will move quickly.

## Logging with the Accesslog Overlay

The Access Logging overlay (`accesslog`) extends the logging abilities of the SLAPD server. First, it makes it possible to track client access to the directory server. Second, it stores this logging data within the directory, making it possible to retrieve access logs from any authorized LDAP client.

Since it stores information inside of the directory server, and since the format for access log entries is not already described in any of the familiar schemas, the access logging overlay needs its own schema.

The access log schema is still considered experimental, and has not yet been finalized. Nor is it included in the schema directory (`/etc/ldap/schema` or `/usr/local/etc/openldap/schema`). The object classes are defined in the man page (`man` `slapo-accesslog`).

However, the access log overlay automatically loads its own schema, so there is no manual schema configuration to be done.

The process of installing the `accesslog` is of four steps:

1.  Load the `accesslog` module
2.  Configure the `accesslog` backend section
3.  Create a database to store the access log
4.  Configure the directory backend to log to the new database

### Loading the accesslog Module

By now, this step should be familiar. Along with the other `moduleload` statements at the top of `slapd.conf`, we need to add one to load the `accesslog` module:

```
modulepath      /usr/local/libexec/openldap
moduleload      back_hdb
moduleload      denyop
moduleload      refint
moduleload      unique
moduleload      accesslog

```

When SLAPD is restarted the `accesslog` module, which contains the `accesslog` overlay, will be loaded.

### Configuring the Access Log Backend

The `accesslog` overlay needs a location within the directory server to write the access information. We will create an extra database backend that will hold the logging data.

There is nothing particularly special about this backend. It functions just like any other, and we will use the standard set of configuration directives. But there is one catch to implementing `accesslog`: the database where the access logs are stored must appear in `slapd.conf` *before* the database that it is going to record access data about.

We want to log access to our first database (the one with suffix `dc=example,dc=com`), so we need to insert the configuration directives for the access log before the `dc=example,dc=com` database. Here's the beginning of the original Example.Com database definition:

```
##############################
# BDB Database Configuration #
##############################
# Database 1: Example.Com

database        hdb
suffix          "dc=example,dc=com" "o=My Company,c=US"
rootdn          "cn=Manager,dc=example,dc=com"
```

We will insert our access log configuration above the `database` directive in the previous example:

```
##############################
# BDB Database Configuration #
##############################
# Database 1: Logging DB

database hdb
suffix cn=log
rootdn          "cn=Manager,cn=log"
rootpw          secret
directory      /var/lib/ldap/accesslog
#directory       /usr/local/var/openldap-data/accesslog
index reqStart eq

##############################
# Database 2: Example.Com

database        hdb
suffix          "dc=example,dc=com" "o=My Company,c=US"
```

The highlighted section is the definition for the access log database.

As with the other databases, this one uses the HDB backend. The suffix for our logging directory will simply be `cn=log`.

Each logging event will be stored as an LDAP record, and each entry in the logging directory will have a DN composed of two attributes. The RDN is the `reqStart` attribute (which contains the timestamp indicating when the request started), and ends with the suffix which, in our case, is `cn=log`.

This database also has its own manager account and password (`rootdn` and `rootpw`). The Berkeley DB files will be stored at `/var/lib/ldap/accesslog`—a directory we will create on the file system in the next step.

Finally, the `index` directive configures an equality (`eq`) index for the `reqStart` attribute, which is the attribute SLAPD uses to create DNs. It uses this attribute when performing maintenance operations, so it is a good idea to have this attribute indexed.

There are a few more things to do in `slapd.conf`. But before doing those, we will create a directory for the Berkeley DB files.

### Creating A Directory for the Access Log Files

Like the other HDB databases, this new database needs a location on the server's file system to store Berkeley DB database files. In the earlier configuration, we pointed SLAPD to the directory `/var/lib/ldap/accesslog`. Now, we need to create that directory and configure it for a Berkeley DB environment.

The first thing to do is create the new directory. From a shell this can be done easily:

```
 $ sudo mkdir /var/lib/ldap/accesslog

```

From there, all we need to do is copy the `DB_CONFIG` to the new `accesslog/` directory:

```
 $ sudo cp /var/lib/ldap/DB_CONFIG /var/lib/ldap/accesslog/

```

Depending on the traffic on your server and the amount of data you are logging, you may want to increase or decrease the cache size allocated in `DB_CONFIG`. See the discussion in the previous chapter for more information on tuning the `DB_CONFIG` file.

### Note

**Check the DB_CONFIG files**

The `DB_CONFIG` file we created in the last chapter did not have any absolute references to locations on the file system. But some directives in the `DB_CONFIG` file (like `set_lg_dir`) might have absolute path references, which could result in two databases using the same log. That would have catastrophic consequences. Make sure you adjust the `DB_CONFIG` file accordingly.

Make sure that the new `accesslog/` directory is readable and writable for the user account that runs the SLAPD process, and also make sure that that user can read the `DB_CONFIG` file.

### Enabling Logging for the Main Backend

Now we have the logging environment set up. The next thing to do is configure our `dc=example,dc=com` backend to start using the new logging backend.

Back in `slapd.conf`, we need to add some new overlay-specific directives inside of the `dc=example,dc=com` backend. These directives must come after the database definition for the Example.Com database:

```
##############################
# Database 1: Example.Com

database        hdb
suffix          "dc=example,dc=com" "o=My Company,c=US"

# ... a dozen lines omitted ...

overlay accesslog
logdb cn=log
logops all
logold (objectclass=person)
logpurge 7+00:00 2+00:00
logsuccess TRUE
```

The first directive, `overlay` `accesslog`, loads the access logging overlay within the context of this particular database. The next five directives are the accesslog-specific directives.

The `logdb` directive is the only one required by the `accesslog` overlay. All the rest are optional.

The `logdb` directive specifies which database will be treated as an access log. In our case we want to use the `cn=log` database. For a site hosting multiple directory information trees, separate logging databases could be set up for each suffix.

The `logops` directive is used to specify exactly which LDAP operations should be logged. In this example, the keyword `all` indicates that all operations will be logged. But the following options are supported:

*   Any operation can be specified by name: `add`, `delete`, `modify`, `modrdn`, `search`, `compare`, `extended`, `bind`, `unbind`, and `abandon`.
*   There are a few special keywords that include a collection of operations. These are:

    *   `read` (search, compare)
    *   `write` (add, delete, modify, modrdn)
    *   `session` (bind, unbind, abandon)

*   There is the `all` keyword, which includes all operations.

More than one value can be placed on a `logops` line. Values should be separated by an empty space. For example, `logops` `modify` `modrdn` will log all modify and modrdn operations.

The `logold` ("log old") directive takes a search filter. When a delete or modify operation is successfully executed, then `accesslog` will check to see if the record matches the filter. If it does match, then `accesslog` will store a complete record of the change, including what attributes were added, and what attributes were changed or removed. For example, when I modified a user with the `ldapmodify` command-line tool, an entry detailing the changes was written to the accesslog directory information tree:

```
dn: reqStart=20070117022818.000002Z,cn=log
objectClass: auditModify
reqStart: 20070117022818.000002Z
reqEnd: 20070117022818.000003Z
reqType: modify
reqSession: 4
reqAuthzID: uid=matt,ou=users,dc=example,dc=com
reqDN: uid=barbara,ou=users,dc=example,dc=com
reqResult: 0
reqMod: objectClass:+ labeledURIObject
reqMod: labeledURI:+ http://example.com Home Page
reqMod: entryCSN:= 20070117022818Z#000001#00#000000
reqMod: modifiersName:= uid=matt,ou=users,dc=example,dc=com
reqMod: modifyTimestamp:= 20070117022818Z
reqOld: objectClass: person
reqOld: objectClass: organizationalPerson
reqOld: objectClass: inetOrgPerson
reqOld: entryCSN: 20061228230549Z#000000#00#000000
reqOld: modifiersName: cn=Manager,dc=example,dc=com
reqOld: modifyTimestamp: 20061228230549Z

```

The `reqMod` values show the new modifications, while the `reqOld` attribute values show the old lines. Note that two lines were added (the object class and the `labeledURI`), and two were changed (`modifiersName`, `modifyTimestamp`).

Why use `logold`? It may not be particularly useful for log evaluation but, when combined with SyncRepl, synchronization between SLAPD servers can be done more efficiently. (This form of SyncRepl is called **Delta-SyncRepl**.) If you are not using SyncRepl, you probably won't want to use `logold` at all. We will discuss SyncRepl (and Delta-SyncRepl) in detail in the next chapter.

The `logpurge` directive directs SLAPD to periodically check the access log and delete old entries. It takes two parameters that provide the following information: how old an entry must be before it is a candidate for being purged, and how long of an interval should pass between checking for entries to remove.

The format of the two parameters is the same:

```
[<number of days>+]<hours>:<minutes>[:<seconds>]
```

Where number of days and number of seconds are optional fields. Our `logpurge` parameter looked like this:

```
logpurge 7+00:00 2+00:00
```

This indicates that logs seven days old are to be considered for deletion. And after running a check, SLAPD will wait the indicated amount of time—two days—before checking for new deletions.

The last parameter is `logsuccess`. By default, `accesslog` records all attempted operations, whether successful or not. To log only the operations that are successfully completed set `logsuccess` to `TRUE`.

That's all there is to configuring `accesslog`. SLAPD will need to be restarted for the new overlay to be added.

### The Log Records

Now that we have our new logging overlay running, let's test it out. The first step is to generate some logging data. Since we are logging all operations (`logops` `all`), any LDAP operation will do.

Here is a simple `ldapsearch`:

```
 $ ldapsearch -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
 '(uid=matt)' mail gn sn

```

This uses a simple bind, and searches for my own record (`uid=matt`), retrieving the values for the `mail`, `gn` (given name) and `sn` attributes.

With a search like this, what is written to the access log? To find out, we can use `ldapsearch`:

```
$ ldapsearch -LL -U matt -b 'cn=log'

```

The output for this command, even with the results of only one command, is surprisingly large:

```
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
version: 1

dn: cn=log
objectClass: auditContainer
cn: log

dn: reqStart=20070117044539.000000Z,cn=log
objectClass: auditBind
reqStart: 20070117044539.000000Z
reqEnd: 20070117044539.000001Z
reqType: bind
reqSession: 0
reqAuthzID:
reqDN: uid=matt,ou=users,dc=example,dc=com
reqResult: 0
reqVersion: 3
reqMethod: SIMPLE

dn: reqStart=20070117044539.000002Z,cn=log
objectClass: auditSearch
reqStart: 20070117044539.000002Z
reqEnd: 20070117044539.000003Z
reqType: search
reqSession: 0
reqAuthzID: uid=matt,ou=Users,dc=example,dc=com
reqDN: dc=example,dc=com
reqResult: 0
reqScope: sub
reqDerefAliases: never
reqAttrsOnly: FALSE
reqFilter: (uid=matt)
reqAttr: mail
reqAttr: gn
reqAttr: sn
reqEntries: 1
reqTimeLimit: 3600
reqSizeLimit: 500

dn: reqStart=20070117044540.000000Z,cn=log
objectClass: auditObject
reqStart: 20070117044540.000000Z
reqEnd: 20070117044540.000001Z
reqType: unbind
reqSession: 0
reqAuthzID: uid=matt,ou=Users,dc=example,dc=com
```

There are four different entries returned from the `ldapsearch` and each one has a different structural object class. We will look at each in turn.

The first LDIF entry it displays is the base record for `cn=log`:

```
dn: cn=log
objectClass: auditContainer
cn: log
```

The `auditContainer` object class is designed as a sort of general-purpose object class for the access log. It's schema looks like this:

```
objectClass 
  ( 
   1.3.6.1.4.1.4203.666.11.5.2.0 
   NAME 'auditContainer' 
   DESC 'AuditLog container' 
   SUP top 
   STRUCTURAL 
   MAY ( cn $ reqStart $ reqEnd ) 
  )
```

The base record only uses the optional `cn` attribute.

In the `accesslog` schema there are object classes defined for each LDAP operation: `auditAbandon`, `auditAdd`, `auditBind`, `auditCompare`, `auditDelete`, `auditModify`, `auditModRDN`, `auditSearch`, and `auditExtended`. In addition, there is a special object class called `auditObject` that describes general events.

In fact (in the current version) all of the operation object classes listed are subordinates to the `auditObject` object class. Because it is the parent of these other object classes, we will begin by looking at the `auditObject` schema definition.

The `auditObject` object class definition looks like this:

```
objectclass 
  ( 
   1.3.6.1.4.1.4203.666.11.5.2.1 
   NAME 'auditObject' 
   DESC 'OpenLDAP request auditing' 
   SUP top 
   STRUCTURAL 
   MUST ( reqStart $ reqType $ reqSession ) 
   MAY ( reqDN $ reqAuthzID $ reqControls $ reqRespControls $
         reqEnd $ reqResult $ reqMessage $ reqReferral ) 
  )
```

The three required attributes are:

*   `reqStart`:A timestamp indicating the starting time of the operation
*   `reqType`: A string indicating the operation being executed
*   `reqSession`: The connection ID number used (internally) by SLAPD

In addition to these required attributes, there are eight optional attributes:

*   `reqDN`: This records the DN of the record the operation is currently operating on.
*   `reqAuthzID`: This records the DN of the user performing the operation. If the user is Anonymous the value is left blank.
*   `reqControls` and `reqRespControls`: If the client sets any controls, they are indicated here.
*   `reqEnd`: This stores the timestamp indicating when the operation was completed.
*   `reqResult`: This contains the numeric error code if an error was encountered. If the operation is successful this returns `0`.
*   `reqMessage`: If the error code is accompanied by a text message, the message is put in this attribute.
*   `reqReferral`: If the operation returned a referral, the referral is noted here.

The second entry returned in the search records the client's bind operation:

```
dn: reqStart=20070117044539.000000Z,cn=log
objectClass: auditBind
reqStart: 20070117044539.000000Z
reqEnd: 20070117044539.000001Z
reqType: bind
reqSession: 0
reqAuthzID:
reqDN: uid=matt,ou=users,dc=example,dc=com
reqResult: 0
reqVersion: 3
reqMethod: SIMPLE
```

This first entry records the bind operation, and is an instance of the `auditBind` object class. The `auditBind` object class is a subordinate of `auditObject`:

```
objectClass 
  ( 
   1.3.6.1.4.1.4203.666.11.5.2.6 
   NAME 'auditBind' 
   DESC 'Bind operation' 
   SUP auditObject 
   STRUCTURAL 
   MUST ( reqVersion $ reqMethod ) 
  )

```

It adds two required attributes: `reqVersion`, which records the version of LDAP used for the connection and `reqMethod`, which indicates what method was used in binding.

Looking at the bind entry, we can see that it records the details of a successful bind operation. The start and end times are recorded in `reqStart` and `reqEnd` respectively. The `reqType` indicates that the operation performed is a bind operation. The `reqSession` indicates the internal ID of the request (which happens to be zero because this is the first operation run since we started SLAPD, and connection IDs increment starting at `0`).

Since the bind was performed by the anonymous user, the `reqAuthzID` attribute is present, but has no value. The `reqDN` indicates that the client was attempting to bind as `uid=matt,ou=users,dc=example,dc=com`, and the `reqResult` of `0` indicates that the bind operation was completed successfully. The bottom two attributes are the attributes that belong to the `auditBind` object class. The `reqVersion` attribute indicates that the client used the LDAPv3 protocol and, according to `reqMethod`, the bind was a simple bind.

So, the first operation performed in this LDAP session was a bind. The second operation is the search:

```
dn: reqStart=20070117044539.000002Z,cn=log
objectClass: auditSearch
reqStart: 20070117044539.000002Z
reqEnd: 20070117044539.000003Z
reqType: search
reqSession: 0
reqAuthzID: uid=matt,ou=Users,dc=example,dc=com
reqDN: dc=example,dc=com
reqResult: 0
reqScope: sub
reqDerefAliases: never
reqAttrsOnly: FALSE
reqFilter: (uid=matt)
reqAttr: mail
reqAttr: gn
reqAttr: sn
reqEntries: 1
reqTimeLimit: 3600
reqSizeLimit: 500
```

Since it describes a search operation, this entry uses the `auditSearch` object class, which has the following schema definition:

```
objectClass 
  ( 
   1.3.6.1.4.1.4203.666.11.5.2.11 
   NAME 'auditSearch' 
   DESC 'Search operation' 
   SUP auditReadObject 
   STRUCTURAL 
   MUST ( reqScope $ reqDerefAliases $ reqAttrsonly ) 
   MAY ( reqFilter $ reqAttr $ reqEntries $ reqSizeLimit $
         reqTimeLimit ) 
  )
```

Note that `auditSearch` is a subordinate not of `auditObject` but of `auditReadObject`, another structural object class that is itself subordinate to `auditObject`. In other words, `auditSearch` is an indirect subclass of `auditObject`. The `auditReadObject` (as of OpenLDAP 2.3.30) does not add any additional attributes.

For the most part the attributes inherited from `auditObject` perform in the same capacity here as they did in the entry for the bind operation. The `reqAuthzID` in this case is the authenticated user's DN, instead of empty, and the `reqDN` shows the base DN for the search operation.

The next set of attributes provide detailed information about the nature of the search request.

*   `reqScope` indicates the scope of the search. `reqDerefAliases` indicates that aliased entries (entries mapped to other entries elsewhere in the directory, a concept similar to symbolic linking in Linux file systems) are never dereferenced during searches. The `reqAttrsOnly` flag indicates that the search did not request that only the attribute names be returned. Instead, the names and values were to be returned.
*   `reqFilter` contains the LDAP search filter. This is the filter we specified on the command line when running the `ldapsearch` command.
*   `reqAttr` has three values, `mail`, `gn`, and `sn`, corresponding to the three attributes I requested in the `ldapsearch` command. And `reqEntries` indicates the total number of matching records found in the directory.
*   `reqTimeLimit` and `reqSizeLimit` indicate the (soft) size and time limits requested in the search.

Taken as a whole, this entry provides a detailed record of what my LDAP search was and, from this record alone, it would be trivial to replicate the exact search.

There is one final (short) entry left, the entry that records the client's unbind.

```
dn: reqStart=20070117044540.000000Z,cn=log
objectClass: auditObject
reqStart: 20070117044540.000000Z
reqEnd: 20070117044540.000001Z
reqType: unbind
reqSession: 0
reqAuthzID: uid=matt,ou=Users,dc=example,dc=com
```

Since there are no paramters to the unbind operation (just the closing of a connection), there is no specific object class to model this event. Instead, the `auditObject` object class is used as the structural object class for this entry.

When clients perform other kinds of LDAP operations, such as additions and modifications, different object classes will be used. The object class definitions (and attribute definitions) can be found in the `cn=sucbschema` record. See the earlier section *Retrieving* *the* *Schema* *from* *SLAPD* for information on how to do this.

Now we have finished looking at the `accesslog` overlay. This overlay can come in use not only for record keeping but for debugging troublesome issues, discovering which attributes would most benefit form indexing, and even adding performance-enhancing functionality to directory replication. In the next section, we will look at the password policy overlay.

## Implementing a Complex Overlay: Password Policy

One of the proposed extensions to LDAP is a standardized method for implementing password policies in an LDAP directory. The Password Policy (`ppolicy`) overlay implements the "Password Policy for LDAP Directories" IETF draft, which is likely to soon become an RFC.

A password policy provides account aging, password expirations, password strength checking, grace logins, and a variety of other password maintenance services.

How does this work in OpenLDAP? Password policy information is stored inside of the directory information tree in records described by a specialized schema. The `ppolicy` overlay monitors connections, updating password information and enforcing the password policy as appropriate.

### Note

Password policies operate on the `userPassword` attribute. That means that if you use SASL and store the passwords outside of the directory information tree (in a place such as the `sasldb`), then the `ppolicy` overlay will not function. In this chapter we will be using simple binding.

The password policy schema defines the object class, `pwdPolicy`, that is implemented by password policy entries. There are no object classes for user records. Instead, operational attributes (attributes used internally by SLAPD) are used to store password policy information in user records. These operational attributes are used to store internal information (such as when a user last changed the password), and usually managed solely by the `ppolicy` overlay.

The password policy extension has many features, all documented in the man page, as well as in the IETF draft standard. Since the draft has not been finalized, and is still in a state of change, this module is marked as experimental. New features may be added, or current features altered or even removed, as the standard changes. But the experimental categorization does not reflect on the stability of the code. Administrators of large systems have reported this module to be production quality.

Because of the wealth of features, the `ppolicy` overlay is not a quick and easy install. It will require the following steps:

1.  Include the password policy schema and load the module
2.  Create a password policy
3.  Configure the `ppolicy` overlay

Once the password policy overlay is implemented, we will do some testing.

### Setting the Global Directives in slapd.conf: Schema and Module

The first thing we need to do is configure the global (basic) section of the `slapd.conf` file. As with the other overlays we will need to load the `ppolicy` module. And since we are using a new schema—one stored in the `schema/` directory—we will need to include that too.

Since the directives are close together, we can look at both additions at once:

```
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/ppolicy.schema

#pidfile /var/run/slapd/slapd.pid
#argsfile /var/run/slapd/slapd.args
pidfile /usr/local/var/run/slapd.pid
argsfile /usr/local/var/run/slapd.args
loglevel none

modulepath /usr/lib/ldap
# modulepath /usr/local/libexec/openldap
moduleload back_hdb
moduleload denyop
moduleload refint
moduleload unique
moduleload accesslog
moduleload ppolicy

```

The two highlighted lines show the necessary changes:

1.  The highlighted `include` directive imports the `ppolicy.schema` file into the configuration
2.  The `moduleload` directive loads the `ppolicy` module

In step 3 we come back to the `slapd.conf` file and make a few more changes, but next we need to create a password policy and load it into the directory. That will require restarting SLAPD to pick up the new schema definitions:

```
 $ sudo invoke-rc.d slapd restart

```

### Creating a Password Policy

This step is more demanding than the previous. Our goal is to load a new password policy into the directory. To do this, we will need to get acquainted with the `pwdPolicy` object class in the `ppolicy` schema, create the requisite LDIF entries for our directory information tree, and then load these into the directory with `ldapadd`.

The `pwdPolicy` object class contains a number of attributes that can be used for storing information about a password policy. A password policy is a set of conditions determining what constraints will be placed on password usage within the LDAP server.

Here is the schema for the `pwdPolicy` object class:

```
objectclass 
  ( 
   1.3.6.1.4.1.42.2.27.8.2.1
   NAME 'pwdPolicy'
   SUP top
   AUXILIARY
   MUST ( pwdAttribute )
   MAY ( pwdMinAge $ pwdMaxAge $ pwdInHistory $ pwdCheckQuality $
         pwdMinLength $ pwdExpireWarning $ pwdGraceAuthNLimit $
         pwdLockout $ pwdLockoutDuration $ pwdMaxFailure $
         pwdFailureCountInterval $ pwdMustChange $ pwdAllowUserChange 
         $ pwdSafeModify ) 
  )
```

This object class is an auxiliary object class so, when we create an entry to hold the policy, it will need a structural object class.

There is only one required attribute for `pwdPolicy`: `pwdAttribute`. The value of this attribute should be set to the OID of the attribute used for password storage. Since the schema is part of a proposed standard, the purpose of this attribute is to make it possible for different directory servers to all use the same schema (since different directory server implementations use different attributes for storing password values). However, for OpenLDAP's SLAPD, the only attribute that can be used here is the OID for `userPassword`, which is `2.5.4.35`.

### Note

The `authPassword` attribute, defined in RFC 3112, is a candidate for replacing `userPassword` in future versions of OpenLDAP. However, at this time it is not completely implemented.

The remaining attributes, all of which are optional, are used to store policy information. Here is a brief explanation of what each attribute is used for:

*   `pwdMinAge`: This specifies how much time must pass (in seconds) between the last time the password was changed and the next time SLAPD will allow the password to be changed. Setting this prevents an account from having the password changed multiple times in rapid succession.
*   `pwdMaxAge`: This specifies how long (in seconds) a password will be considered good. This is calculated from the time when the password was last changed. After the elapsed time, the password will be marked as expired.
*   `pwdInHistory`: If you store your passwords in plain text (unencrypted) in the directory then the `ppolicy` overlay can be configured to maintain a password history and prevent users from re-using passwords. This attribute is used to specify the maximum number of passwords that `ppolicy` will maintain for each user. Unless this attribute is set, and to a value greater than zero, no history will be maintained.
*   `pwdCheckQuality`: There are two quality checks done by `ppolicy` if `pwdCheckQuality` is set to check passwords. The first is length checking (discussed next). The second is running a custom quality checking function. It is possible (using the `pwdCheckModule` object class and some custom C code) to add your own password quality checking module to SLAPD, and then use it to check password quality. This attribute takes one of three integer values: `0`, `1`, and `2`. Now we have three cases:

    *   If the value is `0` (the default), then `ppolicy` will not attempt to do any quality checking.
    *   If `1`, then `ppolicy` will attempt checking, but if the password is encrypted and certain checking functions cannot be performed, it will return successful.
    *   If `2`, then if the password checking function cannot run, it will return an error message.

*   `pwdMinLength`: If `pwdCheckQuality` is set to `1` or `2`, then `ppolicy` will make sure that new passwords meet a minimum length requirement. This attribute, which takes a positive integer, can be used to set the minimum acceptable length for a password.
*   `pwdExpireWarning`: When a password approaches its expiration date (set in `pwdMaxAge`), `ppolicy` can provide a warning to the user when the user logs in. This attribute takes the time, in seconds, prior to when the password expires that it should start warning the user. In other words, at `pwdMaxAge`—`pwdExpireWarning` from when the password was set—the user will start getting warning messages. If this is set to `0` (the default) then no expiration warning will be sent.
*   `pwdGraceAuthNLimit`: By default (or if this attribute is set to `0`), when a password expires the account is locked and the user can no longer bind to the directory server. But using this attribute we can allow grace logins. The value of this attribute should be a non-negative integer, which will specify how many grace logins a user with an expired password will be allowed before the account is locked.
*   `pwdLockout`: This attribute allows you to turn on password lockouts. If this is turned on, then when a user fails to bind a certain number of times (`pwdMaxFailures`) in a row, then the account will be locked for some duration of time (`pwdLockoutDuration`). To turn on `pwdLockout`, which is off by default, set the value of this attribute to `TRUE`.
*   `pwdLockoutDuration`: This attribute specifies the amount of time, in seconds, that an account will be locked out if `pwdLockout` is set to `TRUE` and the user fails to log in too many times (the number set in `pwdMaxFailures`). If this is set to `0` or is not set, then the account will be locked until an administrator re-enables it.
*   `pwdMaxFailures`: This specifies the number of times in a row that a user can fail a login before being locked out. `pwdLockout` must be set to `TRUE` before this constraint will be enforced though.
*   `pwdFailureCountInterval`: This attribute can be used to fine-tune the timing involved in password lockouts. By default (or when this attribute is set to `0`), failed login attempts are stored until a successful login is made. But the value of this attribute can be set to a number of seconds that `ppolicy` will wait before clearing the password failure count.
*   `pwdMustChange`: This determines whether or not a user must change their password after an administrator sets it. By default, the user is not prompted to change a password. But if this is set to `TRUE`, if an administrator changes (or initially sets) a password, the user will be prompted to reset the password.
*   `pwdAllowUserChange`: By default, users are allowed to change their own passwords. But if this is set to `FALSE`, users under this policy will not be allowed to change their own passwords. Since different policies can be assigned to different groups of users, this allows finer-grained control of write permissions to a password than ACLs do.
*   `pwdSafeModify`: By default, once a user has successfully performed a bind operation, the user can change passwords without having to re-send the original password. But if `pwdSafeModify` is set to `TRUE`, then the user will have to send both the old password and the new password in order to change the password value. This adds an extra level of security to the password changing process.

Some of the policy attributes—primarily the password checking functions and password history—require that the password be stored in cleartext within the directory. This is the case simply because comparison functions do not work on encrypted values. Two identical password values, if using different salt sequences, will result in different ciphertexts. Two different hashing algorithms (like MD5 and SHA) will generate different hashes for the same password even if the same salt is used. Likewise, given certain hashing algorithms, two different strings could generate the same ciphertext (though the possibility of this happening to a particular user is negligible).

Most of the other features though, work regardless of how the values are stored in the directory.

Now we are ready to create an LDIF file to hold our policy. By convention, password policies are usually located in a separate OU in the directory information tree. We will add a new OU for that purpose.

And for our policy we will use the majority of the possible attributes:

```
dn: ou=Policies,dc=example,dc=com
ou: Policies
description: Directory policies.
objectclass: organizationalUnit

dn: cn=Standard,ou=Policies,dc=example,dc=com
cn: Standard
description: Standard password policy.
pwdAttribute: 2.5.4.35
pwdMinAge: 60
# 30 days: 60 sec * 60 min * 24 hr * 30 days
pwdMaxAge: 2592000
pwdCheckQuality: 1
pwdMinLength: 7
# Warn three days in advance
pwdExpireWarning: 259200
pwdGraceAuthNLimit: 3
pwdLockout: TRUE
pwdLockoutDuration: 1200
pwdMaxFailure: 3
pwdFailureCountInterval: 1200
pwdMustChange: TRUE
pwdAllowUserChange: TRUE
pwdSafeModify: TRUE
objectclass: device
objectclass: pwdPolicy
```

The first entry is for our organizational unit. The second is our password policy. Since the `pwdPolicy` object class is auxiliary we have to give the entry another object class, a structural object class. The `device` object class is typically used (based on the testing schema used in the source distribution of OpenLDAP).

### Tip

**Why is pwdPolicy Auxiliary?**

There are a few reasons why the creators of the password policy specification might have made such a choice. First, according to RFC 4512, a structural object class must represent a physical entity. Second, making the class auxiliary makes it possible to integrate this schema with other existing schemas. For us, though, this presents the minor difficulty that there are no good candidates for a structural object class.

We can now add this LDIF with `ldapadd`. We have the above LDIF saved in a file called `ppolicy.ldif`, so we can add it with the following command:

```
 ldapadd -x -W -D 'uid=matt,ou=users,dc=example,dc=com' -f ppolicy.ldif

```

This adds our two new entries to the directory.

### Note

Make sure you have restarted the server since adding the schema. If the `ppolicy` schema has not been loaded, the above will not work.

Now that we have our entries loaded it is time to return to `slapd.conf` and configure the overlay.

### Configure the Overlay Directives

In the first step of setting up the password policy overlay, we added directives to `slapd.conf` to include the `ppolicy` schema definitions and load the `ppolicy` module. Now we will look at the backend configuration for the overlay.

As with the other overlays, all of the configuration directives are backend specific. Also, since the `ppolicy` overlay does a lot of writing to the directory information tree, not all features work on read-only databases.

While this overlay is sophisticated, there are only three directives for the overlay, and these are all straightforward. The relevant section in our `slapd.conf` file, in the `dc=example,dc=com` directory tree, looks like this:

```
overlay ppolicy
ppolicy_default cn=Standard,ou=Policies,dc=example,dc=com
ppolicy_use_lockout
ppolicy_hash_cleartext 
```

Once the overlay is applied to this database using the `overlay` directive, there are three overlay-specific directives.

The first, `ppolicy_default`, points to the DN of the entry in the directory information tree that is to be treated as the default password policy entry. As we will see shortly different entries can use different policies. But the one indicated by `ppolicy_default` is the one that `ppolicy` will use when another is not explicitly set. For our example above, it is set to the DN of the entry that we created in the previous step.

The second directive is `ppolicy_use_lockout`. This directive alters how SLAPD reports error messages due to account lockouts. When a user's account is locked by the password policy overlay the user is not allowed to bind again. By default (when this directive is not included), the client is notified that the bind failed because of invalid credentials (the generic LDAP error) but no additional information is given. When this directive is present though, then SLAPD sends the *Account* *Locked* error code.

### Note

While this extra error message might be helpful to the user, it could have negative consequences. An attacker might be able to determine, based on this information, that the server is using the password lockout features. Such an attacker could then perform a denial of service attack against known accounts on the server simply by attempting to login on each known account until the account was locked.

The last `ppolicy` directive, `ppolicy_hash_cleartext`, modifies the way SLAPD handles changes to the password. In short, if this directive is present, then SLAPD will automatically hash cleartext passwords when they are changed using the LDAP modify operation (as opposed to the LDAP password modify extended operation).

To understand what this means, let's look at an example. In our directory we have the following record (created in Chapter 3):

```
dn: uid=adam,ou=Users,dc=example,dc=com
cn: Adam Smith
sn: Smith
uid: adam
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
```

This user does not yet have a password. One way to set such a password would be to use the `ldappasswd` too, which (as we saw in Chapter 3) uses the LDAP password modify extended operation. This is the best way to change passwords as the server handles the password encryption. Here's an example of setting a password with `ldappasswd`:

```
 $ ldappasswd -U matt -s secret 'uid=adam,ou=users,dc=example,dc=com'

```

This sets the password for `uid=adam` to `secret`. What will the record look like now? Like this:

```
dn: uid=adam,ou=Users,dc=example,dc=com
cn: Adam Smith
sn: Smith
uid: adam
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
userPassword:: e1NTSEF9WlFzZWdrVUdpT3JKNUgwYXFRdisxQ0dpaTNYUFdkMjA=
```

The `userPassword` value is base64 encoded. Its decoded value is this:

```
{SSHA}ZQsegkUGiOrJ5H0aqQv+1CGii3XPWd20
```

SLAPD performed the SSHA hashing of the value.

There is a second way of modifying the password and this is with the LDAP modify operation (as used by the `ldapmodify` client). When a `userPassword` value is changed with LDAP modify it is assumed that the client is sending the password value in the form in which it should be stored. In fact, the LDAP standard states that this is how the server should act when performing a modification of an attribute value. Thus, SLAPD will not encrypt the password.

Here's an example of using `ldapmodify` to set the password:

```
$ ldapmodify -x -W -D 'uid=matt,ou=users,dc=example,dc=com'
Enter LDAP Password: 
dn: uid=adam,ou=users,dc=example,dc=com
changetype: modify
replace: userPassword
userPassword: secret
modifying entry "uid=adam,ou=users,dc=example,dc=com"
```

The highlighted portion above is the LDIF information to be modified. The value of the `userPassword` attribute was set to `secret`—the same password used in the `ldappasswd` example. But this time, if we look at the entry, the `userPassword` value is not encrypted:

```
dn: uid=adam,ou=Users,dc=example,dc=com
cn: Adam Smith
sn: Smith
uid: adam
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
userPassword:: c2VjcmV0
```

The password is not hashed. Instead, it is just base64 encoded. The decoded value is `secret`.

Including the `ppolicy_hash_cleartext` directive modifies this behavior. During modifications the `ppolicy` overlay checks to see if the modified attribute is `userPassword` and if the value is in cleartext. If the value is in cleartext then `ppolicy` hashes it.

### Note

In effect, turning on this feature causes SLAPD to perform in a nonstandard way, but for the sake of additional security.

For example, we can re-run the same `ldapmodify`:

```
$ ldapmodify -x -W -D 'uid=matt,ou=users,dc=example,dc=com'
Enter LDAP Password: 
dn: uid=adam,ou=users,dc=example,dc=com
changetype: modify
replace: userPassword
userPassword: secret
modifying entry "uid=adam,ou=users,dc=example,dc=com"
```

But this time, since `ppolicy_hash_cleartext` is on, the password is encrypted:

```
dn: uid=adam,ou=Users,dc=example,dc=com
cn: Adam Smith
sn: Smith
uid: adam
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
userPassword:: e1NTSEF9Q0M3QUdSQUlPMG4vYy8rbVZiRE95bC9aYnpqNHcxd1Q=
```

The `userPassword` value, decoded, is `{SSHA}CC7AGRAIO0n/c/+mVbDOyl/Zbzj4w1wT`. When hashing cleartext is enabled, LDAP modify operations (for the `userPassword` attribute only) behave more like LDAP password modify extended operations.

We've now configured the overlay completely. It will take a restart for SLAPD to pick up the changes to `slapd.conf` though. Now we are ready to test some of these features.

### Test the Overlay

As it is configured now, SLAPD will enforce policy controls on any entry with a `userPassword` attribute. Let's do a little testing to see how the password policy works.

### Tip

**The Administrator**

As I work through the examples I use the `uid=matt` account as a managing account. This account is allowed (by the ACLs) to perform administrative tasks. But it is also subject to the constraints of the `ppolicy` overlay.

The root DN account `(cn=manager,dc=example,dc=com on this server)` is treated differently. For example, the manager can set a password for a user without having to know the user's old password, even if `pwdSafeModify` is on.

First, let's see how the policy responds to some password changes. And let's start this examination with an `ldapmodify` attempt:

```
$ ldapmodify -x -W -D 'uid=matt,ou=users,dc=example,dc=com'
Enter LDAP Password: 
dn: uid=adam,ou=users,dc=example,dc=com
changetype: modify
replace: userPassword
userPassword: new_password
modifying entry "uid=adam,ou=users,dc=example,dc=com"
ldap_modify: Insufficient access (50)
        additional info: Must supply old password to be changed as
        well as new one
```

The modification attempt fails because the the `pwdSafeModify` is set to `TRUE`. There is no way to satisfy this requirement with `ldapmodify`. Instead we will have to use `ldappasswd` to change the password and we will have to set it to supply the server with the old password. This is what we will get:

```
$ ldappasswd -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
   -s new_password -a secret 'uid=adam,ou=users,dc=example,dc=com'
Enter LDAP Password: 
Result: Success (0)
```

The `-s` flag is used to specify the new password, while the `-a` flag is used to provide the old password (and then `ldappasswd` prompts for the password of the DN that is binding too). With both of these set we meet the requirements of `pwdSafeModify`.

Since we have password checking turned on we should be able to test password length:

```
$ ldappasswd -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
  -s short -a new_password  'uid=adam,ou=users,dc=example,dc=com'
Enter LDAP Password: 
Result: Constraint violation (19)
Additional info: Password fails quality checking policy
```

In this case the new password, `short`, is (as the name implies) too short. The `pwdMinLength` of the policy states that the password must be seven characters long, and when the password quality checking function is performed (which it will since `pwdCheckQuality` is set to `1`), the server returns an error noting that it failed. Unfortunately for the user the message does not indicate the precise reason.

Next, let's look at password expiration warnings and password expirations. This will require some minor changes to our policy for the sake of testing—namely we will want to set the values for `pwdMaxAge` and `pwdExpireWarning` to lower values (values that would normally be too low for a production environment). Let's set the password to expire every ten minutes, and the expiration message to come up for the last nine minutes:

```
$ ldapmodify -x -W -D 'uid=matt,ou=users,dc=example,dc=com'
Enter LDAP Password: 
dn: cn=Standard,ou=Policies,dc=example,dc=com
changetype: modify
replace: pwdMaxAge
pwdMaxAge: 600
-
replace: pwdExpireWarning
pwdExpireWarning: 540
modifying entry "cn=Standard,ou=Policies,dc=example,dc=com"
```

Now, when `uid=adam` binds, the following message is logged in the LDAP log:

```
ppolicy_bind: Setting warning for password expiry for 
              uid=adam,ou=users,dc=example,dc=com = 536 seconds
```

Unfortunately, no message is sent to the client so the user does not see the message. This may be due to the fact that the draft specification doesn't require that the messages be sent to the client. Expiry warnings then, are useful mainly to administrators.

After ten minutes the `userPassword` value will be past the expiration point and, the next time the user logs in, SLAPD will mark the password as expired. Again, the user gets no explicit warning of this fact. An entry in the log file indicates the expiration of the account:

```
ppolicy_bind: Entry uid=adam,ou=Users,dc=example,dc=com 
              has an expired password: 3 grace logins
```

But in addition to this log entry, a new operational attribute is added to the user's record. The `pwdGraceUseTime` attribute is added to the user's record, and the time stamp there indicates the last time the user performed a bind operation:

```
$ ldapsearch -LL -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
 '(uid=adam)' pwdGraceUseTime
Enter LDAP Password: 
version: 1

dn: uid=adam,ou=Users,dc=example,dc=com
pwdGraceUseTime: 20070121172107Z
```

Each time a DN with an expired `userPassword` binds to the directory, a new value is added to the `pwdGraceUseTime` attribute. So after `uid=adam` has performed three binds after the password expiration date, the user's record will contain three `pwdGraceUseTime` attribute values:

```
$ ldapsearch -LL -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
 '(uid=adam)' pwdGraceUseTime
Enter LDAP Password: 
version: 1

dn: uid=adam,ou=Users,dc=example,dc=com
pwdGraceUseTime: 20070121172107Z
pwdGraceUseTime: 20070121173638Z
pwdGraceUseTime: 20070121174603Z
```

After the number of `pwdGraceUseTime` values reaches the number in the `pwdGraceAuthNLimit` attribute of the policy, the account will be treated as locked, and that DN (`uid=adam`, in this case) will not be allowed to bind anymore. If `uid=adam` attempts to bind he will get an error message:

```
$ ldapsearch -x -W -D 'uid=adam,ou=users,dc=example,dc=com' \
 '(uid=adam)'
Enter LDAP Password: 
ldap_bind: Invalid credentials (49)
```

Furthermore, a message is added to the log noting the problem:

```
ppolicy_bind: Entry uid=adam,ou=Users,dc=example,dc=com 
              has an expired password: 0 grace logins
```

At this point an administrator will have to take steps to enable the account again.

### Password Policy Operational Attributes

In the previous section we tested several different features of the password policy. Now we will look at performing administration operations on accounts.

The `ppolicy` overlay stores information about a user's adherence to a password policy in that user's record. The information is stored in operational attributes.

Unlike regular attributes, operational attributes are not returned to clients unless the client explicitly requests them (either by name, or with the special plus (`+`) attribute specifier, which matches any operational attribute). And SLAPD can prevent clients from being able to modify operational attributes.

To begin we will look at an example of what happens when password lockout (`pwdLockout`) is turned on, and an account gets locked out. The `ppolicy` overlay uses operational attributes to store information about failures and lockouts.

In our policy, when a user fails to authenticate correctly three times in a row (according to `pwdMaxFailure`), they will be locked out of their account for some period of time (determined by `pwdLockoutDuration`).

One of the other users in our directory is `uid=dave,ou=users,dc=example,dc=com`. This user has failed to authenticate three times. The next time the user attempts to authenticate, even if he uses the right password, he will be disallowed from binding:

```
$ ldapsearch -x -W -D 'uid=dave,ou=users,dc=example,dc=com'\
   '(uid=dave)'
Enter LDAP Password: 
ldap_bind: Invalid credentials (49)
```

A few operational attributes in that user's record indicate what the problem is:

```
$ ldapsearch -LL -x -W -D 'uid=matt,ou=users,dc=example,dc=com' \
   '(uid=dave)' +
Enter LDAP Password: 
version: 1

dn: uid=dave,ou=Users,dc=example,dc=com
structuralObjectClass: inetOrgPerson
entryUUID: efbf8838-c734-102a-935c-57e457da105f
creatorsName: cn=Manager,dc=example,dc=com
createTimestamp: 20060823205147Z
pwdChangedTime: 20070121180110Z
pwdFailureTime: 20070121180139Z
pwdFailureTime: 20070121180140Z
pwdFailureTime: 20070121180142Z
pwdAccountLockedTime: 20070121180142Z
entryCSN: 20070121180142Z#000000#00#000000
modifiersName: cn=Manager,dc=example,dc=com
modifyTimestamp: 20070121180142Z
entryDN: uid=dave,ou=Users,dc=example,dc=com
subschemaSubentry: cn=Subschema
hasSubordinates: TRUE
```

Note that the `ldapsearch` in the example is for all of (and only) the operational attributes for entries that match the filter—that's what the plus sign (`+`) does.

The highlighted lines show the attributes in which we are interested: `pwdFailureTime` and `pwdAccountLockedTime`.

The `pwdFailureTime` operational attribute has a timestamp for every time the user failed a login. When a user has a successful login, the values of `pwdFailureTime` are cleared, so having three values indicates that three logins in a row have failed.

The `pwdAccountLockedTime` indicates what time the password was locked. According to our configuration, the lockout should only last for twenty minutes, after which the user will be allowed to try again.

If the user succeeds the `pwdFailureTime` and `pwdAccountLockedTime` attributes will be removed from the user's record:

```
$ ldapsearch -LL -x -W -D 'uid=matt,ou=users,dc=example,dc=com' '(uid=dave)' +
Enter LDAP Password: 
version: 1

dn: uid=dave,ou=Users,dc=example,dc=com
structuralObjectClass: inetOrgPerson
entryUUID: efbf8838-c734-102a-935c-57e457da105f
creatorsName: cn=Manager,dc=example,dc=com
createTimestamp: 20060823205147Z
pwdChangedTime: 20070121180110Z
entryCSN: 20070121182203Z#000000#00#000000
modifiersName: cn=Manager,dc=example,dc=com
modifyTimestamp: 20070121182203Z
entryDN: uid=dave,ou=Users,dc=example,dc=com
subschemaSubentry: cn=Subschema
hasSubordinates: TRUE
```

In such cases administrators do not have to make any special changes to a user's entry. But what if the user gets locked out? This can happen if `pwdLockDuration` is set to `0` and the user fails to login too many times. It can also happen, as we saw in the example, if the user's password has expired and the user has exhausted the allowed grace logins.

Once the account has been locked, the user will not even be allowed to change his or her password. That means that the manager will need to intervene on the user's behalf and change the password using `ldappasswd`, `ldapmodify`, or another similar tool.

In rare cases, it may be desirable to modify the operational attributes directly. For example, `pwdAccountLockedTime`, `pwdReset`, and `pwdPolicySubentry` can be modified by the manager:

```
$ ldapmodify -x -W -D 'cn=manager,dc=example,dc=com'
Enter LDAP Password: 

dn: uid=adam,ou=users,dc=example,dc=com
changetype: modify
add: pwdReset
pwdReset: TRUE

modifying entry "uid=adam,ou=users,dc=example,dc=com"
```

In this example, the `pwdReset` flag for the `uid=adam` account was set to `TRUE`. This will require the user to change the password the next time a bind is performed.

But SLAPD may not allow the other operational attributes to be modified by the standard LDAP modification. This is because the `ppolicy` schema sets the `NO-USER-MODIFICATION` flag on these schema definitions.

Can these operational attributes ever be modified? Using a special control, the **Relax Rules control** (formerly called ManageDIT), managers can change the values of operational parameters that usually do not allow such changes. However, the Relax Rules control is not yet officially released and is not enabled by default in OpenLDAP. We would have to build the development version of OpenLDAP to enable the control.

### Summary of ppolicy Operational Attributes

We have looked at a few more operational attributes that `ppolicy` can attach to a record used for bindng. Here's a list of all of the possible attributes along with a brief description of each:

*   `pwdChangedTime`: This contains a timestamp indicating when the password was last changed. There can only be one value for this attribute. Passwords in entries that do not have this attribute will never expire.
*   `pwdAccountLockedTime`: This attribute is added to an entry when the entry is locked. It contains a timestamp indicating at what time SLAPD marked the account as locked. We saw this used when a user failed to authenticate too many times in a row.
*   `pwdFailureTime`: A `pwdFailureTime` attribute value is added to a record every time a user tries to bind, but fails to supply the right password. A successful login clears all `pwdFailureTime` attributes.
*   `pwdGraceUseTime`: If a user's account has expired, and the policy allows grace logins, a new `pwdGraceUseTime` value will be added every time the user logs in with an expired password. Resetting the password clears all `pwdGraceUseTime` values.
*   `pwdHistory`: If password history tracking is turned on then every time a user changes passwords, the old password is stored in a `pwdHistory` attribute value. Only the number of password specified in the policy are retained in the history
*   `pwdPolicySubentry`: This attribute, which allows only one value, takes the DN of the password policy that this record should use. If this attribute is not found, SLAPD uses the default policy (as specified by the `ppolicy_default` directive in `slapd.conf`).
*   `pwdReset`: This attribute takes a boolean value. When a manager changes the password the flag is set to `TRUE`. If the policy also has `pwdMustChange` set to `TRUE` then the user will have to change her or his password on the next bind (using `ldappasswd`).

At this point we are done working with the Password Policy overlay. Next we will move on to create our own schema.

# Creating a Schema

Up to this point we have taken an in-depth look at schema definitions and then implemented a few overlays that made use of custom schemas. By now you should be comfortable working with and reading schemas. Here we are going to create our own schema.

Our goal in this section is to create a small schema for adding blog information to our directory. We want to be able to store a record in the directory to represent a blog, and also link existing entries to these blogs, indicating, for example, that a particular user maintains a particular blog.

To do this we are going to add two object classes—one structural and one auxiliary—and a handful of new attributes. The structural object class, `blog`, will describe an individual blog. It will contain the necessary attributes to describe a blog.

The auxiliary class `blogOwner`, will be used to add blog ownership information to a particular entry. Since the information about the blog will be stored in a `blog` entry, the `blogOwner` object class will only need one attribute that can be used to point to the appropriate `blog` entry.

The first thing we will do is walk through the process of obtaining an OID. Then we will create our object classes. After the object classes are created we will define our new attributes. Finally, we will try out our new schema.

## Getting an OID

As we have seen so far the OID (Object Identifier) plays an important role in defining a schema.

An OID is a sequence of integers separated by dots (`.`). But OIDs are not arbitrary combinations of digits. They are structured to represent the pedigree of an object. As we will use them here, for creating a new schema, we will treat the OID as being composed of three parts:

*   The base OID
*   The type number
*   The item number

The base part of an OID number is assigned by a naming authority. We will get ours from the **Internet Assigned Numbers Authority** (**IANA**).

### Note

**IANA** is not the only naming authority. Each country may have its own registry. For instance, in the United States the **American National Standards Institute (ANSI)** also has a registry.

IANA maintains a registry of OIDs for private enterprises. It allocates numbers free of charge and all that is necessary is a one-time registration. However, IANA only gives one number to each enterprise so, if your organization has one already, you should use the existing one. You can view the registry at [http://www.iana.org/assignments/enterprise-numbers](http://www.iana.org/assignments/enterprise-numbers).

To obtain a number go to [http://iana.org/cgi-bin/enterprise.pl](http://iana.org/cgi-bin/enterprise.pl) and complete the form there. You will then be assigned an OID looking something like this: 1.3.6.1.4.1.?, where the question mark is replaced with an integer. This OID serves as the basis for the OIDs we use when creating schemas. By appending your own series of digits and dots to this string you can create your own OID numbers, and as long as you take care to keep your OIDs unique within your own domain, you can assume that these OIDs are also globally unique (for you are the only one with the exact base OID).

### Note

In these examples I am using the OID registered to me. These OIDs may be used to replicate the examples herein, but do not use my OID to create your own schemas. The practice of using someone else's OID is called **OID hijacking**, and is frowned upon because it compromises the assumption that OIDs are globally unique.

While this series of digits has some semantic meaning (it means, roughly, that the owner is a private enterprise operating within IANA's namespace), there are no constraints on how you decide to structure your OIDs. You could, for example, just append a new set of random digits to the base OID each time you needed to create a new OID:

```
1.3.6.1.4.1.8254.78.45146762
1.3.6.1.4.1.8254.57.483729598
```

But it is often more manageable to come up with some semantic scheme for organization. A version derived from the OpenLDAP foundation's scheme is recommended. From the base OID, create a segment to be used just for LDAP OIDs:

```
1.3.6.1.4.1.8254.1021
```

Now we have just one portion of the namespace that will be used only for LDAP OIDs. From here we will use a simple subcategory identifier. Starting with the OID arc 1.3.6.1.4.1.8254.1021, we will create OIDs of the form:

```
1.3.6.1.4.1.8254.1021.x.y
```

Where `x` indicates the type of object and `y` indicates the specific object we are identifying. The OpenLDAP Foundation uses the following types:

*   LDAP syntaxes (`1`)
*   Matching rules (`2`)
*   Attribute types (`3`)
*   Object classes (`4`)
*   Supported features (`5`)
*   Protocol mechanisms (`9`)
*   Controls (`10`)
*   Extended operations (`11`)

We are only going to create object classes and attributes, so the value of `x` for our classes will be `3` for OIDs attached to attributes and `4` for OIDs attached to object classes.

For the `y` value, we will just start with the digit `1` and increment each time we define a new object of that type. For example, our first object class will have the OID:

```
1.3.6.1.4.1.8254.1021.4.1
```

And for our second object class we will just increment the last value from `1` to `2`:

```
1.3.6.1.4.1.8254.1021.4.2
```

Again, this is just one convention and different organizations use different conventions. While I advocate this convention you are free to choose another if you find that it is better for your needs.

There are two things to keep in mind though. First, you need to ensure that the OIDs are unique across your arc. That means you should maintain a registry of them in a place accessible to all people in your organization who work with the OIDs. Second, adding meaning to the numbers can provide tremendous utility, as it can help you recall or derive what an otherwise arbitrary string of numbers represents.

Now we are ready to begin creating our schema.

## Giving Our OID a Name

Our schema definitions are all going in a file called `blog.schema`, which we will later reference in an `include` statement in `slapd.conf`.

Most usually once the base OID for LDAP objects is defined, it is convenient to use the `objectidentifier` directive in `slapd.conf` to make the OIDs more readable, and make the process of creating schema definitions less error prone.

We can do this in the first few lines of our schema file:

```
objectidentifier blogSchema 1.3.6.1.4.1.8254.1021
objectidentifier blogAttrs blogSchema:3
objectidentifier blogOCs blogSchema:4
```

The first line maps the name `blogSchema` onto the OID `1.3.6.1.4.1.8254.1021`. Now we can refer to that long OID as `blogSchema`, which is much easier to remember.

The second and third `objectidentifier` directives add a few more aliases. The second one sets the name `blogAttrs` refer to the OID `blogSchema:3` (which is `1.3.6.1.4.1.8254.1021.3`). Thus, when we define attributes we can use the shortcut `blogAttrs:1` instead of typing the whole thing out as `1.3.6.1.4.1.8254.1021.3.1`.

Similarly, `blogOCs` alias (short for "blog object classes") can be used to refer to the `1.3.6.1.4.1.8254.1021.4` arc.

With this mechanism in place we have implemented the organizational strategy explained in the previous section, and our OID naming from here on should be a simple matter of incrementing the last integer of an OID.

## Creating Object Classes

We will be starting with our object classes, and then use these defined object classes to guide the creation of our attributes. This is typically the way creation of schemas is done, but it does have one counter-intuitive result: object classes must be defined after the attributes that they contain. In effect then, we are jumping to the end of our schema file to add object classes, and will later add attribute definitions between the object identifiers and the object classes.

The first object class to describe is the `blog` class. This object class will define the attributes necessary to define a blog. For our purposes we are going to create a very simple object class, though there are many more attributes that could be attached.

We want the class to have the following attributes:

*   `blogTitle`: The title of the blog
*   `blogUrl`: The URL (Uniform Resource Locater) of the main page for the blog
*   `blogFeedUrl`: The URL for the RSS or Atom feed of the URL
*   `description`: A brief text description of the blog

Of these, the `blogUrl` and `blogTitle` attributes should be required. `blogUrl` is an essential component of a blog. Without this, an entry describing a blog would be of little value. And the `blogTitle` attribute is necessary to give us a naming component to use in DNs.

For the sake of clarity of meaning, here we have prepended the `blog` string to any new attributes so that they can be immediately distinguished from other similar attributes.

### Tip

**Naming Object Classes and Attributes**

If your object classes or attributes are designed for internal use, or for application-specific use, it is advised that the name of the organization or application be prepended to the attribute and object class names. That helps to make the purpose of the defined items explicit.

Fortunately for us, `description` is already defined. While we could use the `title` attribute, as defined in `core.schema`, this could introduce confusion, as that attribute is used to refer to the title of a person in an organization. To avoid any confusion then, we will avoid reusing that attribute.

Already we have said that this object class is going to be structural, and we have a scheme for determining an OID number. There are no similar object classes so we will create a class whose superior is `top`. We now have all the information we need to create our schema definition:

```
objectclass 
  (
   blogOCs:1
   NAME 'blog'
   DESC 'Describes an online blog accessible by URL.'
   SUP top
   STRUCTURAL
   MUST ( blogUrl $ blogTitle )
   MAY ( blogFeedUrl $ description )
  )
```

In the OID field we used the object identifier we assigned in the last section. And we started with `1`, our first object class.

The `blogOwner` object class is to be marked auxiliary so that we can attach it to a variety of different entries, regardless of the structural object class. For example, regardless of whether the blog is a corporate blog, or is maintained by an organizational unit, or is simply an individual's, we can add this object class to the desired entry.

We want to use the `blogOwner` object class to insert a pointer from an entry to the appropriate `blog` entry in the directory information tree. Since that is all we need, a single attribute will suffice for these purposes:

*   `blogDN`: The DN describing the `blog` that this entry is affiliated with.

This object class then, turns out to be even simpler than the previous one:

```
objectclass 
  (
   blogOCs:2
   NAME 'blogOwner'
   DESC 'Indicates that this entry is responsible for a blog.'
   AUXILIARY
   MUST ( blogDN )
  )
```

This OID number differs from the first only in that the last value has been incremented. This follows the scheme we defined in the previous section.

Since this is an auxiliary object class, there is no need for a superior. And since we want this class to be used to point to a `blog` entry elsewhere in the directory, the `blogDN` attribute is required.

Now we have our two object classes. In creating them we have referred to four attributes that currently do not exist. It is time to create them.

## Creating Attributes

As we created the `blog` and `blogOwner` object classes, we tentatively defined (in our text) four attributes: `blogTitle`, `blogUrl`, `blogFeedUrl`, and `blogDN`. Now we will define each of these, beginning with `blogTitle`.

In order to define our attribute we want to decide on the syntax of the attribute and also the matching rules that SLAPD will use for this attribute. The `blogTitle` will contain values that are strings of text data. So the syntax we want is one that supports this. The **Directory String syntax**, defined in RFC 4517, is intended for just such a purpose. And it supports internationalization, storing characters in UTF-8.

When performing searches, we do not want the case of the text (upper or lower) to make a difference. In other words, we want "My Blog" and "my blog" to be treated as matches. So we need to find the matching rule that will best support this. There are over three dozen matching rules supported in OpenLDAP (you can see a list by searching the `cn=Subschema` entry). We want to implement string-based equality and substring matching on our `blogTitle` attribute, so the pair of matching rules we will want to use are `caseIgnoreMatch` and `caseIgnoreSubstringsMatch`.

Now, we have all of the information necessary for creating a new attribute type:

```
attributetype 
  (
   blogAttrs:1
   NAME 'blogTitle'
   DESC 'Title of a blog.'
   EQUALITY caseIgnoreMatch
   SUBSTR caseIgnoreSubstringsMatch
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{256}
  )
```

The OID field is `blogAttrs:1`, indicating that this is our first attribute.

The LDAP syntax OID is the OID for a **Directory String**. At the end of the OID, the `{256}` suggests that the maximum length of the title be constrained to 256 characters.

### Note

The characters are in UTF-8, so this might take up as much as 512 bytes of space if each of the 256 characters is two bytes.

The next two attributes, `blogUrl` and `blogFeedUrl`, are similar and we can take advantage of that as we define them.

The first thing to examine is the LDAP syntax of these attributes. Unlike `blogTitle`, we do not want the values of `blogUrl` and `blogFeedUrl` to be in the Directory String syntax, because (according to RFC 3986 and the previous URL standards) URLs are to use a subset of the ASCII character set.

### Note

For more on URLs and internationalization, see the W3C's *Web* *Naming* *and* *Addressing* page: [http://www.w3.org/Addressing/](http://www.w3.org/Addressing/). Links to information as well as pertinent RFCs can be found there.

Instead of using Directory String syntax, we should use the **IA5 String syntax** which describes an extended ASCII character set. The OID for this syntax is `1.3.6.1.4.1.1466.115.121.1.26`.

Similarly, when we specify matching rules, we want to use the IA5 matching rules. And since URLs are case-sensitive, we want exact matches. We do not want the case to be ignored. So for matching rules we want `caseExactIA5Match` and `caseExactIA5SubstringsMatch`.

Now we can define both attributes:

```
attributetype 
  (
   blogAttrs:2
   NAME 'blogUrl'
   DESC 'Uniform Resource Locator (URL) for a blog.'
   EQUALITY caseExactIA5Match
   SUBSTR caseExactIA5SubstringsMatch
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{512}
  )

attributetype 
  (
   blogAttrs:3
   NAME 'blogFeedUrl'
   DESC 'URL to an XML feed for a blog.'
   SUP blogUrl
  )
```

Since the `blogUrl` field contains the matching rules and syntax that `blogFeedUrl` uses, and since there is an obvious similarity in usage between the two, it makes sense to treat `blogUrl` as the supertype of `blogFeedUrl`. So, `blogFeedUrl` inherits the LDAP syntax and matching rules from `blogUrl`.

Finally, we need to define our `blogDN` field, which will hold a DN. There is syntax and specific matching rules for DNs, and we will use those. The **Distinguished Name syntax**, defined with the OID `1.3.6.1.4.1.1466.115.121.1.12`, is used for values that are DNs. And the `distinguishedNameMatch` matching rule is used for performing exact matches on DNs. There are no substring or ordering matches for DNs.

Our last attribute then, looks like this:

```
attributetype 
  (
   blogAttrs:4
   NAME 'blogDN'
   DESC 'DN of a blog entry in the directory.'
   EQUALITY distinguishedNameMatch
   SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 
  )
```

Now we have our entire schema defined. We are ready to test it.

## Loading the New Schema

As with all other schemas, in order to load this schema, we must include it in `slapd.conf`.

```
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/ppolicy.schema
include /etc/ldap/schema/blog.schema

```

It is assumed here that `blog.schema` is located in the `/etc/ldap/schema` directory (which is a good place to put the schema). If you choose to locate the schema elsewhere, adjust the path accordingly.

The highlighted line in the code is the only addition necessary (the rest should be there already). Note that our schema is only dependent on `core.schema`. The other three are not necessary to make our schema work.

Restarting SLAPD will load the schema.

### Troubleshooting Schema Loading

If there is an error in the schema SLAPD will not start, failing instead with an elaborate error message like this:

```
/etc/schema/blog.schema: line 89: Unexpected token before 
                         MUST ( blogDN ) )
ObjectClassDescription = "(" whsp
   numericoid whsp  ; ObjectClass identifier
   [ "NAME" qdescrs ]
   [ "DESC" qdstring ]
   [ "OBSOLETE" whsp ]
   [ "SUP" oids ]  ; Superior ObjectClasses
   [ ( "ABSTRACT" / "STRUCTURAL" / "AUXILIARY" ) whsp ]
                   ; default structural
   [ "MUST" oids ]  ; AttributeTypes
   [ "MAY" oids ]  ; AttributeTypes
   whsp ")"
slapd stopped.
connections_destroy: nothing to destroy.
```

This error was triggered when we misspelled `AUXILIARY`—a cause not easily divined by this error message. But it illustrates the fact that the process of writing a schema definition takes patience and precision.

The best strategy for dealing with such failures is to carefully read the errant schema definition over, hunting for errors. Sometimes simplifying a definition can help eliminate other possible errors too. Finally, checking the definition against the specification in RFC 4512 can help you spot any nondescript syntactical errors.

## A New Record

Now we can use `ldapadd` to add a new `blog` entry to our directory information tree. We will add information about the official corporate blog of Example.Com:

```
$ ldapadd -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: blogTitle=Example.Com News,dc=example,dc=com
blogTitle: Example.Com News
blogUrl: http://example.com/blogs/main
blogFeedUrl: http://example.com/rss/main
description: The Official Example.Com Blog.
objectclass: blog

adding new entry "blogTitle=Example.Com News,dc=example,dc=com"
```

The highlighted portion above is the new entry we are adding. The last line, returned by SLAPD, indicates that the entry has been added successfully.

Our user `uid=barbara` is responsible for maintaining this blog so we can indicate this relationship by adding the `blogOwner` object class and `blogDN` attribute to her record with `ldapmodify`:

```
$ ldapmodify -U matt
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers

dn: uid=barbara,ou=users,dc=example,dc=com
changetype: modify
add: objectclass
objectclass: blogOwner
-
add: blogDN
blogDN:  blogTitle=Example.Com News,dc=example,dc=com

modifying entry "uid=barbara,ou=users,dc=example,dc=com"
```

The record for `uid=barbara` now looks like this:

```
dn: uid=barbara,ou=Users,dc=example,dc=com
ou: Users
uid: barbara
sn: Jensen
cn: Barbara Jensen
givenName: Barbara
displayName: Barbara Jensen
mail: barbara@example.com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: labeledURIObject
objectClass: blogOwner
blogDN: blogTitle=Example.Com News,dc=example,dc=com

```

We have just successfully created and implemented a new schema including new attributes and object classes.

# Summary

The focus of this chapter has been the schema. We began with a theoretical look at what makes up a schema and how schemas are defined. Then we looked at the organization of schemas in the directory, focusing on the different types of object class and how they work together to compose a hierarchical directory. From there we turned to more practical material. We looked at the `accesslog` and `ppolicy` overlays, each of which requires its own schema. Finally, we ended by creating our own custom schema, creating a pair of object classes, and a handful of attributes.

In the next chapter we will discuss working with multiple directories, focusing particluarly on directory replication, the process of keeping two or more directory servers synchronized with the same content.