# Specification

*THIS SPECIFICATION IS UNDER DEVELOPMENT*

## Notation

The syntax is specified using Extended Backus-Naur Form (EBNF):

```
production  ::= PRODUCTION_NAME '::=' expression?
expression  ::= alternative ("|" alternative)* 
alternative ::= term term*
term        ::= PRODUCTION_NAME | TOKEN | set | group | option | repetition
set         ::= '[' (range | CHAR) (rang | CHAR)* ']'
range       ::= CHAR '-' CHAR 
group       ::= '(' expression ')'
option      ::= expression '?'
repetition  ::= expression '*'
```

Productions are expressions constructed from terms and the following operators, in increasing precedence:

```
|   alternation
()  grouping
?  option (0 or 1 times)
*  repetition (0 to n times)
```

Uppercase production names are used to identify lexical tokens. Non-terminals are in lower case. Lexical tokens are enclosed in single quotes ''.

The form `a..b` represents the set of characters from a through b as alternatives.

## Source code representation

A program consists of one or more _translation units_ stored in files written in the Unicode character set,
 stored as a sequence of bytes using the UTF-8 encoding. Except for comments and the contents of character and string literals, all input elements are formed only from the ASCII subset (U+0000 to U+007F) of Unicode.

A raw byte stream is translated into a sequence of tokens which white space and non-doc comments are discarded. Doc comments may optionally be discarded as well. The resulting input elements form the tokens that are the terminal symbols of the syntactic grammar.

### Lexical Translations

A raw byte stream is translated into a sequence of tokens which white space and non-doc comments are discarded. Doc comments may optionally be discarded as well. The resulting input elements form the tokens that are the terminal symbols of the syntactic grammar.

The longest possible translation is used at each step, even if the result does not ultimately make a correct program while another lexical translation would.

>Example: `a--b` is translated as `a`, `--`, `b`, which does not form a grammatically correct expression, even >though the tokenization `a`, `-`, `-`, `b` could form a grammatically correct expression.

### Line Terminators

The C3 compiler divides the sequence of input bytes into lines by recognizing *line terminators*

Lines are terminated by the ASCII LF character (U+000A), also known as "newline". A line termination specifies the termination of the // form of a comment.

### Input Elements and Tokens

An input element may be:

1. White space
2. Comment
3. Doc Comment
4. Token

A token may be:

1. Identifier
2. Keyword
3. Literal
4. Separator
5. Operator

A Doc Comment consists of:

1. A stream of descriptive text
2. A list of directive Tokens

Those input elements that are not white space or comments are tokens. The tokens are the terminal symbols of the syntactic grammar. Whitespace and comments can serve to separate tokens that might be tokenized in another manner. For example the characters `+` and `=` may form the operator token `+=` only if there is no intervening white space or comment.

### White Space

White space is defined as the ASCII horizontal tab character (U+0009), form feed character (U+000A), vertical tab (U+000B), carriage return (U+000D), space character (U+0020) and the line terminator character (U+000D).

```
WHITESPACE      ::= [ \t\f\v\r\n]
```

### Letters and digits

```
UC_LETTER       ::= [A-Z]
LC_LETTER       ::= [a-z]
LETTER          ::= UC_LETTER | LC_LETTER
DIGIT           ::= [0-9]
HEX_DIGIT       ::= [0-9a-fA-F]
BINARY_DIGIT    ::= [01]
OCTAL_DIGIT     ::= [0-7]
LC_LETTER_US    ::= LC_LETTER | "_"
UC_LETTER_US    ::= UC_LETTER | "_"
ALPHANUM        ::= LETTER | DIGIT
ALPHANUM_US     ::= ALPHANUM | "_"
UC_ALPHANUM_US  ::= UC_LETTER_US | DIGIT
LC_ALPHANUM_US  ::= LC_LETTER_US | DIGIT
```

### Comments

There are three types of regular comments:

