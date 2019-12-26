# Ideas

**WARNING** Unfinished ideas / brain dumps

## Module versioning

Follow Go Modules: 

1. Build file contains required library versions: e.g. 2.5.7
2. First number is major version number and is considered incompatible (basically a different module completely)
3. Dependency resolution per major library version is done by picking the minimal version. E.g. module Foo requires Bar 1.5+ and module Baz requires 1.3+. Our module using Bar and Baz will resolve the minimal version to 1.3. This is the version that will be used. Note that if Foo used 2.5+, then both Bar 2.5 and 1.3 would be required.

## Polymorphic functions

It would theoretically be possible to replace the generics with ad hoc polymorphic functions.


```
module adding(TypeA, TypeB, TypeC)

func TypeC add_module(TypeA a, TypeB b)
{
    return a + b;
}
```

```
generic func $c add_poly($a, $b)
{
    return $a + $b;
}
```


Usage:

```
// Module:
import adding;
define add_module(int, float, float) as int_float_add;
define add_module(int, double, double) as int_double_add;

func void test()
{
    float f = int_float_add(2, 3.0)
    float f2 = add_poly(2, 3.0);
    
    double d = int_double_add(2, 3.0);
    double d2 = add_poly(2, 3.0);
}
```

## Tests built in

Unit tests built in as an integral part of the language like D.

## Allow variable alias

Possibility to alias a variable name.

## Attribute to ensure alignment

`@assertalign` works as GCCs warn_if_not_aligned. On non packed structs, this will prevent compilation if padding is inserted in front of the member. On a packed struct, it will prevent compilation if it is not aligned.

## Binary include

The ability to include a binary file during compile time.

```
byte[] file = @binary_include("foo.dat");
```

## Compile time run-include

The ability to run a piece of code at compile time and include the result in the code.

```
@run_include("foo.sh", $some_param, "-x", $another_param);
```



## Macro text interpolation

For certain cases, pure text interpolation might be needed. Within macros, any text within ``` `` ``` can be evaluated and parsed.

```
macro void @foo($x, #f)
{
    `#f $x * $x`;
}

func void test()
{
    int x = 1;
    @foo(4, "x +=");
    // Expands to ->
    // x += 4 * 4;
}
```

Another example, showing the difference between `#var` and ``` `#var` ```:

```
macro void @foo2(#f)
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


   
## Implicit "this" in method functions

```
struct Foo
{
    int i;
}

func Foo.next(Foo*)
{
    i++;
}
```

## C interop

Steps:

1. Add group header in the builds: [c-libs]
2. Add the search paths: cpath = ["/xxx/foo/**", "/bar/headers/**"]
3. Add each library: [[lib]]

```
[c-libs]
    cpath = ["/xxx/foo/**", "/bar/headers/**"]
    [[lib]]
        module = "windows"
        header = "windows.h"
    [[lib]]
        module = "stdlib"
        header = "stdlib.h"
```



## Unsorted

## Halloc

Hierarchal memory allocation http://swapped.cc/#!/halloc support it?

##### Unsigned conversion to signed

Perhaps consider signed, rather than unsigned implicit conversion. Currently `ulong + long` becomes `ulong + @cast(ulong, long)` (according to C rules). Consider changing that to `@cast(long, ulong) + long`. That is, instead of "unsigned wins", use "signed wins".
 

##### Tagged any

A tagged pointer union type for any possible type.


##### Require explicit uninitialization

```
int a = ---;
int a = void; // Other possible variant
```

##### Extended "case"

Switch as "if-else"

```
switch (x) 
{
    case x > 200: 
        return 0;
    case x < 2: 
        small_x_warning();
    case 0:
        ....
    case x > y && a < 1:
        ...
}
```

##### Case as a range

```
switch (x) 
{
    case 1 .. 10: 
        ...
    case 11 .. 100:
        ...
}
```

##### Easy to get properties

* Endianness
* Register size
* Query what type of add is the fastest (wrapping, trapped) for the processor (with macros to select type)
* Query what type of overflow the processor supports

##### Associate properties to an enum

```
enum State [char* name, byte bit] int
{
    START("begin!", 0x01) = 0,
    END("end it!", 0x10)
}

