# Change log

Current revision made 2019-08-14.

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


