---
title: GDB CheatSheet
cover: /img/cpp/cpp.png
mathjax: true
categories: C++
tags: 
    - GDB
    - Debugging
    - Compiler
abbrlink: 17350
date: 2022-05-14 00:00:00
updated: 2023-02-23 00:00:00
---

This is a gdb cheat sheet I used when I was a student. gdb is useful debug tool, especially for low level system development. In my case, I used it frequently when I was working with kernel projects.

## Before

To use gdb, we need to compile the source files with `-g` flag
```shell=
$ g++ -o main -g main.cpp
```

To start gdb
```shell=
$ gdb ./main
```

## Essential
There are a few useful features that gdb provides:
### break
Set up a breakpoint.
For convenience, we will usually use `b` for short.

For example, to set up a breakpoint at line 25 of main.cpp
```shell=
(gdb) b main.cpp:25
```

### Run
After setting up out breakpoints, we need to make the program `run`. To do this, 
```shell=
(gdb) run
```

### Continue
gdb will stop the program at the breakpoint, to cotinue running the program, type `continue` or simply `c`.
```shell=
(gdb) c
```

### step
Step through one line of code.
Use `s` for short.
```shell=
(gdb) s
```

### next
Similar to `step`, but will not go into another function.

That is, it will **see a function call as one line**, 
whereas `step` will jump to the code of called function.
```cpp=
int boo() {
    int x = 10;
    foo(x);         // (gdb) n
    int y = 10;     // get here
    return;
}
```

Use `n` for short.

## Useful
Belows are some useful features you need to know.
### tui
gdb supports UI interface when debugging.
In this mode, gdb will show you the code blocks you are debugging.

To use this, run gdb in the terminal with `-tui` flag
```shell=
$ gdb ./main -tui
```

### backtrace
Show you the function call stacks.
`bt` for short.
```shell=
(gdb) bt
```

## xv6
The followings will demonstrate how to use gdb on xv6.

Before debugging, we need to [download and install](https://pdos.csail.mit.edu/6.828/2022/tools.html)
- riscv toolchain
- qemu

After setting up tools, we can start debugging.

First, open **two** terminals.
In the first terminal, we will boot xv6 by the command
```shell=
$ make qemu-gdb
```

In the second terminal, we will use `riscv64-unknown-elf-gdb` (which is gdb from riscv toolchain) for debugging.
```shell=
$ riscv64-unknown-elf-gdb
```

To "connect" the two terminals, we will point gdb we started in the second terminal to xv6 booted in the first terminal.

Inside the first terminal, we can see something like
```shell=
$ make qemu-gdb
sed "s/:1234/:25501/" < .gdbinit.tmpl-riscv > .gdbinit
*** Now run 'gdb' in another window.
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -global virtio-mmio.force-legacy=false -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 -S -gdb tcp::25501
```

In the last few words, we can see `tcp::25501`, and we will target gdb to that port!
So inside gdb, type
```shell=    
(gdb) target remote:25501
```

After that, we can start debugging.

#### user program
In the default setup, we can debug kernel functions. But to debug user programs, we need a little more effort.

Inside gdb, we need to load the user file we want to debug into gdb.
For example, we want to debug `ls.c`, 
```shell=
(gdb) file user/_ls
```
(`_ls` is the compiled executable of `ls.c`)

After that we can set breakpoints inside `ls.c` and start debugging.

#### ref
- [MIT6.828准备 — risc-v和xv6环境搭建](https://zhayujie.com/mit6828-env.html)
