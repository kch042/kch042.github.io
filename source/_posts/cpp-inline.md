---
title: C++ Inline
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 123481
date: 2024-10-28 00:00:00
updated: 2024-10-28 00:00:00
---

Originally, the keyword `inline` used to be an indicator to the compiler that the function is preferred to be inline substituted rather than a function call.

That semantics changed over time and since C++17, it is used to indicate that a function could have been definied more than once. (Notice that it is not overloading, it means that a function with same signature could have multiple definition)

> Because the meaning of the keyword inline for functions came to mean "multiple definitions are permitted" rather than "inlining is preferred" since C++98, that meaning was extended to variables. (Since C++17)

However, note that it doesn't mean that the function should have different definitions. In the end, all translation units will use the same definition and share the function-local objects! (exception: static functions, see the [excercise](#Excercise) below)

## Recap: Compilation process
* Preprocess: Do the substitution and symbol analysis
* Compile: Gather translation units and generate object files
* Link: Link the objects file together into a executable (Find function definitions for each function call)

When we do the `#include "myheader.h"` inside a source file, the compiler will essentially substitute this line with the contents in the header during the preprocessing stage.

## Function definition inside header file
Usually, we will only **declare** function in the header file but leave the implementation (i.e. function definition) in the source file. Why don't we include definitions in the header file?

When we include function definitions in the header file, different source files that include this header file will all contain the function definitions, therefore the linker will notice that a function has been defined more than one times, so it will throw error since it doesn't know which definition to use.

If we define the function inside the source file, the definition will only have one copy and therefore the linking won't break.

## Functions definied more than once but same definition

In modern C++, the keyword `inline` allows a function to have more than one definitions.

A classical example is multiple translation units using an inline function defined in a header.

```cpp=
// helper.h
# include <iostream>
inline void sayHi() {
    std::cout << "Hi\n";
}
```

```cpp=
// source1.cpp
# include <iostream>
# include "helper.h"
void Source1SayHi() {
    std::cout << "This is source 1\n";
    sayHi();
}
```

```cpp=
// source2.cpp
# include <iostream>
# include "helper.h"
void Source2SayHi() {
    std::cout << "This is source 2\n";
    sayHi();
}
```

```cpp=
// main.cpp
void Source1SayHi();
void Source2SayHi();

int main() {
    Source1SayHi();
    Source2SayHi();
}
```

With `inline` indicated, different translation units will **share the same function definitions and local variables/types**!

In this case, the linker will see that the function `sayHi()` has been definied more than once, therefore it will pick one of them to use.

The above programs can be compiled with the following makefile.
```makefile=
# Compiler
CXX = g++

# Compiler flags
CXXFLAGS = -Wall -Wextra -std=c++17

# Source files
SRCS = main.cpp source1.cpp source22.cpp helper.cpp

# Object files
OBJS = $(SRCS:.cpp=.o)

# Target executable
TARGET = main

# Default target
all: $(TARGET)

# Link the object files to create the executable
$(TARGET): $(OBJS)
	$(CXX) $(OBJS) -o $(TARGET)

# Compile each .cpp file into an .o file
%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

# Clean up generated files
clean:
	rm -f $(OBJS) $(TARGET)

# Phony targets
.PHONY: all clean
```

## Excercise

### Exercise 1: Functions definied more than once but different definitions

```cpp=
// helper.h
# include <iostream>
inline void sayHi() {
    std::cout << "Hi\n";
}
```

```cpp=
// source1.cpp
# include <iostream>
# include "helper.h"
void Source1SayHi() {
    std::cout << "This is source 1\n";
    sayHi();
}
```

```cpp=
// source2.cpp
# include <iostream>

inline void sayHi() {
    std::cout << "Hiiiiiiiiiiiiiiiiiii\n";
}

void Source2SayHi() {
    std::cout << "This is source 2\n";
    sayHi();
}
```

```cpp=
// main.cpp
void Source1SayHi();
void Source2SayHi();

int main() {
    Source1SayHi();
    Source2SayHi();
}
```

What would be the output of `main`?

The answer is 
```bash=
$ ./main
Hi
This is source 1
Hi
This is source 2
```

### Exercise 2: Static functions definied more than once but different definitions

```cpp=
// helper.h
# include <iostream>
static inline void sayHi() {
    std::cout << "Hi\n";
}
```

```cpp=
// source1.cpp
# include <iostream>
# include "helper.h"
void Source1SayHi() {
    std::cout << "This is source 1\n";
    sayHi();
}
```

```cpp=
// source2.cpp
# include <iostream>

static inline void sayHi() {
    std::cout << "Hiiiiiiiiiiiiiiiiiii\n";
}

void Source2SayHi() {
    std::cout << "This is source 2\n";
    sayHi();
}
```

```cpp=
// main.cpp
void Source1SayHi();
void Source2SayHi();

int main() {
    Source1SayHi();
    Source2SayHi();
}
```

The output is 
```bash=
$ ./main
Hi
This is source 1
Hiiiiiiiiiiiiiiiiiii
This is source 2
```

This is because a static function is only visible in its own translation units. So the function will not use definitions from external linkage.

## Reference
* [cppreference: inline specifier](https://en.cppreference.com/w/cpp/language/inline)