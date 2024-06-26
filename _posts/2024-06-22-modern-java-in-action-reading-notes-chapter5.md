---
layout: post
title: 'Modern Java in Action reading notes--Chapter 5'
date: 2024-06-22 23:35:00 +0000
---
In this chapter, you’ll have an extensive look at the various operations supported by the Streams API.
# Filtering
## Filtering with a predicate
The Stream interface supports a filter method. This operation takes as argument a predicate (a function returning a boolean) and returns a stream including all elements that match the predicate.
## Filtering unique elements
Streams also support a method called distinct that returns a stream with unique elements (according to the implementation of the hashcode and equals methods of the objects produced by the stream).

For example, the following code filters all even numbers from a list and then eliminates duplicates (using the equals method for the comparison).
```java19
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i % 2 == 0)
       .distinct()
       .forEach(System.out::println);
```

# Mapping
## map
Streams support the *map* method, which takes a function as argument. The function is applied to each element, mapping it into a new element (the word mapping is used because it has a meaning similar to transforming but with the nuance of “creating a new version of” rather than “modifying”).
```java8
List<String> dishNames = menu.stream()
                             .map(Dish::getName)
                             .collect(toList());
```
## flatMap
The *flatMap* method lets you replace each value of a stream with another stream and then concatenates all the generated streams into a single stream.
```java8
List<String> uniqueCharacters =
  words.stream()
       .map(word -> word.split(""))
       .flatMap(Arrays::stream)
       .distinct()
       .collect(toList());
```

# Reducing
In functional programming-language jargon, the reduction operation is referred to as a fold because you can view this operation as repeatedly folding a long piece of paper (your stream) until it forms a small square, which is the result of the fold operation.
```java8
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

