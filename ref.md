Introduction
============

This is the reference manual for the Eon programming language.

Eon is a general-purpose low level language designed mainly for systems programming. It is strongly typed, has explicit control over allocated memory support, and forces clean separation of data and functionality. The language attempts to move as much functionallity from keywords into the syntax with as few simple rules as possible.

Source code representation
==========================

Source code is ASCII text encoded. This document will from here on out use the unqualified term *character* to refer to a ASCII code point in the source text.

Each code point is distinct; for instance, uppercase and lowercase letters are different characters.

Characters
----------

The following terms are used to denote specific ASCII character categories:

```
newline      = /* the ASCII code point 0x0A */ .
ascii_char   = /* an arbitrary ASCII code point except newline */ .
ascii_letter = /* a ASCII code point categorized as "Letter" */ .
ascii_digit  = /* a ASCII code point categorized as "Number, decimal digit" */ .
```

Letters and digits
------------------

The underscore character _ (0x5F) is considered a lowercase letter.

```
letter        = ascii_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```

Lexical elements
================

Comments
--------

Comments serve as program documentation. There are two forms:

1.	Line comments start with the character sequence // and stop at the end of the line.
2.	General comments start with the character sequence /* and stop with the first subsequent character sequence */.

A comment cannot start inside a string literal or inside a comment. A general comment containing no newlines acts like a space. Any other comment acts like a newline.

Tokens
------

There are four classes of tokens within Eon: identifiers, keywords, operators, and literals. White space, formed from spaces (0x20), horizontal tabs (0x09), carriage returns (0x0D), and newlines (0x0A), is ignored except as it separates tokens that would otherwise combine into a single token. While breaking the input into tokens, the next token is the longest sequence of characters that form a valid token.

Semicolons
----------

Eon uses semicolons ";" as line terminators. Only exception to this rule are expressions ending with a "}".

Identifiers
-----------

Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. The first character in an identifier must be a letter.

```
identifier = letter { letter | ascii_digit } .
```

Some identifiers are predeclared.

Keywords
--------

The following keywords are reserved and may not be used as identifiers.

```
const       if          continue        import
var         else        as              struct
function    switch      is              enum
pub         while       defer
return      break       discard
```

Operators
---------

The following character sequences represent operators (including assignment operators).

```
+    &     +=    &=     &&    ==    !=    (    )    ??     ||>>    ?|>>     !||>
-    |     -=    |=     ||    <     <=    [    ]    ??=    $|>     ?||>>    &&=
*    ^     *=    ^=     ++    >     >=    {    }    |>     $||>    ?$|>     ||=
/    <<    /=    <<=    --    =           ,    ;    ||>    ?|>     ?$||>    <:
%    >>    %=    >>=          !           .    :    |>>    ?||>    !|>      ::
```

Integer literals
----------------

An integer literal is a sequence of digits representing an integer constant. An optional prefix sets a non-decimal base: 0b or 0B for binary, 0o, or 0O for octal, and 0x or 0X for hexadecimal. A single 0 is considered a decimal zero. In hexadecimal literals, letters a through f and A through F represent values 10 through 15.

For readability, an underscore character _ may appear after a base prefix or between successive digits; such underscores do not change the literal's value.

```
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
```

Floating point literals
-----------------------

A floating-point literal is a decimal representation of a floating-point constant.

A decimal floating-point literal consists of an integer part (decimal digits), a decimal point, a fractional part (decimal digits), and an exponent part (e or E followed by an optional sign and decimal digits). One of the integer part or the fractional part may be elided; one of the decimal point or the exponent part may be elided. An exponent value exp scales the mantissa (integer and fractional part) by 10^exp.

For readability, an underscore character _ may appear after a base prefix or between successive digits; such underscores do not change the literal value.

```
float_lit = decimal_digits "." [ decimal_digits ] [ exponent ] |
                    decimal_digits exponent |
                    "." decimal_digits [ exponent ] .
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .
```

Char literals
-------------