1. `// text` a line comment. The text between `//` and line end is ignored.
2. `/* text */` block comments. The text between `/*` and `*/` is ignored. It has nesting behaviour, so for every `/*` discovered between the first `/*` and the last `*/` a corresponding `*/` must be found.

### Doc comments

1. `/** text **/` doc block comment. The text between `/**` and `**/` is optionally parsed using the doc comment syntactic grammar. A compiler may choose to read `/** text **/` as a regular comment.
2. `///` text doc line comment. The text between `///` and line end is optionally parsed using the doc comment syntactic grammar. A compiler may choose to read `///` as a regular comment.

### Identifiers

Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. 
The first character in an identifier must be a letter or underscore.

C3 has three types of identifiers: const identifiers - containing only underscore and upper-case letters, 
type identifiers - starting with an upper case letter followed by at least one underscore letter and regular identifiers, starting with a lower case letter.  

```
IDENTIFIER      ::=  "_"* LC_LETTER ALPHANUM_US*
CONST_IDENT     ::=  "_"* UC_LETTER UC_ALPHANUM_US*
TYPE_IDENT      ::=  "_"* UC_LETTER "_"* LC_LETTER ALPHANUM_US*
CT_IDENT        ::=  "$" IDENTIFIER
CT_CONST_IDENT  ::=  "$" CONST_IDENT
CT_TYPE_IDENT   ::=  "$" TYPE_IDENT
PATH_SEGMENT    ::= "_"* LC_LETTER LC_ALPHANUM_US*
```

### Keywords

The following keywords are reserved and may not be used as identifiers:


```
asm         any         anyfault
assert      attribute   break
case        cast        catch
const       continue    default
defer       def         do
else        enum        extern
errtype     false       fn
generic     if          import
inline      macro
module      nextcase    null
public      return      struct
switch      true        try
typeid      def
var         void
while

bool        quad        double      
float       long        ulong
int         uint        byte
short       ushort      char
isz         usz         float16
float128

$assert     $case       $default
$if         $for        $else
$elif       $if         $switch
$foreach    $endswitch  $endif      
$endforeach      
                  
```

### Operators and punctuation

The following character sequences represent operators and punctuation.

```
&       @       ~       |       ^       :
,       /       $       .       ;       )
>       <       #       {       }       -
(       )       *       [       ]       %
>=      <=      +       +=      -=      !
?       ?:      &&      ??      &=      |=
^=      /=      ..      ==      ({      })
[<      >]      (<      >)      ++      --      
%=      !=      ||      ::      <<      >>      
!!      ...     <<=     >>=
```

### Integer literals

An integer literal is a sequence of digits representing an integer constant. 
An optional prefix sets a non-decimal base: 0b or 0B for binary, 
0o, or 0O for octal, and 0x or 0X for hexadecimal. 
A single 0 is considered a decimal zero. 
In hexadecimal literals, letters a through f and A through F represent values 10 through 15.

For readability, an underscore character _ may appear after a base prefix 
or between successive digits; such underscores do not change the literal's value.

```
INTEGER         ::= DECIMAL_LIT | BINARY_LIT | OCTAL_LIT | HEX_LIT
DECIMAL_LIT     ::= '0' | [1-9] ('_'* DECIMAL_DIGITS)?
BINARY_LIT      ::= '0' [bB] '_'* BINARY_DIGITS
OCTAL_LIT       ::= '0' [oO] '_'* OCTAL_DIGITS
HEX_LIT         ::= '0' [xX] '_'* HEX_DIGITS

BINARY_DIGIT    ::= [01]
HEX_DIGIT       ::= [0-9a-fA-F]

DECIMAL_DIGITS  ::= DIGIT ('_'* DIGIT)*
BINARY_DIGITS   ::= BINARY_DIGIT ('_'* BINARY_DIGIT)*
OCTAL_DIGITS    ::= OCTAL_DIGIT ('_'* OCTAL_DIGIT)*
HEX_DIGITS      ::= HEX_DIGIT ('_'* HEX_DIGIT)*
```


