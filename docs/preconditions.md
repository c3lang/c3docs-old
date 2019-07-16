# Pre and post conditions

Pre and post conditions are optional checks that the compiler may use for optimization and runtime checks. Note that _compilers are not obliged to process pre and post conditions at all_. However, violating either pre or post conditions is considered undefined behaviour, so a compiler may optimize as if they always hold – even if a potential bug may cause them to be violated.

# Pre conditions

Pre conditions are usually used to validate incoming arguments. Each condition must be an expression that can be evaluated to a boolean. 

# Post conditions

Post conditions are evaluated to make checks on the resulting state after passing through the function. There is a special post condition called "const condition", which guarentees that the memory pointed to is not altered within the scope of a function.

