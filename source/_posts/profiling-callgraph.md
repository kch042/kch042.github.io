---
title: Profiling - Call Graph
cover: /img/run.png
mathjax: true
categories: HPC
tags:
    - profiling
    - perf
    - call graph
abbrlink: 434294
date: 2025-06-30 00:00:00
updated: 2025-06-30 00:00:00
---

Performance is a key reason we use C++ for development. Although it should be fast enough for most circumstances, there's always a need for faster speed in fields like quant trading or machine learning. 

To evaluate the performance of a program, the most intuitive way is to measure its running time. We could use tools like linux `time` to do that. However, this way is more like giving us an overall score of the performance, lacking the individual components of the score. We are interested in the breakdown of the program, and most importantly, its bottleneck. 

And here come's the linux perf tool.

# Perf Tool

`perf` is a built-in profiling tool on Linux. It provides us with useful information such as 
* L1 data cache misses
* L1 instruction cache misses
* number of branches
* page fault rate
* context switch rate
* IPC (instruction per cycle)

Besides the statistics it shown above, it also provides fine-grained timing by sampling and capturing events during the program run. This allows us the track the function calls with their call time and figure out the program hot spot.

# Stack Trace

Stack trace is the call history of functions. For example, `main()` calls `func1()` which then calls `func2()`, then `main -> func1 -> fun2` is a stack trace.

There are 3 main methods for capturing the stack trace.
* fp (frame pointer)
* dwarf
* lbr (last branch record)

## fp

In modern CPU, it has a register called frame pointer that records the start address of current function frame. For every function frame, it will store the pointer of the previous frame. In this way, we could get the stack trace using a reasonable effort.

