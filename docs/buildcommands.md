# Build Commands

When starting out, with C3 it's natural to use `compile` and `compile-run` to try things out. For larger projects, the built-in build system is instead recommended.

## compile

The default for c3c is compiling stand-alone files to output a binary.

`c3c compile <file1> <file2> <file3>`

## compile-run

The same as `compile` but then also runs the executable.

## Common additional parameters

Additional parameters:
- `--lib <path>` add a library to search.
- `--output <path>` override the output directory.
- `--path <path>` execute as if standing at <path>
    
## init

`c3c init <project_name> [optional path]`.

Create a new project structure in the current directory.

Use the `--template` to select a template. The following are built in:

- `default` - the default template, produces an executable.
- `lib` - template for producing a library.
- `staticlib` - template for producing a static library.

It is also possible to give the path to a custom template.

Additional parameters:
- `--template <path>` indicate an alternative template to use. 

`c3c init hello_world` will create the following structure:

```
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
    ├── project.toml
    ├── resources
    ├── src
    │   └── hello_world
    │       └── main.c3
    └── test
        └── hello_world
```
## build

`build [target]`

Build the project in the current path. It doesn't matter where in the project structure you are. 

The built in templates define two targets: `debug` (which is the default) and `release`.

## clean

`clean`

## run

`run [target]`

Build the target (if needed) and run the executable.

## clean-run

`clean-run [target]`

Clean, build and run the target.

## dist

`dist [target]`

Clean, build and package the target.

## docs

`docs [target]`

Rebuilds the documentation.

## bench

`bench [target]`

Runs benchmarks on a target.