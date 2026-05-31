---
title: 
date: 2026-05-26T15:53:00+08:00
tags: [Go]
series: [Learning Go]
featured: true
description: "我在小红书实习的数据库上游模块是go写的，整理一下基础语法"
draft: true
---
> [a tour of go](https://go.p2hp.com/tour/welcome/1) 中文版入门

## 基础概念

### 包 package
CPP中为了实现不同模块的分离使用文件(.h/.cpp)命令空间(namespace)保证，Go，Java,Python等更modern的语言使用目录包package避免繁琐的约定。

可执行程序入口：package main + func main
其他package 名都被编译成了库

导入路径vs 包名
import

可见性靠首字母大小写来保证，大写包外可见，小写包内私有（Go没有C++的public/private/protected的关键字实现权限控制）

包内文件互相可见
<!--more-->

### 函数
函数参数数量不限(>=0)，变量名在前，类型在后，连续参数共享同一类型可以共用；

允许多值返回，返回值可以定义名字，这样的好处有：1.裸返回，不用写变量名了；2.签名，返回语义更清晰3.配合defer做错误处理

### var


### for循环
Go中没有while循环，所有循环都是用for,而且与cpp不通，不需要要括号括起来，但是大括号不能省略

### if条件语句
条件语句if 和for类似，不用括号括起来条件语句，大括号不能省略
但是if的条件语句前可以执行短语句，短语句只在条件分支if/else内生效
```go
	if v := math.Pow(x, n); v < lim {
		return v
	}
```

### switch语句
类似cpp,只是go默认为每个case都提供了break



### defer语句
defer 语句会将函数推迟到外层函数返回之后执行

推迟调用的函数其参数会立即求值，但直到外层函数返回前，该函数都不会被调用。

## 指针
指针保存值的内存地址，操作符*,&操作符生成指向操作数的指针
```go
var p *int
```
注意，go没有指针运算

## struct结构体
struct是字段集合，使用.访问，指针访问时无需显式解引用

## 数组Arrays
类型 [n]T表示拥有n个T类型的值的数组
```go
var a [10]int
```
数组长度具有固定大小，但是切片可以为数组元素提供动态灵活视角。
```go
a[low : high]
```
更改切片元素会修改底层数组的相应元素，共享的切片会看到更改，切片上下限可以使用默认值





