---
layout: post 
title: JVM Garbage Collector 
categories: JDK, JVM, garbage collector, memory management
tags: [Garbage collector, memory management, JVM, JDK]
author: raul
---
An essay to explain some of the main ideas regarding the garbage collector on the JVM

## JVM Garbage Collector ##

The aim of this document is to give insights about the garbage collection process on Java Virtual Machines. This process, which helps programmers to forget about allocating and freeing memory, is often misunderstood, which leads to problems such as running out of memory, spending too much time on processes such as freeing or allocating memory and generating too long pauses (stop-the-world events, which are explained below) during its execution. Following, general concepts about the automatic management of memory are explained and then the main ways to perform garbage collection in Java are explained. This document does not deal with flags needed to tune the garbage collector since these will be analysed in future publications

You can download the whole document using the following link.

[***JVM Garbage Collector***](/files/guides/JVMGarbageCollectionV1.1.pdf)