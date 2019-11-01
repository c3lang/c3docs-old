# Arrays

Arrays has a central role in programming. C3 offers 3 types of arrays:

## Fixed arrays

`<type>[<size>]` e.g. `int[4]`. These are treated as values and will be copied if given as parameter. Unlike C, the number is part of its type. Taking a pointer to a fixed array will create a pointer to a fixed array, e.g. `int[4]*`. 

Unlike C, fixed arrays do not decay into pointers, instead an int[4]* may be implicitly converted into an int*.

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

## Variable arrays

Variable arrays are specially allocated arrays that are prefixed with both a size and a capacity. If allocated on the heap, they cannot expand, but heap allocated variable arrays will do so automatically using whatever memory allocator was used to create it. 

```
int[] x = @malloc(int[]);
x += 10;
x += 11;
x.size; // => 2
x[1]; // => 11
```

A variable array can implicitly convert to a pointer:

```
int[] x = @malloc(int[]);
int *z = x;
```

A variable array's reference is not *stable* under expansion, so any alias may be invalidated if appending occurs.

```
int[] x = @malloc(int[]);
int[] *y = &x;
int *z = x;
for (int i = 0; i < 33; i++)  x += 10;
assert(y != &x);
int *w = x;
assert(z != w);
```

Assigning a fixed array to a variable array will copy the contents:

```
int[3] b = { 1, 2, 3};
int[] a = @malloc(int[]);
a = b;
```

#### Built-in functions on variable arrays

**pop()** removes the last element
**last()** retrieves the last element
**remove(index)** removes an element in the array
**capacity** return the capacity of the array
**size** return the size of the array
**resize(size)** resize the array to the given size

## Slices

The final type is the slice `<type>[:]`  e.g. `int[:]`. A slice is a view into either a fixed or variable array. Internally it is represented as a struct containing a pointer and a size. Both fixed and variable arrays may be converted into slices, and slices may be implicitly converted to pointers:
    
```
int[4] a = { 1, 2, 3, 4};
int[:] b = &a; // Implicit conversion is always ok.
int[4] c = @cast(int[4], b); // Will copy the value of b into c.
int[4]* d = @cast(int[4]*, b); // Equivalent to d = &a
int[] e = @malloc(int[]);
b.size; // Returns 4
e.size; // Returns 0
e += 1;
e += 2;
e.size; // Returns 2 
int* f = b; // Equivalent to e = &a
f = d; // implicit conversion ok.
f = e; // implicit conversion ok.
d = e; // ERROR! Not allowed
d = @cast(int[4]*, e); // Fine
b = e; // Implicit conversion ok
```
### Conversion list

|  |       int[4] | int[] | int[:] | int[4]* | int* |
|:-:|:-:|:-:|:-:|:-:|:-:|
| int[4] | copy | - | - | - | - |
| int[] | copy | assign | cast | cast | cast |
| int[:] | - | assign | assign | assign | - |
| int[4]* | - | cast | cast | assign | cast |
| int* | - | assign | assign | assign | assign |

#### Internals

Internally the layout of a slize is guaranteed to be `struct { <type>* ptrToArray; usize arraySize; }`.

There is a built in struct `__ArrayType_C3` which has the exact data layout of the fat array pointers. It is defined to be

```
struct __ArrayType_C3 
{ 
    void* ptrToArray;
    usize arraySize;
}
```

## Iteration over arrays

Slices, fixed and variable arrays may all be iterated over using `for ... in`:

```
int[4] a = { 1, 2, 3, 5 };
for (int x in a)
{
    ...
}
```

