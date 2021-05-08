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
module bar;

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


## Visibility

All files in the same module share the same global declaration namespace. However, by default a function is not visible outside the module. To make the symbol visible outside the module, use the keyword `public`.

```
module foo;

public func void init() { .. }

func void open() { .. }
```

In this example, the other modules can use the init() function after importing foo, but only files in the foo module can use open(), as it isn't specified as public.

## Overriding symbol visibility rules

By using `as module` after an import, it's possible to access another module´s private symbols.
Many other module systems have hierarchal visibility rules, but the `as module` feature allows 
visibility to be manipulated in a more ad-hoc manner without imposing hard rules.

For example, you may provide a library with two modules: "mylib::net" and "mylib::file" - which both use functions
and types from a common "mylib::internals" module. The two libraries use `import mylib::internals as module`
to access this module's private functions and type. To an external user of the library, the "mylib::internals"
does not seem to exist, but inside of your library you use it as a shared dependency.

A simple example:
```c
// File a.c3
module a;

func void aFunction() { ... }

// File b.c3
module b;

func void bFunction() { ... }

// File c.c3
module c;
import a;
import b as module;

func void test() 
{
  a::aFunction(); // <-- error, aFunction not public
  b::bFunction(); // Allowed since import was "as module"
}
```

## Using functions and types from other modules

As a rule, functions, macros, constants, variables and types in the same module do not need any namespace prefix. For imported modules the following rules hold:

1. Functions, macros, constants and variables require *at least* the (sub-) module name.
2. Types do not require the module name unless the name is ambiguous.
3. In case of ambiguity, only so many levels of module names are needed as to make the symbol unambiguous.


```
// File a.c3

module a;

public struct Foo { ... }
public struct Bar { ... }
public struct TheAStruct { ... }

public func void anAFunction() { ... }

// File b.c3

module b;

public struct Foo { ... }
public struct Bar { ... }
public struct TheBStruct { ... }

public func void aBFunction() { ... }

// File c.c3
module c;
import a;
import b;

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

> Types are used without prefix; functions, variables, macros and constants are prefixed with the sub module name.


## Textual includes

It's sometimes useful to include an entire file, doing so employs the `$include` function.

File `Foo.c3`
```
module foo;

$include("Foo.x");

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


