---
layout: post
title: '[Reading Notes]Modern Java in Action--Chapter 15'
date: 2024-08-02 23:13:00 +0000
toc: true
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## PROBLEMS WITH THREADS
The problem is that operating-system threads are expensive to create and to destroy (involving interaction with page tables), and moreover, only a limited number exist.

## THREAD POOLS AND WHY THEY’RE BETTER
These threads are returned to the pool when their tasks terminate. One great outcome is that it’s cheap to submit thousands of tasks to a thread pool while keeping the number of tasks to a hardware-appropriate number.

## THREAD POOLS AND WHY THEY’RE WORSE
This situation is generally good, in that it allows you to submit many tasks without accidentally creating an excessive number of threads, but you have to be wary of tasks that sleep or wait for I/O or network connections. In the context of blocking I/O, these tasks occupy worker threads but do no useful work while they’re waiting. 

Try taking four hardware threads and a thread pool of size 5 and submitting 20 tasks to it. You might expect that the tasks would run in parallel until all 20 have completed. But suppose that three of the first-submitted tasks sleep or wait for I/O. Then only two threads are available for the remaining 15 tasks, so you’re getting only half the throughput you expected (and would have if you created the thread pool with eight threads instead).

## Sleeping (and other blocking operations) considered harmful
The lesson to remember is that tasks sleeping in a thread pool consume resources by blocking other tasks from starting to run. (They can’t stop tasks already allocated to a thread, as the operating system schedules these tasks.)

It’s not only sleeping that can clog the available threads in a thread pool, of course. Any blocking operation can do the same thing. 

Blocking operations fall into two classes: waiting for another task to do something, such as invoking get() on a Future; and waiting for external interactions such as reads from networks, database servers, or human interface devices such as keyboards.

### What can you do?
You can break your task into two parts—before and after—and ask Java to schedule the after part only when it won’t block.

Compare code A, shown as a single task
```java
work1();
Thread.sleep(10000); 
work2();
```

with code B:
```java
public class ScheduledExecutorServiceExample {
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService
            = Executors.newScheduledThreadPool(1);
        work1();
        scheduledExecutorService.schedule(
            ScheduledExecutorServiceExample::work2, 10, TimeUnit.SECONDS);
        scheduledExecutorService.shutdown();
    }
    public static void work1(){
        System.out.println("Hello from Work1!");
    }
    public static void work2(){
        System.out.println("Hello from Work2!");
    }
}
```

Code B is better, but why? Code A and code B do the same thing. The difference is that code A occupies a precious thread while it sleeps, whereas code B queues another task to execute (with a few bytes of memory and no requirement for a thread) instead of sleeping.

Tasks occupy valuable resources when they start executing, so you should aim to keep them running until they complete and release their resources. Instead of blocking, a task should terminate after submitting a follow-up task to complete the work it intended to do.

