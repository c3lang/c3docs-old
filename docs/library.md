# Library

## Ref counting

Any struct can enable ref counting by including the RefCount struct:

```
import refcount local;

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
    
    // More briefly, use @rcmalloc
    Person@ person2 = @rcmalloc(Person); 
    // Above will call malloc & free and return a managed pointer
}
```

