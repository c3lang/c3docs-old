#####if-statement
```
fn void if_example(int a) 
{
    if (a > 0) 
    {
        // ..
    } 
    else 
    {
        // ..
    }
}
```

#####for-loop
```
fn void example_for() 
{
    // the for-loop is the same as C99. 
    for (int i = 0; i < 10; i++) 
    {
        io::printf("%d\n", i);
    }

    // also equal
    for (;;) 
    {
        // ..
    }
}
```

#####foreach-loop
```
fn void example_foreach(float[] values) 
{
    foreach (index, value : values) 
    {
        io::printf("%d: %f\n", index, value);
    }
}
```


#####while-loop

```
fn void example_while() 
{
    // again exactly the same as C
    int a = 10;
    while (a > 0) 
    {
        a--;
    }

    // Declaration 
    while (Point* p = getPoint()) 
    {
        // ..
    }
}
```

#####enum + switch

Switches have implicit break and scope. Use "nextcase" to implicitly fallthrough or use comma:

```
enum Height : uint
{
    LOW = 0,
    MEDIUM,
    HIGH,
}

fn void demo_enum(Height h)
{
    switch (h)
    {
        case LOW:
        case MEDIUM:
            io::println("Not high");
            // Implicit break.
        case HIGH:
            io::println("High");
    }

    // This also works
    switch (h)
    {
        case LOW:
        case MEDIUM:
            io::println("Not high");
            // Implicit break.
        case Height.HIGH:
            io::println("High");
    }

    // Completely empty cases are not allowed.
    switch (h)
    {
        case LOW:
            break; // Explicit break required, since switches can't be empty.
        case MEDIUM:
            io::println("Medium");
        case HIGH:
            break;
    }

    // special checking of switching on enum types
    switch (h)
    {
        case LOW:
        case MEDIUM:
        case HIGH:
            break;
        default:    // warning: default label in switch which covers all enumeration value
            break;
    }

    // Using "nextcase" will fallthrough to the next case statement,
    // and each case statement starts its own scope.
    switch (h)
    {
        case LOW:
            int a = 1;
            io::println("A");
            nextcase;
        case MEDIUM:
            int a = 2;
            io::println("B");
            nextcase;
        case HIGH:
            // a is not defined here
            io::println("C");
    }
}
```


Enums are always namespaced.

Enums also define `.min` and `.max`, returning the minimum and maximum value for the enum values. `.values` returns an array with all enums.

```
enum State : uint 
{
    START,
    STOP,
}

const uint LOWEST = State.min;
const uint HIGHEST = State.max;

State start = State.values[0];
```

#####defer

Defer will be invoked on scope exit.

```
fn void test(int x)
{
    defer io::println();
    defer io::print("A");
    if (x == 1) return;
    {
        defer io::print("B");
        if (x == 0) return;
    }
    io::print("!");
}

fn void main()
{
    test(1); // Prints "A"
    test(0); // Prints "BA"
    test(10); // Prints "B!A"
}
```

Because it's often relevant to run different defers when having an error return there is also a way to create an error defer, by using the `catch` keyword directly after the defer.

*Note that this is currently not implemented.*
```
fn void! test(int x)
{
    defer io::println();
    defer io::print("A");
    defer catch io::print("B")
    defer catch (err) io::printf("%s", err.message);
    if (x == 1) return FooError!;
    print("!")
}

test(0); // Prints "!A"
test(1); // Prints "FOOBA" and returns a FooError
```

#####struct types

```
define Callback = fn int(char c);

enum Status : int
{
    IDLE,
    BUSY,
    DONE,
}

struct MyData
{
    char* name;
    Callback open;
    Callback close;
    State status;

    // named sub-structs (x.other.value)
    struct other 
    {
        int value;
        int status;   // ok, no name clash with other status
    }

    // anonymous sub-structs (x.value)
    struct 
    {
        int value;
        int status;   // error, name clash with other status in MyData
    }

    // anonymous union (x.person)
    union 
    {
        Person* person;
        Company* company;
    }

    // named sub-unions (x.either.this)
    union either 
    {
        int this;
        bool  or;
        char* that;
    }
}
```


#####Function pointers

```
module demo;

define Callback = fn int(char* text, int value);

fn int my_callback(char* text, int value) 
{
    return 0;
}

Callback cb = &my_callback;

fn void example_cb() 
{
    int result = cb("demo", 123);
    // ..
}
```

#####Error handling

