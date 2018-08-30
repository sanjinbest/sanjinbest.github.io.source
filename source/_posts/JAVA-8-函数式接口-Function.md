title: JAVA 8 函数式接口--Function
author: 木子三金
tags:
  - JDK8
  - 函数式接口
  - Function
categories:
  - JAVA
date: 2018-08-30 14:15:00
---
从JDK8开始java支持函数式编程，JDK也提供了几个常用的函数式接口，这篇主要介绍Function接口。
文本介绍的顺序依次为：

- 源码介绍
- 使用示例
- JDK内Function的使用举例
- 扩展类介绍

<!-- more -->

### 源码介绍
```
/**
 * 
 * 表示“接受一个参数输入并产生一个输出值的操作“。 
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     * 将此函数应用于给定的参数。即自定义参数类型和返回值类型的函数。
     * 
     */
    R apply(T t);

    /**
     * 
     * 返回一个组装后的函数，这个函数接收的参数为一个Function类型对象（before）。
     * 提供了一种调用链方式的函数，该函数会先执行参数before的apply方法，并将before返回值做为自身apply方法的输入参数。
     * 在执行期间函数调用链中的任一函数发生异常，都会传递给调用链函数的调用方。
     *
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     *
     * 与compose方式提供类似的功能。与其不同的是andThen方法会优先执行自身的apply方法，并将其返回值做为after的输入参数。
     * 在执行期间函数调用链中的任一函数发生异常，都会传递给调用链函数的调用方。
     *
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 
     * 始终返回输入参数的方法
     *
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```
下面来看一下具体的使用方法。

### 使用示例
```
package jdk8.function;

import java.util.function.Function;

public class FunctionDemo {

    public static void main(String[] args) {
        FunctionDemo demo = new FunctionDemo();
        demo.simpleDemo();
        demo.composeAndThen();
    }

    public void simpleDemo(){
        Function<Integer,Integer> func = a -> a+2;
        System.out.println(func.apply(1));
    }

    public void composeAndThen(){
        Function<String,String> befor = name -> "befor 输入参数[" + name + "]";
        Function<String,String> after = name -> "after 输入参数[" + name + "]";

        System.out.println("compose:" + after.compose(befor).apply("demo"));
        System.out.println("andThen:" + befor.andThen(after).apply("demo"));
    }
}

```
这是一个Function接口的简单例子，下面是输出结果：
```
3
compose:after 输入参数[befor 输入参数[demo]]
andThen:after 输入参数[befor 输入参数[demo]]
```
注意最后2行输出，虽然输出结果一样，但是调用的方法和顺序是不相同的。

### JDK内Function的使用举例
JDK8的Map接口提供了一个新方法computeIfAbsent，该方法接收2个参数key和Function。方法会判断传入的key所对应的Value是否在map内，如果Value不存在或为null，则调用Function接口生成一个Value并与key关联存入map中。

```
default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }
```

### Function扩展接口
|||
| - |
|类名|描述|
|BiFunction|提供了接收2个参数的Function功能|
|ToDoubleBiFunction|提供了接收个参数并返回double类型的Function功能|
|ToIntBiFunction|提供了接收2个参数并返回int类型的Function功能|
|ToLongBiFunction|提供了接收2个参数并返回long类型的Function功能|
|IntFunction|提供了接收一个int类型参数的Function功能|
|DoubleFunction|提供了接收一个double类型参数的Function功能|
|LongFunction|提供了接收一个long类型参数的Function功能|
|ToDoubleFunction|提供了接收一个参数并返回double类型的Function功能|
|ToIntFunction|提供了接收一个参数并返回int类型的Function功能|
|ToLongFunction|提供了接收一个参数并返回long类型的Function功能|
|DoubleToIntFunction|提供了接收一个double类型参数并返回一个Int类型的Function功能|
|DoubleToLongFunction|提供了接收一个double类型参数并返回一个long类型的Function功能|
|IntToDoubleFunction|提供了接收一个int类型参数并返回double类型的Function功能|
|IntToLongFunction|提供了接收一个long类型参数并返回int类型的Function功能|
|LongToDoubleFunction|提供了接收一个long类型参数并返回double类型的Function功能|
|LongToIntFunction|提供了接收一个long类型参数并报返回int类型的Function功能|