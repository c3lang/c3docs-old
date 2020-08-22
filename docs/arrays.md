# Arrays

Arrays has a central role in programming. C3 offers 3 types of arrays:

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

## Variable arrays

Variable arrays are specially allocated arrays that are prefixed with both a size and a capacity. If allocated on the heap, they cannot expand, but heap allocated variable arrays will do so automatically using whatever memory allocator was used to create it. 

*THIS PART WILL BE SUBJECT TO REVISION*

```
int[*] x = @malloc(int[*]);
x += 10;
x += 11;
x.size; // => 2
x[1]; // => 11
```

A variable array can implicitly convert to a pointer:

```
int[*] x = @malloc(int[*]);
int *z = x;
```

A variable array's reference is not *stable* under expansion, so any alias may be invalidated if appending occurs.

```
int[*] x = @malloc(int[*]);
int[*] *y = &x;
int *z = x;
for (int i = 0; i < 33; i++)  x += 10;
assert(y != &x);
int *w = x;
assert(z != w);
```

Assigning a fixed array to a variable array will copy the contents:

```
int[3] b = { 1, 2, 3};
int[*] a = @malloc(int[*]);
a = b;
```

#### Built-in functions on variable arrays

###### pop()
Removes the last element.
###### last()
Retrieves the last element.
###### remove(index)
Removes an element in the array in O(n) time.
###### capacity
Return the capacity of the array.
###### size
return the size of the array.
###### resize(size)
Resize the array to the given size.

## Subarray

The final type is the subarray `<type>[]`  e.g. `int[]`. A subarray is a view into either a fixed or variable array. Internally it is represented as a struct containing a pointer and a size. Both fixed and variable arrays may be converted into slices, and slices may be implicitly converted to pointers:
    
```
int[4] a = { 1, 2, 3, 4};
int[] b = &a; // Implicit conversion is always ok.
int[4] c = cast(b as int[4]); // Will copy the value of b into c.
int[4]* d = cast(b as int[4]); // Equivalent to d = &a
int[*] e = @malloc(int[]);
b.size; // Returns 4
e.size; // Returns 0
e += 1;
e += 2;
e.size; // Returns 2 
int* f = b; // Equivalent to e = &a
f = d; // implicit conversion ok.
f = e; // implicit conversion ok.
d = e; // ERROR! Not allowed
d = cast(e as int[4]*); // Fine
b = e; // Implicit conversion ok
```

### Slicing arrays

It's possible to use a range syntax to create subarrays from pointers, arrays, vararrays and other subarrays. The usual syntax is `arr[<start index>..<end index>]`. The end index is not included in the final result.
    
```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[0..5]; // The whole array as a slice.
int[] c = a[1..3]; // { 20, 50 }
```

It's possible to omit the first and last index, in which case the start and the len is inferred. Note that omitting the last index is not allowed for pointers.

The following are all equivalent:

```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[0..5];
int[] c = a[..5];
int[] d = a[0..];
int[] e = a[..];
```

One may also slice from the end. Again this is not allowed for pointers.

```
int[5] a = { 1, 20, 50, 100, 200 };
int[] b = a[1..^1]; // { 20, 50, 100 }
int[] c = a[^3..]; // { 50, 100, 200 }
```

One may also use assign to slices:

```
int[3] a = { 1, 20, 50 };
a[1..3] = 0; // a = { 1, 0, 0}
```

Or copy slices to slices:

```
int[3] a = { 1, 20, 50 };
int[3] b = { 2, 4, 5 }
a[1..3] = b[0..2]; // a = { 1, 2, 4}
```

Copying overlapping ranges, e.g. `a[1..3] = a[0..2]` is undefined behaviour.

    
### Conversion list

|  |       int[4] | int[*] | int[] | int[4]* | int* |
|:-:|:-:|:-:|:-:|:-:|:-:|
| int[4] | copy | - | - | - | - |
| int[*] | copy | assign | cast | cast | cast |
| int[] | - | assign | assign | assign | - |
| int[4]* | - | cast | cast | assign | cast |
| int* | - | assign | assign | assign | assign |

Note that all casts above are inherently unsafe and will only work if the type cast is indeed compatible.

For example:

```
int[4] a;
int[4]* b = &a;
int* c = b;
// Safe cast:
int[4]* d = cast(c as int[4]*); 
// Faulty code with undefined behaviour:
int[*] e = cast(c as int[*]*); 
```

```
int[*] a = @malloc(int[*]);
a += 11;
a += 12;
a += 13;
// Safe (2 is less than the dynamic size of a)
int[2]* b = cast(a as int[2]*);
// Faulty code with undefined behaviour
// (4 is greater than the dynamic size of a)
int[4]* c = cast(a as int[4]*);
```

#### Internals

Internally the layout of a slice is guaranteed to be `struct { <type>* ptrToArray; usize arraySize; }`.

There is a built in struct `__ArrayType_C3` which has the exact data layout of the fat array pointers. It is defined to be

```
struct __ArrayType_C3 
{ 
    void* ptrToArray;
    usize arraySize;
}
```

## Iteration over arrays

Slices, fixed and variable arrays may all be iterated over using `for (Type x : array)`:

```
int[4] a = { 1, 2, 3, 5 };
for (int x : a)
{
    ...
}
```

It is possible for any type to get this iteration by implementing the macro `iterator(<Type> *type : value)`:

```
struct Vector
{
    usize size;
    int* elements;
}

macro Vector.iterator(Vector *vector : value)
{
    for (int i = 0; i < vector.size; i++)
    {
        yield value;
    }
}

Vector v = Vector.new();
v.add(3);
v.add(7);

// Will print 3 and 7
for (int i : v)
{
    printf("%d\n");
}
```