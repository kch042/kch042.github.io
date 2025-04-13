---
title: C++ Copy Elision & RVO
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 19415
date: 2023-11-07 00:00:00
updated: 2024-02-20 00:00:00
---

This page introduces the copy elision and return value optimization feature in C++.

## Prerequisites
This article: {% post_link cpp-lvalue-rvalue-move-swap-forward 'lvalue, rvalue, move, swap, forward' %}

### Recap
- A function call returning value is considered an rvalue
- A function call returning lvalue reference is an lvalue

```cpp=
class A {
public:
    A& get() {
        return *this;    
    }
};

A makeA() {
    A a;
    return a;
}

int main() {
    A a;
    
    A.get();    // lvalue
    makeA();    // rvalue
    
    // so they can bind to lvalue/rvalue reference , respectively
    A& a2 = A.get();
    A&& a3 = makeA();
}
```

Specifically, `makeA()` is a prvalue, see [this](https://stackoverflow.com/questions/45317763/xvalues-vs-prvalues-what-does-identity-property-add) for more.

## Example
```cpp
#include <iostream>

class Thing {
public:
    Thing() {std::cout << "Constructor\n";}
    ~Thing() {}
    Thing(const Thing&) {std::cout << "Copy Constructor\n";}
};

Thing getThing() {
    Thing s;
    return s;
}
 
int main() {
  std::cout << "Hello World!\n";
  Thing t = getThing();
}
```

In theory, the program should output
```
Hello world
Constructor
Copy Constructor
Copy Constructor
```

**Why**
1. `getThing()` initialize a local variable `s` of class `Thing`, which triggers the constructor of `Thing` one time. 
2. Then, `getThing` copies `s` to a temporary space that holds the return object, which triggers copy constructor. 
3. Finally, the object inside the temporary space is copied to the space of `t`, which triggers the copy constructor again.

**However**
After compiled with C++14 (or 11)
```bash
$ g++ -o test -std=c++14 test.cpp
$ ./test
Hello world
Constructor
```

**No copy!!**

This is happens because compiler does some optimization for us.

For now, we can try to turn off the optimization using the flag `-fno-elide-constructors` and see the result.
```bash
$ g++ -fno-elide-constructors -o test -std=c++14 test.cpp
$ ./test
Hello world
Constructor
Copy Constructor
Copy Constructor
```

The result matches what we expect in the beginning.

Furthermore, if we implement move constructor, it will use move constructor instead of copy constructor.
```cpp
class Thing {
public:
    Thing() {cout << "Constructor\n";}
    ~Thing() {}
    Thing(const Thing&) {cout << "Copy Constructor\n";}
    Thing(Thing&&) {cout << "Move Constructor\n";}
};
```
```bash
$ g++ -fno-elide-constructors -o test -std=c++14 test.cpp
$ ./test
Hello world
Constructor
Move Constructor
Move Constructor
```

If we compile with C++17, then we would get the result
```bash
$ g++ -fno-elide-constructors -o test -std=c++17 test.cpp
$ ./test
Hello world
Constructor
Move Constructor
```

One less move. It's because in C++17, the space of `t` is used as the temporary space for holding the returned object.

### Return Value Optimization (RVO)
Let's get back to the optimization compiler do for us. What exactly does the it do? 

In fact, this technique is called **Return Value Optimization**, or **Copy/Move Ellision**. 

If an object is returned by value in a function, then it will create the object inside the space of `t` instead of the local variable `s`. This way, we can avoid two times of moving/copying.

Two notes:
1. If the returned object is an lvalue (i.e. has name), we will call it **Named Return Value Optimization (NRVO)**. In this case, our example uses NVRO.
2. Before C++17, RVO is an optimization. And from on, it is a standard.

## Exercises: return std::move(obj);
```cpp
class Thing {
public:
    Thing() {cout << "Constructor\n";}
    ~Thing() {}
    Thing(const Thing&) {cout << "Copy Constructor\n";}
    Thing(Thing &&) {cout << "Move Constructor\n";}
};

Thing getThing() {
    Thing s;
    return std::move(s);
}

int main() {
    Thing t = getThing();
}
```

What should be the output compiled with C++14?
The answer is 
```
Constructor
Move Constructor
Move Constructor
```
By casting the return value to rvalue, it prevents move from being ellided. In fact, it is essentially doing
```cpp
Thing s;
Thing tmp = std::move(s);    
Thing t = std::move(tmp);    // use move instead of copy
```

## References
1. [What are copy elision and return value optimization](https://stackoverflow.com/questions/12953127/what-are-copy-elision-and-return-value-optimization)
2. [Copy/move elision: C++ 17 vs C++ 11](https://zhuanlan.zhihu.com/p/379566824)