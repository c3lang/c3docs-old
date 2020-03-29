# Macros

**WILL be subject to later revision!**

The macro capabilities of C3 reaches across several constructs: macros (prefixed with `@` at invocation), [generic functions](../generics/#generic-functions), [generic modules](../generics/#generic-modules), compile time variables (prefixed with `$e`), macro compile time execution (using `$if`, `$each`, `$switch`), attributes and incremental structs, enums and arrays.

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
macro @square(x) { return x * x }

macro @squarePlusOne(x)
{
    return @square(x) + 1; // Expands to "return x * x + 1;"
}
```

The maximum recursion depth is limited to the `macro-recursion-depth` build setting.


## Macro directives

Inside of a macro, we can use the compile time statements `$if`, `$each` and `$switch`. Macros may also be recursively invoked. As previously mentioned, `$if` and `$switch` may also be invoked on the top level.

### $if, $else and $elif

`$if (<const expr>)` takes a compile time constant value and evaluates it to true or false.

```
macro @foo($x, $y)
{
    $if ($x > 3)
    {
        $y += $x * $x;
    } 
    $else
    {
        $y += $x;
    }
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

There is an alternative form of `$if`/`$else`/`$elif` using `:` and terminated
with `$endif`. These have by convention no indentation.

This is mostly useful for top level use where indentation might not be desired.

```
macro @foo($X, $y)
{

$if ($x > 3):

    $y += $x * $x;

$elif ($x < 100):
    
    $y += -$x;

$else:
        
    $y += $x;

$endif;

}
```  

### Loops using $each

`$each (<range> as <variable>) { ... }` allows compile time recursion. `$each` may recurse over enums, struct fields or constant ranges. Everything must be known at compile time.


Looping over ranges:
```
macro @foo($a)
{
    $each (0..$a as $x) 
    {
        printf("%d\n", $x);     
    }
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
    $each($some_enum as $x)  
    {
        printf("%d\n", cast($x, int));     
    }
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
    // printf("%d\n", cast(MyEnum.A, int));
    // printf("%d\n", cast(MyEnum.B, int));    
}
```

An important thing to note is that the content of the `$each` body must be a complete statement.
It's not possible to compile partial statements.

### Switching on type with $switch

It's possible to switch on type, similar to generic functions, but used internal to the macro:

```
macro void foo(a, b)
{
    $switch(a, b)
    {
        $case int, int: 
            return a * b;
    }
    return a + b;
}
```

## Escape macros

Usually macro will generate its own scope, so that break, return, continue and goto only stays valid inside of the macro's "scope". A `return` from inside a macro does not normally escape the scope into which it's called:

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

This is not limited to return: `break`, `continue` and even `goto` is allowed.

```
macro @goto!($f) 
{ 
     goto $f;
}

func void test()
{
    @goto!(test)

    printf("Foo!\n")    
    return;
    
    test:
    printf("Bar!\n")
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
$if (@defined($os) && $os == 'WIN32')

func void doSomethingWin32Specific()
{
    /* .... */
}

$endif
```

### Global compile time variables

Variables on the top level work like compile time variables in macros – with the exception
that they must always be declared constant. They are always evaluated in order, which 
has to be taken into account when used in conjunction with @defined.

Consider this code:

```
macro @foo()
{
    $if (@defined($a)) return $a + 1;
    return 1;
}
const $z = @foo(); // $z = 1
const $a = @foo(); // $a = 1
const $b = @foo(); // $b = 2
```

### Attributes

Attributes are tags placed on functions, types and variables. 
They may take a type as argument, much like a function. It's possible
iterate over all the objects that have an attribute of a particular type.


```
attribute func @myvar(int i = 0);
attribute struct, union, enum @foo;

func void test() @myvar(2)
{
    /** ... */
}

struct Test @foo
{
    int i;
}
```

What's useful about attributes is that they can be accessed during compile
time:

```
macro @fooCheck($a)
{
    $if (@defined($a.@foo))
    {
        return "Was fooed";
    }
    return "Ok";
}

struct TestA
{
    int i;
}

struct TestB @Foo
{
    float f;
}

func void test()
{
    printf("Check TestA: " @fooCheck(TestA) "\n");    
    printf("Check TestB: " @fooCheck(TestB) "\n");
    // Prints:
    // Check TestA: Ok
    // Check TestB: Was fooed
}
```

It's also possible to get hold of the values:

```
attribute func @myvar(int i = 0);
func void test() @myvar(200) { ... }

func void test()
{
    printf("%d", test.@myvar.i); // Prints 200
}
```

### Incremental arrays

Incremental arrays allows compile time arrays to be constructed piecemeal within a single source file. An incremental array uses the `[+]` ending, but will be considered to be a fixed size array for all other purposes.
Append to an incremental array using `+=` which done during compile time.

```
int[+] a = { 1 };

a += 1;

/* ... other code .. */

a += 2;

// Equivalent to the declaration int[3] a = { 1, 1, 2 };
```

This can be especially useful in conjuction with `$each`:

```
enum MyEnum { A, B }

macro type @foo_enum($theEnum)
{
    string[+] arr = {};
    $each($theEnum AS $x)  
    {
        arr += @name($x);     
    }
}

// allMyEnum will contain { "A", "B" }
const string[] allMyEnum = @foo_enum(MyEnum);
```

Here is a similar example but for attributes:

```
attribute struct @special;

struct TestA @special { int i; }
struct TestB { float f; }
struct TestC @special { float f; }

macro void @specialStructs()
{
    string[+] res = {};
    $each(@special as $x)
    {
        res += @name($x);
    }
    // The above expands to:
    // res += "TestA";
    // res += "TestC";    
    return res;
}

// SPECIAL_STRUCTS = { "TestA", "TestB" }
const string[] SPECIAL_STRUCTS = @specialStructs();
```
