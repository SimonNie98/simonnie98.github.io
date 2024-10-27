---
title: Cuda learn (1)
date: 2023-09-06T00:40:23+08:00
tags: [CUDA]
categories: [MLSys]
toc: true
comments: true
---
<!-- more -->
# Cuda learn (1)

[toc]


## Basic Concepts

A GPU is built around an array of Streaming Multiprocessors (SMs) (see [Hardware Implementation](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#hardware-implementation) for more details). A multi-threaded program is partitioned into blocks of threads that execute independently from each other, so that a GPU with more multiprocessors will automatically execute the program in less time than a GPU with fewer multiprocessors.

<!--more-->

here is your content.
