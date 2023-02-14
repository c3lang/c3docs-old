# Reflection

C3 allows both compile time and runtime reflection.

During compile time the type information may be directly used as compile time constants, the same data is then available dynamically at runtime.

*Note that not all reflection is implemented in the compiler at this point in time.*

## Compile time reflection

During compile time there are a number of compile time fields that may be accessed directly.

### Type properties

It is possible to access properties on the type itself:

- `associated`
- `elements`
- `inf`
- `inner`
- `kind`
- `len`
- `max`
- `membersof`
- `min`
- `nan`
- `names`
- `params`
- `returns`
- `sizeof`
- `typeid`
- `values`

#### associated

*Not yet implemented*

*Only available for enums.*
Returns an array containing the types of associated values if any.

    enum Foo : int(double d, String s)
    {
        BAR(1.0, "normal"),
        BAZ(2.0, "exceptional")
    }
    String s = $nameof(Foo.associated[0]); // "double"

#### elements

Returns the element count of an enum or fault.

    enum FooEnum
    {
        BAR,
        BAZ
    }
    int x = FooEnum.elements; // 2


#### inf

*Only available for floating point types*

Returns a representation of floating point "infinity".

#### inner

This returns a typeid to an "inner" type. What this means is different for each type:

- Array -> the array base type.
- Bitstruct -> underlying base type.
- Enum -> underlying enum base type.
- Pointer -> the type being pointed to.
- Vector -> the vector base type.
- Distinct -> the underlying type.

It is not defined for other types.

#### kind

Returns the underlying `TypeKind` as defined in std::core::types.

    TypeKind kind = int.kind; // TypeKind.SIGNED_INT

#### len

Returns the length of the array.

    usz len = int[4].len; // 4

#### max

Returns the maximum value of the type (only valid for integer and float types).

    ushort max_ushort = ushort.max; // 65535

#### membersof

*Only available for struct and union types.*

Returns an array containing the fields in a struct or enum.

    struct Baz
    {
        int x;
        Foo* z;
    }
    String x = $nameof($typefrom(Baz.membersof[1])); // "z"

#### min

Returns the minimum value of the type (only valid for integer and float types).

    ichar min_ichar = ichar.min; // -128


#### names

Returns a subarray containing the names of an enum or fault.

    enum FooEnum
    {
        BAR,
        BAZ
    }
    String[] x = FooEnum.names; // ["BAR", "BAZ"]

#### params

*Not yet implemented*

*Only available for function types.*
Returns a list of all parameters.

    typedef TestFunc = fn int(int, double);
    String s = $nameof(TestFunc.params[1]); // "double"

#### returns

*Only available for function types.*
Returns the type of the return type.

    typedef TestFunc = fn int(int, double);
    String s = $nameof($typeform(TestFunc.returns)); // "int"

#### sizeof

Returns the size in bytes for the given type, like C `sizeof`.

    usz x = Foo.sizeof;

#### typeid

Returns the typeid for the given type. Typedefs will return the typeid of the underlying type. The typeid size is the same as that of an `iptr`.

    typeid x = Foo.typeid;

#### values

Returns a subarray containing the values of an enum or fault.

    enum FooEnum
    {
        BAR,
        BAZ
    }
    String x = $nameof(FooEnum.values[1]); // "BAR"

### Compile time functions

There are several built-in functions to inspect the code during compile time.

- `$alignof`
- `$checks`
- `$defined`
- `$eval`
- `$evaltype`
- `$extnameof`
- `$nameof`
- `$offsetof`
- `$qnameof`
- `$sizeof`
- `$stringify`
- `$typeof`

### $alignof

Returns the alignment in bytes needed for the type or member.

    module test::bar;

    struct Foo
    {
      int x;
      char[] y;
    }
    int g = 123;
    
    $alignof(Foo.x); // => returns 4
    $alignof(Foo.y); // => returns 8 on 64 bit
    $alignof(Foo);   // => returns 8 on 64 bit
    $alignof(g);     // => returns 4

### $checks

Returns true if the expression can be parsed and analysed, false otherwise.

    int a;
    typeid b;
    bool x = $checks(a + a); // x = true
    bool y = $checks(b + b); // y = false

This function can be very useful when checking macro arguments.

### $defined

Returns true if the expression inside is defined.

    $defined(Foo.x); // => returns true
    $defined(Foo.z); // => returns false

### $eval

Converts a compile time string with the corresponding variable:

    int a = 123;         // => a is now 123
    $eval("a") = 222;    // => a is now 222
    $eval("mymodule::fooFunc")(a); // => same as mymodule::fooFunc(a) 

`$eval` is limited to a single, optionally path prefixed, identifier.
Consequently methods cannot be evaluated directly:

    struct Foo { ... }
    fn int Foo.test(Foo* f) { ... }
 
    fn void test()
    {
       void* test1 = &$eval("test"); // Works
       void* test2 = &Foo.$eval("test"); // Works
       // void* test3 = &$eval("Foo.test"); // Error
    }

### $evaltype

Similar to `$eval` but for types:

    $evaltype("float") f = 12.0f;

### $extnameof

Returns the external name of a type, variable or function. The external name is
the one used by the linker.

    fn void testfn(int x) { }
    String a = $extnameof(g); // => "test.bar.g";
    string b = $extnameof(testfn); // => "test.bar.testfn"

### $nameof

Returns the name of a function, type or variable as a string without module prefixes.

    typedef Bar = Foo;
    String a = $nameof(int[4]); // => "int[4]"
    String b = $nameof(Foo) // => "Foo"
    String c = $nameof(Bar); // => "Foo" 
    String d = $nameof(g); // => "g"

### $offsetof

Returns the offset of a member in a struct.

    $offsetof(Foo.y); // => returns 8 on 64 bit, 4 on 32 bit

### $qnameof

Returns the same as `$nameof`, but with the full module name prepended.

    int x;
    String a = $qnameof(int[4]); // => "int[4]"
    String b = $qnameof(Foo) // => "test::bar::Foo"
    String c = $qnameof(Foo[4]); // => "test::bar::Foo[4]" 
    String d = $qnameof(g); // => "test::bar::g"
    
### $sizeof

This is used on a value to determine the allocation size needed. `$sizeof(a)` is equivalent
to doing `$typeof(a).sizeof`. Note that this is only used on values and not on types.

    $typeof(a)* x = allocate_bytes($sizeof(a));
    *x = a;

### $stringify

Returns the expression as a string. It has a special behaviour for macro expression parameters,
where `$stringify(#foo)` will return the expression contained in `#foo` rather than simply return
"#foo"

### $typeof

Returns the type of an expression or variable as a type itself.

    Foo f;
    $typeof(f) x = f;

