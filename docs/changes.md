# Change log

##### Revision 2022-03-04
- Updates to $sizeof.
- Addition of $eval / $evaltype.
- Removal of $unreachable.

##### Revision 2022-02-16
- Updates to imports.
- Updates to project files.

##### Revision 2022-02-09
- Major revision to bring everything up to date.

##### Revision 2021-10-20
- `func` replaced by `fn`
- Compound literal now `Type { ... }` like C++.
- Update of conversion rules
- New error syntax


##### Revision 2021-08-27
- Updated reflection functionality.
- Added documentation for multi-line strings.
- Added documentation for base64 and hex array literals.

##### Revision 2021-08-12
- Updated error type and error handling with try/catch
 
##### Revision 2021-07-13
- Added nesting to /* ... */ removed /+ ... +/
- Added primer.

##### Revision 2021-06-20
- Updated array layout.
- Revised macros for foreach.
- Removed old generic functions. 
- Added new ideas around generic, macros
- Changed macro body definition syntax.
- Introduced both $for and $foreach.

##### Revision 2021-05-31
- Removal of vararray type.
- Updated user defined attributes.
- Removed incremental arrays.
- Added information on `define`.
- Added private modules and import.

##### Revision 2021-05-18
- Change cast to (type)(expression)

##### Revision 2021-05-08
- Added rationale for some changes from C.
- Updated undefined and [undefined behaviour](../undefinedbehaviour).
- Removed many of the fine grained module features.
- Removed "local" visibility in [modules](../modules).
- All modules are now distinct, parent modules do not have any special access to sub modules.
- Added `as module` imports.

##### Revision 2021-04-05
- "next" is now "nextcase".
- Added link to the C3 discord.
- The [conversions](../conversion) page updated with new conversion rules.
- Updated compound literal syntax.
- Removed [undefined behaviour](../undefinedbehaviour) behaviour on integer overflow and added a list of unspecified behaviour.

##### Revision 2020-12-23
- Updated slice behaviour.
- Updated expression block syntax.  
- Added link to specification-in-progress.

##### Revision 2020-12-04
- Local variables are implicitly zero.
- Removed in-block declarations.
- Changed struct member initialization syntax.
- Changed named parameter syntax.
- Updated on macro syntax.
- Removed built in c types.

##### Revision 2020-08-22
- Added slice operations.
- Changed cast syntax to `cast(<expr> as <type>)`.
    
##### Revision 2020-07-08
- Additions to [error handling](../errorhandling).
- Introduction of labelled `nextcase`, `break` and `continue`.
- Removal of `goto`.

##### Revision 2020-06-17
- Alternate casts in [idea](../ideas).
- Method functions simply renamed to "method".
- Completely revised [error handling](../errorhandling).

##### Revision 2020-04-23
- Updated error handling, adding try-else-jump and changed how errors are passed.
- Included [reflection](../reflection) page

##### Revision 2020-03-30
- Added Odin and D to comparisons.
- Updated text on how to contribute.
- Updated the example on undefined behaviour.
- Updated text on conversions.
- Moved double -> float conversion to "ideas"
- Fixed some typos.

##### Revision 2020-03-29
- Type inference for enums.
- Included [macro](../macros) page.
- Corrected precedence rules with `try` and `@`.
- Type functions.
- Managed variables back to ideas.
- Volatile moved back to ideas.
- Removed implicit lossy signed conversions.
- Introducing safe signed-unsigned comparisons.
- "Function block" renamed "expression block".
- `@` sigil removed from macros and is only used with macro invocations.
- Changed cast syntax from `@cast(Type, var)` to `cast(var, Type)`

##### Revision 2019-12-26
- Added module versioning system [idea](../ideas). 
- Fleshed out polymorphic functions.
- Unsigned to signed promotion mentioned in "changes from C"

