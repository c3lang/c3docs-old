# Expressions

Expressions work like in C.

## Compound literals

_Syntax and implementation under consideration!_

C3 has C's compound literals, but it uses C++ style initialization without the `()`.

```
struct Foo
{
    int a;
    double b;
}

...
func void test(Foo x) { ... }

test(Foo { 1, 2.0 });
```

Literals are allocated on the stack, and similarly it's possible to allocate fixed size integers:

```
// By value
func void test1(int[3] x) { ... }

// By slice
func void test2(int[:] y) { ... }

// By reference
func void test2(int[3]* z) { ... }

test1(int[3] { 1, 2, 3 });
test2(&int[3] { 1, 2, 3 });
test3(&int[3] { 1, 2, 3 });
```

## Array literals

_Syntax and implementation under consideration!_

Arrays can be initialized using compound literals, but there is also a special format for array initialization using `[]`. In this case, the type is inferred. It allows uniform initialization of all types of arrays:

```
int[] x = [1, 2, 3]; // Variable array allocated on the stack
int[3] y = [1, 2, 3]; // Fixed array allocated on the stack
int[] z = [1, 2, 3]; // Slice pointing at literal allocated on the stack
```

It offers some convenience when calling functions taking arrays:

```
func void test1(int[3] x) { ... }
func void test2(int[] y) { ... }
func void test2(int[3]* z) { ... }

test1([ 1, 2, 3 ]);
test2([ 1, 2, 3 ]);
test3([ 1, 2, 3 ]);
```

## Associative array literals

_Syntax and implementation under consideration!_

Associative arrays are mappings between keys and values:

```
IntStringMap x = { "a": 2, "b": { "3" : "foo" } };
```

To use an associative array, the struct needs to define the following method macros: `@map_init()` `@map_init_add()`. In the example above, the actual code is compiled to:

```
IntStringMap x;
x.@map_init();
x.@map_init_add("a", 2);
IntStringMap _temp;
_temp.@map_init();
_temp.@map_init_add("3", "foo");
x.@map_init_add("b", _temp);
```

