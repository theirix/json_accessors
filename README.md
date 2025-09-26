JSON accessor functions for PostgreSQL
======================================

Extension provides stored functions for accessing [JSON](http://www.json.org/) fields by keys and converting JSON arrays.

This project contains PostgreSQL [extension](https://www.postgresql.org/docs/current/extend-extensions.html) `json_accessors` with stored functions. Extension is native and writen in C on top of modified [cJSON](http://sourceforge.net/projects/cjson/) library (very old version).

Extension supports PostgreSQL 9.1 through 18 (and probably even higher versions).

PostgreSQL have had no JSON support until version 9.2, which [introduced some support](http://www.postgresql.org/docs/9.2/static/functions-json.html).
These 9.2 functions won't help with indexing JSON data.

JSON parsing functions may be written using [PL/V8](http://code.google.com/p/plv8js/wiki/PLV8) module,
[this article](http://people.planetpostgresql.org/andrew/index.php?/archives/249-Using-PLV8-to-index-JSON.html) has an example of PL/V8 usage.
This project provides accessor functions for JSON without using PL/V8.


Usage
-----

Please consult with [`doc/json_accessors.md`](doc/json_accessors.md) for a function and operator reference.

On PGXN please click on extension from _Extensions_ section to view reference.


Installing extension
--------------------

To use an extension one must be built, installed into PostgreSQL directory
and registered in a database.

### Building extension

#### Using PGXN network

The easisest method to get and install an extension from PGXN network.
PGXN client downloads and builds the extension.

    pgxn --pg_config <postgresql_install_dir>/bin/pg_config install json_accessors

PGXN client itself is available at [github](https://github.com/dvarrazzo/pgxnclient) and
can be installed with your favourite method, i.e. `easy_install pgxnclient`.

#### Using PGXS makefiles

C extension are best built and installed using [PGXS](https://www.postgresql.org/docs/current/extend-pgxs.html).
PGXS ensures that make is performed with needed compiler and flags. You only need GNU make and a compiler to build
an extension on an almost any UNIX platform (Linux, Solaris, OS X). 

Compilation:

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config

Installation (as superuser):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config install

PostgreSQL server must be restarted. 

To uninstall extension completely you may use this command (as superuser):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config uninstall

Project contains SQL tests that can be launched on PostgreSQL with installed extension.
Tests are performed on a dynamically created database with a specified user (with the 
appropriated permissions - create database, for example):

    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config install
    gmake PG_CONFIG=<postgresql_install_dir>/bin/pg_config PGUSER=postgres installcheck

#### Manually

Use this method if you have a precompiled extension and do not want to install this with help of PGXS.
Or maybe you just do not have GNU make on a production server.
Or if you use Windows (use MSVC 2008 for Postgres 9.1 and MSVC 2010 for Postgres 9.2).

Copy library to the PostgreSQL library directory:

    cp json_accessors.so `<postgresql_install_dir>/bin/pg_config --pkglibdir` 

Copy control file to the extension directory:
    
    cp json_accessors.control `<postgresql_install_dir>/bin/pg_config --sharedir`/extension

Copy SQL prototypes file to the extension directory:
    
    cp json_accessors--<version>.sql `<postgresql_install_dir>/bin/pg_config --sharedir`/extension

To uninstall extension just remove files you copied before.

### Creating extension in a database

Extension must be previously installed to a PostgreSQL directory.

Extension is created in a particular database (as superuser):

    create extension json_accessors;

It creates all the functions, operators and other stuff from extension.
Note that you must restart a server if a previous library was already installed
at the same place. In other words, always restart to be sure. 

To drop an extension use:

    drop extension json_accessors cascade;


License information
-------------------

You can use any code from this project under the terms of [PostgreSQL License](http://www.postgresql.org/about/licence/).

Please consult with the COPYING for license information.
