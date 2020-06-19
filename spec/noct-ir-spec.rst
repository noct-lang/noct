Noct intermediate representation specification (Low-level)
==========================================================

.. contents::


Introduction
============

This file contains the specification of the `noct` IR (Low-level Intermediate representation), which is SSA (single static assignment) representation.

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
Binary code will have the extension: .nxir

The binary code is not meant to be used separatly, but will be used either as part of a module file, or supplied with its associate module file as an optional file.

Textual representation
----------------------

This is can only be output and exists to give a visual representation. The textual represenation will be output in UTF-8.
Textual representation will have the extension: .nxirt

Binary encoding
===============

This section lays out the basic encoding for the IL, the encoding of specific elements can be found by the respective byte code elements.

Header
------

The IL header is a block of 32 bytes giving basic info about the IL bytecode

.. code-block::

    |        Magic number (.NIR)        |  Ver.  |         Reserved         |                               File size                               |
    v                                   v        V                          v                                                                       v
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xFF  |  0x4E  |  0x49  |  0x52  |  0x??  |  0x00  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    0                                   4        5                          8                                                                      16

    16                                                                     24                                                                      32
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xFF  |  0x4E  |  0x49  |  0x4C  |  0x??  |  0x00  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |  0x??  |
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    ^                                                                       ^                                                                       ^
    |                            type table size                            |                            name table size                            |



Primitive types
---------------

====== ====
 type   id
====== ====
 i8     0
 i16    1
 i32    2
 i64    3
 i128   4
 u8     5
 u16    6
 u32    7
 u64    8
 u128   9
 f16    A
 f32    B
 f64    C
 f128   D
====== ====


Primitive operations
--------------------

Primitive operations are operation that can be called on primitive types

Binary encoding
^^^^^^^^^^^^^^^

.. code-block::

    | def id |  op id |
    v        v        v
    +--------+--------+
    |  0x??  |  0x??  |
    +--------+--------+
    0        1        2


    // Op ID
    0xOT
      ^^
      |+- type nybble
      +-- op nybble


add
```
Nybble: 0


sub
```
Nybble: 1

mul
```
Nybble: 2

div
```
Nybble: 3


and
```
Nybble: 4

xor
```
Nybble: 5

or
``
Nybble: 6




Vector types
------------

======== ====
  type    id
======== ====
 i8x8
 i8x16
 i16x4
 i16x8
 i32x2
 i32x4
 u8x16
 u16x8
 u32x4
 f32x4
 f32x8
 f64x2
 f64x4

 bx64
 bx128
 bx256
 bx512
======== ====


Vector instructions
-------------------

vadd
vsub
vmul
vmulhi
vmullo
vdiv
vrcp
vsqrt
vrsqrt
vmax
vmin
vor
vand

vcvt

vcmp

vpack
vpadd




---------------


zext
sext