```
42
4_2
0_600
0o600
0O600           // second character is capital letter 'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

0600            // Invalid, non zero decimal number may not start with 0 
_42             // an identifier, not an integer literal
42_             // invalid: _ must separate successive digits
0_xBadFace      // invalid: _ must separate successive digits
```

### Floating point literals

A floating-point literal is a decimal or hexadecimal representation of a floating-point constant.

A decimal floating-point literal consists of an integer part (decimal digits), a decimal point, 
a fractional part (decimal digits), and an exponent part (e or E followed by an optional 
sign and decimal digits). One of the integer part or the fractional part may be elided; 
one of the decimal point or the exponent part may be elided. An exponent value exp scales 
the mantissa (integer and fractional part) by powers of 10.

A hexadecimal floating-point literal consists of a 0x or 0X prefix, an integer part 
(hexadecimal digits), a radix point, a fractional part (hexadecimal digits), 
and an exponent part (p or P followed by an optional sign and decimal digits). 
One of the integer part or the fractional part may be elided; the radix point 
may be elided as well, but the exponent part is required. 
An exponent value exp scales the mantissa (integer and fractional part) by powers of 2.

For readability, an underscore character _ may appear after a base prefix or between successive digits; 
such underscores do not change the literal value.

```
FLOAT_LIT       ::= DEC_FLOAT_LIT | HEX_FLOAT_LIT
DEC_FLOAT_LIT   ::= DECIMAL_DIGITS '.' DECIMAL_DIGITS? DEC_EXPONENT? 
                    | DECIMAL_DIGITS DEC_EXPONENT
                    | '.' DECIMAL_DIGITS DEC_EXPONENT?
DEC_EXPONENT    ::= [eE] [+-]? DECIMAL_DIGITS
HEX_FLOAT_LIT   ::= '0' [xX] HEX_MANTISSA HEX_EXPONENT
HEX_MANTISSA    ::= HEX_DIGITS '.' HEX_DIGITS?
                    | HEX_DIGITS
                    | '.' HEX_DIGITS 
HEX_EXPONENT    ::= [pP] [+-] DECIMAL_DIGITS                    
```

### Character literals

A character literal may either: (a) represent a constant value of up to 8 bytes (16 bytes on platforms with Int128 support) or (b) a single unicode character. Escape sequences can be used to represent byte values 0-31 and 128-255.



The following backslash escapes are available to escape:

```
\0      0x00 zero value
\a      0x07 alert/bell
\b      0x08 backspace
\e      0x1B escape
\f      0x0C form feed
\n      0x0A newline
\r      0x0D carriage return
\t      0x09 horizontal tab
\v      0x0B vertical tab
\\      0x5C backslash
\'      0x27 single quote '
\"      0x22 double quote "
\x      Escapes a single byte hex value
\u      Escapes a two byte unicode hex value 
\U      Escapes a four byte unicode hex value
```

```
CHAR_ELEMENT    ::= [\x20-\x26] | [\x28-\x5B] | [\x5D-\x7F]
CHAR_LIT_BYTE   ::= CHAR_ELEMENT | \x5C CHAR_ESCAPE
CHAR_ESCAPE     ::= [abefnrtv\'\"\\] 
                    | 'x' HEX_DIGIT HEX_DIGIT
UNICODE_CHAR    ::= unicode_char                    
                    | 'u' HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT
                    | 'U' HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT 
                          HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT
CHARACTER_LIT   ::= "'" (CHAR_LIT_BYTE+) | UNICODE_CHAR "'"
```

### String literals

