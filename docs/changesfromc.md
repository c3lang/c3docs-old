# Changes from C

Although C3 is trying to improve on C, this does not only mean addition of features, but also removal, or breaking changes:

##### No mandatory header files

There is a C3 interchange header format for declaring interfaces of libraries, but it is only used for special applications.

##### Removal of the old C macro system

The old C macro system is replaced by a new C3 macro system.

##### Import and modules

C3 uses module imports instead of header includes to link modules together.

##### Member access using `.` even for pointers

The `->` operator is removed, access uses dot for both direct and pointer access. Note that this is just single access: to access a pointer of a pointer (e.g. `int**`) an explicit dereference would be needed.

##### Different operator precedence

Notably bit operations have higher precedence than +/-, making code like this: `a & b == c` evaluate like `(a & b) == c` instead of C's `a & (b == c)`. See the page about [precedence rules](/precedence/).

##### Removal of the volatile type qualifier

The volatile type qualifier is replaced by volatile sections. A volatile section is guaranteed to not be optimized away.

```
\\ C volatile
void test()
{
    volatile v = 0;
    for (int i = 0; i < 100; i++)
    {
        // Usually this would be optimized away,
        // but volatile will ensure it is executed.
        v = 1; 
    }
}

\\ C3
func void test()
{
    v = 0;
    for (int i = 0; i < 100; i++)
    {
        // Everything in the block
        // will avoid optimization
        @volatile
        {
            v = 1; 
        }
    }
}
```


