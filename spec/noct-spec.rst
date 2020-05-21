Noct programming language specification
=======================================

.. contents::


Introduction
============

This file contains the specification of the `noct` programming language.

This language will not be fully stable until 1.0 is reached, this can cause breaking changes and unforeseen issues.

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

Source code representation
==========================

Source code for `noct` exists out of a valid sequence of UTF-8 characters. It's important to note that any unicode character that is represented as multiple unicode codepoints is interpreter as a sequence of multiple unicode character instead of a single unicode character.
A source file will have the extension: .nx

Whitespace, end of line, and end of file
----------------------------------------

.. code-block::

    space = "\u0009"
          | "\u000B"
          | "\u000C"
          | "\u0020";

    whitespace = { space }

    eol = "\u000A"
        | "\u000D"
        | "\u000A", "\u000D"
        | "\u2028"
        | "\u2029";

    eof = ?end of character sequence?;

Unicode, letters and digits
---------------------------

.. code-block::

    unicode-character = ?valid unicode codepoint? - "\u0000";
    unicode-letter = ?valid unicode codepoint, qualified as "letter"?;
    unicode-digit = ?valid unicode codepoint, qualified as "digit"?;

    letter = unicode-letter;
    bin-digit = '0' | '1';
    oct-digit = bin-digit | '2' | '3' | '4' | '5' | '6' | '7';
    dec-digit = oct-digit | '8' | '9';
    hex-digit = dec-digit | 'a' | 'A' | 'b' | 'B' | 'c' | 'C' | 'd' | 'D' | 'e' | 'E' | 'f' | 'F';

Escape codes
------------

An escape code is a special character encoding that can be used in character and string literals, and which represent a certain unicode value.

.. code-block::

    escape-code = simple-escape-code | oct-escape-code | hex-escape-code | small-u-escape-code | big-u-escape-code;
    simple-escape-code = '\', ( '0' | 'a' | 'b' | 'f' | 'n' | 'r' | 't' | 'v' | '\' );
    oct-escape-code = '\o', oct-digit, oct-digit, oct-digit;
    hex-escape-code = '\x', hex-digit, hex-digit;
    small-u-escape-code = '\u', hex-digit, hex-digit, hex-digit, hex-digit;
    big-u-escape-code =  '\U', hex-digit, hex-digit, hex-digit, hex-digit, hex-digit, hex-digit, hex-digit, hex-digit;

Examples::

    \0
    \a
    \b
    \f
    \n
    \r
    \t
    \v
    \\
    \0377
    \x7F
    \u12E4
    \U00101234

Lexical elements
----------------

Comments
````````

A comment allows additional information to be added to `noct` code.

Line Comments
^^^^^^^^^^^^^

A line comment takes up a single line, starting from the required identifier.

.. code-block::

    line-comment = '//', {unicode-character}, eol;

Block Comments
^^^^^^^^^^^^^^

A block comment can take up multiple lines. It can also be nested in itself.

.. code-block::

    block-comment = '/*', {unicode-character | block-comment }, '*/';

Semicolons
``````````

Semicolons (`;`) are an important part of the `noct` language, as they signal the end of an expression.

Keywords
````````
.. _keyword:

A keyword is a special `identifier`_, which has a specific meaning in the `noct` language.

Below is a list of keywords::

- as
- break
- comptime
- const
- continue
- defer
- do
- else
- enum
- errdefer
- fallthrough
- for
- func
- goto
- if
- immutable
- import
- impl
- in
- !in
- interface
- is
- !is
- lazy
- loop
- macro
- module
- move
- public
- return
- static
- struct
- switch
- throw
- throws
- transmute
- try
- typealias
- typedef
- union
- unsafe
- while

Type keywords
^^^^^^^^^^^^^

- bool
- char
- f16
- f32
- f64
- f128
- isize
- i8
- i16
- i32
- i64
- i128
- usize
- u8
- u16
- u32
- u64
- u128

Constant keywords
^^^^^^^^^^^^^^^^^

- false
- null
- true

Context dependent keywords
^^^^^^^^^^^^^^^^^^^^^^^^^^

- dynlib
- package
- Self
- self
- throws
- weak
- where

Special Keywords
^^^^^^^^^^^^^^^^

Special keywords are keywords that start using a `#`, they are meant as keywords, without additionally limiting the identifiers that can be used, while still allowing easy expansion of the language without changing the core syntax. This type of keywords is the only kind of keyword that can contain capital letter (with the exception of the `Self` type keyword)

Below is a list of special keywords::

- #file
- #fileFullPath
- #package
- #module
- #fullModule
- #line
- #func
- #funcName
- #prettyFunc
- #conditional
- #debug
- #run
- #errorhandler
- #if
- #unittest
- #benchmark

Reserved keywords
`````````````````

Keywords reserved for future use::

- async
- await
- yield

Identifiers
```````````
.. _identifier:

An identifier is a name that references some kind of value, e.g. a variable.

.. code-block::

    identifier = ( unicode-letter | '_' ), { unicode-letter | unicode-digit | '_' };
    identifier-list = identifier, { ',', identifier };

Certain identifier are reserved by the language. The use of these identifiers as names can cause unexpected errors or undefined behavior.
The following are language reserved identifiers::

- blank identifier: `_`
- `keyword`_
- Any identifier containing a double underscore: `__`

Operators and punctuation
`````````````````````````

An operator defines a certain operation that will happen on an expression, where as punctuation adds additional info used by the grammar. Since these can overlap with each other, since both are a specific sequence of non-letter characters, they are defined together.

.. code-block::

    operator-punctuation = '=';
                         | '=='
                         | '=>'
                         | '+'
                         | '++'
                         | '+='
                         | '-'
                         | '--'
                         | '-='
                         | '->'
                         | '*'
                         | '*='
                         | '/'
                         | '/='
                         | '%'
                         | '%='
                         | '~'
                         | '~='
                         | '&'
                         | '&&'
                         | '&='
                         | '|'
                         | '||'
                         | '|='
                         | '^'
                         | '^='
                         | '<'
                         | '<<'
                         | '<<<'
                         | '<<*'
                         | '<='
                         | '<<='
                         | '<<<='
                         | '<<*='
                         | '>'
                         | '>>'
                         | '>>>'
                         | '>>*'
                         | '>='
                         | '>>='
                         | '>>>='
                         | '>>*='
                         | '!'
                         | '!!'
                         | '!='
                         | '!<'
                         | '!('
                         | '!{'
                         | '!['
                         | '?'
                         | '('
                         | ')'
                         | '{'
                         | '}'
                         | '['
                         | ']'
                         | ','
                         | ';'
                         | ':'
                         | '::'
                         | ':='
                         | '.'
                         | '..'
                         | '...'
                         | '..='
                         | '@'
                         | '@:'
                         | '??'
                         | '??='
                         | '?:'
                         | '?.'
                         | '?['
                         | '#'
                         | '$';

Literals
========

Literals represent a compile-time constant.

Integer Literals
----------------

An integer literal represents an integer value, meaning a number without any decimal parts.

.. code-block::

    integer-lit = ( bin-lit | oct-lit | dec-lit | hex-lit ), [integer-suffix];
    bin-lit = ( '0b' | '0B' ), bin-digit, { bin-digit | '_'] };
    oct-lit = ( '0o' | '0O' ), oct-digit, { oct-digit | '_' };
    dec-lit = [ '-' ], dec-digit, { dec-digit | '_' };
    hex-lit = ( '0x' | '0X' ), hex-digit, { hex-digit };
    integer-suffix = ( 'i' | 'u' ), ( '8' | '16' | '32' | '64' | '128' );

Examples::

    0b1010
    0o347
    1235
    127u8
    0xA2B

Floating-point Literals
-----------------------

A floating-point literal represents a numeric value, which can have a decimal part.

.. code-block::

    fp-lit = [ '-' ], ( dec-digit, { dec-digit | '_' }, fp-exp )
                    | ( dec-digit, { dec-digit | '_' }, '.', dec-digit, { dec-digit | '_' }, [fp-exp] )
                    [fp-suffix];
    fp-exp = ( 'e' | 'E' ), [ '-' ], dec-digit, { dec-digit | '_' };
    fp-suffix = 'f', ( '16' | '32' | '64' | '128' );

Examples::

    -1.23
    45e10
    23e-4
    4.56e7
    .1f64

Character literal
-----------------
A character literal is a unicode character represented by a single UTF-8 codepoint. The value of the character will be represented by its unicode codepoint, unlike a unicode scalar value, it is not stored in an encoded UTF-8 form. While a character literal will always be represented by a 32-bit value, depending on its encoding, will be accepted as a 1 to 4 byte value, when used as a unicode scalar value.

The value encoded in a character literal may take up more bytes than the unicode codepoint might make it seem, for example, the the literal `\x61` or `ä` will take up 2 bytes in its UTF-8 encoded form.

A character literal can also represent an escape code or escaped single quote ( `'` ).