However, compiler may often do some optimization trick that could get us incorrect stack traces. For example, for functions with [tail calls](https://stackoverflow.com/questions/22037261/what-does-sibling-calls-mean), compiler might make use of the same frames for them. Another example is inlining functions. In this way, we are unable to capture stack traces correctly. 

**We could disable these optimization by adding the compiler flags `-fno-omit-frame-pointer`, `-fno-omit-frame-pointer` and `-fno-inline`**

## dwarf

A method to collect most detailed stack trace. **It is the most accurate.**

However, it is has the most heavy overhead and not practical for large program profiling.

## lbr

This requires hardware support. In some processor, it has a register to store the from/to addresses for branch instructions. We could collect stack trace by capturing values from this register.

The advantage is that the overhead is low, but it is not suitable for tracing deep stack since it's relying on the LBR register and the register has a limited space.

# Symbols

Every variable, function, class, has a unique name. We call them symbols. 

In the stack trace collected, we only have addresses. To translate them back to human-readable names, we need the symbol table which maps the symbol name to its address. 

We could check with the `nm` command to see if the symbol table exists in the binary
```bash
# without symbol table
$ root@50e0b03b2104:~# nm main
nm: callgraph: no symbols

# with symbol table
$ root@50e0b03b2104:~# nm main
nm: callgraph: no symbols
0000000000000908 T A
0000000000000864 T B
00000000000007c0 T C
0000000000000724 T D
0000000000010dc8 a _DYNAMIC
0000000000010fa8 a _GLOBAL_OFFSET_TABLE_
...
```

**In short, it is recommended to compile the program under test with `-g` flag and do NOT strip it.** (With `-g`, we will have more debugging information and stripping removes the symbol table from the binary.)

# Call Graph

This section describes how to analyze the stack trace and interpret the call graphs. I am taking the terminology and ideas from this amazing [blog](https://zh-blog.logan.tw/2019/10/06/intro-to-perf-events-and-call-graph/), just adding some personal interpretation based on these. 

First, we can think of a stack trace, or part of it, as a **path**.

Suppose we want to know the function `A()`, we have two types of call graph and they focuses on different points.
- Caller-Based: What's the time of each path **starting at** `A()`
- Callee-Based: What's the time of each path **ending at** `A()`

![](/img/pc/perf1.png)

## Example

Let's reuse the program in the [blog](https://zh-blog.logan.tw/2019/10/06/intro-to-perf-events-and-call-graph/) with some slight modification.

```cpp
#include <stdio.h>
#include <stdlib.h>

#define SIZE ((4 * 1024 * 1024))
#define NUM_ROUNDS 256

static char data[SIZE];


#define COMPUTE(N, CH) \
  do { \
    unsigned i, j; \
    for (j = 0; j < (N); ++j) { \
      for (i = 0; i < SIZE; ++i) { \
        data[i] = CH + i + j; \
      } \
      escape(data); \
    } \
  } while (0)



__attribute__((always_inline))
static inline void escape(void *p) {
  __asm__ volatile ("" : : "r"(p) : "memory");
}


__attribute__((noinline))
void D() {
  COMPUTE(1, 'D');
}


__attribute__((noinline))
void C() {
  D();
  COMPUTE(NUM_ROUNDS, 'C');
}


__attribute__((noinline))
void B() {
  COMPUTE(NUM_ROUNDS, 'B');
  C();
  C();
}


__attribute__((noinline))
void A() {
  COMPUTE(NUM_ROUNDS, 'A');
  B();
  C();
  B();
}

__attribute__((noinline))
void W() {
  COMPUTE(NUM_ROUNDS, 'W');
  A();
}


int main() {
  COMPUTE(NUM_ROUNDS, 'M');
  A();
  A();
  W();
  return 0;
}
```

The time line looks like this
![](/img/pc/perf2.png)

First let's obtain caller-based graph.
```bash
# collect stack traces and output to perf.data file
# using dwarf here for the most accurate results
$ perf record -F 199 -g --call-graph dwarf ./callgraph

# read perf.data and output call graphs to stdout
# default output caller-based graphs
$ perf report --stdio
```

The output contains call graphs starting from every function. For the first line of each graph, it 
displays the header information. For example, let's look at the graph of `main`. 

As we can see, we have 4.07% of time inside `main()` frame and 95.93% of time in its desendant functions of `main()`. We have 62.23% + 31.02% = 93.25% of time in `A()` or its descendant functions.
```bash
# Children      Self  Command    Shared Object  Symbol               
# ........  ........  .........  .............  .....................
#
   100.00%     4.07%  callgraph  callgraph      [.] main
            |          
            |--95.93%--main
            |          |          
            |          |--62.23%--A
            |          |          |          
            |          |          |--48.72%--B
            |          |          |          |          
            |          |          |           --32.56%--C
            |          |          |          
            |          |           --8.11%--C
            |          |          
            |           --33.70%--W
            |                     |          
            |                      --31.02%--A
            |                                |          
            |                                |--24.30%--B
            |                                |          |          
            |                                |           --16.22%--C
            |                                |          
            |                                 --4.04%--C
            |          
             --4.07%--_start
                       __libc_start_main
                       main
```

To see how many time is spent in `A()` but not its descendant functions. Let's look at `A()`'s call graph, and we can see 8.08% of time are spent in `A()` (i.e. `main -> A` and `main -> W -> A`)  but not its descendant functions.
```bash
# Children      Self  Command    Shared Object  Symbol               
# ........  ........  .........  .............  .....................
#
    93.25%     8.08%  callgraph  callgraph      [.] A
            |          
            |--85.17%--A
            |          |          
            |          |--73.02%--B
            |          |          |          
            |          |           --48.78%--C
            |          |          
            |           --12.15%--C
            |          
             --8.08%--_start
                       __libc_start_main
                       main
                       |          
                       |--5.40%--A
                       |          
                        --2.68%--W
                                  A
```

![](/img/pc/perf3.png)

If we were to see the how much time for `main -> A` and `main -> W -> A` respectively, we can get A's callee-based graph.
```bash
$ perf report --no-children --stdio
...
# Overhead  Command    Shared Object  Symbol              
# ........  .........  .............  ....................
#
...
     8.08%  callgraph  callgraph      [.] A
            |
            ---A
               |          
               |--5.40%--main
               |          __libc_start_main
               |          _start
               |          
                --2.68%--W
                          main
                          __libc_start_main
                          _start
``` 

We can see that 5.4% of time is spent in `main -> A` and 2.68% of time for `main -> W -> A`, which sums up to exactly 8.08%.

# Case Study: Combination Sum

In practice, we usually profile with large programs. But for demo purpose, I will show a simple one, even though it took me much time to debug when I first time seen it.

Suppose we are given an integer array `nums` and a number `target`, and we would like to know if we are able to pick some numbers from `nums` that sums up to `target`.

It's a very simple problem, and should be familiar with everyone who has learnt algorithms. And here's the program I wrote when I first faced it.

```cpp
#include <numeric>
#include <vector>

bool combinationSum(std::vector<int> nums, int target, int i) {
  if (target == 0) {
    return true;
  }
  if (i == nums.size() || target < 0) {
    return false;
  }

  return combinationSum(nums, target, i + 1) ||
         combinationSum(nums, target - nums[i], i + 1);
}

int main() {
  // test data
  std::vector<int> nums(25);
  std::iota(nums.begin(), nums.end(), 1);
  int target = 10000;

  combinationSum(nums, target, 0);
}
```

Let's first get the overall performance
```bash
$ perf stat ./main

 Performance counter stats for './main':

       2296.484151      task-clock:u (msec)       #    0.999 CPUs utilized          
                 0      context-switches:u        #    0.000 K/sec                  
                 0      cpu-migrations:u          #    0.000 K/sec                  
               117      page-faults:u             #    0.051 K/sec                  
        7756480407      cycles:u                  #    3.378 GHz                    
       20355933838      instructions:u            #    2.62  insn per cycle         
        4765061486      branches:u                # 2074.938 M/sec                  
          18856150      branch-misses:u           #    0.40% of all branches        

       2.298209049 seconds time elapsed
```

2.3 seconds seems not bad. But unfortunately it doesn't pass the time limit. Let's see the stack traces and knows what's taking so long under the hood.


First collect stack traces with `perf record`.
```bash
# sampling at 199 Hz
# outputs a perf.data file
$ perf record -F 199 --call-graph lbr ./main
```

Then display caller-based graph report to stdout.
```bash
# read perf.data and display caller-based grapg report to stdout
$ perf report --stdio
```

Let's first see the call graph of `combinationSum`. The call stack is deep and I have omitted some unrelevant calls for clarity.
```bash
# Children      Self  Command  Shared Object        Symbol                           
# ........  ........  .......  ...................  .................................
#
    99.95%    26.07%  main     main                 [.] combinationSum
            |          
            |--73.88%--combinationSum
            ...
            |          combinationSum
            |          combinationSum
            |          |          
            |          |--72.86%--combinationSum
            |          |          |          
            |          |          |--71.01%--combinationSum
            |          |          |          |          
            |          |          |          |--67.32%--combinationSum
            |          |          |          |          |          
            |          |          |          |          |--58.50%--combinationSum
            |          |          |          |          |          |          
            |          |          |          |          |          |--40.46%--combinationSum
            |          |          |          |          |          |          |          
            |          |          |          |          |          |          |--15.59%--operator delete@plt
            |          |          |          |          |          |          |          
            |          |          |          |          |          |          |--15.11%--operator new
            |          |          |          |          |          |          |          |          
            |          |          |          |          |          |          |          |--7.61%--malloc@plt
            |          |          |          |          |          |          |          |          
            |          |          |          |          |          |          |           --7.50%--malloc
            |          |          |          |          |          |          |                     _int_malloc
            |          |          |          |          |          |          |          
            |          |          |          |          |          |          |--7.92%--memcpy@plt
            |          |          |          |          |          |          |          
            |          |          |          |          |          |           --1.85%--operator new@plt
            |          |          |          |          |          |          
            |          |          |          |          |          |--7.79%--operator new
            |          |          |          |          |          |          |          
            |          |          |          |          |          |          |--5.33%--malloc@plt
            |          |          |          |          |          |          |          
            |          |          |          |          |          |           --2.46%--malloc
            |          |          |          |          |          |                     _int_malloc
            |          |          |          |          |          |          
            |          |          |          |          |          |--7.38%--operator delete@plt
            |          |          |          |          |          |          
            |          |          |          |          |          |--2.25%--memcpy@plt
            |          |          |          |          |          |          
            |          |          |          |          |           --0.61%--operator new@plt
            |          |          |          |          |          
            |          |          |          |          |--4.31%--operator delete@plt
            |          |          |          |          |          
            |          |          |          |          |--3.28%--operator new
            |          |          |          |          |          |          
            |          |          |          |          |          |--2.05%--malloc@plt
            |          |          |          |          |          |          
            |          |          |          |          |           --1.23%--malloc
            |          |          |          |          |                     _int_malloc
            |          |          |          |          |          
            |          |          |          |           --1.23%--memcpy@plt
            ...
```

As we can see, at each level of `combinationSum`, it is spending time to do the heap allocation, looking kinda suspicious.

To see the overall effect of heap allocation, we could turn to the callee-based graph.
```bash
$ perf report --no-children --stdio
```

And let's look at the `malloc` part. Overall, we are spending 15.81% of time at `malloc`. 
```bash
# Overhead  Command  Shared Object        Symbol                 
# ........  .......  ...................  .......................
#
    15.81%  main     libc-2.17.so         [.] malloc
            |
            ---malloc@plt
               operator new
               combinationSum
               ...
               combinationSum
               |          
                --15.40%--combinationSum
                          |          
                           --14.99%--combinationSum
                                     |          
                                     |--14.38%--combinationSum
                                     |          |          
                                     |          |--12.53%--combinationSum
                                     |          |          |          
                                     |          |          |--7.41%--combinationSum
                                     |          |          |          main
                                     |          |          |          __libc_start_main
                                     |          |          |          
                                     |          |           --5.13%--main
                                     |          |                     __libc_start_main
                                     |          |          
                                     |           --1.84%--main
                                     |                     __libc_start_main
                                     |          
                                      --0.61%--main
                                                __libc_start_main

    ...
```

For all memory operations, they take 64.67% of time 
```bash
# Overhead  Command  Shared Object        Symbol                 
# ........  .......  ...................  .......................
#
    23.38%  main     libc-2.17.so         [.] _int_free
    ...
    15.81%  main     libc-2.17.so         [.] malloc
    ...
    12.63%  main     libc-2.17.so         [.] __memcpy_ssse3_back
    ...
    12.42%  main     libc-2.17.so         [.] _int_malloc
    ...
```

At this point, or maybe from earlier moments, we should be clear what's the hotspot of this program. Each time `combinationSum` is called, `nums` is copied. It could be easily optimized by reusing the same vector `nums` for all `combinationSum` calls.

After optimization, it only takes 0.23s, 10 times faster, awesome!
```bash
 Performance counter stats for 'main':

        235.681078      task-clock:u (msec)       #    0.993 CPUs utilized          
                 0      context-switches:u        #    0.000 K/sec                  
                 0      cpu-migrations:u          #    0.000 K/sec                  
               103      page-faults:u             #    0.437 K/sec                  
         794405800      cycles:u                  #    3.371 GHz                    
        2518372430      instructions:u            #    3.17  insn per cycle         
         470069848      branches:u                #    1994.517 M/sec                  
            243573      branch-misses:u           #    0.05% of all branches        

       0.237236131 seconds time elapsed
```

And if we look at the call graphs, the memory operations all vanishes, because they are little to count. Note that for the callee-based graph, most of the samples will fall on the longest path (i.e., the path with the largest number of `combinationSum` calls).


**Spotting such mistake is easy when the size of the program is small. As the program scales and becomes complicated, the diffculty to spot such mistake increases drastically.** At this point, the profiling tool will be our lovely friends to help us out.

# Excercise

In the above case, we were capturing stack trace with lbr. Now I want to collect stack trace by frame pointers. First I am compiling with
```bash
$ g++ -g -fno-omit-frame-pointer -fno-omit-frame-pointer -fno-inline -o main main.cc
```

And collect stack traces with
```bash
$ perf record -F 199 -g --call-graph fp ./main
```

Here's the report
```bash
$ perf report --stdio
# Children      Self  Command  Shared Object        Symbol                           
# ........  ........  .......  ...................  .................................
#
    56.00%     0.00%  main     main                 [.] main
            |
            ---main
               combinationSum
               combinationSum
               combinationSum
               combinationSum
               combinationSum
    ...
```

In theory, most sample points should fall under `main` or the functions called by `main`, which sums up to near 100%. But it only gets 56%. Why?

{% hideToggle Answer %}

The system libraries were compiled without flags such as `-fno-omit-frame-pointer`. The stack traces couldn't be captured correctly.

{% endhideToggle %}

# Reference
1. https://www.brendangregg.com/perf.html
2. https://zh-blog.logan.tw/2019/10/06/intro-to-perf-events-and-call-graph/
3. ChatGPT