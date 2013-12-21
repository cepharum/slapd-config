

OpenLDAP started to use runtime configuration stored in LDAP tree to completely replace previous configuration file(s). By intention this is preferred to support configuring LDAP service with no downtime thus also referred to as “runtime configuration”. While intentions are clear, there seem to be little tools other than ldapmodify for managing cn=config.
cn=config the easy way

slapd-config to the rescue. After several days of developing and testing I’m introducing a bash script called slapd-config for managing subset of features controlled by cn=config here.

    cn=config is the suffix for a thread of your OpenLDAP based tree using special backend driver.

slapd-config is a wrapper around client tools such as ldapsearch and ldapmodify trying to simplify access on cn=config. You don’t need to manage LDIF yourself to update access rules of your server or switching log level. The script has been developed and tested and Debian Squeeze and thus strongly relies on Debian-style authentication for accessing cn=config by using SASL mechanism EXTERNAL. This way there is no need to preconfigure a bind DN or password for accessing cn=config, but invoke the script as root.

slapd-config implements different “modules” each providing several “actions”. Current state of development is putting focus on access rule management and schema management. In addition basic modification of existing configuration properties is supported as well. Again, if you’ve set up your LDAP server requiring bind for managin cn=config, this script doesn’t work out-of-box for you.
Installing

The script is distributed under terms of GPLv3. Thus you can use it free of any charge, however it comes without any warranty or liability for damages to your setup and/or data.



# Usage

The script includes a very brief help system listing supported modules and their actions as well as providing basic syntax of using selected actions. Common help and module list is available by:

```
slapd-config help
```

Module-related help is available by inserting a module’s name as first argument like so:

```
slapd-config db help
```

This is listing all actions available in context of module “db”. Since most actions require additional arguments invoking a action without its required parameter results in another short note on how to invoke that action actually, e.g.

```
slapd-config db read-access
```


## Module db

The module db is available for listing existing databases and for managing access rules per database.

```
slapd-config db list
```

This is listing all backends of your LDAP server managing actual thread of data inside your tree. Thus it’s excluding databases such as cn=config itself. The list contains name, suffix of managed thread and DN of its runtime configuration for every database.

```
slapd-config db list-all
```

In addition to normal database listing this one is including all other databases as well, such as cn=config.

```
slapd-config db show hdb
```

Action “show” is available to get all attributes of a single database quite easily.

```
slapd-config db read-access hdb
```

This action is extracting all access rules of selected database dropping any LDIF related extra code. The output can be used with a text editor for revisiting all access rules in a single step for later write-back into runtime configuration.

```
slapd-config db write-access hdb
```

Serving as counterpart to read-access before, this action is expecting output of “read-access” or whatever you have modified in it on stdin replacing all existing access rules of selected database.

```
slapd-config db insert-access hdb 3 "to attrs=uid by * read stop"
```

If you don’t want to edit whole set of access rules this action is a convenient way of inserting another rule into your existing set of access rules. Starting with 1 every access rule is numbered and thus can be addresses here to select insertion of another rule right before it. The example above is inserting rule at position 3, thus turning previous rule at third position into fourth rule. Any rule number is bound to valid range 1 to number of rules, thus providing a very large rule number can be used to safely append another rule at end of set.

```
slapd-config db delete-access hdb 3
```

Again, here is the counterpart to a previously mentioned action. This time it’s about deleting single access rule in set of rules. And here selected rule number has to exist.


## Module schema

Schema files were previously used to declare entities available in your LDAP tree. Sometimes it’s required to have extra schema definitions for supporting extended sets of attributes e.g. to manage user accounts of an Active Directory using an OpenLDAP server. This module tries to simplify schema-related management.

```
slapd-config schema list
```

This action is returning list of schemata previously added to server. Every schema is described by it’s short name, e.g. inetorgperson, core or cosine.

```
slapd-config schema exists samba
```

Action “exists” tests whether some schema exists or not. The example is testing whether samba schema has been imported recently so extended Samba/PDC attributes are supported in LDAP.

```
slapd-config schema read cosine
```

If you want to edit/backup a single schema previously imported into your LDAP server this action will read out all rules of a schema writing them to stdout in a format you might be familiar with when working with schema files. Since comments are never imported into LDAP tree, there aren’t any comments in schema file exported this way.

```
slapd-config schema write cosine
```

As a counterpart to “schema read” this action is writing back/restoring a schema definition file dropping any comments found in file. The schema is provided on stdin. If selected schema didn’t exist before it’s added last to the list of existing schemata. Otherwise existing schema is replaced completely by provided one.

```
slapd-config schema delete cosine
```

Though supported as an action here, deleting schema isn’t supported by OpenLDAP and it doesn’t seem to be supported in near future for issues related to data integrity.


## Module raw

Providing raw and very basic access to cn=config the module raw offers access on methods used internally by slapd-config to provide all operations described before. Most of its actions are related to navigating in tree selecting some entry and accessing it’s attributes for read/write.

```
slapd-config raw read ou=people,dc=acme,dc=com objectClass
```

This is reading out all values of an attribute (objectClass here) unfolding longer lines and dropping any extra markup.

```
slapd-config raw write ou=people,dc=acme,dc=com myAttribute
```

Having read multiple values distributed on separate lines this action is managing to update values of an entry’s single attribute supporting automatic numbering.

```
slapd-config raw insert ou=people,dc=acme,dc=com myAttribute 3 "another value"
```

This action is inserting another value to single attribute of selected entry. Every value of an attribute is virtually numbered starting with 1. Selecting 3 means to insert new value after 2nd and before 3rd value. If attribute of entry has less than 3 values the new value is appended.

```
slapd-config raw delete ou=people,dc=acme,dc=com myAttribute 3
```

Drops a single value in a set of values assigned to attribute of entry selected by its DN.

```
slapd-config raw list cn=config
```

Lists DNs of entries subordinated to selected one. By default DNs of all subordinated entries are listed. By providing additional option “one” direct children of selected entry are listed, only.

```
slapd-config raw show cn=config
```

Renders all attributes of entry selected by its DN.