.. code-block::

    char-lit = "'", ( unicode-character | escape-character | "\'" ), "'";

Examples::

    `a`
    `ä`
    `本`
    `\t`
    `\o000`
    `\o007`
    `\o377`
    `\x07`
    `\xff`
    `\u12E4`
    `\U00101234`
    `\'`            // char literal containing single quote
    `\aa`           // illegal: too many characters
    `\xa`           // illegal: too few hexadecimal digits
    `\o0`           // illegal: too few octal digits
    `\DFFF`         // illegal: surrogate half (UTF-16)
    `\U00110000`    // illegal: invalid codepoint
    `\400`          // illegal: exceeding max octal value of \377

String literals
---------------

A string literal represents a sequence of text. There are 2 possible representation of a string literal.

.. code-block::

    string-lit = basic-string-lit | wysiwyg-string-lit;

Basic string
````````````

A basic string literal is a simple sequence of characters, where escape code will be interpreted as the value they represent

.. code-block::

    basic-string-lit = '"', { unicode-character | escape-code | '\"' }, '"';

Wysiwyg or raw string
`````````````````````

A wysiwyg ( What you see is what you get ) string, also know as a raw string, represents a sequence of characters, without any escape codes, as `\` will be interpreted as its own value. The only exception is 2 double quotes after each other, which will be interpreted as a single value ( `"` ), and will therefore not terminate the literal.

.. code-block::

    wysiwyg-string-lit = 'r"', { unicode-character | '""' }, '"';

Examples
^^^^^^^^
.. code-block::

    r"abc"               // same as "abc"
    r"\n
    \n"r                 // same as "\\n\n\\n"
    "\n"
    "\""                 // same as `"`
    "Hello, world!\n"
    "日本語"
    "\u65e5本\U00008a9e"
    "\xff\u00FF"
    "\uD800"             // illegal: surrogate half
    "\U00110000"         // illegal: invalid Unicode codepoint


These examples all represent the same string:
.. code-block::

    "日本語"                                 // UTF-8 input text
    r"日本語"                                // UTF-8 input text as a wysiwig literal
    "\u65e5\u672c\u8a9e"                    // the explicit Unicode codepoints
    "\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode codepoints
    "\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes

Boolean literal
---------------

A boolean literal represents one of 2 possible states: true or false.

.. code-block::

    bool-lit = 'true' | 'false';

Null literal
------------

A null literal is a value that can be assigned to pointer and nullable types.

.. code-block::

    null-lit = 'null'

Literal conversions
-------------------

Literals can be implicitly converted to corresponding types, below is a table of possible conversions. Trying to use an implicit conversion that is not in the table will result in a compilation error. When an explicit bit length is defined in the suffix, the value will default to the corresponding width.

=========================== ============== ======================================
 Literal                     Default type   Implicit types
=========================== ============== ======================================
 integer                     i32            i8, i16, i32, i64, u8, u16, u32, u64
 integer (negative)          i32            i8, i16, i32, i64
 integer (unsigned suffix)   u32            u8, u16, u32, u64
 floating point              f64            f32, f64
 character                   char           none
 string                      []char         none
 boolean                     bool           none
=========================== ============== ======================================

Qualified names
===============

A qualified name allows types, variables, etc, to be identified, including scope and the disambiguation of types, which implement multiple interfaces, with a common member.

When a qualified name start with a double colon, it means the symbol resides in the global namespace, the namespace where packages and modules are located in.

.. code-block::

    qualified-name = [ '::' ], { qualified-identifier, '::' ), qualified-identifier;
    qualified-identifier = identifier | generic-instance | interface-disambiguation;
    interface-disambiguation = '<', type, 'as', interface-type, '>'

Types
=====

A type specifies the properties that a value has:

- Memory layout, alignment and size
- How to access the value
- Valid operations
- Allowed members or methods, if available

.. code-block::

    type = { type-attrib }, ( simple-type | elaborate-type );
    simple-type = builtin-type
                | identifer-type;
    elaborate-type = ptr-type
                   | ref-type
                   | arr-type
                   | slice-type
                   | tuple-type
                   | optional-type
                   | func-type
                   | inline-struct
                   | inline-enum;

Builtin types
-------------

A builtin or primitive type, is a type that is native to the compiler.

.. code-block::

    builtin-type = int-type | fp-type | bool-type | character-type;

Integer types
`````````````

An integer type can store a number, which does not contain a decimal point or `whole numbers`. All integer types have a single letter ( `i` or `u` ), which decided if the type contains a signed or unsigned value, followed by the bit-length of the value.

============ ======== ==========
 bit-length   signed   unsigned
============ ======== ==========
 8            i8       u8
 16           i16      u16
 32           i32      u32
 64           i64      u64
 128          i128     u128
 arch         isize    usize
============ ======== ==========

.. note:: The `arch` size defines a bit-length that depends on the architecture, i.e. 32-bit arch -> 32 bits, 64-bit arch -> 64 bit.

.. code-block::

    int-type = ( 'i' | 'u' ), ( '8' | '16 ' | '32 ' | '64' | 'size' );

Floating-point types
````````````````````

A floating-point type can store a number, which may contain a decimal point. All integer types start with the letter `f`, followed by the bit-length of the value.

============ ========
 bit-length   float 
============ ========
 16           f16
 32           f32
 64           f64
 128          f128
============ ========

.. code-block::

    fp-type = 'f', ( '32 ' | '64' );

.. note::

    `f16` and `f128` will be added in the future

Boolean types
`````````````

A boolean type can store a single, 2 value state.

.. code-block::

    bool-type = 'bool';

Character types
```````````````

A character type can store a unicode codepoint.

.. code-block::

    character-type = 'char';

Identifier types
----------------

An identifier type refers to a user defined type.

.. code-block::

    identifier-type = qualified-name;

Pointer types
-------------

A pointer type is a type that can refer to location or address in memory of a value of its `base type`.

To prevent a common issue, of trying to dereference pointer with a null value, a pointer cannot be assigned `null` to it. Any pointer that should be able to have `null` assigned to it, should be part of a nullable type. When applied to a pointer type, it acts both as syntactic sugar and a compiler hint.

.. code-block::

    ptr-type = '*', type;

Reference types
---------------

A reference types is a type that refers to another value of the type's `base type`. A value with this type does not contain the data of the `base type` it references.

.. code-block::

    ref-type = '&', type;

Array types
-----------

.. _`array type`:

An array type contains a range of values, each being of the type of the `base type`. An array type has its size known at compile-time. While an expression for an array can be used, the expression needs to be able to be calculated at compile time

.. code-block::

    array-type = '[', expression, ']', type;

Slice types
-----------

A slice type is similar to an `array type`_, but has no known size at compile time. As a consequence of not having a size known at compile time, a slice cannot own any memory.

.. code-block::

    slice-type = '[', ']', type;

Tuple types
-----------

A tuple type is a compound of multiple different subtypes. Like an array, the size of a tuple is defined at compile-time.
A tuple can be empty, in this case, the empty-tuple acts like the 'void' type in C.

.. code-block::

    tuple-type = '(', [ type, { ',', type }] , ')';


Optional types
--------------

Am optional type, is a type that may not have any value associated with it.

To prevent any issues with calling or accessing an optional type with no value, the type is required to be checked for `null`, before being able to use it. When the type has been checked with null and is guaranteed to have a value, the type is promoted to its `base type`.

.. code-block::

    optional-type = '?', type;

Null coalescing
```````````````

