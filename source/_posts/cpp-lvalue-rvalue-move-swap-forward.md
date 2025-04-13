---
title: C++ lvalue, rvalue, move, swap, forward
cover: /img/cpp/cpp.png
mathjax: true
tags: C++
abbrlink: 39356
date: 2023-04-01 00:00:00
updated: 2025-04-10 00:00:00
---

The modern C++ starts from C++11 and introduces lvalue and rvalue. It completely redefines the lifecycle of objects in a program.

## Prerequisites
- constructor
- copy constructor
- reference
- stack/heap
- function/operator overloading

## Why

Imagine we have a vector named `v1` of size `10**6`, and we want to copy it to another variable `v2`.

Then in the first attempt, we would
- allocate a space of size `10**6` for `v2`
- copy the `10**6` elements to `v2`

The above is the so-called **deep copy**.

C++ vector provides a method called **copy constructor** to perform deep copy. The implementation looks like this
```cpp
// copy constructor
template<typename T>
std::vector(const vector<T>& v2) {
    int n = v2.size();
    for (int i = 0; i < n; i++)
        this->data[i] = v2.data[i];
}
```

If we don't need `v1` after copy to `v2`, the destructor of `v1` will be called and its memory will be freed when it is out of scope.

Then we might start thinking: can `v2` take over the elements of `v1` instead of making a copy?

## Interface: move constructor

In C++11, they introduce the move semantics, which makes the members of an object to be taken over from another one of the same class. (P.S. The members taken over should be in heap, not on stack)

The first step is to make a constructor that take over the data member from another object. Then it would look something like this

```cpp
// move constructor
template<typename T>
std::vector(const vector<int>& old) {
    // copy the pointer, only 1 op
    this->data = old->data;
    old->data = nullptr;
}
```
However, the above implementation has the same signature as the copy constructor. We need to make a little difference, and here comes the lvalue & rvalue.

## Value type

### Concepts
In general (from c++11 onward), we can divide the values into two types: `lvaue` and `rvalue`. 

Loosely speaking, we can define them to be
- `lvalue`:
    - is still alive after expression ended
    - has name
    - addressable
- `rvalue`:
    - is temporary and will be dead after expression
    - has no name
    - not addressable

```cpp
int a = 3;    // lvalue
int& b = a;   // lvalue reference (it is an lvalue)
int&& c = 3;  // rvalue reference (it is still a lvalue!!)
```

If we were to define a move constructor, then we certainly want it to take a rvalue as input to indicates that the value is temporary and won't be needed after.

```cpp
template<typename T>
std::vector(vector<T>&& old) {
    this->data = old->data;
    old->data = nullptr;
}
```

### Details
*This section is more advanced, and can be skipped.*

In fact, the values are categorized into `lvalue`, `xvalue`, `prvalue`.

