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

### Using throws

The throws parameter may return a generic throws clause – in this case the error type is unknown.

The below example might throw errors from both the `SomeError` error domain as well as the `OtherError` error domain.

```
func int test() throws
{
    if (random_value() > 0.2) throw SomeError.BAD_JOSS_ERROR;
    if (random_value() > 0.5) throw OtherError.BAD_LUCK_ERROR;
    return 0;
}
```

The errors can be explicitly documented, by listing the error domains, separated by `,`

```
func int test() throws SomeError, OtherError
{
    if (random_value() > 0.2) throw SomeError.BAD_JOSS_ERROR;
    if (random_value() > 0.5) throw OtherError.BAD_LUCK_ERROR;
    return 0;
}
```

## Member functions

Member functions look exactly like functions, but are prefixed with the struct, union or enum name:

```
struct Point
{
    int x;
    int y;
}

func void Point.add(Point& p, int x) 
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

If a member function does not take the type as the first parameter, then it may only be invoked qualified with the type name:

```
func Point& Point.new(int x, int y) 
{
    Point &p = malloc(@sizeof(Point));
    p.x = x;
    p.y = y;
    return p;
}

func void example2() 
{
    Point &p = Point.new(1, 2);
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


### Restrictions on member functions

Member functions may not:

- Member functions on a struct/union may not have the same name as a member.
- Member functions only works on struct, union and enum types.
- When taking a function pointer of a member function, use the full name.
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
