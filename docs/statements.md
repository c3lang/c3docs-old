# Statements

Statements largely work like in C, but with some additions.

## Volatile section

Volatile sections replace volatile type qualifiers on variable types.

```
func void test()
{
    v = 0;
    for (int i = 0; i < 100; i++)
    {
        volatile
        {
            v = 1; 
        }
    }
}
```

Note that volatile sections may also be used as expressions:

```
// The v = 1 assignment may not be optimized away,
// But the assignment to x can be. 
x = volatile(v = 1);
```

## Function blocks

Function blocks (delimited using `({ })`) are compound statements that opens its own function scope. Jumps cannot be done into or out of a function block, and `return` exits the block, rather than the function as a whole.

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

Function blocks may also return values:

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


