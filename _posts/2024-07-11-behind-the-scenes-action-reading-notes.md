---
layout: post
title: 'Behind the scenes reading notes'
date: 2024-07-11 21:56:00 +0000
---
## Look into the bytecode to see how Java handles lambdas
What does a lambda expression look like inside Java code and inside the JVM? It is obviously some type of value, and Java permits only two sorts of values: primitive types and object references. Lambdas are obviously not primitive types, so a lambda expression must therefore be some sort of expression that returns an object reference.

Let’s look at an example:
```java
public class LambdaExample {
    private static final String HELLO = "Hello World!";

    public static void main(String[] args) throws Exception {
        Runnable r = () -> System.out.println(HELLO);
        Thread t = new Thread(r);
        t.start();
        t.join();
    }
}
```

Decompiling the bytecode via *javap -c -p* reveals two things. First is the fact that the lambda body has been compiled into a private static method that appears in the main class:
```java
private static void lambda$main$0();
    Code:
       0: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #9                  // String Hello World!
       5: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
```
You might guess that the signature of the private body method matches that of the lambda, and indeed this is the case. A lambda such as this:
```java
public class StringFunction {
    public static final Function<String, Integer> fn = s -> s.length();
}
```
will produce a body method such as this, which takes a string and returns an integer, matching the signature of the interface method:
```java
private static java.lang.Integer lambda$static$0(java.lang.String);
    Code:
       0: aload_0
       1: invokevirtual #2                  // Method java/lang/String.length:()I
       4: invokestatic  #3                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       7: areturn
```
The second thing to notice about the bytecode is the form of the main method:
```java
public static void main(java.lang.String[]) throws java.lang.Exception;
    Code:
       0: invokedynamic #2,  0              // InvokeDynamic #0:run:()Ljava/lang/Runnable;
       5: astore_1
       6: new           #3                  // class java/lang/Thread
       9: dup
      10: aload_1
      11: invokespecial #4                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      14: astore_2
      15: aload_2
      16: invokevirtual #5                  // Method java/lang/Thread.start:()V
      19: aload_2
      20: invokevirtual #6                  // Method java/lang/Thread.join:()V
      23: return
```
Notice that the bytecode begins with an invokedynamic call.

The most straightforward way to understand the invokedynamic call in this code is to think of it as a call to an unusual form of the factory method. The method call returns an instance of some type that implements Runnable. The exact type is not specified in the bytecode. The actual type does not exist at compile time and will be created on demand at runtime.

To better explain this, I’ll discuss three mechanisms that work together to produce this capability: call sites, method handles, and bootstrapping.

## Call sites
A location in the bytecode where a method invocation instruction occurs is known as a *call site*.

Java bytecode has traditionally had four opcodes that handle different cases of method invocation: **static methods**, **“normal” invocation** (a virtual call that may involve method overriding), **interface lookup**, and **“special” invocation** (for cases where override resolution is not required, such as superclass calls and private methods).

Here, invokedynamic call sites are represented as CallSite objects in the Java heap.

When the invokedynamic instruction is reached, the JVM locates the corresponding call site object (or it creates one, if this call site has never been reached before). The call site object contains a *method handle*, which is an object that represents the method that I actually want to invoke.

## Method handles
A method handle (MH) is Java’s version of a type-safe function pointer. It’s a way of referring to a method that the code might want to call, similar to a Method object from Java reflection. The MH has an invoke() method that actually executes the underlying method, in just the same way as reflection.

## Bootstrapping
The first time each specific invokedynamic call site is encountered in the bytecode instruction stream, the JVM doesn’t know which method it targets. In fact, there is no call site object associated with the instruction.

The call site needs to be *bootstrapped*, and the JVM achieves this by running a bootstrap method (BSM) to generate and return a call site object.

Each invokedynamic call site has a BSM associated with it, which is stored in a separate area of the class file. These methods allow user code to programmatically determine linkage at runtime.

Decompiling an *invokedynamic* call, such as that from my original example of a *Runnable*, shows that it has this form:
```java
0: invokedynamic #2,  0
```
And in the class file’s constant pool, notice that entry #2 is a constant of type *CONSTANT_InvokeDynamic*. The relevant parts of the constant pool are
```java
#2 = InvokeDynamic      #0:#31
   ...
  #31 = NameAndType        #46:#47        // run:()Ljava/lang/Runnable;
  #46 = Utf8               run
  #47 = Utf8               ()Ljava/lang/Runnable;
```

