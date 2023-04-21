# Ideas

**WARNING** Unfinished ideas / brain dumps

## Module versioning


Go Modules: 

1. Build file contains required library versions: e.g. 2.5.7
2. First number is major version number and is considered incompatible (basically a different module completely)
3. Dependency resolution per major library version is done by picking the minimal version. E.g. module Foo requires Bar 1.5+ and module Baz requires 1.3+. Our module using Bar and Baz will resolve the minimal version to 1.5. This is the version that will be used. Note that if Foo used 2.5+, then both Bar 2.5 and 1.3 would be required.

Possible design:
1. Dependency contains no-version, major version, minor version, major + minor + build or list of major versions
2. Try to resolve the major version to use. If none is selected, pick max. More than one are required -> compile error 
(example: A needs 1, 2, B needs 2, 3 => pick 2, A needs 1, 2, B needs 3, 4 => error)
3. Pick the maximum minor + build version available. If this is less than minimum minor + build version needed => error.
4. Excluding versions should be possible, eg exclude 1.3.4 

## Generic as keyword for polymorphic functions

For macros that essentially are polymorphic functions, we could use the keyword "generic" instead:

```c
macro swap(&a, &b) { ... } // Cannot be generic, captures are not allowed.
macro max(a, b) { ... } // Can be generic
macro doSomething(a, #a) { ... } // Cannot be generic, uses unevaluated expressions
macro malloc($Type) { ... } // Can be generic.

// Example of "max":
generic max(a, b)
{
    return a > b ? b : a;
}
// Example of "malloc"
generic malloc($Type)
{
    _builtin_malloc($Type.sizeof);
}
```

Invocation changes:
```c
int max_value = @max(foo(), bar()); // macro
int max_value = max(foo(), bar()); // generic
```

This would furthermore remove the "!" addition for escaping macros, adding it as an attribute:

```c
macro returnme() @escaping
{
    return;
}
```

Macros could then also more natural invoke defer:

```c
macro deferclose() @scopeless
{
    defer close();    
}
```

Both would be invoked:

```c
@deferclose();
@returnme();
```

It would be possible to define a generic, such a generic could be taken the address of:

```c
#define intmax = max(int, int);
#define TwoIntfn = fn int(int, int);
TwoIntFunc x = &intmax;
```

## Associative array literals

_Syntax and implementation under consideration!_

Associative arrays are mappings between keys and values:

```
IntStringMap x = { "a": 2, "b": { "3" : "foo" } };
```

To use an associative array, the struct needs to define
the following generics: `_map_init()` and `_map_init_add`. In the example above, the actual code is compiled to:

```
IntStringMap x;
_map_init(&x);
_map_init_add(&x, "a", 2);
IntStringMap _temp;
_map_init(&_temp);
_map_init_add(&x, "3", "foo");
_map_init_add(&x, "b", _temp);
```


## Array literals

Arrays can be initialized using compound literals, but there is also a special format for array initialization using `[]`. In this case, the type is inferred. It allows uniform initialization of all types of arrays:

```
int[3] y = [1, 2, 3]; // Fixed array allocated on the stack
int[] z = [1, 2, 3]; // Slice pointing at literal allocated on the stack
```

It offers some convenience when calling functions taking arrays:

```
fn void test1(int[3] x) { ... }
fn void test2(int[] y) { ... }
fn void test2(int[3]* z) { ... }

test1([ 1, 2, 3 ]);
test2([ 1, 2, 3 ]);
test3([ 1, 2, 3 ]);
```


## Allow slicing of user defined containers

```c
// Supporting the full set.
macro Foo._slice(Foo* foo, a, b, $a_from_end, $b_from_end)
{
    $if ($a_from_end)
        $if ($b_from_end)
            return foo.values[^a..^b];
        $endif
        return foo.values[^a..b];
    $endif    
    $if ($b_from_end)
        return foo.values[a..^b];
    $endif
    return foo.values[a..b];    
}

// Not supporting reverse
macro Bar._slice(Bar *bar, a, b, $a_from_end, $b_from_end)
{
    $assert(!$a_from_end && !$b_from_end, "Slicing from the end is not possible for Bar types.");
    return bar.valyes[a..b];
}
```

## Allow narrowing conversions for floats

Narrowing conversions for double -> float are common and might not be sufficiently important to do explicitly.

## Attribute to ensure alignment

`@assertalign` works as GCCs warn_if_not_aligned. On non packed structs, this will prevent compilation if padding is inserted in front of the member. On a packed struct, it will prevent compilation if it is not aligned.

## Binary include

The ability to include a binary file during compile time.

An additional `embed-path` gives search dirs.

```
byte[] file = @binary_include("foo.dat");
```

Limiting embed to x bytes:

```
byte[] file = @binary_include("/dev/urandomg", 16);
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

fn void test()
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
    io::printf("%s was %d\n", #f, `#f`);
}

funct void test2()
{
    int x = 1;
    @foo2(x);
    // Expands to ->
    // io::printf("%s was %d\n", "x", x);
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

fn Foo.next(Foo*)
{
    i++;
}
```



## Unsorted

##### Tagged any

A tagged pointer union type for any possible type.




##### Easy to get properties

* Endianness
* Register size
* Query what type of add is the fastest (wrapping, trapped) for the processor (with macros to select type)
* Query what type of overflow the processor supports


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
        io::printf("Was %d\n", foo.i);
    case Foo.c: 
        io::printf("Was %s\n", foo.c);
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





## Built in maps

Same reasoning as arrays. Question about memory management is the same.

```
int[int] map; // Built-in maps
map[1] = 11;
// Retrieving a value
int i = map[0]!!; // Requires a rethrow

// Retrive or use default
int i = try map[12] else -1;

// Extend a map:
fn bool int[int].remove_if_negative(int[int] *map, int index)
{
    if (try map[index] >= 0 else true) return false;    
    map.remove(index);
    return true;
}

// The underlying C function becomes:
// bool module_name__map_int_int__remove_if_negative(struct _map_int_int *map, int32_t index);
```



## Built in managed pointers

Taking a hint from Cyclone, Rust etc one could consider managed pointers / objects. There are several possibilities:

1. Introduce something akin to move/borrow syntax with a special pointer type, eg. Foo@ x vs Foo* y and make the code track Foo@ to have unique ownership.
2. Introduce ref-counted objects with ref-counted pointers. Again use Foo@ x vs Foo* y with the latter being unretained. This should be internal refcounting to avoid any of the issues going from retained -> unretained that shared_ptr has. Consequently any struct that is RC:ed needs to be explicitly declared as such.
3. Managed pointers: you alloc and the pointer gets a unique address that will always be invalid after use. Any overflows will be detected, but use of managed pointers is slower due to redirect and check.
