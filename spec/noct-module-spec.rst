Noct programming language specification
=======================================

.. contents::


Introduction
============

Current version: 0.0

This file contains the specification of the `noct` module.

This language will not be fully stable until 1.0 is reached, this can cause breaking changes and unforeseen issues.

Module structure
================

A module file consists out of multiple sections

Module header
-------------

The header section is a special section, as it does not contain an identifier, since it is always the first section of a module.

.. code-block::

    |        Magic number (.NXM)        |  Ver.  |    Reserved     | Flags  |                               File size                               |
    v                                   v        V                 v        v                                                                       v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xFF  |  0x4E  |  0x58  |  0x4D  |  0x??  |  0x00  |  0x00  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0                                   4        5                          8                                                                      16

    |            Iden size              |  package name  |  module name   |  num   |
    v                                   v                v                v  secs  v
    +--------+--------+--------+--------+----------------+----------------+--------+
    |  0x??  |  0x??  |  0x??  |  0x??  |                |                |  0x??  |
    +--------+--------+--------+--------+----------------+----------------+--------+
    16                                  20               ?                ?        ?

Flags
`````

.. code-block::

    | optional  | optional  |           |           |           |           |           |           |
    V IL (file) v IR (file) v           v           v           v           v           v           v
    +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
    |     ?     |     ?     |     ?     |     ?     |     ?     |     ?     |     ?     |     ?     |
    +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
    0           1           2           3           4           5           6           7           8


Section Header
``````````````

The header contains a small amount of data about each section in the module, these simply contain a 4 byte identifier to identify the section to the loader + the offset in the file and it size (both in bytes). A section header is included at the start of each section

.. code-block::

    |           Section iden            |                             Section size                              |
    v                                   v                                                                       v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0                                   4                                                                      12


A description of each identifier and its corresponding section is given in the table below.

======== =========================== ===========
 Iden     Section                     Mandatory
======== =========================== ===========
 'IMPO'   `Import section`_           No
 'NAME'   `Name section`_             Yes
 'FEAT'   `Feature set section`_      No
 'MACR'   `Macro section`_            No
 'SYMS'   `Symbol section`_           Yes
 'SLNK'   `Symbol linking section`_   No
 'ILBC'   `IL Bytecode section`_      No
======== =========================== ===========


Sections
========

Each section of the module file contains 2 parts, a section header and section data. The section header is a section-specific header, containing data required to read the data.

Import section
--------------

This section contains all modules that the current module relies on.
If an import section is included, it will always be the first section that is encountered

.. code-block::

    |  Num. modules   |     import     |
    v                 v                v
    +--------+--------+----------------+
    |  0x??  |  0x??  |                |
    +--------+--------+----------------+
    0                 2                ?

.. code-block::

    | Flags  |     import     |
    v        v                v
    +--------+----------------+
    |  0x??  |                |
    +--------+----------------+
    0        1                ?

    // Flags

    | pub |                reserved                 |
    v imp v                                         v
    +-----+-----+-----+-----+-----+-----+-----+-----+
    |  ?  |  0  |  0  |  0  |  0  |  0  |  0  |  0  |
    +-----+-----+-----+-----+-----+-----+-----+-----+
    0                                               8

The names are stored as null terminated strings, making this indepent from the name section

Name section
------------

This section contains all names (mangled and unmangled), that are used in this module and where other sections index in to reduce duplication. The names are stored as a list of null terminated names.
If a name with id of 0 is encountered, it is represented by a blank string.

.. code-block::

    |         Number of names           |     names      |
    v                                   v                v
    +--------+--------+--------+--------+----------------+
    |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+----------------+
    0                                   4                ?

Names are encoded using a id encoded with a variable length (1 to 4 bytes).
When multiple bytes are used for a variable, the bytes are layed out in LE, so for a 4 byte encoding (the max encoding), the bits are layed as followed: (High bit) 3 3333 3332 2222 2211 1111 1000 0000 (Low bit)

.. code-block::

    | add |                 var id                  |  | add |                 var id                  |  | add |                 var id                  |  |                    var id                     | 
    v byt v                                         v  v byt v                                         v  v byt v                                         v  v                                               v 
    +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+ 
    |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  |  ?  | 
    +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+  +-----+-----+-----+-----+-----+-----+-----+-----+ 
    0                                               8  8                                               16 16                                              24 24                                              32

Feature set section
-------------------

.. Note::

    This section is incomplete

Macro section
-------------

.. Note::

    This section is incomplete

This section contains all macros declared by in a module and their corresponding tokens

Header
``````

The macro section header contains:

.. code-block::

    |            Decl count             |            Proc count             |                              Proc offset                              |
    v                                   v                                   v                                                                       v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0                                   4                                   8                                                                      16

Data
````

The data of the macro section contains 2 distinct types of macros, each with their specific layout. All macros are sequentially layed out without any padding between the data.

Declarative macro data
^^^^^^^^^^^^^^^^^^^^^^
======= ============== =========================================================================================================
 bytes   Name           Description
======= ============== =========================================================================================================
 4       Size           Size of current entry
 N       Qual name      Qualified name of macro (relative to module) (identifiers separated by '.' and name terminated by '\0')
 4       Pattern size   Size of pattern (in bytes)
 N       Pattern        Encoded pattern (stored as unicode) (see token encoding for more info)
 4       Body size      Size of macro body
 N       Body           Encoded body (stored as unicode) (see token encoding for more info)
======= ============== =========================================================================================================

Procedural macro Data
^^^^^^^^^^^^^^^^^^^^^
======= ============== =========================================================================================================
 bytes   Name           Description
======= ============== =========================================================================================================
 4       Size           Size of current entry
 N       Qual name      Qualified name of macro (relative to module) (identifiers separated by '.' and name terminated by '\0')
 4       Pattern size   Size of pattern (in bytes)
 N       Pattern        Encoded pattern (stored as unicode) (see token encoding for more info)
 4       IL func size   size of macro's IL bytecode
 N       IL bytecode    IL bytecode
======= ============== =========================================================================================================

Token encoding
^^^^^^^^^^^^^^

To decrease the size of the macro sections, the tokens generated for the macro are encoded.


=========== ========== =============================================================
 Token       Encoding   Additional info
=========== ========== =============================================================


 Unknown     FF         Should not occur, occurrence mean that something went wrong
=========== ========== =============================================================



Symbol section
--------------

The symbol  section contains all symbols for the module

.. code-block::

    |        Number of symbols          |    symbols     |
    v                                   v                v
    +--------+--------+--------+--------+----------------+
    |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+----------------+
    0                                   4                ?


==== ==================
 ID   kind
==== ==================
 0    struct
 1    union
 2    val enum
 3    val enum member
 4    adt enum
 5    adt enum member
 6    marker interface
 7    weak interface
 8    strong interface
 9    typealias
 A    typedef
 B    func
 C    method
 D    var
==== ==================


.. code-block::

    // Version for struct, union, adt enum, marekr interface, weak interface and strong interface

    |  sym   |         Mangled name id           |
    v  kind  v                                   v
    +--------+--------+--------+--------+--------+
    |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+
    0        1                                   ?

    // Version for typedef, val enum, val enum member, adt enum member, func and var

    |  sym   |    mangled     |    mangled     |
    v  kind  v    name id     v    type id     v
    +--------+----------------+----------------+
    |  0x??  |                |                |
    +--------+----------------+----------------+
    0        1                ?                ?

    // Version for typealias and method

    |  sym   | mangled iface  |    mangled     |    mangled     |
    v  kind  v    name id     v    name id     v    type id     v
    +--------+----------------+----------------+----------------+
    |  0x??  |                |                |                |
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ?


Symbol linking section
----------------------

The symbol linking section defines the hierarchy of the symbols (parent-child, impls, etc)

.. code-block::

    |         Number of links           |  section info  |
    v                                   v                v
    +--------+--------+--------+--------+----------------+
    |  0x??  |  0x??  |  0x??  |  0x??  |                |
    +--------+--------+--------+--------+----------------+
    0                                   4                ?

==== ==================
 ID   kind
==== ==================
 0    parent
 1    impl
 2    variants
 3    interface ext
 4    member idx
==== ==================

.. code-block::


    |  link  |    mangled     | mangled iface  | mangled parent |
    v  kind  v    name id     v    name id     v    name id     v
    +--------+----------------+----------------+----------------+
    |  0x00  |                |                |                |
    +--------+----------------+----------------+----------------+
    0        1                ?                ?                ?

    // Only exists for: struct, unions, val and adt enums, and typedefs

    |  link  |   num ifaces    |    mangled     | mangled iface  |
    v  kind  v                 v    name id     v    name ids    v  
    +--------+--------+--------+----------------+----------------+
    |  0x01  |  0x??  |  0x??  |                |                |
    +--------+--------+--------+----------------+----------------+
    0        1                 3                ?                ?

    // Only exists for types

    |  link  |   num ifaces    |    mangled     | mangled iface  |
    v  kind  v                 v    type id     v    name ids    v  
    +--------+--------+--------+----------------+----------------+
    |  0x01  |  0x??  |  0x??  |                |                |
    +--------+--------+--------+----------------+----------------+
    0        1                 3                ?                ?


    |  link  |  num variants   |    mangled     |  mangled var.  |
    v  kind  v                 v    type id     v    name ids    v
    +--------+--------+--------+----------------+----------------+
    |  0x02  |  0x??  |  0x??  |                |                |
    +--------+--------+--------+----------------+----------------+
    0        1                 3                ?                ?

    |  link  |   num members   | mangled parent | mangled member |
    v  kind  v                 v    name id     v    name ids    v
    +--------+--------+--------+----------------+----------------+
    |  0x04  |  0x??  |  0x??  |                |                |
    +--------+--------+--------+----------------+----------------+
    0        1                 3                ?                ?


IL Bytecode section
-------------------