A char literal represents a char constant, an integer value identifying a ASCII code point. A char literal is expressed as one or more characters enclosed in single quotes, as in 'x' or '\n'. Within the quotes, any character may appear except newline and unescaped single quote. A single quoted character represents the ascii value of the character itself, while multi-character sequences beginning with a backslash encode values in various formats.

The simplest form represents the single character within the quotes. An unrecognized character following a backslash in a char literal is illegal.

```
char_lit         = "'" ( ascii_value | byte_value ) "'" .
ascii_value      = ascii_char | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .

escaped_char     = `\` ( "n" | "r" | "t" | `\` | "'" | `"` ) .
```

String literals
---------------

A string literal represents a string constant obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.

Raw string literals are character sequences between back quotes, as in `foo`. Within the quotes, any character may appear except back quote. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes have no special meaning and the string may contain newlines. Carriage return characters ('\r') inside raw string literals are discarded from the raw string value.

Interpreted string literals are character sequences between double quotes, as in "bar". Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in char literals (except that \' is illegal and \" is legal), with the same restrictions.

```
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { ascii_char | newline } "`" .
interpreted_string_lit = `"` { ascii_value | byte_value } `"` .
```

Variables
=========

A variable is a storage location for holding a *value*. The set of permissible values is determined by the variable's *type*.

A variable declaration or, for function parameters, the signature of a function declaration reserves storage for a named variable on the application stack.

Structured variables of array, slice, and struct types have elements and fields that may be addressed individually by using the dot notation. Each such element acts like a variable.

Pointer variables contain an address of some data which can be accessed by *dereferencing* said variable using the dot notation.

Each variable's address can be read explicitly using the dot notation.

Variables are declared using one of two keyword which have separate semantics:

-	`var` declares a *mutable* variable whose value can be modified during runtime
-	`const` declares an *immutable* variable whose value must be initialized and cannot be modified during runtime

Both mutable and immutable values must not have the type `type` unless their initialization and every assignment can be fully resolved during compilation of the program.

```
var x: u32 = 123;
var y: i16;
const char: u8 = 'E';
const buffer: []u8 = get_slice_in_runtime();

// valid, its value is always known during compilation
var my_type: type = u32;
my_type = isize;

// invalid, z is not a constant and its value cannot be resolved during compilation
var z: i32 = read_number_from_stdin();
const inv_cond_type: type = if (z == 321) u32 else bool;

// valid, w1 is a constant and w2 has a compile time known value at the point of usage
const w1: i32 = 123 * 321; 
var w2: i32 = 555;
const cond_type: type = if (w1 > w2) u32 else bool; 
var w2 = read_number_from_stdin();

var var_with_cond_type: cond_type;
```

A variable's value is retrieved by referring to the variable in an expression; it is the most recent value assigned to the variable. If a variable has not yet been assigned a value, its value is undefined.

Constants
---------

Constants are a special kind of *variables* which satisfy 2 conditions:

-	Are immutable (declared using the `const` keyword
-	Have their value fully known during compilation

Constants have no type restrictions placed upon them.

```
const char: u8 = 'E';                               // constant
const buffer: []u8 = get_slice_in_runtime();        // not a constant, depends on runtime code
const my_type: type = u32;                          // constant, acts like an alias to u32
```

A constant's value might be inlined or used as metadata by the compiler.

Eon implements 3 predeclared *literal* constants, `true` and `false` which represent boolean truth values, and `null` which represents an absence of a value.

Dot notation
------------

The dot notation is used to access individual fields, elements, addresses, or the underlying memory of a variable. Basically, it is used to access data within data.

```
array.5                     // access the fifth element of 'array'
array.[5]                   // access the fifth element of 'array'
array.[i]                   // access the 'i'th element of 'array' where 'i' is a non-negative integer
my_struct.some_field        // access a field named 'some_field'
ptr.*                       // dereference the pointer 'ptr'
my_struct.&                 // grab the address of 'my_struct'
```

Types
-----

A *type* determines a set of values together with operations and methods specific to those values. A type may be denoted by a *type name*, *function name* which returns a type with or without specified arguments, or an *unnamed type*. A type may also be specified using a *type literal*, which composes a type from existing types.

```
Type     = TypeName | TypeLit .
TypeName = identifier | FunctionName | FunctionInvocation | void .
TypeLit  = ArrayType | StructType | PointerType | FunctionType | EnumType | SliceType | NullableType .
```

The language predeclares certain type names. Composite types—array, struct, pointer, function, interface, slice, map, and channel types—may be constructed using type literals.

Boolean types
-------------

A *boolean* type represents the set of Boolean truth values denoted by the predeclared constants `true` and `false`. The predeclared boolean type is `bool`.

Numeric types
-------------

An *integer* or *floating-point* type represents the set of integer or floating-point values respectively. They are collectively called numeric types. The predeclared architecture-independent numeric types are:

```
u8          the set of all unsigned  8-bit integers (0 to 255)
u16         the set of all unsigned 16-bit integers (0 to 65535)
u32         the set of all unsigned 32-bit integers (0 to 4294967295)
u64         the set of all unsigned 64-bit integers (0 to 18446744073709551615)