funct void test()
{
    printf("%s\n", State.START.name); // Prints "begin!"
    printf("%d\n", State.END.bit); // Prints "16"
}
```

##### Tagged unions

```
tagged union Foo {
    int i;
    const char *c;
};

Foo foo;
foo.i = 3;
@istag(foo.i) // => true
@istag(foo.c) // => false
foo.c = "hello";
@istag(foo.i) // => false
@istag(foo.c) // => true

switch(@tag(foo)) 
{
    case Foo.i: 
        printf("Was %d\n", foo.i);
    case Foo.c: 
        printf("Was %s\n", foo.c);
}
```  

Alternative syntax etc:

```
struct Shape
{
    int centerX;
    int centerY;
    byte kind; // Implicit enum
    union (kind) 
    {
        SQUARE: { int side; }
        RECTANGLE: { int length, height; }
        CIRCLE: { int radius; }
    }
}
```

And another...

```
struct Shape
{
    int centerX;
    int centerY;
    byte kind; // Implicit enum
    union (kind) 
    {
        struct square 
        {
            int side;
        }    
        struct rectangle 
        {
            int length;
            int height;
        }
        struct circle
        {
            int radius;
        }
    }
}


```

And yet another...

```
struct Shape
{
    int centerX;
    int centerY;
    tagged union (kind)
    {
        case SQUARE:
            int side; 
        case RECTANGLE:
            int length;
            int height;
        case CIRCLE:
            int radius;            
    }
    byte kind;
}
```

## Interfaces

```
func void Foo.renderPage(Foo* foo, Page* info)
{
    /* ... */
}

interface Renderer
{
    void renderPage(Renderer* renderer, Page* info);
}

func void render(Renderer* renderer, Page* info)
{
    if (!rendered) return;
    renderer->renderPage(info);
}

func void test()
{
    Foo* foo = getFoo();
    Page* page = getPage(); 
    
    // Option 1
    Renderer.render(foo, page);
    
    // Option 2
    Renderer* renderer = foo;
    renderer.render(page);
}

// C equivalent: 
// struct RendererVtable { 
//     void (*renderPage)(void*, Page*); 
// };
// struct RendererRef { 
//     void* ref; RendererVTable *vtable; 
// };
// void Renderer__render(struct RendererRef renderer, Page *info) {
//     if (renderer.ref == null) return;
//     renderer.vtable->renderPage(renderer.ref, info);
// }
// 
// void test() {
//     Foo *foo = getFoo();
//     Page *page = getPage(); 
//
//     static RenderVTable FooRendererVTable = { &Foo__renderPage };
//     Renderer__render(struct RendererRef { foo, &FooRendererVTable }, page);
//    
//     struct RendererRef renderer = { foo, &FooRendererVTable };
//     Renderer__render(renderer, page);
// }
```

Possible syntax ideas:
```
func void test()
{
    Renderer* renderer1 = getFoo(); // Implicit

    Renderer* renderer2 = Renderer(getFoo());

    Renderer* renderer3 = getFoo() as Renderer;

    Renderer* renderer4 = @wrap(Renderer, getFoo()); // Macro

    Renderer renderer5 = getFoo(); // Look, no pointer!

    Renderer@ renderer6 = getFoo(); // Special pointer type?

    interface Renderer renderer7 = getFoo(); // Love long lines

    @Renderer renderer8 = getFoo();

    Renderer^ renderer9 = getFoo(); // Pascal    
    
    Renderer renderer10 = interface getFoo();

    Renderer renderer11 = interface(getFoo());

    Renderer renderer12 = virtual { getFoo() };
    
    virtual Renderer renderer13 = getFoo();
    
    virtual Renderer renderer14 = getFoo();
    
}

func void render(Renderer renderer, Page* info)
{
    renderer.renderPage(info);
}

virtual Renderer
{
    void renderPage(Page* info);
}

func void render(virtual Renderer* renderer, Page* info)
{
    renderer.renderPage(info);
}

func void render(virtual Renderer* renderer, Page* info)
{
    renderer->renderPage(info);
}

func void render(interface Renderer* renderer, Page* info)
{
    renderer->renderPage(info);
}


interface Renderer*[4] renderers;

interface Renderer[4] renderers;

virtual Renderer[4] renderers;

