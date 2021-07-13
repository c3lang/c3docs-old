## A quick primer on C3 for C programmers

Functions are declared like C, but you need to put `func` in front:
   
    // C:
    int foo(Foo *b, int x, void *z) { ... }

    // C3
    func int foo(Foo* b, int x, void *z) { ... }

Name standards are enforced:

    // Starting with uppercase and followed somewhere by at least
    // one lower case is a user defined type:
    Foo x;
    M____y y;
    
    // Starting with lowercase is a variable or a function or a member name:

    x.myval = 1;
    int z = 123;
    func void fooBar(int x) { ... }

    // Only upper case is a constant or an enum value:

    const int FOOBAR = 123;
    enum Test 
    {
      STATE_A = 0,
      STATE_B = 2
    }    

Declaring more than one variable at a time is not allowed:

    // C
    int a, b; // Not allowed in C3

    // C3
    int a;
    int b;

Cast syntax is slightly different:

    // C
    int a = (int)foo();

    // C3
    int a = (int)(foo());

Compound literals look different, but assigning to a struct will infer the type even if
it's not the initializer.

    // C
    Foo f = { 1, 2 };
    f = (Foo) { 1, 2 };
    callFoo((Foo) { 2, 3 });

    // C3
    Foo f = { 1, 2 };
    f = { 1, 2 };
    callFoo(Foo({ 2, 3 }));


Instead of `typedef`, use `define`

    // C
    typedef Foo* FooPtr;

    // C3
    define FooPtr = Foo*;

Don't add a `;` after enum, struct and union declarations, and note the slightly 
different syntax for declaring a named struct inside of a struct.

    // C
    struct Foo
    {
      int a;
      struct 
      {
        double x;
      } bar;
    };

    // C3
    struct Foo
    {
      int a;
      struct bar 
      {
        double x;
      }
    }

Array sizes are written next to the type and arrays do not decay to pointers, 
you need to do it manually:

    // C
    int x[2] = { 1, 2 }; 
    int *y = x;

    // C3
    int[2] x = { 1, 2 };
    int *y = &x;

You will probably prefer slices to pointers when passing data around:

    // C
    int x[100] = ...;
    int y[30] = ...;
    int z[15] = ...;
    sortMyArray(x, 100);
    sortMyArray(y, 30);
    // Sort part of the array!
    sortMyArray(z + 1, 10); 

    // C3
    int[100] x = ...;
    int[30] y = ...;
    sortMyArray(&x); // Implicit conversion from int[100]* -> int[] 
    sortMyArray(&y); // Implicit conversion from int[30]* -> int[]
    sortMyArray(z[1..10]; // Inclusive ranges!

Note that declaring an array of inferred size will look different in C3:

    // C
    int x[] = { 1, 2, 3 }; // x is int[3]

    // C3
    int[*] x = { 1, 2, 3 }; // x is int[3]

Arrays are trivially copyable:

    // C
    int x[3] = ...;
    int y[3];
    for (int i = 0; i < 3; i++) y[i] = x[i];

    // C3
    int[3] x = ...;
    int[3] y = x;

Several C types that would be variable sized are fixed size, and others changed names:

    // C
    int16_t a;
    int32_t b;
    int64_t c;
    uint64_t d;
    size_t e;
    ssize_t f;
    ptrdiff_t g;
    intptr_t h;

    // C3
    short a;    // Guaranteed 16 bits
    int b;      // Guaranteed 32 bits
    long c;     // Guaranteed 64 bits
    ulong d;    // Guaranteed 64 bits
    usize e;    // Same as C size_t, depends on target
    isize f;    // Same width as usize but signed
    iptrdiff g; // Same as C ptrdiff_t depends on target
    iptr h;     // Same as intptr_t depends on target
    ireg i;     // Register sized integer

Modules are not mandatory but create a namespace, to import the names from a module, 
use `import`:

    module mylib::foo;
    
    func void test() { ... }
    struct FooStruct { ... }

    module mylib::bar;
    import mylib::foo;
   
    func void myCheck()
    {
      foo::test(); // foo prefix is mandatory.
      mylib::foo::test(); // This also works;
      FooStruct x; // But user defined types don't need the prefix.
      mylib::foo::FooStruct y; // But it is allowed.
    }

The /* */ comments are nesting:

    /* This /* will all */ be commented out */

Qualifiers like `const` and `volatile` are removed, but `const` before a constant
will make it treated as a compile time constant. The constant does not need to be typed.

    const A = false;
    // Compile time 
    $if (A):
      // This will not be compiled
    $else:
      // This will be compiled
    $endif

`goto` is removed, but there are labelled `break` and `continue` as well as `defer`
to handle the cases when it is commonly used in C.

`case` statements automatically break. Use `nextcase` to fallthrough to the 
next statement, but empty case statements have implicit fallthrough:

    // C
    switch (a)
    {
      case 1:
      case 2:
        doOne();
        break;
      case 3:
        i = 0;
      case 4:
        doFour();
        break;
      case 5:
        doFive();
      default:
        return false;
    }

    // C3
    switch (a)
    {
      case 1:
      case 2:
        doOne();
      case 3:
        i = 0;
        nextcase;
      case 4:
        doFour();
      case 5:
        doFive();
        nextcase;
      default:
        return false;
    }

Note that we can jump to an arbitrary case using C3:

    // C
    switch (a)
    {
      case 1:
        doOne();
        goto LABEL3;
      case 2:
        doTwo();
        break; 
      case 3:
    LABEL3:
        doThree();
      default:
        return false;
    }

    // C3
    switch (a)
    {
      case 1:
        doOne();
        nextcase 3;
      case 2:
        doTwo();
      case 3:
        doThree();
        nextcase;
      default:
        return false;
    }


Things that doesn't exist in C at all, but which aren't essential to get started:

- Expression blocks
- Defer
- Methods  
- Errors
- Semantic macros
- Generic modules
- Contracts
- Reflection
- Macro methods