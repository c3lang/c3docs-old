# Pre and post conditions

Pre and post conditions are optional checks that the compiler may use for optimization and runtime checks. Note that _compilers are not obliged to process pre and post conditions at all_. However, violating either pre or post conditions is considered undefined behaviour, so a compiler may optimize as if they always hold – even if a potential bug may cause them to be violated.

# Pre conditions

Pre conditions are usually used to validate incoming arguments. Each condition must be an expression that can be evaluated to a boolean. A pre condition use the `@require` annotation.

```
/**
 * @require foo > 0, foo < 1000
 **/
func int testFoo(int foo)
{
    return foo * 10;
}
```

# Post conditions

Post conditions are evaluated to make checks on the resulting state after passing through the function. There are two special post conditions: `const` and `pure`.

The post condition uses the `@ensure` annotation. Where `return` is used to mark the return value from the function. 

For `const` and `pure`, they can either be given as separate annotations: `@pure` and `@const <parameter>, ...`, or inside an `@ensure` using the format `const(<parameter>)` and `pure(<function name>)`. A parameter marked `const` guarentees that the memory pointed to is not altered within the scope of a function. `pure` guarantees that the function does not read or write to any global variables.
    
```
/**
 * @ensure foo != nil, const(foo), return > foo.x;
 * @pure
 **/
func uint checkFoo(Foo* foo)
{
    uint y = abs(foo.x) + 1;
    // If we had row: foo.x = 0, then this would be a compile time error.
    return y * abs(foo.x);
}
```

### Const in detail

The `const` annotation allows a program to make assumtions in regards of how the function treats the parameter. This can then be used by a compiler make optimizations for any caller of the function.

However, it should be noted that the compiler might not detect whether the annotation is correct or not! This program might compile, but will behave strangely:

```
func void badFunc(int* i)
{
    *i = 2;
}

/**
 * @ensure const(i)
 */
func void lyingFunc(int* i)
{
    badFunc(i); // The compiler might not check this!
}

func void test()
{
    int a = 1;
    lyingFunc(&a);
    printf("%d", a); // Might print 1!
}
```

However, compilers will usually detect this:
```

/**
 * @ensure const(i)
 */
func void badFunc(int* i)
{
    *i = 2; // <- Compiler error: violating post condition const(i)
}
```

### Pure in detail

The `pure` annotation allows a program to make assumtions in regards of how the function treats global variables. Unlike for `const`, a pure function is not allowed to call a function which is known to be impure.

However, just like for `const` the compiler might not detect whether the annotation is correct or not! This program might compile, but will behave strangely:

```
int i = 0;

type SecretFunc func void();

func void badFunc()
{
    i = 2;
}

SecretFunc foo = nil;

/**
 * @pure
 */
func void lyingFunc()
{
    SecretFunc(); // The compiler cannot reason about this!
}

func void test()
{
    foo = &badFunc;
    i = 1;
    lyingFunc();
    printf("%d", a); // Might print 1!
}
```

However, compilers will usually detect this:

```
int i = 0;

func void badFunc()
{
    i = 2;
}

/**
 * @pure
 */
func void lyingFunc()
{
    badFunc(); // Error! Calling an impure function
}
```

Consequently circumventing "pure" annotations is undefined behaviour.


# Pre conditions for macros

Macros have an additional class of pre conditions, that are used to confirm that the values actually will work inside the macro body. This improves the error messages, since otherwise it would be hard to know if the error is in the implementation of the macro, or in the parameters. These are placed under the `@reqparse` annotation, or together with the `@require` annotations, surrounded by `parse()`. 

```
/**
 * @reqparse resource.open()
 * @require resource != nil, parse(void *x = resource.open())
 **/
macro openResource(resource)
{
    return resource.open();
}
```
