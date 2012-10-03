json_accessors
==============

Installing
----------

    CREATE EXTENSION json_accessors;

Extension is compatible witgh PostgreSQL 9.1.

Description
-----------

Extension provides stored functions for accessing [JSON](http://www.json.org/) fields by keys and converting JSON arrays.

Usage
-----

Extension has two usage scenarios:

1. Extract a json object by key:

        select json_get_text('{"create_date":"2009-12-01 01:23:45","key":"foobar"}', 'key') = 'foobar';
        select json_get_int('{"bar": 42 }', 'key') = 42;

2. Convert an JSON array to SQL array:

        select json_array_to_text_array('["foo", "bar", "baz"]') = array['foo','bar','baz'];

Functions can be also used for:

 - creating queries to JSON object fields
 - creating B-tree (default) indexes on JSON object fields 
 - creating [GIN](http://www.postgresql.org/docs/9.1/static/gin.html) indexes on JSON arrays

Interface
---------

All functions takes a JSON string as a first argument.
If a function should extract an object by the key, second
argument is a key.
Functions usually fails when actual object is not of requested type.

### Scalar extraction functions ###

#### ``json_get_object(text, text) -> text``

Extracts a child JSON object as a string.

Example:

        select json_get_text('{"foo": 42, "key":"foobar"]}', 'key') = 'foobar';

#### `json_get_text(text, text) -> text`

Extracts and returns a child JSON text object converted to PG `text`. 

Example:

        select json_get_text('{"key":"foobar"]}', 'key') = 'foobar';

#### `json_get_boolean(text, text) -> boolean`

Extracts and returns a child JSON boolean object converted to PG `boolean`.

Example:

        select json_get_boolean('{"key": true]}', 'key');

#### `json_get_int(text, text) -> int`

Extracts and returns a child JSON integer object converted to PG `int` (`int4`). 

Example:

        select json_get_int('{"key": 42]}', 'key') = 42;

#### `json_get_bigint(text, text) -> bigint`

Extracts and returns a child JSON integer object converted to PG `bigint` (`int8`). 

Example:

        select json_get_bigint('{"key": 42]}', 'key') = 42;

#### `json_get_numeric(text, text) -> numeric`

Extracts and returns a child JSON integer object converted to PG `numeric`. 

Example:

        select json_get_numeric('{"key": 42.99]}', 'key') = 42.99;

#### `json_get_timestamp(text, text) -> timestamp`

Extracts and returns a child JSON text object converted to PG `timestamp without timezone`. 
Timestamp format `YYYY-MM-DD HH:MI:SS` is fixed.

Example:

        select json_get_timestamp('{"foo":"qq", "bar": "2009-12-01 01:23:45"}', 'bar') = timestamp('2009-12-01 01:23:45');



### Array extractor functions ###

Functions convert JSON arrays to PostgreSQL arrays.

#### `json_array_to_object_array(text) -> text[]`

Converts a JSON array of any JSON objects to PG array `text[]`. Each object is represented as a string.

Example:

        select json_array_to_object_array('[{"foo":42}, {"bar":[]}]') = array['{"foo":42}','{"bar":[]}']


#### `json_array_to_text_array(text) -> text[]`

Converts a JSON array of text objects to PG array `text[]`. 

Example:

        select json_array_to_text_array('["foo", "bar"]') = array['foo','bar'];

#### `json_array_to_boolean_array(text) -> boolean[]`

Converts a JSON array of boolean objects to PG array `boolean[]`. 

#### `json_array_to_int_array(text) -> int[]`

Converts a JSON array of integer objects to PG array `int[]` (`int4[]`)

#### `json_array_to_bigint_array(text) -> bigint[]`

Converts a JSON array of integer objects to PG array `bigint[]` (`int8[]`) 

#### `json_array_to_numeric_array(text) -> numeric[]`

Converts a JSON array of integer objects to PG array `numeric[]`. 

#### `json_array_to_timestamp_array(text) -> timestamp without time zone[]`

Converts a JSON array of text objects to PG array `timestamp[]` without time zones.
Time format is the same.


### Indirect array exractor functions ###

Shortcut functions to directly extract an array by the key.
Could be emulated by `json_get_object` and array convertor functions.
Array is referenced in a JSON expression by a key.

#### `json_get_object_array(text, text) -> text[]`

Extract and converts a JSON array of any JSON objects to PG array `text[]`. JSON objects are represented as a text.

Example:

        select json_get_object_array('{"key" : [{"foo":42}, {"bar":[]}]}', 'key') = array['{"foo":42}','{"bar":[]}'];

#### `json_get_text_array(text, text) -> text[]`

Extract and converts a JSON array of text objects to PG array `text[]`. 

Example:

        select json_get_text_array('{"foo":"qq", "bar": ["baz1", "baz2", "baz3"]}', 'bar') = array['baz1','baz2','baz3'];

#### `json_get_boolean_array(text, text) -> boolean[]`

Extract and converts a JSON array of boolean objects to PG array `boolean[]`. 

#### `json_get_int_array(text, text) -> int[]`

Extract and converts a JSON array of integer objects to PG array `int[]` (`int4[]`)

#### `json_get_bigint_array(text, text) -> bigint[]`

Extract and converts a JSON array of integer objects to PG array `bigint[]` (`int8[]`) 

#### `json_get_numeric_array(text, text) -> numeric[]`

Extract and converts a JSON array of integer objects to PG array `numeric[]`. 

#### `json_get_timestamp_array(text, text) -> timestamp without time zone[]`

Extract and converts a JSON array of text objects to PG array `timestamp[]` without time zones.
Time format is the same.

### Limitations ###

PostgreSQL `numeric` data type is parsed by a fixed position pattern and could be trimmed from a very big value.

Author
------

Copyright (c) 2012, Con Certeza LLC. All Right Reserved.

Developed by [Eugene Seliverstov](theirix@concerteza.ru)

Copyright and License
---------------------

You can use any code from this project under the terms of [PostgreSQL License](http://www.postgresql.org/about/licence/).
Please consult with the COPYING for license information.