A string literal represents a string constant obtained from concatenating a sequence of characters. String literals are character sequences between double quotes, as in "bar". Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in rune literals, with the same restrictions. The two-digit hexadecimal (\xnn) escapes represent individual bytes of the resulting string; all other escapes represent the (possibly multibyte) UTF-8 encoding of individual characters. Thus inside a string literal \xFF represent a single byte of value 0xFF=255, while ÿ, \u00FF, \U000000FF and \xc3\xbf represent the two bytes 0xc3 0xbf of the UTF-8 encoding of character U+00FF.

```
STRING_LIT      ::= \x22 (CHAR_LIT_BYTE | UNICODE_CHAR)* \x22
```

## Types

Types consist of built-in types and user-defined types (enums, structs, unions, bitstructs, errtype and distinct).

### Boolean types
The

### Integer types


### Floating point types


### Vector types
### Complex types


### String types
### Array types

An array has the alignment of its element. Zero sized arrays are allowed and have the size 0.

### Subarray types

The subarray consist of a pointer, followed by an usz length, having the alignment of pointers.

### Pointer types
### Struct types

A struct without any members or with only arrays of size 0 have the size 0.

The alignment of a struct without any members is 1.

### Union types

The alignment of a union without any members is 1.

### Error types


### Enum types
### Function types
### Virtual types

## Declarations and scope

## Expressions

### Casts

### Pointer casts

#### Integer to pointer cast
Any integer of pointer size or larger may be explicitly cast to a pointer. An integer to pointer cast is considered non-constant, except in the special case where the integer == 0. In that case, the result is constant `null`.

Example:
```
byte a = 1;
int* b = (int*)a; // Invalid, pointer type is > 8 bits.
int* c = (int*)1; // Valid, but runtime value.
int* d = (int*)0; // Valid and constant value.
```

#### Pointer to integer cast
A pointer may be cast to any integer, truncating the pointer value if the size of the pointer is larger than the pointer size. A pointer to integer cast is considered non-constant, except in the special case of a null pointer, where it is equal to the integer value 0.

Example:

```
fn void test() { ... }
def VoidFunc = fn void test();

VoidFunc a = &test;
int b = (int)null;
int c = (int)a; // Invalid, not constant
int d = (int)((int*)1); // Invalid, not constant
```

### Subscript operator

The subscript operator may take as its left side a pointer, array, subarray or vararray. The index may be of any integer type. TODO
*NOTE* The subscript operator is not symmetrical as in C. For example in C3 `array[n] = 33` is allowed, but not `n[array] = 33`. This is a change from C. 

### Operands
### Compound Literals

Compound literals have the format

```
compound_literal   ::= TYPE_IDENTIFIER '(' initializer_list ')'
initializer_list   ::= '{' (initializer_param (',' initializer_param)* ','?)? '}'
initializer_param  ::= expression | designator '=' expression
designator         ::= array_designator | range_designator | field_designator
array_designator   ::= '[' expression ']'
range_designator   ::= '[' range_expression ']'
field_designator   ::= IDENTIFIER
range_expression   ::= (range_index)? '..' (range_index)?
range_index        ::= expression | '^' expression
```

Taking the address of a compound literal will yield a pointer to stack allocated temporary.

### Function calls
#### Varargs

For varargs, a `bool` or *any integer* smaller than what the C ABI specifies for the c `int` type is cast to `int`. Any float smaller than a double is cast to `double`. Compile time floats will be cast to double. Compile time integers will be cast to c `int` type.

## Statements

```
stmt               ::= compound_stmt | non_compound_stmt
non_compound_stmt  ::= assert_stmt | if_stmt | while_stmt | do_stmt | foreach_stmt | foreach_r_stmt 
                       | for_stmt | return_stmt | break_stmt | continue_stmt | var_stmt 
                       | declaration_stmt | defer_stmt | nextcase_stmt | asm_block_stmt
                       | ct_echo_stmt | ct_assert_stmt | ct_if_stmt | ct_switch_stmt 
                       | ct_for_stmt | ct_foreach_stmt | expr_stmt 
```

