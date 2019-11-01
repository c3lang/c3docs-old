# Variables

## Managed pointer variables

Managed pointer variables are introduced using `@` after the type, rather than `*` e.g. `Foo@ f`. A managed variable will automatically call the type's `release` member function on its value when the variable goes out of scope or is reassigned. If there is no `release` function, then `free` is called.


```
Foo@ f = Foo.alloc(); // * -> @ no retain.
Foo@ b = f;           // => f.retain(); b = f;
f = nil;              // => f.release(); f = nil;
```

Any managed variable that goes out of the scope will automatically invoke `release`, as if the pointer was set to `nil`.

```
{
    Foo@ b = Foo.alloc();
} // Automatic invocation of b.release();
```

In order to return a managed pointer that can be used as a temporary, it's often convenient to mark the return value as managed.

```
func Foo@ createFoo()
{
    return Foo.alloc();
}

createFoo(); // Implicitly introduces a deferred release.
```

If we assign a managed pointer to a variable, the release/retain is elided

```
// The following becomes f1 = createFoo() - no deferred release or retains.
Foo@ f1 = createFoo(); 
```

It's possible to manually manage a managed pointer:

```
Foo* f2 = createFoo().retain();
f2.release(); // Required to prevent leaks.
```

A managed pointer may safely assigned to a regular pointer as long as it's not retained outside of the scope.

```
{
    Foo* f3 = createFoo(); 
    printf("%d", f3.someValue);
    // Safe, since f3 isn't actually used after the scope.
}

Foo* unsafeFoo;
{
    unsafeFoo = createFoo();
}
// <- access to unsafeFoo at this point will likely break things.
```

### Managed variables not pointers

Managed variables should not be confused with automatic reference counting and similar. It is not possible to – for example – to make a struct member a "managed" pointer. It is strictly limited to variables and return values.