---
layout: post
title: '[Reading Notes]Modern Java in Action--Chapter 16'
date: 2024-08-26 22:11:00 +0000
toc: true
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## Simple use of Futures
The **Future** interface was introduced in Java 5 to model a result made available at some point in the future. Advantage of Future is that it’s friendlier to work with than lower-level Threads.

To work with a **Future**, you typically have to wrap the time-consuming operation inside a **Callable** object and submit it to an **ExecutorService**. The following listing shows an example written before Java 8.
```java
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double>() {
        public Double call() {
            return doSomeLongComputation();
        }});
doSomethingElse();
try {
    Double result = future.get(1, TimeUnit.SECONDS);
} catch (ExecutionException ee) {
    // the computation threw an exception
} catch (InterruptedException ie) {
    // the current thread was interrupted while waiting
} catch (TimeoutException te) {
    // the timeout expired before the Future completion
}
```

### Limitations of Futures
Futures would be more useful to have more declarative features in the implementation, such as these:
- Combining two asynchronous computations both when they’re independent and when the second depends on the result of the first
- Waiting for the completion of all tasks performed by a set of Futures
- Waiting for the completion of only the quickest task in a set of Futures (possibly because the Futures are trying to calculate the same value in different ways)
  and retrieving its result
- Programmatically completing a Future (that is, by providing the result of the asynchronous operation manually)
- Reacting to a Future completion (that is, being notified when the completion happens and then being able to perform a further action with the result of the Future instead of being blocked while waiting for its result)

### Synchronous vs. asynchronous API
The phrase **synchronous** API is another way of talking about a traditional call to a method: you call it, the caller waits while the method computes, the method returns, and the caller continues with the returned value. Even if the caller and callee were executed on different threads, the caller would still wait for the callee to complete. This situation gives rise to the phrase **blocking** call.

By contrast, in an **asynchronous** API the method returns immediately (or at least before its computation is complete), delegating its remaining computation to a thread, which runs asynchronously to the caller—hence, the phrase **nonblocking** call. The remaining computation gives its value to the caller by calling a callback method, or the caller invokes a further “wait until the computation is complete” method.

This style of computation is common in I/O systems programming: you initiate a disc access, which happens asynchronously while you do more computation, and when you have nothing more useful to do, you wait until the disc blocks are loaded into memory.

## Implementing an asynchronous API
To start implementing the best-price-finder application, define the API that each shop should provide. First, a shop declares a method that returns the price of a product, given its name:
```java
public class Shop {
            public double getPrice(String product) {
                // to be implemented
            }
}
```

### Converting a synchronous method into an asynchronous one
The Java 8 CompletableFuture class gives you various possibilities to implement this method easily, as shown in the following listing.
```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread( () -> {
                double price = calculatePrice(product);
                futurePrice.complete(price);
    }).start();
    return futurePrice;
}
```

### Dealing with errors
```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread( () -> {
        try {
            double price = calculatePrice(product);
            futurePrice.complete(price);
        } catch (Exception ex) {
            futurePrice.completeExceptionally(ex);
        }
    }).start();
    return futurePrice;
}
```

#### CREATING A COMPLETABLEFUTURE WITH THE SUPPLYASYNC FACTORY METHOD
```java
public Future<Double> getPriceAsync(String product) {
    return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```
## Making your code nonblocking
You’ve been asked to develop a best-price-finder application, and all the shops you have to query provide only the same synchronous API. In other words, you have a list of shops, like this one:
```java
List<Shop> shops = List.of(new Shop("BestPrice"),
                           new Shop("LetsSaveBig"),
                           new Shop("MyFavoriteShop"),
                           new Shop("BuyItAll"));
```
You have to implement a method with the following signature, which, given the name of a product, returns a list of strings. Each string contains the name of a shop and the price of the requested product in that shop, as follows:
```java
public List<String> findPrices(String product);
```

Your first idea probably will be to use the Stream features.
```java
public List<String> findPrices(String product) {
    return shops.stream()
            .map(shop -> String.format("%s price is %.2f",
                    shop.getName(), shop.getPrice(product)))
            .collect(toList());
}
```

### Parallelizing requests using a parallel Stream
```java
public List<String> findPrices(String product) {
    return shops.parallelStream()
            .map(shop -> String.format("%s price is %.2f",
                    shop.getName(), shop.getPrice(product)))
            .collect(toList());
}
```

### Making asynchronous requests with CompletableFutures
```java
public List<String> findPrices(String product) {
    List<CompletableFuture<String>> priceFutures =
            shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(
                    () -> shop.getName() + " price is " +
                            shop.getPrice(product)))
            .collect(Collectors.toList());
    return priceFutures.stream()
            .map(CompletableFuture::join)
            .collect(toList());
}
```
Given the lazy nature of intermediate stream operations, if you’d processed the stream in a single pipeline, you’d have succeeded only in executing all the requests to different shops synchronously and sequentially. The creation of each **CompletableFuture** to interrogate a given shop would start only when the computation of the pre- vious one completed, letting the join method return the result of that computation.

### Using a custom Executor
```java
CompletableFuture.supplyAsync(() -> shop.getName() + " price is " +
                                    shop.getPrice(product), executor);
```

## Pipelining asynchronous tasks
### Using the Discount service
```java
public List<String> findPrices(String product) {
    return shops.stream()
            .map(shop -> shop.getPrice(product))
            .map(Quote::parse)
            .map(Discount::applyDiscount)
            .collect(toList());
}
```

### Composing synchronous and asynchronous operations
```java
public List<String> findPrices(String product) {
    List<CompletableFuture<String>> priceFutures =
            shops.stream()
                 .map(shop -> CompletableFuture.supplyAsync(
                                       () -> shop.getPrice(product), executor))
                 .map(future -> future.thenApply(Quote::parse))
                 .map(future -> future.thenCompose(quote ->
                             CompletableFuture.supplyAsync(
                               () -> Discount.applyDiscount(quote), executor)))
                    .collect(toList());
    return priceFutures.stream()
            .map(CompletableFuture::join)
            .collect(toList());
}
```

### Combining two CompletableFutures: dependent and independent
```java
Future<Double> futurePriceInUSD =
        CompletableFuture.supplyAsync(() -> shop.getPrice(product))
        .thenCombine(
                CompletableFuture.supplyAsync(
                        () ->  exchangeService.getRate(Money.EUR, Money.USD)),
                (price, rate) -> price * rate
        );
```

### Using timeouts effectively
```java
Future<Double> futurePriceInUSD =
                CompletableFuture.supplyAsync(() -> shop.getPrice(product))
                .thenCombine(
                    CompletableFuture.supplyAsync(
                        () ->  exchangeService.getRate(Money.EUR, Money.USD)), 
                    (price, rate) -> price * rate
                )
                .orTimeout(3, TimeUnit.SECONDS);
```