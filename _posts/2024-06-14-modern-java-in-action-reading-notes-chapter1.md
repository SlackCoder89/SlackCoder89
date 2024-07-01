---
layout: post
title: 'Modern Java in Action reading notes--Chapter 1'
date: 2024-06-14 23:00:00 +0000
---

## New Features in Java 8

- The Stream API
- Techniques for passing code to methods
- Default methods in interfaces

## The Stream API

- It provides a support for stream processing. Programmers can express the thoughts of turning a stream of this to stream of that.
- It can be run transparently on several CPU on disjoint parts of the input 

## Functional Programming

- No shared mutable data
- Pass methods and function code to other methods

### Methods as first-class citizens
Prior to Java 8, programmers can only pass values to methods. As the following code shows:
```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();  
    }
});
```
In this example, in order to pass *file.isHidden* function to method, programmers need to wrap it in the class and initialize it as new objects.

Starting with Java 8, programmers can pass functions to methods.
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```