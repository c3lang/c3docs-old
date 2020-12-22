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

The form a..b represents the set of characters from a through b as alternatives.

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

White space is defined as the ASCII CR (U+000D), the ASCII horizontal tab character (U+0009) and the space character (U+0020) and the line terminator character.

```
WHITESPACE      ::= [ \t\v\n\f]
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
UC_LETTER_US    ::= UC_LETTER | "_"
ALPHANUM        ::= LETTER | DIGIT
ALPHANUM_US     ::= ALPHANUM | "_"
UC_ALPHANUM_US  ::= UC_LETTER_US | DIGIT
```

### Comments

Thre are three types of regular comments:

1. `/* text */` block comments. The text between `/*` and `*/` is ignored.
2. `// text` a line comment. The text between `//` and line end is ignored.
3. `/+ text +/` nesting comments. The text between `/+` and `+/` is ignored. Unlike `/* text */` it has nesting behaviour, so for every `/+` discovered between the first `/+` and the last `+/` a corresponding `+/` must be found.

### Doc comments

1. `/** text **/` doc block comment. The text between `/**` and `**/` is optionally parsed using the doc comment syntatic grammar. A compiler may choose to read `/** text **/` as a regular comment.
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
```

### Keywords

The following keywords are reserved and may not be used as identifiers:


```
alias       as          asm
assert      attribute   break
case        cast        catch
const       continue    default
defer       define      do
else        enum        extern
error       false       func
generic     if          import
in          local       macro
module      next        nil
public      return      struct
switch      true        try
typeid      typeof      typedef
var         volatile    void
while

bool        quad        double      
float       long        ulong
int         uint        byte
short       ushort      char
isize       usize       half

$assert     $case       $default
$if         $for        $else
$elif       $if         $switch
$foreach    $endswitch  $endif      
$endforeach $unreachable     
                  
```

### Operators and punctuation

The following character sequences represent operators and punctuation.

```
&       @       ~       |       ^       :
,       /       $       .       ;       )
>       <       #       {       }       -
(       )       *       [       ]       %
>=      <=      +       +=      -=      !
?       ?:      &&      ->      &=      |=
^=      /=      ..      ==      ({      })
-%      +%      *%      ++      --      %=
!=      ||      ::      <<      >>      !!
...     *%=     +%=     -%=     <<=     >>=
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

A character literal represents a constant value of 1, 2, 4 or 8 bytes, and may only be printable ASCII characters (U+0020 - U+007F). Each character is intepreted as part of an 1, 2, 4 or 8 byte integer constant. Escape sequences can be used to represent byte values 0-31 and 128-255.

A 2 byte literal is referred to as a 2cc literal, a 4 byte literal 4cc and an eight byte literal is called 8cc. We will commonly refer to a single byte character literal as just "character literal".

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
\u      Escapes a two byte hex value
\U      Escapes a four byte hex value
```

```
CHAR_ELEMENT    ::= [\x20-\x26] | [\x28-\x5B] | [\x5D-\x7F]
CHAR_LIT_BYTE   ::= CHAR_ELEMENT | \x5C CHAR_ESCAPE
CHAR_ESCAPE     ::= [abefnrtv\'\"\\] 
                    | 'x' HEX_DIGIT HEX_DIGIT
                    | 'u' HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT
                    | 'U' HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT 
                          HEX_DIGIT HEX_DIGIT HEX_DIGIT HEX_DIGIT
CHARACTER_LIT   ::= "'" CHAR_LIT_BYTE "'"
TWOCC_LIT       ::= "'" CHAR_LIT_BYTE CHAR_LIT_BYTE "'"
FOURCC_LIT      ::= "'" CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE "'"
EIGHTCC_LIT     ::= "'" CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE
                        CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE CHAR_LIT_BYTE "'"
```

### String literals

A string literal represents a string constant obtained from concatenating a sequence of characters. String literals are character sequences between double quotes, as in "bar". Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in rune literals, with the same restrictions. The two-digit hexadecimal (\xnn) escapes represent individual bytes of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual characters. Thus inside a string literal \xFF represent a single byte of value 0xFF=255, while ÿ, \u00FF, \U000000FF and \xc3\xbf represent the two bytes 0xc3 0xbf of the UTF-8 encoding of character U+00FF.

STRING_LIT      ::= \x22 CHAR_LIT_BYTE* \x22

## Types
### Boolean types
### Integer types


### Floating point types


### Vector types
### Complex types

A complex type is defined as a struct with two elements of the same floating point type. The first member holds the real part of a complex number, and the second member holds the imaginary part.


```
struct Complex
{
    float real;
    float imaginary;
}
```

### String types
### Array types

An array has the alignment of its element. Zero sized arrays are allowed and have the size 0.

### Vararray types

### Subarray types

The subarray consist of a pointer, followed by a usize length, having the alignment of pointers.

### Pointer types
### Struct types

A struct without any members or with only arrays of size 0 have the size 0.

The alignment of a struct without any members is 1.

### Union types

The alignment of a union without any members is 1.

### Error types

Th
### Enum types
### Function types
### Virtual types

## Declarations and scope

## Expressions
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
### If statement

### Switch stat
## Modules

