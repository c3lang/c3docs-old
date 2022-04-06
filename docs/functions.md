# Functions

C3 has both regular functions and member functions. Member functions are functions namespaced using type names, and allows invocations using the dot syntax.

## Regular functions

Regular functions are the same as C aside from the keyword `fn`, which is followed by the conventional C declaration of `<return type> <name>(<parameter list>)`.


    fn void test(int times)
    {
        for (int i = 0; i < times; i++)
        {
            libc::printf("Hello %d\n", i);
        }
    }

### Function arguments

C3 allows use of default arguments as well as named arguments. Note that
named and unnamed arguments cannot be combined.

    fn int testWithDefault(int foo = 1)
    {
        return foo;
    }

    fn void test()
    {
        testWithDefault();
        testWithDefault(100);
    }

Named arguments

    fn void testNamed(int times, double data)
    {
        for (int i = 0; i < times; i++)
        {
            printf("Hello %d\n", i + data);
        }
    }

    fn void test()
    {
        testNamed(.data = 3.0, .times = 1)
        testNamed(3, 4.0);
    }

Named arguments with defaults:

    fn void testNamedDefault(int times = 1, double data = 3.0, bool dummy = false)
    {
        for (int i = 0; i < times; i++)
        {
            printf("Hello %d\n", i + data);
        }
    }

    fn void test()
    {
        testNamed(.data = 3.0)
    
        // Mixing named and defaults:
        testNamed(3, .dummy = false);
    
         // Not allowed:
         testNamed(1, .data = 1.0); // ERROR
    }

#### Varargs

There are three types of varargs: 
the usual C-style untyped varargs, typed varargs and variant varargs. 
Untyped varargs will always send arguments as-is, whereas typed 
arguments will do normal conversions. The variant varargs will implicitly
convert the arguments to variant values.

    fn void varargsUntyped(string foo, ...)
    {
        /* ... */
    }

    fn void varargsTyped(string bar, int... ints)
    {
        /* ... */
    }

    fn void varargsVariant(string bar, values...)
    {
        variant[] v = values;
        /* ... */
    }

    fn void test()
    {
        varargsUntyped("Hello", 2, 1.0, (char)1, "Test");
        varargsTyped("Test", 2, (char)1);
        // The second parameter will be converted to an int implicitly.
        varargsVariant("Hello", 2, 1.0, (char)1, "Test");
    }

### Functions and optional returns

The return parameter may be an *optional result type* – a type suffixed by `!` indicating that this 
function might either return a regular value or an *optional result value*.

The below example might return optional values from both the `SomeError` optional enum as well as 
the `OtherResult` type.

    fn double! testError()
    {
        double val = random_value();
        if (val >= 0.2) return SomeError.BAD_JOSS_ERROR!;
        if (val > 0.5) return OtherError.BAD_LUCK_ERROR!;
        return val;
    }

*A function call* which is passed one or more *optional result type* arguments will only execute 
if all optional values contain *expected results*, otherwise *the first* *optional result value* is returned.

    fn void test()
    {
        // The following line is either prints a value less than 0.2
        // or does not print at all:
        printf("%d\n", testError());
        
        double x = (testError() + testError()) else 100;
        
        // This prints either a value less than 0.4 or 100:
        printf("%d\n", x);
    }

This allows us to chain functions:

    fn void printInputWithExplicitChecks()
    {
        string! line = readLine();
        if (try line)
        {
            // line is a regular "string" here.
            int! val = atoi(line);
            if (try val)
            {
                printf("You typed the number %d\n", val);
                return;
            }
        }
        printf("You didn't type an integer :(\n");    
    }
    
    fn void printInputWithChaining()
    {
        if (try int val = atoi(readLine()))
        {
            printf("You typed the number %d\n", val);
            return;
        }
        printf("You didn't type an integer :(\n");    
    }

## Methods

Methods look exactly like functions, but are prefixed with the type name and is (usually) 
invoked using dot syntax:

    struct Point
    {
        int x;
        int y;
    }
    
    fn void Point.add(Point* p, int x) 
    {
        p.x = x;
    }
    
    fn void example() 
    {
        Point p = { 1, 2 }
        
        // with struct-functions
        p.add(10);
        
        // Also callable as:
        Point.add(&p, 10);
    }


Struct and unions will always take pointer, whereas enums take the enum value.

    enum State
    {
        STOPPED,
        RUNNING
    }
    
    fn bool State.mayOpen(State state) 
    {
        switch (state)
        {
            case STOPPED: return true;
            case RUNNING: return false;
        }
    }


### Restrictions on methods

- Methods on a struct/union may not have the same name as a member.
- Methods only works on distinct, struct, union and enum types.
- When taking a function pointer of a method, use the full name.
- Using sub types, overlapping function names will be shadowed.

## Contracts

C3's error handling is not intended to use errors to signal invalid data or to check invariants and post conditions. Instead C3's approach is to add annotations to the function, that conditionally will be compiled into asserts.

As an example, the following code:

    /**
     * @param foo : the number of foos 
     * @require foo > 0, foo < 1000
     * @return number of foos x 10
     * @ensure return < 10000, return > 0
     **/
    fn int testFoo(int foo)
    {
        return foo * 10;
    }

Will in debug builds be compiled into something like this:

    fn int testFoo(int foo)
    {
        assert(foo > 0);
        assert(foo < 1000);
        int _return = foo * 10;
        assert(_return < 10000);
        assert(_return > 0);
        return _return;
    }

The compiler is allowed to use the contracts for optimizations. For example this:

    fn int testExample(int bar)
    {
        if (testFoo(bar) == 0) return -1;
        return 1;
    }

May be optimized to:

    fn int testExample(int bar)
    {
        return 1;
    }

In this case the compiler can look at the post condition of `result > 0` to determine that `testFoo(foo) == 0` must always be false.

Looking closely at this code, we not that nothing guarantees that `bar` is not violating the preconditions. In Safe builds this will usually be checked in runtime, but a sufficiently smart compiler will warn about the lack of checks on `bar`. Execution of code violating pre and post conditions has unspecified behaviour.
