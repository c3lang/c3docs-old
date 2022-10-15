# What is C3?

C3 is a system programming language based on C. It is an evolution of C enabling the same paradigms and 
retaining the same syntax as far as possible.

C3 started as an extension of the [C2 language](http://www.c2lang.org/) by [Bas van den Berg](https://github.com/bvdberg). 
It has evolved significantly, not just in syntax but also in regard to error handling, macros, generics and strings.

The C3 compiler can be found on github: [https://github.com/c3lang/c3c](https://github.com/c3lang/c3c).

Last updated: [Revision 2022-10-14](changes).

## Features

- Full C ABI compatibility  
- Module system 
- Generic modules
- Zero overhead errors
- Struct subtyping 
- Semantic macro system
- Safe array access using sub arrays
- Zero cost simple gradual & opt-in pre/post conditions
- High level containers and string handling
- C to C3 conversion (for a subset of C) *TODO*
- LLVM backend

## Design principles

- Procedural "get things done"-type of language.
- Stay close to C - only change where there is a significant need.
- Flawless C integration.
- Learning C3 should be easy for a C programmer.
- Dare add conveniences if the value is great.
- Data is inert and zero is initialization.
- Avoid "big ideas".
- Avoid the kitchen sink language trap.

#### Thank yous

Special thank yous to: Bas van der Berg (Author of [C2](http://www.c2lang.org)), Jon Goodwin (Author of [Cone](http://cone.jondgoodwin.com)) and Andrey Penechko (Author of [Vox](https://github.com/MrSmith33/vox)).
