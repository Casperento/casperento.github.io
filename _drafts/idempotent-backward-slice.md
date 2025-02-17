---
layout: post
title: Idempotent Backward Slice
categories:
- Compilers
tags:
- compilers
---
Have you ever wondered what programmers ask themselves when they need to fix a bug in a program? They need to understand how the program's variables, functions, and data structures interact with each other. However, a program can be very large, and one doesn't need to cover all instructions to fix a specific issue. Therefore, they can limit the scope of the search by selecting just a subset of instructions used to compute a given value at a program point. This subset of instructions is called a *slice* [X], and it is an executable program that, given an input value, will always produce an output at the end of its execution.


**Note**: this post references a video I produced as part of a series on compiler research topics, available on the [Compilers Laboratory's YouTube channel](https://www.youtube.com/@compilerslab).

## Problem Statement

TODO.

## Definitions

TODO.

## Algorithm

TODO.

## Conclusion

TODO.

## References

[Y] Tip, Frank. *A Survey of Program Slicing Techniques*. CWI (Centre for Mathematics and Computer Science), NLD. 1994.

[X] Weiser, Mark. *Program Slicing*. IEEE Transactions on Software Engineering, vol. SE-10, no. 4, July 1984, pp. 352–57.
