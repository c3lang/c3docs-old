# Expressions

Expressions work like in C, with one example: it is possible to take the address of a temporary. This uses the operator `&&` rather than `&`.

Consequently this is valid:

```
fn void test(int* x) { ... }

test(&&1);

// In C:
// int x = 1;
// test(&x);
```

## Compound literals

C3 has C's compound literals, but unlike C's cast style syntax `(MyStruct) { 1, 2 }`, it uses a slightly different syntax, similar to C++: `MyStruct({ 1, 2 })`.

```
struct Foo
{
    int a;
    double b;
}

fn void test(Foo x) { ... }

... 

test(Foo({ 1, 2.0 }));
```

Arrays follow the same syntax:

```
fn void test(int[3] x) { ... }

...

test(int[3]({ 1, 2, 3 }));
```

One may take the address of temporaries, using `&&` (rather than `&` for normal variables). This allows the following:

Passing a slice
```
fn void test(int[] y) { ... }

// Using &&
test(&&int[3]({ 1, 2, 3 }));

// Explicitly slicing:
test(int[3]({ 1, 2, 3 }[..]));
```

Passing a pointer to an array
```
fn void test1(int[3]* z) { ... }
fn void test2(int* z) { ... }

test1(&&int[3]({ 1, 2, 3 }));
test2(&&int[3]({ 1, 2, 3 }));
```

## Constant expressions

In C3 all _constant expressions_ are guaranteed to be calculated at runtime. The following are considered constant expressions:

1. The `null` literal.
2. Boolean, floating point and integer literals.
3. The result of arithmetics on constant expressions.
4. Compile time variables (prefixed with `$`)
5. Global constant variables with initializers that are constant expressions.
6. The result of macros that does not generate code and only uses constant expressions.
7. The result of a cast if the value is cast to a boolean, floating point or integer type and the value that is converted is a constant expression.
8. String literals.
9. Initializer lists containing constant values.

Some things that are *not* constant expressions:

1. Any pointer that isn't the `null` literal, even if it's derived from a constant expression.
2. The result of a cast except for casts of constant expressions to a numeric type.
3. Compound literals - even when values are constant expressions.
