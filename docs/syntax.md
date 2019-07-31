
# Syntax

## Keywords

The following are 72 reserved keywords used by C3:

as, asm, break, case, cast, catch, const, continue,
default, defer, do, else, enum, error, false, for, func,
generic, goto, if, import, local, macro, module, nil,
public, return, struct, switch, throw, throws, true, 
try, typedef, union, until, void, volatile, while,
f32, f64, float, double, 
u1, i8, i16, i32, i64, u8, u16, u32, u64, 
char, bool, byte, short, ushort, int, uint, long, ulong, isize, usize,
c_short, c_ushort, c_int, c_uint, c_long, c_ulong, c_longlong, 
c_ulonglong, c_longdouble


In addition to those, the following 12 are reserved but currently not used:
f16, f128, f256, half, quad, i128, i256, u128, u256, type, var, alias
 
For macros the the following 9 `@` identifiers are reserved as keywords:
@param, @throws, @return, @ensure, @require, @pure, @const, @reqparse, @deprecated

## Railroad grammar

The following (broken) grammar can be used with this: https://bottlecaps.de/rr/ui to get a railroad diagram of the grammar.
```bnf

compilation_unit ::= module? import* top_level_decl*

module ::= 'module' IDENT ';'

import ::= 'import' IDENT ('as' IDENT)? 'local'? ';'

top_level_decl ::= func_decl
                 | struct_decl 
                 | union_decl
                 | enum_decl 
                 | error_decl 
                 | macro_decl 
                 | const_decl 
                 | var_decl
                 | type_decl
                 | macro_var_decl
                 | docs
                 | generic_decl

error_decl ::= 'error' IDENT '{' IDENT (',' IDENT)* ','? '}'

enum_decl ::= 'enum' IDENT (':' type)? '{' enum_member (',' enum_member)* ','? '}' 
enum_member ::= IDENT ('=' expr)?

const_decl ::= 'const' type IDENT '=' expr ';'
var_decl ::= type IDENT '=' expr ';'
type_decl ::= 'typedef' type 'as' IDENT ';'
            | 'typedef' 'func' type IDENT '(' function_args ')' 'as' IDENT ';'

func_decl ::= 'func' type IDENT '(' function_args ')' (compound_stmt | ';')

generic_decl ::= 'generic' IDENT '(' generic_arg_list ')' compound_stmt

generic_arg_list ::= generic_arg (',' generic_arg)*
generic_arg ::= type? IDENT

base_type ::= built_in_type 
            | IDENT

built_in_type ::= bit_types 
                | named_types 
                | c_types

bit_types ::= 'u1'
            | 'u8' | 'u16' | 'u32' | 'u64' | 'u128' | 'u256'
            | 'i8' | 'i16' | 'i32' | 'i64' | 'i128' | 'i256'
            | 'f16' | 'f32' | 'f64' | 'f128'

named_types ::= 'void' | 'bool' 
              | 'byte' | 'ushort' | 'uint' | 'ulong' | 'usize'
              | 'char' | 'short' | 'int' | 'long' | 'isize'
              | 'float' | 'double' | 'quad'

c_types ::= 'c_char' | 'c_short' | 'c_int' | 'c_long' | 'c_longlong'
          | 'c_uchar' | 'c_ushort' | 'c_uint' | 'c_ulong' | 'c_ulonglong'
          | 'c_float' | 'c_double' | 'c_longdouble'

type ::= base_type ('[' expr? ']' | '*' | '?' | '@')*

macro_decl ::= 'macro' AT_IDENT '('  macro_arg_list? ')' compound_stmt

macro_arg_list ::= macro_arg (',' macro_arg)*

macro_arg ::= DOLLAR_IDENT | IDENT | HASH_IDENT


---> The rest is todo


struct_or_union ::= 'struct' | 'union'

enum_def ::= 'enum' TIDENT enum_def type attributes? '{' enum_body? '}'

enum_body ::= enum_value (',' enum_value)* ','?

enum_value ::= TIDENT ('=' expression)?

error_def ::= 'error' TIDENT '{' TIDENT (',' TIDENT)* '}'

func_type_def ::= 'func' type attributes? '(' function_args ')' EOS

struct_def ::= struct_or_union TIDENT attributes? struct_body

struct_body ::= '{' (struct_member (',' struct_member)* )? '}'

struct_member ::= (type VIDENT) | (struct_or_union TIDENT? struct_body)

compound_stmt ::= '{' (statement | declaration)* '}'
    
statement ::= label_stmt | compound_stmt | expr_stmt | if_stmt | switch_stmt | iter_stmt | jump_stmt

jump_stmt ::= ( ('goto' TIDENT) | 'continue' | 'break' | 'next' | ('return' expression?) ) EOS

label_stmt ::= TIDENT ':'

if_stmt ::= 'if' control_expr statement (ELSE statement)?

switch_stmt ::= 'switch' control_expr '{' switch_body '}'

switch_body ::= case_stmt | default_stmt

case_stmt ::= 'case' expression ':' case_body

default_stmt ::= 'default' ':' case_body

case_body ::= (statement | declaration)*

control_expr ::= '(' var_def_list? expression ')'

var_def_list ::= var_def (',' var_def)* EOS

iter_stmt ::= while_stmt | do_stmt | for_stmt

while_stmt ::= 'while' control_stml statement

do_stmt ::= 'do' statement while '(' expression ')' EOS

for_stmt ::= 'for' '(' var_def_list? expression? EOS expression ')' statement

type ::= base_type (pointer_suffix | array_suffix)*

qualified_type ::= qualifier* base_type ((qualifier* pointer_suffix) | array_suffix)*

base_type ::= built_in_type | TIDENT

built_in_type ::= bit_types | named_types | c_types

bit_types ::= unsigned_bit_types | signed_bit_types | float_bit_types

unsigned_bit_types ::= 'u8' | 'u16' | 'u32' | 'u64' | 'u1'

signed_bit_types ::= 'i8' | 'i16' | 'i32' | 'i64'

float_bit_types ::=  'f16' | 'f32' | 'f64' | 'f128'

named_types ::= signed_named_types | unsigned_named_types | float_named_types | 'void'

signed_named_types ::= 'char' | 'short' | 'int' | 'long' | 'isize'

unsigned_named_types ::= 'bool' | 'byte' | 'ushort' | 'uint' | 'ulong' | 'usize'

float_named_types ::= 'float' | 'double' | 'quad'

c_types ::= signed_c_types | unsigned_c_types | float_c_types

signed_c_types ::= 'c_ichar' | 'c_ushort' | 'c_int' | 'c_long' | 'c_longlong'

unsigned_c_types ::= 'c_uchar' | 'c_ushort' | 'c_uint' | 'c_ulong' | 'c_ulonglong'

float_c_types ::= 'c_float' | 'c_double' | 'c_longdouble'

pointer_suffix ::= '*' | '&'

array_suffix ::= '[' expression? ']'

expr_stmt ::= expression EOS

primary_expr ::= IDENT | CONSTANT | STRING_LITERL | '(' expression ')'

postfix_expr ::= primary_expr postfix_op*

postfix_op ::= array_op | call_op | dot_op | inc_dec_op

array_op ::= '[' expression ']'

call_op ::= '(' arg_expr_list? ')'

dot_op ::= '.' VIDENT

inc_dec_op ::= '++' | '--'

arg_expr_list ::= assign_expr (',' assign_expr)*

unary_op ::= inc_dec_op | '&' | '*' | '+' | '-' | '~' | '!'

unary_expr ::= inc_dec_op* postfix_expr

binary_op ::= mult_op | arith_op | assign_op | rel_op | bit_op | bool_op

assign_op ::= '=' | mult_assign_op | arith_assign_op | bit_assign_op

mult_assign_op ::= '*=' | '/=' | '%='

arith_assign_op ::= '+=' | '-='

bit_assign_op ::= '>>=' | '<<=' | '>>>=' | '|=' | '&=' | '^='

arith_op ::= '+' | '-'

mult_op ::= '*' | '/' | '%'

bit_op ::= '|' | '&' | '^' | '>>' | '<<' | '>>>'

bool_op ::= '&&' | '||'

rel_op ::= '==' | '!=' | '<' | '>' | '<=' | '>='

binary_expr ::= unary_expr (binary_op unary_expr)*

expression ::= binary_expr ('?' expression ':' cond_expr)?

declaration ::= qualified_type VIDENT ('=' init_expr)? EOS

init_expr ::= expression | struct_init

struct_init ::= '{' stuct_init_list? '}'

struct_init_list ::= strict_init_decl (',' struct_init_decl)*

struct_init_decl ::= member_init | init_expr

member_init ::=_ '.' VIDENT '=' init_expr
```