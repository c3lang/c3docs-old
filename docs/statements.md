# Statements

Statements largely work like in C, but with some additions.


## Expression blocks

*NOTE: This syntax may be subject to change*

Expression blocks (delimited using `{| |}`) are compound statements that opens its own function scope. Jumps cannot be done into or out of a function block, and `return` exits the block, rather than the function as a whole.

The function below prints `World!`

```
func void test()
{
    int a = 0;
    {|
        if (a != 0) return;
        printf("Hello ");
        return;
    |};
    printf("World!\n");
}
```

Expression blocks may also return values:

```
func void test(int x)
{
    int a = {|
        if (x > 0) return x * 2;
        if (x == 0) return 100;
        return -x;
    |};            
    printf("The result was %d\n", a);
}
```

## Labelled break and continue

Labelled `break` and `continue` lets you break out of an outer scope. Labels can be put on `if`, 
`switch`, `catch`, `while` and `do` statements.
   
```
func void test(int i)
{
    if FOO: (i > 0)
    {
        while (1)
        {
            printf("%d\n", i);
            // Break out of the top if statement.
            if (i++ > 10) break FOO;
        }
    }
}
```

## Do-without-while

Do-while statements can skip the ending `while`. In that case it acts as if the `while` was `while(0)`:

```
do 
{
    printf("FOO\n");
} while (0);

// Equivalent to the above.
do 
{
    printf("FOO\n");
}
```

## Nextcase and labelled nextcase

The `nextcase` statement is used in `switch` and `catch` to jump to the next statement:

```
switch (i)
{
   case 1:
     doSomething();
     nextcase; // Jumps to case 2
   case 2:
     doSomethingElse();
}
```

It's also possible to use `nextcase` with an expression, to jump to an arbitrary case:

```
switch (i)
{
    case 1:
        doSomething();
        nextcase 3; // Jump to case 3
    case 2:
        doSomethingElse();
    case 3:
        nextcase rand(); // Jump to random case
    default:
        printf("Ended\n");
}  
```

Which can be used as structured `goto` when creating state machines.