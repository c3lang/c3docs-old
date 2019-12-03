# Undefined behaviour

Like C, C3 uses undefined behaviour. In contrast, C3 will *trap* - that is, print an error trace and abort – on undefined behaviour in debug builds. This is similar to using C with a UB sanitizer. It is only during release builds that actual undefined behaviour occurs.

In C3, undefined behaviour means that the compiler is free to interpret *undefined behaviour as if behaviour cannot occur*.

In the example below, a ushort is compared to a short:

```
ushort u = foo();
short s = bar();
if (s < 0 && u > s) return 1;
return 0;
```

In this example we know that for the result `1`

1. `s < 0`
2. `u > s`

The comparison between `u` and `s` will cast `u` to a short. For any value u >= 0x8000 this will overflow, and with 2s complement we know that this will yield a negative value. However, overflow on implicit cast of unsigned -> signed is undefined behaviour (see [conversions](../conversion)). This means that the compiler is allowed to assume that overflow never happens. Consequently the code can be reduced to the following in release builds:

```
ushort u = foo();
short s = bar();
if (s < 0) return 1;
return 0;
```

As a contrast, the debug build will compile code equivalent to the following.

```
ushort u = foo();
short s = bar();
if (s < 0)
{
    if (u >= 0x0800) trap("Unsigned to signed conversion overflow.");
    return 1;
}
return 0;
```

## List of undefined behaviours

The following operations cause undefined behaviour in release builds of C3:

| operation | well defined alternative | will trap in debug builds
| --- | --- | :-: |
| overflow during implicit conversion signed -> unsigned | explicit cast to signed, will use 2s complement on overflow | Yes |
| overflow on + | Use +% or @add_overflow | Yes |
| overflow on - | Use -% or @sub_overflow | Yes |
| overflow on * | Use *% or @mult_overflow | Yes |
| int / 0 | - | Yes |
| int % 0 | - | Yes |
| using uninitialized memory | - | Yes |
| array index out of bounds | - | Yes |
| dereference NULL | - | Yes |
| dereferencing memory not allocated | - | Implementation dependent |
| dereferencing memory outside of its lifetime | - | Implementation dependent |
| casting pointer to the incorrect array or vararray | - | Implementation dependent |
| violating pre or post conditions | - | Yes |
| violating asserts | - | Yes |

