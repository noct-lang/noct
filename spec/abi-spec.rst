Noct Application Binary Interface (ABI) Specification
====================================================

.. contents::


Introduction
============

This file contains the `noct` ABI specification

This IL will not be fully stable until 1.0 is reached, this can cause breaking changes and unforeseen issues.

Name Mangling
=============

While in `noct`, functions/methods can be differentiated by just their encoded name (qualified name + function parameter labels), type information is also encoded in the mangled name.

A mangled name is identified with a `_N` prefix, followed by a mangled qualified name and type;

.. code-block:

    mangled-name = '_N', mangle-sym;

    mangle-sym = mangle-func
               | mangle-method
               | mangle-interface-method;

Mangled function
----------------

A mangled function represents a free function

.. code-block::

    mangle-func = 'F', qual-name, type;

Mangled Method
----------------

A mangled method represents any possible method, but has 2 types:

- Normal method: 'M' followed by the mangled name and type
- Interface method implemenation: 'N' followed by the mangled name of the interface, then a 'Z', followed by the mangled name and type

.. code-block::

    mangle-method = 'M', qual-name, type;

    mangle-interface-method = 'N', qual-name, 'Z', qual-name, type;

    qual-name = { sym-name }+

    sym-name = lname
             | generic;


    lname = number, identifier;


    number = { digit }+;
    digit = ? 0-9 ?;

    identifier = [ letter | '_' ], { letter | '_' | digit };
    letter = ? a-zA-Z ?;


    gen-inst = lname, 'G', { gen-param | gen-arg }, 'Z';

    gen-param = gen-type-param
              | gen-val-param;

    gen-type-param = 'T', lname, [gen-type-constraint], 'Z';
    gen-val-param = 'V', lname, type, [gen-val-constraint], 'Z';

    gen-constraint = 'C', { type }, 'Z';


    // specialized params
    gen-arg = gen-type-arg
            | gen-val-arg;

    gen-type-arg = 'U', type, 'Z';
    gen-val-arg = 'W', lit, 'Z';







    type = [type-mod], sub-type;

    type-mod = const-type-mod;
    const-type-mod = 'C';


    sub-type = builtin-type
             | iden-type
             | ptr-type
             | ref-type
             | arr-type
             | slice-type
             | opt-type
             | tup-type
             | func-type;



    
    builtin-type = bool-type
                 | i8-type
                 | i16-type
                 | i32-type
                 | i64-type
                 | i128-type
                 | u8-type
                 | u16-type
                 | u32-type
                 | u64-type
                 | u128-type
                 | f16-type
                 | f32-type
                 | f64-type
                 | f128-type
                 | char-type;
                 
    
    bool-type  = 'b';
    i8-type    = 'i';
    i16-type   = 'j';
    i32-type   = 'k';
    i64-type   = 'l';
    i128-type  = 'm';
    isize-type = 'n';
    u8-type    = 'u';
    u16-type   = 'v';
    u32-type   = 'w';
    u64-type   = 'x';
    u128-type  = 'y';
    usize-type = 'z';
    f16-type   = 'e';
    f32-type   = 'f';
    f64-type   = 'g';
    f128-type  = 'h';
    char-type  = 'c';

    iden-type = qual-name;
    ptr-type = 'P', type;
    ref-type = 'R', type;
    arr-type = 'A', number, type;
    slice-type = 'S', type;
    opt-type = 'O', type;
    tup-type = 'T', { type }, 'Z';
    func-type = 'F', { type }, 'Z', [type], 'Z';









    




Calling convention
==================