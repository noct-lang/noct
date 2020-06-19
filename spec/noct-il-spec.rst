Noct intermediate language specification (Mid-level)
====================================================

.. contents::


Introduction
============

This file contains the specification of the `noct` IL (Medium-level Intermediate representation), which is SSA (single static assignment) representation. 
The module stores the implementations of functions and methods and some minimal data to be able to decode them.

This IL will not be fully stable until 1.0 is reached, this can cause breaking changes and unforeseen issues.

Notation
========

The grammar is specified using `EBNF` or `Extended Backus-Naur Form`.

`EBNF` follows the following rules

================== ================
 Usage              Notation
================== ================
 literal            "lit"
 value              name
 assignment         ... = ...
 concatenation      ... | ...
 optional           [ ... ]
 repetition         { ... }
 grouping           ( ... )
 terminal string    "..." or '...'
 comment            (* ... \*)
 special sequence   ? ... ?
 exception          - ...
================== ================

 .. note:: 
    `...` represents any valid `EBNF` syntax in the table above

IL representation
=================

IL has 2 representation:

Binary code
-----------

This is what the compiler will generate and can read.
Binary code will have the extension: .nxil

The binary code is not meant to be used separatly, but will be used either as part of a module file, or supplied with its associate module file as an optional file.

Textual representation
----------------------

This is can only be output and exists to give a visual representation. The textual represenation will be output in UTF-8.
Textual representation will have the extension: .nxilt




Binary encoding
===============

This section lays out the basic encoding for the IL, the encoding of specific elements can be found by the respective byte code elements.

Header
------

The IL header is a block of 32 bytes giving basic info about the IL bytecode

.. code-block::

    |        Magic number (.NIL)        |  Ver.  |         Reserved         |                               File size                               |
    v                                   v        V                          v                                                                       v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xFF  |  0x4E  |  0x49  |  0x4C  |  0x??  |  0x00  |  0x00  |  0x00  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0                                   4        5                          8                                                                      16

    16                                                                     24                                                                      32
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    ^                                                                       ^                                                                       ^
    |                            type table size                            |                            name table size                            |

Encoded types
=============

Types used in the IL are stored in a table at the beginning to the IL files, types are stored as a mangled name.

.. code-block::

    | def id |            Num Entries            |     entries    |
    v        v                                   v                v
    +--------+--------+--------+--------+--------+----------------+
    |  0xFE  |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+--------+----------------+
    0        1                                   5                ?

