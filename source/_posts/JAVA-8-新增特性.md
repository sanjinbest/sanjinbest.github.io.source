title: JAVA 8 新增特性之Lambda表达式+方法引用
author: 木子三金
tags:
  - java8
  - lambda
  - 默认方法
  - 批量处理
categories:
  - JAVA
date: 2018-06-23 16:39:00
---
JDK8 已经发布了有几年了，但平时很少会用到新增的特性。最近特意抽出了一些时间仔细的学习一下新特性，本文主要是关于Lambda表达式的内容。

<!-- more -->


## 闭包
在正式介绍Lambda表达式前先要理解一下“闭包”的概念，和java对“闭包”的实现方式。
### 闭包是个啥？

>闭包就是能够读取其他函数内部变量的函数。例如在javascript中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成“定义在一个函数内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。--摘自百度百科

读起来不是特别好理解吧，文章《[深入理解Java闭包概念](https://www.cnblogs.com/ssp2110/p/3797666.html)》里有一段很好的解释：
>闭包能够将一个***方法作为一个变量去存储***，这个方法可以访问所在类的自由变量。
注：自由变量，除了局部变量的其它变量。

说白了，就是一个变量存储的不在是实例对象，而是存储了一个可执行的方法。

### JAVA里闭包是怎么实现的？
我们先一步一步来解决这个问题：
1. 怎么用变量去存储方法？
		java里能够保存方法的变量指的就是普通的实例对象。
2. 那么如何让这个实例对象访问到该对象所在类的自由变量？
		答案就是内部类，内部类能够访问到外部类的所有属性及方法，同时能够隐藏具体实现。
3. 怎么将闭包传递到外部使用呢？
		可以让内部类实现通用接口，然后将内部类对象向上转型为接口类型。
        这个接口类型对象就可以理解为保存了方法的变量。
		
所以，在java里闭包就是通过***内部类+接口***来实现的。参考文章里有一个很好的例子，可以方便大家理解。
下面正式开始介绍JDK8的新特性。

## Lambda
jdk8新增的Lambda表达式功能是一个非常酷也非常好用的东西。

它大概是这个样子，比如我们要实现多线程的功能，以前我们可能会这样写：
```
public void threadDemo(){
	new Thread(new Runnable() {
		@Override
		public void run() {
			System.out.println("I am a Runner!");
		}
	}).start();
}
```
当我们有了Lambda之后，一行搞定
```
public void threadDemo(){
	new Thread(() -> System.out.println("I am a Runner!")).start();
}
```
SO COOL !

### Lambda语法

	(参数) -> 单行语句/表达式 或者 （参数）-> {代码块;}
    
lambda语法包括3个部分
1. 括号+参数，代表了***函数式接口***里的方法，参数就是接口方法的参数。
2. 操作符 ->
3. 方法体，方法体可以是表达式也可以是一个代码块。代码块必须使用{}。如果函数式接口的方法有返回值，需要进行return。

*上面的例子Runnable就是一个函数式接口，只包含了一个无参方法run()，返回值是void，所以lambda左边没有参数只有一对括号，右边也没有return语句，而且只是打印了一个字符串。*

### 函数式接口
在java中lambda表达式无法单独的出现，它需要一个函数式接口来承载。lambda的表达式方法体就是对函数接口的具体实现。

函数式接口（functional interface），一个接口如果只包含一个方法，那这个接口就叫作函数式接口，比如上例中的Runnable接口，就是一个函数式接口。

jdk8提供了一个注解@FunctionalInterface，这个注解的作用就是声明该接口是函数式接口，当然这个注解不是必须的。不过建议对函数式接口都加上该注解，方便其它开发者直观的知道该接口为函数式接口。

Lambda的用法非常简单也很容易理解，所以这里不在多说。

### 方法引用
lambda表达式箭头（->）右边是要实现的执行代码。但有时候，右边实现的执行代码已经有类实现了我们想要的功能，这时就可以使用方法引用来直接调用现有类的功能方法。

看一个例子，jdk8提供了一个函数式接口**Consumer**，其中有一个方法**accept**接收一个参数，下面我们实现一个打印字符串的功能。

```
Consumer<String> consumer = str -> System.out.println(str);
consumer.accept("Hello World!");
```
这里使用到的System.out.println就是一个已经实现了打印字符串功能的类，所以可以直接调用，像下边这样
```
Consumer<String> consumer - str -> System.out::println;
consumer.accept("Hello World!");
```
这种调用方式就是方法引用。

***方法引用使用操作符“::”实现。左边是类名或实例名，右边是方法名或者是“new”（构造器引用）***

方法引用共有4种类型：
- 静态方法引用
```
//lambda写法
Function<Long,Long> function = x -> Math.abs(x);
function.apply(-3L);
//静态方法引用
Function<Long,Long> function = Math::abs;
function.apply(-3L);
```
abs是Math类的一个静态方法，Function中的唯一抽象方法apply接收的参数列表与abs方法相同。

- 对象引用的方法引用
```
//lambda写法
Consumer<String> consumer = str -> System.out.println(str);
consumer.accept("Hello World!");
//对象引用的方法引用
Consumer<String> consumer - str -> System.out::println;
consumer.accept("Hello World!");
```
这里的System.out其实就是PrintStream类型的对象引用,而println是一个实例的方法。

- 类的方法引用
```
//lambda写法
BiPredicate<String, String> b = (x,y) -> x.equals(y);
b.test("I'm King!","I' King too!");
//类的方法引用
BiPredicate<String, String> b = String:equals;
b.test("I'm King!","I' King too!");
```
如果lambda表达式的参数列表中，第一个参数是实例方法的调用者，第二个参数是实例方法的参数的情况下，才可以这样调用。上面的例子第一个String是调用者，第二个String是参数，大概就是这样的意思: "I'm King!".equals("I' King too!")

- 构造方法引用
```
//lambda写法
Function<Integer, StringBuffer> fun = n -> new StringBuffer(n); 
StringBuffer buffer = fun.apply(10);
//构造方法引用
Function<Integer, StringBuffer> fun = StringBuffer::new;
StringBuffer buffer = fun.apply(10);
```


总结，本文主要介绍了jdk8的lambda表达式和方法引用，同时加了一些闭包相关的内容。关于文章里的例子个人感觉没有很好的体现新特性的强大之处。由于本人目前能力有限，同时还未在工作中尝试使用新特性，固还无法找到更好的例子，待后续更新吧。

参考文章：
- [深入理解Java闭包概念](https://www.cnblogs.com/ssp2110/p/3797666.html)
- [OSC闲人的相关文章](https://my.oschina.net/benhaile?tab=newest&catalogId=0)