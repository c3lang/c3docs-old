# Conversions and promotions

C3 differs in some crucial respects when it comes to number conversions and promotions. These are the rules for C3:

- float to int conversions require a cast
- int to float conversions do not require a cast
- bool to float converts to 0.0 / 1.0
- widening float conversions do *not* require a cast
- narrowing conversions require a cast(*)
- widening conversions do *not* require a cast
- signed <-> unsigned conversions do not require a cast.
- In conditionals float to bool *do not* require a cast, any non zero float value considered true
- Implicit conversion to bool only occurs in conditionals or when the value is enclosed in `()` e.g. `bool x = (1.0)` or `if (1.0) { ... }`

C3 uses two's complement arithmetic for all integer math.

## Target type

The left hand side of an assignment, or the parameter type in a call is known as the *target type* the target type is used for implicit widening and inferring struct initialization.

## Implicit promotion

Like C, C3 uses implicit promotion of integer and floating point variables:

1. For any floating point type with a bit width smaller than 32 bits, widen to `float`. E.g. `half -> float`
2. After 1, for any floating point type with a bit width smaller than the *target type* widen to the target type.
3. For an integer type smaller than the *minimum arithmetic width*, promote the value to a signed integer of the *minimum arithmetic width* (this usually corresponds to a c int). E.g. `uchar -> int`
4. After 3, for an integer type smaller than the *target type*, promote the value to an integer of the the same bit width. E.g. `ulong = int + int -> ulong = long + long`

## Maximum type

The *maximum type* is a concept used when unifying two or more types. The algorithm follows:

1. First perform implicit promotion.
3. If both types are the same, the maximum type is this type. 
4. If one type is a floating point type, and the other is an integer type, the maximum type is the floating point type. E.g. `int + float -> float`.
5. If both types are floating point types, the maximum type is the widest floating point type. E.g. `float + double -> double`.
6. If both types are integer types with the same signedness, the maximum type is the widest integer type of the two. E.g. `uint + ulong -> ulong`.
7. If both types are integer types with different signedness, the maximum type is a signed integer with the same bit width as the maximum integer type. `ulong + int -> long`
8. If at least one side is a struct or a pointer to a struct with an `inline` directive on a member, check recursively check if the type of the inline member can be used to find a maximum type (see below under sub struct conversions)
9. All other cases are errors.


## Sub struct conversions

Substructs may be used in place of its parent structs in many cases. The rule is as follows:

1. A substruct pointer may implicitly convert to a parent struct.
2. A substruct *value* may be implicitly assigned to a variable with the parent struct type, This will *truncate* the value, copying only the parent part of the substruct. However, a substruct value cannot be assigned its parent struct.
3. Substruct subarrays, vararrays and arrays *can not* be cast (implicitly or explicitly) to an array of the parent struct type.

## Pointer conversions

Pointer conversion between types usually need explicit casts. The exception is `void *` which any type may implicitly convert *to* or *from*. Conversion rules from and to arrays are detailed under [arrays](../arrays)


## Implicit narrowing

Implicit narrowing is only allowed for floating point values and integer types. The following must hold:

1. To narrow to a floating point type, all sub expressions must be integers or a floating point as narrow or more narrow than the target type.
2. To narrow to an integer type, all sub expressions must be integers and as narrow or narrower than the target type, ignoring signedness.

```
half h = 12.0;
float f = 13.0;
double d = 22.0;

char x = 1;
short y = -3;
int z = 0xFFFFF;
ulong w = -0xFFFFFFF;

x = x + x; // => calculated as x = (char)((int)(x) + (int)(x));
x = y + x; // => error
w = x + y; // => calculated as w = (ulong)((long)(x) + (long)(x));

h = x * h; // => calculated as h = (half)((float)(x) * (float)(h));
h = f + x; // => error
d = f * h; // => calculated as d = (double)(f) * (double)(h);
```

## Binary conversions

### 1. Multiplication, division, remainder, subtraction / addition with both operands being numbers

These operations are only valid for integer and float types.

1. Resolve the operands, left to right, pushing down the target type.
2. Find the maximum type of the two operands.
3. Promote both operands to the resulting type.
4. The resulting type of the expression is the resulting type.

### 2. Addition with left side being a pointer