Optional types support coalescing operators, are certain operators starting with '?'. When the preceding expression is null, the null value is propagated, otherwise the expression is executed.
A special, null coalescing compatible operator can also be called on an optional type, the so called 'null-panicking' operator (postfix `!`).

Struct types
------------

A structure is a user defined type, which contains a range of contiguous members data.

A structure declaration defines a new user declared struct.

There are 2 possible 'types' of structs:

- Named struct: A struct declared with an name, this is the default type of struct
- Inline struct: A struct, which' type is not accessible, but the variable being assigned that type can still access the members of it.

A structure can be defined using a struct declaration:

.. code-block::

    struct-decl = { struct-attribute }, struct, identifier, [generic-decl], '{', { typed-var-decl }, '}';


It should be noted that struct may not contain a variable with the struct as its type, or with a type, that includes the current type, since this would create a circular dependency, resulting in a struct that would be infinite in size. If the struct needs to be self referential, the use of a pointer or a reference should be used.

.. code-block::

    struct S
    {
        s : S // illegal, self referential struct
    }

    struct S0
    {
        s1 : S1 // Illegal, self referential struct via 'S1'
    }

    struct S1
    {
        s0 : S0
    }

Union types
-----------

A union type is a user defined type, which consist of a group of members, which occupy the same memory

There are 2 possible 'types' of unions:

- Named union: A union declared with an name, this is the default type of union
- Anonymous union: A union declared without a name, this can only be used in certain places.

A union can be defined using a struct declaration:

.. code-block::

    union-decl = { union-attribute }, 'union', identifier, [generic-decl], '{', { typed-var-decl }, '}';

It should be noted, that even when all members overlap the same memory, a union may not contain a variable with the union as its type, or with a type, that includes the union type, since this would create a circular dependency, while not creating union with an infinite size, unlike a struct, this is a bad practice and will therefore count as an error. If the struct needs to be self referential, the use of a pointer or a reference should be used.

.. code-block::

    union S
    {
        s : S // illegal, self referential struct
    }

    union S0
    {
        s1 : S1 // Illegal, self referential struct via 'S1'
    }

    union S1
    {
        s0 : S0
    }

Enum types
----------

An enum is a user declared type, that contains a collection of values or tagged data.

There are 3 possible enum subtypes:

- Value enum
- Adt enum
- Inline enum

.. code-block::

    enum-declaration = value-enum-decl | adt-enum-decl;

Value enum types
````````````````

A value enum is an enum, where each member simply represents a values. Each member can have a value assigned, but this requires the value to be able to be calculated at compile-time. A value enum can have its underlying type explicitly be defined, if no underlying type is defined, i32 will be used as default, when a value is greater than 32-bits, the next smallest size of signed integer will be used.

A value enum can be declared with a value enum declaration:

.. code-block::

    value-enum-decl = { enum-attribute }, 'enum', identifier, [ ':', int-type ], '{', [ value-enum-member, { ',', value-enum-member }, [','] ], '}';
    value-enum-member = identifer, [ '=', expression ];

ADT enum types
``````````````

An ADT enum is an enum that represents a tagged union, meaning that each member is either an empty tag, or a tag for tuple or members connected with it. Unlike a value enum the value of a member can not be manually set, as an ADT enum will always try to use the smallest possible integer type as tag. When an adt enum has named members, the members are encapsulated in an inline struct.

An ADT enum can be declared with a value enum declaration:

.. code-block::

    adt-enum-decl = { enum-attribute }, 'enum', identifer, [ generic-decl], '{', [ adt-enum-member, { ',', adt-enum-member }, [','] ], '}';
    adt-enum-member = identifier, [ ( '(', type, { ',', type } ) |  ]

Inline enum types
`````````````````
An inline enum is an enum which is mostly meant to be the type of a function parameter. It is declared an a value enum, but as the type of a param and cannot assign a value to the members. 
After the declaration of the inline enum, the values can be access in the following way: `::InlEnumMember`, where `InlEnumMember` is the name of the member.

.. code-block::

    inline-enum = 'enum', '{', value-enum-member, { ',', value-enum-member } '}';

Interface types
---------------

An interface type is a user declared type, which does not hold data by itself, but imposes requirements for any type that wants to implement it.

Interfaces can only be declared in the module's scope, meaning they cannot be nested inside other declarations

There are 3 types of interfaces:

.. code-block::

    interface-decl = marker-interface-decl
                   | weak-interface-decl
                   | strong-interface-decl;
    interface-member = func-decl
                     | method-decl
                     | typealias-decl;

Marker interfaces
`````````````````

A marker interface is the simplest type of interface, since it just marks a type, because of this, they cannot have any members.

.. code-block::

    marker-interface-decl = 'interface', identifier, ';';

Weak interfaces
```````````````

A weak interface is an interface, which is implicitly implemented when the implementation for a type implements all required members.

.. code-block::

    weak-interface-decl = 'weak', 'interface', identifier, '{', interface-member, { interface-member } '}';

Strong interfaces
`````````````````

.. _`strong interfaces`:

A strong interface is an interface that needs to be explicitly implemented for a type.

    strong-interface-decl = 'interface', identifier, [generic-decl], '{', { interface-member } '}';

Multi interfaces
````````````````

Multi interfaces are a grouping of multiple interfaces, that may be used in certain location to note multiple interfaces that need to be implemented:

.. code-block::

    multi-interface = 'identifier', { '+', identifier };

Type alias
----------

A type alias is a way of referring to a type with a different identifier. When the typealias is part of an interface, no type needs to be given. Both the type and alias will be counted as the same type.

.. code-bloc::

    type-alias-decl = 'typealias', [ generic-decl ], identifier, [ '=', type ];

Typedef
-------

A typedef is similar to a type alias, but it creates a type that is distinct to the type it represents, meaning that a type and a typdef do not count as the same type.

.. code-block::

    typedef-decl = 'typedef', [ generic-decl ], identifier, '=', type.

Function types
--------------

A function type defines which type a function has, but is itself not a function, but defines the parameters that are that are taken and the type that gets returned. A method is a function, but which takes the receiver as it's first argument.

The last parameter is a variadic parameter which can take a 0 or more arguments. If a type is supplied, all variadic parameters will be of that type.

A function type can actually 3 different types of functions: free functions, methods and closures.

Each parameter can also have a parameter label, which is an identifier followed by '=>' before the varaible name. If this label is supplied, this is the name that will be used for function overloading. see `Function overloading`_

.. code-block::

    func-type = 'func', func-signature;
    func-signature = '(', [ parameters, { ',', parameters } ], [ variadic-parameter ] ')', [ func-throws ], [ '->', ret-type |  ]
    func-named-ret = '(', identifier, { ',', identifier }, ':', type, { ',', identifier, { ',', identifier }, ':', type }, ')';
    parameters = parameter-identifier-list, ':', type;
    parameter-identifier-list = parameter-identifier, { ',', parameter-identifier };
    parameter-identifier = { func-param-attribute }, [ identifier, '=>' ], identifier;
    variadic-parameter = identifier, '...'
                       | identifier, ':', type, '...';
    ret-type = type
             | '(', identifier, ':', type, { ',' identifier, ':', , type }, ')';


