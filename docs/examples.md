#####if-statement
```
func void if_example(int a) 
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
func void example_for() {
    // the for-loop is the same as C99. 
    for (int i = 0; i < 10; i++) 
    {
        io.printf("%d\n", i);
    }

    // also equal
    for (;;) 
    {
        // ..
    }
}
```

#####while-loop

```
func void example_while() 
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

#####In-block declarations

Any control structure may declare block level variables.

```
func void example_if_while_for()
{
    // a is initialized only once, when entering the while statement
    while (int a = 10; a > 0) 
    {
        a--;
    }
    
    // a is initialized every time through the loop.
    while (int a = foo()) 
    {
        // ...
    }
    
    // a is initialized once, but assigned every time through the loop.
    while (int a = 0; a = foo()) 
    {
        // ...
    }
    
    if (int a = foo(), long b = bar(); a > 1) return @cast(int, a + b);
    
    for (int a = 0, long b = 0; a < 10; a++)
    {
        b += foo();
        if (b > 10) break;
    }
}

```
#####enum + switch

Switches have implicit break and scope. Use "next" to implicitly fallthrough or use comma:

```
enum Height : uint 
{
    LOW = 0,
    MEDIUM,
    HIGH,
}

func void demo_enum(Height h) 
{
    switch (h) 
    {
        case Height.LOW, Height.MEDIUM:
            io.printf("Not high");
            // Implicit break.
        case Height.HIGH:
            io.printf("High");
    }

    // This also works
    switch (h) 
    {
        case Height.LOW, 
        case Height.MEDIUM:
            io.printf("Not high");
            // Implicit break.
        case Height.HIGH:
            io.printf("High");
    }

    // Completely empty cases are not allowed.
    switch (h) 
    {
        case Height.LOW:
            break; // Explicit break required, since switches can't be empty.
        case Height.MEDIUM:
            io.printf("Medium");
        case Height.HIGH:
            break;
    }

    // special checking of switching on enum types
    switch (h) 
    {
        case Height.LOW,
        case Height.MEDIUM,
        case Height.HIGH,
            break;
        default:    // warning: default label in switch which covers all enumeration value
            break;
    }
    
    // Using "next" will fallthrough to the next case statement, 
    // and each case statement starts its own scope.
    switch (h) 
    {
        case Height.LOW:
            int a = 1;
            printf("A\n");
            next;
        case Height.MEDIUM,
            int a = 2;
            printf("B\n");
            next;
        case Height.HIGH,
            // a is not defined here
            printf("C\n");
    }
}
```

Enums are always namespaced.

Enums also define `.min` and `.max`, returning the minimum and maximum value for the enum values. `.all` returns an array with all enums.

```
enum State : uint 
{
    Start,
    Stop,
}

const uint lowest = State.min;
const uint highest = State.max;

State start = State.all[0];
```

#####defer

Defer will be invoked on scope exit.

```
func void test(int x)
{
    defer printf("A");
    if (x = 1) return;
    {
        defer printf("B");
        if (x == 0) return;
    }
    printf("!")
}

test(1); // Prints "A"
test(0); // Prints "BA"
test(10); // Prints "!BA"
```

Because it's often relevant to run different defers when having an error return there is also a way to create an error defer, by using the `catch` keyword directly after the defer.

```
func void test(int x)
{
    defer printf("A");
    defer catch printf("B")
    defer catch (error e) printf("%s", @name(e));
    if (x = 1) return throw Error.FOO;
    printf("!")
}

test(0); // Prints "!A"
test(1); // Prints "FOOBA" and throws
```

#####struct types

```
type Callback func int(char c);

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

type Callback func int(char* text, int value);

// also shows function attribute
func int my_callback(char* text, int value) @(unused_params) 
{
    return 0;
}

Callback cb = demo.my_callback;

func void example_cb() 
{
    int result = cb("demo", 123);
    // ..
}
```

#####Error handling

Errors are sent as a result value but uses an exception-like try/catch syntax.
```
error RandomError
{
    NORMAL,
    EXCEPTIONAL
}

func int mayThrowError() throws RandomError
{
    if (rand() > 0.5) throw RandomError.NORMAL;
    if (rand() > 0.99) throw RandomError.EXCEPTIONAL;
    return 1;
}

func void testMayError() throws
{
    // all throwable sites must be annotated with "try"
    try mayThrowError(); 
}

func void testWithoutError()
{
    try testMayError();
    
    // Catching will catch any try above in the scope.
    catch (error e)
    {
        case RandomError.NORMAL:
            io.printf("Normal Error\n");
        case RandomError.EXCEPTIONAL:
            io.printf("You win!");
        default:
            io.printf("What is this error you're talking about?");                 
    }
}

```


##### Pre and post conditions

Pre- and postconditions are optionally compiled into asserts helping to optimize the code.
```
/**
 * @param foo : the number of foos 
 * @require foo > 0, foo < 1000
 * @return number of foos x 10
 * @ensure testFoo < 10000, testFoo > 0
 **/
func int testFoo(int foo)
{
    return foo * 10;
}

/**
 * @param array : the array to test
 * @param length : length of the array
 * @require length > 0
 **/
func int getLastElement(int? array, int length)
{
    return array[length - 1];
}
```

##### Macros

Macro arguments may be immediately evaluated.
```
macro foo(a, b)
{
    return *a(b);
}

func int square(int x)
{
    return x * x;
}

func int test()
{
    int a = 2;
    int b = 3;    
    return @foo(&square, 2) + a + b; // 9
    // return @foo(square, 2) + a + b; 
    // Error the symbol "square" cannot be used as an argument.
}
```

Macro arguments may have deferred evaluation, which is basically text expansion.
```
macro foo(&a, b, &c)
{
    c = a(b) * b;
}

macro foo2(&a)
{
    return a * a;
}

func int square(int x)
{
    return x * x;
}

func int test1()
{
    int a = 2;
    int b = 3; 
    foo(square, a + 1, b);
    return b; // 27   
}

func int test2()
{
    return foo2(1 + 1); // 1 + 1 * 1 + 1 = 3
}
```

Improve macro errors with preconditions:
```
/**
 * @param x : value to square
 * @require x * x >= 0 : "cannot multiply"
 **/
macro square(x)
{
    return x * x;
}

func void test()
{
    square("hello"); // Error: cannot multiply "hello"
    int a = 1;
    square(&a); // Error: cannot multiply '&a'
}
```

##### Nullability annotations

Use & for nullable pointers, * for non nullable pointers.

```
func Foo* getFoo() { ... }
func Foo& getFooNotNull() { ... }

func testThings()
{
    int i;
    Foo* foo1 = getFoo();
    // i = foo1.intVal;
    // Error, reference may be null
    Foo& foo2 = getFooNotNull();
    i = foo2.intVal;
    
    assert(foo1);
    i = foo1.intVal;
}
```

