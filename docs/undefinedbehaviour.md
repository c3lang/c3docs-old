# Undefined behaviour

Like C, C3 uses undefined behaviour. In contrast, C3 will *trap* - that is, print an error trace and abort – on undefined behaviour in debug builds. This is similar to using C with a UB sanitizer. It is only during release builds that actual undefined behaviour occurs.

In C3, undefined behaviour means that the compiler is free to interpret *undefined behaviour as if behaviour cannot occur*.

In the example below:

```
uint x = foo();
uint z = x + 1;
if (x == 0xFFFF) return bar();
return 1;
```

The case of `x == 0xFFFF` would invoke undefined behaviour (due to overflow) for the assignment to `z`. For that reason, 
the compiler may assume that `x < 0xFFFF` and compile it into the following code: 

```
foo();
return 1;
```

As a contrast, the debug build will compile code equivalent to the following.

```
uint x = foo();
uint z;
if (!@add_overflow(&z, x, 1)) trap("Unsigned overflow");
if (x == 0xFFFF) return bar();
return 1;
```

## List of undefined behaviours

The following operations cause undefined behaviour in release builds of C3:

| operation | well defined alternative | will trap in debug builds
| --- | --- | :-: |
| overflow on + | Use +% or @add_overflow | Yes |
| overflow on - | Use -% or @sub_overflow | Yes |
| overflow on * | Use *% or @mult_overflow | Yes |
| int / 0 | - | Yes |
| int % 0 | - | Yes |
| using explicitly uninitialized memory | - | Yes |
| array index out of bounds | - | Yes |
| dereference NULL | - | Yes |
| dereferencing memory not allocated | - | Implementation dependent |
| dereferencing memory outside of its lifetime | - | Implementation dependent |
| casting pointer to the incorrect array or vararray | - | Implementation dependent |
| violating pre or post conditions | - | Yes |
| violating asserts | - | Yes |