i8          the set of all signed  8-bit integers (-128 to 127)
i16         the set of all signed 16-bit integers (-32768 to 32767)
i32         the set of all signed 32-bit integers (-2147483648 to 2147483647)
i64         the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

f32         the set of all IEEE 754 32-bit floating-point numbers
f64         the set of all IEEE 754 64-bit floating-point numbers
```

The value of an n-bit integer is n bits wide and represented using two's complement arithmetic.

There is also a set of predeclared integer types with implementation-specific sizes:

```
usize       an unsigned integer large enough to store the uninterpreted bits of a pointer value
isize       a signed integer large enough to store the uninterpreted bits of a pointer value
```

Array types
-----------

An array is a numbered sequence of elements of a single type, called the element type. The number of elements is called the length of the array and is never negative.

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

The length is part of the array's type; it must evaluate to a non-negative constant representable by a value of type `usize` known during compilation. The length of an array can be discovered by accessing its *len* field. The elements can be addressed by integer indices 0 through `a.len - 1`. Array types are always one-dimensional but may be composed to form *multi-dimensional* types.

```
[32]u8              // string of 32 ascii characters
[2 * N] user_t      // N must be known during compilation
[1000]*f64          // array of 1000 pointers
[3][5]i32           // 2D matrix
[2][2][2]f64        // same as [2]([2]([2]float64))
```

Slice types
-----------

A slice is a descriptor for a contiguous segment of an underlying array, containing the address of its first element and the number of elements represented by a value of type `usize`. The number of elements is called the length of the slice and is never negative.

The length of a slice can be discovered by accessing its *len* field. The elements can be addressed by integer indices 0 through `a.len - 1`. Slice types are always one-dimensional but may be composed to form *multi-dimensional* types.

Unlike array types, a slice type does not need to have its length known during compilation.

```
SliceType = "[" "]" ElementType .
```

The array underlying a slice may extend past the end of the slice, just as a slice's first element doesn't have to be the first element of the underlying array.

A slice can be created by *slicing* an array or an existing slice of the same type. This does *not* clone the underlying array.

```
array.[:]               // creates a slice that aligns perfectly with the array
array.[5:8]             // creates a slice whose first element is the 6th element of the array with a length of 3
array.[:7]              // creates a slice starting at the array's start with a length of 7
array.[3:]              // creates a slice whose first element is the 4th element of the array with a length of array.len - 3
slice.[1:3]             // creates a slice whose first element is offset from the slice it was created from by 1, with a length of 2
array.[1:3].[1:2]       // you can even chain slicing operations
```

Struct types
------------

A struct is a sequence of named elements, called fields, each of which has a name and a type. Field names must be specified explicitly and must be unique.

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = Identifier ":" Type .
```

```
// An empty struct.
struct {}

// A struct with 3 fields.
struct {
	x: u32;
	y: u32;
	inner_struct: inner_struct_t;
}
```

A struct type T may not contain a field of type T, or of a type containing T as a component, directly or indirectly, if those containing types are only array or struct types.

