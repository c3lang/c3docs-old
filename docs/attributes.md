# Attributes

Attributes are compile time annotations on functions, types, global constants and variables. Similar to Java annotations, a decoration may also take arguments. A attribute can also represent a bundle of attributes.

## Built in attributes

### `pure` (call)

Used to annotate a non pure function as "pure" when checking for conformance to `@pure` on 
functions.

### `packed` (struct, union, enum)

If used on a struct or enum: packs the type, including any components to minimum size. On an enum, it uses the smallest representation containing all its values.

### `section(name)` (var, fn)

Declares that a global variable or function should appear in a specific section.

### `inline` (fn)

Declares a function to always be inlined.

### `aligned(alignment)` (struct, union, var, fn)

This attribute sets the minimum alignment for a field or a variable.

### `noreturn` (fn)

Declares that the function will never return.

### `weak` (fn, var)

Emits a weak symbol rather than a global. 

### `opaque` (struct, union, enum)

Prevents the union or struct from being statically allocated by other modules. In the case of enums,
prevents the enum from being converted to a value.

## User defined attributes

User defined attributes are intended for conditional application of built-in attributes.
 
```
define @MyAttribute = @noreturn @inline;

// The following two are equivalent:
fn void foo() @MyAttribute { ... }
fn void foo() @noreturn @inline { ... }
```

A user defined attribute may also be completely empty:

```
define @MyAttributeEmpty = void;
```
