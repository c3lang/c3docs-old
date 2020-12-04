# Library

The standard library design is a sketch, no work has been done on this yet.


## lang

### macro scoped(..., @body)

Scopes a list of variables:

```
int a = 3;
double b = 1.0;

@scoped(a, b)
{
    a = 4;
    b = 1.2;    
}

// Prints a = 3, b = 1.0
printf("a = %d, b = %f\n", a, b);
```

This can be useful to push another allocator:

```
@scoped(mem::defaultAllocator)
{
    mem::defaultAllocator = myArenaAllocator;
    
    // This will now use myArenaAllocator:
    doSomethingThatAllocates();
}
// The default allocator is restored here.
```

### Other macros:

* `max(a, b)` maximum of two values using >
* `min(a, b)` minimum of two values using <
* `swap($a, $b)` swap two variables using `=` and a temporary variable.

## mem

Mem contains memory allocators

### Globals

* `Alloc* defaultAlloc`
* `Alloc* tempAlloc`
````

### allocators

* `RingAlloc` ring buffer allocator
* `ArenaAlloc` arena allocator

## Encodings

- 

## Ref counting

Any struct can enable ref counting by including the RefCount struct:

```
import std::refcount;

struct Person
{
    RefCount rc inline;
    char[] name;
}

func void test()
{
    Person* person = malloc(sizeof(Person));
    person.initRC(&free);
    printf("RC = %d\n", person.refCount); // Prints 1
    person.retain();
    printf("RC = %d\n", person.refCount); // Prints 2
    person.release(); 
    printf("RC = %d\n", person.refCount); // Prints 1
    person.release(); // Will call free(person)        
}
```
