# Appendix B. LDAP URLs

To query a directory a client must send the server several different pieces of information. To make it possible to group all of this information together into one standards-based string format, LDAP developers proposed a standard LDAP URL syntax, which follows the URL standard (RFC 3986). In this appendix we will take a look at the format of LDAP URLs.

# The LDAP URL

The LDAP URL is composed of eight different parts:

1.  The **protocol**, which is usually LDAP (`ldap://`), though the non-standard LDAPS protocol (`ldaps://`) is used.
2.  The **domain name** (or IP address) of the server. The default is `localhost`.
3.  The **port number** of the server. The default is the standard LDAP port, `389`.
4.  The **base DN** for the search.
5.  The list of **attributes** to be returned. The default is to return all the attributes.
6.  The **scope** specifier. The default is to use the `base` scope.
7.  The **search filter**. The default is `(objectclass=*)`.
8.  The **extension** field. If the server supports extensions, parameters for those extensions can be passed in the last field.

Combining seven of the eight parts (we will skip the extension field) we can create a URL that looks something like this:

```
ldap://example.com:389/ou=Users,dc=example,dc=com?mail?sub?(uid=matt)
```

This URL is composed of the seven parts in this way:

```
<protocol>://<domain>:<port>/<basedn>?<attrs>?<scope>?<filter>
```

Where we have to use an extension we would simply append a question mark `(?`) and the extension information to the end of the given URL.

Using this URL to perform an LDAP search, the result would be as follows:

*   The client would connect to Example.Com on port 389 using the LDAP protocol.
*   The based DN would be set to `ou=Users,dc=example,dc=com`.
*   The client would request the **mail** attributes for all the entries in the subtree of `ou=Users,dc=example,dc=com` where the UID was `matt`.

To use LDAPS (the non-standard practice of using LDAP over a dedicated SSL/TLS port), use `ldaps://` instead of `ldap://`.

In many cases it is convenient to shorten the URL and accept the default options. For example, the default domain is `localhost` (or the IP address `127.0.0.1`), the address of the server on which the URL is executed. And the default port is 389 (unless the protocol is `ldaps://` instead of `ldap://`, in which case the default port is the LDAP port 636).

The port can be left off in most cases. But the domain portion of the URL can be omitted too:

```
ldap:///ou=Users,dc=example,dc=com?mail?sub?(uid=matt)
```

Note that there are now three slashes at the beginning, `ldap:///`. The domain name, which normally appears between the second and third slash, is not specified. If this URL were used, the LDAP application would connect to the localhost (the default host) at port 389 (the default LDAP port), and then proceed to run the search.

Now let's say that instead of wanting the LDAP server to return just the `mail` attribute, we want it to return all of the standard (non-operational) attributes. To do this, we simply leave the attribute specification empty:

```
ldap:///ou=Users,dc=example,dc=com??sub?(uid=matt)
```

Now, the attribute position has no value, though the two adjacent question marks (`??`) indicate where the empty attribute position is.

In the previous two examples, when we have omitted specific field values, we have had to leave the designators in the URL, so we have `ldap:///` for the domain portion of the URL, and `?` without a value for the attribute specification (which looks like `??` in the given example).

But when we drop values from the *end* of the URL we do not need to leave the empty position designators. For example, if we were to drop the filter from the end, we do not need to leave trailing `?` at the end of the URL. Here's an example:

```
ldap:///ou=Users,dc=example,dc=com?mail?sub
```

In this example the `mail` attributes for every entry under `ou=Users,dc=example,dc=com` are returned.

# Common Uses of LDAP URLs

Throughout this book LDAP URLs have been used for various purposes.

In Chapter 4 we used LDAP URLs to perform searches in the `authz-regexp` directive in `slapd.conf`.

While a full LDAP URL, as we examined, can be a useful way to formulate a search, this is probably not the primary use of LDAP URLs. More commonly the LDAP URL syntax is simplified and used to capture only basic information.

## Not all LDAP URLs are for Searching

In Chapter 3 we used LDAP URLs to connect to SLAPD from the `ldapsearch` utility, but we were not using the LDAP URL as a way to specify a search string. In many cases in fact, an LDAP URL may be used simply to provide protocol, host, and port information in one convenient string:

```
ldap://example.com:646
```

In this example the LDAP URL provides sufficient information for a client to use the plain LDAP protocol when connecting to the server `Example.Com` on the non-standard port 646.

Directory referrals, handled in the `slapd.conf` file by the referral directive, also use LDAP URL syntax, but only use the protocol, domain, and port settings.

LDAP URLs then, are used for two main purposes, and the purpose of each determines the form:

*   LDAP search URLs follow the sophisticated eight-field format, and can convey all the information needed for an LDAP agent to perform a search
*   LDAP connection URLs utilize only protocol, host, and port information, and are used mainly to convey information about how to connect to a directory

There are currently no LDAP URL forms for modifying or deleting LDAP records.

# For More Information on LDAP URLs...

The LDAP URL format is described in the standards-track RFC 4516\. The RFC is loaded with examples, and covers the use of extensions and encoding of special characters. The RFC is available online at [http://rfc-editor.org/rfc/rfc4516.txt](http://rfc-editor.org/rfc/rfc4516.txt).

# Summary

This brief primer provides an overview of the LDAP URL syntax. LDAP URLs are used in a variety of contexts, to provide connection information, and sometimes (in their more sophisticated form) to provide information necessary for performing an LDAP search.