Renderer*[4] renderers;
```

## Built in maps

Same reasoing as arrays. Question about memory management is the same.

```
int[int] map; // Built-in maps
map[1] = 11;
// Retrieving a value
int i = try map[0]; // Requires a try

// Retrive or use default
int i = try map[12] else -1;

// Extend a map:
func bool int[int].remove_if_negative(int[int] *map, int index)
{
    if (try map[index] >= 0 else true) return false;    
    map.remove(index);
    return true;
}

// The underlying C function becomes:
// bool module_name__map_int_int__remove_if_negative(struct _map_int_int *map, int32_t index);
```

## Built in string type

Strings are built-in, refcounted(?) null-terminated character arrays.

Take a long hard look at memory management (here too)

```
string = "Hello";
string += " World";
char* data = &string; // Taking a pointer to the string, which may later be invalid.
```

## Built in managed pointers

Taking a hint from Cyclone, Rust etc one could consider managed pointers / objects. There are several possibilities:

1. Introduce something akin to move/borrow syntax with a special pointer type, eg. Foo@ x vs Foo* y and make the code track Foo@ to have unique ownership.
2. Introduce ref-counted objects with ref-counted pointers. Again use Foo@ x vs Foo* y with the latter being unretained. This should be internal refcounting to avoid any of the issues going from retained -> unretained that shared_ptr has. Consequently any struct that is RC:ed needs to be explicitly declared as such.
3. Managed pointers: you alloc and the pointer gets a unique address that will always be invalid after use. Any overflows will be detected, but use of managed pointers is slower due to redirect and check.


## Ideas around macros

Just some previous thoughts and ideas I've considered. Many of those actually go against the current design.

### Compile time variables


This is a variant of what already exists in C, but in a syntactically more friendly way.

For example this would be ok:

```
macro swap(a, b) 
{
    $x = typeof(a);
    static_assert(typeof(b) == $x);
    $x temp = a;
    a = b;
    b = a;
}
```

The example above is a bit contrived as in the above example we could simply have:

```
macro swap(a, b) 
{
    static_assert(typeof(b) == typeof(b));
    typeof(a) temp = a;
    a = b;
    b = a;
}
```

But still, it serves as an example on how to use it.

### Capturing trailing compound statement

```
public macro foreach(thelist, @body(Element *) ) 
{ 
    Element* iname = thelist.first;
    while (iname != nil) 
    {
        @body(iname);
        iname = iname.next;
    }
}
```

Or a version that is more flexible:

```
public macro foreach(thelist, @body(typeof(thelist.first)) ) 
{ 
    typeof(thelist.first) iname = thelist.first;
    while (iname != nil) 
    {
        @body(iname);
        iname = iname.next;
    }
}

// Usage:

foreach(list, Element *i) // <- Note type declaration!
{ 
    i.print();
}
```

Since type is going to appear very often, we could make a shortcut for it, like $@ as prefix meaning "typeof".

We then get

```
public macro foreach(thelist, @body($@thelist.first)) 
{ 
    $@thelist.first iname = thelist.first;
    while (iname != nil) 
    {
        @body(iname);
        iname = iname.next;
    }
}
```

Possibly one could even write the code like this:

```
public macro foreach(thelist, @body($element_type) ) 
{ 
    $element_type iname = thelist.first;
    while (iname != nil) {
        @body(iname);
        iname = iname.next;
    }
}
```

In this case `$element_type` works like "auto", but is also assigned the type, which then can be referred to in the signature.

### Yet another way to do macros :D

This was an older attempt...

First, we have to consider macros as always expanding where they are referenced to keep it simple.

```
macro @foo(int v) 
{
    v++;
    if (v > 10) return 10;
    return v;
}

func void test()
{
    int a = 10;
    @foo(a);
}
```

This code would then be exactly equal to:

```
func void test()
{
    int a = 10;
    a++;
    if (a > 10) return 10;
}
```

Macros simply expand in place.

Secondly, we can have macros returning values:

```
macro int @foo2(int v, int w) 
{
    v++;
    if (v > 10) return 10;
    w += 3;
    return 0;
}

