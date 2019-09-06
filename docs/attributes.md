# Attributes

Attributes are compile time annotations on functions, types, global constants and variables. Similar to Java annotations, a decoration may also take arguments. A attribute can also represent a bundle of attributes.

## Built in attributes

### `packed` (struct, union, enum)

If used on a struct or enum: packs the type, including any components to minimum size. On an enum, it uses the smallest representation containing all its values.

### `section(name)` (var, func)

Declares that a global variable or function should appear in a specific section.

### `inline` (func)

Declares a function to always be inlined.

### `aligned(alignment)` (struct, union, var, func)

This attribute sets the minimum alignment for a field or a variable.

### `noreturn` (func)

Declares that the function will never return.

### `weak` (func, var)

Emits a weak symbol rather than a global. 

### `opaque` (struct, union, enum)

Prevents the union or struct from being statically allocated by other modules. In the case of enums,
prevents the enum from being converted to a value.

## New attributes

The basic syntax for creating a new attribute is `attribute <declaration type> @<name>`. The type category is one of `var`, `const`, `func`, `union`, `struct`, `enum`, `typedef` and `error`.
   
For example:
 
```
attribute func @myattribute;

func void foo() @myattribute
{ ...  }
```

### Multi type attributes

It's possible to make an attribute match more than a single declaration type, by separating them using comma:

```
attribute func, enum, struct @fooattribute;

func void foo() @fooattribute
{ 
    /* ... */  
}

struct Bar @fooattribute
{ 
    int i;
}
```

### Arguments

An attribute may take a number of arguments, similar to a function. All such arguments must be compile time constants.

```
attribute func @intdec(int i)

void foo() @intdec(2)
{ ... }
```

Just like for function calls, the arguments may have defaults, use named arguments etc. However, varargs are not allowed.

```
attribute func @dec(int d, int i = 0, float f = 1.0)

void foo() @dec(2)
{ ... }

void bar() @dec(7, f = 3.0)
{ ... }
```

    
## Compile time attribute access

What's useful about attributes is that they can be accessed during compile
time:

```
macro @fooCheck($a)
{
    $if (@defined($a.@foo))
    {
        return "Was fooed";
    }
    return "Ok";
}

struct TestA
{
    int i;
}

struct TestB @Foo
{
    float f;
}

func void test()
{
    printf("Check TestA: " @fooCheck(TestA) "\n");    
    printf("Check TestB: " @fooCheck(TestB) "\n");
    // Prints:
    // Check TestA: Ok
    // Check TestB: Was fooed
}
```

The values may be recovered:

It's also possible to get hold of the values:

```
attribute func @myvar(int i = 0);

func void test() @myvar(200) 
{ 
    /* ... */
}

func void test()
{
    printf("%d", test.@myvar.i); // Prints 200
}
```

## Bundled attributes

It's also possible to create bundles of attributes, that simply contain other attributes. Unlike a normal attribute, an attribute bundle is a simple alias, so it has no real compile time information.

```
attribute func @foodec;
attribute func @bardec(int i = 0);

attribute @mixdec alias @foodec, @bardec(3);

// As if we had added @foodec, @bardec(3)
func void test() @mixdec
{
    /* ... */
}
```

A attribute bundle might also be completely empty.

## Iterating over attributes

Within a macro it's possible to iterate in compile time over all non-alias attributes 
using `$each`.

```
attribute struct @special;

struct TestA @special { int i; }
struct TestB { float f; }
struct TestC @special { float f; }

macro @specialStructs()
{
    string[+] res = {};
    $each(@special as $x)
    {
        res += @name($x);
    }
    // The above expands to:
    // res += "TestA";
    // res += "TestC";    
    return res;
}

// SPECIAL_STRUCTS = { "TestA", "TestB" }
const string[] SPECIAL_STRUCTS = @specialStructs();
```
