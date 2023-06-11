# Generics

Generic modules are parameterized modules that allow functionality for arbitrary types.

For generic modules, the generic parameters follows the module name:

```
// TypeA, TypeB, TypeC are generic parameters.
module vector <TypeA, TypeB, TypeC>;
```

The code inside of the module can use the generic parameters as if they were well defined symbols:

```
module foo_test <Type1, Type2>;

struct Foo 
{
   Type1 a;
}

fn Type2 test(Type2 b, Foo *foo) 
{
   return foo.a + b;
}
```

Including a generic module works as usual, but to use a type, it must be *defined* before use.

```
def FooFloat = Foo<float, double>;
def test_float = foo_test::test<float, double>;

...

FooFloat f;

...

test_float(1.0, &f);

```

Just like for macros, optional constraints may be added to improve compile errors:

```
/**
 * @require $checks(TypeA a = (TypeB)1 + (TypeC)1)
 */ 
module vector <TypeA, TypeB, TypeC>;

/* .. code * ../
```

```
def testFunction = vector::testFunc<Bar, float, int>;

// This would give the error 
// --> Parameter(s) failed validation: 
//     @require "$checks(TypeA a = (TypeB)1 + (TypeC)1)" violated.
```

