## The "define" statement

The `define` statement in C3 encompasses the `typedef` of C, as well as aliasing using 
`#define`

### Define a type alias

`define <type alias> = <type>` creates a type alias, just as if one had used 
`typedef` in C. All type aliases need to follow the name convention of user defined types.

```
define CharPtr = char*;
define Numbers = int[10];
```

Function pointers _must_ be aliased in C3. The syntax is somewhat different from C:

`define Callback = fn void(int a, bool b);`

This defines an alias to function pointer type of a function that returns nothing and requires two arguments: an int and a bool. Here is a sample usage:

```
Callback cb = my_callback;
cb(10, false);
```

## Distinct types

`define` may also be used to create distinct new types. Unlike type aliases,
they do not implicitly convert to any other type.

```
define Foo = distinct int;
Foo f = 0;
f = f + 1;
int i = 1;
// f = f + i Error!
f = f + (Foo)i; // Valid
```

## Function and variable aliases

It's possible to use `define` to create aliases for functions and variables.

The syntax is `define <alias> = <original identifier>`.

```
fn void foo() { ... }
int foo_var;

define bar = foo;
define bar_var = foo_var;

fn void test() 
{
  // These are the same:
  foo();
  bar();
  
  // These access the same variable:
  int x = foo_var;
  int y = bar_var;
}  
```

## Using define to create generic types

Generic modules uses `define` to create aliases to parameterized types, functions 
and variables. For more information, see the chapter on [generics](../generics).