Func throws
```````````

A function can 'throw' an error. This is mostly syntactic sugar, as a function that throws will return a `Result` enum, which will either contain the actual return value, or an error value.
`throws` additionally guarantees, that when an error is returned, the error value needs to be explicitly checked, called with try, or have a null coalescing operator called on it.

.. code-block::

    func-throws = 'throws', [ '(', identifier-type, ')' ];

Note
----

For each qualified name, only a single user defined type can exist, regardless of generic parameters.

File structure
==============

A file follows the grammar, to produce a part of a module.

.. code-block::

    file = [ module-definition ], { module-statement | unit-test-statement | benchmark-statement };

Module definition
-----------------

The module definition defines which module the file is part of, and can additionally add attributes to the module, that can effect the generation of code.

.. code-block::

    module-definition = { module-attribute }, 'module', identifier, { '.', identifier };

Statements
==========

A statement is a piece of code, which can contain a collection of other statements or expressions. There are 2 types of statements:

- Module statements: these statements can be declared as a part of a file/module, or as part of another statement.
- Sub-statements: these statements cannot exist by themselves and need to be part of another statement.

.. code-block::

    module-statement = declaration | import-statement | conditional-compilation-statement;
    sub-statement = control-flow-statement 
                  | expressions-statement 
                  | var-decl 
                  | defer-statement 
                  | unsafe-statement
                  | error-handler-statement;
    statement = module-statement | sub-statement;

Import statements
-----------------

An import statement allows the use of symbols defined in a different module, while generating a dependency on that module (only if any symbol from that module is used).

There are modifiers that can change the behavior of the import::

- public: Gives access to all symbols imported by this statement to any module that imports this module.
- static: Imports symbols, without allowing the symbols to be used without their full qualified name.

An import can also select certain symbols that should be imported from the module, and can give the imported symbols another name.

.. code-block::

    import-statement = [ 'public' ], [ 'static' ], 'import', identifier, { '.', identifier }, [ ':' import-symbol { ',', import-symbol } ], ';'
    import-symbol = identifier, [ 'as', identifier ];

Declarations
------------

A declaration is a way of defining one of the following:
s
- User definable type
- Variable
- Function
- Method

.. code-block::

    declarations = struct-decl
                 | union-decl
                 | enum-decl
                 | interface-decl
                 | var-decl
                 | func-decl
                 | method-decl
                 | impl-decl;

Variable declarations
`````````````````````

A variable declaration generates one or more variables in the current scope. Variables can be declared with or without an explicit type, in case no type is explicitly defined, an expression is required to deduce the type of.

.. code-block::

    var-decl = { var-decl-attribute } typed-var-decl | untyped-var-decl;
    typed-var-decl = identifier-list, ':', type, [ '=', expression | block-expression ];
    untyped-var-decl = identifier-list, ':=', expression | block-expression;
    var-init-decl = expression | block-expression;

Function declarations
`````````````````````

.. code-block::

    func-decl = { func-attribute }, 'func', identifier, [ generic-decl ], func-signature, [ generic-where-clause ], '{', { statement }, '}';

Function overloading
^^^^^^^^^^^^^^^^^^^^

Function overloading in Noct works differently to most languages, instead of overloads being differentiated by the types of the parameters, they are differentiated by the name of the parameters..

.. code-block::

    // conventional overloading
    func Name(a:A) {}
    func Name(a:B) {} // Error in noct: function with parameter names already exists

    // noct overloading
    func Name(a:A) {}
    func Name(b:B) {}

If a function has no overloads, it can be called without specifying the name of the arguments passed, otherwise, the name of the argmument needs to be specified.

.. code-block::

    // functions
    func NoOverload(a:i32) {}
    func Overload(a:i32) {}
    func Overload(b:f32) {}
    func Overload(c => b:f64) {}

    // calls
    NoOverload(1);
    Overload(a:2);
    Overload(b:3.0f32);
    Overload(c:5.0);

Method declarations
```````````````````

.. code-block::

    method-decl = normal-method-decl | empty-method-decl;
    normal-method-decl = { method-attribute }, 'func', method-receiver, identifier, [generic-decl], func-signature, [ generic-where-clause ], '{', { statement }, '}';
    empty-method-decl = { method-attribute }, 'func', method-receiver, identifier, [generic-decl], func-signature, ';';
    method-receiver = '(', [ '&', [ 'const' ] ], 'self', ')';

Implementation declaration
``````````````````````````

An implementation declaration allows methods and specific members to be implemented for a specific type, the statement can also implement `strong interfaces`_ for a type.

.. code-block::

    impl-decl = 'impl', generic-decl, type, [ ':', type ], '{', { statement }, '}';

Block statement
---------------

A block statement is a collection of statements, that are defined in an inner scope of the scope the statement resides.

.. code-block::

    block-statement = '{', { statement }, '}';

Control flow statements
-----------------------

A control-flow statement affect how code will be executed, dependent on one or multiple values.

.. code-block::

    control-flow-statement = if-statement
                           | loop-statement
                           | while-statement
                           | do-while-statement
                           | for-statement
                           | switch-statement
                           | label-statement
                           | break-statement
                           | continue-statement
                           | fallthrough-statement
                           | goto-statement
                           | return-statement
                           | comp-if-statement
                           | cond-comp-statement;

If statements
`````````````

.. _`if statement`:

An if statement alters the control-flow, depending on a condition.

.. code-block::

    if-statement = 'if', [ var-decl ';' ], ? expression, except aggr-init-expression ? | block-expression, block-statement, [ 'else', ( if-statement | block-statement ) ];

Loop statements
```````````````

A loop statement executes its `body` will be continued to be executed, until the loop is explicitly exited. Because of this, a loop statement is required to have reachable code to exit the loop.

.. code-block::

    loop-statement = [ label-statement ], 'loop', statement;

While statements
````````````````

A while statement executes its `body`, while its condition is `true`.

.. code-block::

    while-statement = [ label-statement ], 'while', ? expression, expect aggr-init-expression ? | block-expression, '{', statement, '}';

Do-while statements
```````````````````

A do-while statement is similar to a while statement, with the difference being that the `body` is guaranteed to execute at least once.

.. code-block::

    do-while-statement = [ label-statement ], 'do', '{', statement, '}', 'while', ? expression, expect aggr-init-expression ? | block-expression, ';';

For statements
``````````````

.. _`for statement`:

A for-range statement iterates over a range of values. It will run over all the values that are part of the range given to the statement.
After the range of the for loop, an optional where clause can be added, this clauses is an expression that returns a boolean value and decides if the iteration needs to be looped over.

.. code-block::

    for-range-statement = [ label-statement ], 'for', identifier-list, 'in', ? expression, expect aggr-init-expression ?, [ for-where-clause ], '{', statement, '}';
    for-where-clause = 'where', expression;

Switch statements
`````````````````

A switch statement does a pattern match on a given value and changes the code flow based on that. All possible paths are defined as cases, a case exists out of 3 elements::

- Pattern: The pattern to match when switching the value.
- Expression: An additional runtime expression, which can be used to distinguish between multiple cases with the same static expression, these conditions are check from top to bottom.
- Statement: A statement to be executed when the case is selected.

If a case is defined as '_', and no dynamic expression is included, this is used as the default case.
Each case will automatically break after the execution of its statement, unless that statement ends in a fallthrough.

.. code-block::

    switch-statement = 'switch', ? expression, expect aggr-init-expression ?, '{', { 'switch-case' }, '}';
    switch-case = pattern, [ 'where', expression ], '=>', statement;

Label statements
````````````````

A label statement defines a location where certain statements may go to. A label is only valid inside of the scope in which it is defined, this is done to prevent edge cases that can be caused by entering an inner scope.

.. code-block::

    label-statement = ":", identifier, ':'


Break statements
````````````````
A break statement can be used to exit a loop, if an optional identifier is added, the break will exit all loops until the loop with the specific label is exited.

.. code-block::

    break-statement = 'break', [ identifier ], ';';

Continue statements
```````````````````

A continue statement will skip the code in the body of a loop and will go to the next iteration of that loop, if an optional identifier is added, the continue will skip to the next iteration of the loop with the specific label.

.. code-block::

    continue-statement = 'continue', [ identifier ], ';'

Fallthrough statements
``````````````````````

A fallthrough statement can cause a case of a switch statement to continue executing the next case, instead of automatically exiting that case.
s

.. code-block::

    fallthrough-statement = 'fallthrough', ';';

Goto statements
```````````````

A goto statement can jump to any label, as long as that label is defined in the same scope, or one of the outer scopes of the scope where the goto is defined. It can not jump into an inner scope, or any scope that is not reachable from the scope it is in.

.. code-block::

    goto-statement = 'goto', identifier, ';';

Return statements
`````````````````

A return statement exist the current function, with a possible value. Multiple values can be returned, if the function it is in, returns a tuple.

.. code-block::

    return-statement = 'return' [ expression, { ',', expression } ], ';'

Throw statements
----------------

A throw statement can be called within any function that is defined as throws, it will early out the function and return the error supplied.

.. code-block::

    throw-statement = 'throw', expression, ';';

Expression statements
---------------------

An expression statement allows an expression to be used as a statement.

.. code-block::

    expression-statement = expressions, ';';

Defer statements
----------------

A defer statement delays the execution of the expression following it. There are 2 possible ways to defer an expression.

Scoped defer statements
```````````````````````

