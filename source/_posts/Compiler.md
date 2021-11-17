---
title: RCC: Retarded C-like Compiler
date: 2021-11-17 16:29:37
tags: C++
---

Course project for Compiler Principle at ZJU. Co-Authored with @[Tinghao(Vitus) Xie](http://vtu.life/posts/2020/06/RCC/).[github](https://github.com/Luke-Skycrawler/rcc/)


### Features
- [x] Advanced self-defined types (nested struct and arrays)
- [x] MACRO support (#define/#define f(X),#ifdef/ifndef and nested)
- [ ] Error detection and recovery (primary)

### Language definitions

This language is a miniature of C. We highlight our differences as follows:
- type system: char, int, double and n-dimensional array type; Pointer type is not supported in this version.
- no controled jumps, gotos and labels , i.e. break, continue and switch statements are not supported.
- `scanf` and `printf` are automaticly embedded in runtime. Do not override it.
- calling convention of scanf is modified. e.g. you shall use `scanf("%d",i);` to read the value into variable i and drop the & symbol.
- `for` loop switched to pascal-like, i.e. `for(i: 0 to n){}` and `for(i: n downto 0){}` snippet. `i` is only seen within the scope of this loop.
- unary operators are not supported in this version.
You are encouraged to refer to the test examples to get a better understanding of the gramma.

