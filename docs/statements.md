# Statements

## Volatile section

Volatile sections replace volatile variables. 

```
func void test()
{
    v = 0;
    for (int i = 0; i < 100; i++)
    {
        @volatile
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
x = @volatile(v = 1);
```

