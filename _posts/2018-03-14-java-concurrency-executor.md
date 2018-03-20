---
layout: post
title: "Executor框架"
author: "郝强"
categories: resources
tags: [resources,concurrency]
image: java-concurrency-executor-0.jpeg
---

## Executor框架简介

1. Execute框架的两级调度模型

   在HotSpot VM的线程模型中，Java线程（java.lang.Tread）被一对一映射为本地操作系统线程。Java线程启动时会创建一个本地操作系统线程；当该Java线程终止时，这个操作系统线程也会被回收。操作系统会调度所有线程并将它们分配给可用的CPU。

2. Executor框架的结构与成员

   1. Executor框架的结构

      - 任务。包括执行任务需要实现的接口：Runnable接口或Callable接口。

      - 任务的执行。包括执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架有两个关键类实现了ExecutorService接口（TreadPoolExecutor和ScheduledThreadExecutor）。

      - 异步计算的结果。包括接口Future和实现Future接口的FutureTask类。

        下面是这些接口的简介：

      - Executor是一个接口，他是Executor的框架基础，它将任务的提交与任务的执行分离开来。

      - ThreadPoolExecutor是线程池的核心实现类，用来执行被提交的任务。

      - ScheduledTreadPoolExecutor是一个实现类，可以在给定的延迟后运行命令，或者定期执行命令。ScheduledTreadPoolExecutor比Timer更灵活，功能更强大。

      - Future接口和实现Future接口的FutureTask类，代表异步计算的结果。

      - Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor或者ScheduledThreadPoolExecutor执行。

   2. Executor框架的成员

      1. ThreadPoolExecutor

        ThreadPoolExecutor通常使用工厂类Executor来创建。Executors可以创建3中类型的ThreadPoolExecutor：SingleThreadExecutor、FixedThreadExecutor和CachedThreadPool。

        - FixedThreadPool。适用于为了满足资源管理需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器。
        - SingleThreadExecutor。适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景。
        - CachedThreadPool。是大小无界的线程池，适用于执行很多的短期异步任务的小程序，或者是负载比较轻的服务器。

      2. ScheduledThreadPoolExecutor

         - ScheduledThreadPoolExecutor。适用于需要多个后台线程执行周期任务，同时为了满足资源管理的需求而需要限制后台线程的数量的应用场景。
         - SingleThreadScheduledExecutor。适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的应用场景。

      3. Future接口

         Future接口和实现Future接口的FutureTask类用来表示异步的计算的结果。

      4. Runnable接口和Callable接口

         Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。他们之间的区别是Runnable不会返回结果，而Callable可以返回结果。

