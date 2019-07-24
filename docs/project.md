# Customizing A Project

A new project is provided with a barebone structure:


```toml
[[executable]]
# name of the target
name = "hello_world"
# version using semantic versioning
version = "0.1.0"
# authors, optionally with email
authors = ["John Doe <john.doe@example.com>"]
# language version of C3
langrev = "1"
# warnings used
warnings = ["no-unused"]
# sources compliled
sources = ["src/**"]
# libraries to use
libs = ["lib/**"]
```

Libraries look a little different:

```toml
[[static-lib]]
name = "graphics"
version = "0.1.0"
authors = ["John Doe <john.doe@example.com>"]
langrev = "1"
warnings = ["no-unused"]
sources = ["src/**"]
# exported modules
export = ["api"]
```


## Target options

#### config

Under the config you define external constants ("key = value") that will be included in compilation as if they were global macro constants.

#### export

Define the list of modules to be exported by a library. Not valid for executables.

#### generate

C3 defaults to generating C code that is then compiled and remove. Simply compile to C without further compilation by setting generate = "C"

#### warnings

List of warnings to enable during compilation.

#### lib

List of libraries to use when compiling the target.

#### macro-recursion-depth

Set the depth for recursion of macros. Typically set to a value around 10,000.

## Using environment variables

In addition to constants any values starting with "$" will be assumed to be environment variables.

For example "$HOME" would on unix systems return the home directory. For strings that start with $ but *should not* be interpreted as an environment variable. For example, the string `"\$HOME"` would be interpreted as the plain string "$HOME"