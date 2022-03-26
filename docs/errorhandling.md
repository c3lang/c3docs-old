## Error Handling

Unlike usual exception handling, errors in C3 build on optional returns. 
An *optional result* works as a union containing either the *expected result* or an *optional result value*.


### Error returns

For C the interface to C3, the *optional result value* is returned as the regular value, while the return value 
is instead returned as an out parameter.

C3 code:
```
fn int! getValue();
```

Corresponding C code:
```c
OptEnum getValue(int *value);
```

The `int!` here is the optional result type, which either contains an optional result value or an int.

*Note: reflection methods on optionals are not complete yet.*
```
// Open a file, we will get an optional result:
// Either a File* or an error.
File*! file = openFile("foo.txt");

// We can extract the optional result value using "catch"
if (catch err = file)
{
    // Might print "Error was FILE_NOT_FOUND"
    printf("Error was %s\n", err.name()); 
    
    // Might print "Error was FileError.FILE_NOT_FOUND"
    printf("Error was %s\n", err.fullName()); 
    
    // Might print "Error code: 931938210"
    printf("Error code: %ull\n", (ulong)err); 
    return;
}

// We can also just execute of success:
File*! file2 = openFile("bar.txt");

// Only true if there is an expected result.
if (try file2)
{
    // Inside here file2 is a regular File*
}
```

A function, method or macro call with one or more parameters will only execute if the optional result has the *expected result*. 
This makes optional result returns composable. 

```
fn int! fooMayError() { ... }
fn int mult(int i) { ... }
fn int! save(int i) { ... }

fn void test()
(
    int! i = fooMayError();
    
    // "mult" is only called if "fooMayError()"
    // returns a non optional result.
    int! j = mult(fooMayError());
    
    int! k = save(mult(fooMAyError()));
    if (catch err = k)
    {
        // The optional result value may be from fooMayError
        // or save!
    }    
)
```

#### Implicit unwrapping

If a `if-catch` returns or jumps out of the current scope in some way, then the variable becomes
unwrapped to it's non-optional type in that scope:

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

##### Defining an optional result value

An result if effectively an enum, and is defined in the same way:

```
optenum IoError
{
    FILE_NOT_FOUND,
    FILE_NOT_READABLE,
}    
```


##### Returning an optional value result

Returning an optional result looks like a normal return but with the `!`

```
fn void! findFile()
{
    if (File.doesFileExist("foo.txt")) return IoError.FILE_NOT_FOUND!;
    /* ... */
}
```

##### Calling a function automatically returning any optional result

The `?` suffix will create an implicit return if the *expected result* is missing.

```
fn void! findFileAndTest()
{
    findFile()?;
    // Implictly:
    // catch (err = findFile()) return err!;
}
```

##### Panic on error

The `!!` will issue a panic if the *expected value* is missing.

```
fn void! findFileAndTest()
{
    findFile()!!;
    // Implictly:
    // catch (err = findFile()) panic("Unexpected error");
}
```

##### Catching optionals

Catching an optional and returning will implicitly unwrap the checked variable.

```
fn void findFileAndNoErr()
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

##### Only do if no optional value return

```
fn void doSomethingToFile()
{
    void! res = findFile();    
    if (try res)
    {
        printf("I found the file\n");
    }
}
```

##### Catching some optional value results

```
fn void! findFileAndParse2()
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

A function returning an optional type may be followed by an `??` and an expression. The expression on the right hand
side will be returned if the left hand side returns an optional result value.

```
fn int testDefault()
{
    return getIntNumberOrFail() ?? -1;
}

// The above is equivalent to:

fn int testDefault()
{
    int! i = getIntNumberOrFail();    
    catch (i) return -1;
    return i;
}

```