1. Resolve the left hand operand pushing down the target type.
2. Resolve the right hand operand pushing down iptrdiff as the target type.
3. If the rhs is not an integer, this is an error.   
3. If the rhs has a bit width that exceeds iptrdiff, this is an error.
4. The result of the expression is the lhs type.

### 3. Subtraction with lhs pointer and rhs integer 

1. Resolve the left hand operand pushing down the target type.
2. Resolve the right hand operand pushing down iptrdiff as the target type.
3. If the right hand type has a bit width that exceeds iptrdiff, this is an error.
4. The result of the expression is the left hand type.

### 4. Subtraction with both sides pointers

1. Resolve the operands, left to right.
2. If the either side is a `void *`, it is cast to the other type.
3. If the types of the sides are different, this is an error.   
4. The result of the expression is iptrdiff.
5. If this result exceeds the target width, this is an error.

### 6. Bit operations `^` `&` `|`

These operations are only valid for integers and booleans.

1. Resolve the operands, left to right, pushing down the target type.
2. Find the maximum type of the two operands.
3. Promote both operands to the resulting type.
4. The result of the expression is the resulting type.

### 6. Shift operations `<<` `>>` 

These operations are only valid for integers.

1. Resolve the left operand pushing down the target type.
2. Resolve the right operand *without pushing down* a target type.   
3. In safe mode, insert a trap to ensure that rhs >= 0 and rhs < bit width of the left hand side.
4. The result of the expression is the lhs type.

### 7. Assignment operations `+=` `-=` `*=` `*=` `/=` `%=` `^=` `|=` `&=`

1. Resolve the type of the left operand.
2. Resolve the right operand pushing down the lhs type as target type
3. The result of the expression is the lhs type.

### 8. Assignment shift `>>=` `<<=`

1. Resolve both operands
3. In safe mode, insert a trap to ensure that rhs >= 0 and rhs < bit width of the left hand side.
4. The result of the expression is the lhs type.

### 9. `&&` and `||`

1. Resolve both operands.
2. Insert bool cast of both operands.
3. The type is bool.

### 10. `<=` `==` `>=` `!=`

1. Resolve the operands, left to right.
2. Find the maximum type of the two operands.
3. Promote both operands to the resulting type.
4. The type is bool.

## Unary conversions

### 1. Bit negate

1. Resolve the inner operand, pushing down the target type.
2. If the inner type is not an integer this is an error.   
2. The type is the inner type.

### 2. Boolean not

1. Resolve the inner operand.
2. The type is bool.

### 3. Negation

1. Resolve the inner operand, pushing down the target type.
2. If the type inner type is not a number this is an error.
3. If the inner type is an unsigned integer, cast it to the same signed type.
4. The type is the type of the result from (3)

### 4. `&` and `&&`

1. Resolve the inner operand.
2. The type is a pointer to the type of the inner operand.

### 5. `*`

1. Resolve the inner operand.
2. If the operand is not a pointer, or is a `void *` pointer, this is an error.
3. The type is the pointee of the inner operand's type.

Dereferencing 0 is implementation defined.

### 6. `++` and `--`

1. Resolve the inner operand.
2. If the type is not a number, this is an error.
3. The type is the same as the inner operand.

## Base expressions

### 1. Typed identifiers

1. The type is that of the declaration.
2. If the width of the type is less than that of the target type, widen to the target type.
3. If the width of the type is greater than that of the target type, it is an error.

### 2. Constants and literals

1. If it is untyped, the type is that of the smallest type that the constant can fit in.
2. If the width of the type is less than that of the target type, widen to the target type.
3. If the width of the type is greater than that of the target type, it is an error.

## Ternary and return type conversions

In some cases an expression may have more than one branch and those branches have different types. A simple example is the ternary expressions. To resolve this, C3 does *return type conversion*. In essence this involves trying to implicitly cast each of the branches to the expected *return type*.

C3 resolves the type in this manner:

1. Is there an expected return type? Proceed error checking with the expected return type as the *target type* 
2. Is there no expected return type? Find the maximum type of all results and promote all results to the value. 

```
int a = foo();
short b = bar();

// This is using return type conversion:
long c = baz() ? a : b;

// The above will compile to:
long c = baz() ? (long)(a) : (long)(c);

char d = foobar();

// This is using maximum type because the ternary
// is inside of an addition.
baz() ? a : b;

// The above will compile to:
baz() ? a : (int)(b);               
```