Encoding in expressions
```````````````````````

When encoded in an expression, an index in the table will be stored, depending on the IL flags, the width of the encoded type will be chosen

Encoded names
=============

Types used in the IL are stored in a table at the beginning to the IL files, types are stored as a mangled name.

.. code-block::

    | def id |            Num Entries            |     entries    |
    v        v                                   v                v
    +--------+--------+--------+--------+--------+----------------+
    |  0xFD  |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+--------+----------------+
    0        1                                   5                ?

Entry
-----

Each entry is fairly simple, as it just contains a null terminated mangled name.

Encoded variables
=================

Inside of an implementation, varaibles will be identified with a unique identifier, the width of this identifier will be decided by the IL flags.
The first 2 bits of the identifier are special, the first bit indicates the value is moved, the second bit means a reference is taken. If none of these bits are set the value is copied.
The actual id is encoded with a variable length, with a single byte indicating whether an additional byte follows the current byte, the last (4th) byte, does not include this bit.

When multiple bytes are used for a variable, the bytes are layed out in LE, so for a 4 byte encoding (the max encoding), the bits are layed as followed: (High bit) 333 33333222 22221111 11100000 (Low bit)

.. code-block::

    | mov | ref | add |           var id            |  | add |                 var id                  |  | add |                 var id                  |  |                    var id                     | 
    V bit v bit v byt v                             v  v byt v                                         v  v byt v                                         v  v                                               v 
    +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+ 
    |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  | 
    +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+ 
    0     1     2     3                             8  8                                               16 16                                              24 24                                              32

In case that both bits are set, the variable is a literal and follows the following encoding

.. code-block::

    |  literal  |       lit type              | Bl. |  value   |
    V   bits    v                             v     v          v
    +-----+-----+-----+-----+-----+-----+-----+-----+----------+
    |  1  |  1  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |          |
    +-----+-----+-----+-----+-----+-----+-----+-----+----------+


======== ==== =============================
 Type     ID   Data bits
======== ==== =============================
 bool     0    0, Stored in Bl. bit
 i8       1    1
 i16      2    2
 i32      3    4
 i64      4    8
 i128     5    16
 u8       6    1
 u16      7    2
 u32      8    4
 u64      9    8
 u128     10   16
 f32      12   4
 f64      13   8

 char     15   UTF-8 character
 string   16   null-terminated UTF-8 string
 null     17   null value
======== ==== =============================

Functions and Methods
=====================

Since the main purpose of the IL is to store the implementations of functions an methods, a big chunk will be dedicated to functions and the underlying statements.

.. code-block::

    | def id |     mangled name id      |  num sub-statements/expressions   |             Func size             |
    v        v                          v                                   v                                   v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xF0  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0        1                          4                                   8                                  16

    20               22                ?                 ?                ?
    +--------+--------+----------------+--------+--------+----------------+
    |  0x??  |  0x??  |                |  0x??  |  0x??  |                |
    +--------+--------+----------------+--------+--------+----------------+ 
    ^                 ^                ^                 ^                ^
    |   param count   |   param types  |   local count   |   local types  |



Elements
========

Block statements
----------------

Encoding
````````

.. code-block::

    | def id |          label           |   num sub-statements/expressions  |
    v        v                          v        (without terminal)         v
    +--------+--------+--------+--------+--------+--------+--------+--------+
    |  0x01  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+
    0        1                          4                                   8

Textual represenation
`````````````````````

.. code-block::

    block-statement = '{', { statement | expression }, '}'; 

Assign Expressions
------------------

Encoding
````````

.. code-block::

    | def id |    dst var     |    dst type    |     src var    |
    v        v                v                v                v
    +--------+----------------+----------------+----------------+
    |  0x40  |                |                |                |
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    assign-expr = identifier, '=', identifier;

Primitive assignment operator
-----------------------------

A primitive operator is any operator that is applied to primitive types

Encoding
````````

.. code-block::

    | def id |   op   |    dst var     |    dst type    |     src var    |
    v        v        v                v                v                v
    +--------+--------+----------------+----------------+----------------+
    |  0x41  |  0x??  |                |                |                |
    +--------+--------+----------------+----------------+----------------+
    0        1        2                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    prim-assign-expr = identifier, op, identifier;

Primitive binary operator
-------------------------

A primitive operator is any operator that is applied to primitive types

Encoding
````````

.. code-block::

    | def id |   op   |    dst var     |    dst type    |    src0 var    |    src1 var    |
    v        v        v                v                v                v                v
    +--------+--------+----------------+----------------+----------------+----------------+
    |  0x42  |  0x??  |                |                |                |                |
    +--------+--------+----------------+----------------+----------------+----------------+
    0        1        2                ?                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    assign-expr = identifier, '=', identifier, op, identifier;

Primitive unary operator
-------------------------

A primitive operator is any operator that is applied to primitive types

Encoding
````````

.. code-block::

    | def id |   op   |    dst var     |    dst type    |     src var    |
    v        v        v                v                v                v
    +--------+--------+----------------+----------------+----------------+
    |  0x43  |  0x??  |                |                |                |
    +--------+--------+----------------+----------------+----------------+
    0        1        2                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    assign-expr = identifier, '=', op, identifier;

Primitive cast operator
-----------------------

Encoding
````````

.. code-block::

    | def id |    dst var     |    dst type    |     src var    |
    v        v                v                v                v
    +--------+----------------+----------------+----------------+
    |  0x44  |                |                |                |
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    assign-expr = identifier, '=', 'cast', ', identifier;

Ternary operator
----------------

