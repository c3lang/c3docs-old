# Macros

**!!!THIS IS JUST AN EXPERIMENTAL DRAFT!!!**

Macros is a form of compile time evaluation. There are compile time variables (prefixed with `$`) and compile time template code (macros).

## Compile time evaluation

The C3 compiler always does a first pass through the code reading all definitions and resolving compile time variables. Compile time variables with the `$` prefix are resolved *in order of declaration*.

## Macro declarations

A macro is defined using `macro @<name>(<parameters>)`. All user defined macros use the @ symbol.

The parameters have different sigils: `$` means compile time evaluated (captured variable, symbol or expression). `#` means full string capture. Any parameters without sigils are passed by *value*, as if it was a normal function parameter.


A basic swap:

```
/**
 * @ensure parse($a = $b), parse($b = $a)
 */
macro @swap($a, $b)
{
    typeof($a) temp = $a;
    $a = $b;
    $b = temp;
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
macro @badswap(a, b)
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

## Capturing a macro body

We may add a trailing parameter in the format: `: @<param name>(arg1, arg2, ...)` it then captures any trailing compound statement.

```
/**
 * @ensure parse(int i = a.len), parse(value1 = a[i])
 * @ensure parse(value2 = 2 * value1)
 */
macro @foreach(a : @body(value1, value2) )
{
    for (int i = 0; i < a.len; i++)
    {
        @body(a[i], a[i] * 2);
    }
}

func void test()
{
    int[] a = { 1, 2, 3 };
    @foreach(a : int value, int mult)
    {
        printf("Value: %d, x2: %d\n", value, mult);
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
            typeof(__a[__i]) __value1 = __a[__i];
            typeof(__a[__i] * 2) __value2 = __a[__i] * 2;
            printf("Value: %d, x2: %d\n", __value1, __value2);
        }
    }
}
```

## Macros returning values

A macro may return a value, it is then considered an expression rather than a statement:

```
macro square(x)
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

Obviously this can cause infinite recursion, so the actual recursion depth is limited to what the build setting `macro-recursion-depth` is set to.


## Macro directives

Inside of a macro, we can use the compile time statements `$if`, `$else`‚ `$each` and `$switch`. Macros may also be recursively invoked.

### $if, $else and $elif

`$if (<const expr>)` takes a compile time constant value and evaluates it to true or false.

```
macro @foo($X, $y)
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
macro @foo($A)
{
    $each (0..$A as $x) 
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
macro @foo_enum($THE_ENUM)
{
    $each($THE_ENUM as $x)  
    {
        printf("%d\n", cast(int, $x));     
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
    // printf("%d\n", cast(int, MyEnum.A));
    // printf("%d\n", cast(int, MyEnum.B));    
}
```

An important thing to note is that the content of the `$each` body must be a complete statement.
It's not possible to compile partial statements.

### Switching on type with $switch

It's possible to switch on type, similar to generic functions, but used internal to the macro:

```
macro foo(a, b)
{
    $switch(a, b)
    {
        $case int, int: 
            return a * b;
    }
    return a + b;
}
```

## Macro text interpolation

For certain cases, pure text interpolation might be needed. The sigil `#` is used to indicate a stringified capture. Within a macro, any text within ``` `` ``` is evaluated as text.

```
macro @foo($x, #f)
{
    `#f $x * $x`;
}

func void test()
{
    int x = 1;
    @foo(4, x += );
    // Expands to ->
    // x += 4 * 4;
}
```

Another example, showing the difference between `#var` and ``` `#var` ```:

```
macro @foo2(#f)
{
    printf("%s was %d\n", #f, `#f`);
}

funct void test2()
{
    int x = 1;
    @foo2(x);
    // Expands to ->
    // printf("%s was %d\n", "x", x);
}
```

### Compile time string functions

In order to facilitate certain types of macros, the following macros are built in:

- `@strToUpper(#f)` Convert string to upper case.
- `@strToLower(#f)` Convert string to lower case.
- `@strToVarName(#f, #space)` Convert string to camel case from a space-based name scheme.
- `@strToTypeName(#f, #space)` Convert string to title case from a space-based name scheme.
- `@strFromName(#f, #space)` Convert title case or lower camel case to a space based scheme.
- `@strReplace(#str, #pattern, #replacement, #count)` Replace a string with another.
- `@subString(#str, #start, #length)` Return a substring of a compile time string.
- `@strFind(#str, #stringToFind)` Find a substring in a compile time string.
- `@strHash(#str)` Return the FNV1a hash of a string.
- `@strLen(#str)` Return a compile time length of a string.
- `@stringify(#str)` Escapes a string so it becomes a valid string.

## Escape macros

A usual macro will generate its own scope, so that break, return, continue and goto only stays valid inside of its own "scope". A `return` from inside a macro does not normally escape the scope into which it's called:

```
macro @foo() { return; }

func void test()
{
    @foo(); // Doesn't do anything.
    printf("Test\n");
}
```

However, there is a way to make a macro break out of the outer scope, and that is to add a `!` to the macro name. Such a macro may escape its boundaries.

```
macro @foo!() 
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
global compile time varibles, decorators and incremental arrays.


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

### Decorators

Decorators are attributes placed on functions, types and variables. 
They may take a type as argument, much like a function. It's possible
iterate over all decorated symbols of a particular type.


```
decorator func @myvar(int i = 0);
decorator struct, union, enum @foo;

func void test() @myvar(2)
{
    /** ... */
}

struct Test @foo
{
    int i;
}
```

What's useful about decorators is that they can be accessed during compile
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
decorator func @myvar(int i = 0);
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

macro @foo_enum($theEnum)
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

Here is a similar example but for decorators:

```
decorator struct @special;

struct TestA @special { int i; }
struct TestB { float f; }
struct TestC @special { float f; }

macro @specialStructs()
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