### Assert statement

The assert statement will evaluate the expression and call the panic function if it evaluates
to false.

```
assert_stmt        ::= "assert" "(" expr ("," assert_message)? ")" ";"
assert_message     ::= constant_expr ("," expr)*
```

#### Conditional inclusion

`assert` statements are only included in "safe" builds. They may turn into **assume directives** for
the compiler on "fast" builds.

#### Assert message

The assert message is optional. It can be followed by an arbitrary number of expressions, in which case
the message is understood to be a format string, and the following arguments are passed as values to the
format function.

The assert message must be a compile time constant. There are no restriction on the format argument expressions.

#### Panic function

If the assert message has no format arguments or no assert message is included, 
then the regular panic function is called. If it has format arguments then `panicf` is called instead.

In the case the `panicf` function does not exist (for example, compiling without the standard library),
then the format and the format arguments will be ignored and the `assert` will be treated
as if no assert message was available.

### Break statement

A break statement exits a `while`, `for`, `do`, `foreach` or `switch` scope. A labelled break
may also exit a labelled `if`.

```
break_stmt         ::= "break" label? ";"
```

#### Break labels

If a break has a label, then it will instead exit an outer scope with the label.

#### Unreachable code

Any statement following break in the same scope is considered unreachable.

### Continue statement

A continue statement jumps to the cond expression of a `while`, `for`, `do` or `foreach`

```
continue_stmt      ::= "continue" label? ";"
```

#### Continue labels

If a `continue` has a label, then it will jump to the cond of the while/for/do in the outer scope 
with the corresponding label.

#### Unreachable code

Any statement following `continue` in the same scope is considered unreachable.

### Declaration statement

A declaration statement adds a new runtime or compile time variable to the current scope. It is available after the declaration statement.

```
declaration_stmt   ::= const_declaration | local_decl_storage? optional_type decls_after_type ";"
local_decl_storage ::= "tlocal" | "static"
decls_after_type   ::= local_decl_after_type ("," local_decl_after_type)*
decl_after_type    ::= CT_IDENT ("=" constant_expr)? | IDENTIFIER opt_attributes ("=" expr)?
```

#### Thread local storage

Using `tlocal` allocates the runtime variable as a **thread local** variable. In effect this is the same as declaring
the variable as a global `tlocal` variable, but the visibility is limited to the function. `tlocal` may not be
combined with `static`.

The initializer for a `tlocal` variable must be a valid global init expression.

#### Static storage

Using `static` allocates the runtime variable as a function **global** variable. In effect this is the same as declaring 
a global, but visibility is limited to the function. `static` may not be combined with `tlocal`.

The initializer for a `static` variable must be a valid global init expression.

#### Scopes

Runtime variables are added to the runtime scope, compile time variables to the compile time scope. See **var statements**.

#### Multiple declarations

If more than one variable is declared, no init expressions are allowed for any of the variables.

#### No init expression

If no init expression is provided, the variable is **zero initialized**.

#### Opt-out of zero initialization

Using the @noinit attribute opts out of **zero initialization**.

#### Self referencing initialization

An init expression may refer to the **address** of the same variable that is declared, but not the **value** of the 
variable.

Example:
```c
void* a = &a;  // Valid
int a = a + 1; // Invalid
```

### Defer statement

The defer statements are executed at (runtime) scope exit, whether through `return`, `break`, `continue` or rethrow.

```
defer_stmt         ::= "defer" ("try" | "catch")? stmt
```

#### Defer in defer

The defer body (statement) may not be a defer statement. However, if the body is a compound statement then
this may have any number of defer statements.

#### Static and tlocal variables in defer

Static and tlocal variables are allowed in a defer statement. Only a single variable is instantiated regardless of
the number of inlining locations.

#### Defer and return

If the `return` has an expression, then it is evaluated before the defer statements (due to exit from the current function scope),
are executed.

