---
layout: post
title: 'Modern Java in Action reading notes--Chapter 4'
date: 2024-06-21 23:23:00 +0000
---
The Streams API in Java 8 lets you write code that’s
- Declarative—More concise and readable
- Composable—Greater flexibility 
- Parallelizable—Better performance

# Stream
Stream is “a sequence of elements from a source that supports data-processing operations.”

Stream operations have two important characteristics:
- Pipelining—Many stream operations return a stream themselves, allowing operations to be chained to form a larger pipeline.
- Internal iteration—In contrast to collections, which are iterated explicitly using an iterator, stream operations do the iteration behind the scenes for you.

# Streams vs. Collections

The difference between collections and streams has to do with when things are computed. 

A collection is an in-memory data structure that holds all the values the data structure currently has—every element in the collection has to be computed before it can be added to the collection. A collection is eagerly constructed.

By contrast, a stream is a conceptually fixed data structure whose elements are computed on demand. These elements are produced only as and when required. A stream is like a lazily constructed collection.

![Streams versus collections](_site/assets/images/collections_vs_streams.png)

# External vs. Internal iteration
With the Collection interface, the user need to do the iteration by themselves (for example, using for-each); this is called external iteration. 

The Streams library, by contrast, uses internal iteration—it does the iteration for you and takes care of storing the resulting stream value somewhere; you merely provide a function saying what’s to be done.
