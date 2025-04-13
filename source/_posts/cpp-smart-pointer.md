---
title: C++ Smart Pointer
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 90119
date: 2023-11-08 00:00:00
updated: 2025-05-01 00:00:00
---

## TL;DR

1. Use `make_unique` or `make_shared` instead of `new`.
```cpp
// The following two expressions have similar effect
new T(args);
make_unique<T>(args);

// So the following two lines are equal 
// except the later one is exception-safe
unique_ptr<int> p1(new int(50));
unique_ptr<int> p1 = make_unique<int>(50);
```

2. Shared pointer

Each shared pointer is an object consisting of
- a pointer to the resource
- a pointer to the control block (this includes reference count)

Every shared pointer sharing a resource share the same control block.

**The last person going out the room has the responsiblity to turn off the light.**


## Exercise: How to pass the unique pointer to a function

Ans: two ways
- pass the unique pointer by reference
- move the pointer into the function argument
    - caveat: the ownership of the pointer will be transferred, as the move constructor will be called to construct the function argument

```cpp=
void func1(unique_ptr<int> ptr) {
    
}

void func2(unique_ptr<int>& ptr) {
    
}

int main() {
    unique_ptr<int> ptr;
    func1(ptr);            // ERROR
    func2(ptr);            // OK
    func1(std::move(ptr)); // OK
}
```

The reaseon that line 10 fails is that the **copy constructor of unique pointer is deleted**.
```cpp=
// might look something like this
template<class T>
unique_ptr<T>::unique_ptr(const unique_ptr<T>&) = delete;
```