# Conversions and promotions

C3 differs in some crucial respects when it comes to number conversions and promotions. These are the rules for C3:

- float to int conversions require a cast
- int to float conversions do not require a cast
- bool to float converts to 0.0 / 1.0
- widening float conversions do *not* require a cast
- narrowing integer conversions require a cast
- widening conversions do *not* require a cast
- signed -> unsigned conversions must be explicit
- unsigned -> signed conversions do *not* need to be explicit if all unsigned values can be contained inside of the signed type. 
- In conditionals float to bool *do not* require a cast, any non zero float value considered true
- Implicit conversion to bool only occurs in conditionals 
  or when the value is enclosed in `()` e.g. `bool x = (1.0)` or `if (1.0) { ... }`

C3 uses two's complement arithmetic for all integer math.

## Signed/unsigned int conversions

In C, the can be simply described as "convert to the type that can contain the other". If this cannot be guaranteed, the conversion must be explicit.

|  | byte | ushort | uint  | ulong | char | short | int | long |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| byte | byte | ushort | uint | ulong | - | short | int | long |
| ushort | ushort | ushort | uint | ulong | - | - | int | long |
| uint | uint | uint | uint | ulong | - | - | - | long |
| ulong | ulong | ulong | ulong | ulong | - | - | - | - |
| char | - | - | - | - | char | short | int | long |
| short | short | - | - | - | short | short | int | long |
| int | int | int | - | - | int | int | int | long |
| long | long | long | long | - | long | long | long | long |

## Sub struct conversions

Substructs may be used in place of its parent structs in many cases. The rule is as follows:

1. A substruct pointer may implicitly convert to a parent struct.
2. A substruct *value* may be implicitly assigned to a variable with the parent struct type, This will *truncate* the value, copying only the parent part of the substruct. However, a substruct value cannot be assigned its parent struct.
3. Substruct vararray and arrays *can not* be cast (implicitly or explicitly) to an array of the parent struct type.
4. Substruct subarrays may at a later point be possible to convert into a subarray of the parent struct type.

## Pointer conversions

Pointer conversion between types usually need explicit casts. The exception is `void *` which any type may implicitly convert *to* or *from*. Conversion rules from and to arrays are detailed under [arrays](../arrays)

# Coercions

Coercion in C3 is slightly more strict than C but not overly so. C3 prohibits implicit narrowing in assignment but is otherwise similar. In binary expressions there is usually the concept of the *maximum type*.

For integers and floating points this is simply f64 > f32 > i64 > i32 > i16 > i8. So given the types i64 and i8, the maximum type is i64. If it was instead i64 and f32, the maximum is f32.

For pointers, the rules are slightly different. The maximum type of a struct is the maximum common parent struct. So if A is a subtype of B which is a subtype of C, and D is a subtype of E which is a subtype of E, then the maximumum type is C.


## Binary conversions

| operation | action | result |
| --- | --- | --- |
| `*` `/` `%` `^` `|` `&` `*%` | promote both operands to the maximum type | maximum type |
| ptr `+`/`-` int | integer is promoted to a isize or usize | ptr |
| number `+`/`-`/`+%`/`+%` number | promote both operands to the maximum type | maximum type |
| ptr `-` ptr | only valid if pointers are the same type or one pointer is void * | isize |
| `<<` `>>` `<<=` `>>=` | no coercions | left hand type |
| `&&` `||` | left and right side are evaluated as boolean | bool |
| `+=` `+%=` `-=` `-%=` `*=` `*%=` `/=` `%=` `^=` `|=` `&=` | right hand side is explicitly cast to left size type | left side type |
| `=` | right hand side is implicitly cast to left side type | left side type |
| `<` `<=` `>` `>=` `==` `!=` | promote both operands to the maximum type | bool |

## Return type conversions

In some cases an expression may have more than one branch and those branches have different types. A simple example is the ternary expressions. To resolve this, C3 does *return type conversion*. In essence this involves trying to implicitly cast each of the branches to the expected *return type*.

C3 resolves the type in this manner:

1. Is there an expected return type? If so, try to implicitly cast each branch to this value. Failure is a compile time error.
2. Is there no expected return type? Fall back to picking maximum type.

```
int a = foo();
short b = bar();

// This is using return type conversion:
long c = baz() ? a : b;

// The above will compile tog:
long c = baz() ? cast(a as long) : cast(c as long);

byte d = foobar();

// This is using maximum type because the ternary
// is inside of an addition.
long e = (baz() ? a : b) + (baz2() ? b : d);

// The above will compile to:
long e = cast((baz() ? a : cast(b as int)) 
               + cast((baz2() ? b : cast(d as short) as int)) as long);
```
