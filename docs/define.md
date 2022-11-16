# The "define" statement

The `define` statement in C3 encompasses the `typedef` of C, as well as aliasing using 
`#define`.

## Defining a type alias

`define <type alias> = <type>` creates a type alias, just as if one had used 
`typedef` in C. Type aliases need to follow the name convention of user defined types (i.e. capitalized
names with at least one lower case letter).

    define CharPtr = char*;
    define Numbers = int[10];

Function pointers _must_ be aliased in C3. The syntax is somewhat different from C:

    define Callback = fn void(int a, bool b);

This defines an alias to function pointer type of a function that returns nothing and requires two arguments: an int and a bool. Here is a sample usage:

    Callback cb = my_callback;
    cb(10, false);


## Distinct types

`define` may also be used to create distinct new types. Unlike type aliases,
they do not implicitly convert to any other type.

    define Foo = distinct int;
    Foo f = 0;
    f = f + 1;
    int i = 1;
    // f = f + i Error!
    f = f + (Foo)i; // Valid

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

## Using define to create generic types, functions and variables

Generic modules uses `define` to create aliases to parameterized types, functions 
and variables:

     import generic_foo;

     // Parameterized function aliases
     define int_foo_call = generic_foo::foo_call<int>;
     define double_foo_call = generic_foo::foo_call<double>;
  
     // Parameterized type aliases
     define IntFoo = Foo<int>;
     define DoubleFoo = Foo<double>;

     // Parameterized global aliases
     define int_max_foo = generic_foo::max_foo<int>;
     define double_max_foo = generic_foo::max_foo<double>;

For more information, see the chapter on [generics](../generics).

## Function pointer default arguments and named parameters

It is possible to attach default arguments to function pointer aliases. There is no requirement
that the function has the same default arguments. In fact, the function pointer may have 
default arguments where the function doesn't have it and vice-versa. Calling the function
directly will then use the function's default arguments, and calling through the function pointer
will yield the function pointer alias' default argument.

Similarly, named parameter arguments follow the alias definition when calling through the 
function pointer:

    define TestFn = fn void(int y = 123);

    fn void test(int x = 5)
    {
        io::printfln("X = %d");
    }

    fn void main()
    {
        TestFn test2 = &test;
        test();        // Prints X = 5
        test2();       // Prints X = 123
        test(.x = 3);  // Prints X = 3 
        test2(.y = 4); // Prints X = 4
    }

