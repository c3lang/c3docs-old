# Build Commands

## Compiling files directly

The default for c3c is compiling stand-alone files to output a binary.

`c3c <file1> <file2> <file3>`

## Common additional parameters

Additional parameters:
- `-lib <path>` add a library to search.
- `-output <path>` override the output directory.
- `-path <path>` execute as if standing at <path>
    
## new

`c3c -new <project_name> [optional path]`.

Create a new project structure in the current directory.

Additional parameters:
- `-template <path>` indicate an alternative template to use.

`c3c -new hello_world` will create the following structure:

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

## clean

`clean`

## run

`run [target]`

Build the target (if needed) and run the executable.

## run-clean

`run-clean [target]`

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