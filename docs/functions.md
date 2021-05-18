# Functions

C3 has both regular functions and member functions. Member functions are name spaced using a type name, and allows invocations using the dot syntax.

## Regular functions

Regular functions looks similar to C. It starts with the keyword `func`, followed by the conventional C declaration of `<return type> <name>(<parameter list>)`. C3 adds a `throws` declaration, that will declare that the function might return an error instead of its usual return value.

```
func void test(int times)
{
    for (int i = 0; i < times; i++)
    {
        printf("Hello %d\n", i);
    }
}
```

### Function arguments

C3 allows use of default arguments as well as named arguments.

```
func int testWithDefault(int foo = 1)
{
    return foo;
}

func void test()
{
    testWithDefault();
    testWithDefault(100);
}
```

Named arguments

```
func void testNamed(int times, double data)
{
    for (int i = 0; i < times; i++)
    {
        printf("Hello %d\n", i + data);
    }
}

func void test()
{
    testNamed(.data = 3.0, .times = 1)
    testNamed(3, 4.0);
}
```

Named arguments with defaults:

```
func void testNamedDefault(int times = 1, double data = 3.0, bool dummy = false)
{
    for (int i = 0; i < times; i++)
    {
        printf("Hello %d\n", i + data);
    }
}

func void test()
{
    testNamed(.data = 3.0)
    
    // Mixing named and defaults:
    testNamed(3, .dummy = false);
    
    // Mixing named and defaults leaving out initial values:
    testNamed(,,false, .times = 1);
}
```

#### Varargs

There are two types of varargs: the usual C-style untyped varargs and typed varargs. Untyped varargs will always send arguments as-is, whereas typed arguments will do normal conversions.


```
func void varargsUntyped(string foo, ...)
{
    /* ... */
}

func void varargsTyped(string bar, int... ints)
{
    /* ... */
}

func void test()
{
    varargsUntyped("Hello", 2, 1.0, (char)(1), "Test");
    varargsTyped("Test", 2, (char)(1));
    // The second parameter will be converted to an int implicitly.
}
```

### Functions and failables

The return parameter may be a *failable* – a type suffixed by `!` indicating that this function might either return a regular value or an error.

The below example might throw errors from both the `SomeError` error domain as well as the `OtherError` error domain.

```
func double! testError() throws
{
    double val = random_value();_
    if (val >= 0.2) return BadJossError!;
    if (val > 0.5) return BadLuckError!;
    return val;
}
```

*A function* that is passed a *failable* value will only conditionally execute if and only if all failable values evaluate to true, otherwise *the first* error is returned.

```
func void test()
{
    // The following line is either prints a value less than 0.2
    // or does not print at all:
    printf("%d\n", testError());
    
    double x = (testError() + testError()) else 100;
    
    // This prints either a value less than 0.4 or 100:
    printf("%d\n", x);
}
```

This allows us to chain functions:

```

func void printInputWithExplicitChecks()
{
    string! line = readLine();
    if (line)
    {
        // line is a regular "string" here.
        int! val = atoi(line);
        if (val)
        {
            printf("You typed the number %d\n", val);
            return;
        }
    }
    printf("You didn't type an integer :(\n");    
}

func void printInputWithChaining()
{
    if (int val = atoi(readLine()))
    {
        printf("You typed the number %d\n", val);
        return;
    }
    printf("You didn't type an integer :(\n");    
}
```

## Methods

Methods look exactly like functions, but are prefixed with the struct, union or enum name and is (usually) invoked using dot syntax:

```
struct Point
{
    int x;
    int y;
}

func void Point.add(Point* p, int x) 
{
    p.x = x;
}

func void example() 
{
    Point p = { 1, 2 }

    // with struct-functions
    p.add(10);

    // Also callable as:
    Point.add(&p, 10);
}
```

If a method does not take the type as the first parameter, then it may only be invoked qualified with the type name:

```
func Point* Point.new(int x, int y) 
{
    Point* p = malloc(@sizeof(Point));
    p.x = x;
    p.y = y;
    return p;
}

func void example2() 
{
    Point* p = Point.new(1, 2);
}
```

Struct and unions will always take pointer, whereas enums take the enum value.

```
enum State
{
    STOPPED,
    RUNNING
}

func bool State.mayOpen(State state) 
{
    switch (state)
    {
        case State.STOPPED: return true;
        case State.RUNNING: return false;
    }
}
```


### Restrictions on methods

- Methods on a struct/union may not have the same name as a member.
- Methods only works on distinct, struct, union and enum types.
- When taking a function pointer of a method, use the full name.
- Using sub types, overlapping function names will be shadowed.

## Pre and post conditions

C3's error handling is not intended to use errors to signal invalid data or to check invariants and post conditions. Instead C3's approach is to add annotations to the function, that conditionally will be compiled into asserts.

As an example, the following code:
```
/**
 * @param foo : the number of foos 
 * @require foo > 0, foo < 1000
 * @return number of foos x 10
 * @ensure return < 10000, return > 0
 **/
func int testFoo(int foo)
{
    return foo * 10;
}
```

Will in debug builds be compiled into something like this:

```
func int testFoo(int foo)
{
    assert(foo > 0);
    assert(foo < 1000);
    int _return = foo * 10;
    assert(_return < 10000);
    assert(_return > 0);
    return _return;
}
```

The compiler is allowed to use the pre and post conditions for optimizations. For example this:

```
func int testExample(int bar)
{
    if (testFoo(bar) == 0) return -1;
    return 1;
}
```

May be optimized to:

```
func int testExample(int bar)
{
    return 1;
}
```

In this case the compiler can look at the post condition of `result > 0` to determine that `testFoo(foo) == 0` must always be false.

Looking closely at this code, we not that nothing guarantees that `bar` is not violating the preconditions. In debug builds this will usually be checked in runtime, but a sufficiently smart compiler will warn about the lack of checks on `bar`. Execution of code violating pre and post conditions has undefined behaviour.