A scoped defer statement will execute its code when the current scope is exited, only defers that are defined in the same scope will be executed on scope exit.

.. code-block::

    defer-statement = 'defer', expression, ';';

Error defer statements
``````````````````````

An error defer will only execute when a function is exited with a `throw` or catches an error using `try`, this allows the function to cleanup, before returning the error.

.. code-block::

    error-defer-statement = 'errdefer', expression, ';';

Unsafe statement
----------------

An unsafe statement is a statement in which any statements, not deemed safe, can be executed.

.. code-block::

    unsafe-statement = 'unsafe', '{', { statement }, '}';

Compile-time statements
-----------------------

Compile-time if statements
``````````````````````````

A compile time if expression selects the branch to take at compile-time.

.. code-block::

    comp-if-statement = '#if', [ var-decl ';' ], ? expression, except aggr-init-expression ? | block-expression, block-statement, [ 'else', ( if-statement | block-statement ) ];

Conditional compilation statements
``````````````````````````````````

A conditional compilation statement is a statement where the body will only be executed when certain compile conditions are met.

.. code-block::

    cond-comp-statement = ( '#conditional' | '#debug' ), ' identifier, [ cond-cmp, int-lit ], block-statement, [ 'else', ( cond-comp-statement | block-statement ) ];
    cond-cmp = '==' | '!=' | '<' | '<=' | '>' '>=';

Error handler statement
-----------------------

An error handler statement is used when calling `try` and an error gets returned. When this statement is present, the `try` will call this handler, instead of propagating the error.

.. code-block::

    error-handler-statement = '#errorhandler', '(', identifier, [ ':', type ], ')', '{', { statement }, '}';

Unit test statements
--------------------

A unit test statement is used to run unittests on code.
The `std.unittest` module is required to run a benchmark.

.. code-block::

    unit-test-statement = '#unittest', string-lit, '{', { statement }, '}';

Benchmark statements
--------------------

A benchmark statement allows the user to run a benchmark. A context is provided to allow the user to pause and resume the benchmark, and to know how long the benchmark needs to keep running. 
The `std.bench` module is required to run a benchmark.

.. code-block::

    benchmark-statement = '#benchmark', string-lit, '(', identifier ')', '{', { statement }, '}';

Expressions
===========

.. code-block::

    expression = assign-expr;

Assignment expressions
----------------------

An assignment expression allows a value to be assigned, to one or more variables. Values can also be modified, depending on the assignment operator used.
Unlike other operators, the assignment operator is right associative, meaning that the value on the right of the operator has precedence over the assignment, with the exception of `??=`, where the left has precedence, since `??=` depends on the value of the left expression.

.. note::

    The expression on either side needs to conform to auto-referencing, unless the basic `=` is used, where only the left expression needs to conform

.. code-block::

    assign-expr = ternary-expression, [ assign-op, assign-expression ];
    assign-op = '='
              | '+='
              | '-='
              | '*='
              | '/='
              | '%='
              | '~='
              | '<<='
              | '<<<='
              | '<<*='
              | '>>='
              | '>>>='
              | '>>*='
              | '&='
              | '^='
              | '|='
              | ??=;

========== =================================================== ====================
 Operator   Description                                         Overload Interface
========== =================================================== ====================
 `+=`       addition                                            OpAddAssign
 `-=`       subtraction                                         OpSubAssign
 `*=`       multiplication                                      OpMulAssign
 `/=`       division                                            OpDivAssign
 `~=`       concatenation                                       OpConcatAssign
 `&=`       binary and                                          OpBinAndAssign
 `^=`       binary xor                                          OpBinXorAssign
 `|=`       binary or                                           OpBinOrAssign
 `<<=`      shift left                                          OpShlAssign
 `<<<=`     'arithmetic' shift left                             OpAShlAssign
 `<<*=`     rotate left                                         OpRotlAssign
 `>>=`      shift right                                         OpShrAssign
 `>>>=`     arithmetic shift right                              OpAShrAssign
 `>>*=`     rotate right                                        OpRotrAssign
 `??=`      null-coalescing assign (assign if left is `null`)   n/a
========== =================================================== ====================

Ternary expressions
-------------------

A ternary expression is similar to an `if statement`_, but selects one of two values depending on a condition. Since this is an expression, it is required that both possible options have the same type.

.. note::

    If both conditional expressions conform to auto-referencing, the ternary expression also conforms to auto-referencing

.. code-block::

    ternary-expression = binary-expression, [ '?', ternary-expression, ':', ternary-expression ];

Binary expressions
------------------

A binary expression uses 2 values, on both sides of it, to generate a new value.

.. note::

    The expression on either side needs to conform to auto-referencing

========== =============================== ====================
 Operator   Description                     Overload Interface
========== =============================== ====================
 `+`        addition                        OpAdd
 `-`        subtraction                     OpSub
 `*`        multiplication                  OpMul
 `/`        division                        OpDiv
 `~`        concatenation                   OpRem
 `&`        binary and                      OpBinAnd
 `&&`       logical and                     n/a
 `|`        binary or                       OpBinOr
 `||`       logical or                      n/a
 `<`        less than                       OpPartialEq
 `<<`       shift left                      OpShl
 `<=`       less or equal than              OpPartialEq
 `<<<`      'arithmetic' shift left         OpAShl
 `<<*`      rotate left                     OpRotl
 `>`        greater then                    OpPartialEq
 `>>`       shift right                     OpShr
 `>=`       greater or equal than           OpPartialEq
 `>>>`      arithmetic shift right          OpAShr
 `>>*`      rotate right                    OpRotr
 `==`       equal to                        OpEq
 `!=`       not equal to                    OpEq
 `..`       range [) (exclusive)            OpRange
 `..=`      range [] (inclusive)            OpRangeInc
 `??`       null coalescence                n/a
 `?:`       elvis operator                  n/a
 `in`       contains operator               OpContains
 `!in`      inverted contains operator      OpContains
========== =============================== ====================

.. code-block::

    binary-expression = postfix-expression, [ bin-op, binary-expression ]
    bin-op = '+'
           | '-'
           | '*'
           | '/'
           | '%'
           | '~'
           | '&'
           | '&&'
           | '|'
           | '||'
           | '^'
           | '<'
           | '<<'
           | '<='
           | '<<<'
           | '<<*'
           | '>'
           | '>='
           | '>>'
           | '>>>'
           | '>>*'
           | '=='
           | '!='
           | '..'
           | '..='
           | '??'
           | '?:'
           | 'in'
           | '!in';

