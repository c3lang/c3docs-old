# Generics

Generic functionality is mostly provided by macros and generic modules. Bridging the gap is generic functions, that allow a limited for of overloading.

## Generic functions

Generic functions is a specialized type of macros. They allow overloading using a common symbol, just like C11's `_Generic`:

```
generic abs(x)
{
    case double:
        return fabs(x);
    case int:
        return abs(x); // Redefining abs(!)
}
```

Unlike a regular macro, a generic function is not invoked with @. It may also be extended by other packages / files:

```
// File 1
module foo;
generic abs(x)
{
    case double:
        return fabs(x);
    case int:
        return abs(x);
}

// File 2
module bar;
generic foo::abs(x)
{
    case long:
        return absl(x);
    case int: // Compile time error, already defined
        return abs(x);
}
```

It is possible to dispatch on multiple arguments:

```
module foo;
generic add(x, y)
{
    case double, double:
        return x + y;
    case int, double:
        return (double)x + y;
}
```

Generics can also be used for overloading operators:

```
generic operator_add(x, y)
{
    case vector3, vector3:
        return vector_add(x, y);
}

generic operator_index(x, y)
{
    case DynArray, int:
        return x.get(y); // Enables foo[12]
}

generic operator_index_assign(x, y, z)
{
    case DynArray, int:
        return x.set(y, z); // Enables foo[12] = 2
}
```


Note that generics is actually a macro expansion. Consequently this is possible:

```
generic weird_fun(x)
{
    case int:
        printf("%d", x);
    case Foo:
        while (foo.bar > 0)
        {
            foo.bar--;
            printf("Hi");
        }
}

void func test()
{
    Foo foo;
    weird_fun(2); // Prints "2"
    foo.bar = 4;
    wierd_fun(foo);  // Prints "HiHiHiHi"
}
```

#### Shorthand

There is also a shorthand for declaring generics that only match a single type – it's done by declaring the types directly in the header:

```
generic add(double x, double y)
{
    return x + y;
}
```

## Generic modules

Generic modules are parameterized modules that allow functionality for arbitrary types.

For generic modules, the generic parameters follows the module name:

```
module vector (A, B, C); // A, B, C are generic parameters.
```

The code inside of the module can use the generic parameters as if they were well defined symbols:

```
module foo_test (A, B);

struct Foo {
   A a;
}

func C test(B b, Foo *foo) {
   return a + b;
}
```

Including a generic module works as usual, but to use a type, it must be *defined* before use.

```
import foo_test;

define Foo(float, double) as FooFloat;
define foo_test::test(float, double) as testFloat;

...

FooFloat f;

...

testFloat(1.0, f);

```

Just like for macros, optional constraints may be added to improve compile errors:

```
/**
 * @require c = a + b
 */ 
module vector (A, B, C);

/* .. code * ../
```

```
import vector(Bar, f32, i32) as gen_test;

// This would give the error 
// --> Illegal arguments for generic module vector, breaks requirement 'Bar' = 'f32' + 'i32'
```