A primitive operator is any operator that is applied to primitive types

Encoding
````````

.. code-block::

    | def id |   op   |    dst var     |    dst type    |    cond var    |    src0 var    |    src1 var    |
    v        v        v                v                v                v                v                v
    +--------+--------+----------------+----------------+----------------+----------------+----------------+
    |  0x45  |  0x??  |                |                |                |                |                |
    +--------+--------+----------------+----------------+----------------+----------------+----------------+
    0        1        2                ?                ?                ?                ?                ?

Transmute
---------

Encoding
````````

.. code-block::

    | def id |    dst var     |    dst type    |     src var    |
    v        v                v                v                v
    +--------+----------------+----------------+----------------+
    |  0x46  |                |                |                |
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ?

Textual represenation
`````````````````````

.. code-block::

    assign-expr = identifier, '=', 'transmute', '(', type, ')', identifier;


Compiler Intrinsics
-------------------

Compiler intrinsics are builtin compiler 'functions' that can take different data depending on the intrin id

Encoding
````````

.. code-block::

    | def id | intrin |      data      |
    v        v   id   v                v
    +--------+--------+----------------+
    |  0x47  |  0x??  |                | 
    +--------+--------+----------------+
    0        1        2                ?

data
^^^^

Data for type intrins

.. code-block::

    |    dst var     |    dst type    |      type      |
    v                v                v                v
    +----------------+----------------+----------------+
    |                |                |                |
    +----------------+----------------+----------------+
    2                ?                ?                ?

Data for type intrins

.. code-block::

    |   arg  |    dst var     |    dst type    |      args      |
    v  count v                v                v                v
    +--------+----------------+----------------+----------------+
    |  0x??  |                |                |                |
    +--------+----------------+----------------+----------------+
    2        3                ?                ?                ?


Function calls
--------------

Free functions
``````````````

Encoding
^^^^^^^^

Without return:

.. code-block::

    | def id |             func name             |   arg  |      args      |
    v        v                                   v  count v                v
    +--------+--------+--------+--------+--------+--------+----------------+
    |  0x50  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+--------+--------+----------------+
    0        1                                   5        6                ?

With return:

.. code-block::

    | def id |             func name             |   arg  |    dst var     |    dst type    |      args      |
    v        v                                   v  count v                v                v                v
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+
    |  0x51  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |                |                |                |
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+
    0        1                                   6        2                ?                ?                ?

Method
``````

Encoding
^^^^^^^^

Without return:

.. code-block::

    | def id |             func name             |   arg  |     caller     |      args      |
    v        v                                   v  count v                v                v
    +--------+--------+--------+--------+--------+--------+----------------+----------------+
    |  0x52  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |                |                |
    +--------+--------+--------+--------+--------+--------+----------------+----------------+
    0        1                                   5        6                ?                ?

With return:

.. code-block::

    | def id |             func name             |   arg  |    dst var     |    dst type    |     caller     |      args      |
    v        v                                   v  count v                v                v                v                v
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+----------------+
    |  0x53  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |                |                |                |                |
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+----------------+
    0        1                                   5        6                ?                ?                ?                ?

Indirect functions
``````````````````

Encoding
^^^^^^^^

Without return:

.. code-block::

    | def id |   arg  |    func addr   |      args      |
    v        v  count v                v                v
    +--------+--------+----------------+----------------+
    |  0x50  |  0x??  |                |                |
    +--------+--------+----------------+----------------+
    0        1        2                ?                ?

With return:

.. code-block::

    | def id |   arg  |    dst var     |    dst type    |    func addr   |      args      |
    v        v  count v                v                v                v                v
    +--------+--------+----------------+----------------+----------------+----------------+
    |  0x51  |  0x??  |                |                |                |                |
    +--------+--------+----------------+----------------+----------------+----------------+
    0        1        2                ?                ?                ?                ?

Member Access
-------------

Encoding
````````

.. code-block::

    | def id |            member name            |    dst var     |    dst type    |    src var     |
    v        v                                   v                v                v                v
    +--------+--------+--------+--------+--------+----------------+----------------+----------------+
    |  0x56  |  0x??  |  0x??  |  0x??  |  0x??  |                |                |                |
    +--------+--------+--------+--------+--------+----------------+----------------+----------------+
    0        1                                   5                ?                ?                ?

Tuple Access
------------

Encoding
````````

.. code-block::

    | def id |      index      |    dst var     |    dst type    |    src var     |
    v        v                 v                v                v                v
    +--------+--------+--------+----------------+----------------+----------------+
    |  0x57  |  0x??  |  0x??  |                |                |                |
    +--------+--------+--------+----------------+----------------+----------------+
    0        1                 3                ?                ?                ?


Struct init
---------

Encoding
````````

.. code-block::

    | def id |   arg  |    dst var     |      type      |      args      | 
    v        v  count v                v                v                v 
    +--------+--------+----------------+----------------+----------------+
    |  0x60  |  0x??  |                |                |                | 
    +--------+--------+----------------+----------------+----------------+
    0        1        2                ?                ?                ? 


Union init
---------

Encoding
````````

.. code-block::

    | def id |    dst var     |      type      |      arg       | 
    v        v                v                v                v 
    +--------+----------------+----------------+----------------+
    |  0x61  |                |                |                | 
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ? 


Value enum init
---------------

Encoding
````````

.. code-block::

    | def id |            member name            |    dst var     |      type      | 
    v        v                                   v                v                v 
    +--------+--------+--------+--------+--------+----------------+----------------+
    |  0x62  |  0x??  |  0x??  |  0x??  |  0x??  |                |                | 
    +--------+--------+--------+--------+--------+----------------+----------------+
    0        1                                   5                ?                ? 

Adt enum init
-------------

Encoding
````````

.. code-block::

    | def id |            member name            |  arg   |    dst var     |      type      |      args      | 
    v        v                                   v  count v                v                v                v 
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+
    |  0x63  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |                |                |                | 
    +--------+--------+--------+--------+--------+--------+----------------+----------------+----------------+
    0        1                                   5        2                ?                ?                ? 

tuple init
----------

Encoding
````````

.. code-block::

    | def id |  arg   |    dst var     |      type      |      args      | 
    v        v  count v                v                v                v 
    +--------+--------+----------------+----------------+----------------+
    |  0x64  |  0x??  |                |                |                | 
    +--------+--------+----------------+----------------+----------------+
    0        1        2                ?                ?                ? 

Array init
----------

Encoding
````````

.. code-block::

    | def id |  arg   |    dst var     |      type      |      args      | 
    v        v  count v                v                v                v 
    +--------+--------+----------------+----------------+----------------+
    |  0x65  |  0x??  |                |                |                | 
    +--------+--------+----------------+----------------+----------------+
    0        1        2                ?                ?                ? 


Terminals
=========

If terminal
-----------

Encoding
````````
.. code-block::

    | def id |       true label         |       false label        |  res.  |    cond var    |
    v        v                          v                          v        v                v
    +--------+--------+--------+--------+--------+--------+--------+--------+----------------+
    |  0x70  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x00  |                |
    +--------+--------+--------+--------+--------+--------+--------+--------+----------------+
    0        1                          4                          4        8                ?

Textual represenation
`````````````````````

.. code-block::

    if-statement = 'if', expression, '{', { statement | expression }, '}', [ 'else', '{', { statement | expression }, '}' ];


Switch terminal
---------------
// TODO


Goto terminal
-------------

Encoding
````````
.. code-block::

    | def id |          label           |
    v        v                          v
    +--------+--------+--------+--------+
    |  0x72  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+
    0        1                          5


Return terminal
---------------

Encoding
````````

Without value

.. code-block::

    | def id |
    v        v
    +--------+
    |  0x73  |
    +--------+
    0        1

With value

.. code-block::

    | def id |     ret var    |
    v        v                v
    +--------+----------------+
    |  0x74  |                |
    +--------+----------------+
    0        1                ?