Please see the [references](#References) for more.


## std::move

Once we have our move constructor, we only need to pass an object whose value type is rvalue. Howover, compiler **won't automatically** bind rvalue reference to lvalue reference, 
[in case of some accidental usage](https://stackoverflow.com/questions/35652953/why-c-lvalue-objects-cant-be-bound-to-rvalue-references)

```cpp
int main() {
    vector<int> v1(100);
    vector<int> v2(v1);    // copy constructor
    cout << v1.size() << "\n"; // 100
}
```

Instead, we have to explicitly cast the expiring object to rvalue type.
```cpp
int main() {
    vector<int> v1(100);
    vector<int> v2(static_cast<vector<int>&&>(v1));    // move constructor
    cout << v1.size() << "\n"; // 0
}
```

To make the code more readable, C++11 provides a syntax sugar `std::move` that does the casting for us
```cpp
template<typename T>
T&& std::move(T&& a) {
    return static_cast<T&&>(a);
}
```

So the code can be simplified to
```cpp
int main() {
    vector<int> v1(100);
    vector<int> v2(std::move(v1));    // move constructor
    cout << v1.size() << "\n";        // 0
}
```


There are two points about `std::move` to notice here

- The paramter of `std::move` is a [forwarding reference](https://stackoverflow.com/questions/47852347/why-does-stdmove-take-rvalue-reference-as-argument), not a rvalue reference. This would be another topic... (type deduction, reference collapsing). You can click [here](http://tommyjswu-blog.logdown.com/posts/1766009-c-11-mouth-cannon-universal-reference-rvalue-reference-and-move-semantics) if you want to know.
- The return value is a rvalue. In most circumstances, we don't want to return rvalue. See this discussion: [Does it make sense for a function to return an rvalue reference?](https://stackoverflow.com/questions/55958970/does-it-make-sense-for-a-function-to-return-an-rvalue-reference)

## std::swap

```cpp
template<typename T>
void swap(T& a, T& b) {
    T tmp(std::move(a));
    a = std::move(b);
    b = std::move(tmp);
}
```

## std::forward

Notice that the **rvalue reference is a lvalue**, so when we pass the an rvalue to a function `A()`, and in function `A()` we were to pass it to another function `B()`, then it would not be the rvalue anymore.

```cpp

void B(int& a) {
    cout << "This is lvalue\n";
}

void B(int&& a) {
    cout << "This is rvalue\n";
}

void A(int&& a) {
    B(a);
}

int main() {
    int a = 10;
    A(std::move(a));    // "This is lvalue".
}
```

To address this issue, we have another utility function called `forward()` to "forward" the original value category for us.

```cpp

void B(int& a) {
    cout << "This is lvalue\n";
}

void B(int&& a) {
    cout << "This is rvalue\n";
}

void A(int&& a) {
    // here a is a rvalue reference, which is a lvalue
    B(std::forward<int>(a));
}

int main() {
    int a = 10;
    A(std::move(a));    // "This is rvalue"
}
```

Note that `std::move` requires only a function argument, while `std::forward` **requires both a function argument and a template type argument**.


### forwarding reference (universal reference)

This section is the **generic case of the above example**.

Forwarding references are a special kind of references that preserve the value category of a function argument (so it accepts both lvalue and rvalue as the input), making it possible to forward it by means of `std::forward`. Forwarding references are either:

1) function parameter of a function template declared as rvalue reference to cv-unqualified type template parameter of that same function template.
2)`auto&&` except when deduced from a brace-enclosed initializer list.

In simple word, it is a variable **whose type is written as a rvalue reference to a type deduced by compiler when instantiating**.

```cpp
void B(int& a) {
    std::cout << "This is lvalue\n";
}

void B(int&& a) {
    std::cout << "This is rvalue\n";
}

template<class T>
void A1(T&& a) {
    // a is a forwarding reference since T is deduced when calling A1()
    B(std::forward<T>(a));
}

void A2(auto&& a) {
    // a is a forwarding reference since T is deduced when calling A2()
    B(std::forward<decltype(a)>(a));
}

int main() {
    int x = 20;
    
    // output: This is lvalue
    // T is int&, T&& collapes to int&
    A1(x);
    
    // output: This is rvalue
    // T is int, T&& collapes to int&&
    A1(10);
    
    // output: This is lvalue
    A2(x);
    
    // output: This is rvalue
    A2(10);
}
```

This is [another example](https://stackoverflow.com/a/28828432/14310073) for type deduction and also demonstrate the coordination between forwarding reference and `std::forward`.

```cpp

void func1(int&& arg) {
    // arg is a rvalue reference
}

template<class T>
void func2(T&& arg) {
    // arg is a forwarding reference (accepting both lvalue and rvalue)
}

template<class U>
class myclass {
public:
    void func3(U&& arg) {
        // arg is a rvalue reference (since the type of U is fixed when the class is instantiated, not deduced when calling this method)
    }
};

int main() {
    int x = 10;
    myclass<int> myinstance;
    
    func1(x);            // compile error: rvalue reference cannot bind to a
    func2(x);            // ok
    myinstance.func3(x); // compile error
}
```

## Exercise: copy or move?


Which one is produced by the following code, `"123"` or `""` ?
```cpp
int main() {
    string&& s1 = "123";
    string s2{s1};
    cout << s1 << "\n";
}
```

At first glance, it seems like the code triggers the move constructor of `string`, but in fact it uses the copy constructor. So the answer is that is outputs `"123"`;

The reason it uses copy constructor instead of move constructor is that **every named object is lvalue**, even if the variable's type is rvalue reference, the expression consisting of its name is an lvalue expression.

Both **lvalue reference** and **rvalue reference** are of **lvalue** cateogory.


[More info](https://stackoverflow.com/questions/54951635/why-to-use-stdmove-despite-the-parameter-is-an-r-value-reference)

## Exercise: function call

For this problem, please refer to [C++ value categories](https://en.cppreference.com/w/cpp/language/value_category).

```cpp=
class A {
    int data;
    
public:
    A() {}
    ~A() {}

    A& operator+() {
        return *this;
    }
    
    int&& getData() {
        return std::move(data);
    }
};

A makeA() {
    A a;
    return a;
}

int make10() {
    return 10;
}

int main() {
    A a;
    +a;          // expr1
    makeA();     // expr2
    make10();    // expr3
    a.getData(); // expr4
}
```

What are the value categories of these?

Ans:
- expr1: lvalue
- expr2: prvalue
- expr3: prvalue
- expr4: xvalue

Explain:
- lvalue: a function call or an overloaded operator expression, whose return type is **lvalue reference**
- xvalue: a function call or an overloaded operator expression, whose return type is **rvalue reference** to object
- prvalue: a function call or an overloaded operator expression, whose return type is **non-reference**


## Exercise: forwarding reference in class method

Starting from C++17, we have Class Template Argument Deduction (**CTAD**), which allows compiler to figure out the type template argument `T` for us from constructor.

```cpp
template<class T>
class MyClass {
public:
    MyClass(T value) {
        
    }
    
    void func(T&& arg) {
        
    }
};

int main() {
    MyClass MyInstance(10);
    auto MyInstance2 = MyClass(10);
    
    std::vector myvec{10};    // no need to specify std::vector<int>
}
```

In this case, is `arg` in `MyClass<T>::func(arg)` a rvalue reference or a forwarding reference?

**Ans**: rvalue reference

**Explain**: `func()` can only be called when `MyClass` is instantiated. At this point, `T` is fixed and won't perform any type deduction anymore when calling `func()`.

However, **forwarding reference can appear inside the constructor**, as the type argument `T` is deduced when instantiating the class.


## References
- [Why can I still access an object that has been moved?](https://stackoverflow.com/questions/25129247/why-can-i-still-access-an-object-that-has-been-moved)
- [Is the introduction of rvalue references actually useful?](https://stackoverflow.com/questions/18203201/is-the-introduction-of-rvalue-references-actually-useful)
- [C++的左值(lvalue)和右值(rvalue)](https://zhuanlan.zhihu.com/p/572298912)
- [C++11 嘴砲： Universal Reference, Rvalue Reference, Move Semantics](http://tommyjswu-blog.logdown.com/posts/1766009-c-11-mouth-cannon-universal-reference-rvalue-reference-and-move-semantics)
- [一文读懂C++右值引用和std::move](https://zhuanlan.zhihu.com/p/335994370)
- [Does it make sense for a function to return an rvalue reference?](https://stackoverflow.com/questions/55958970/does-it-make-sense-for-a-function-to-return-an-rvalue-reference)
- [stackoverflow: xvalue v.s. prvalue](https://stackoverflow.com/questions/45317763/xvalues-vs-prvalues-what-does-identity-property-add)