---
layout: post
title: Daedalus Debug Toolkit
date: 2025-08-29 21:12 -0300
---
For the past year, I've been working on project [Daedalus](https://github.com/lac-dcc/Daedalus) at the [Compilers Lab - DCC/UFMG](https://lac-dcc.github.io/), while I'm pursuing my master's degree. We implemented an out-of-source LLVM pass that compresses code by leveraging *Program Slices* in *Gated Single Assignment* (GSA) form.

We primarily developed and tested our implementation using a patched version of the [LLVM Test Suite](https://github.com/Casperento/llvm-test-suite/tree/daedalus). This [patch](https://casperento.github.io/posts/how-to-build-llvm-test-suite-with-an-arbitrary-pass/) forces the compilation of fully linked bitcode files, which increases the likelihood of finding recurring *program slices* across different functions within a module. At one point, we successfully compiled 2028 programs from this corpus without any build errors.

This was a challenging task. Many programs, especially after being optimized with `clang -Os`, had intricate control flow. As you can imagine, our pass broke quite a few times before it could correctly handle the entire suite. To streamline the process of handling these large bitcode files and isolate faulty functions, I developed a collection of Bash and Python scripts to aid in debugging our pass.

While these scripts are tailored to the Daedalus project, they essentially act as wrappers around standard LLVM tools. Therefore, I'm sharing this collection in the hope that it might save others some time when debugging their own LLVM passes.

All scripts, along with explanations of the files they generate, are documented in the repository's README.

Hope you find it useful!

**Link:** [daedalus-dbg-toolkit](https://github.com/Casperento/daedalus-dbg-toolkit)
