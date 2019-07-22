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

Post conditions are evaluated to make checks on the resulting state after passing through the function. There is a special post condition called "const condition" in the format `const(<parameter>)`, which guarentees that the memory pointed to is not altered within the scope of a function. The post condition uses the `@ensure` annotation. `return` is used to mark the return value from the function.


```
/**
 * @ensure const(foo), return > foo.x;
 **/
func uint checkFoo(Foo& foo)
{
    uint y = abs(foo.x) + 1;
    // If we had row: foo.x = 0, then this would be a compile time error.
    return y * abs(foo.x);
}
```

# Pre conditions for macros

Macros have an additional class of pre conditions, that are used to confirm that the values actually will work inside the macro body. This improves the error messages, since otherwise it would be hard to know if the error is in the implementation of the macro, or in the parameters. These are placed under the `@ensure-parse` annotation, or together with the `@ensure` annotations, surrounded by `parse()`. 

```
/**
 * @ensure-parse resource.open()
 * @ensure resource != nil, parse(void *x = resource.open())
 **/
macro openResource(resource)
{
    return resource.open();
}
```
