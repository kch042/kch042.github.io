---
title: C++ Uniform Initialization
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 45415
date: 2024-02-20 00:00:00
updated: 2024-02-20 00:00:00
---

## Constructor

Here's a recap.

```cpp=
class myClass {
public:
    myClass() {cout << "Default Constructor\n";}
    myClass(const myClass& mycls) {cout << "Copy Constructor\n";}
    myClass& operator=(const myClass& mycls) {
        cout << "Assignment Operator\n"; 
        return *this;
    }
};

int main() {
    myClass m1;         // Default Constructor
    myClass m2(m1);     // Copy Constructor
    myClass m3{m1};     // Copy Constructor
    myClass m4 = m1;    // Copy Constructor (implicitly converted)
    m4 = m2;            // Assignment Operator
}
```

Note that in the case of `m4`, we implicitly convert the call to copy constructor. We could avoid that by adding the keyword `explicit` before the constructor to prevent misusage.

```cpp=
class myClass {
public:
    myClass() {cout << "Default Constructor\n";}
    explicit myClass(const myClass& mycls) {cout << "Copy Constructor\n";}
    myClass& operator=(const myClass& mycls) {
        cout << "Assignment Operator\n"; 
        return *this;
    }
};

int main() {
    myClass m1;
    myClass m4 = m1;    // ERROR: no matching constructor
}
```

## Initializer List & Uniform Initialization

Initializer list is associated with **constructor that takes the `std::initializer_list` as the first input**. For example, in C++14, the vector has a constructor of signature
```cpp=
vector(initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
```

Uniform initialization is introduced in C++11 and refers to **initializing variables with curly braces**. This way it will make initialization **consistent** regardless of the variable type. Moreover, it **doesn't allow narrow conversion** and thus more type safe. For example,
```cpp=
string s{"abc"};
int i{1};

double d{1.0};
int i2{d};    // ERROR: need to explicit cast `d` to int
```

### Some conflicts between them

When a class has multiple constructors which includes the one that takes `std::initializer_list` as the input, we could run into some ambibuity.

For example, we would expect the code to print `First!`.
```cpp=
class Bar {
public:
    Bar(int x, double y) {cout << "First!\n";}
    Bar(std::initializer_list<bool> init_list) {cout << "Second!\n";}
};

int main() {
    Bar bar{1, 0.5}; // takes the second constructor, and causes ERROR
}
```

Unfortunately, the code will NOT print `First!`.

**The problem is that, when instantiating an object with uniform initialization, the compiler has high preference to select the constructor that takes in the `initializer_list`.**

Moreover, since the above example is using a initializer list of type `bool`, we are doing implicit narrow conversion which is NOT allowed in uniform initialization. Therefore an error will be thrown by the compiler.

If we change the type here, the error will be gone and we won't even notice we are using the wrong constructor!
```cpp=
class Bar {
public:
    Bar(int x, int y) {cout << "First!\n";}
    Bar(std::initializer_list<int> init_list) {cout << "Second!\n";}
};

int main() {
    Bar bar{1, 2};    // Second!
}
```

## Most vexing parse

Consider the following code
```cpp=
A Foo();
```

This line could be interpreted as both **object creation** and **function declaration** at the same time! This ambiguity is the so called **most vexing parse**.


Consider another example that instantiates a `std::vector` in the class.
```cpp=
class Bar {
private:
    vector<int> myvec(5, 0); // ERROR: the compiler assumes it is a function declaration
public:
    Bar() {}    
};
```

The introduction of uniform initialization somehow eases the pain of this problem,
```cpp=
class Bar {
private:
    vector<int> myvec{0, 0, 0, 0, 0}; // OK
public:
    Bar() {}    
};
```

But if we insist using the constructor that takes the input`(size, default_value)` (probably due to too large `size` so uniform initializaion is not practical), we could either move the initialization to the constructor
```cpp=
class Bar {
private:
    vector<int> myvec;
public:
    Bar(): myvec(5, 0) {}
};
```
or do the **copy initialization** (will be opitimzed in C++17, see {% post_link cpp-copy-elision-rvo 'Copy Elision & RVO' %} for more)

```cpp=
class Bar {
private:
    vector<int> myvec = vector<int>(5, 0);
public:
    Bar() {}
};
```

## Conclusion

**Important Guideline**

Use uniform initialization (i.e. use curly braces) whenever possible for consistency and type safety. 

Explicity call the constructor using paranthesis when the  constructor you want to use is hidden by the constructor of initializer list. (e.g. `vector<int>(5, 0)`)

## Reference

Highly recommend these two 
- [C++ Uniform Initialization - Benefits & Pitfalls](https://ianyepan.github.io/posts/cpp-uniform-initialization/)
- [GotW #1 Solution: Variable Initialization – or Is It?](https://herbsutter.com/2013/05/09/gotw-1-solution/)