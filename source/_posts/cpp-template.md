---
title: C++ Template
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 1231239
date: 2024-02-25 00:00:00
updated: 2024-10-28 00:00:00
---

A template is the same as **Macro** in C, but a far less evil one. Compiler could do more checking to ensure the safety of the program.

Besides, template is also an implementation of {% post_link cpp-polymorphism 'static polymorphism' %} since it will generate different kinds function of the same name during compile time.

## What is template

The compiler **substitutes** the template parameters into the functions and classes to make more them more dynamic.

Since template is essentially doing the substitution, then **the template parameter should be known at compile-time**.

The template parameters can be one of these:
- type
- value
    - pointers (including function pointers!)
    - references
    - integer constant expressions (i.e. can be evaluated in compile time)

## Why template
1. Values could be precomputed in compile-time
2. Compiler optimization (e.g., loop unrolling etc.)
3. More generic

## Exercise

### Putting template function definitions in header or source files? Why?

The answer is header files. 

Recall from the {% post_link compiler-basics 'compilation process' %}, we first have multiple translation units (TUs) and then we will link them together.

Now let's suppose we put the definition in source file and consider a scenario where
* TU A: include the header instantiate the template functions
* TU B: includes the source file of template function definition

When we are compiling TU B, it will have no idea what kind of template function TU A needs. Therefore, TU A will have no usable function definition when linking.


### Pass function to template

```cpp=
#include <iostream>

// fn should be deducted to a function pointer value
template<auto fn, int sz>
int wrapper1() {
    return fn(sz);
}

// fn_t is a function pointer type
template<typename fn_t>
int wrapper2(fn_t fn, int sz) {
    return fn(2);
}

constexpr int square(int x) {
    return x * x;
}

int main() {
    int sz = 10;
    std::cout << wrapper1<square, sz>();    // ERROR: sz should be constant
    
    constexpr int const_sz = 10;
    wrapper1<square, const_sz>();    // OK
    
    wrapper2<decltype(square)>(square, sz) ;    // OK
    wrapper2(square, sz);    // OK, template param is deducted by the compiler
    
}
```

## Reference
- [What does template <unsigned int N> mean?](https://stackoverflow.com/questions/499106/what-does-template-unsigned-int-n-mean)
- [Wikipedia - 模板元編程](https://zh.wikipedia.org/zh-tw/模板元編程)