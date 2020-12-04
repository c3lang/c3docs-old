# Changes from C

Although C3 is trying to improve on C, this does not only mean addition of features, but also removal, or breaking changes:

##### No mandatory header files

There is a C3 interchange header format for declaring interfaces of libraries, but it is only used for special applications.

##### Removal of the old C macro system

The old C macro system is replaced by a new C3 macro system.

##### Import and modules

C3 uses module imports instead of header includes to link modules together.

##### Member access using `.` even for pointers

The `->` operator is removed, access uses dot for both direct and pointer access. Note that this is just single access: to access a pointer of a pointer (e.g. `int**`) an explicit dereference would be needed.

##### Different operator precedence

Notably bit operations have higher precedence than +/-, making code like this: `a & b == c` evaluate like `(a & b) == c` instead of C's `a & (b == c)`. See the page about [precedence rules](../precedence).

##### Removal of the const type qualifier

The const qualifier is only retained for actual constant variables. C3 uses a special type of [post condition](../preconditions) for functions to indicate that they do not alter in parameters.

```
/**
 * This function ensures that foo is not changed in the function, nor is bar.x altered.
 * @ensure const(foo), const(bar.x)
 **/
func void test(Foo* foo, Bar* bar)
{
    bar.y = foo.x;
    // bar.x = foo.x - compile time error!
    // foo.x = bar.y - compile time error!
}
```

##### Fixed arrays do not decay and have copy sematics

C3 has three different array types. Variable arrays and slices decay to pointers, but fixed arrays are value objects and do not decay.

```
int[3] a = { 1, 2, 3 };
int[4]* b = &a; // No conversion
int* c = a; // ERROR
int* d = &a; // Valid implicit conversion
int* e = b; // Valid implicit conversion
int[3] f = a; // Copy by value!
```

##### Removal of multiple declaration syntax

Only a single declaration is allowed per statement in C3:

```
int i, j; // ERROR
int a;    // Fine
```

In conditionals, a special form of multiple declarations are allowed but each must then provide its type:

```
for (int i = 0, int j = 1; i < 10; i++, j++) { ... }
```

##### Integer promotions rules and safe signed-unsigned comparisons

Promotion rules for integer types are different from C. C3 only does widening to the largest possible type and so implicit conversion will only occur where it is lossless (read more on the [conversion page](../conversion). C3 also adds *safe signed-unsigned comparisons*, this means that comparing signed and unsigned values will always yield the correct result:

```
// The code below will print "Hello C3!" on C3 and "Hello C!" in C.
int i = -1;
uint j = 1;
// int z = i + j; <- Error, explicit cast needed.
if (i < j)
{
  printf("Hello C3!\n");
}
else
{
  printf("Hello C!\n");
}
```

##### Goto removed

`goto` is removed and replaced with labelled `break` and `continue` together with the `next` statement that allows you to jump between cases in a `switch` statement.

##### Locals variables are implictly zeroed

In C global variables are implicitly zeroed out, but local variables aren't. In C3 local variables are zeroed out by default, but may be explicitly undefined to get the C behaviour.