# Naming rules

As a basic rule, all identifiers are limited to a-z, A-Z, 0-9 and `_`. The initial character can not be a number. Furthermore, all identifiers are limited to 31 character.

### Structs, unions and enums

All user defined types must start with A-Z. For C-compatibility it's possible to alias the type to a C name using the attribute "cname".

### Variables and parameters

All variables and parameters *except for* global constant variables must start with a-z after any optional initial `_`. `___a` `fooBar` and `_test_` are all valid variable / parameter names. `_`, `_Bar`, `X` are not.

### Global constants

Global constants must start with A-Z after any optional initial `_`. `_FOO`, `BAR_FOO`, `X` are all valid global constants, `_`, `_bar`, `x` are not. 

### Modules

Module names are limited to a-z and underscore.

### Functions and macros

Functions and macros must start with a-z after any optional initial `_`.