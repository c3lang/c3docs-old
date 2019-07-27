# What is C3?

C3 a systems language based on C. It intends tries to be a short step up from C rather than to replace it with a new paradigm or completely new syntax. 

C3 is built on top of the [C2 lang project](http://www.c2lang.org/) by [Bas van den Berg](https://github.com/bvdberg). It goes substantially further in regards to error handling, macros, generics and strings. Although it might have some breaking changes, it can – at least currently – be considered a superset of C2.

Last updated: [2019-07-27](changes).

## Planned features

- Transpile to C
- Transpile-compile using Clang, GCC or TCC
- Compile directly using LLVM
- C to C3 conversion (for a subset of C)
- Module system
- Generic modules
- Zero overhead errors
- Struct subtyping
- Built-in safe arrays
- High level containers and string handling

## Design principles

- Procedural "get things done"-type of language.
- Try to stay close to C - only change where truly needed.
- Flawless C integration.
- Learning C3 should be easy for a C programmer.
- Dare violating the "close to metal" principle if the value is great.
- Not an object oriented language.
- Avoid "big ideas".
- Avoid the kitchen sink language trap.
