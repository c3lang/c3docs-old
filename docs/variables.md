# Variables


### Zero init by default

Unlike C, C3 local variables are zero-initialized by default. To avoid zero-init, you need to explicitly opt-out.

```
int x;              // x = 0
int y = void;       // y is explicitly undefined and must be assigned before use.

AStruct foo;        // foo is implicitly zeroed
AStruct bar = {};   // boo is explicitly zeroed
AStruct baz = void; // baz is explicitly undefined
```

Using a variable that is explicitly undefined before will trap in debug builds and is undefined behaviour in release builds.