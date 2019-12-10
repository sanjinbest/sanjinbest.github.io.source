title: JAVA 8 函数式接口--Supplier
author: 木子三金
tags: []
categories:
  - JAVA
date: 2018-08-31 13:59:00
---
从JDK8开始java支持函数式编程，JDK也提供了几个常用的函数式接口，这篇主要介绍Supplier接口。
文本介绍的顺序依次为：

- 源码介绍
- 使用示例
- 扩展类介绍

<!-- more -->

### 源码介绍
```
package java.util.function;

/**
 * 供应商函数，每次调用get()方法返回一个T类型对象
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```
下面来看一下具体的使用方法。

### 使用示例
```
package jdk8.function;

import java.util.UUID;
import java.util.function.Supplier;

public class SupplierDemo {

    public static void main(String[] args) {
        Supplier<String> uuid = () -> UUID.randomUUID().toString();
        Supplier<String> timestamp = () -> System.currentTimeMillis() + "";

        SupplierDemo supplierDemo = new SupplierDemo();
        System.out.println("uuid-sessionId : " + supplierDemo.sessionId(uuid));
        System.out.println("timestamp-sessionId : " + supplierDemo.sessionId(timestamp));
    }

    /**
     * 可自义的session_id生成器
     * @param supplier
     * @return
     */
    public String sessionId(Supplier<String> supplier){
        return supplier.get();
    }
}

```
下面是输出结果：
```
uuid-sessionId : 9893eca1-a3c0-49a0-abb2-f4e8dbb0aa38
timestamp-sessionId : 1535698167233
```

### Predicate扩展接口
|||
| - |
|类名|描述|
|BooleanSupplier|提供了生产boolean型返回值功能|
|DoubleSupplier|提供了生产double型返回值功能|
|IntSupplier|提供了生产int型返回值功能|
|LongSupplier|提供了生产long型返回值功能|