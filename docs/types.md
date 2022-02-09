# Types

As usual, types are divided into basic types and user defined types (enum, union, struct, optenums, aliases). All types are defined on a global level. Using the `public` prefix is necessary for any type that is to be exposed outside of the current module.

##### Naming

All user defined types in C3 starts with upper case. So `MyStruct` or `Mystruct` would be fine, `mystruct_t` or `mystruct` would not. Since this affects C compatibility, it is possible to use attributes to change the external name of a type:

```
struct Stat @extname("stat")
{
    // ...
} 

fn CInt stat(const char* pathname, Stat* buf);
```

##### Differences from C

Unlike C, C3 does not use type qualifiers. `const` exists, but is a storage class modifier, not a type qualifier. Instead of `volatile`, volatile loads and stores are used. In order to signal restrictions on variable usage, like const-ness [preconditions](../preconditions/) are used.

## Basic types

Basic types are divided into floating point types, and integer types. Integer types being either signed or unsigned.

##### Integer types

| Name         | bit size | signed |
| :----------- | --------:|:------:|
| bool*        | 1        | no     |
| ichar        | 8        | yes    |
| char         | 8        | no     |
| short        | 16       | yes    |
| ushort       | 16       | no     |
| int          | 32       | yes    |
| uint         | 32       | no     |
| long         | 64       | yes    |
| ulong        | 64       | no     |
| iptr**       | varies   | yes    |
| uptr**       | varies   | no     |
| iptrdiff**   | varies   | yes    |
| uptrdiff**   | varies   | no     |
| isize**      | varies   | yes    |
| usize**      | varies   | no     |

\* `bool` will be stored as a byte.  
\*\* size, pointer and pointer sized types depend on platform.

##### Integer arithmetics

All signed integer arithmetics uses 2's complement.

##### Integer constants

Integer constants are 1293832 or -918212. Without a suffix, suffix type is assumed to the signed integer of *arithmetic promotion width*. Adding the `u` suffix gives a unsigned integer of the same width. Use `ixx` and `uxx` – where `xx` is the bitwidth for typed integers, e.g. `1234u16`

Integers may be written in decimal, but also

- in binary with the prefix 0b e.g. `0b0101000111011`, `0b011`
- in octal with the prefix 0o e.g. `0o0770`, `0o12345670`
- in hexadecimal with the prefix 0x e.g. `0xdeadbeef` `0x7f7f7f`

Furthermore, underscore `_` may be used to add space between digits to improve readability e.g. `0xFFFF_1234_4511_0000`, `123_000_101_100`


##### TwoCC, FourCC and EightCC

