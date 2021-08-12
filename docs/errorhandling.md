## Error Handling

Unlike usual exception handling, errors in C3 build on normal returns. A function returning errors add a `!` to the return type. In C3 this called a *failable* type.

### Error returns

For C the interface to C, the error is returned as the normal value, and any return is instead returned on as an out parameter.

C3 code:
```
func int! getValue();
```

Corresponding C code:
```c
Error getValue(int *value);
```

The `int!` here is the failable return type, which is a tagged union: it might hold either the error or an int.

*Note: reflection methods on errors are not complete yet.*
```
// Open a file, we will get a failable:
// Either a File* or an error.
File*! file = openFile("foo.txt");

// We can extract the error using "catch"
if (catch err = file)
{
    // Might print "Error was FILE_NOT_FOUND"
    printf("Error was %s\n", err.name()); 
    
    // Might print "Error was FileError.FILE_NOT_FOUND"
    printf("Error was %s\n", err.fullName()); 
    
    // Might print "Error code: 931938210"
    printf("Error code: %ull\n", (ulong)(err)); 
    return;
}

// We can also just execute of success:
File*! file2 = openFile("bar.txt");

// Only true if there is no error.
if (try file2)
{
    // Inside here file2 is a regular File*
}
```

A function, method or macro call with one or more parameters will only execute if the failable has no error. This makes error returns composable. 

```
func int! fooMayError() { ... }
func int mult(int i) { ... }
func int! save(int i) { ... }

func void test()
(
    int! i = fooMayError();
    
    // "mult" is only called if "fooMayError()"
    // returns a non error result.
    int! j = mult(fooMayError());
    
    int! k = save(mult(fooMAyError()));
    if (catch err = k)
    {
        // The error may be from fooMayError
        // or save!
    }    
)
```

#### Implicit unwrapping

If a `if-catch` returns or jumps out of the current scope in some way, then the variable becomes
unwrapped to it's non-failable type in that scope:

```
int! i = fooMayError();
    
if (catch i)
{
    return;
}

// i is now considered an int:

if (i > 10) doSomething();
```

### Some simple examples.

##### Defining an error

An error if effectively an enum, and is defined in the same way:

```
errtype IoError
{
    FILE_NOT_FOUND,
    FILE_NOT_READABLE,
}    
```

##### Returning an error

Returning an error looks like a normal return but with the `!`

```
func void! findFile()
{
    if (File.doesFileExist("foo.txt")) return IoError.FILE_NOT_FOUND!;
    /* ... */
}
```

##### Calling a function automatically returning any error

The `!!` suffix will create an implicit return on error.

```
func void! findFileAndTest()
{
    findFile()!!;
    // Implictly:
    // catch (err = findFile()) return err!;
}
```

##### Catching errors

Catching an error and returning will implicitly unwrap the checked variable.

```
func void findFileAndNoErr()
{
    File*! res = findFile();    
    if (catch res)
    {
        printf("An error occurred!\n");
        return;
    }
    // res is implicitly unwrapped here.
    // and have an effective type of File* here.
}
```

##### Only do if no error

```
func void doSomethingToFile()
{
    void! res = findFile();    
    if (try res)
    {
        printf("I found the file\n");
    }
}
```

##### Catching some errors

```
func void! findFileAndParse2()
{
    if (catch err = findFileAndParse())
    {
        case IOError.FILE_NOT_FOUND:
            printf("Error loading the file!\n");
        default:
            return err;
    }
}
```


##### Default values

A function returning an error may be followed by an `else` and an expression. The call then executes and returns the expression.

```
func int testDefault()
{
    return getIntNumberOrFail() else -1;
}

// The above is equivalent to:

func int testDefault()
{
    int! i = getIntNumberOrFail();    
    catch (i) return -1;
    return i;
}

```

##### Default jump

The else can also be followed by a jump statement: `return`, `break` or `continue`.

```
func int testBreak(int times)
{
    int index;
    for (index = 0; i < times; i++)
    {
       callTest(index) else break; 
    }
    if (index < times)
    {
        printf("Aborted test at index: %d\n", index);
    }
    return index;
}
```