The presence of 0 in the constant is a clue. Constant pool entries are numbered from 1, so the 0 reminds you that the actual BSM is located in another part of the class file.

For lambdas, the *NameAndType* entry takes on a special form. The name is arbitrary, but the type signature contains some useful information.

The return type corresponds to the return type of the *invokedynamic* factory; it is the target type of the lambda expression. Also, the argument list consists of the types of elements that are being captured by the lambda. In the case of a stateless lambda, the return type will always be empty. Only a Java closure will have arguments present.

A BSM takes at least three arguments and returns a CallSite. The standard arguments are of these types:
- **MethodHandles.Lookup**: A lookup object on the class in which the call site occurs
- **String**: The name mentioned in the NameAndType
- **MethodType**: The resolved type descriptor of the NameAndType

## Decoding the lambda’s bootstrap method
Use the -v argument to *javap* to see the bootstrap methods. This is necessary because the bootstrap methods live in a special part of the class file and make references back into the main constant pool. For this simple *Runnable* example, there is a single bootstrap method in that section:
```java
BootstrapMethods:
  0: #28 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
        (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;
         Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;
         Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #29 ()V
      #30 REF_invokeStatic LambdaExample.lambda$main$0:()V
      #29 ()V
```
The bootstrap method for this call site is entry #28 in the constant pool. This is an entry of type *MethodHandle*. Now let’s compare it to the case of the string function example:
```java
0: #27 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
        (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;
         Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;
         Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #28 (Ljava/lang/Object;)Ljava/lang/Object;
      #29 REF_invokeStatic StringFunction.lambda$static$0:(Ljava/lang/String;)Ljava/lang/Integer;
      #30 (Ljava/lang/String;)Ljava/lang/Integer;
```
The method handle that will be used as the BSM is the same static method *LambdaMetafactory.metafactory( ... )*.

The part that has changed is the method arguments. These are the additional static arguments for lambda expressions, and there are three of them. They represent the lambda’s signature and the method handle for the actual final invocation target of the lambda: the lambda body. The third static argument is the erased form of the signature.

## The lambda metafactories
The BSM makes a call to this static method, which ultimately returns a call site object. When the *invokedynamic* instruction is executed, the method handle contained in the call site will return an instance of a class that implements the lambda’s target type.

The source code for the *metafactory* method is relatively simple:
```java
public static CallSite metafactory(MethodHandles.Lookup caller,
                                       String invokedName,
                                       MethodType invokedType,
                                       MethodType samMethodType,
                                       MethodHandle implMethod,
                                       MethodType instantiatedMethodType)
            throws LambdaConversionException {
        AbstractValidatingLambdaMetafactory mf;
        mf = new InnerClassLambdaMetafactory(caller, invokedType,
                                             invokedName, samMethodType,
                                             implMethod, instantiatedMethodType,
                                             false, EMPTY_CLASS_ARRAY, EMPTY_MT_ARRAY);
        mf.validateMetafactoryArgs();
        return mf.buildCallSite();
}
```
The *Lookup* object corresponds to the context where the *invokedynamic* instruction lives. In this case, that is the same class where the lambda was defined, so the lookup context will have the correct permissions to access the private method that the lambda body was compiled into.

The invoked name and type are provided by the VM and are implementation details. The final three parameters are the additional static arguments from the BSM.

In the current implementation, the *metafactory* delegates to code that uses an internal, shaded copy of the ASM bytecode libraries to spin up an inner class that implements the target type.

## Summary
When you create a lambda expression, an *invokedynamic* call will be generated in the bytecode. The *invokedynamic* call has a BSM (bootstrap method) associated with it. JVM uses BSM to decide which method the *invokedynamic* targets. The BSM points to a static method *LambdaMetafactory.metafactory( ... )* which returns an instance of a class that implements the lambda’s target type.

In a nutshell, when you create a lambda expression, JVM uses an *invokedynamic* call to create an instance of some type that implements the functional interface.

## Reference
[Behind the scenes: How do lambda expressions really work in Java?](https://blogs.oracle.com/javamagazine/post/behind-the-scenes-how-do-lambda-expressions-really-work-in-java)
