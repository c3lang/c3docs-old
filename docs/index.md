# What is C3?

C3 is a systems language based on C. It tries to be a short step up from C rather than to replace it with a new paradigm or completely new syntax. 

The syntax and improvements on C inherits from the [C2 lang project](http://www.c2lang.org/) by [Bas van den Berg](https://github.com/bvdberg). It goes substantially further in regards to error handling, macros, generics and strings.

Last updated: [2019-11-01](changes).

## Planned features

- Transpile-compile using Clang, GCC or TCC
- Compile directly using LLVM
- C to C3 conversion (for a subset of C)
- Module system
- Generic modules
- Zero overhead errors
- Struct subtyping
- Built-in safe arrays
- Zero cost simple gradual & opt-in pre/post conditions.
- High level containers and string handling

## Design principles

- Procedural "get things done"-type of language.
- Try to stay close to C - only change where truly needed.
- Flawless C integration.
- Learning C3 should be easy for a C programmer.
- Dare add convienences if the value is great.
- Data is inert.
- Avoid "big ideas".
- Avoid the kitchen sink language trap.

#### Thank yous

Special thank yous to: Bas van der Berg (Author of [C2](http://www.c2lang.org), Jon Goodwin (Author of [Cone](http://cone.jondgoodwin.com)), Andrey Penechko and the people on the /r/ProgrammingLanguages Discord. 