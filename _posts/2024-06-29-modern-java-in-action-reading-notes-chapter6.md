---
layout: post
title: 'Modern Java in Action reading notes--Chapter 6'
date: 2024-06-29 23:00:00 +0000
---
## Collectors as advanced reductions
Invoking the collect method on a stream triggers a reduction operation (parameterized by a Collector) on the elements of the stream itself.

## Reducing and summarizing
### Finding maximum and minimum in a stream of values
You can use two collectors, *Collectors.maxBy* and *Collectors.minBy*, to calculate the maximum or minimum value in a stream.
```java8
Comparator<Dish> dishCaloriesComparator =
    Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish =
    menu.stream()
        .collect(maxBy(dishCaloriesComparator));
```

### Summarization
The Collectors class provides a specific factory method for summing: *Collectors.summingInt*.
```java8
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
A *Collectors.averagingInt* is available to calculate the average of the same set of numeric values:
```java8
double avgCalories =
            menu.stream().collect(averagingInt(Dish::getCalories));
```
You can count the elements in the menu and obtain the sum, average, maximum, and minimum of the calories contained in each dish with a single summarizing operation:
```java8
IntSummaryStatistics menuStatistics =
                menu.stream().collect(summarizingInt(Dish::getCalories));
```

## Collect vs. reduce
You may wonder what the differences between the *collect* and *reduce* methods of the stream interface are, because often you can obtain the same results using either method. 

For instance, you can achieve what is done by the *toList* Collector using the *reduce* method as follows:
```java8
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(
                               new ArrayList<Integer>(),
                               (List<Integer> l, Integer e) -> {
                                         l.add(e);
                                         return l; },
                               (List<Integer> l1, List<Integer> l2) -> {
                                         l1.addAll(l2);
                                         return l1; });
```
This solution has two problems: a semantic one and a practical one. 

### Semantic problem

The semantic problem lies in the fact that the *reduce* method is meant to combine two values and produce a new one; it’s an immutable reduction. 

In contrast, the *collect* method is designed to mutate a container to accumulate the result it’s supposed to produce. 

This means that the previous snippet of code is misusing the reduce method, because it’s mutating in place the List used as accumulator. 

### Practical problem

Using the *reduce* method with the wrong semantic is also the cause of a practical problem: this reduction process can’t work in parallel, because the concurrent modification of the same data structure operated by multiple threads can corrupt the List itself. 

In this case, if you want thread safety, you’ll need to allocate a new List every time, which would impair performance by object allocation. This is the main reason why the collect method is useful for expressing reduction working on a mutable container but crucially in a parallel-friendly way.

## Grouping
A common database operation is to group items in a set, based on one or more properties. 

You can easily perform this task using a collector returned by the *Collectors.groupingBy* factory method, as follows:
```java8
Map<Dish.Type, List<Dish>> dishesByType =
                      menu.stream().collect(groupingBy(Dish::getType));
```

### Manipulating grouped elements
Frequently after performing a grouping operation you may need to manipulate the elements in each resulting group. 

Suppose, for example, that you want to filter only the caloric dishes, let’s say the ones with more than 500 calories.
```java8
Map<Dish.Type, List<Dish>> caloricDishesByType =
              menu.stream()
                  .collect(groupingBy(Dish::getType,
                           filtering(dish -> dish.getCalories() > 500, toList())));
```

### Multilevel grouping
You can perform a two-level grouping. To achieve this you can pass to it a second inner *groupingBy* to the outer *groupingBy*, defining a second-level criterion to classify the stream’s items, as shown in the next listing.
```java8
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
        menu.stream().collect(
                groupingBy(Dish::getType,
                    groupingBy(dish -> {
                        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                        else return CaloricLevel.FAT;
                    })
                )
        );
```

### Collecting data in subgroups
The second collector passed to the first *groupingBy* can be any type of collector, not just another *groupingBy*. 

For instance, it’s possible to count the number of Dishes in the menu for each type, by passing the *counting* collector as a second argument to the *groupingBy* collector:
```java8
Map<Dish.Type, Long> typesCount = menu.stream().collect(
                    groupingBy(Dish::getType, counting()));
```

## The Collector interface
The *Collector* interface consists of a set of methods that provide a blueprint for how to implement specific reduction operations (collectors). You’re free to create customized reduction operations by providing your own implementation of the *Collector* interface.

The next listing shows the interface signature together with the five methods it declares.
```java8
public interface Collector<T, A, R> {
            Supplier<A> supplier();
            BiConsumer<A, T> accumulator();
            Function<A, R> finisher();
            BinaryOperator<A> combiner();
            Set<Characteristics> characteristics();
}
```
In this listing, the following definitions apply:
- T is the generic type of the items in the stream to be collected.
- A is the type of the accumulator, the object on which the partial result will be
accumulated during the collection process.
- R is the type of the object (typically, but not always, the collection) resulting
from the collect operation.