Operator precedence
```````````````````

A lower precedence means it will be executed before operators with a higher precedence

============ ===================================
 precedence   operators
============ ===================================
 0            `*` `/` `%` `~`
 1            `+` `-`
 2            `<<` `<<<` `<<*` `>>` `>>>` `>>*`
 3            `&`
 4            `^`
 5            `|`
 6            `..` `..=`
 7            `in` `!in`
 8            `==` `!=` `<` `<=` `>` `>=`
 9            `??` `?:`
 10           `&&`
 11           `||`
============ ===================================

Unary expressions
-----------------

A unary expression takes in a value, and returns another value, depending on the operand.

========== ============================ ====================
 Operator   Description                  Overload Interface
========== ============================ ====================
 `+`        positive                     OpPos
 `++`       increment                    OpInc
 `--`       negative                     OpNeg
 `-`        decrement                    OpDec
 `!`        logical negation             OpNot
 `~`        binary negation              OpBinNeg
 `*`        dereference                  OpDeref
 `&`        address of/ref               n/a
 `!!`       true-ish or null-panicking   OpNullPanic
========== ============================ ====================

.. code-block::

    postfix-expression = ( postfix-expression | prefix-expression ), [ postfix-op ];
    prefix-expression = [ prefix-op ], ( operand | prefix-expression );
    postfix-op = '++'
               | '--'
               | '!!';
    prefix-op = '+'
              | '++'
              | '-'
              | '--'
              | '!'
              | '~'
              | '*'
              | '&'
              | '!!';

.. note:

    The '&' has some special behavior and depends on the type expected, the operator will return a reference to the sub-expression, with the exception for when the expected type for the expression is a pointer type, where it will return a pointer to the sub-expression

Operands
--------

An operand is a value, where operators can be called on. These are things like sub expressions and calls.

.. code-block::

    operand = qualified-name-expression
            | index-slice-expression
            | function-call
            | member-access
            | method-call
            | tuple-access
            | literal-expression
            | init-expression
            | cast-expression
            | transmute-expression
            | move-expression
            | bracketed-expression
            | block-expression
            | unsafe-expression
            | is-expression
            | try-expression
            | throw-expression
            | comp-run-expression;

Auto referencing
````````````````

Certain expression have auto-refencing, meaning that the compiler will implicitly take a reference to the sub expression if needed.
Auto referncing will only happen in very specific cases:

- On any expression that explicitly references a varaible, e.g. qualified name expressions
- On any intermediate values, e.g. values returned from functions before being assigned
- On any literal

Auto-referencing can happen for the following expressions:
- Operators: since they work on references to const types
- Method calls (caller/receiver): When a method is called, which takes in the caller as a reference, automatic referencing on that caller


Qualified name expressions
--------------------------

A qualified name expression is an expression that refers to a variable.

.. code-block::

    qualified-name-expression = identifier, { '::', identifier };

Index and slice expressions
---------------------------

An index expression allows you to access an element of any type which has an index operator defined, a builtin example of this is the array.
A slice expression on the other hands will always generated a value with a slice type, and can therefore contain a range of value, instead of one. A slice can also be created using a special slice index, which exists out of 2 expressions, separated by a colon, while either expression can be optional, at least 1 needs to be defined. If no value is defined before the colon, this will be interpreted as the first value, the latter is similar, but will be interpreted as the last value.
If the null-coercing version is used, the expression will return a nullable value.

.. code-block::

    index-slice-expression = expression, ( '[' | '?[' ), ( expression | slice-index ), ']';
    slice-index = [ expression ], ':', [ expression ];

Function call expressions
-------------------------

A function call is an expression that can generate an expression, based upon the arguments passed to the function being called. It can only be used as an operand for another expression, if the function being called, returns a value. Each argument passed to the function, can be prefixed by the name of the parameter, which will than be passed as the value for that parameter.

.. code-block::

    func-call = qualified-name, '(', [ argument, { ',' argument } ], ')';
    argument = [ identifier, ':', ], expression;

Member access expressions
-------------------------

A member access expression retrieves the value of the member that is selected by an identifier.
If the null-coercing version is used, the expression will return an optional value.

.. code-block::

    member-access = operand, ( '.' | '?.' ), expression;

Method call expressions
-----------------------

A method call is very similar to a function, but call a method that is defined by the type of the value it is called on.
If the null-coercing version is used, the expression will return an optional value.

.. code-block::

    method-call = operand, ( '.' | '?.' ), expression, '(', [ argument, { ',', argument } ], ')';

Method resolution
`````````````````
When a disambiguous method is implement on a type itself and/or one or multiple interfaces, the following steps are used:
1. If the method is available on the type itself (outside of interface implementations), that method will be used
2. Otherwise, a qualified name with a type disambiguation is required

Tuple access Expressions
------------------------

A tuple access expression retrieves a value at a specific index in the tuple. While this function may seem similar to index with an integer, the statement is not called dynamically, but generates specific code to access that 'member'.
If the null-coercing version is used, the expression will return an optional value.

.. code-block::

    tuple-access = expression, ( '.' | '?.' ), int-lit;

Literal expressions
-------------------

A literal expressions allows the use of a literal as an expression.

.. code-block::

    lit-expression = literal;

Initialize expressions
----------------------

.. code-block::

    init-expressions = struct-init
                     | union-init
                     | enum-init
                     | tuple-init
                     | array-init
                     | empty-init;

Struct initialize expressions
`````````````````````````````

An aggregate initialize expressions is create a new instance of a struct with each member being assigned a specific value. Each member is required to be initialized.

.. code-block::

    struct-init = qualified-name, '{', [ argument, { ',', argument } ], '}';

Union initializer
`````````````````

A union initialize expressions is create a new instance of a union where exactly one member of the union is assigned, if it happens that the specific member being initialize contains multiple values, all values in that member need to be initialized.

.. code-block::

    union-init = qualified-name, '{', [ argument, { ',', argument } ], '}';

Enum initializer
````````````````

An enum initialize expressions is create a new instance of a enum, how the enum is initialized, depends on the member. If the member just represents a value, the qualified name of it is used. If the member is a tuple member, it is initialized like a tuple, but with the qualified name before the initializer. Otherwise the member is initialized as if it's a struct.

.. code-block::

    enum-init = value-enum-member-init | tuple-enum-member-init | struct-enum-member-init;
    value-enum-member-init = qualified-name;
    tuple-enum-member-init = qualified-name, '(', expression, { ',', expression }, ')'; 
    struct-enum-member-init = qualified-name, '{' argument, { ',', argument }, '}';

Tuple initializer
`````````````````

A tuple initializer creates a new instance of a tuple, with each member being given a value in the order that they are defined inside of the enum.

.. code-block::

    tuple-init = '(' expression, ',', expression, { ',', expression }, ')';

Array initializer
`````````````````

An array initializer creates a new instance of an array, with the same amount of elements being passed to the initializer. The type of each element needs to be the same as the others.

.. code-block::

    array-init = '[' expression, { ',', expression }, ']';

Empty initializer
`````````````````

An empty expression is a special type of expression, which can be used when declaring a variable, without initializing it. This also means that any use of a variable without the actual initialization is illegal.

.. code-block::

    empty-init = '_';

Cast expressions
----------------

A cast expression converts a value from one type to another.

.. code-block::

    cast-expression = simple-cast-expression
                    | safe-cast-expression
                    | null-panicing-cast-expression;

    simple-cast-expression = operand, 'as', type;
    safe-cast-expression = operand, 'as?', type;
    null-panicing-cast-expression = operand, 'as!', type;

Transmute expression
--------------------

A transmute expression converts a value from one type to another, by the way of a bit cast.

.. code-block::

    transmute-expression = operand, 'transmute', type;

Move expressions
----------------

A move expression moves a value from one location to another, the value that is moved from, will become invalid after this statement and can not be used after it.

.. code-block::

    move-expression = 'move', operand;

Bracketed expressions
---------------------

Bracketed expressions are sub-expression that will be executed, before any other the outer expression can be executed.

.. code-block::

    bracketed-expression = '(', expression, ')';

Block expressions
-----------------

A block expression is a special type of expression, which acts as if its a block statement, but it returns a value at the end of the block.

.. code-block::

    block-expression = '{', { statement }, return-statement, '}';

Unsafe expressions
------------------

An unsafe block expression is a special type of expression, which acts as if its an unsafe block statement, but it returns a value at the end of the block.

.. code-block::

    unsafe-block-expression = 'unsafe', expression;

Comma expression
----------------

A comma expression is a expression that exists out of multiple sub-expressions. It is limited to certain locations where it can be used.

.. code-block::

    comma-expression = expression, { ',', expression };

Closure expression
------------------

A closure expression generates a new closure.

The parameters of a closure expression can be written without a type, when the types of the variables are inferable from the surrounding code.

A closure may capture variables from the scope it's declared in, their are 2 types of captures: global and local. The global capture will only count for variables that do not have a local capture.

global::

    - `=`: The captured variables are copied
    - `&`: The captured variables are references to the variables
    - `move`:  The captured variables are moved

local::

    - `iden`: the specific variable will be copied
    - `&iden`: the specific variable will be a reference to the variable
    - `move`:  The captured variables are moved

.. code-block::

    closure-expression = '|', closure-param, { ',', closure-var }, '|', closure-ret, closure-captures, closure-body;
    closure-param = identifier-list, [ ':', type ];
    closure-ret = '->', type;
    closure-body = expression;

Closure captures
````````````````

When using a variable that is declared outside of the closure is being used, the compiler tries to be as smart as possible whe it comes to using the capture. To capture, the following rules will be used:

- If the variable is a reference, it will be captures as a reference.
- If the variable is used after the closure, the closure will copy the variable (for optimization, the compiler is allowed to move the closure expression behind the last use of the captured variable, in case this variable is not written to and its last use is after the first use of the closure).
- Otherwise, the variable will be moved into the closure

Is expression
-------------

The is expression checks if a variable of a specific type or implements a specific interface, or if it isn't a specific type or it doesn't implement a specific interface.
When the is-expression is being used as a condition in a control-flow statement, the expression is a variable and the type is not an interface type, the exclusive path in the control-flow will promote that variable to the type being compared against.

.. code::

    is-expression = [ expr ], 'is' | '!is' , type;

Try expressions
---------------

A try expression will run a function that can 'throw'.
When an error handler is defined inside of the function it is called from, the error handler will be called. Otherwise it will propagate the error and 'throw' in the calling function. (When no error handler is defined, the function that uses the try expression, is required to be able to throw a compatible error.)

.. code-block::

    try-expr = 'try', operand;

Special keyword expressions
---------------------------

A special keyword expression is an expression with a single keyword, that will be replaced with a specific value during compile time.

List of keyword meanings

- #file: File name (string literal)
- #fileFullPath: File name with full path (string literal)
- #package: Package name (string literal)
- #module: Module name (string literal)
- #fullModule: Package and module name (string literal)
- #line: Line number (integer literal)
- #func: Function name with simple signature (string literal)
- #funcName: Name of function (string literal)
- #prettyFunc: Function name, including namespace and full signature (string literal)

.. code-block::

    special-keyword-expression = '#file'
                               | '#fileFullPath'
                               | '#package'
                               | '#module'
                               | '#fullModule'
                               | '#line'
                               | '#func'
                               | '#funcName'
                               | '#prettyFunc';

Compile-time expressions
------------------------

compile-time run expression
```````````````````````````

