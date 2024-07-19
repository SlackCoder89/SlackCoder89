---
layout: post
title: 'Information hiding'
date: 2024-07-20 00:08:00 +0000
toc: true
---
## Table Of Contents
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## Description
Information hiding is the principle of segregation of the **design decisions** in a computer program that are most likely to change, thus protecting other parts of the program from extensive modification if the design decision is changed. The protection involves providing a stable interface which protects the remainder of the program from the implementation (whose details are likely to change). 

Written in another way, information hiding is the ability to prevent certain aspects of a class or software component from being accessible to its clients.

## Why
The primary goal is to prevent extensive modification to clients whenever the implementation details of a module or program are changed. 

## Benefits
Information hiding helps us improve the ability to change our system while minimizing the impact on other parts of the system.

## How
1. Identify all of the pieces of a design that are likely to change or other design details that you might want to hide
2. Isolate each secret into its own module, class, or function
3. Design intermediate interfaces that are insensitive to changes in the underlying secrets

## Reference
- [Information Hiding](https://embeddedartistry.com/fieldmanual-terms/information-hiding/)
- [Wikipedia: Information Hiding](https://en.wikipedia.org/wiki/Information_hiding)
