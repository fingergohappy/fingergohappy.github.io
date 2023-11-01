---
title: jyy-operate-system
tags:
  - null
categories:
  - null
cover: https://source.unsplash.com/1200x628
date: 2023-10-21 13:55:24
abstracts:
---


# 什么是程序

## 源码视角 

程序其实也是状态机(🚩)[https://www.bilibili.com/video/BV12L4y1379V?t=16m28s]
函数调用就是状态机状态的变化,其实就是栈帧的变化(🚩)[https://www.bilibili.com/video/BV12L4y1379V?t=23m:49s]

## 二进制视角

只进行计算的程序是无法停止的(🚩)[https://www.bilibili.com/video/BV12L4y1379V?t=43m44s]
所以需要`syscall`来停止程序(🚩)[https://www.bilibili.com/video/BV12L4y1379V?t=46m12s]
操作系统只是操作了程序当前的程序内存的状态(🚩)[https://www.bilibili.com/video/BV12L4y1379V?t=49m47s]