func void test()
{
    int d = 0;
    int a = 10;
    int b = @foo(a, d);
}
```

This expands to:

```
func void test()
{
    int d = 0;
    int a = 10;
    a++
    int b;
    if (a > 10) 
    {
        b = 10;
    } 
    else 
    {
        b = 0;
        d += 3;
    }
}
```

Note that I'm using a sigil to indicate the code expansion to make macros more obvious.

We can allow the macro to take a body (here I'm calling the type of the body "{}")

```
macro int @foo3(int a, {} body)
{
    while (a > 10) 
    {
        a--;
        @body(); // Like a macro!
    }
}

func void test()
{
    b = 20;
    @foo3(b) {
        print(b);
    }
}
```

We expand this to:

```
func void test()
{
    b = 20;
    while (b > 10) {
        b--;
        print(b);
    }
}
```

#### Stepwise from C macros into C3 macros

```
#define ADD_TO(x, y) {
    x += y;
}

ADD_TO(x, 1)
```

The { } introduces a multiline macro that does not need explicit linebreaks.

No, add the "$" symbol to introduce hygienic temporaries:

```
#define SWAP(x, y) {
    typeof(x) $tmp = x;
    x = y;
    y = $tmp;
}
```

Here $tmp will actually be replaced by `__<macro>_<variable_name>_<instance>` when translating to C, so `__SWAP_tmp_1`, `__SWAP_tmp_2` etc.

We then introduce the syntax macros using:

```
macro swap(&a, &b) 
{
    typeof(a) $tmp = a;
    b = a;
    a = $tmp;
}
```

(Note the different use of $ here as compared to previous macro ideas where $ is a compile time evaluated variable!)


The use of &a follows C++ standard: it simply refers to a variable OR EXPRESSION that is imported into its scope. Using the unadorned variable name as evaluated expression allows us to write this code:

```
macro max(a, b) 
{
    return (a > b ? a : b)
}
```

The above code is equivalent to:

```
macro max(&a, &b) 
{
    typeof(a) $tmp_a = a;
    typeof(b) $tmp_b = b;
    $tmp_a > $tmp_b ? $tmp_a : $tmp_b
}
```

Or in (GNU) C:

```
#define max(a,b) \
   ({ __typeof__ (a) _a = (a); \
       __typeof__ (b) _b = (b); \
     _a > _b ? _a : _b; })
```

To recap:

1. Add the { } format to #define for multiline defines.
2. Add `$<name>` format as hygienic variable names.
3. Add the syntax "macro" type of definition.
4. The syntax macro makes a difference between "normal" parameters (with & as prefix) and "evaluated" parameters (unadorned variables)

Important is also to scope the macros:

`#define` is always defined local to a scope (unlike in C). 

This means that 

```
#define FOO printf("foo");
{
   #define BAR printf("bar");
}
FOO // adds printf("foo");
BAR; // Error, define not available in scope;
```

This also means that a define can be declared public to be accessed as if defined from the top of the file scope:

```
// file 1
module foo
public #define FOO { printf("FOO!\n"); }

// file 2
import foo

func void test() {
  foo.FOO
}
```

Only defines in the file scope that exists in the file scope may be public and used in other modules.


### Yet another macro proposal I found

Simple macros
```
macro @foo(&b)
{
    b++;
}

func test()
{
    int x = 1;
    @foo(x); 
}

// Same as:
func void test()
{
    int x = 1;
    x++;	
}

```

Macro with compile time values:
```
macro @foo($x, &b)
{
    b += $x;
}

func void test()
{
    int x = 1;
    @foo(10, x);
}

// Expands to:
func void test()
{
    int x = 1;
    x += 10;
}
```

Macro with string capture

```
macro @foo($x, #f)
{
    `#f $x * $x`;
}

func void test()
{
    i32 x = 1;
    @foo(4, "x += ");
}

// Expands to
func void test()
{
    i32 x = 1;
    x += 4 * 4;	
}