Errors are handled using optional results, denoted with a '!' suffix. A variable of an optional
result type may either contain the regular value or an `optnum` enum value.

```
optnum MathError
{
    DIVISION_BY_ZERO
}

fn double! divide(int a, int b)
{
    // We return an optional result of type DIVISION_BY_ZERO
    // when b is zero.
    if (b == 0) return MathError.DIVISION_BY_ZERO!;
    return (double)a / (double)b;
}

// Re-returning an optional result uses "?" suffix
fn void! testMayError()
{
    divide(foo(), bar())?;
}

fn void main()
{
    // ratio is an optional result.
    double! ratio = divide(foo(), bar());

    // Handle the optional result value if it exists.
    if (catch err = ratio)
    {
        case MathError.DIVISION_BY_ZERO:
            io::printf("Division by zero\n");
            return;
        default:
            io::printf("Unexpected error!");
            return;
    }
    // Flow typing makes "ratio"
    // have the plain type 'double' here.
    io::printf("Ratio was %f\n", ratio);
}
```

```
fn void printFile(char[] filename)
{
    char[]! file = io::load_file(filename);

    // The following function is not executed on error.
    io::printf("Loaded %s and got:\n%s", filename, file);

    if (catch err = file)
    {
        case IoError.FILE_NOT_FOUND:
            io::printf("I could not find the file %s\n", filename);
        default:
            io::printf("Could not load %s.\n", filename);
    }
}
```

##### Contracts

Pre- and postconditions are optionally compiled into asserts helping to optimize the code.
```
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

/**
 * @param array : the array to test
 * @param length : length of the array
 * @require length > 0
 **/
fn int getLastElement(int* array, int length)
{
    return array[length - 1];
}
```

##### Macros

Macro arguments may be immediately evaluated.
```
macro foo(a, b)
{
    return a(b);
}

fn int square(int x)
{
    return x * x;
}

fn int test()
{
    int a = 2;
    int b = 3;    
    return @foo(&square, 2) + a + b; // 9
    // return @foo(square, 2) + a + b; 
    // Error the symbol "square" cannot be used as an argument.
}
```

Macro arguments may have deferred evaluation, which is basically text expansion using `#var` syntax.

```
macro foo(#a, b, #c)
{
    c = a(b) * b;
}

macro foo2(#a)
{
    return a * a;
}

fn int square(int x)
{
    return x * x;
}

fn int test1()
{
    int a = 2;
    int b = 3; 
    foo(square, a + 1, b);
    return b; // 27   
}

fn int test2()
{
    return foo2(1 + 1); // 1 + 1 * 1 + 1 = 3
}
```

Improve macro errors with preconditions:
```
/**
 * @param x : value to square
 * @checked x * x >= 0 : "cannot multiply"
 **/
macro square(x)
{
    return x * x;
}

fn void test()
{
    square("hello"); // Error: cannot multiply "hello"
    int a = 1;
    square(&a); // Error: cannot multiply '&a'
}
```

##### Methods

It's possible to namespace functions with a union, struct or enum type to enable "dot syntax" calls:

```
struct Foo
{
    int i;
}

fn void Foo.next(Foo* this)
{
    if (this) this.i++;
}

fn void test()
{
    Foo foo = { 2 };
    foo.next();
    foo.next();
    // Prints 4
    printf("%d", foo.i); 
}
```


##### Generic modules

Generic modules implements a generic system.

```
module stack <Type>;
struct Stack
{
    usize capacity;
    usize size;
    Type* elems;
}


fn void Stack.push(Stack* this, Type element)
{
    if (this.capacity == this.size)
    {
        this.capacity *= 2;
        this.elems = mem::realloc(this.elems, Type.sizeof * this.capacity);
    }
    this.elems[this.size++] = element;
}

fn Type Stack.pop(Stack* this)
{
    assert(this.size > 0);
    return this.elems[--this.size];
}

fn bool Stack.empty(Stack* this)
{
    return !this.size;
}
```

Testing it out:

```
define IntStack = Stack<int>;
define DoubleStack = Stack<double>;

fn void test()
{
    IntStack stack;
    stack.push(1);
    stack.push(2);
    // Prints pop: 2
    printf("pop: %d\n", stack.pop());
    // Prints pop: 1
    printf("pop: %d\n", stack.pop());
    
    DoubleStack dstack;
    dstack.push(2.3);
    dstack.push(3.141);
    dstack.push(1.1235);
    // Prints pop: 1.1235
    printf("pop: %f\n", dstack.pop());
}
```
