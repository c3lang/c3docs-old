# Builtins

The compiler offers builtin constants and functions. Some are only available on certain targets. All builtins use the `$$`
name prefix.

## Builtin constants

These can all safely be used by the user.

#### $$LINE
The current line as an integer.

#### $$FUNC
The current function name, will return "<GLOBAL>" on the global level.

#### $$FILE
The current file name.

#### $$LINEREAL
Usually the same as $$LINE, but in case of a macro inclusion it returns the line in the macro rather than
the line where the macro was included.

## Builtin functions

These functions are not guaranteed to exist on all platforms. They are intended for use standard library
use, and typically the standard library has macros that wrap these builtins, so they should not be used on its own.

#### $$trap

Emits a trap instruction.

#### $$unreachable

Inserts an "unreachable" annotation.

#### $$stacktrace

Returns the current "callstack" reference if available. Compiler dependent.

#### $$volatile_store

Takes a variable and a value and stores the value as a volatile store.

#### $$volatile_load

Takes a variable and returns the value using a volatile load.

#### $$memcpy

Builtin memcpy instruction.

#### $$memset

Builtin memset instruction.

### Math functions

Functions $$ceil, $$trunc, $$sin, $$cos, $$log, $$log2, $$log10, $$sqrt, $$pow, $$min, $$max, $$exp, $$fma and $$fabs.

Can be applied to float vectors or numbers. Returns the same type.
