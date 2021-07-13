# Types

As usual, types are divided into basic types and user defined types (enum, union, struct, error, aliases). All types are defined on a global level. Using the `public` prefix is necessary for any type that is to be exposed outside of the current module.

##### Naming

All user defined types in C3 starts with upper case. So `MyStruct` or `Mystruct` would be fine, `mystruct_t` or `mystruct` would not. Since this affects C compatibility, it is possible to use attributes to change the external name of a type:

```
struct Stat @extname("stat")
{
    // ...
} 

func CInt stat(const char* pathname, Stat* buf);
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

##### Base64 and hex data literals

Base64 encoded values work like TwoCC/FourCC/EightCC, in that is it laid out in byte order in memory. It uses the format `'<base64>'b64`. Hex encoded values work as base64 but with the format `'<hex>'x`. In data literals any whitespace is ignored, so `'00 00 11'x` encodes to the same value as `'000011'x`.

In our case we could encode `'Rk9PQkFSMTE='b64` as `'FOOBAR11'`.

Base64 and hex data literals also has a form to allow initializing char arrays, instead of enclosing the data in `''`, enclose the data in `""`

```
char[*] hello_world_base64 = "SGVsbG8gV29ybGQh"b64;
char[*] hello_world_hex = "4865 6c6c 6f20 776f 726c 6421"x;
```

##### Floating point types

| Name         | alias | bit size |
| ------------ | -----:| --------:|
| half*        | f16   | 16       |
| float        | f32   | 32       |
| double       | f64   | 64       |
| quad*        | f128  | 128      |

*support depends on platform

##### Floating point constants

Floating point constants will *at least* use 64 bit precision. Just like for integer constants, it is allowed to use underscore, but it may not occur immediagely before or after a dot or an exponential.

Floating point values may be written in decimal or hexadecimal. For decimal, the exponential symbol is e (or E, both are acceptable), for hexadecimal p (or P) is used: `-2.22e-21` `-0x21.93p-10` 

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

### Array types

Arrays are indicated by `[<size>]` after the type, e.g. `int[4]`. Subarrays use the `type[]`. For initialization the wildcard `type[*]` can be used to infer the size
from the initializer. See the chapter on [arrays](../arrays).

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

### Enum type inference

When an enum is used in where the type can be inferred, like in case-clauses or in variable assignment, it is allowed to drop the enum name:

```
State foo = PENDING; // State.PENDING is inferred.
switch (foo)
{
    case RUNNING: // State.RUNNING is inferred
        ...
    default:
        ...
}

func void test(State s) { ... }

...

test(RUNNING); // State.RUNNING is inferred
```

In the case that it collides with a global in the same scope, it needs the qualifier:

```
module test;

func void testState(State s) { ... }

State RUNNING = State.TERMINATED; // Don't do this!

... 

test(RUNNING); // Ambiguous
test(test::RUNNING); // Uses global.
test(State.RUNNING); // Uses enum constant.
```


## Error

Error types are similar to enums, and are used for error returns.

```
error IOError;
error ParseError
{
    int line;
    int col;
}
```

The data inside of an error cannot exceed the size of an `iptr` that is pointer sized.

An error is similar to a struct and is initialized the same way. One exception is that simple errors without a body does not need to be
created using a `()`:

```
return IOError!; 
return IOError({})!; // Same as above
return ParseError({ line, col })!;
```

In order to pass the error on the error side channel instead of as a value, the `!`.


## Failable

A failable is created by taking a type and appending `!`. A failable is a tagged union containing either the given type or an error.

```
int! i;
i = 5; // Assigning a real value to i.
i = IOError!; // Assigning an error to i.
```

Only variables and return variables may be of the *failable* type. Function and macro parameters may not be failable types.

```
func Foo*! getFoo() { ... } // Ok!
func void processFoo(Foo*! f) { ... } // Error!
int! x = 0; // Ok!
```

Read more about the errors on the page about [error handling](../errorhandling).


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

io::printf("%s is %d years old.", p.age, p.name);

Person* pPtr = &p;
pPtr.age = 20; // Ok!

io::printf("%s is %d years old.", pPtr.age, pPtr.name);
```

(One might wonder whether it's possible to take a `Person**` and use dot access. – It's not, only one level of dereference is done.)

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
    io::printf("%s is %d years old.", p.age, p.name);
}


ImportantPerson important_person;
important_person.age = 25;
important_person.name = "Jane Doe";
important_person.title = "Rockstar";
printPerson(important_person); // Only the first part of the struct is copied.
```

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

## Casting

Casting does not use the C-style `(<NewType>) <expression>` instead uses `(<NewType>)(<expression>)`
    
```
float f = 2.0;
int i = (int)(f);
```


## Anonymous structs

It's possible to use anonymous structs (structs without name) as arguments. These will only be checked for structural equivalence.

_NOTE: This syntax is not final._

```
func void set_coordinates(struct { int i; int j; } coord) { ... }

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

func void test()
{
    Vec2 v2 = { 1, 2 };
    Vector v = { 1, 4 };
    Vec3 v3 = { 1, 2, 3 };
    set_coordinates(v2);  // valid
    set_coordinates(v);   // valid
    set_coordinates(v3);  // ERROR, no structural equivalence.
    struct { int i; int j; } xy = v2;
    v = (struct { int, int })(v2);
}
```