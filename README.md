JSON accessor functions for PostgreSQL
======================================

[PostgreSQL](http://www.postgresql.org/) stored functions for accessing [JSON](http://www.json.org/) fields.

This project contains PostgreSQL [extension](http://www.postgresql.org/docs/9.1/static/extend-extensions.html) `json_accessors` with stored functions. Extension is native and writen in C on top of [cJSON](http://sourceforge.net/projects/cjson/) library.

If you have text (varchar) columns with data like this:

    {"create_date":"2009-12-01 01:23:45","tags":["foo","bar","baz"]}

These functions can be used for:

 - creating queries to JSON object fields
 - creating B-tree (default) indexes on JSON object fields (`create_date` field)
 - creating [GIN](http://www.postgresql.org/docs/9.1/static/gin.html) indexes on JSON arrays (`tags` field)

PostgreSQL have had no JSON support until version 9.2, which [introduced some support](http://www.postgresql.org/docs/9.2/static/functions-json.html).
These 9.2 functions won't help with indexing JSON data.

JSON parsing functions may be written using [PL/V8](http://code.google.com/p/plv8js/wiki/PLV8) module,
[this article](http://people.planetpostgresql.org/andrew/index.php?/archives/249-Using-PLV8-to-index-JSON.html) has an example of PL/V8 usage.
This project provides accessor functions for JSON without using PL/V8.

Functions
---------

__Function for accessing JSON object fields:__

    function json_get_text(text, text) returns text

Usage example, returns `qq`:

    select json_get_text('{"foo":"qq", "bar": true}', 'foo')

There are also similar functions returning `boolean`, `int`, `bigint`, `numeric` and `timestamp without timezone`.
Timestamp format `yyyy-MM-dd HH:mm:ss` is fixed.

To access complex JSON object fields you can use:

    function json_get_object(text, text) returns text

It extractc child JSON object and returns it as text.
Usage example, returns `{"boo":42}`:

    select json_get_object('{"foo":{"boo":42}, "bar": true}', 'foo')

To access JSON object fields, that contain arrays, there are functions for different array types
(including arrays of objects and multidimensional arrays), this example returns `array[42,43]`:

    json_get_int_array('{"boo": [42, 43]}', 'boo')

Arrays with different element types are not supported

__Function for converting JSON arrays into PostgreSQL arrays:__

    function json_array_to_text_array(text) returns text[]

Usage example, returns `array['foo', 'bar']`:

    select json_array_to_text_array('["foo", "bar"]')

There are also similar functions returning `boolean[]`, `int[]`, `bigint[]`, `numeric[]` and `timestamp without timezone[]`.
All primitive arrays returns from Java functions in boxed form (`Boolean[]` etc.) to allow returning `NULL` elements.
Having nulls in such arrays is not a good idea, but "Cannot assign null to int" errors in stored functions are worse.
Functions for arrays of objects and multidimensional arrays return `text[]`.


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
    
    cp json_accessors--1.3.sql `<postgresql_install_dir>/bin/pg_config --sharedir`/extension

Create an extension by running:

    create extension json_accessors

It creates all the accessors functions. Note that you must restart a server if a previous library was
already installed at the same place.

To drop all functions use:

    drop extension json_accessors cascade

To uninstall extension just remove files you copied before.


License information
-------------------

You can use any code from this project under the terms of [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0).
