---
layout: post
title: '[Reading Notes]Modern Java in Action--Chapter 18'
date: 2024-09-03 22:33:00 +0000
toc: true
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## Implementing and maintaining systems
### Shared mutable data
The reason for the unexpected-variable-value problem is that shared mutable data structures are read and updated by more than one of the methods on which your maintenance centers.

In other words, shared mutable data structures make it harder to track changes in different parts of your program.

#### Side effect
In computer science, an operation, function or expression is said to have a **side effect** if it has any observable effect other than its primary effect of reading the value of its arguments and returning a value to the invoker of the operation. Example side effects include modifying a non-local variable, a static local variable or a mutable argument passed by reference; raising errors or exceptions; performing I/O; or calling other functions with side effects.

In the presence of side effects, a program's behaviour may depend on history; that is, the order of evaluation matters. Understanding and debugging a function with side effects requires knowledge about the context and its possible histories.

The degree to which side effects are used depends on the programming paradigm. For example, imperative programming is commonly used to produce side effects, to update a system's state. By contrast, declarative programming is commonly used to report on the state of system, without side effects.

### Declarative programming
One way of implementing a system centers on what’s to be done. This “what” style is often called *declarative programming*. You provide rules saying what you want, and you expect the system to decide how to achieve that goal. This type of programming is great because it reads closer to the problem statement.

## What’s functional programming?
In the context of functional programming, however, a function corresponds to a mathematical function: it takes zero or more arguments, returns one or more results, and has no side effects.

### Functional-style Java
A function might do nonfunctional things internally as long as it doesn’t expose any of these side effects to the rest of the system. In other words, programmers perform a side effect that can’t be observed by callers. The callers don’t need to know or care, because it can’t affect them. This is functional-style programming.

Our guideline is that to be regarded as functional style, a function or method can mutate only local variables. There’s an additional requirement to being functional, that a function or method shouldn’t throw any exceptions. A justification is that throwing an exception would mean that a result is being signaled other than via the function returning a value.

Finally, for pragmatic reasons, you may find it convenient for functional-style code to be able to output debugging information to some form of log file. This code can’t be strictly described as functional, but in practice, you retain most of the benefits of functional-style programming.

### Referential transparency
The restrictions on no visible side effects (no mutating structure visible to callers, no I/O, no exceptions) encode the concept of referential transparency. A function is referentially transparent if it always returns the same result value when it’s called with the same argument value.

Put another way, a function consistently produces the same result given the same input, no matter where and when it’s invoked.