---
layout: post
title: "Java中的并发工具类"
author: "郝强"
categories: resources
tags: [resources,concurrency]
image: java-concurrency-util-0.jpeg
---

## 等待多线程完成的CountDownLatch

CountDownLatch允许一个或多个线程等待其他线程完成操作。

CountDownLatch的构造函数一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。

## 同步屏障CyclicBarrier

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程达到一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

## CyclicBarrier和CountDownLatch的区别

CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重制。所以CyclicBarrier能处理更为复杂的业务场景。例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。

CyclicBarrier还提供了其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken()方法用来了解阻塞线程是否被中断。

## 控制并发线程数的Semaphore

## 线程间交换数据的Exchanger