[FourCC](https://en.wikipedia.org/wiki/FourCC) codes are often used to identify binary format types. C3 adds direct support for 4 character codes, but also 2 and 8 characters:

- 2 character strings, e.g. `'C3'`, would convert to an ushort or short.
- 4 character strings, e.g. `'TEST'`, converts to an uint or int. 
- 8 character strings, e.g. `'FOOBAR11'` converts to an ulong or long.
 
Conversion is always done so that the character string has the correct ordering in memory. This means that the same characters may have different integer values on different architectures due to endianess.

##### Base64 and hex data literals

Base64 encoded values work like TwoCC/FourCC/EightCC, in that is it laid out in byte order in memory. It uses the format `b64'<base64>'`. Hex encoded values work as base64 but with the format `x'<hex>'`. In data literals any whitespace is ignored, so `'00 00 11'x` encodes to the same value as `x'000011'`.

In our case we could encode `b64'Rk9PQkFSMTE='` as `'FOOBAR11'`.

Base64 and hex data literals initializes to arrays of the char type:

```
char[*] hello_world_base64 = b64"SGVsbG8gV29ybGQh";
char[*] hello_world_hex = x"4865 6c6c 6f20 776f 726c 6421";
```

##### String literals, multi-line and raw strings

Regular string literals is text enclosed in `" ... "` just like in C. C3 also offers two other types of literals: *multi-line strings* and *raw strings*.

Multi-line strings start and end with `"""`. Newline after the initial `"""` are removed, and so is a trailing new line on the last line before `"""`. Initial whitespace will be trimmed to the
leftmost character in a line:

```
char* foo = """
    <html>
      <body>
        <p>Hello World</p>
      </body>
    </html>
    """;
// Same as:
char* foo = "<html>\n"
            "  <body>\n"
            "    <p>Hello World</p>\n"
            "  </body>\n"
            "</html>";    
```

You can use `\|` (escapes to a zero length string) and `\s` (escapes to a space) to control the automatic trim:
```
char* foo = """
  \|  <body>
  \|    <p>Hello World</p>    \s
  \|  </body>
    """;
// Same as:
char* foo = " <body>\n"
            "   <p>Hello World</p>     \n"
            " </body>";
```

Raw strings uses text between \` \`. Inside of a raw string, no escapes are available. To write a \` double the character:

```
char* foo = `C:\foo\bar.dll`;
char* bar = `"Say ``hello``"`;
// Same as
char* foo = "C:\\foo\\bar.dll";
char* bar = "\"Say `hello`\"";
```
     
##### Floating point types

| Name         | bit size |
| ------------ | --------:|
| float16*     | 16       |
| float        | 32       |
| double       | 64       |
| float128*    | 128      |

*support depends on platform

##### Floating point constants

Floating point constants will *at least* use 64 bit precision. Just like for integer constants, it is allowed to use underscore, but it may not occur immediately before or after a dot or an exponential.

Floating point values may be written in decimal or hexadecimal. For decimal, the exponential symbol is e (or E, both are acceptable), for hexadecimal p (or P) is used: `-2.22e-21` `-0x21.93p-10` 

It is possible to type a floating point by adding a suffix:

| Suffix       | type     |
| ------------ | --------:|
| f16          | float16  |
| f32 *or f*   | float    |
| f64          | double   |
| f128         | float128 |


### C compatibility

For C compatibility the following types are also defined when including std::cinterop

| Name         | c type             |
| ------------ | ------------------:|
| CChar        | char               |
| CShort       | short int          |
| CUShort      | unsigned short int |
| CInt         | int                |
| CUInt        | unsigned int       |
| CLong        | long int           |
| CULong       | unsigned long int  |
| CLongLong    | long long          |
| CULongLong   | unsigned long long |
| CFloat       | float              |
| CDouble      | double             |
| CLongDouble  | long double        |

    
Note that signed C char and unsigned char will correspond to `ichar` and `char`. `CChar` is only available to match the default signedness of `char` on the platform.

### Pointer types

Pointers mirror C: `Foo*` is a pointer to a `Foo`, while `Foo**` is a pointer to a pointer of Foo.

### The `typeid` type

The `typeid` can hold a runtime identifier for a type. Using `<typename>.typeid` a type may be converted to its unique runtime id, 
e.g. `typeid a = Foo.typeid;`. This value is pointer-sized.

### The `variant` type

C3 contains a built-in variant type, which is essentially struct containing a `typeid` plus a `void*` pointer to a value.
It is possible to cast the variant type to any pointer type, which will return `null` if the types match,
or the pointer value otherwise.

    int x;
    variant y = &x;
    double *z = (double*)y; // Returns null
    int* w = (int*)x; // Returns the pointer to x

Switching over the variant type is another method to unwrap the pointer inside:

    fn void test(variant z)
    {
        // Unwrapping switch
        switch (z)
        {
            case int: 
                // z is unwrapped to int* here
            case double:
                // z is unwrapped to double* here
        }
        // Assignment switch
        switch (y = z)
        {
            case int:
                // y is int* here
        }
        // Direct unwrapping to a value is also possible:
        switch (w = *z)
        {
            case int:
                // w is int here
        }
    }

`variant.type` returns the underlying pointee typeid of the contained value. `variant.ptr` returns 
the raw `void*` pointer.

### Array types

Arrays are indicated by `[<size>]` after the type, e.g. `int[4]`. Subarrays use the `type[]`. For initialization the wildcard `type[*]` can be used to infer the size
from the initializer. See the chapter on [arrays](../arrays).

## Enum

Enum (enumerated) types use the following syntax:

    enum State : int 
    {
      PENDING = 0,
      RUNNING,
      TERMINATED
    }

Enum constants are namespaces by default, just like C++'s class enums. So accessing the enums above would for example use `State.PENDING` rather than `PENDING`.

### Enum type inference

When an enum is used in where the type can be inferred, like in case-clauses or in variable assignment, it is allowed to drop the enum name:

    State foo = PENDING; // State.PENDING is inferred.
    switch (foo)
    {
      case RUNNING: // State.RUNNING is inferred
        ...
      default:
        ...
    }

    fn void test(State s) { ... }

    ...

    test(RUNNING); // State.RUNNING is inferred


In the case that it collides with a global in the same scope, it needs the qualifier:

    module test;

    fn void testState(State s) { ... }

    State RUNNING = State.TERMINATED; // Don't do this!

    ... 

    test(RUNNING); // Ambiguous
    test(test::RUNNING); // Uses global.
    test(State.RUNNING); // Uses enum constant.



## Optenums

`optenum` defines a set of optional result values, that are similar to enums, but are used for 
optional returns.

    optenum IOResult
    {
      IO_ERROR,
      PARSE_ERROR
    }   

    optenum MapResult
    {
      NOT_FOUND
    }
  
Like the typeid, the constants are pointer sized and each value is globally unique, even when 
compared to other optenums. For example the underlying value of `MapResult.NOT_FOUND` is guaranteed
to be different from `IOResult.IO_ERROR`. This is true even if they are separately compiled.

An optenum may be stored as a normal value, but is also unique in that it may be passed
as the optional result value using the `!` suffix operator.


## Optional Result Types

An *optional result type* is created by taking a type and appending `!`. 
An optional result type is a tagged union containing either the *expected result* or *an optional result value* 
(which is an optenum).

    int! i;
    i = 5; // Assigning a real value to i.
    i = IOResult.IO_ERROR!; // Assigning an optional result to i.


Only variables and return variables may be optionals. Function and macro parameters may not be optionals.

    fn Foo*! getFoo() { ... } // Ok!
    fn void processFoo(Foo*! f) { ... } // Error
    int! x = 0; // Ok!


Read more about the optional types on the page about [error handling](../errorhandling).


## Struct types

Structs are always named:

    struct Person  
    {
        char age;
        char[] name;
    }

A struct's members may be accessed using dot notation, even for pointers to structs.

    Person p;
    p.age = 21;
    p.name = "John Doe";

    io::printf("%s is %d years old.", p.age, p.name);

    Person* pPtr = &p;
    pPtr.age = 20; // Ok!

    io::printf("%s is %d years old.", pPtr.age, pPtr.name);

(One might wonder whether it's possible to take a `Person**` and use dot access. – It's not allowed, only one level of dereference is done.)

## Struct subtyping

C3 allows creating struct subtypes using `inline`:

    struct ImportantPerson 
    {
        inline Person person;
        char[] title;
    }

    fn void printPerson(Person p)
    {
        io::printf("%s is %d years old.", p.age, p.name);
    }


    ImportantPerson important_person;
    important_person.age = 25;
    important_person.name = "Jane Doe";
    important_person.title = "Rockstar";
    printPerson(important_person); // Only the first part of the struct is copied.


## Union types

Union types are defined just like structs and are fully compatible with C.

    union Integral  
    {
        byte as_byte;
        short as_short;
        int as_int;
        long as_long;
    }

As usual unions are used to hold one of many possible values:

    Integral i;
    i.as_byte = 40; // Setting the active member to as_byte

    i.as_int = 500; // Changing the active member to as_int

    // Undefined behaviour: as_byte is not the active member, 
    // so this will probably print garbage.
    io::printf("%d\n", i.as_byte); 


Note that unions only take up as much space as their largest member, so `$sizeof(Integral)` is equivalent to `$sizeof(long)`.


## Anonymous and nested sub-structs / unions

Just like in C99 and later, anonymous sub-structs / unions are allowed. Note that
the placement of struct / union names is different to match the difference in declaration.

    struct Person  
    {
        char age;
        char[] name;
        union 
        {
            int employee_nr;
            uint other_nr;
        }
        union subname 
        {
            bool b;
            Callback cb;
        }
    }


## Anonymous structs

It's possible to use anonymous structs (structs without name) as arguments. These will only be checked for structural equivalence.

_NOTE: This syntax will be changed._

```
fn void set_coordinates(struct { int i; int j; } coord) { ... }

struct Vec2
{
    int x;
    int y;
}

struct Vector
{
    int px;
    int py;
}

struct Vec3
{
    int x;
    int y;
    int z;
}

fn void test()
{
    Vec2 v2 = { 1, 2 };
    Vector v = { 1, 4 };
    Vec3 v3 = { 1, 2, 3 };
    set_coordinates(v2);  // valid
    set_coordinates(v);   // valid
    set_coordinates(v3);  // ERROR, no structural equivalence.
    struct { int i; int j; } xy = v2;
    v = (struct { int, int })v2;
}
```