# Conversions and promotions

C3 differs in some crucial respects when it comes to number conversions and promotions. These are the rules for C3:

- float to int conversions require a cast
- float to boolean conversions *do not* require a cast.
- int to float conversions do not require a cast
- bool to float converts to 0.0 / 1.0
- double to float conversion is allowed, but may be set to warn
- narrowing integer conversions require a cast
- widening conversions do *not* require a cast
- signed/unsigned int conversions work differently from C as detailed below
- float to bool *do not* require a cast, any non zero float value considered true

C3 uses two's complement arithmetic for all integer math.

## Signed/unsigned int conversions

In C, the can be simply described as "convert to the type with the largest positive value" (aside from the promotion up to int). Consequently a signed int + unsigned int converts to unsigned int. In C3, the sign is considered more important. Consequently the conversion with mixed signed / unsigned will *always* be converted to a signed value.

On debug builds, a conversion of unsigned to signed which results in an overflow (for example, converting the ushort 0x8000 to a short) will trap. For release builds overflow is instead undefined behaviour.


The rule is as follows:

1. Find the largest bit size of the two operands.
2. If either is signed: convert to a signed integer of the largest bit size.
3. If both are unsigned: convert to an unsigned integer of the largest bit size.
4. If the conversion results in overflow in debug builds this will trap.
5. If the conversion results in overflow in release builds the result is undefined behaviour.

|  | bool | byte | ushort | uint  | ulong | char | short | int | long |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| byte | bool | byte | ushort | uint | ulong | char | short | int | long |
| byte | byte | byte | ushort | uint | ulong | char | short | int | long |
| ushort | ushort | ushort | ushort | uint | ulong | short | short | int | long |
| uint | uint | uint | uint | uint | ulong | int | int | int | long |
| ulong | ulong | ulong | ulong | ulong | ulong | long | long | long | long |
| char | char | char | short | int | long | char | short | int | long |
| short | short | short | short | int | long | short | short | int | long |
| int | int | int | int | int | long | int | int | int | long |
| long | long | long | long | long | long | long | long | long | long |

## Sub struct conversions

Substructs may be used in place of its parent structs in many cases. The rule is as follows:

1. A substruct pointer may implicitly convert to a parent struct.
2. A substruct *value* may be implicitly assigned to a variable with the parent struct type, This will *truncate* the value, copying only the parent part of the substruct. However, a substruct value cannot be assigned its parent struct.
3. Substruct vararray and arrays *can not* be cast (implicitly or explicitly) to an array of the parent struct type.
4. Substruct subarrays may at a later point be possible to convert into a subarray of the parent struct type.

## Pointer conversions

Pointer conversion between types usually need explicit casts. The exception is `void *` which any type may implicitly convert *to* or *from*. Conversion rules from and to arrays are detailed under [arrays](../arrays)