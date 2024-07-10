---
layout: post
title: 'Modern Java in Action reading notes--Chapter 11'
date: 2024-07-09 16:35:00 +0000
---

## How do you model the absence of a value?

Imagine that you have the following nested object structure for a person who owns a car and has car insurance in the
following listing.

```java
public class Person {
    private Car car;

    public Car getCar() {
        return car;
    }
}

public class Car {
    private Insurance insurance;

    public Insurance getInsurance() {
        return insurance;
    }
}

public class Insurance {
    private String name;

    public String getName() {
        return name;
    }
}
```

What’s problematic with the following code?

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

### Reducing NullPointerExceptions with defensive checking

Typically, you can add null checks where necessary and often with different styles.

A first attempt to write a method preventing a NullPointerException is shown in the following listing.

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

We labeled this method “deep doubts” because it shows a recurring pattern: every time you doubt that a variable could
be `null`, you’re obliged to add a further nested `if` block, increasing the indentation level of the code. This
technique clearly scales poorly and compromises readability.

Try to avoid this problem by doing something different as shown in the next listing.

```java
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
```

Nevertheless, this solution is also far from ideal; now the method has four distinct exit points, making it hard to
maintain.

### Problems with null

- **It’s a source of error**. `NullPointerException` is by far the most common exception in Java.
- **It bloats your code**. It worsens readability by making it necessary to fill your code with *null* checks that are
  often deeply nested.
- **It’s meaningless**. It doesn’t have any semantic meaning, and in particular, it represents the wrong way to model
  the absence of a value in a statically typed language.
- **It breaks Java philosophy**. Java always hides pointers from developers except in one case: the null pointer.
- **It creates a hole in the type system**. `null` carries no type or other information, so it can be assigned to any
  reference type. This situation is a problem because when `null` is propagated to another part of the system, you have
  no idea what that
  `null` was initially supposed to be.

## Introducing the Optional class

Java 8 introduces a new class called `java.util.Optional<T>` that’s inspired by Haskell and Scala. The class
encapsulates an optional value.

When a value is present, the `Optional` class wraps it. Conversely, the absence of a value is modeled with an empty
optional returned by the method `Optional.empty`.

An important, practical semantic difference in using `Optional`s instead of `null`s is that in the first case, declaring
a variable of type `Optional<Car>` instead of `Car` clearly signals that a missing value is permitted there.

```java 8
public class Person {
    private Optional<Car> car;
    public Optional<Car> getCar() { return car; }
}
public class Car {
    private Optional<Insurance> insurance;
    public Optional<Insurance> getInsurance() { return insurance; }
}
public class Insurance {
    private String name;
    public String getName() { return name; }
}
```

At the same time, the fact that the name of the insurance company is declared of type `String` instead
of `Optional<String>` makes it evident that an insurance company must have a name. This way, you know for certain
whether you’ll get a `NullPointerException` when dereferencing the name of an insurance company; you don’t have to add
a `null` check, because doing so will hide the problem instead of fixing it. An insurance company must have a name, so
if you find one without a name, you’ll have to work out what’s wrong in your data instead of adding a piece of code to
cover up this circumstance.

Consistently using `Optional` values creates a clear distinction between a missing value that’s planned for and a value
that’s absent only because of a bug in your algorithm or a problem in your data. It’s important to note that the
intention of the `Optional` class isn’t to replace every single `null` reference. Instead, its purpose is to help you
design more-comprehensible APIs so that by reading the signature of a method, you can tell whether to expect an optional
value. You’re forced to actively unwrap an optional to deal with the absence of a value.

## Patterns for adopting Optionals

### Extracting and transforming values from Optionals with map

A common pattern is to extract information from an object. You may want to extract the name from an insurance company,
for example. You need to check whether insurance is `null` before extracting the name as follows:

```java
String name = null;
if(insurance !=null){
    name =insurance.getName();
}
```

`Optional` supports a map method for this pattern, which works as follows:

```java 8
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

This method is conceptually similar to the map method of Stream. The `map` operation applies the provided function to
each element of a stream. You could also think of an `Optional` object as being a particular collection of data,
containing at most a single element. If the `Optional` contains a value, the function passed as argument to `map`
transforms that value. If the `Optional` is empty, nothing happens.

### Chaining Optional objects with flatMap
Because you’ve learned how to use `map`, your first reaction may be to use `map` to rewrite the code as follows:
```java 8
Optional<Person> optPerson = Optional.of(person);
Optional<String> name =
    optPerson.map(Person::getCar)
             .map(Car::getInsurance)
             .map(Insurance::getName);
```
Unfortunately, this code doesn’t compile. Why? The
variable `optPerson` is of type `Optional<Person>`, so it’s
perfectly fine to call the `map` method. But `getCar`
returns an object of type `Optional<Car>`, which means that the result of the map operation is an object of type `Optional<Optional<Car>>`.
As a result, the call to `getInsurance` is invalid because the outermost optional contains as its value another optional, which of course doesn’t support the `getInsurance` method.

How can you solve this problem? Again, you can look at a pattern you’ve used previously with streams: the `flatMap` method. With streams, the `flatMap` method takes a function as an argument and returns another stream. This function is applied to each element of a stream, resulting in a stream of streams. But `flatMap` has the effect of replacing each generated stream with the contents of that stream. In other words, all the separate streams that are generated by the function get amalgamated or flattened into a single stream. What you want here is something similar, but you want to flatten a two-level optional into one.


```java 8
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");
}
```







