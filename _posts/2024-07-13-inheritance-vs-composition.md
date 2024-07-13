---
layout: post
title: 'Inheritance versus Composition'
date: 2024-07-13 22:58:00 +0000
---
## Inheritance
### Advantages
- Class inheritance is straightforward to use, since it's supported directly by the programming language.
- Class inheritance also makes it easier to modify the implementation being reused. When a subclass overrides some but not all operations, it can affect the operations it inherits as well, assuming they call the overridden operations.

### Disadvantages
- You can't change the implementations inherited from parent classes at run-time, because inheritance is defined at compile-time.
- Parent classes often define at least part of their subclasses' physical representation. Because inheritance exposes a subclass to details of its parent's implementation, it's often said that "inheritance breaks encapsulation".
  - The implementation of a subclass becomes so bound up with the implementation of its parent class that any change in the parent's implementation will force the subclass to change.
  - Should any aspect of the inherited implementation not be appropriate for new problem domains, the parent class must be rewritten or replaced by something more appropriate.

## Composition
### Advantages
- Objects are accessed solely through their interfaces, we don't break encapsulation.
- Any object can be replaced at run-time by another as long as it has the same type.
- Because an object's implementation will be written in terms of object interfaces,there are substantially fewer implementation dependencies.

### Disadvantages
- Methods being provided by individual components may have to be implemented in the derived type, even if they are only forwarding methods. 

## Reference
- *Design Patterns: Elements of Reusable Object-Oriented Software*
- [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance#Drawbacks)