```
const linked_list_t = struct {
    val: i64;
    next: *linked_list_t;
}
```

A struct type T0 may become a *subtype* (superset) of an arbitrary number of types Ti (where i > 0) in which case it must contain all of their combined fields and can be used in their place. Once a type T0 becomes a subtype of another type T1, that type (T1) becomes a *supertype* (subset) of the subtype type (T0).

`<:` is the subtype operator from *type theory* and means "is a subtype of".

```
const T1 = struct {
    x: u32;
    y: f32;
}

const T2 = struct {
    y: f32;
    z: usize;
}

const T0 = struct <: T1, T2 {
    x: u32;
    y: f32;
    z: usize;
    w: isize;
}
```

If 2 separate types Tp and Tq have a *conflicting field*, defined as a field with the same identifier but different type, they may not become a supertype of the same subtype.

```
const T1 = struct {
    x: u32;
    y: f32;
}

const T2 = struct {
    y: usize;
    z: usize;
}

// invalid, T1 has a field named 'y' of type u32 and T2 has a field with the same name of type usize
const T0 = struct <: T1, T2 { }
```

Enum types
----------

An enum type denotes the set of possible named values a variable can hold represented as unsigned integers during runtime.

The value of each enum field is part of that enum's type; it must evaluate to a non-negative constant representable by a value of an unsigned integer type known during compilation. If a value is not explicitly specified it defaults to the lowest non taken value greater than the value of the field before it.

Individual enum field can be accessed by their names with the *dot notation*.

```
EnumType  = "enum" "{" { EnumFieldDecl ";" } "}" .
EnumFieldDecl = Identifier [ "=" Expression ] .
```

```
const sat: u8 = 11;
const week_days_t: type = enum {
    monday;         // = 0
    tuesday = 5;
    wednesday;      // = 6
    thursday;       // = 7
    friday;         // = 8
    saturday = sat;
    sunday;         // = 12
};

const monday: week_days_t = week_days_t.monday; // fully specified enum field
const monday: week_days_t = .monday; // implicitly specified enum field
```

Union types
-----------

A union type represents a value that can be one of several *case types*. Their size is dependant on the size of the biggest case type.

```
UnionType = CaseType { "|" CaseType }
CaseType  = Type
```

Active case type of an individual value of a union type must be explicitly checked before usage using the *pattern matching* keyword `is`, after which it can either be casted into the correct type, or depending on the context in which pattern matching was performed, it may be *captured* into the correct type.

```
const byte: u8 = 'x';
const float: f64 = 123.456;
const slice: []f32 = some_arr[:];

const my_union_t: type = u8 | f64 | []f32; // has size = 64 bits + prefix, because the largest case type is 64 bits

var union: my_union_t = byte;
union = float;

discard (union is f64); // true
discard (union is u8); // false
discard (union is u64); // compilation error, my_union_t can never be of type u64

union = slice;
discard (union is f32); // false
discard (union is []f32); // true

const slice_from_union: []f32 = union as []f32;
```

Pointer types
-------------

A pointer type denotes the set of all pointers to variables of a given type, called the base type of the pointer. The value of a pointer to a non-existant variable is null. Essentially, a pointer holds an address of a memory location holding its base type.

```
PointerType = "*" [ "const" ] BaseType .
BaseType    = Type .
```

Accessing an address of a variable, as well as the memory a pointer points to, is done using the *dot notation*.

```
var x: u32 = 123;
const ptr: *u32 = x.&; // ptr holds the address of 'x'
ptr.* = 5; // same as 'x = 5;'
```

A pointer type can only hold an address of a variable with *less or equal strictness* in terms of *mutability*. This means a mutable pointer can only point to a mutable variable, while an immutable pointer can point to either an immutable or mutable variable. If an immutable pointer points to a mutable variable, it can not be used to modify its value.

```
var x: u32 = 123;
const y: u32 = 321;

var ptr: *const u32 = x.& // ptr holds the address of 'x'
ptr.* = 5; // invalid, ptr points to an immutable value

ptr = y.&; // valid, only pointed value is immutable, the actual variable holding the address (pointer) is still mutable
ptr.* = 5; // invalid, even though ptr points to a mutable value it considers it immutable due to its own type
```

