# Any and protocols

## Working with the type of `any*` at runtime.

The `any*` type is recommended for writing code that is polymorphic at runtime where macros are not appropriate.
It can be thought of as a typed `void*`. Note that it is a fat pointer and is two pointers wide (unlike `void*`).
It cannot be dereferenced.

An `any*` can be created by assigning any pointer to it. You can then query the `any*` type for the typeid of 
the enclosed type (the type the pointer points to) using the `type` field.

This allows switching over the typeid, either using a normal switch:

    switch (my_any.typeid)
    {
        case Foo.typeid:
            ...
        case Bar.typeid:
            ...
    }

Or the special `any*`-version of the switch:

    switch (my_any)
    {
        case Foo:
            // my_any can be used as if it was Foo* here
        case Bar:
            // my_any can be used as if it was Bar* here
    }

Sometimes one needs to manually construct an any-pointer, which
is typically done using the `any_make` function: `any_make(ptr, type)`
will create an `any*` pointing to `ptr` and with typeid `type`.

Since the runtime `typeid` is available, we can query for any runtime `typeid` property available
at runtime, for example the size, e.g. `my_any.typeid.sizeof`. This allows us to do a lot of work
on with the enclosed data without knowing the details of its type.

For example, this would make a copy of the data and place it in the variable `any_copy`:

    void* data = malloc(a.type.sizeof);
	mem::copy(data, a.ptr, a.type.sizeof);
    any* any_copy = any_make(data, a.type);    


## Protocols

Most statically typed object-oriented languages implements extensibility using vtables. In C, and by extension
C3, this is possible to emulate by passing around structs containing list of function pointers in addition to the data.

While this is efficient and often the best solution, but it puts certain assumptions on the code and makes interfaces
more challenging to evolve over time.

As an alternative there are languages (such as Objective-C) which instead use message passing to dynamically typed
objects, where the availability of a certain functionality may be queried at runtime.

C3 provides this latter functionality over the `any*` type using *protocols*.

### Defining a protocol

The first step is to define a protocol:

    protocol MyName
    {
        fn String myname(&self);
    }

While `myname` will behave as a method, we declare it without type, and always with an inferred type using `&self`.

### Implementing the protocol

After the protocol is added, we can add methods that implement it:

    struct Bob { int x; }
    fn String Bob.myname(Bob*) : MyName { return "I am Bob!"; }

    fn String int.myname(int*) : MyName { return "I am int!"; }

One of the protocols available in the standard library is Printable, which contains `to_format` and `to_string`. 
If we implemented it for our struct above it might look like this:

    fn String Bob.to_string(Bob* bob, Allocator* using) : Printable
    {
        return string::printf("Bob(%d)", bob.x, .using = using);
    }

### Referring to a protocol by pointer

A pointer to a protocol e.g. `MyName*` is can be cast back and forth to `any*`, but only types which 
implement the protocol completely may implicitly be cast to the protocol pointer.

So for example:

    Bob b = { 1 };
    double d = 0.5;
    MyName* a = &b; // Valid, Bob implements MyName.
    // MyName* c = &d; // Error, double does not implement MyName.

### Calling dynamic methods

Methods implementing protocols are like normal methods, and if called directly, they are just normal function calls. The
difference is that they may be invoked through `any*`:

An example helps illustrate the typical use:

    fn void whoareyou(any* a)
    {
        // Query if the function exists
        if (!&a.myname)
        {
            io::printn("I don't know who I am.");
            return;
        }
        // Dynamically call the function
        io::printn(a.myname());
    }


We first query if the method exists on the value wrapped by `any*`. If it doesn't then we print
`"I don't know who I am."` otherwise we the value's `myname()` method and print it.

Rather than using the `any*` type, it's common to use the protocol instead. As with *any*, we always
refer to it as a pointer:

    fn void whoareyou2(MyName* a)
    {
        ...
    }

If we pass a value which doesn't implement `Test` to this function it would be a compiler error.

Here is another example, showing how the correct function will be called depending on type:

    fn void main()
    {
        int i;
        double d;
        Bob bob;

        any* a = &i; 
	    whoareyou(a); // Prints "I am int!"
	    a = &d;
	    whoareyou(a); // Prints "I don't know who I am."
	    a = &bob;
	    whoareyou(a); // Prints "I am Bob!"
    }

## Variable argument functions with implicit `any`

Regular typed varargs are of a single type, e.g. `fn void abc(int x, double... args)`.
In order to take variable functions that are of multiple types, `any` may be used. 
There are two variants:

### Explicit `any` vararg functions

This type of function has a format like `fn void vaargfn(int x, any... args)`. Because only
pointers may be passed to an `any`, the arguments must explicitly be pointers (e.g. `vaargfn(2, &b, &&3.0)`).

While explicit, this may be somewhat less user-friendly than implicit vararg functions:

### Implicit `any` vararg functions

The implicit `any` vararg function has instead a format like `fn void vaanyfn(int x, args...)`.
Calling this function will implicitly cause taking the pointer of the values (so for
example in the call `vaanyfn(2, b, 3.0)`, what is actually passed are `&b` and `&&3.0`).

Because this passes values implicitly by reference, care must be taken *not* to mutate any
values passed in this manner. Doing so would very likely break user expectations.