A compile-time run expression execute an expression at compile time.

.. code-block::

    comp-run-expression = '#run', expression;

Patterns
========

A pattern can be used to match against a given value, e.g. the value passed to a switch-statement.

.. code_block::

    pattern = placeholder-pattern
                | wildcard-pattern
                | value-bind-pattern
                | literal-pattern;
                | tuple-pattern
                | enum-pattern
                | aggr-pattern
                | slice-pattern
                | either-pattern;
                | type-pattern;

Pattern List
------------


Placeholder pattern
-------------------

A placeholder pattern matches any value.

.. code-block::

    placeholder-pattern = '_';

Wildcard pattern
----------------

A wildcard pattern is like an empty-pattern, but matches all remaining values.

.. code-block::

    wildcard-pattern = '...';

Value bind pattern
------------------

A value bind pattern is defined using an identifier, this identifier will than be bound to a certain value in the expressions matched to the pattern. A bound value may have a bound attached to it, which is represented by an additional pattern added to it.

.. code-block::

    value-bind-pattern = identifier [ '->', pattern ];

Literal pattern
---------------

A literal pattern matches when the respective value is the same as the literal.

.. code-block::

    literal-pattern = ? any literal, except floating point literals, because of possible floating point inaccuracies ?;

Range pattern
-------------

A range pattern matches any value that fits into the range it creates. This does limit the pattern to be used with sub-patterns that have a builtin type.

.. code-block::

    range-pattern = pattern, ( '..' | '..=' ), pattern;

Tuple pattern
-------------

A tuple pattern matches the values inside a tuple.

.. code-block::

    tuple-pattern = '(', simple-pattern, { ',', simple-pattern }, ')';

Enum pattern
------------

An enum pattern matches the enum member that corresponds to the given value. This type of pattern can only match value enums or adt enum members with a tuple type. To match enum with an aggregate type, the aggregate pattern should be used.

.. code-block::

    enum-pattern = qual-name, [ '(' , simple-pattern, { ',', simple-pattern }, ')' ];

Aggregate pattern
-----------------

An aggregate pattern matches the aggregate that corresponds to the given value. 
Members of an aggregate type are not required to be matched in the same order as they appear in the aggregate, in this case, the identifier of the member can be added to each sub pattern. When matching members to patterns, all members have to be matched in that way.

.. code-block::

    aggr-pattern = qual-name, '{', simple-pattern, { ',', simple-pattern }, '}';
    agg-sub-pattern = [ identifier, ':' ], pattern;

Slice pattern
-------------

A slice pattern matches any array or slice with the corresponding size.

.. code-block::

    slice-pattern = '[', pattern, { ',', pattern }, ']';

Either pattern
--------------

An either pattern matches is one of its sub-patterns matches the given value

.. code-block::

    either-pattern = pattern, '|', pattern, { '|', pattern };

Type pattern
------------

A type patterns matches when the type of the given value corresponds to the type supplied to the pattern.

.. code-block::

    type-pattern = 'is', type;


Attributes
==========

Compiler attributes
-------------------

A compiler attribute provides a hint to the compiler on how it should handle a certain piece of code.

=================== ============ ============================================================================================================================ ==============================
 Attribute           Arguments   Description                                                                                                                   Restrictions (None if empty)
=================== ============ ============================================================================================================================ ==============================
 align               i16          Set the minimum alignment of a type, alignment specified in range (1--512)
 inline              inl-kind     Hints how to inline a function (never -> will never inline, prefer -> try to inline if possible, always -> force inlining)
 no_mangle                        Just use the function name as the mangled name
 mangle_name         string-lit   Set a specific mangled name to use


 use_outside_macro                Allows a declaration made inside of a macro to permeate out of a macro 
=================== ============ ============================================================================================================================ ==============================

.. code-block::

    compiler-attribute = '@:', identifier, [ '[', arg, { ',', arg }, ']' ];

User defined attributes
-----------------------

A user defined attribute is a custom attribute that can be defined by the user, by creating a struct type that implements the `UserDefAttrib` attribute.

.. code-block::

    user-def-attribute = '@', identifier, [ '[', arg, { ',', arg }, ']' ];

Visibility attributes
---------------------

The visibility attributes modifies the scope where the variable is available. By default, every variable is private to its scope, but this can be changed.

When the `public` attribute is used, it can specify a scope in which it is public, if none is specified, it will be accessible from anywhere it is imported, with the exception for when it is dynamically linked::

- `module`: It is accessible from anywhere in the module
- `package`: It is accessible from anywhere in the package
- `dynlib`: It is public, but also accessible when dynamically linked.

.. code-block::

    visibility-attribute = 'public', [ '(', visibility-scope, ')' ]
    visibility-scope = 'module'
                     | 'package'
                     | 'dynlib''

Type Attributes
---------------

A type modifiers changes the meaning of how the type stores a value.

- `const`: A const type is an immutable 'reference' to a value, meaning that the variable references a value, that can be changed by this 'reference' to the value, but might be modified by another reference to

.. code-block::

    type-modifiers = 'const';

Function and method Attributes
------------------------------

A function attribute modifies the generation and execution of a function. The same counts for methods.

- `comptime`: Compile time function, will not generate any runtime code.

.. code-block::

    func-attribute = compiler-attribute
                   | user-def-attribute
                   | 'comptime';

Parameter attributes
````````````````````

A function parameter can have an attribute, which tells how the argument is passed to the funtion::

- `move`: The value will always be moved into the function, invalidating the variable passed to the function
- `lazy`: The value will be passed lazily, meaning that all execution, which only needs to be done for that variable, will be passed lazily to the function and will only be calculate on the first use of that parameter.

.. code-block::

    func-param-attribute = 'move'
                         | 'lazy';

Variable declaration attributes
-------------------------------

A variable declaration attributes modifies the variables that are declared.

- `const`: Compile-time constant, type needs to be provided in the declaration.
- `lazy`: The execution to initialize the variable will only be executed on the first use.
- `static`: Only one version of this variable will exist and it will keeps its value between uses.

.. code-block::

    var-decl-attribute = compiler-attribute
                       | user-def-attribute
                       | 'const'
                       | 'lazy'
                       | 'static';


Other attributes
----------------

All attributes in this section are a collection of compiler, user defined, or visibility attributes and do not have any special attribute on top of those.

.. code-block::

    struct-attribute = compiler-attribute | user-def-attribute | visibility-attribute;
    union-attribute = compiler-attribute | user-def-attribute | visibility-attribute;
    enum-attribute = compiler-attribute | user-def-attribute | visibility-attribute;
    module-attribute = compiler-attribute | user-def-attribute;
    macro-attribute = visibility-attribute;

Generics
========

