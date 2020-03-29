# Statements

Statements largely work like in C, but with some additions.


## Expression blocks

Expression blocks (delimited using `({ })`) are compound statements that opens its own function scope. Jumps cannot be done into or out of a function block, and `return` exits the block, rather than the function as a whole.

The function below prints `World!`

```
func void test()
{
    int a = 0;
    ({
        if (a != 0) return;
        printf("Hello ");
        return;
    });
    printf("World!\n");
}
```

Expression blocks may also return values:

```
func void test(int x)
{
    int a = ({
        if (x > 0) return x * 2;
        if (x == 0) return 100;
        return -x;
    });            
    printf("The result was %d\n", a);
}
```


