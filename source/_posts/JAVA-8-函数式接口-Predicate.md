title: JAVA 8 函数式接口--Predicate
author: 木子三金
tags:
  - JDK8
  - Predicate
  - 函数式接口
categories:
  - JAVA
date: 2018-08-30 19:03:00
---
从JDK8开始java支持函数式编程，JDK也提供了几个常用的函数式接口，这篇主要介绍Predicate接口。
文本介绍的顺序依次为：

- 源码介绍
- 使用示例
- JDK内Predicate的使用举例
- 扩展类介绍

<!-- more -->

### 源码介绍
```
package java.util.function;

import java.util.Objects;

/**
 * 提供了对输入的参数进行断定并返回boolean类型能力的函数式接口
 *
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * 基于输入的参数进行断定，并返回boolean类型
     *
     */
    boolean test(T t);

    /**
     * 逻辑“与”判断，接收一个Predicate类型参数。
     * 优先对自身逻辑进行断定，若断定为true则进行参数other断定；否则不对other断定
     * 若断定期间发生异常，停止所有断定并将异常传递给调用者
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 对断定的结果取“非”值
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * 逻辑“或”判断，接收一个Predicate类型参数。
     * 优先对自身逻辑进行断定，若断定为true则不对参数other断定；否则对other断定
     * 若断定期间发生异常，停止所有断定并将异常传递给调用者
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * 可设置一个参数，并对后续所有调用test传入数与该参数进行equals比较
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```
下面来看一下具体的使用方法。

### 使用示例
```
package jdk8.function;

import java.util.function.Predicate;

public class PredicateDemo {

    public static void main(String[] args) {
        PredicateDemo predicateDemo = new PredicateDemo();
        predicateDemo.simpleDemo();
    }

    public void simpleDemo(){
        Predicate<Integer> predicate = i -> i > 10;
        Predicate<Integer> other = i -> i < 20;

        Integer param = 10;
        String str = "demo";

        System.out.println(param + ">10：" + predicate.test(param));
        System.out.println(param + " > 10 and " + param + " < 20：" + predicate.and(other).test(111));
        System.out.println(param + " > 10 or " + param + " < 20：" + predicate.or(other).test(30));
        System.out.println("!("+param+" > 10)：" + predicate.negate().test(0));
        System.out.println("demo.equal("+str+")：" + Predicate.isEqual("demo").test("demo"));
    }

}

```
下面是输出结果：
```
10>10：false
10 > 10 and 10 < 20：false
10 > 10 or 10 < 20：true
!(10 > 10)：true
demo.equal(demo)：true
```
个人觉得这个函数式接口是一个非常好用的工具，可以实现一些校验器、过滤器的功能。
### JDK内Predicate的使用举例
Collection接口提供了一个默认的方法removeIf，使用者可以根据自定义的filter对集合内的元素进行移除。代码如下:
```
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

### Predicate扩展接口
|||
| - |
|类名|描述|
|BiPredicate|提供了接收2个参数的断定功能|
|DoublePredicate|提供了接收double类型参数的断定功能|
|IntPredicate|提供了接收int数的断定功能|
|LongPredicate|提供了接收long的断定功能|
