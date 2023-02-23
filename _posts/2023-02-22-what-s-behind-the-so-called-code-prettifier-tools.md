---
layout: post
title: What's behind the so called 'code prettifier' tools?
categories:
- Compilers
tags:
- compilers
- open-source
date: 2023-02-23 00:00 +0000
---
If you've ever wondered the same question as the title, the short answer is: a compiler.

Note: I have limited knowledge on code prettifier tools and their implementations.

Now, let's proceed with a brief introduction on compilers and how I used my knowledge to solve a daily problem. This article is intended to be a technical report about a short implementation I've made for an open-source project called [SQFvm](https://github.com/SQFvm/runtime), more on that later.

## What is a compiler ?

A compiler is a program that takes a source code as input and translate it into a target code as output (figure 1).

![Figure 1](/assets/img/posts/code_prettifier_post/fig1.png)

**Figure 1**: _a compiler._

Traditionally, the source code is written in a source language and the target code in a target language [1].

The typical approach to designing a compiler is to divide it into a frontend and a backend. The former parses (analyze) the input text to generate code in an intermediate representation to pass it to the latter. Figure 2 shows the phases of a compiler.

![Figure 2](/assets/img/posts/code_prettifier_post/fig2.png)

**Figure 2**: _phases of a compiler (frontend and backend)._


Let’s focus on the frontend.

The first step is to provide the source code to a compiler, after which the lexical analyzer starts to recognize groups of characters called tokens. These tokens are then passed to the syntax analyzer, which tries to derive the entire program as a long word by following the rules of a grammar and constructing an abstract syntax tree (AST or syntax tree, for short). The next phase is to traverse the AST in a manner that allows us to verify if any semantic rules have been broken. Finally, an intermediate representation of the original code is generated.

I won’t delve deeper into each theoretical concept involved in each phase of a compiler. For now, the most important thing to know is that if we traverse an AST and print out a string at each node, that string can represent the formatted code, instead of an intermediate representation.

Given the following SQF code, we could build an AST for it, like figure 3 shows:

```
if (100 > 2) exitWith {hint "Some string..."};
```

At this point, it doesn’t matter what the code is doing, just that it is syntactically and semantically correct.

Each node in the AST represent either a non-terminal or a terminal of our fictitious grammar. When we derive to the left or to the right child node, we apply a grammar rule until we get into a terminal (leaf node).

![Figure 3](/assets/img/posts/code_prettifier_post/fig3.png)

**Figure 3**: _AST for the SQF code above._

Note: this AST doesn’t follow the grammar rules implemented in the [SQFvm project](https://github.com/SQFvm/runtime/blob/master/src/parser/sqf/parser.y), it is just an illustrated example.

So, if we want to print the code in a formatted form, we need to traverse the AST recursively and print the content of each node or some contextual character (as seen in the red text in Figure 4).

![Figure 4](/assets/img/posts/code_prettifier_post/fig4.png)

**Figure 4**: _example of AST with printable characters._

Given the AST above, it is possible to print an output code like:

```
if (100 > 2) exitWith {
    hint "Some string..."
};
```

The output code (red text in figure 4) is printed in the following order:

1. Print `if` and derive the left node (to get _**expression**_ value);
2. Print `(` and derive the only node child;
3. Print left node value `100`, then the current node character `<`, finally print the right node value `2`;
4. Print `)`, after the recursive call that derived _**expression**_;
5. Print `exitWith` and then derive the right node (to get _**block_of_code**_ value);
6. Print `{` and a newline character;
7. Derive _**expressions**_, then print the _**unary_operator**_ `hint` after four spaces characters, finally print the child string `"Some string..."`;
8. Print `};`.

From this example, we can see that we received a one-line code and translated it to a formatted version (with spaces and newline characters). However, it’s not a universal solution to format every possible script written in SQF. By the end of the day, we still need to handle some idiosyncrasies to print the output code correctly.

## What is SQFvm ?

Originally created by X39, [SQFvm](https://github.com/SQFvm/runtime) is _“a fully working and open-source Virtual Machine for the scripting language of the ArmA Games”_, as stated in the project’s description.

They already implemented a parser for the SQF language. So, the idea here is to use their AST to translate a non-formatted code into a formatted one.

The following sections provide a detailed description of the _Problem Statement_, followed by examples of _Input_ and _Output_. Finally, the _Solution_ is explained along with the function that I’ve implemented in the _SQFvm_ project.

## Problem Statement

Given a source code written in SQF language, format it into a more ‘readable’ form. The output code must stay ‘parseable’ by SQF parser.

## Input

Let’s assume that the worst case scenario is a one-line source code.

```
private _addons=["accessorys","ai","arrays","common","diagnostic","disposable","ee","events","hashes","help","jam","jr","keybinding","main","main_a3","modules","music","network","optics","settings","statemachine","strings","ui","vectors","versioning","xeh"];private _tests=["arrays","common","diagnostic","events","hashes","network","strings","vectors"];private _functions=[];{private _addons=_x;private _addonPath=format["\x\cba\addons\%1",_x];private _addonSqfFiles=allFiles__[".sqf"];{private _addonFunctionPrefix=format["%1/fnc_",_addons];if(_x find _addonFunctionPrefix>0)then{private _splitStr=_x splitString "/";private _filename=_splitStr select(count _splitStr-1);private _filePath=format["%1\%2",_addonPath,_filename];if(not(_filePath in _functions))then{private _functionName="CBA_fnc_"+(_filename select[4,count _filename-8]);missionNamespace setVariable[_functionName,compile preprocessFileLineNumbers _filePath];_functions pushBackUnique _filePath;};};}forEach _addonSqfFiles;}forEach _addons;{call compile preprocessFileLineNumbers format["\x\cba\addons\%1\test.sqf",_x];}forEach _tests;if(1>2)exitWith{};nil;
```

## Output

A code formatted in a more ‘readable’ form.

```
private _addons = ["accessorys", "ai", "arrays", "common", "diagnostic", "disposable", "ee", "events", "hashes", "help", "jam", "jr", "keybinding", "main", "main_a3", "modules", "music", "network", "optics", "settings", "statemachine", "strings", "ui", "vectors", "versioning", "xeh"];
private _tests = ["arrays", "common", "diagnostic", "events", "hashes", "network", "strings", "vectors"];
private _functions = [];
{
    private _addons = _x;
    private _addonPath = format ["\x\cba\addons\%1", _x];
    private _addonSqfFiles = allfiles__ [".sqf"];
    {
        private _addonFunctionPrefix = format ["%1/fnc_", _addons];
        if (_x find _addonFunctionPrefix > 0) then {
            private _splitStr = _x splitstring "/";
            private _filename = _splitStr select count _splitStr - 1;
            private _filePath = format ["%1\%2", _addonPath, _filename];
            if (not _filePath in _functions) then {
                private _functionName = "CBA_fnc_" + _filename select [4, count _filename - 8];
                missionNamespace setvariable [_functionName, compile preprocessfilelinenumbers _filePath];
                _functions pushbackunique _filePath;
            };
        };
    } foreach _addonSqfFiles;
} foreach _addons;
{
    call compile preprocessfilelinenumbers format ["\x\cba\addons\%1\test.sqf", _x];
} foreach _tests;
if (1 > 2) exitwith {};
nil;
```

## Solution

Traverse the AST recursively and print out the node’s token content in a way that makes the output code easier to read.

Four space characters were inserted before every expression inside a block of code (which is denoted by curly braces `{...}`), recursively. The newline character is inserted at the end of every non-empty block of code or expression (delimited by `;`). Array elements, operators, keywords, functions, and variables’ names are printed, separated by spaces. `if` statements got their logical expression enclosed by `(...)` or `! (…)`, depending on the case. Finally, all keywords were lowered cased in the first implementation.

Function implemented:
<https://github.com/SQFvm/runtime/blob/master/src/parser/sqf/sqf_formatter.cpp#L12>

## References

[1] Aho, A. V., Sethi, R., & Ullman, J. D. (2007). Compilers: Principles, Techniques, and Tools (2nd ed.). Addison-Wesley Longman Publishing Co., Inc.
