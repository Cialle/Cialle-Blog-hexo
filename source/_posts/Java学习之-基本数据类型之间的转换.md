---
title: Java学习之 基本数据类型的转换
date: 2022-06-01 13:14:00
author: Cialle
categories:
- Java学习
tags:
- 类型转换
- 数据类型
- java学习
---

# 概述

本章主要了解Java中，基本数据类型之间的转换：自动(隐式)类型转换 与 强制(显式)类型转换



# Java基本数据类型转换

这里讨论只是7种基本数据类型变量间的运算，boolean不参与运算

## 自动(隐式)类型转换

当容量小的数据类型的变量与容量大的数据类型的变量做运算时，结果自动提升为容量大的数据类型。

在Java中，整数类型（byte/short/int/long）中，对于未声明数据类型的整形，其默认类型为int型。

（**byte->char/short） -> int -> long -> float -> double**









## 强制(显式)类型转换
