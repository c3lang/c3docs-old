# Undefined behaviour

Like C, C3 uses undefined behaviour. In contrast, C3 will *trap* - that is, print an error trace and abort – on undefined behaviour in debug builds. This is similar to using C with a UB sanitizer. It is only during release builds that actual undefined behaviour occurs.

In C3, undefined behaviour means that the compiler is free to interpret *undefined behaviour as if behaviour cannot occur*.

In the example below:

```
uint x = foo();
uint z = 255 / x;
if (x == 0 || z > 10) return bar();
return 1;
```

The case of `x == 0` would invoke undefined behaviour for `255/x`. For that reason, 
the compiler may assume that `x != 0` and compile it into the following code: 

```
foo();
return 1;
```

As a contrast, the safe build will compile code equivalent to the following.

```
uint x = foo();
if (x == 0) trap("Division by zero")
return 1;
```

## List of undefined behaviours

The following operations cause undefined behaviour in release builds of C3:

| operation | will trap in safe builds
| --- | :-: |
| int / 0 | Yes |
| int % 0 | Yes |
| using explicitly uninitialized memory | Implementation dependent |
| array index out of bounds | Yes |
| dereference `null` | Yes |
| dereferencing memory not allocated | Implementation dependent |
| dereferencing memory outside of its lifetime | Implementation dependent |
| casting pointer to the incorrect array or vararray | Implementation dependent |
| violating pre or post conditions | Yes |
| violating asserts | Yes |
| reaching $unreachable code | Yes |

## List of implementation dependent behaviours

Some behaviour is allowed to differ between implementations.

| operation | will trap in safe builds | possible behaviour
| --- | :-: | --- |
| comparing pointers of different provenance | No | Any result |
| shifting by more than bit width | Yes | Any result |

