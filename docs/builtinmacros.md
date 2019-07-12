# Built-in macros

There are several built-in macros to inspect the code during compile and runtime.

- @bitcast
- @cast
- @describe
- @elements
- @name
- @sizeof
- @typeof

### @bitcast

Bitcast reinterprets the contents of a variable without any conversion. Bitsize must be the same for both types.

```
float f = 12.3;
int i = @bitcast(int, f);
printf("%x", i); // Prints 4144cccd
```

### @cast

Cast one type to the other using the default conversion rules. Bit extension or truncation may occur.
```
float f = 12.3;
int i = @cast(int, f);
printf("%d", i); // Prints 12
```

### @describe

Creates a runtime description of a value.

```
type FooEnum enum 
{
    A,
    B
}

func char[] test()
{
    FooEnum x = FooEnum.A;
    return @describe(x); // Returns "FooEnum.A"
}

type Foo struct
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
    return @name(x); // Returns "A"
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