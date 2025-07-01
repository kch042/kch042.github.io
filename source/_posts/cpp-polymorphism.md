---
title: C++ Polymorphism
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 40119
date: 2023-10-24 00:00:00
updated: 2025-04-27 00:00:00
---

Polymorphism is an import programming concept. In short, it boils down to **the same symbol (function name) with different implementations based on data types**.

For example, suppose we want to counts number of elements in the input and we have two kinds of input, string and vector. Instead of creating two functions `count_string(std::string)`, `count_vector(std::vector)`, we utilize the same name `count` to serve them both by creating `count(std::string)`, `count(std::vector)`. That's the essence of polymorphism.

## Overview

- Compile time polymorphism (static polymorphism)
    - Function overloading
    - Operator overloading
    - {% post_link cpp-template 'Template' %}
- Runtime polymorphism (dynamic polymorphism)
    - Virtual function (dynamic dispatching)

## Compile time polymorphism

### Function overloading
Functions of **different signature** and return type can share the same function name
```cpp
void fn(int a) {
    cout << "Integer: " << a << "\n";
}

void fn(float b) {
    cout << "Float: " << b << "\n";
}

int main() {
    fn(10);     // Integer: 10
    fn(1.0);    // Float: 1.0
}
```

Similarly, we also have function overloading in class hierarchy. The method executed will depend on the type of the variable. 
```cpp
class Base {
 public:
  void print_info() { std::cout << "Base\n"; }
};

class Derived : public Base {
 public:
  void print_info() { std::cout << "Derived\n"; }
};

int main() {
  Derived d;
  d.print_info();  // Derived

  Base& b = d;
  b.print_info();  // Base
}
```

**However, we cannot overload functions that only differ in the return type.**
```cpp
/* Can NOT compile */

float fn(int a) {
    return (float)a;
}

int fn(int a) {
    return a;
}
```

There are two reasons why we can't do that
1. Compiler cannot deduce the return type. Especially when using `auto` as type when declaration.
2. We call some functions purely for their **side effects**. Compiler can't decide which functions we should execute.

### Operator overloading

Similar to the function overloading above.

## Runtime polymorphism

Runtime polymorphism is implemented with virtual functions. Let's look at an example to see why it is needed and how it is achieved.

### Example

Suppose we have a base class `Worker` with a class method `work()`, and two derived classes `Engineer` and `Sales` with their own implementation of `work()`. 
```cpp
class Worker {
 public:
  void work() { std::cout << "Worker Working\n"; };
};

class Engineer : public Worker {
 public:
  void work() { std::cout << "Engineer Working\n"; };
};

class Sales : public Worker {
 public:
  void work() { std::cout << "Sales Working\n"; };
};
```

Now we would like to a have function `do_work()` that accepts a worker object and make it work by calling method function `work()`. **The function has no idea which worker it will receive, depending on the runtime result.** Therefore it will accept input of type `Worker` for all kinds of worker.

```cpp
void do_work(Worker* worker) {
  worker->work();
}
```

However, since the type in the input is `Worker*`, it will uses static polymorphism and the results will be the same no matter passing `Engineer` or `Sales` to it.
```cpp
Engineer e;
Sales s;

// Output: Worker Working
// Expected: Engineer Working
do_work(&e);

// Output: Worker Working
// Expected: Sales working
do_work(&s);
```

To address this issue, C++ introduced a feature called virtual functions.

### Virtual Function

To make the object use the correct function, we just need to add a `virtual` keyword to the `work` method.

```cpp
class Worker {
 public:
  virtual void work() { std::cout << "Worker Working\n"; };
};

class Engineer : public Worker {
 public:
  virtual void work() { std::cout << "Engineer Working\n"; };
};

class Sales : public Worker {
 public:
  virtual void work() { std::cout << "Sales Working\n"; };
};
```

The results will become 
```cpp
Engineer e;
Sales s;
do_work(&e);    // Engineer Working
do_work(&s);    // Sales Working
```

### Function Binding
Recall that a typical function call inside a piece code works as follows:
1. Save the context (caller-save registers)
2. PC Jump to the **function address**

How do we **get the address of the function to jump to** in step 2? Normally, compiler will have knowledge about all functions and decides where to put them in the memory. Therefore, it will put address of the function for each function call during compile time. If it is a class method, the method to use will be determined based on the type of the object.

However, compiler cannot deduce the address for those determined at runtime. To achieve this, each object will initialize a pointer to a table (called **vtable**) as a hidden data member in the constructor. This vtable is an array and stores the pointers to all of its virtual functions.

