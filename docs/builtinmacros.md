# Compile time functions

*This has changed and needs a revision.*

There are several built-in macros to inspect the code during compile and runtime.

- `$nameof`
- `$qname`
- `$extname`
- `$sizeof`
- `$typeof`


### @describe

Creates a runtime description of a value.

```
enum FooEnum
{
    A,
    B
}

func char[] test()
{
    FooEnum x = FooEnum.A;
    return @describe(x); // Returns "FooEnum.A"
}

struct Foo
{
    int x;
    char[] y;
}

func char[] test_struct()
{
    Foo foo = { .x = 2, y = "bar" }
    return @describe(x); // Returns "Foo { x: 2, y: "bar" }"
}
```

### @elements

Returns an array of an error set or enum set.

```
FooEnum[] values = @elements(FooEnum); 
// Same as FooEnum[] values = { FooEnum.A, FooEnum.B };
```

### @name

Creates a runtime name of a value.

```
func char[] test()
{
    FooEnum x = FooEnum.B;
    return @name(x); // Returns "B"
}

func char[] test_struct()
{
    Foo foo = { .x = 2, y = "bar" }
    return @name(x); // Returns "Foo"
}
```

### @offset
Returns the byte offset in a structure, like `offsetof` in C

```
@offset(Foo, y); // => returns 4 or 8
```

### @sizeof

Returns the size of a type or an expression

```
func usize size_test()
{
    return @sizeof(Foo); // Might compile to 16
}

func usize size_test2()
{
    return @sizeof(size_test()); // Usually returns 8 (the size of usize)
}
```

### @typeof

Returns the type of an expression

```
@typeof(size_test()) x = size_test();
@typeof(x) y = x + 1;
```