Example:

```c
int a = 0;
defer a++;
return a;
// This is equivalent to
int a = 0;
int temp = a;
a++;
return temp;
```

#### Defer and jump statements

A defer body may not contain a `break`, `continue`, `return` or rethrow that would exit the statement.

#### Defer execution

Defer statements are executed in the reverse order of their declaration, starting from the last declared
defer statement.

#### Defer try

A `defer try` type of defer will only execute if the scope is left through normal fallthrough, `break`, 
`continue` or a `return` with a result.

It will not execute if the exit is through a rethrow or a `return` with an optional value.

#### Defer catch

A `defer catch` type of defer will only execute if the scope is left through a rethrow or a `return` with an optional value

It will not execute if the exit is a normal fallthrough, `break`, `continue` or a `return` with a result.

#### Non-regular returns - longjmp, panic and other errors

Defers will not execute when doing `longjmp` terminating through a `panic` or other error. They 
are only invoked on regular scope exits.

### If statement

An if statement will evaluate the cond expression, then execute the first statement (the "then clause") in the if-body
if it evaluates to "true", otherwise execute the else clause. If no else clause exists, then the
next statement is executed.

```
if_stmt            ::= "if" (label ":")? "(" cond_expr ")" if_body
if_body            ::= non_compound_stmt | compound_stmt else_clause?
else_clause        ::= "else" (if_stmt | compound_stmt)
```

#### Scopes

Both the "then" clause and the else clause open new scopes, even if they are non-compound statements.
The cond expression scope is valid until the exit of the entire statement, so any declarations in the 
cond expression are available both in then and else clauses. Declarations in the "then" clause is not available
in the else clause and vice versa.

#### Special parsing of the "then" clause

If the then-clause isn't a compound statement, then it must follow on the same row as the cond expression.
It may not appear on a consecutive row.

#### Break

It is possible to use labelled break to break out of an if statement. Note that an unlabelled `break` may not
be used.

### Nextcase statement

Nextcase will jump to another `switch` case.

```
nextcase_stmt      ::= "nextcase" (label ":")? expr? ";" 
```

TODO

### Switch statement

```
switch_stmt        ::= "switch" (label ":")? ("(" cond_expr ")")? switch body
switch_body        ::= "{" case_clause* "}"
case_clause        ::= default_stmt | case_stmt
default_stmt       ::= "default" ":" stmt*
case_stmt          ::= "case" label? expr ":" stmt*
```

#### Regular switch

If the cond expression exists and all case statements have constant expression, then first the
cond expression is evaluated, next the case corresponding to the expression's value will be jumped to
and the statement will be executed. After reaching the end of the statements and a new case clause *or* the
end of the switch body, the execution will jump to the first statement after the switch.

#### If-switch

If the cond expression is missing or the case statements are non-constant expressions, then each case clause will
be evaluated in order after the cond expression has been evaluated (if it exists): 

1. If a cond expression exists, calculate the case expression and execute the case if it is matching the 
cond expression. A default statement has no expression and will always be considered matching the cond expression
reached.
2. If no con expression exists, calculate the case expression and execute the case if the expression evaluates to 
"true" when implicitly converted to boolean. A default statement will always be considered having the "true" result.

#### Fallthrough

If a case clause has no statements, then when executing the case, rather than exiting the switch, the next case clause
immediately following it will be executed. If that one should also be missing statement, the procedure
will be repeated until a case clause with statements is encountered (and executed), or the end of the switch is reached.

#### Exhaustive switch

If a switch case has a default clause *or* it is switching over an enum and there exists a case for each enum value
then the switch is exhaustive.

#### Break

If an unlabelled break, or a break with the switch's label is encountered, 
then the execution will jump out of the switch and proceed directly after the end of the switch body.

#### Unreachable code

If a switch is exhaustive and all case clauses end with a jump instruction, containing no break statement out 
of the current switch, then the code directly following the switch will be considered **unreachable**.

