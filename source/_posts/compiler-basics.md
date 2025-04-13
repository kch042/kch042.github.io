---
title: Compiler Basics
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: Compiler
abbrlink: 55949
date: 2022-05-15 00:00:00
updated: 2023-10-30 00:00:00
---

This article covers the high level view of compiler process and serves as a beginner tutorial into compiler.

## Compiling Procedure

1. Preprocess
2. Compile
3. Assemble
4. Link

### Preprocess
Substitute macro into the source file (.c)

**Macro** is a  piece of code using **#define** 

For example:
```c=
#define x 10

int main() {
    int y = x;
}
```

After preprocessing, it will becomes
```c=
int main() {
    int y = 10;
}
```

It's also useful to use macro to replace small function.
As building and using another function is more expensive than just replacing the code of function called.
```c=
#define min(a, b) ((a > b) ? b : a)

int main() {
    // The following will become
    // int c = ((10 > 20) ? 10 : 20)
    // after preprocessing
    int c = min(10, 20)
}
```


### Compile + Assemble
Compiler performs a sereis of ops, including
- Lexical and Semantic Analysis
- Syntax checking
- Collect and Summarize all symbols (will talk later)
- Code optimization

and finally generates an assembly file which will be converted to machine codes by the assembler.

The resulting file is called the object file (.o)

#### Translation unit
A translation unit is the basic unit of compilation.

A translation unit is **a source file + all header files it includes**, which will then be converted to a single object file.

.c + many .h  ---> .o

For exmaple, 
Suppose we have 3 files: *main.c*, *func.c*, *func.h*
- *main.c* contains the main() routine.
- *func.c* contains the function that main() uses.

```c=
// main.c
#include "func.h"    

int main() {
    myFunc();
}
```
```c=
// func.h
void myFunc();
```
```c=
// func.cpp
void myFunc() {
    // do something...
}
```

Here we have **2** translation units, which means that we need to produces 2 object files.
```shell
$ gcc -c main.c func.c  # produces main.o and func.o
```

To produce the final executable, we need to link the two object file together, and here comes the **linker**.

### Link
Before talking about linker, think a question:
Why do we include header file in source file ?

The answer is to tell the compiler that **we have** the function in the header file to use. 

Every function and variable is a **symbol** to the compiler, compiler will collect and summarize the symbols used in the translation unit into a symbol table.

When the program calls a function, it will refer to the symbol table to find the location of that function.

Here comes the trick, **compiler is NOT responsible to know the location of every symbol**. (since some function or variable may be implemented in other translation units.)

To convert object files into an executable, we need to make sure we know location of every symbol, and that is the job of linker.

```shell
$ gcc -o main main.o func.o  # produces main
```

## Template

Please refer to this [How does the compilation of templates work?](https://stackoverflow.com/questions/19798325/how-does-the-compilation-of-templates-work)

The takeaway is, when you have instantiate a class with a specific type, the compiler will generates a class of that type.

So **if you have class with a type parameter `T` in a header file (`.h` or `.hpp`), then every method involving `T` must have definition (i.e. implementation) in that head file.** Otherwise, the compiler will have no clue as to how to generate the code of that type.

## Further topics
- makefile
- static/extern
- inline