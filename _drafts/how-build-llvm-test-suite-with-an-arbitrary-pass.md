---
layout: post
title: How build LLVM Test Suite with an arbitrary pass?
categories:
- Compilers
tags:
- compilers
- open-source
---
Recently, I embarked on my journey in compiler design by joining UFMG's Compilers Lab as a graduate researcher.

My first challenge was to become familiar with LLVM and learn how to develop a pass for it. However, that's not all, as we are also working with code compression techniques and require an infrastructure to test our developed passes. Fortunately, the LLVM Test Suite is available to us.

We are using LLVM 17 and its suite of test suites. However, we discovered that compiling the entire project with a newly created pass is not a trivial task, since there are no intermediate steps in the CMake project for us to specify our new pass in the compilation pipeline. Therefore, we began researching new ways to compile the entire project with our new pass.