When calling a virtual function, it will
1. look up the vtable to get the function pointer
2. jump to the function and execute

The type that refers to it doesn't matter anymore.

![](/img/cpp/vtable.png)

The process of getting function address to jump to is called **binding**. In short, we have two types of binding.
1. **Static Binding (Early Binding)**
The compiler will know which function to call and **put the address in the assembly** for us. (i.e. bind the function call to the function definition from the type of pointer referring to that object at compile time) 
2. **Dynamic Binding (Late Binding)**
The binding address will be determined during runtime. This case happens when the compiler **cannot** duduce which function to use, it will leave the runtime to figure out (looking up vtable, then call the appropriate function). The downside is obvious as it will require more memory access and cause more runtime overhead.

### Downcasting/Upcasting

In the previous example, we were feeding a `Engineer*` to `do_work()`. This involves a implicit casting from `Engineer*` to `Worker*`.  This kind of pointer conversion from a derived class to base class is called **upcasting**. In fact, upcasting is the most common usage in virtual functions.

On the otherhand, `base* -> derived*` is called **downcasting** and is often thought of as a result of BAD design. Besides, direct downcasting is unsafe, we need to do some runtime check before casting. We could use `dynamic_cast` to do it.
```cpp
// Given a Worker* w;
Engineer* e = dynamic_cast(w);
if (e == nullptr) {
  // fail, w might be a Sales*
} else {
  // success
}
```

### Object Slicing

Happens when assigning a derived class object to a base class object. Some information will be lost.

https://stackoverflow.com/questions/274626/what-is-object-slicing

## Exercise: memset for initializing a class

Is it a good idea to initialize a class with `memset()`?
```cpp
class Base {
public:
    virtual void sayHi() {
        cout << "Hi I am B\n";
    }
};
int main() {
    Base b;
    memset(&b, 0, sizeof(b));
}
```

From the previous section, we know that **vtable pointer is a data member** inserted by the compiler to achieve dynamic dispatching. Using `memset` to set 0 will result in erase of vtable pointer!

```bash=
$ g++ -o test -std=c++17 test.cpp
>>> test.cpp:44:12: warning: destination for this 'memset' call is a pointer to dynamic class 'Base'; vtable pointer will be overwritten [-Wdynamic-class-memaccess]
    memset(&b, 0, sizeof(b));
    ~~~~~~ ^
```

## Exercise: Can we make constructor virtual?

The answer is NO.

**Objects of the same class usually share the same virtual table.** When calling the constructor, it will need to initialize the vtable pointer and let it point to the class vtable.

So it's clear why a constructor cannot be virtual: it cannot lookup the vtable before setting up the address of vtable.

## Exercise: Can we make destructor virtual?
The destructor **is recommended to be set virtual**, as we need the implementation of the actual class instead of the implementation from the class that refers to it.

Here's an example, without virtual, we could skip calling destructor of the derived class.
```cpp
class Base {
public:
    Base() {std::cout << "Base Constructor\n";}
    ~Base() {std::cout << "Base Destructor\n";}
};

class Derived: public Base {
public:
    Derived() {std::cout << "Derived Constructor\n";}
    ~Derived() {
        std::cout << "Derived Destructor\n";
        // contains a lot of cleanup
    }
};

int main() {
    Base *b = new Derived();
    delete b;
}
```
```bash
Base Constructor
Derived Constructor
Base Destructor
```

If we include make the destructor virtual
```cpp
class Base {
public:
    Base() {std::cout << "Base Constructor\n";}
    virtual ~Base() {std::cout << "Base Destructor\n";}
};

class Derived: public Base {
public:
    Derived() {std::cout << "Derived Constructor\n";}
    virtual ~Derived() {
        std::cout << "Derived Destructor\n";
        // contains a lot of cleanup
    }
};

int main() {
    Base *b = new Derived();
    delete b;
}
```
```bash
Base Constructor
Derived Constructor
Derived Destructor
Base Destructor
```

## Excercise: Pointer parameters

TODO: `void foo(int *arr)` vs void `foo(std::unique_ptr<int[]> arr)`

## References
1. [Why do we need a virtual table?](https://stackoverflow.com/questions/3004501/why-do-we-need-a-virtual-table)
2. [memset for initialization in C++](https://stackoverflow.com/questions/2481654/memset-for-initialization-in-c)
3. [C++中關於 virtual 的兩三事](https://medium.com/theskyisblue/c-中關於-virtual-的兩三事-1b4e2a2dc373)
4. [Dynamic dispatch wiki](https://en.wikipedia.org/wiki/Dynamic_dispatch)