#### Switching over typeid

If the switch cond expression is a typeid, then case declarations may use only the type name after the case,
which will be interpreted as having an implicit `.typeid`. Example: `case int:` will be interpreted as if 
written `case int.typeid`.

#### Nextcase without expression

Without a value `nextcase` will jump to the beginning of the next case clause. It is not allowed to
put `nextcase` without an expression if there are no following case clauses.

#### Nextcase with expression

Nextcase with an expression will evaluate the expression and then jump *as if* the switch was entered with
the cond expression corresponding to the value of the nextcase expression. Nextcase with an expression cannot
be used on a switch without a cond expression.

#### Do statement

The do statement first evaluates its body (inner statement), then evaluates the cond expression.
If the cond expression evaluates to true, jumps back into the body and repeats the process.

```
do_stmt            ::= "do" label? compound_stmt ("while" "(" cond_expr ")")? ";" 
```

#### Unreachable

The statement after a `do` is considered unreachable if the cond expression cannot ever be false
and there is no `break` out of the do.

#### Break

`break` will exit the do with execution continuing on the following statement.

#### Continue

`continue` will jump directly to the evaluation of the cond, as if the end of the statement had been reached.

#### Do block

If no `while` part exists, it will only execute the block once, as if it ended with `while (false)`, this is 
called a "do block"

### For statement

The `for` statement will perform the (optional) init expression. The cond expression will then be tested. If
it evaluates to `true` then the body will execute, followed by the incr expression. After execution will
jump back to the cond expression and execution will repeat until the cond expression evaluates to `false`.

```
for_stmt           ::= "for" label? "(" init_expr ";" cond_expr? ";" incr_expr ")" stmt
init_expr          ::= decl_expr_list?
incr_expr          ::= expr_list? 
```

#### Init expression

The init expression is only executed once before the rest of the for loop is executed.
Any declarations in the init expression will be in scope until the for loop exits.

The init expression may optionally be omitted.

#### Incr expression

The incr expression is evaluated before evaluating the cond expr every time except for the first one.

The incr expression may optionally be omitted.

#### Cond expression

The cond expression is evaluated every loop. Any declaration in the cond expression is scoped to the 
current loop, i.e. it will be reinitialized at the start of every loop.

The cond expression may optionally be omitted. This is equivalent to setting the cond expression to 
always return `true`.

#### Unreachable

The statement after a `for` is considered unreachable if the cond expression cannot ever be false, or is 
omitted and there is no `break` out of the loop.

#### Break

`break` will exit the `for` with execution continuing on the following statement after the `for`.

#### Continue

`continue` will jump directly to the evaluation of the cond, as if the end of the statement had been reached.

#### Equivalence of `while` and `for`

A `while` loop is functionally equivalent to a `for` loop without init and incr expressions.

### Foreach and foreach_r statements

The `foreach` statement will loop over a sequence of values. The `foreach_r` is equivalent to
`foreach` but the order of traversal is reversed.
`foreach` starts with element `0` and proceeds step by step to element `len - 1`.
`foreach_r` starts starts with element `len - 1` and proceeds step by step to element `0`.


```
foreach_stmt       ::= "foreach" label? "(" foreach_vars ":" expr ")" stmt
foreach_r_stmt     ::= "foreach_r" label? "(" foreach_vars ":" expr ")" stmt
foreach_vars       ::= (foreach_index ",")? foreach_var
foreach_var        ::= type? "&"? IDENTIFIER
```

#### Break

`break` will exit the foreach statement with execution continuing on the following statement after.

#### Continue

`continue` will cause the next iteration to commence, as if the end of the statement had been reached.

#### Iteration by value or reference

Normally iteration are by value. Each element is copied into the foreach variable. If `&` 
is added before the variable name, the elements will be retrieved by reference instead, and consequently
the type of the variable will be a pointer to the element type instead.

