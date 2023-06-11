# Dynamic code

## Working with the type of `any` at runtime.

The `any` type is recommended for writing code that is polymorphic at runtime where macros are not appropriate.

An `any` value can be created by assigning any pointer to it. You can then query the `any` type for the typeid of 
the enclosed type (the type the pointer points to) using the `type` field.

This allows switching over the typeid, either using a normal switch:

    switch (my_any.typeid)
    {
        case Foo.typeid:
            ...
        case Bar.typeid:
            ...
    }

Or the special `any`-version of the switch:

    switch (my_any)
    {
        case Foo:
            // my_any can be used as if it was Foo* here
        case Bar:
            // my_any can be used as if it was Bar* here
    }

Sometimes one needs to manually construct an `any`, and it can be done similar to creating any struct type: `any { ptr, type }`
will create an `any` pointing to `ptr` and with typeid `type`.

Since the runtime `typeid` is available, we can query for any runtime `typeid` property available
at runtime, for example the size, e.g. `my_any.typeid.sizeof`. This allows us to do a lot of work
on with the enclosed data without knowing the details of its type.

For example, this would make a copy of the data and place it in the variable `any_copy`:

    void* data = malloc(a.type.sizeof);
	mem::copy(data, a.ptr, a.type.sizeof);
    any any_copy = { data, a.type };    


## Dynamic calls

Most statically typed object-oriented languages implements extensibility using vtables. In C, and by extension
C3, this is possible to emulate by passing around structs containing list of function pointers in addition to the data.

While this is efficient and often the best solution, but it puts certain assumptions on the code and makes interfaces
more challenging to evolve over time.

As an alternative there are languages (such as Objective-C) which instead use message passing to dynamically typed
objects, where the availability of a certain functionality may be queried at runtime.

C3 provides this latter functionality over the `any` type using `@interface` and `@dynamic` annotations.

### Defining an interface

The first step is to define an interface, this is done by defining an `any` method *without a body* annotated 
`@interface`:

    fn String any.myname(void*) @interface;

Note how `void*` rather than `any` is the first parameter. Other than this, it is like any other method 
declaration.

### Implementing the interface

After the interface is added, we can create methods that implement this interface.

    struct Bob { int x; }
    fn String Bob.myname(Bob*) @dynamic { return "I am Bob!"; }

    fn String int.myname(int*) @dynamic { return "I am int!"; }

One of the interfaces available in the standard library is to_string. If we implemented it for our struct above
it might look like this:

    fn String Bob.to_string(Bob* bob, Allocator* using) @dynamic
    {
        return string::printf("Bob(%d)", bob.x, .using = using);
    }

### Calling dynamic methods

`@dynamic` methods are just like normal methods. If called directly, they are just normal function calls. The
difference is that they may be invoked through `any`:

An example helps illustrate the typical use:

    fn void whoareyou(any a)
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

We first query if the method exists on the value wrapped by `any`. If it doesn't then we print
`"I don't know who I am."` otherwise we the value's `myname()` method and print it.

We could use it like this:

    fn void main()
    {
        int i;
        double d;
        Bob bob;

        any a = &i; 
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