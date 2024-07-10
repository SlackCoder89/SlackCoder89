---
layout: post
title: 'Modern Java in Action reading notes--Chapter 3'
date: 2024-06-15 23:51:00 +0000
---
## Functional interface
Functional interface is an interface that specifies only one abstract method.

### Predicate
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

### Consumer
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

### Function
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

### Supplier
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

## Method Reference
The following shows how we can change from passing objects to using method references.
### Step 1: Pass code
```java
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());
```
### Step 2: Use an anonymous class
```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```
### Step 3: Use lambda expressions
```java
inventory.sort((Apple a1, Apple a2)
                        -> a1.getWeight().compareTo(a2.getWeight())
);
```
### Step 4: Use method references
```java 8
inventory.sort(comparing(Apple::getWeight));
```
## Compose lambda expressions
### Composing Comparators
#### Reversed order
```java 8
inventory.sort(comparing(Apple::getWeight).reversed());
```
#### Chaining Comparators
```java 8
inventory.sort(comparing(Apple::getWeight)
         .reversed()
         .thenComparing(Apple::getCountry));
```
### Composing Predicates
#### negate
```java
Predicate<Apple> notRedApple = redApple.negate();
```
This produces the negation of the existing Predicate object redApple.
#### and
```java
Predicate<Apple> redAndHeavyApple =
    redApple.and(apple -> apple.getWeight() > 150);
```
This chains two predicates to produce another Predicate object. The new object predicate an apple is both red and heavy.
#### or
```java
Predicate<Apple> redAndHeavyAppleOrGreen =
    redApple.and(apple -> apple.getWeight() > 150)
            .or(apple -> GREEN.equals(a.getColor()));
```
This chains 3 predicates to produce a more complex Predicate object. The new object predicate apples that are red and heavy or only green apples. 
### Composing Functions
#### andThen
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1); // This returns 4
```
In mathematics, you’d write g(f(x)).
#### compose
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g);
int result = h.apply(1); // This returns 3
```
In mathematics, you’d write f(g(x)).
