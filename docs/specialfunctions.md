# Built in functions

C3 offers direct access to built in functions where available for certain operations.

Often these are implemented as generic functions that may have multiple implementations.

* abs
* byteswap
* bitreverse
* ceil
* clz
* cos
* ctz
* divfloor
* divtrunc
* exp
* exp2
* floor
* ln
* log2
* log10
* memcpy
* memset
* mod
* muloverflow
* popcount
* shlexact
* shrexact
* sin
* sqrt
* suboverflow
* trunc

## abs

Returns the absolute value of a float or integer value. The underlying functions are:

* `float fabsf(float)`
* `double fabsf(double)`
* `quad fabsl(quad)`*
* `char absc(char)`
* `int abs(int)`
* `long absl(long)`

## byteswap

Swaps the byte order, switching between little endian and big endian. The underlying functions are:

* `ushort byteswapus(ushort)`
* `short byteswaps(short)`
* `uint byteswapu(uint)`
* `int byteswap(int)`
* `ulong byteswapul(ulong)`
* `long byteswapl(long)`

## bitreverse

Reverses all bits in an integer, including the sign bit the underlying functions are

* `byte bitreverseb(byte)`
* `char bitreversec(char)`
* `ushort bitreverseus(ushort)`
* `short bitreverses(short)`
* `uint bitreverseu(uint)`
* `int bitreverse(int)`
* `ulong bitreverseul(ulong)`
* `long bitreversel(long)`

## ceil

Return the closest integral number, rounded up.

* `float ceilf(float)`
* `double ceil(double)`

## clz

Return the number of leading zeroes.

* `uint clzb(byte/char)`
* `uint clz(int/uint)`
* `uint clzl(long/ulong)`

## cos

Return the cos value from radian angle.

* `float cosf(float)`
* `double cos(double)`
* `quad cos(quad)`

## ctz

Return the number of trailing zeroes.

* `uint ctzb(byte/char)`
* `uint ctz(int/uint)`
* `uint ctzl(long/ulong)`

