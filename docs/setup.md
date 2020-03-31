# Setup

C3 is not ready for end users yet, but should be possible to get it up and running on any platform that LLVM can compile on. You will need CMake installed.

## 1. Install LLVM

See LLVM the [LLVM documentation](https://llvm.org/docs/GettingStarted.html) on how to set up LLVM 10 for development. On OS X, installing through Homebrew works fine.
Using apt-get on Linux should work fine as well.

## 2. Clone the C3 compiler source code from Github

This should be as simple as doing:

```
git clone https://github.com/c3lang/c3c.git
```

... from the command line.

## 3. Build the compiler

Create the build directory:

```
MyMachine:c3c$ mkdir build
MyMachine:c3c$ cd build/
```

Use CMake to set up:

```
MyMachine:c3c/build$ cmake ../
```

Build the compiler:

```
MyMachine:c3c/build$ make
```

## 4. Test it out

```
MyMachine:c3c/build$ ./c3c compile ../resources/testfragments/helloworld.c3
```

