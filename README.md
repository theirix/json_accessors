JSON accessor functions for PostgreSQL
======================================

Extension provides stored functions for accessing [JSON](http://www.json.org/) fields by keys and converting JSON arrays.

This project contains PostgreSQL [extension](http://www.postgresql.org/docs/9.1/static/extend-extensions.html) `json_accessors` with stored functions. Extension is native and writen in C on top of [cJSON](http://sourceforge.net/projects/cjson/) library.

PostgreSQL have had no JSON support until version 9.2, which [introduced some support](http://www.postgresql.org/docs/9.2/static/functions-json.html).
These 9.2 functions won't help with indexing JSON data.

JSON parsing functions may be written using [PL/V8](http://code.google.com/p/plv8js/wiki/PLV8) module,
[this article](http://people.planetpostgresql.org/andrew/index.php?/archives/249-Using-PLV8-to-index-JSON.html) has an example of PL/V8 usage.
This project provides accessor functions for JSON without using PL/V8.

Usage
-----

Please consult with [doc/json_accessors.md](doc/json_accessors.md) for a function reference.


Installing extension
--------------------

### Building and installing extension with PGXS

C extension are best built and installed using [PGXS](http://www.postgresql.org/docs/9.1/static/extend-pgxs.html).
PGXS ensures that make is performed with needed compiler and flags. You only need GNU make and a compiler to build
an extension on an almost any UNIX platform (Linux, Solaris, OS X).

Compilation:

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config

Installation (as superuser):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config install

PostgreSQL server must be restarted and extension created in particular database as superuser:

    create extension json_accessors

To drop all functions use:

    drop extension json_accessors cascade

To uninstall extension completely you may use this command (as superuser):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config uninstall

Project contains SQL tests that can be launched on PostgreSQL with installed extension.
Tests are performed on a dynamically created database with a specified user (with the 
appropriated permissions - create database, for example):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config PGUSER=postgres installcheck


### Installing manually

Use this method if you have a precompiled extension and do not want to install this with help of PGXS.
Or maybe you just do not have GNU make on a production server.

Copy library to the PostgreSQL library directory:

    cp json_accessors.so `<postgresql_install_dir>/bin/pg_config --pkglibdir` 

Copy control file to the extension directory:
    
    cp json_accessors.control `<postgresql_install_dir>/bin/pg_config --sharedir`/extension

Copy SQL prototypes file to the extension directory:
    
    cp json_accessors--1.3.0.sql `<postgresql_install_dir>/bin/pg_config --sharedir`/extension

Create an extension by running:

    create extension json_accessors

It creates all the accessors functions. Note that you must restart a server if a previous library was
already installed at the same place.

To drop all functions use:

    drop extension json_accessors cascade

To uninstall extension just remove files you copied before.


License information
-------------------

You can use any code from this project under the terms of [PostgreSQL License](http://www.postgresql.org/about/licence/).
Please consult with the COPYING for license information.
