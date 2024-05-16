---
layout: post
title: How to build the LLVM Test Suite with an arbitrary pass?
categories:
- Compilers
tags:
- compilers
- open-source
date: 2024-05-16 00:00 +0000
---
Recently, I embarked on my journey in compiler design by joining [UFMG's Compilers Lab](https://lac-dcc.github.io/) as a graduate researcher.

My first challenge was to become familiar with LLVM and learn how to develop a pass for it. However, that's not all, as we are also working with code compression techniques and require an infrastructure to test our developed passes. Fortunately, the [LLVM Test Suite](https://github.com/llvm/llvm-test-suite) is available to us.

We are using LLVM 17 and its test suite. However, we discovered that compiling the entire project with a newly created pass is not a trivial task, since there are no intermediate steps in the CMake project for us to specify our new pass in the compilation pipeline. Therefore, we began researching new ways to compile the entire project with our new pass.

## Introduction

Perhaps one of the nicest parts of the LLVM project is the Test Suite. This is a large collection of C and C++ benchmarks that compiler engineers can use to test the techniques that they implement. The suite includes a diverse array of programs and libraries, ranging from small, single-file programs to large, complex applications, ensuring comprehensive coverage of real-world scenarios.

By leveraging this extensive test suite, developers can assess the performance, correctness, and robustness of their compiler optimizations and transformations. Furthermore, the suite is continually updated and maintained by the LLVM community, incorporating new benchmarks and reflecting the latest industry practices. This makes it an invaluable resource for anyone involved in compiler development or research, providing a solid foundation for testing and validating new ideas in compiler technology.

## Problem Statement

Our task is to compile the entire Test Suite with an arbitrary pass, so that we can compare different metrics between a baseline and a chosen experiment. In other words, we want to enable the following pipeline:

1. Compile every program in the Test Suite, producing a set of LLVM IR
files. Let this collection of bytecodes be called "Control";
2. Collect metrics from "Control";
3. Compile every program in the Test Suite, this time with a set of
passes of interest, producing a set of LLVM IR files. Let this new
collection of bytecodes be called "Test";
4. Collect metrics from "Test";
5. Produce a report comparing metrics from "Control" and "Test".

As an example, assume that we want to test the [func-merging](https://github.com/rcorcs/llvm-project/commit/246386f55764c35901099ca03b6fda4e20d36354)
pass. This pass takes functions that are structurally equal and merge
them to reduce code size. Imagine that we want to measure the
code-size reduction enabled by this optimization. LLVM provides a pass
to count instructions: **instcount**. Thus, we can combine these two
passes to get the numbers that we want, e.g.:

1. Produce the "Control" set of bytecode files;
2. Use the **instcount** pass to count the number of instructions in the
"Control" dataset;
3. Produce the "Test" dataset applying **func-merging** onto the Control dataset;
4. Use the **instcount** pass to count the number of instructions in the
"Test" dataset;
5. Produce a report comparing the number of instructions in the
"Control" and "Test" datasets.

## Solution

To solve our problem, we're going to: 

1. Apply a patch in the Test Suite's CMake project;
2. Run some shell commands to get the project compiled;
3. Run the test suite with LIT to get the results;
4. Finally, get the results compared.

### Cloning LLVM Test Suite

To clone the project's repository, run the following command inside a folder of your choice:

```bash
$ git clone -b release/17.x https://github.com/llvm/llvm-test-suite.git
```

### Building Test Suite Binaries

To build the project, we need to apply our patch into the CMake files. Then, we'll configure and build the entire Test Suite with our patch.

#### Applying the Patch

The following commit synthesizes all changes we've made in our patch, for you to apply it locally:

[Patch's commit](https://github.com/Casperento/llvm-test-suite/commit/ac2d88f53feff0057b60e15b65890ee12be496ae)

#### Configuring and Building CMake project

In our case, we want to count how many instructions programs have before applying our pass. Thus, we're going to build the project two times, one to get the baseline tests' results, and the second one to get our experiment tests' result.

##### First running

Assuming your current directory is *"path/to/llvm-test-suite"*, run:

```bash
$ mkdir -p build
$ cd build
$ cmake -G "Ninja" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ \
-DCMAKE_C_FLAGS="-flto" -DCMAKE_CXX_FLAGS="-flto" \
-DTEST_SUITE_COLLECT_INSTCOUNT=ON -DTEST_SUITE_SELECTED_PASSES= \
-DTEST_SUITE_RUN_BENCHMARKS=OFF -DTEST_SUITE_COLLECT_COMPILE_TIME=OFF \
-DCMAKE_EXE_LINKER_FLAGS="-flto -fuse-ld=lld -Wl,--plugin-opt=-lto-embed-bitcode=post-merge-pre-opt" \
-C ../cmake/caches/O1.cmake ..
```

**Note**: we assume you already have **ninja-build** installed. If you don't, just ignore the ```-G "Ninja"``` argument in the CMake command.

Now, build the project with:

```bash
$ cmake --build .
```

##### Running LIT

To run the tests with LIT, create a separate folder for the results and run it:

```bash
$ mkdir -p ~/lit-results
$ llvm-lit -v -o ~/lit-results/results_instcount.json .
```

##### Second running

The chosen pass' name must be specified by the **TEST_SUITE_SELECTED_PASSES** CMake env. variable. If you need to run a sequence of passes, just write them in the desired order separated by a comma, like:

> -DTEST_SUITE_SELECTED_PASSES=pass-1,pass-2

Then, to compile the test suite again with the chosen pass, reconfigure the CMake project with the following command:

```bash
$ cmake -G "Ninja" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ \
-DCMAKE_C_FLAGS="-flto" -DCMAKE_CXX_FLAGS="-flto" \
-DTEST_SUITE_COLLECT_INSTCOUNT=ON -DTEST_SUITE_SELECTED_PASSES=func-merging \
-DTEST_SUITE_RUN_BENCHMARKS=OFF -DTEST_SUITE_COLLECT_COMPILE_TIME=OFF \
-DCMAKE_EXE_LINKER_FLAGS="-flto -fuse-ld=lld -Wl,--plugin-opt=-lto-embed-bitcode=post-merge-pre-opt" \
-C ../cmake/caches/O1.cmake ..
```

Then, build the project again:

```bash
$ cmake --build .
```

##### Running LIT

Run the tests with LIT again:

```bash
$ llvm-lit -v -o ~/lit-results/results_func-merging.json .
```

##### Comparing Results

To compare the results by **instcount** and **size..text** metrics, run the following python script provided by the test suite:

Assuming your current directory is *"path/to/llvm-test-suite/build"*.

```bash
$ python3 ../utils/compare.py --full --diff -m instcount -m size..text ~/lit-results/results_instcount.json ~/lit-results/results_func-merging.json > ~/lit-results/test-results.txt
```

[Comparison Result](https://gist.github.com/Casperento/debc184fb978231015e3e39765307040)

## Conclusion

Our objective is to compile the Test Suite using different LLVM passes for metric comparison. Initially, we compile the suite with the **instcount** pass to determine the instruction count of generated bytecode files, establishing a baseline metric. Subsequently, we compile the same suite with the **func-merging** pass applied to the binaries and then reevaluate the instruction count.

By analyzing the differences in instruction counts between the baseline (using **instcount**) and the experimental setup (with **func-merging**), we can draw conclusions on the effectiveness of the **func-merging** pass for code compression within the LLVM framework.

## References

[1] https://discourse.llvm.org/t/how-to-use-llvm-test-suite-to-experiment-with-alias-analysis/69668/10

[2] https://releases.llvm.org/17.0.1/docs/TestSuiteGuide.html

[3] https://releases.llvm.org/17.0.1/docs/CommandGuide/lit.html