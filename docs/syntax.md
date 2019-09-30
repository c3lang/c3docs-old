
# Syntax

## Keywords

The following are 73 reserved keywords used by C3:

as, asm, attribute, break, case, cast, catch, const, continue,
default, defer, do, else, enum, error, false, for, func,
generic, goto, if, import, local, macro, module, nil,
public, return, struct, switch, throw, throws, true, 
try, type, typedef, union, until, var, void, volatile, while,
f32, f64, float, double, 
u1, i8, i16, i32, i64, u8, u16, u32, u64, 
char, bool, byte, short, ushort, int, uint, long, ulong, isize, usize,
c_short, c_ushort, c_int, c_uint, c_long, c_ulong, c_longlong, 
c_ulonglong, c_longdouble


In addition to those, the following 12 are reserved but currently not used:
f16, f128, f256, half, quad, i128, i256, u128, u256, alias
 
For macros the the following 9 `@` identifiers are reserved as keywords:
@param, @throws, @return, @ensure, @require, @pure, @const, @reqparse, @deprecated

## Yacc grammar

```
%token IDENT AT_IDENT CT_IDENT CONSTANT TYPE_IDENT STRING_LITERAL SIZEOF
%token INC_OP DEC_OP LEFT_OP RIGHT_OP LE_OP GE_OP EQ_OP NE_OP
%token AND_OP OR_OP MUL_ASSIGN DIV_ASSIGN MOD_ASSIGN ADD_ASSIGN
%token SUB_ASSIGN LEFT_ASSIGN RIGHT_ASSIGN AND_ASSIGN
%token XOR_ASSIGN OR_ASSIGN TYPE_NAME VAR

%token TYPEDEF MODULE IMPORT
%token CHAR SHORT INT LONG FLOAT DOUBLE CONST VOLATILE VOID
%token BYTE USHORT UINT ULONG BOOL
%token STRUCT UNION ENUM ELLIPSIS AS LOCAL

%token CASE DEFAULT IF ELSE SWITCH WHILE DO FOR GOTO CONTINUE BREAK RETURN
%token TYPE FUNC ERROR MACRO GENERIC CTIF CTELIF CTENDIF CTELSE CTSWITCH CTCASE CTDEFAULT CTEACH
%token THROWS THROW TRY CATCH SCOPE PUBLIC DEFER ATTRIBUTE

%start translation_unit
%%

primary_expression
	: IDENT
	| IDENT SCOPE IDENT
	| CONSTANT
	| STRING_LITERAL
	| CT_IDENT
	| IDENT SCOPE AT_IDENT
	| '(' expression ')'
	;

postfix_expression
	: primary_expression
	| postfix_expression '[' expression ']'
	| postfix_expression '(' ')'
	| postfix_expression '(' argument_expression_list ')'
	| postfix_expression '.' IDENT
	| postfix_expression INC_OP
	| postfix_expression DEC_OP
	;

argument_expression_list
	: expression
	| argument_expression_list ',' expression
	;

unary_expression
	: postfix_expression
	| INC_OP unary_expression
	| DEC_OP unary_expression
	| unary_operator unary_expression
	| SIZEOF '(' type_expression ')'
	;

unary_operator
	: '&'
	| '*'
	| '+'
	| '-'
	| '~'
	| '!'
	;


multiplicative_expression
	: unary_expression
	| multiplicative_expression '*' unary_expression
	| multiplicative_expression '/' unary_expression
	| multiplicative_expression '%' unary_expression
	;

shift_expression
	: multiplicative_expression
	| shift_expression LEFT_OP multiplicative_expression
	| shift_expression RIGHT_OP multiplicative_expression
	;

bit_expression
	: shift_expression
	| bit_expression '&' shift_expression
	| bit_expression '^' shift_expression
	| bit_expression '|' shift_expression
	;

additive_expression
	: bit_expression
	| additive_expression '+' bit_expression
	| additive_expression '-' bit_expression
	;

relational_expression
	: additive_expression
	| relational_expression '<' additive_expression
	| relational_expression '>' additive_expression
	| relational_expression LE_OP additive_expression
	| relational_expression GE_OP additive_expression
	| relational_expression EQ_OP additive_expression
	| relational_expression NE_OP additive_expression
	;

logical_expression
	: relational_expression
	| logical_expression AND_OP relational_expression
	| logical_expression OR_OP relational_expression
	;

conditional_expression
	: logical_expression
	| logical_expression '?' expression ':' conditional_expression
	;

assignment_expression
    : conditional_expression
    | unary_expression assignment_operator assignment_expression
    ;

expression
	: assignment_expression
	| TRY assignment_expression
	| TRY assignment_expression ELSE assignment_expression
	;


assignment_operator
	: '='
	| MUL_ASSIGN
	| DIV_ASSIGN
	| MOD_ASSIGN
	| ADD_ASSIGN
	| SUB_ASSIGN
	| LEFT_ASSIGN
	| RIGHT_ASSIGN
	| AND_ASSIGN
	| XOR_ASSIGN
	| OR_ASSIGN
	;

constant_expression
	: conditional_expression
	;



enumerator_list
	: enumerator
	| enumerator_list ',' enumerator
	;

enumerator
	: IDENT
	| IDENT '=' constant_expression
	;

identifier_list
	: IDENT
	| identifier_list ',' IDENT
	;

macro_argument
    : CT_IDENT
    | IDENT
    ;

macro_argument_list
    : macro_argument
    | macro_argument_list ',' macro_argument
    ;

implicit_decl
    : IDENT
    | IDENT '=' initializer
    ;

explicit_decl
    : type_expression IDENT '=' initializer
    | type_expression
    ;

declaration
    : explicit_decl
    | explicit_decl ',' implicit_decl
    | explicit_decl ',' explicit_decl
    ;

declaration_list
    : declaration
    ;

param_declaration
    : type_expression IDENT
    | type_expression IDENT '=' initializer
    ;

parameter_type_list
	: parameter_list
	| parameter_list ',' ELLIPSIS
	| parameter_list ',' type_expression ELLIPSIS
	;

parameter_list
	: param_declaration
	| parameter_list ',' param_declaration
	;

base_type
    : VOID
    | BOOL
    | CHAR
    | BYTE
    | SHORT
    | USHORT
    | INT
    | UINT
    | LONG
    | ULONG
    | FLOAT
    | DOUBLE
    | TYPE_IDENT
    | IDENT SCOPE TYPE_IDENT
    | TYPE '(' unary_expression ')'
    ;
type_expression
    : base_type
    | type_expression '*'
    | type_expression '[' constant_expression ']'
    | type_expression '[' ']'
    | type_expression '[' '+' ']'
    ;

initializer
	: expression
	| '{' initializer_list '}'
	| '{' initializer_list ',' '}'
	;

initializer_list
	: initializer
	| initializer_list ',' initializer
	;

ct_case_statement
    : CTCASE type_list ':' statement
    | CTDEFAULT ':' statement
    ;

ct_elif_body
    : ct_elif compound_statement
    | ct_elif_body ct_elif compound_statement
    ;

ct_else_body
    : ct_elif_body
    | CTELSE compound_statement
    | ct_elif_body CTELSE compound_statement
    ;

ct_switch_body
    : ct_case_statement
    | ct_switch_body ct_case_statement
    ;

ct_statement
    : ct_if compound_statement
    | ct_if compound_statement ct_else_body
    | ct_switch '{' ct_switch_body '}'
    | CTEACH '(' expression AS CT_IDENT ')' statement
    ;

throw_statement
    : THROW expression ';'

statement
	: compound_statement
    | labeled_statement
	| expression_statement
	| selection_statement
	| iteration_statement
	| jump_statement
	| declaration_statement
	| volatile_statement
	| catch_statement
	| try_statement
	| defer_statement
	| ct_statement
	| throw_statement
	;

defer_catch_body
    : compound_statement
    | expression_statement
    | jump_statement
    | iteration_statement
    | selection_statement
    ;

defer_statement
    : DEFER defer_catch_body
    | DEFER catch_statement
    ;

catch_statement
    : CATCH '(' type_expression IDENT ')' defer_catch_body
    | CATCH '(' ERROR IDENT ')' defer_catch_body
    ;

try_statement
    : TRY selection_statement
    | TRY iteration_statement
    | TRY jump_statement
    ;

volatile_statement
    : VOLATILE compound_statement
    ;

labeled_statement
	: IDENT ':' statement
	| CASE constant_expression ':' statement
	| DEFAULT ':' statement
	;

compound_statement
	: '{' '}'
	| '{' statement_list '}'
	;

statement_list
	: statement
	| statement_list statement
	;

declaration_statement
    : declaration ';'
    ;

expression_statement
	: ';'
	| expression ';'
	;


control_expression
    : expression
    | declaration_list ';' expression
    ;

control_statement
    : expression_statement
    | declaration_list ';'
    ;


selection_statement
	: IF '(' control_expression ')' statement
	| IF '(' control_expression ')' compound_statement ELSE compound_statement
	| SWITCH '(' control_expression ')' statement
	;

iteration_statement
	: WHILE '(' control_expression ')' statement
	| DO statement WHILE '(' expression ')' ';'
	| FOR '(' control_statement expression_statement ')' statement
	| FOR '(' control_statement expression_statement expression ')' statement
	;

jump_statement
	: GOTO IDENT ';'
	| CONTINUE ';'
	| BREAK ';'
	| RETURN ';'
	| RETURN expression ';'
	;

attribute
    : AT_IDENT
    | IDENT SCOPE AT_IDENT
    | AT_IDENT '(' constant_expression ')'
    | IDENT SCOPE AT_IDENT '(' constant_expression ')'
    ;

attribute_list
    : attribute
    | attribute_list attribute
    ;

opt_attributes
    : attribute_list
    |
    ;

error_type
    : IDENT SCOPE TYPE_NAME
    | TYPE_NAME
    | ERROR '(' expression ')'
    ;

error_list
    : error_type
    | error_list error_type
    ;

throw_declaration
    : THROWS
    | THROWS error_list
    ;

func_name
    : IDENT SCOPE TYPE_IDENT '.' IDENT
    | TYPE_IDENT '.' IDENT
    | IDENT
    ;

func_declaration
    : FUNC type_expression func_name '(' parameter_type_list ')' opt_attributes
    | FUNC type_expression func_name '(' parameter_type_list ')' opt_attributes throw_declaration
    ;

func_definition
    : func_declaration compound_statement
    | func_declaration ';'
    ;

macro_declaration
    : MACRO AT_IDENT '(' macro_argument_list ')' compound_statement
    ;


struct_or_union
	: STRUCT
	| UNION
	;

struct_declaration
    : struct_or_union TYPE_IDENT opt_attributes struct_body
    ;

struct_body
    : '{' struct_declaration_list '}'
	;

struct_declaration_list
    : struct_member_declaration
    | struct_declaration_list struct_member_declaration
    ;

struct_member_declaration
    : type_expression identifier_list ';'
    | struct_or_union IDENT struct_body
    | struct_or_union struct_body
	;

enum_declaration
    : ENUM TYPE_IDENT ':' type_expression '{' enumerator_list '}'
    | ENUM TYPE_IDENT '{' enumerator_list '}'
    ;

error_declaration
    : ERROR TYPE_IDENT '{' identifier_list '}'
    ;

type_list
    : type_expression
    | type_list ',' type_expression
    ;

generics_case
    : CASE type_list ':' statement

generics_body
    : generics_case
    | generics_body generics_case
    ;

generics_declaration
    : GENERIC IDENT '(' macro_argument_list ')' '{' generics_body '}'

const_declaration
    : CONST CT_IDENT '=' initializer ';'
    | CONST type_expression IDENT '=' initializer ';'
    ;

typedef_declaration
    : TYPEDEF type_expression AS IDENT ';'
    | TYPEDEF func_declaration AS IDENT ';'
    ;

attribute_domain
    : FUNC
    | VAR
    | ENUM
    | STRUCT
    | UNION
    | TYPEDEF
    | CONST
    ;

attribute_domains
    : attribute_domain
    | attribute_domains ',' attribute_domain
    ;

attribute_declaration
    : ATTRIBUTE AT_IDENT attribute_domains
    | ATTRIBUTE AT_IDENT attribute_domains '(' parameter_type_list ')'
    ;

global_declaration
    : type_expression IDENT ';'
    | type_expression IDENT '=' initializer ';'
    ;

ct_if
    : CTIF '(' expression ')'
    ;

ct_elif
    : CTELIF '(' expression ')'
    ;

ct_switch
    : CTSWITCH '(' expression ')'
    ;

top_level_block
    : '{' top_level_statements '}'
    ;

tl_ct_elif_body
    : ct_elif top_level_block
    | tl_ct_elif_body ct_elif top_level_block
    ;

tl_ct_else_body
    : tl_ct_elif_body
    | tl_ct_else_body CTELSE top_level_block
    ;

tl_ct_case
    : CTCASE type_list ':' top_level_statements
    | CTDEFAULT ':' top_level_statements
    ;

tl_ct_switch_body
    : tl_ct_case
    | tl_ct_switch_body tl_ct_case
    ;

conditional_compilation
    : ct_if top_level_block
    | ct_if top_level_block tl_ct_else_body
    | ct_switch '{' tl_ct_switch_body '}'
    ;

module
    : MODULE IDENT ';'
    ;

import_decl
    : IMPORT IDENT ';'
    | IMPORT IDENT AS IDENT ';'
    | IMPORT IDENT AS IDENT LOCAL ';'
    | IMPORT IDENT LOCAL ';'
    ;

imports
    : import_decl
    | imports import_decl
    ;

translation_unit
    : module imports top_level_statements
    ;

top_level_statements
    : visibility top_level
    | top_level_statements visibility top_level
    ;

visibility
    : LOCAL
    | PUBLIC
    | LOCAL PUBLIC
    | PUBLIC LOCAL
    ;

top_level
	: func_definition
	| conditional_compilation
	| struct_declaration
	| attribute_declaration
	| enum_declaration
	| error_declaration
	| const_declaration
	| global_declaration
	| macro_declaration
	| generics_declaration
	| typedef_declaration
	;


%%
#include <stdio.h>

extern char yytext[];
extern int column;

yyerror(s)
char *s;
{
	fflush(stdout);
	printf("\n%*s\n%*s\n", column, "^", column, s);
}
```