# Modules

C3 groups functions, types, variables and macros into namespaces called modules. All C3 files must start with the `module` keyword, specifying the module. A module can consist of multiple files, e.g.

`file_a.c3`

```
module foo;

/* ... */
```

`file_b.c3`

```
module foo;

/* ... */
```

`file_c.c3`

```
module baz;

/* ... */
```

Here `file_a.c3` and `file_b.c3` belong to the same module, **foo** while `file_c.c3` belongs to to **bar**.

## Details

Some details about the C3 module system:

- Modules are not nested, there only a single level to the name.
- Module names must be alphanumeric lower case letters plus the underscore character: `_`.
- Module names are limited to 31 characters.

## Importing modules

Importing a module uses the `import` keyword. Imports have file scope, so consequently if `file_a.c3` imports the module `networking`, then `file_b.c3` cannot use those symbols unless it also imports `networking`.

`file_a.c3`
```
module foo;

//import bar and stdio
import bar;
import stdio;

/* ... */
```

`file_b.c3`
```
module foo;

//import bar and networking imported, but not storage
import bar;
import networking;

/* ... */
```

### Named imports

It is often convenient to alias the module name (affecting the current file only) to a shorter alias.

```
module foo;

import extended_filesystems_io as fs;
```

The code in the file can now use the `fs` instead of the longer name. However, both names remain valid in the file scope.

### Local imports

In many cases prefixes can become cumbersome. It is therefore also possible to use the `local` keyword to avoid the prefix completely.

```
import networking as net local;
import filesystem local;

// Equivalent
filesystem.doSomething();
doSomething();

// Equivalent
net.connect();
networking.connect();
connect();
```

In the case where a symbol would be ambiguous, for example if both `networking` and `filesystem` would have an `open()` function, then the prefix is still mandatory.

## Visibility

All files in the same module share the same global declaration namespace. However, by default a function is not visible outside the module. To make the symbol visible outside the module, use the keyword `public`.

```
module foo;

public func void init() { .. }

func void open() { .. }
```

In this example, the other modules can use the init() function after importing foo, but only files in the foo module can use open(), as it isn't specified as public.
