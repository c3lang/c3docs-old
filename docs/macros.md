# Macros

**WILL be subject to later revision!**

The macro capabilities of C3 reaches across several constructs: macros (prefixed with `@` at invocation), [generic functions](../generics/#generic-functions), [generic modules](../generics/#generic-modules), compile time variables (prefixed with `$e`), macro compile time execution (using `$if`, `$each`, `$switch`), attributes and incremental structs, enums and arrays.

## A quick comparison of C and C3 macros

### Conditional compilation

```
// C
#if defined(x) && y > 3
int z;
#endif

// C3
$if ($defined(x) && y > 3):
    int z;
$endif;
```

### Macros

```
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
macro ret!(x)
{
    return x;
}
define u32 = uint;

// Use:
int y = @m(foo() + 2);
@ret!(bar());
u32 b = y;
```

### Dynamic scoping

```
// C
#define Z() ptr->x->y->z
int z = Z();

// C3
macro z(implicit ptr)
{
    return ptr->x->y->z;
}
int z = @z();
```

### Reference arguments

```
// C
#define M(x, y) x = 2 * (y);

// C3
macro m(int &x, int y)
{
    x = 2 * y;
}
```

### First class types

```
// C
#define SIZE(T) (sizeof(T) + sizeof(int))

// C3
macro size($Type)
{
    return sizeof($Type) + sizeof(int);
}
```

### First class statements
```
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
macro for_each(list, macro void(it) @body)
{
    for (typeof(list) x = list; x; x = x->next) @body(x);
}
// Use:
@for_each(list; Foo* x)
{
  if (!process(x)) return;
}
```

### First class names

```
// C
#define offsetof(T, field) (size_t)(&((T*)0)->field)

// C3
macro offsetof($Type, $field)
{
    return (size_t)(&((T*)0)->field);
}
```

### Declaration attributes

```
// C
#define NONNULL(args...) __attribute__((nonnull(args)))

// C3
... currently no corresponding functionality ...
```

### Declaration macros

```
// C
#define DECLARE_LIST(name) List name = { .head = NULL };
// Use:
DECLARE_LIST(hello)

// C3
... currently no corresponding functionality ...
```

### Stringingification

```
#define DECLARE_STRING(name, s) char *name##_str = #s;

// C3
... currently no corresponding functionality ...
```


## Top level evaluation

Script languages, and also upcoming languages like *Jai*, usually have unbounded top level evaluation. The flexibility of this style of meta programming has a trade off in making the code more challenging to understand. 

In C3, top level compile time evaluation is limited to `$if` and `$switch` constructs. This makes the code easier to read by sacrificing flexibility. However, the incremental structs, enums and arrays provided by the compiler offers a way to build compile time structures without the necessity for full meta programming capabilities.

## Macro declarations

A macro is defined using `macro <name>(<parameters>)`. All user defined macros use the @ symbol.

The parameters have different sigils: `$` means compile time evaluated (captured variable, symbol or expression). Any parameters without sigils are passed by *value*, as if it was a normal function parameter.


A basic swap:

```
/**
 * @ensure parse(a = b), parse(b = a)
 */
macro void swap($a, $b)
{
    typeof(a) temp = a;
    a = b;
    b = temp;
}
```

This expands on usage like this:

```
func void test()
{
    int a = 10;
    int b = 20;
    @swap(a, b);
}
// Equivalent to:
func void test()
{
    int a = 10;
    int b = 20;
    {
        int __temp = a;
        a = b;
        b = __temp;
    }
}
```

Note the necessary `$`. Here is an incorrect swap and what it would expand to:

```
macro void badswap(auto a, auto b)
{
    typeof(a) temp = a;
    a = b;
    b = temp;
}

func void test()
{
    int a = 10;
    int b = 20;
    @badswap(a, b);
}
// Equivalent to:
func void test()
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
```

## Method macros

Similar to *method functions* a macro may also be associated with a particular type:

```
struct Foo { ... }

macro Foo.generate(Foo *foo) { ... }
Foo f;
@f.generate();
```

## Capturing a macro body

It is often useful for a macro to take a trailing compound statement as an argument. In C++ the pattern is usually expressed with a lambda, but in C3 this is completely inlined.

Any macro that takes a macro in the last position may be expressed as a trailing body instead. When the call puts the parameters for the trailing macro after `;`.

Here's an example to illustrate it the use:

```
/**
 * A macro looping through a list of values, executing the body once
 * every pass.
 *
 * @ensure parse(int i = a.len), parse(value2 = a[i])
 */
macro foreach(auto a, macro void(value, value) foo)
{
    for (int i = 0; i < a.len; i++)
    {
		@body(i, a[i]);
    }
}

func void test()
{
    double[] a = { 1.0, 2.0, 3.0 };
    @foreach(a; int index, double value)
    {
        printf("a[%d] = %f\n", index, value);
    }
}

// Expands to code similar to:
func void test()
{
    int[] a = { 1, 2, 3 };
    {
        int[] __a = a;
        for (int __i = 0; i < __a.len; i++)
        {
            printf("Value: %d, x2: %d\n", __value1, __value2);
        }
    }
}
```


## Macros returning values

A macro may return a value, it is then considered an expression rather than a statement:

```
macro auto square(auto x)
{
    return x * x;
}

int getTheSquare(int x)
{
    return square(x);
}

double getTheSquare2(double x)
{
    return square(x);
}
```

## Calling macros

It's perfectly fine for a macro to invoke another macro or itself.

```
macro @square(x) { return x * x; }

macro @squarePlusOne(x)
{
    return @square(x) + 1; // Expands to "return x * x + 1;"
}
```

The maximum recursion depth is limited to the `macro-recursion-depth` build setting.


## Macro directives

Inside of a macro, we can use the compile time statements `$if`, `$each` and `$switch`. Macros may also be recursively invoked. As previously mentioned, `$if` and `$switch` may also be invoked on the top level.

### $if, $else and $elif

`$if (<const expr>):` takes a compile time constant value and evaluates it to true or false.

```
macro @foo($x, $y)
{
    $if ($x > 3):
        $y += $x * $x;
    $else:
        $y += $x;
    $endif;    
}

const int FOO = 10;

func void test()
{
    int a = 5;
    int b = 4;
    @foo(1, a); // Allowed, expands to a += 1;
    // @foo(b, a); // Error: b is not a compile time constant.
    @foo(FOO, a); // Allowed, expands to a += FOO * FOO;
}
```

### Loops using $foreach

`$foreach (<range> as <variable>): ... $endforeach;` allows compile time recursion. `$each` may recurse over enums, struct fields or constant ranges. Everything must be known at compile time.


Looping over ranges:
```
macro @foo($a)
{
    $foreach (0..$a as $x):
        printf("%d\n", $x);     
    $endforeach;
}

func void test()
{
    @foo(2);
    // Expands to ->
    // printf("%d\n", 0);     
    // printf("%d\n", 1);         
}
```

Looping over enums:
```
macro @foo_enum($some_enum)
{
    $foreach($some_enum as $x):
        printf("%d\n", (int)($x));     
    $endforeach;
}

enum MyEnum
{
    A,
    B,
}

func void test()
{
    @foo_enum(MyEnum);
    // Expands to ->
    // printf("%d\n", (int)(MyEnum.A));
    // printf("%d\n", (int)(MyEnum.B));    
}
```

An important thing to note is that the content of the `$each` body must be a complete statement.
It's not possible to compile partial statements.

### Switching on type with $switch

It's possible to switch on type, similar to generic functions, but used internal to the macro:

```
macro void foo(a, b)
{
    $switch(a, b):
        $case int, int: 
            return a * b;
    endswitch;
    return a + b;
}
```

## Escape macros

Usually macro will generate its own scope, so that break, return, continue and next only stays valid inside of the macro's "scope". A `return` from inside a macro does not normally escape the scope into which it's called:

```
macro void @foo() { return; }

func void test()
{
    @foo(); // Doesn't do anything.
    printf("Test\n");
}
```

However, sometimes macros are needed that does not create its own scope, allowing return, break etc work as if it was part of the included scope. Escape macros does exactly that. An escape macro adds a "!" to the macro name. Note that this becomes part of the macro name.


```
macro void @foo!() 
{ 
    return; 
}

func void test()
{
    @foo!(); // The function returns here.
    printf("Test\n"); // Never printed!
}
```

This is not limited to return: `break`, `continue` and `next` is allowed.

```
macro next!($f) 
{ 
     nextcase $f;
}

func void test()
{
    int i = 1;
    switch (i)
    {
        case 1:
            @next!(3);
            printf("Foo\n");
        case 3:
            printf("Bar!\n")
    }
}
```

The above code will print "Bar!"

## Conditional macros at the top level

A limitation with the macros is that they are only used *within* functions. 
This is deliberate – macros expanding at the top level are much harder to 
reason about since they should be able to define new types or change the
meaning of the code that follows.

Still, the usefulness of top level macros is great, which is why C3 offers 
four pieces of functionality for the top level: conditional compilation,
global compile time varibles, attributes and incremental arrays.


### Conditional compilation

Conditional compilation is done with $if and $else, which works just like
inside of functions.

```
$if (@defined($os) && $os == 'WIN32'):

func void doSomethingWin32Specific()
{
    /* .... */
}

$endif;
```

### Global compile time variables

Variables on the top level work like compile time variables in macros – with the exception
that they must always be declared constant. They are always evaluated in order, which 
has to be taken into account when used in conjunction with @defined.

Consider this code:

```
macro @foo()
{
    $if ($defined(A)): 
        return A + 1; 
    endif;
    return 1;
}
const Z = @foo(); // Z = 1
const A = @foo(); // A = 1
const B = @foo(); // B = 2
```
