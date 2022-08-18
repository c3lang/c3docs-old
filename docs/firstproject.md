# Your First Project

Starting out with C3, you probably want to get a feel for the language, without using the integrated build system.

Open a text editor and enter the following in a file you call `hello_world.c3`:

```
module hello_world;
import std::io;

fn int main(char[][] argv) 
{
    io::println("Hello World!");
    return 0;
}
```

Now in the terminal type:

```
$ c3c compile hello_world.c3
$ ./hello_world
Hello World
$ 
```

## A real project

Once you go beyond simple files, you want to create a real project. Do so by entering `c3c init hello_world`.

You will get the following structure:


```
$ c3c init hello_world 
$ tree .
.
└── hello_world
    ├── LICENSE
    ├── README.md
    ├── build
    ├── docs
    │   ├── about.md
    │   └── src
    │       └── index.html
    ├── lib
    ├── project.c3p
    ├── resources
    ├── src
    │   └── hello_world
    │       └── main.c3
    └── test
        └── hello_world
```

Enter main.c3 and write the same code as above, then anywhere in the project structure:

```
$ c3c run
Hello World
$ 
```