macro @foo2(#f)
{
    printf("%s was %d\n", #f, `#f`);
}

funct void test2()
{
    i32 x = 1;
    @foo2("x");
}

// Expands to
funct void test2()
{
    i32 x = 1;
    printf("%s was %d\n", "x", x);
}
```

Macro with conditional compile time values

```
macro @foo($x, &b)
{
    $IF ($x > 3) 
    {
        b += $x * $x;
    } 
    $ELSE
    {
        b += $x;
    }
}

func void test()
{
    i32 x = 1;
    
    @foo(10, x);
    @foo(2, x);
}

// Expands to
func void test()
{
    i32 x = 1;
    
    x += 100;
    x += 2;
}

```


Nested macros (don't do this, but an example)

```
macro @foo($a)
{
    printf("%d\n", $a);
    $IF($a > 0) 
    {
        @foo($a - 1);
    }
}

func void test()
{
    @foo(2);
}

// Expands to
func void test()
{
    printf("%d\n", 2);
    printf("%d\n", 1);
    printf("%d\n", 0);
}
```

The above suggests macro sugar of loops:

```
macro @foo($a)
{
    $EACH(0..$a AS $x) 
    {
        printf("%d\n", $x);		
    }
}

macro @foo_enum(&a)
{
    $EACH(a AS $x)  
    {
        printf("%d\n", @cast(int, $x));		
    }
}

enum MyEnum
{
    A,
    B,
    FOO
}

func void test()
{
    @foo_enum(MyEnum);
}

// Expands to
func void test()
{
	printf("%d\n", @cast(int, A));
	printf("%d\n", @cast(int, B));
	printf("%d\n", @cast(int, FOO));
}
```

Each may iterate over: struct members (returned as string), enums (returned as the enum value)


Type group helper

```
macro @foo(int& a) // Macro only valid for a is any type of signed integer
macro @foo(integer& a) // Valid for unsigned and signed integers
macro @foo(number& a) // Valid for any float or integer
```
	
Macros may force non local return

```
macro @foo()		
{
	exit 1; 
    // other keyword? 'escape'? I think exit is good, 
    // but clashes with function name!
}

func int test()
{
    @foo();
}

// expands to
func int test()
{
    return 1;
}
```

Normal return creates a statement expression

```
macro @foo(&a)
{
    int x = a;
    x++;
    if (x > 0) return 1;
    return 2; 
}

func int test()
{
    b = 10;
    int y = @foo(b);
}

// Expands to:
func int test()
{
    b = 10;
    int __macro_ret_1;
    do
    {
        int __macro_x = b;
        __macro_x++;
        if (__macro_x > 0) 
        {
            __macro_ret_1 = 1;
            break;
        } 
        else 
        {
            __macro_ret_1 = 2;
            break;
        }
    } 
    while (0);
    int y = __macro_ret_1;
}
```

Bodies in macros

```
macro @foo(&a, @body)
{
    int z = 0;
    while (a < 10) 
    {
        @body();
        z++;	
    } 
}

func void test()
{
    int i = 0;
    @foo(i) 
    {
        i += 1;
    }
}

// Expands to
func void test()
{
    int i = 0;
    {
        int __macro_z = 0;
        while (i < 10) 
        {
            i += 1;
            __macro_z++;
        }
    }
}
```

Bodies in macros with parameters

```
macro @foo(&a, @body(&x, $y))
{
    int z = 0;
    while (a < 10) 
    {
        @body(z, 2);
        z++;	
    } 
}

func void test()
{
    int i = 0;
    @foo(i) 
    {
        printf("%d / %d\n", x, y);
    }
}

// Expands to

func void test()
{
    int i = 0;
    {
        int __macro_z = 0;
        while (i < 10) {
            printf("%d / %d\n", __macro_z, 2);
            __macro_z++;
        }
    }
}
```

Expression is extended to parse:

MACRO_IDENT => lookup $x in current macro scope and replace it with literal. Error if not in macro.
MACRO_REPLACEMENT => invoke lexer on code inside, after doing a replace of any `#` inside. 

`$IF` requires that the expression can be evaluated to a constant value, similar holds for the range in `$EACH`.

The general rule:
1. An argument prefixed with `$` is always something that must be constant.
2. An argument prefixed with `&` is always a reference to an outer variable.
3. An argument prefixed with `#` always matches a string. It will be expanded when lexed in \`\` statements
4. A $ variable can be converted to a `#` variable.
5. A `#` can be evaluated to a `$`
6. $ and `#` cannot be assigned to, they are always constant.
7. `$`, `&` and `#` will never shadow variables from the outer scope.
	
