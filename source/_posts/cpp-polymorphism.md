---
title: C++ Polymorphism
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: C++
abbrlink: 40119
date: 2023-10-24 00:00:00
updated: 2024-02-20 00:00:00
---

## Overview

- Compile time polymorphism
    - Function overloading
    - Operator overloading
- Runtime polymorphism
    - Virtual function (dynamic dispatching)

## Compile time polymorphism

### Function overloading
Functions of **different signature** and return type can share the same function name
```cpp=
void fn(int a) {
    cout << a << "\n";
}

void fn(float b) {
    cout << b << "\n";
}

int main() {
    fn(10);
    fn(1.0);
}
```

**However, we cannot overload functions that only differ in the return type.**
```cpp=
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



### Virtual Function

Suppose we have a base class `Base` and two derived class `Derived1` and `Derived2`. They all have their own implementation of `sayHi()`.

Now we want to write a function `sayHi()` that will invoke each object's implementation of`sayHi()`. 

But this could cause a problem: In order to let objects of `Base`, `Derived1`, `Derived2` use the same functions, we declare the argument type `Base`. This will cause the function to bind the `sayHi()` to `Base`'s. 

```cpp=
class Base {
public:
    void sayHi() {
         cout << "Hi I am B\n";
    }
};

class Derived1: public Base {
public:
    void sayHi() {
        cout << "Hi I am D1\n";
    }
};

class Derived2: public Base {
public:
    void sayHi() {
        cout << "Hi I am D2\n";
    }
};

// Driver code
void sayHi(Base &b) {
    b.sayHi();
}

int main() {
    Base b;
    Derived1 d1;
    Derived2 d2;
    sayHi(b);     // Hi I am B
    sayHi(d1);    // Hi I am B
    sayHi(d2);    // Hi I am B
    
    Base &b2 = d1;
    b2.sayHi();   // Hi I am B
    
    Base *b3 = new Derived2();
    b3->sayHi();  // Hi I am B
}

```

**We want methods from their actual class, not the class that referencing it!!**

Here we introduce virtual function to solve the problem.
```cpp=
class Base {
public:
    // only add virtual keyword here
    virtual void sayHi() {
         cout << "Hi I am B\n";
    }
};

class Derived1: public Base {
public:
    void sayHi() {
        cout << "Hi I am D1\n";
    }
};

class Derived2: public Base {
public:
    void sayHi() {
        cout << "Hi I am D2\n";
    }
};

// Driver code
void sayHi(Base &b) {
    b.sayHi();
}

int main() {
    Base b;
    Derived1 d1;
    Derived2 d2;
    sayHi(b);     // Hi I am B
    sayHi(d1);    // Hi I am D1
    sayHi(d2);    // Hi I am D2
    
    Base &b2 = d1;
    b2.sayHi();   // Hi I am D1
    
    Base *b3 = new Derived2();
    b3->sayHi();  // Hi I am D2
}
```

Each object keeps a pointer to a table (called **vtable**) as a data member. This vtable stores a list of function pointers, each of which is the virtual function of that object.

![](/img/cpp/vtable.png)

So in our second example, when calling `sayHi(d1)`, it will look up `d1`'s vtable and call its `sayHi()`. (Even though `d1` is assumed to be of type `Base` in `sayHi()`)

#### Function Call
A typical function call inside a piece code works as follows:
1. Save the context (caller-save registers)
2. PC Jump to the **function address**

How do we get the address of the function to run in step 2?
This behavior is called **binding**, and we have two ways to do binding.

1. **Static Binding (Early Binding)**
The compiler will know which function to call and put the address in the assembly for us. (i.e. bind the function call to the function definition from the type of pointer referring to that object at compile time) 
In the first example, compiler decides to use `sayHi()` from `Base` in `sayHi(Base &b)` (since `sayHi()` is a non-virtual function in the first example)
2. **Dynamic Binding**
The binding address will be determined during runtime. This case happens when the compiler **cannot** duduce which function to use.
In the second example, `sayHi()` is a virtual function and compiler has no idea which to use, so it leaves the runtime to find out (looking up vtable, call the appropriate function).

## Exercise: memset for initializing a class

Is it a good idea to initialize a class with `memset()`?
```cpp=
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

## Exercise: can a constructor be virtual?

The answer is NO.

Objects of the same class usually share the same virtual table. When calling the constructor, it will need to initialize the vtable pointer and let it point to the class vtable.

So it's clear why a constructor cannot be virtual: it cannot lookup the vtable before setting up the address of vtable.


## Exercise: can be a destructor be virtual?
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

## Ref
1. [Why do we need a virtual table?](https://stackoverflow.com/questions/3004501/why-do-we-need-a-virtual-table)
2. [memset for initialization in C++](https://stackoverflow.com/questions/2481654/memset-for-initialization-in-c)
3. [C++中關於 virtual 的兩三事](https://medium.com/theskyisblue/c-中關於-virtual-的兩三事-1b4e2a2dc373)
4. [Dynamic dispatch wiki](https://en.wikipedia.org/wiki/Dynamic_dispatch)