Nullable types
--------------

A nullable type `?T` represents all values of its underlying value type T and an additional `null` value. For example, you can assign any of the following three values to a `?bool` variable: `true`, `false`, or `null`. An underlying value type T must not be a nullable type itself.

Value `null` represents the absence of a value.

```
NullableType        = "?" UnderlyingValueType .
UnderlyingValueType = Type .
```

A value of a nullable type may only be used in a comparison against null in order to *unwrap it* into its underlying value type using the *capture* syntax.

If a fallback is provided for a nullable value immediately before use, it may be used as its underlying value without unwrapping.

```
var x: ?u32 = null;

if (x != null) |safe_x| {
    // safe_x is of type 'u32'
}

if (x) |safe_x| {
    // shorthand capture, means "if x exists"
}

x ??= 7; // if x is null, set it to 7
var y: u32 = x - 5; // valid, x has a provided fallback in a line above
```

Function pointer types
----------------------

A function pointer type is a special kind of pointer which holds the address of a compiled function in order to allow creating *virtual method tables* (or vtables for short).

The actual function pointer type only contains the address of the starting instruction compiled from a function and cannot be dereferenced directly. Dereferencing a function would mean **reading the value of a function**, which is meaningless because a function has no value, instead it defines how (a) value is processed.

More broadly, a function defines *behaviour* which is why only its address can be kept as data.

```
FunctionPointerType = "*" "fn" Signature .
Signature     = Parameters Result .
Result        = Type .
Parameters    = "(" [ ParameterList [ "," ] ] ")" .
ParameterList = ParameterDecl { "," ParameterDecl } .
ParameterDecl = Identifier ":" Type .
```

In order to invoke a function based on its address, a new *function identifier* must be declared and assigned that address.

```
const print_fn: *fn (string: []u8) void = print; // 'print' is a function available in this scope with the exact same signature

fn my_print(string: []u8) void = print_fn;

// these 2 calls do the exact same thing, and even compile to the same instruction
print("Hello Eon");
my_print("Hello Eon"); 
```

Type identity
-------------

Two types are *either* identical ("the same") or *different*.

-	Two array types are identical if they have identical element types and the same array length
-	Two slice types are identical if they have identical element types
-	Two struct types are identical if they have the same sequence of fields, and if corresponding pairs of fields have the same names, identical types, and identical alignments
-	Two pointer types are identical if they have identical base types
-	Two function types are identical if they have the same signature

Assignability
-------------

A value x of type V is *assignable* to a variable of type T ("x is assignable to T") if one of the following conditions applies:

-	V and T are identical
-	V is a subtype of T
-	T is a union and V is a case type of T

Blocks
======

A *block* is a possibly empty sequence of declarations and statements within matching brace brackets.

A block is a *named block* if it is directly preceded by a label.

```
Block         = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

In addition to explicit blocks in the source code, each file has an implicit *module block* allowing for comptime code and conditional compilation.

Blocks nest and influence *scoping*.

Declarations and scope
======================

A declaration binds an identifier to a constant, type, variable, function, or label. Every identifier in a program must be declared. No identifier may be declared twice in the same block.

```
Declaration  = ConstDecl | VarDecl | FunctionDecl .
```

Eon is lexically scoped using blocks:

-	The scope of a predeclared identifier encompasses all blocks
-	The scope of an identifier denoting a constant or function declared at top level (outside any function) is the module block
-	The scope of an imported identifier is the module block of the file containing the import declaration
-	The scope of an identifier denoting function parameter is the function body
-	The scope of a constant or variable identifier declared inside a block begins at the end of the declaration and ends at the end of the innermost containing block

An identifier declared in a block may not be redeclared in an inner block.

Labels
------

Labels are declared right before an opening brace bracket of a block and are used in the "break" and "continue" statements. It is illegal to define a label that is never used. In contrast to other identifiers, labels do not conflict with identifiers that are not labels. The scope of a label is the body of the function in which it is declared and excludes the body of any nested function.

Predeclared identifiers
-----------------------

The following identifiers are implicitly declared in all scopes.

```
Types:
    void
    type
    bool
    i8 i16 i32 i64
    u8 u16 u32 u64
    f32 f64