Generics allows the reuse of code for different types.

Declarations
------------
 
A generic declaration defines what parameters the generic can use. 
There are 2 types of generic parameters that exists::

- Type parameter: A type parameter can be used as a type inside of the generic and it can have a default type. In addition to this, a simple constraint can be added, by defining what interfaces the type should implement.
- Value parameter; A value parameter is any value that can passed to a parameter of the value type.

Allowed value parameter types:
    - Reference
    - Pointer
    - Builtin integer type (signed and unsigned)
    - value enum

.. note::

    Floating points types were deliberatly excluded, since these can cause issues when getting them as the result of a compile time function, caused by the characteristics of floating point math

.. note::

    Unless there is any need, other types will probably not be added to valid types for value generics

.. code-block::

    generic-decl = '<', generic-param, { ',', generic-param }, '>';
    generic-param = generic-type-param | generic-value-param | generic-param-specialization;
    generic-type-param = identifier, [ 'is', type ], [ '=', type ];
    generic-value-param = identifier, ':', type, [ '=', expression ];

Where clause
------------

The where clause can add additional constraints to a declaration, where the version with the where clause will only be used, if the where clause results in `true`.

.. code-block::

    generic-where-clause = 'where', type-bound, { ',', type-boundS, };
    type-bound = type, 'is', type { '+', type };

Specializations
---------------

It is possible to specialize a generic for certain types and values. To specialize, first a base generic (no specializations) is needed, after this, a specialization can be created by repeating the base generic, but exchanging certain parameters by the specialized values, with a ':' before it.

.. code-block::

    generic-specialization = ':', ( type | '{', expression, '}' );

Instantiation
-------------

To use anything with generics, the generic needs to be instantiate.

.. code-block::

    generic-instance = '!<', generic-arg, { ',', generic-arg }, '>';
    generic-arg = type | '{', expression, '}';

Generic instance collision resolution
-------------------------------------

When using generics, it is possible that multiple versions of a generic type can be used for a single instantiation: specializations.
A first pass is done, which excludes any generics where the `where clause` evaluates to true, then these rules are followed::

- If there is a full specialization, use the specialized version.
- If there is no specialization, use the only version there is.
- If there is exactly one partial specialization, use that specialization.
- If there are multiple specializations:
    - If a specialization is a better fit, i.e. more args match, use that one.
    - If 2 or more have the same number of matching args, go over all of them left to right, and pick the first one that matches all args.
    - Otherwise, generate an error

Limitations
-----------

- Multiple generics with the same identifier, are required to be of the same type, i.e. `struct`, `union`, etc
- Multiple generics with the same name, and same number of parameters, are not allowed. No distinguishment is made between type or value parameters, only the number of arguments is distinguished.

Macros
======

A macro is a way of expressing code, that can be manually separated form the code, even when the use of a function or a generic is not possible. The macro system is an AST-based macro system and should therefore contain syntactically correct code, although this is not guaranteed to generate valid code semantically.

Any macro is expected to result in one of the following kinds::

- block (statement or expression)
- statement
- expression
- pattern
- type

Anything else will result in invalid code.

Declarative Macro
-----------------

A declarative macro or static macro is a macro that has a predefined body, which will only differ because of the pattern it is matched against, but it cannot produce code during compile time.

Macro declaration
`````````````````

A macro can be defined in 2 ways. When only a single pattern is needed, the pattern can be added behind the identifier. If multiple patterns are needed, a macro with cases can be used. When a macro has cases, they will be checked from top to bottom and it will try to use the first macro with a matching pattern, if an error occurs during instantiation, it will not try any of the other cases, this needs to be taken in mind when defining a macro with multiple patterns.

.. code-block::

    decl-macro = [ macro-attribute ], 'macro', identifier, '(', macro-pattern, )', '{', { statement } '}', ';'
               | [ macro-attribute ], 'macro', identifier, '{', macro-rule, { ',', macro-rule } '}';
    macro-rule = '(', macro-pattern, )', '=>', '{', { statement } '}';

Procedural macros
-----------------

Procedural macros are a bit different to declarative macros, as these are not a special declaration, but instead a specific compile-time function that can generate a stream of tokens. Each procedural macro get a token stream, which' identifier is defined in the first set of parentheses and the body is required to return a token stream.

Like declarative macros, procedural macros can also have multiple patterns to match to.

.. code-block::

    proc-macro = [ macro-attribute ], 'macro', 'func', identifier, '(', identifier, ')', '(', macro-pattern, ')', '{', {statement} '}'
               | [ macro-attribute ], 'macro', 'func', identifier, '(', identifier, ')', '{', macro-rule, { ',', macro-rule } '}';

Matching
--------

When a macro is evaluated of instantiation, a pattern is needed to match to the input parameters. This match exists out of 0 or more sub-patterns and if possibly followed by a repetition (requires more than 1 sub-pattern to exist).

A sub-pattern can be a single variable, or a group of variables, separated by a specific token. A group can consist out of just 1 variable, when this happens, it will be ignored if no repetition characters follow it.

Each macro variable has a special kind defined after the identifier:

- stmt: Statement
- expr: Expression
- type: Type
- qual: Qualified name
- iden: Identifier
- attr: Attribute
- toks: Token stream (A single token or tokens surrounded with '{}', '[]' and '()')(must contain matching bracket)
- patr: Pattern

Each macro variable kind also has specific characters that may follow it, this is to prevent any future addition to the syntax from braking the macros
Below is a list of allowed tokens after the variable::

- `stmt` and `expr`: `=>` `,` or `;`
- `type`, `iden` or `qual`:  `=>` `,` `;` `:` `=` `|`
- patr: `=>` `;` `:` `=` `|` `in`
- attr: any token other than `(` `[` or `{`
- toks: any token

A repetition character tell how many times the sub-pattern needs to appear:

- `*`: repeat 0 or more times
- `+`: repeat 1 or more times
- `?`: optional, may occur 0 or 1 time

The sub-pattern inside of a repetition may not contain another repetition.

.. code-block::

    macro-pattern = { [ macro-separator ], macro-pattern-fragment | macro-var };
    macro-pattern-fragment = '$(', macro-pattern, ')', [? character sequence, except '?', '+' or '*' ?], [ '*' | '+' | '?' ];
    macro-separator = { ? character sequence, except '$' or ')' ? };
    macro-var = '$', identifier [ ':', macro-var-kind ];
    macro-var-kind = 'stmt'
                   | 'expr'
                   | 'iden'
                   | 'qual'
                   | 'attr'
                   | 'toks'
                   | 'patr';

Macro body
----------

The body of a macro contains a combination of normal noct code and some special macro expressions/statements.

A macro value can be used inside of the code by using its name with a '$' before it.

When the match can have a repeating pattern, a special statement can be used: the macro-repeat expression, which will expand the expression inside of it, for each repeating match of a sub-pattern. If multiple macro variables appear in a macro repeat expression, these variables need to appear the same amount of times in the macro.

.. code-block::

    macro-var-expression = '$', identifier;
    macro-repeat-expression = '${', { statement }, '}';

Instantiation
-------------

A macro instantiation is an expression or statement that tells the compiler what macro to invoke.

The macro being used, will be processed from top to bottom, meaning that the first pattern that it can match to, is the macro case it will try to instantiate, if this results in any error, the compiler will not try to match any additional macro cases.

When parsing a macro instantiation, the compiler needs to be able to figure out what the macro is, for this, it will base it's type on the surrounding tokens, if any ambiguity exists, the compiler will expect that the instantiation is a statement.

.. code-block::

    macro-instantiation = identifier, macro-argument-list;
    macro-argument-list = '!(', macro-argument, { operator, macro-argument }, ')'
                        | '!{', macro-argument, { operator, macro-argument }, '}'
                        | '![', macro-argument, { operator, macro-argument }, ']';

Macro hygiene
-------------

Macros are hygienic, meaning that any declaration inside of a macro is local to  the macro and will not be expanded into the place it is instantiated. It is still possible to have a macro define a variable for you, but this requires the identifier of that variable to be explicitly passed to the macro, or by having a declaration with the 'use_outside_macro' compiler attribute added to it.

Further reading
===============

For further reading about noct, or for any of the companion specification, you can go to the relevant pages.


// TODO
