# Macros

The macro capabilities of C3 reaches across several constructs: macros (prefixed with `@` at invocation), [generic functions](../generics/#generic-functions), [generic modules](../generics/#generic-modules), compile time variables (prefixed with `$`), macro compile time execution (using `$if`, `$for`, `$foreach`, `$switch`) and attributes.

## A quick comparison of C and C3 macros

### Conditional compilation

    // C
    #if defined(x) && Y > 3
    int z;
    #endif
    
    // C3
    $if $defined(x) && $y > 3:
        int z;
    $endif;

### Macros

    // C
    #define M(x) ((x) + 2)
    #define RETURN(x) return x;
    #define u32 unsigned int
    
    // Use:
    int y = M(foo() + 2);
    RETURN(bar());
    u32 b = y;
    
    // C3
    macro m(x)
    {
        return x + 2;
    }
    macro ret(x) @escape
    {
        return x;
    }
    define u32 = uint;
    
    // Use:
    int y = @m(foo() + 2);
    @ret(bar());
    u32 b = y;

### Dynamic scoping

    // C
    #define Z() ptr->x->y->z
    int x = Z();
    
    // C3
    ... currently no corresponding functionality ...


### Reference arguments

Use `&` in front of a parameter to capture the a variable and pass it by reference without having to explicitly use `&` and pass a pointer. 
(Note that in C++ this is allowed for normal functions, whereas for C3 it is only permitted with macros.)

    // C
    #define M(x, y) x = 2 * (y);
    
    // C3
    macro m(int &x, int y)
    {
        x = 2 * y;
    }


### First class types

    // C
    #define SIZE(T) (sizeof(T) + sizeof(int))
    
    // C3
    macro size($Type)
    {
        return $Type.sizeof + int.sizeof;
    }

### Trailing blocks for macros

    // C
    #define FOR_EACH(x, list) \
    for (x = (list); x; x = x->next)
    
    // Use:
    Foo *it;
    FOR_EACH(it, list) 
    {
        if (!process(it)) return;
    }
    
    
    // C3
    macro for_each(list; @body(it))
    {
        for ($typeof(list) x = list; x; x = x.next)
        {
            @body(x);
        }    
    }
    
    // Use:
    @for_each(list; Foo* x)
    {
        if (!process(x)) return;
    }

### First class names

    // C
    #define offsetof(T, field) (size_t)(&((T*)0)->field)
    
    // C3
    macro usize offset($Type, #field)
    {
        $Type* t = null;
        return (usize)(uptr)&t.#field;
    }

### Declaration attributes


    // C
    #define PURE_INLINE __attribute__((pure)) __attribute__((always_inline))
    int foo(int x) PURE_INLINE { ... }
    
    // C3
    define @PureInline = @pure, @inline
    fn int foo(int) @PureInline { ... }    


### Declaration macros

    // C
    #define DECLARE_LIST(name) List name = { .head = NULL };
    // Use:
    DECLARE_LIST(hello)
    
    // C3
    ... currently no corresponding functionality ...

### Stringingification

    #define CHECK(x) do { if (!x) abort(#x); } while(0)
    
    // C3
    macro fn check(#expr)
    {
       if (!#expr) abort($stringify(#expr));
    }

## Top level evaluation

Script languages, and also upcoming languages like *Jai*, usually have unbounded top level evaluation. The flexibility of this style of meta programming has a trade off in making the code more challenging to understand. 

In C3, top level compile time evaluation is limited to `$if` and `$switch` constructs + macros with constant expression evaluation. This makes the code easier to read, but at the cost of expressive power.

## Macro declarations

A macro is defined using `macro <name>(<parameters>)`. All user defined macros use the @ symbol.

The parameters have different sigils: `$` means compile time evaluated (constant expression or type). `#` indicates an expression that is not yet evaluated, but is bound to where it was defined. Finally `&` is used to *implicitly* pass a parameter by reference.
`@` is required on macros that use `#` and `&` parameters.

A basic swap:

    /**
     * @checked a = b, b = a
     */
    macro void @swap(&a, &b)
    {
        $typeof(a) temp = a;
        a = b;
        b = temp;
    }

This expands on usage like this:

    fn void test()
    {
        int a = 10;
        int b = 20;
        @swap(a, b);
    }
    // Equivalent to:
    fn void test()
    {
        int a = 10;
        int b = 20;
        {
            int __temp = a;
            a = b;
            b = __temp;
        }
    }

Note the necessary `&`. Here is an incorrect swap and what it would expand to:

    macro void badswap(a, b)
    {
        $typeof(a) temp = a;
        a = b;
        b = temp;
    }
    
    fn void test()
    {
        int a = 10;
        int b = 20;
        badswap(a, b);
    }
    // Equivalent to:
    fn void test()
    {
        int a = 10;
        int b = 20;
        {
            int __a = a;
            int __b = b;
            int __temp = __a;
            __a = __b;
            __b = __temp;
        }
    }

## Macro methods

Similar to regular *methods* a macro may also be associated with a particular type:

    struct Foo { ... }
    
    macro Foo.generate(Foo *foo) { ... }
    Foo f;
    f.generate();

## Capturing a trailing block

It is often useful for a macro to take a trailing compound statement as an argument. In C++ this pattern is usually expressed with a lambda, but in C3 this is completely inlined.

To accept a trailing block, `; @name(param1, ...)` is placed after declaring the regular macro parameters.

Here's an example to illustrate its use:

    /**
     * A macro looping through a list of values, executing the body once
     * every pass.
     *
     * @checked { int i = a.len; value2 = a[i]; }
     **/
    macro @foreach(a; @body(index, value))
    {
        for (int i = 0; i < a.len; i++)
        {
		    @body(i, a[i]);
        }
    }
    
    fn void test()
    {
        double[] a = { 1.0, 2.0, 3.0 };
        @foreach(a; int index, double value)
        {
            libc::printf("a[%d] = %f\n", index, value);
        }
    }
    
    // Expands to code similar to:
    fn void test()
    {
        int[] a = { 1, 2, 3 };
        {
            int[] __a = a;
            for (int __i = 0; i < __a.len; i++)
            {
                libc::printf("Value: %d, x2: %d\n", __value1, __value2);
            }
        }
    }


## Macros returning values

A macro may return a value, it is then considered an expression rather than a statement:

    macro square(x)
    {
        return x * x;
    }
    
    fn int getTheSquare(int x)
    {
        return square(x);
    }
    
    fn double getTheSquare2(double x)
    {
        return square(x);
    }

## Calling macros

It's perfectly fine for a macro to invoke another macro or itself.

    macro square(x) { return x * x; }
    
    macro squarePlusOne(x)
    {
        return square(x) + 1; // Expands to "return x * x + 1;"
    }

The maximum recursion depth is limited to the `macro-recursion-depth` build setting.


## Macro directives

Inside of a macro, we can use the compile time statements `$if`, `$for` and `$switch`. Macros may also be recursively invoked. As previously mentioned, `$if` and `$switch` may also be invoked on the top level.

### $if, $else and $elif

`$if (<const expr>):` takes a compile time constant value and evaluates it to true or false.

    macro foo($x, $y)
    {
        $if ($x > 3):
            $y += $x * $x;
        $else:
            $y += $x;
        $endif;    
    }
    
    const int FOO = 10;
    
    fn void test()
    {
        int a = 5;
        int b = 4;
        foo(1, a); // Allowed, expands to a += 1;
        // foo(b, a); // Error: b is not a compile time constant.
        foo(FOO, a); // Allowed, expands to a += FOO * FOO;
    }

### Loops using $foreach and $for

`$foreach (<range> : <variable>): ... $endforeach;` allows compile time recursion. `$foreach` may recurse over enums, struct fields or constant ranges. Everything must be known at compile time.


Compile time looping:

    macro foo($a)
    {
        $for (var $x = 0; $x < $a; $x++):
            libc::printf("%d\n", $x);     
        $endfor;
    }
    
    fn void test()
    {
        foo(2);
        // Expands to ->
        // libc::printf("%d\n", 0);     
        // libc::printf("%d\n", 1);         
    }

Looping over enums:

    macro foo_enum($SomeEnum)
    {
        $foreach ($x : $SomeEnum.elements):
            libc::printf("%d\n", (int)$x);     
        $endforeach;
    }
    
    enum MyEnum
    {
        A,
        B,
    }
    
    fn void test()
    {
        foo_enum(MyEnum);
        // Expands to ->
        // libc::printf("%d\n", (int)MyEnum.A);
        // libc::printf("%d\n", (int)MyEnum.B);    
    }

An important thing to note is that the content of the `$foreeach` or `$for` body must be a complete statement.
It's not possible to compile partial statements.

### Switching on type with $switch

It's possible to switch on type:

    macro void foo(a, b)
    {
        $switch(a, b):
            $case int, int: 
                return a * b;
        $endswitch;
        return a + b;
    }


## Conditional macros at the top level

A limitation with the macros is that they are only used *within* functions. 
This is deliberate – macros expanding at the top level are much harder to 
reason about since they should be able to define new types or change the
meaning of the code that follows.

Still, the usefulness of top level macros is great, which is why C3 offers 
three pieces of functionality for the top level: conditional compilation,
global constants and attributes


### Conditional compilation

Conditional compilation is done with `$if` and `$else`, which works just like
inside of functions.

    $if ($defined(platform::OS) && platform::OS == WIN32):
    
    fn void doSomethingWin32Specific()
    {
        /* .... */
    }
    
    $endif;

### Global constants

Global constant on the top level work like compile time variables in macros – with the exception
that they must always be declared constant. They are evaluated in order, but will resolve on-demand if needed.