Constants:
    true false

Zero value:
    null
```

Exported identifiers
--------------------

An identifier may be exported to permit access to it from another module.

An identifier is exported if both:

-	the identifier's declaration is preceded by the keyword `pub`
-	the identifier is declared in the module block

All other identifiers are not exported.

Uniqueness of identifiers
-------------------------

Given a set of identifiers, an identifier is called *unique* if it is different from every other in the set. Two identifiers are different if they are spelled differently, or if they appear in different modules. Otherwise, they are the same.

Variable declarations
---------------------

A variable declaration creates one variable, binds the corresponding identifier to it, and giving it a type and an optional initial value.

```
VarDecl = "var" Identifier [ ( ":" Type [ "=" Expression ] ) | ( "=" Expression ) ] .
ConstDecl = "const" Identifier [ ":" Type ] "=" Expression .
```

```
var a: u32 = 123;
var b = 456; // infers type as i32
var c: bool; // value is undefined, must be assigned before use
var d = true; // infers type as bool

const x: f32 = 123.456;
const y = 123.456; // infers type as f64
```

If a type is not explicitly specified, an initial value must be given in which case the variable assumes the *most specific* type that can hold the given value. This is called *type inference*.

The predeclared identifier `null` cannot be used to initialize (as the inital value of) a variable without an explicit type.

It is illegal to define a variable that is never used or exported.

Function declarations
---------------------

A function declaration binds an identifier, the *function name*, to a function.

It is illegal to define a function that is never used or exported.

```
FunctionDecl = "fn" FunctionName Signature FunctionBody .
FunctionName = identifier .
FunctionBody = Block .
```

```
fn add_two(number: i32) i32 {
    return number + 2;
}
```

Expressions
===========

An expression specifies the *computation* of a value by applying operators and functions to operands.

Operands
--------

Operands denote the elementary values in an expression. An operand may be a literal, a named block, an identifier denoting a constant, variable, or function, or an expression.

```
Operand     = Literal | identifier  [ TypeArgs ] | Expression .
Literal     = BasicLit | CompositeLit .
BasicLit    = int_lit | float_lit | char_lit | string_lit .
```

Qualified identifiers
---------------------

A *qualified identifier* is an identifier qualified with a module name prefix.

```
QualifiedIdent = ModuleName "::" identifier .
```

A qualified identifier accesses an identifier in a different module, which must be imported. The identifier must be exported and declared in the module block of that module.

Composite literals
------------------

Composite literals construct new values for structs and arrays.

```
CompositeLit       = [LiteralType] LiteralValue .
LiteralType        = StructType | ArrayType .
LiteralValue       = StructLiteralValue | ArrayLiteralValue
StructLiteralValue = "." "{" [ KeyedElementList [ "," ] ] "}"
ArrayLiteralValue  = "." "{" [ ElementList [ "," ] ] "}"
KeyedElementList   = KeyedElement { "," KeyedElement } .
ElementList        = Element { "," Element } .
KeyedElement       = Key ":" Element .
Key                = FieldName | Expression | LiteralValue .
FieldName          = identifier .
Element            = Expression | LiteralValue .
```

A literal type may be omitted from a composite literal if it is possible to deduce its type from the context in which it is being created.

For struct literals the following rules apply:

-	Every element must have a key
-	Each key must be a valid field selector for a field of the struct; the key selects that field
-	The element list must have an element for each struct field

Following ways of assigning a struct literal to a variable are valid:

```
const vector2_t: type = struct {
    x: f32;
    y: f32;
};

const v1: vector2_t = vector2_t.{
    x: 5,
    y: 7.5,
};

const v2 = vector2_t.{ // the variable type is inferred
    x: 5,
    y: 7.5,
};

