# Types

As usual, types are divided into basic types and user defined types (enum, union, struct, error, aliases). All types are defined on a global level. Using the `public` prefix is necessary for any type that is to be exposed outside of the current module.

##### Naming

All user defined types in C3 starts with upper case. So `MyStruct` or `Mystruct` would be fine, `mystruct_t` or `mystruct` would not. Since this runs into probles with C compatibility, it is possible to use attributes to change the c name of a type, as well as control whether a C typedef should be emitted for the type.

```
struct Stat @(cname="stat", no_typedef)
{
    // ...
} 

func c_int stat(const c_char* pathname, Stat* buf);
```

##### Differences from C

Unlike C, C3 does not use type qualifiers. `const` exists, but is a storage class modifier, not a type qualifier. Instead of `volatile`, volatile blocks are used. In order to signal restrictions on variable usage, like const-ness [preconditions](../preconditions/) are used.

## Basic types

Basic types are divided into floating point types, and integer types. Integer types being either signed or unsigned.

##### Integer types

| Name         | alias | bit size | signed |
| ------------ | -----:| --------:|:------:|
| bool         | u1    | 1        | no     |
| char         | i8    | 8        | yes    |
| byte         | u8    | 8        | no     |
| short        | i16   | 16       | yes    |
| ushort       | u16   | 16       | no     |
| int          | i32   | 32       | yes    |
| uint         | u32   | 32       | no     |
| long         | i64   | 64       | yes    |
| ulong        | u64   | 64       | no     |
| isize*       | -     | varies   | yes    |
| usize*       | -     | varies   | no     |

*`isize` and `usize` are pointer sized.

##### Integer arithmetics

All signed integer arithmetics uses 2's complement.

##### Integer constants

Integer constants are 1293832 or -918212. Unlike C the "type" of an integer constant is a special compile time int. All constant operations, for example `9283 << 2` will be resolved at compile time. An compile time error will result if the constant is too large to fit whatever variable it is assigned or compared to.

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


##### Floating point types

| Name         | alias | bit size |
| ------------ | -----:| --------:|
| float        | f32   | 32       |
| double       | f64   | 64       |
| quad*        | f128  | 128      |

*support will depend on platform

##### Floating point constants

Floating point constants will *at least* use 64 bit precision. Just like for integer constants, it is allowed to use underscore, but it may not occur immediagely before or after a dot or an exponential.

Floating point values may be written in decimal or hexadecimal. For decimal, the exponential symbol is e (or E, both are acceptable), for hexadecimal p (or P) is used: `-2.22e-21` `-0x21.93p-10` 

### C compatibility

For C compatibility the following types are also defined:

| Name         | c type             |
| ------------ | ------------------:|
| c_char       | char               |
| c_short      | short int          |
| c_ushort     | unsigned short int |
| c_int        | int                |
| c_uint       | unsigned int       |
| c_long       | long int           |
| c_ulong      | unsigned long int  |
| c_longlong   | long long          |
| c_ulonglong  | unsigned long long |
| c_float      | float              |
| c_double     | double             |
| c_longdouble | long double        |

### Pointer types

Pointers mirror C: `Foo*` is a pointer to a `Foo`, while `Foo**` is a pointer to a pointer of Foo.

### Array types

Arrays are indicated by `[]` after the type, optionally with the size given, e.g. `int[4]`. Unlike C, the "empty" array (without size), is a variable array that may be queried about its size. There is also array slices with using the `[:]` suffix. See the chapter on [arrays](../arrays).

## Enum

Enum (enumerated) types use the following syntax:

```
enum State : int 
{
    PENDING = 0,
    RUNNING,
    TERMINATED
}
```
Enum constants are namespaces by default, just like C++'s class enums. So accessing the enums above would for example use `State.PENDING` rather than `PENDING`.

## Error

Error types are similar to enums, and are used for error returns.

```
error IOError
{
    FILE_CLOSED
    FILE_EOF_REACHED
}
```

Just like enums, the errors are namespaced. Read more about the error types on the page about [error handling](../errorhandling).

## Alias and function types

Alias types are used to give an alias to a different type, like:

```
typedef char* as CharPtr;
typedef int[10] as Numbers;
```

Function pointers _must_ be aliased in C3. The syntax is simpler than that of C:

`public typedef func void(int a, bool b) as Callback;`

This defines an alias to function pointer type of a function that returns nothing and requires two arguments: an int and a bool. Here is a sample usage:

```
Callback cb = my_callback;
cb(10, false);
```

## Struct types

Structs are always named:

```
struct Person  
{
    char age;
    char* name;
}
```

A struct's members may be accessed using dot notation, even for pointers to structs.

```
Person p;
p.age = 21;
p.name = "John Doe";

io.printf("%s is %d years old.", p.age, p.name);

Person* pPtr = &p;
pPtr.age = 20; // Ok!

io.printf("%s is %d years old.", pPtr.age, pPtr.name);
```

(One might wonder whether it's possible to take a `Person**` and use dot access. – It's not, only one level of deref is done)

## Struct subtyping

C3 allows creating struct subtypes:

```
struct ImportantPerson 
{
    inline Person person;
    char* title;
}

func printPerson(Person p)
{
    io.printf("%s is %d years old.", p.age, p.name);
}


ImportantPerson important_person;
important_person.age = 25;
important_person.name = "Jane Doe";
important_person.title = "Rockstar";
printPerson(important_person); // Only the first part of the struct is copied.
```

`inline` is not strictly needed. It allows a struct or union to be both addressed by name and directly as an anonymous struct/union. (See below)

## Union types

Union types are defined just like structs.

```
union Integral  
{
    byte as_byte;
    short as_short;
    int as_int;
    long as_long;
}
```

As usual unions are used to hold one of many possible values:

```
Integral i;
i.as_byte = 40; // Setting the active member to as_byte

i.as_int = 500; // Changing the active member to as_int

// Undefined behaviour: as_byte is not the active member, 
// so this will probably print garbage.
io.printf("%d", i.as_byte); 
```

Note that unions only take up as much space as their largest member, so `sizeof(Integral)` is equivalent to `sizeof(long)`.

## Anonymous sub-structs / unions

Just like in later versions of C, anonymous sub-structs / unions are allowed.

```
struct Person  
{
    char age;
    char* name;
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
```


## Tagged unions

In C, using structs with an enum value to indicate type is common practice. C3 also offers tagged unions, which is formalizing this within the language:

TBD: Exact syntax (see the [ideas](../ideas) page)

## Conversion to symbol to type and back with `type`

Macros and compile time constants may occasionally contain type *symbols*. To convert back and forth, the `type` operator is used.

```
macro @test($i)
{
    $if ($i < 2) return type(int);
    return type(double);
}

$foo = type(int*);
type($foo) i = nil;
type(@test(4)) = 100;
```


