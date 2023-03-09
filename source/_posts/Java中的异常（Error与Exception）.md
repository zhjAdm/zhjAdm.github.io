---
title: Java中的异常（Error与Exception）
abbrlink: 45192
date: 2021-06-20 14:59:06
tags: [Java,异常]
categories: Java
description: Java中的异常常见的异常分类与处理机制

---

# 简介

程序在运行时，发生不被期望的事件，它影响程序的正常执行，这被称为异常。Java中拥有自己的异常处理机制，当异常发生时能够按照代码预设的处理逻辑，来减少异常对后续程序运行影响，尽可能保证程序正常的运行。这些异常有的是因为用户错误引起，有的是程序错误引起的，还有其它一些是因为物理错误引起的。

# Java中的异常体系

为万物皆对象的Java中，异常同样也是作为一个对象处理。Throwable 是 Java 语言中所有```错误（Error）```和```异常（Exception）```的超类。在 Java 中只有 Throwable 类型的实例才可以被```抛出（throw）```或者```捕获（catch）```，它是异常处理机制的基本组成类型。
![](https://raw.githubusercontent.com/zhjAdm/ImageHosting/main/20210620155311.png)

### 1.Error

无法被程序所处理的错误。例如，Java 虚拟机运行错误（Virtual MachineError）、虚拟机内存不够错误(OutOfMemoryError)、类定义错误（NoClassDefFoundError）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

### 2.Exception

程序本身可以处理的异常，可以通过```catch```捕获。 ```Exception```又分为```受检查异常```(必须处理) 和```不受检查异常```(可以不处理)。

##### 受检查异常

Java代码在编译的过程中，如果没有被```try/catch```包围处理的话，就不会被编译通过。除了```RuntimeException```及其子类以外，其他的```Exception```类及其子类都属于受检查异常 。常见的受检查异常有： ```IO 相关的异常```、```ClassNotFoundException``` 、```SQLException```等等。

##### 不受检查异常

Java中即使我们不做处理同样可以编译通过。```RuntimeException```git及其子类都统称为非受检查异常，例如：```NullPointerException```、```NumberFormatException```（字符串转换为数字）、```ArrayIndexOutOfBoundsException```（数组越界）、```ClassCastException```（类型转换错误）、```ArithmeticException```（算术错误）等等。

### 3.try-catch-finally

```try```块：用于包裹可能发生异常的代码，用来捕获异常。后面通常跟着一个或者多个```catch```块，如果没有则必须有个```finally```块。

```

```