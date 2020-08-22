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
    
    if (int a = foo(), long b = bar(); a > 1) return cast(a + b as int);
    
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
        case LOW, MEDIUM:
            io.printf("Not high");
            // Implicit break.
        case HIGH:
            io.printf("High");
    }

    // This also works
    switch (h) 
    {
        case LOW, 
        case MEDIUM:
            io.printf("Not high");
            // Implicit break.
        case Height.HIGH:
            io.printf("High");
    }

    // Completely empty cases are not allowed.
    switch (h) 
    {
        case LOW:
            break; // Explicit break required, since switches can't be empty.
        case MEDIUM:
            io.printf("Medium");
        case HIGH:
            break;
    }

    // special checking of switching on enum types
    switch (h) 
    {
        case LOW,
        case MEDIUM,
        case HIGH,
            break;
        default:    // warning: default label in switch which covers all enumeration value
            break;
    }
    
    // Using "next" will fallthrough to the next case statement, 
    // and each case statement starts its own scope.
    switch (h) 
    {
        case LOW:
            int a = 1;
            printf("A\n");
            next;
        case MEDIUM,
            int a = 2;
            printf("B\n");
            next;
        case HIGH,
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
    if (x == 1) return;
    {
        defer printf("B");
        if (x == 0) return;
    }
    printf("!")
}

test(1); // Prints "A"
test(0); // Prints "BA"
test(10); // Prints "B!A"
```

Because it's often relevant to run different defers when having an error return there is also a way to create an error defer, by using the `catch` keyword directly after the defer.

```
func void! test(int x)
{
    defer printf("A");
    defer catch printf("B")
    defer catch (err) printf("%s", e.message);
    if (x = 1) return FooError!;
    printf("!")
}

test(0); // Prints "!A"
test(1); // Prints "FOOBA" and returns a FooError
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

Errors are sent as a result value, called a "failable":

```
error DivisionByZero;

func double divide(int a, int b)
{
    if (b == 0) return DivisionByZero!;
    return cast(a as double) / cast(b as double);

}

// Rethrowing an error uses "try"
func void! testMayError()
{
    try divide(foo(), bar()); 
}

func void testHandlingError()
{
    // ratio has a failable type.
    double! ratio = divide(foo(), bar());
    
    // Handle the error
    catch (err = ratio)
    {
        case DivisionByZero:
            io::printf("Division by zero\n");
            return;
        default:
            io::printf("Unexpected error!");                 
            return;
    }
    // Flow typing makes "ratio"
    // have the type double here.
    printf("Ratio was %f\n", ratio);
}
```

```
import std::io;
func void printFile(string filename)
{
    string! file = io::load_file(filename);
    
    // The following function is not executed on error.
    io::printf("Loaded %s and got:\n%s", filename, file);

    catch (err = file)
    {
        case FileNotFoundError:
            printf("I could not find the file %s\n", filename);
        default:
            printf("Could not load %s: '%s'", filename, error.message());
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
macro foo($a, b, $c)
{
    c = a(b) * b;
}

macro foo2($a)
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

##### Value functions

It's possible to namespace functions with a union, struct or enum type to enable "dot syntax" calls:

```
struct Foo
{
    int i;
}

func void Foo.next(Foo* this)
{
    if (this) this.i++;
}

func void test()
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
module stack($a)

struct Stack
{
    type($a)[] elems;
}

func Stack.init(Stack* this)
{
    this.elems = nil;
}

func void Stack.push(Stack* this, type($a) element)
{
    this.elems.add(element);
}

func $A Stack.pop(Stack* this)
{
    assert(this.elems.size > 0);
    this.elems.removeLast();
}

func bool Stack.empty(Stack* this)
{
    return this.elems.size == 0;
}
```

Testing it out:

```
import stack;

define Stack(int) as IntStack;
define Stack(double) as DoubleStack;

func void test()
{
    IntStack stack;
    stack.init();
    stack.push(1);
    stack.push(2);
    // Prints pop: 2
    printf("pop: %d\n", stack.pop())
    // Prints pop: 1
    printf("pop: %d\n", stack.pop())
    
    DoubleStack dstack;
    dstack.init();
    dstack.push(2.3);
    dstack.push(3.141);
    dstack.push(1.1235)
    // Prints pop: 1.1235
    printf("pop: %f\n", dstack.pop())
}
```
