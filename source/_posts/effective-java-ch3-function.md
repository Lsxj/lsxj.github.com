---
title: 对于所有对象都通用的方法--《Effective Java》
date: 2017-02-07 20:01:53
categories: [后台开发, JAVA开发]
tags: [JAVA, Effective Java]
---
## 前言
本文为阅读《Effective Java》第三章阅读笔记。

---

## 覆盖equals时请遵守通用约定
### 什么时候应该覆盖`Object.equals`？
-如果类具有自己的“逻辑相等”，且超类没有覆盖equals来实现我们想要的功能。

### 覆盖equals方法的约定
* 自反性  
对于任何非null的引用值x， `x.equals(x)`必须返回true
* 对称性  
对于任何非null的引用值x和y，当且仅当`y.equals(x)`返回true时，`x.equals(y)`必须返回true。
* 传递性  
对于任何非null的引用值x、y和z，如果`x.equals(y)`返回true，并且`y.equals(z)`也true，则`x.equals(z)`必须true。
* 一致性  
对于任何非null的引用值x和y，只要比较操作在对象中所用的信息没有被修改，多次调用`x.equals(y)`结果都会一致。
* 对于任何非null的引用值x，`x.equals(null) = false`

### 实现高质量equals方法诀窍
1. 使用 == 操作符检查“参数是否为这个对象的引用”。如果是则返回true。
2. 使用 instanceof 操作符检查“参数是否为正确类型”。如果不是则返回false。
3. 把参数转换为正确类型
4. 对于该类中每个“关键”域，检查参数中的域是否与该对象中对应的域相匹配。
5. 编写完成后见检查是否对称、传递、一致
6. 覆盖equals时总要覆盖hashCode
7. 不啊哟将equals声明中的Object对象替换为其他类型

---

## 覆盖equals时总要覆盖hashCode
Object规范中： 
* 在应用程序执行期间，只要对象的equals方法的比较操作所用到信息没有被修改，这同一个对象调用多次，hashCode方法必须返回一致的答案。在同一个应用程序的多次执行过程中，每次执行返回的整数可以不一致。
* 如果两个对象根据equals方法比较是相等的，那么调用两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果。
* 如果两个对象根据equals方法比较是不等，那么调用两个对象中任意一个对象的hashCode方法不一定要产生不同的整数结果。但是给不相等的对象产生截然不同的整数结果，有可能提高散列表性能。

---

## 始终要覆盖toString
`java.lang.Object`的toString实现返回的字符串是 类名称+ @ + 散列码的无符号十六机制表示法。
当调用`println`、`printf`、`+`以及`assert`或被调试器打印出来，toString方法会被自动调用。
 

---

## 谨慎地覆盖clone
实现对象拷贝的方法有以下：
* 一个实现了`Cloneable`接口的类，用一个公有方法覆盖`lone。先调用`super.clone`然后修正任何需要修正的域。
* 提供一个拷贝构造器或拷贝工厂。

第二种方法更具优势。
拷贝构造器:

    public Yum(Yum yum);

拷贝工厂是类似于拷贝构造器的静态工厂：

    public static Yum newInstance(Yum yum);

---

## 考虑实现Comparable接口
`compareTo`方法是`Comparable`接口中唯一的方法。不但允许进行简单的等同性比较，而且允许执行顺序比较。
一旦类实现Comparable接口，就可以和许多泛型算法和依赖于该接口的集合实现进行协作。