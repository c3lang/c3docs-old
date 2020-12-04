# What is C3?

C3 is a systems language based on C. It wants to be evolution C, and enable the same paradigms and 
retain the same syntax (as far as possible).

C3 started as an extension of the [C2 language](http://www.c2lang.org/) by [Bas van den Berg](https://github.com/bvdberg). 
It has since departed substantially in regards to error handling, macros, generics and strings.

The C3 compiler can be found on github: [https://github.com/c3lang/c3c](https://github.com/c3lang/c3c).

Last updated: [Revision 2020-12-04](changes).

## Planned features

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

Special thank yous to: Bas van der Berg (Author of [C2](http://www.c2lang.org)), Jon Goodwin (Author of [Cone](http://cone.jondgoodwin.com)), Andrey Penechko and the people on the /r/ProgrammingLanguages Discord. 