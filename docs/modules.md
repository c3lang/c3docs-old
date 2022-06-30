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

Modules are imported using the `import` statement. Imports always *recursively import* sub-modules. Any module
will automatically import all other modules with the same parent module.

`foo.c3`

    module some::foo;
    fn void test() {}

`bar.c3`

    module bar;
    import some;
    // import some::foo; <- not needed, as it is a sub module to "some"
    fn void test()
    {
        foo::test();
        // some::foo::test() also works.
    }

In some cases there may be ambiguities, in which case the full path can be used to resolve the ambiguity:

`abc.c3`

    module abc;
    struct Context
    {
        int a;
    }

`def.c3`

    module def;
    struct Context
    {
        void* ptr;
    }

`test.c3`

    module test1;
    import def, abc;
    // Context c = {} <- ambiguous
    abc::Context c = {};

## Implicit imports

The module system will also implicitly import:

1. The `std::core` module (and sub modules).
2. Any other module sharing the same top module. E.g. the module `foo::abc` will implicitly also import modules `foo` and `foo::cde` if they exist.

## Visibility

All files in the same module share the same global declaration namespace. By default a symbol is visible to all other modules.
To make a symbol only visible inside the module, use the keyword 
`private`.

```
module foo;

fn void init() { .. }

private fn void open() { .. }
```

In this example, the other modules can use the init() function after importing foo, but only files in the foo module can use open(), as it is specified as `private`.

## Overriding symbol visibility rules

By using `import private`, it's possible to access another module´s private symbols.
Many other module systems have hierarchal visibility rules, but the `import private` feature allows 
visibility to be manipulated in a more ad-hoc manner without imposing hard rules.

For example, you may provide a library with two modules: "mylib::net" and "mylib::file" - which both use functions
and types from a common "mylib::internals" module. The two libraries use `import private mylib::internals`
to access this module's private functions and type. To an external user of the library, the "mylib::internals"
does not seem to exist, but inside of your library you use it as a shared dependency.

A simple example:
```c
// File a.c3
module a;

private fn void aFunction() { ... }

// File b.c3
module b;

private fn void bFunction() { ... }

// File c.c3
module c;
import a;
import private b;

fn void test() 
{
  a::aFunction(); // <-- error, aFunction is private
  b::bFunction(); // Allowed since import was "private"
}
```

## Private modules

Modules may be declared using `private`, this makes the whole module private. Such a module must
always be imported using `import private`. Note that `import private` is *not* recursive.

```
module private foo;
...

module bar;
import private foo; // Allowed
...

module baz;
import foo; // Error, trying to import private module. 
```

## Using functions and types from other modules

As a rule, functions, macros, constants, variables and types in the same module do not need any namespace prefix. For imported modules the following rules hold:

1. Functions, macros, constants and variables require *at least* the (sub-) module name.
2. Types do not require the module name unless the name is ambiguous.
3. In case of ambiguity, only so many levels of module names are needed as to make the symbol unambiguous.


```
// File a.c3

module a;

struct Foo { ... }
struct Bar { ... }
struct TheAStruct { ... }

fn void anAFunction() { ... }

// File b.c3

module b;

struct Foo { ... }
struct Bar { ... }
struct TheBStruct { ... }

fn void aBFunction() { ... }

// File c.c3
module c;
import a;
import b;

struct TheCStruct { ... }
struct Bar { ... }

fn void aCFunction() { ... }

fn void test()
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


## Versioning and dynamic inclusion

_NOTE: This feature may significantly change._

When including *dynamic* libraries, it is possible to use optional functions and globals. This is done using the
`@dynamic` attribute.

An example library could have this:

`dynlib.c3i`

    module dynlib;
    fn void do_something() @dynamic(4.0)
    fn void do_something_else() @dynamic(0, 5.0)
    fn void do_another_thing() @dynamic(0, 2.5)

Importing the dynamic library and setting the base version to 4.5 and minimum version to 3.0, we get the following:

`test.c3`

    import dynlib;
    fn void test()
    {
        if (@available(dynlib::do_something))
        {
            dynlib::do_something();
        }
        else
        {
            dynlib::do_someting_else();
        }  
    }

In this example the code would run `do_something` if available (that is, when the dynamic library is 4.0 or higher), or 
fallback to `do_something_else` otherwise.

If we tried to conditionally add something not available in the compilation itself, that is a compile time error:

    if (@available(dynlib::do_another_thing))  
    {
        dynlib::do_another_thing(); // Error: This function is not available with 3.0
    }

Versionless dynamic loading is also possible:

`maybe_dynlib.c3i`

    module maybe_dynlib;
    fn void testme() @dynamic;

`test2.c3`

    import maybe_dynlib;
    fn void testme2()
    {
        if (@available(maybe_dynlib::testme))
        {
            dynlib::testme();
        }
    }

This allows things like optionally loading dynamic libraries on the platforms where this is available.

## Textual includes

It's sometimes useful to include an entire file, doing so employs the `$include` function.

_NOTE: This feature may significantly change._

File `Foo.c3`
```
module foo;

$include("Foo.x");

fn void test() 
{
    printf("%d", testX(2));
}    
```

File `Foo.x`
```
fn testX(int i) 
{ 
    return i + 1; 
}
```

The result is as if `Foo.c3` contained the following:

```
module foo;

fn testX(int i) 
{ 
    return i + 1; 
}

fn void test() 
{
    printf("%d", testX(2));
}    
```

The include may use an absolute or relative path, the relative path is always relative to the source file in which the include appears.


