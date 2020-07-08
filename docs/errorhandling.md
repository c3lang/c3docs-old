## Error Handling

Unlike usual exception handling, errors in C3 build on normal returns. A function returning errors add a `!` to the return type. In C3 this called a "failable" type.

### Error returns

From C, a function returning an error value will appear as an out parameter – if the function returns a union of error codes – or as the return parameter if the function would a single enum.:

C3 code:
```
func int! getValue();
```

Corresponding C code:
```c
int getValue(Error *error);
```

The `int!` here is the failable return type, which is a tagged union: it might hold either the error or an int.

```
// Open a file, we will get a failable:
// Either a File* or an error.
File*! file = openFile("foo.txt");

// We can extract the error using "catch"
catch (err = file)
{
    // Might print "Error was FILE_NOT_FOUND"
    printf("Error was %s\n", err.name()); 
    
    // Might print "Error was FileError.FILE_NOT_FOUND"
    printf("Error was %s\n", err.fullName()); 
    
    // Might print "Error code: 931938210"
    printf("Error code: %ull\n", cast(err, ulong)); 
    return;
}

// We can also just execute of success:
File*! file2 = openFile("bar.txt");

// Only true if there is no error.
if (file2)
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
    catch (err = k)
    {
        // The error may be from fooMayError
        // or save!
    }    
)
```

If a `catch` returns or jumps out of the current scope in a different way, then the variable becomes
unwrapped to it's non-failable type. 

#### Some simple examples.

##### Defining an error

Errors may either be flat or contain additional data.

```
error FileNotFoundError;
error ParseError
{
    int line;
    int col;
}
```

##### Returning an error

Returning an error looks like a normal return but with the `!`

```
func void! findFile()
{
    if (File.doesFileExist("foo.txt")) return FileNotFoundError!;
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
    catch (res)
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
    try (res)
    {
        printf("I found the file\n");
    }
}
```

##### Catching some errors

```
func void! findFileAndParse2()
{
    catch (err = findFileAndParse())
    {
        case FileNotFoundError:
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

#### Check for error or success

`try` and `catch` without a statements returns a boolean true / false:

```
func int testResult()
{
    int! result = getValue();
    if (try(result))
    {
        printf("Success!\n");
    }
    if (catch(result))
    {
        printf("Failure\n");
    }
}
```

