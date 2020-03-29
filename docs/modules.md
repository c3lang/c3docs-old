# Modules

C3 groups functions, types, variables and macros into namespaces called modules. When doing builds, any C3 file must start with the `module` keyword, specifying the module. When compiling single files, the module is not needed and the module name is assumed to be the file name, converted to lower case, with any invalid characters replaced by underscore (`_`).

A module can consist of multiple files, e.g.

`file_a.c3`

```
module foo;

/* ... */
```

`file_b.c3`

```
module foo;

/* ... */
```

`file_c.c3`

```
module baz;

/* ... */
```

Here `file_a.c3` and `file_b.c3` belong to the same module, **foo** while `file_c.c3` belongs to to **bar**.

## Details

Some details about the C3 module system:

- Modules can be arbitrarily nested, e.g. `module foo::bar::baz;` to create the sub module baz in the sub module `bar` of the module `foo`.
- Module names must be alphanumeric lower case letters plus the underscore character: `_`.
- Module names are limited to 31 characters.

## Importing modules

Importing a module uses the `import` keyword. Imports have file scope, so consequently if `file_a.c3` imports the module `networking`, then `file_b.c3` cannot use those symbols unless it also imports `networking`.

`file_a.c3`
```
module foo;

//import bar and stdio
import bar;
import stdio;

/* ... */
```

`file_b.c3`
```
module foo;

//import bar and networking imported, but not storage
import bar;
import networking;

/* ... */
```


### Restricted imports

It is possible to restrict imports to sub modules or individual functions.

```
module foo;

import bar::baz; // Only import the baz sub module.
import some_module : some_function; // Only import some_function
import some_module : SomeType, SomeOtherType, aFunc; // Import multiple symbols.
```

### Aliased imports

To avoid name collisions, it's possible to alias names. This will completely hide their old name in the file scope.

```
module foo;

import bar::io as bio;
import baz::io;
import some_module : SomeType as FooType;
import foo : funcA as fooFuncA, funcB as fooFuncB;
```

## Visibility

All files in the same module share the same global declaration namespace. However, by default a function is not visible outside the module. To make the symbol visible outside the module, use the keyword `public`.

```
module foo;

public func void init() { .. }

func void open() { .. }
```

In this example, the other modules can use the init() function after importing foo, but only files in the foo module can use open(), as it isn't specified as public.

### Visibility in sub modules

For declarations in sub modules, the rule is that a sub module may see any parent name, and any children name. However, other children sub modules are hidden.

Consider the modules `foo`, `foo::bar`, `foo::baz` and `foo::bar::xyzzy`. 

1. From `foo::bar` the symbols of `foo`, `foo::bar` and `foo::bar::xyzzy` are visible. 
2. From `foo::baz` only `foo` and `foo::baz` are visibile. 
3. Finally from `foo`, all the symbols of `foo`, `foo::bar`, `foo::baz` and `foo::bar::xyzzy` are visibile.

## Using functions and types from other modules

As a rule, functions, macros, constants, variables and types in the same module do not need any namespace prefix. For imported modules, and for parent or child modules the following rules hold:

1. Functions, macros, constants and variables require *at least* the (sub-) module name.
2. Types do not require the module name except if the name is ambiguous.
3. In case of ambiguity, only so many levels of module names are needed as to make the symbol unambiguous.


```
// File a.c3

module a;

struct Foo { ... }
struct Bar { ... }
struct TheAStruct { ... }

func void anAFunction() { ... }

// File b.c3

module b;

struct Foo { ... }
struct Bar { ... }
struct TheBStruct { ... }

func void aBFunction() { ... }

// File c.c3

module b::c;
import a;

struct TheCStruct { ... }
struct Bar { ... }

func void aCFunction() { ... }

func void test()
{
    TheAStruct stA;
    TheBStruct stB;
    TheCStruct stC;
    // Name required to avoid ambiguity;
    b::Foo stBFoo;
    // Will always pick the current module's 
    // name.
    Bar bar;
    // Namespace required:
    a::aAFunction();
    b::aBFunction();
    // A local symbol does not require it:
    aCFunction(); 
}
```

This means that the rule for the common case can be summarized as

> Types without prefix, functions, variables, macros and constants prefixed with the sub module name.


## Textual includes

It's sometimes useful to include an entire file, doing so employs the `@include` macro.

File `Foo.c3`
```
module foo;

@include("Foo.x");

func void test() 
{
    printf("%d", testX(2));
}    
```

File `Foo.x`
```
public func testX(int i) 
{ 
    return i + 1; 
}
```

The result is as if `Foo.c3` contained the following:

```
module foo;

public func testX(int i) 
{ 
    return i + 1; 
}

func void test() 
{
    printf("%d", testX(2));
}    
```

The include may use an absolute or relative path, the relative path is always relative to the source file in which the include appears.


