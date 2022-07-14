# Reflection

C3 allows both compile time and runtime reflection.

During compile time the type information may be directly used as compile time constants, the same data is then available dynamically at runtime.

*Note that not all reflection is implemented in the compiler at this point in time.*

## Compile time reflection

During compile time there are a number of compile time fields that may be accessed directly.

### Compile time variables

#### typeid

Returns the typeid for the given type. Typedefs will return the typeid of the underlying type. The typeid size is the same as that of an `iptr`.

    typeid x = Foo.typeid;

#### sizeof

Returns the size in bytes for the given type, like C `sizeof`.

    usize x = Foo.sizeof;

#### length

*Not yet implemented*

*Only available for array and vector types.*
Returns the length of the array.

```
usize len = int[4].length
```


#### values

Returns a subarray containing the values of an enum or fault.

    enum FooEnum
    {
        BAR,
        BAZ
    }
    char[] x = $nameof(FooEnum.values[1]); // "BAR"

#### fields

*Not yet implemented*

*Only available for struct types.*
Returns an array containing the fields in a struct or enum.

    struct Baz
    {
        int x;
        Foo* z;
    }
    char[] x = $nameof(Baz.fields[1]); // "z"

#### signed

*Not yet implemented*

*Only available for integer types.*
Returns true for a signed number.

    bool s1 = int.signed; // => true
    bool s2 = uint.signed; // => false


#### associated

*Not yet implemented*

*Only available for enums.*
Returns an array containing the types of associated values if any.

    enum Foo : int(double d, char[] s)
    {
        BAR(1.0, "normal"),
        BAZ(2.0, "exceptional")
    }
    char[] s = $nameof(Foo.associated[0]); // "double"


#### returns

*Not yet implemented*

*Only available for function types.*
Returns the type of the return type.

    define TestFunc = fn int(int, double);
    char[] s = $nameof(TestFunc.returns); // "int"


#### params

*Not yet implemented*

*Only available for function types.*
Returns a list of all parameters.

    define TestFunc = fn int(int, double);
    char[] s = $nameof(TestFunc.params[1]); // "double"


### Compile time functions

There are several built-in functions to inspect the code during compile time.

- `$alignof`
- `$defined`
- `$nameof`
- `$qnameof`
- `$extnameof`
- `$offsetof`
- `$sizeof`
- `$typeof`
- `$stringify`
- `$typeof`
- `$eval`
- `$evaltype`

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

### $defined

Returns true if the expression inside is defined.

    $defined(Foo.x); // => returns true
    $defined(Foo.z); // => returns false

### $eval

Converts a compile time string with the corresponding variable:

    int a = 123;         // => a is now 123
    $eval("a") = 222;    // => a is now 222
    $eval("mymodule::fooFunc")(a); // => same as mymodule::fooFunc(a) 

### $evaltype

Similar to `$eval` but for types:

    $evaltype("float") f = 12.0f;

### $extnameof

Returns the external name of a type, variable or function. The external name is
the one used by the linker.

    fn void testfn(int x) { }
    char[] a = $extnameof(g); // => "test.bar.g";
    string b = $extnameof(testfn); // => "test.bar.testfn"

### $nameof

Returns the name of a function, type or variable as a string without module prefixes.

    define Bar = Foo;
    char[] a = $nameof(int[4]); // => "int[4]"
    char[] b = $nameof(Foo) // => "Foo"
    char[] c = $nameof(Bar); // => "Foo" 
    char[] d = $nameof(g); // => "g"

### $offsetof

Returns the offset of a member in a struct.

    $offsetof(Foo.y); // => returns 8 on 64 bit, 4 on 32 bit

### $qnameof

Returns the same as `$nameof`, but with the full module name prepended.

    int x;
    char[] a = $nameof(int[4]); // => "int[4]"
    char[] b = $nameof(Foo) // => "test::bar::Foo"
    char[] c = $nameof(Foo[4]); // => "test::bar::Foo[4]" 
    char[] d = $nameof(g); // => "test::bar::g"
    


### $stringify

Returns the expression as a string. It has a special behaviour for macro expression parameters,
where `$stringify(#foo)` will return the expression contained in `#foo` rather than simply return
"#foo"

### $typeof

Returns the type of an expression or variable as a type itself.

    Foo f;
    $typeof(f) x = f;


** The following are not yet implemented **

#### $kindof

Returns the TypeKind of the variable.

    union Bar { ... }
    TypeKind a = $kindof(Foo); // STRUCT
    TypeKind b = $kindof(Bar); // BAR
    TypeKind c = $kindof(int); // INTEGER

#### $elementof

*Only available for array, optional, pointer, vector and subarray types.*
Returns the element (underlying) type of an array, pointer, vector or subarray.

    char x = $nameof($elementof(Foo[])); // "Foo"
    char y = $nameof($elementof(Foo[4])); // "Foo"
    char z = $nameof($elementof(Foo*)); // "Foo"