##### Revision 2019-12-25
- Changes how generic modules work.
- Switched so that vararrays use `Type[*]` and sub arrays use `Type[]`.
- Added sub module granularity, partial imports (only importing selected functions and types), removal of `local`, extended aliasing. See [modules](../modules).
- Updated "changes from C" with removal of multiple declarations.

##### Revision 2019-12-11
- Updated the [setup](../setup) page.

##### Revision 2019-12-03
- Added page on [conversions](../conversion).
- Added page on [undefined behaviour](../undefinedbehaviour).

##### Revision 2019-11-01
- Updated "changes from C" with the lack of array decays.
- Added FourCC to the language 
- Added name alias to ideas
- Added align asserts to ideas
- Added built in tests to ideas
- Added [arrays page](../arrays)
- Added function blocks to [statements page](../statements).
- Added [expressions page](../expressions).
- Added [variables page](../variables).
- Moved managed pointers from idea to the [variables page](../variables).

##### Revision 2019-09-30

- Removed references (non nullable pointers)
- Removed idea with aliasing in import

##### Revision 2019-08-14

- Compile time run-include and include ideas.
- New module system idea.

##### Revision 2019-08-14

- Namespace separator changed to `::` instead of `.` to simplify parsing. 
- Added FourCC, Macro text interpolation to ideas.
- Added Yacc grammar (incomplete)
- Added "attribute" keyword. 
- Changed type alias declaration to use `typedef ... as ...`. 
- Introduced `type` operator. 
- Added section about attributes.

##### Revision 2019-08-02

- Added error example.
- Added generics example.
- Added method function example.
- Added idea implicit method functions
- Expanded the types page somewhat.

##### Revision 2019-07-30

- Added default and named arguments to the [functions page](../functions).
- Added varargs to the [functions page](../functions).
- Added idea about hierarchal memory.
- Added idea of raw dynamic safe arrays & strings.
- Volatile sections are no longer prefixed by '@'
- Added idea regarding c3 interop
- Added [page about c interop](../cinterop).
- Removed `c_ichar` and `c_uchar` types as they are redundant.
- Updates to keywords on the [grammar page]()../syntax).

##### Revision 2019-07-27

- Updated grammar with keywords.
- Added the [docs & comments](../comments) page.
- Updated the [pre and post conditions](../preconditions).

##### Revision 2019-07-24

- Idea: typed varargs.
- Added "pure" [post condition](../preconditions)
- Updated c3c commands.
- Removed the `type` keyword for defining union/struct/enum/error.

##### Revision 2019-07-23

- Added to [generic functions](../generics) examples for [] and []=
- Developed ideas about vectors in the [idea section](../ideas).
- Defined 2's complement for signed integers. 
- Idea: Managed pointers.
- Updated [naming rules](../naming) for types.
- Added more [naming rules](../naming) + examples of them.
- Removed "defer on function signatures" from ideas.
- Removed "managed qualifier" from ideas.
- Removed "defer sugar" from ideas.
- Removed "built in dynamic arrays" from ideas.
- Added [library](../library) section.
- Added more about [pre and post conditions](../preconditions).

##### Revision 2019-07-22

- Added "Design Principles" to the index page.

##### Revision 2019-07-21

- "return" rather than function name is used in post conditions. See [Functions](../functions#pre-and-post-conditions)
- Added "@include" macro for textual includes. See [Modules](../modules#textual-includes).
- Files to without `module` for single file compilations is now ok as a special case. See [Modules](../modules)
- Added cone style array idea to the [idea section](../ideas).
- Added idea about defer on error to the [idea section](../ideas).
- Added idea for aliasing generic structs in the import to the [idea section](../ideas).
- Added idea for changing automatic signed <-> unsigned conversion to the [idea section](../ideas).
- Added [Changes from C](../changesfromc) and [Statements](../statements) sections.
- Removal of `volatile`. See [Changes from C](../changesfromc) and [Statements](../statements)
- Removal of `const` See [Changes from C](../changesfromc)