#### Foreach variable

The foreach variable may omit the type. In this case the type is inferred. If the type differs from the element
type, then an implicit conversion will be attempted. Failing this is a compile time error.


#### Foreach index

If a variable name is added before the foreach variable, then this variable will receive the index of the element.
For `foreach_r` this mean that the first value of the index will be `len - 1`.

The index type defaults to `usz`.

If an optional type is added to the index, the index will be converted to this type. The type must be an
integer type. The conversion happens as if the conversion was a direct cast. If the actual index value
would exceed the maximum representable value of the type, this does not affect the actual iteration, but 
may cause the index value to take on an incorrect value due to the cast.

For example, if the optional index type is `char` and the actual index is `256`, then the index value would show `0`
as `(char)256` evaluates to zero.

Modifying the index variable will not affect the foreach iteration.

#### Foreach support

Foreach is natively supported for any subarray, array, pointer to an array, vector and pointer to a vector.
These types support both iteration by value and reference.

In addition, a type with **operator overload** for `len` and `[]` will support iteration by value,
and a type with **operator overload** for `len` and `&[]` will support iteration by reference.

### Return statement

The return statement evaluates its expression (if present) and returns the result.

```
return_stmt        ::= "return" expr? ";"
```

#### Jumps in return statements

If the expression should in itself cause an implicit return, for example due to the rethrow operator `!`, then this
jump will happen before the return.

An example:

    return foo()!;
    // is equivalent to:
    int temp = foo()!;
    return temp;

#### Return from expression blocks

A `return` from an expression block only returns out of the expression block, it never returns from the
expression block's enclosing scope.

#### Empty returns

An empty return is equivalent to a return with a void type. Consequently constructs like `foo(); return;` and `return (void)foo();`
are equivalent.

#### Unreachable code

Any statement directly following a return in the same scope are considered unreachable.

### While statement

The while statement evaluates the cond expression and executes the statement if it evaluates to true.
After this the cond expression is evaluated again and the process is repeated until cond expression returns false.

```
while_stmt         ::= "while" label? "(" cond_expr ")" stmt
```

#### Unreachable

The statement after a while is considered unreachable if the cond expression cannot ever be false
and there is no `break` out of the while.

#### Break

`break` will exit the while with execution continuing on the following statement.

#### Continue

`continue` will jump directly to the evaluation of the cond, as if the end of the statement had been reached.

### Var statement

A var statement declares a variable with inferred type, or a compile time type variable. It can be used both
for runtime and compile time variables. The use for runtime variables is limited to macros.

```
var_stmt           ::= "var" IDENTIFIER | CT_IDENT | CT_TYPE_IDENT ("=" expr)? ";" 
```

### Inferring type

In the case of a runtime variable, the type is inferred from the expression. Not providing an expression 
is a compile time error. The expression must resolve to a runtime type.

For compile time variables, the expression is optional. The expression may resolve to a runtime or compile time type.

### Scope

Runtime variables will follow the runtime scopes, identical to behaviour in a declaration statement. The compile
time variables will follow the compile time scopes which are delimited by scoping compile time statements (`$if`, `$switch`,
`$foreach` and `$for`).

## Modules

Module paths are hierarchal, with each sub-path appended with '::' + the name:

```
path               ::= PATH_SEGMENT ("::" PATH_SEGMENT)
```

Each module declaration starts its own **module section**. All imports and all `@local` declarations
are only visible in the current **module section**.

```
module_section     ::= "module" path opt_generic_params? attributes? ";"
generic_param      ::= TYPE_IDENT | CONST_IDENT
opt_generic_params ::= "(<" generic_param ("," generic_param)* ">)"
```

Any visibility attribute defined in a **module section** will be the default visibility in all 
declarations in the section.

If the `@test` attribute is applied to the **module section** then all function declarations
will implicitly have the `@test` attribute.