title: JAVA 8 函数式接口--Consumer
author: 木子三金
tags: []
categories:
  - JAVA
date: 2018-08-06 15:33:00
---
从JDK8开始java支持函数式编程，JDK也提供了几个常用的函数式接口，这篇主要介绍Consumer接口。
文本介绍的顺序依次为：
- 源码介绍
- 使用实例
- jdk内对Consumer的典型使用
- 扩展类介绍

<!-- more -->

### 源码介绍
```
package java.util.function;

import java.util.Objects;

/**
 *
 * 表示“接受一个参数输入且没有任何返回值的操作“。不同于其它的函数式接口，Consumer期望通过方法的实现来执行具体的操作。
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * 可实现方法，接受一个参数且没有返回值
     */
    void accept(T t);

    /**
     * 
     * 默认方法，提供链式调用方式执行。执行流程：先执行本身的accept在执行传入参数after.accept方法。
     * 该方法会抛出NullPointerException异常。
     * 如果在执行调用链时出现异常，会将异常传递给调用链功能的调用者，且发生异常后的after将不会在调用。
     * 
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```
源码只有2个方法，也比较容易理解，我们下面来看一下具体的使用方法。

### 使用实例

```
package jdk8;

import java.util.function.Consumer;

public class ConsumerTest {

    public static void main(String[] args) {
        testConsumer();
        testAndThen();
    }

    /**
     * 一个简单的平方计算
     */
    public static void testConsumer(){
        Consumer<Integer> square = x -> System.out.println("print square : " + x * x);
        square.accept(2);
    }

    /**
     * 定义3个Consumer并按顺序进行调用andThen方法，其中consumer2抛出NullPointerException。
     */
    public static void testAndThen(){
        Consumer<Integer> consumer1 = x -> System.out.println("first x : " + x);
        Consumer<Integer> consumer2 = x -> {
            System.out.println("second x : " + x);
            throw new NullPointerException("throw exception test");
        };
        Consumer<Integer> consumer3 = x -> System.out.println("third x : " + x);

        consumer1.andThen(consumer2).andThen(consumer3).accept(1);
    }
}

```
下面是执行结果：
```
print square : 4

first x : 1
second x : 1
Exception in thread "main" java.lang.NullPointerException: throw exception test
	at jdk8.ConsumerTest.lambda$testAndThen$2(ConsumerTest.java:27)
	at java.util.function.Consumer.lambda$andThen$0(Consumer.java:65)
	at java.util.function.Consumer.lambda$andThen$0(Consumer.java:65)
	at jdk8.ConsumerTest.testAndThen(ConsumerTest.java:31)
	at jdk8.ConsumerTest.main(ConsumerTest.java:9)

Process finished with exit code 1
```
在testAndThen()方法的执行结果可以看到打印的顺序和出现异常的情况**（third x : 1 并没有输出）**
上面只是一个简单的使用，主要为了说明使用方式。对于Consumer的工作实践目前还未使用，并没有好的例子。

### jdk内对Consumer的典型使用
在jdk内对Consumer的典型使用非foreach莫属了(在 **java.lang.Iterable**内)，下面是源码：
```
/**
     * Performs the given action for each element of the {@code Iterable}
     * until all elements have been processed or the action throws an
     * exception.  Unless otherwise specified by the implementing class,
     * actions are performed in the order of iteration (if an iteration order
     * is specified).  Exceptions thrown by the action are relayed to the
     * caller.
     *
     * @implSpec
     * <p>The default implementation behaves as if:
     * <pre>{@code
     *     for (T t : this)
     *         action.accept(t);
     * }</pre>
     *
     * @param action The action to be performed for each element
     * @throws NullPointerException if the specified action is null
     * @since 1.8
     */
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```
方法接收一个Consumer对象，对this集合执行循环相同的操作。

*TODO：除Iterable外还有很多地方使用到了Consumer，待后续使用到在添加。*

### 扩展类介绍
Consumer的accept只接受一个参数，那如果要是想使用多个参数要怎么办？jdk8又提供了一个BiConsumer接口类，该类与Consumer的区别是可以接受2个参数。

jdk8还对Consumer和BiConsumer各提供了3个常用的相关接口类，见下表：

|||
| - |
|类名|描述|
|IntConsumer|接受单个int型参数的Consumer操作|
|DoubleConsumer|接受单个double型参数的Consumer操作|
|LongConsumer|接受单个long型参数的Consumer操作|
|ObjIntConsumer|接受2个int型参数的Consumer操作，不支持andThen方法|
|ObjDoubleConsumer|接受2个double型参数的Consumer操作，不支持andThen方法|
|ObjLongConsumer|接受2个long型参数的Consumer操作，不支持andThen方法|

***以上均为个人学习总结，可能有理解不当的地方，欢迎交流！***