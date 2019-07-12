
# Grammar

The following (incomplete) grammar can be used with this: https://bottlecaps.de/rr/ui to get a railroad diagram of the grammar.
```bnf
source ::= module_def import_def+ top_level

module_def ::= 'module' IDENT EOS

import_def_ ::= 'import' IDENT ( ('as' IDENT) | 'local' )? EOS

top_level ::= (array_append | global_decl)*

global_decl ::= 'public'? (type_def | func_def | var_def | macro_def)

array_append ::= IDENT '+=' init_value

func_def ::= return_value IDENT '(' function_args ')' compound_stmt?

generic_def ::= 'generic' IDENT '(' generic_arg_list ')' compound_stmt

generic_arg_list ::= generic_arg (',' generic_arg)*

generic_arg ::= type? IDENT

macro_def ::= 'macro' IDENT '('  macro_arg_list? ')' compound_stmt

macro_arg_list ::= macro_arg (',' macro_arg)*

macro_arg ::= '&'? IDENT

var_def ::= type_qualifier IDENT ...

struct_or_union ::= 'struct' | 'union'

type_def ::= 'type' IDENT type_def_body

type_def_body ::= enum_def | func_type_def | struct_def | error_def

enum_def ::= 'enum' type attributes? '{' enum_body? '}'

enum_body ::= enum_value (',' enum_value)* ','?

enum_value ::= IDENT ('=' expression)?

error_def ::= 'error' IDENT '{' IDENT (',' IDENT)* '}'

func_type_def ::= 'func' type attributes? '(' function_args ')' EOS

struct_def ::= struct_or_union attributes? struct_body

struct_body ::= '{' (struct_member (',' struct_member)* )? '}'

struct_member ::= (type IDENT) | (struct_or_union IDENT? struct_body)

compound_stmt ::= '{' (statement | declaration)* '}'
    
statement ::= label_stmt | compound_stmt | expr_stmt | if_stmt | switch_stmt | iter_stmt | jump_stmt

jump_stmt ::= ( ('goto' IDENT) | 'continue' | 'break' | 'next' | ('return' expression?) ) EOS

label_stmt ::= IDENT ':'

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

base_type ::= built_in_type | IDENT

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

dot_op ::= '.' IDENT

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

declaration ::= qualified_type IDENT ('=' init_expr)? EOS

init_expr ::= expression | struct_init

struct_init ::= '{' stuct_init_list? '}'

struct_init_list ::= strict_init_decl (',' struct_init_decl)*

struct_init_decl ::= member_init | init_expr

member_init ::=_ '.' IDENT '=' init_expr
```