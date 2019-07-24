## Error Handling

Unlike usual exception handling, errors in C3 build on normal returns. A function declaring errors using the `throws` keyword essentially returns a union containing either the return value or an 64 bit integer. A register or a flag is used to determine if the result is an error or a normal result.

Because there is no stack unwinding, error handling is completely deterministic and has no additional runtime cost.

The call of a function which returns an error _must_ be preceeded by the keyword `try`. 

### Error returns

From C, a function throwing an error value appears simply as a union:

C3 code:
```
func int getValue() throws
```

Corresponding C code:
```c
struct Result { char error; union { int res; uint64_t err_code; }; };
struct Result getValue();
```

It is possible to extract the error code and also store it, to retrieve it later:

```
try openFile("foo.txt");
catch (error err)
{
    // Might print "Error was FILE_NOT_FOUND"
    printf("Error was %s\n", @name(err)); 
    
    // Might print "Error was FileError.FILE_NOT_FOUND"
    printf("Error was %s\n", @describe(err)); 
    
    // Might print "Error code: 931938210"
    printf("Error code: %ull\n", @cast(ulong, err)); 
}
```

#### Some simple examples.

##### Defining an error set
```
error FileError
{
    FILE_NOT_FOUND,
    FILE_CANNOT_OPEN,
    PATH_DOES_NOT_EXIST,    
}
```

##### Throwing an error

```
func void findFile() throws
{
    if (File.doesFileExist("foo.txt")) throw FileError.FILE_NOT_FOUND;
    /* ... */
}
```

##### Declaring a function as throwing a specific set of errors

```
func void findFile() throws FileError
{
    /* ... */
}
```

##### Declaring a function to throw a union of errors
```
func void findAndParseFile() throws FileError, ParseError
{
    /* ... */
}
```

##### Calling a function that throws
```
func void findFileAndTest() throws
{
    try findFile();
}
```

##### Catching errors
```
func void findFileAndNoExcept()
{
    try findFile();
    
    catch (error err)
    {
        printf("An error occurred!\n");
        return;
    }
}
```

##### Catching error subsets
```
func void findFileAndParse2() throws ParseError
{
    try findFileAndParse();
    
    catch (FileError err)
    {
        printf("Error loading the file!\n");
        return;
    }
    
    // No catch for the ParseError, so it escapes.
}
```

##### Scoped error catching
```
func void testErrorScopes()
{
    {
        try findFileAndParse();
        
        catch (FileError err)
        {
            try sendAlarm("Error loading the file");
            catch (error err)
            {
                printf("Failed to send the alarm");
            }
            return;
        }        
    }
    
    try someOtherCall();
    catch (error err)
    {
        printf("Some other error\n");
    }
}
```


##### Default values
```
func int testDefault()
{
    return try getIntNumberOrFail() else -1;
}
```
