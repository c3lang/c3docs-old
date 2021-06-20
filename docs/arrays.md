# Arrays

Arrays has a central role in programming. C3 offers 2 built-in types of arrays:

## Fixed arrays

`<type>[<size>]` e.g. `int[4]`. These are treated as values and will be copied if given as parameter. Unlike C, the number is part of its type. Taking a pointer to a fixed array will create a pointer to a fixed array, e.g. `int[4]*`. 

Unlike C, fixed arrays do not decay into pointers, instead an `int[4]*` may be implicitly converted into an `int*`.

```
// C
int foo(int *a) { ... }

int x[3] = { 1, 2, 3 };
foo(x);

// C3
func int foo(int *a) { ... }

int x[3] = { 1, 2, 3 };
foo(&x);
```

When you want to initialize a fixed array without specififying the size, use the [+] array syntax:

```
int[3] a = { 1, 2, 3};
int[+] b = { 4, 5, 6}; // Type inferred to be int[3]
```


## Subarray

The final type is the subarray `<type>[]`  e.g. `int[]`. A subarray is a view into either a fixed or variable array. Internally it is represented as a struct containing a pointer and a size. Both fixed and variable arrays may be converted into slices, and slices may be implicitly converted to pointers:
    
```
int[4] a = { 1, 2, 3, 4};
int[] b = &a; // Implicit conversion is always ok.
int[4] c = (int[4])(b); // Will copy the value of b into c.
int[4]* d = (int[4])(a); // Equivalent to d = &a
b.size; // Returns 4
e += 1;
int* f = b; // Equivalent to e = &a
f = d; // implicit conversion ok.
```

### Slicing arrays

It's possible to use a range syntax to create subarrays from pointers, arrays, vararrays and other subarrays. The usual syntax is `arr[<start index>..<end index>]`. The end index is included in the final result.
    
```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[0..4]; // The whole array as a slice.
int[] c = a[1..2]; // { 20, 50 }
```

It's possible to omit the first and last index, in which case the start and the len is inferred. Note that omitting the last index is not allowed for pointers.

The following are all equivalent:

```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[0..4];
int[] c = a[..4];
int[] d = a[0..];
int[] e = a[..];
```

One may also slice from the end. Again this is not allowed for pointers.

```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[1..^2]; // { 20, 50, 100 }
int[] c = a[^3..]; // { 50, 100, 200 }
```

One may also use assign to slices:

```
int[3] a = { 1, 20, 50 };
a[1..2] = 0; // a = { 1, 0, 0}
```

Or copy slices to slices:

```
int[3] a = { 1, 20, 50 };
int[3] b = { 2, 4, 5 }
a[1..2] = b[0..1]; // a = { 1, 2, 4}
```

Copying overlapping ranges, e.g. `a[1..2] = a[0..1]` is undefined behaviour.

    
### Conversion list

| | int[4] | int[] | int[4]* | int* |
|:-:|:-:|:-:|:-:|:-:|
| int[4] | copy | - | - | - |
| int[] | - | assign | assign | - |
| int[4]* | - | cast | assign | cast |
| int* | - | assign | assign | assign |

Note that all casts above are inherently unsafe and will only work if the type cast is indeed compatible.

For example:

```
int[4] a;
int[4]* b = &a;
int* c = b;
// Safe cast:
int[4]* d = (int[4]*)(c); 
int e = 12;
int* f = &e;
// Incorrect, but not checked
int[4]* g = (int[4]*)(f);
// Also incorrect but not checked.
int[] h = f[0..2];
```


#### Internals

Internally the layout of a slice is guaranteed to be `struct { usize arraySize; <type>* ptrToArray; }`.

There is a built in struct `__ArrayType_C3` which has the exact data layout of the fat array pointers. It is defined to be

```
struct __ArrayType_C3 
{ 
    usize arraySize;
    void* ptrToArray;
}
```

## Iteration over arrays

Slices, fixed and variable arrays may all be iterated over using `foreach (Type x : array)`:

```
int[4] a = { 1, 2, 3, 5 };
for (int x : a)
{
    ...
}
```

*NOTE THAT THE SYNTAX WILL BE REVISED*

It is possible for any type to get this iteration by implementing the method macros `_foreach` `_foreach_index`:

```
struct Vector
{
    usize size;
    int* elements;
}

macro Vector._foreach_index(Vector* vector, $by_ref, $reverse; @body(index, value))
{
    usize size = vector.size;
    $IndexTyoe = typeof(index);
    $if ($reverse):
        usize i = size;
        while (i-- >= 0)
        {
            $if ($by_ref):
                @body(($IndexType)(i), &vector.elements[i]));
            $else:
                @body(($IndexType)(i), vector.elements[i]));
            $endif;    
        }
    $else:
        for (usize i = 0; i < size; i++)
        {
            $if ($by_ref):
                @body(($IndexType)(i), &vector.elements[i]));
            $else:
                @body(($IndexType)(i), vector.elements[i]));
            $endif;    
        }
    $endif;    
}

Vector v;
v.add(3);
v.add(7);

// Will print 3 and 7
for (int i : v)
{
    printf("%d\n");
}
```