const v3: vector2_t = .{ // the struct literal type is inferred
    x: 5,
    y: 7.5,
};
```

For array literals the following rules apply:

-	Each element has an associated integer index marking its position in the array
-	An element uses the previous element's index plus one, with the first element having the index 0
-	The element list does not need to have an element for each position in the array, omitted elements will have *undefined* memory

Following ways of assigning an array literal to a variable are valid:

```
const arr1: [16]u32 = [16]u32.{ 1, 2, 3, 4, 5 }; // remaining elements contain undefined values

const arr2 = [16]u32.{ 1, 2, 3, 4, 5 }; // the variable type is inferred

const arr3: [16]u32 = .{ 1, 2, 3, 4, 5 }; // the array literal type is inferred
```

Primary expressions
-------------------

Primary expressions are the operands for unary and binary expressions.

```
PrimaryExpr   = Operand |
                Cast |
                PrimaryExpr Selector |
                PrimaryExpr Index |
                PrimaryExpr Slice |
                PrimaryExpr Arguments .

Selector      = "." identifier .
Index         = "." int_lit |
                "." "[" Expression "]" .
Slice         = "." "[" [ Expression ] ":" [ Expression ] "]" .
Arguments     = "(" [ ExpressionList [ "," ] ] ")" .
ExpressionList = Expression { "," Expression } .
```

Selectors
---------

For a primary expression x, the selector expression `x.f` denotes the field f of the value x. The identifier f is called the (field) *selector*. The type of the selector expression is the type of f.

```
const vector2_t = struct {
    x: f32;
    y: f32;
};

const v = vector2_t.{
    x: 1.55,
    y: 87.53,
};

const x: f32 = v.x;
const y: f32 = v.y;
```

Index expressions
-----------------

A primary expression of the form `a.5` or `a.[x]` denotes the element of the array or slice a indexed by x. The value x is called the *index*. Literal indices may be specified without brackets. The type of an index expression is the type of the element.

```
const arr: [3]u16 = .{ 1, 2, 3 };
const a: u16 = arr.0; // = 1

const i: usize = arr.1; // = 2
const b: u16 = arr.[i]; // same as arr.2, = 3
```

Slice expressions
-----------------

Slice expressions construct a slice from an array or a slice operand by specifying a *low* and *high* bound. The type of a slice expression is a slice type with the element type equal to the element type of the used array or slice.

For an array or a slice the primary expression `a.[low:high]` constructs a new slice. The *indices* low and high select which elements of operand a appear in the result. The result has indices starting at 0 and length equal to high - low.

For convenience, any of the indices may be omitted. A missing low index defaults to zero; a missing high index defaults to the length of the sliced operand:

```
const a: [5]f32 = .{ 1, 2, 3, 4, 5 };
const s: []f32 = a.[1:3]; // = .{ 2, 3, 4 }

discard a.[2:];  // same as a[2 : a.len]
discard a.[:3];  // same as a[0 : 3]
discard a.[:];   // same as a[0 : a.len]
```

Addressing expressions
----------------------

For an operand x of type T, the address expression `x.&` generates a pointer of type *T to x. The operand must be addressable, that is, either a variable, pointer indirection, or slice indexing operation; or a field selector of an addressable struct operand; or an array indexing operation of an addressable array.

For an operand x of pointer type *T, the *pointer indirection* (dereferencing) `x.*` denotes the variable of type T pointed to by x.

```
const x: u32 = 123;
const ptr: *u32 = x.&;

discard ptr.*; // = 123
discard ptr.*.&.*.&.*.&; // addressing and dereferencing can be chained
```

Calls
-----

Given an expression f of function type F, `f(a1, a2, ... an)` *calls* f with arguments a1, a2, ... an. Arguments must be single-valued expressions assignable to the parameter types of F and are evaluated before the function is called. The type of the expression is the result type of F.

```
fn add_two(number: i32) i32 {
    return number + 2;
}

const a: u32 = 17;
const b: u32 = add_two(a + 3); // equivalent to (17 + 3) + 2 = 22
```

Operators
---------

Operators combine operands into expressions.

```
Expression = UnaryExpr | BinaryExpr | PipeExpr .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .
BinaryExpr = Expression binary_op Expression .

binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" .

unary_op   = "-" | "!" | "~" .
```

For binary expressions, the operand types must be identical.

Operator precedence
-------------------

Unary operators have the highest precedence.

There are five precedence levels for binary operators. Multiplication operators bind strongest, followed by addition operators, comparison operators, && (logical AND), and finally || (logical OR):

```
Precedence Operator

5             *  /  %  <<  >>  &
4             +  -  | ^
3             ==  !=  <  <=  >  >=
2             &&
1             ||
```

Binary operators of the same precedence associate from left to right. For instance, `x / y * z` is the same as `(x / y) * z`.

Arithmetic operators
--------------------

Arithmetic operators apply to numeric values and yield a result of the same type as the operands. The four standard arithmetic operators (+, -, *, /) apply to integer and floating-point types. The bitwise logical and shift operators apply to integers only.

```
+    sum                    integers and floats
-    difference             integers and floats
*    product                integers and floats
/    quotient               integers and floats
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers

<<   left shift             integer << integer >= 0
>>   right shift            integer >> integer >= 0
```

Integer operators
-----------------

For two integer values x and y, the integer quotient `q = x / y` and remainder `r = x % y` satisfy the following relationships:

```
x = q*y + r  and  |r| < |y|
```

with x / y truncated towards zero.

```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```

The shift operators shift the left operand by the shift count specified by the right operand, which must be non-negative. The shift operators implement arithmetic shifts if the left operand is a signed integer and logical shifts if it is an unsigned integer. There is no upper limit on the shift count. Shifts behave as if the left operand is shifted n times by 1 for a shift count of n. As a result, `x << 1` is the same as `x * 2` and `x >> 1` is the same as `x / 2` but truncated towards negative infinity.

```
 x     x / 4     x % 4     x >> 2     x & 3
 11      2         3         2          3
-11     -2        -3        -3          1
```

For integer operands, the unary operators - and ~ are defined as follows:

```
-x    negation              is 0 - x
~x    bitwise complement    is m ^ x  with m = "all bits set to 1" for unsigned x
                                      and  m = -1 for signed x
```

Integer overflow
----------------

For unsigned integer values, the operations +, -, *, and \<< are computed modulo 2n, where n is the bit width of the unsigned integer's type. Loosely speaking, these unsigned integer operations discard high bits upon overflow, and programs may rely on "wrap around".

For signed integers, the operations +, -, *, /, and \<< may legally overflow and the resulting value exists and is deterministically defined by the signed integer representation, the operation, and its operands. A compiler may not optimize code under the assumption that overflow does not occur. For instance, it may not assume that x < x + 1 is always true.

Comparison operators
--------------------

Comparison operators compare two operands and yield a boolean value.

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

In any comparison both operand must be of identical types.

The equality operators `==` and `!=` apply to operands of *comparable types*. The ordering operators `<`, `<=`, `>`, and `>=` apply to operands of *ordered types*. These terms and the result of the comparisons are defined as follows:

-	Boolean types are comparable. Two boolean values are equal if they are either both true or both false.
-	Integer types are comparable and ordered. Two integer values are compared in the usual way.
-	Floating-point types are comparable and ordered. Two floating-point values are compared as defined by the IEEE 754 standard.
-	Pointer types are comparable. Two pointer values are equal if they point to the same memory.
-	Slice types are comparable if they have the same element type. Two slice values are equal if they both have the same pointer and length.
-	Nullable types are comparable and/or ordered if their underlying types are comparable and/or ordered. Two nullable values are equal if they are both null, otherwise the result of their comparasion is identical as if comparing their underlying values directly.

All other types are not comparable.

Logical operators
-----------------

Logical operators apply to boolean values and yield a result of the same type as the operands. The left operand is evaluated, and then the right if the condition requires it.

```
&&    conditional AND    p && q  is  "if p then q else false"
||    conditional OR     p || q  is  "if p then true else q"
!     NOT                !p      is  "not p"
```
