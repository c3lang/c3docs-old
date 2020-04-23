# Library

The standard library design is a sketch, no work has been done on this yet.


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
