---
layout: post
title: '[Reading Notes]Modern Java in Action--Chapter 14'
date: 2024-07-09 16:35:00 +0000
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## LIMITED VISIBILITY CONTROL
Java provides access modifiers to support infor- mation hiding. These modifiers are public, protected, package-level, and private visibility. But what about controlling visibility between packages?

Most applications have several packages defined to group various classes, but packages have limited support for visibility control. If you want classes and interfaces from one package to be visible to another package, you have to declare them as public.

As a consequence, these classes and interfaces are accessible to everyone else as well.

## Java modules
Java 9 provides a new unit of Java program structure: the module. A module is introduced with a new keyword **module**, followed by its name and its body. Such a module descriptor lives in a special file: module-info.java, which is compiled to module-info.class.

The body of a module descriptor consists of clauses, of which the two most important are requires and exports. The former clause specifies what other modules your modules need to run, and exports specifies everything that your module wants to be visible for other modules to use.

### requires
The **requires** clause lets you specify that your module depends on another module at both compile time and runtime.

### exports
The **exports** clause makes the public types in specific packages available for use by other modules. By default, no package is exported. You gain strong encapsulation by making explicit what packages should be exported.



