title: 关于String内的indexOf方法的一些疑问
author: 木子三金
tags:
  - JAVA
  - ''
  - 源码
categories:
  - java
date: 2018-12-14 17:48:00
---
今天浏览了一下java里的String类，发现一个静态方法有点意思，就是我们常用的indexOf(String str)的底层实现，先看下代码调用链。

<!-- more -->
```
public int indexOf(String str) {
    return indexOf(str, 0);
}
    
public int indexOf(String str, int fromIndex) {
    return indexOf(value, 0, value.length,
            str.value, 0, str.value.length, fromIndex);
}

static int indexOf(char[] source, int sourceOffset, int sourceCount,
        String target, int fromIndex) {
    return indexOf(source, sourceOffset, sourceCount,
                   target.value, 0, target.value.length,
                   fromIndex);
}

/**
 * Code shared by String and StringBuffer to do searches. The
 * source is the character array being searched, and the target
 * is the string being searched for.
 *
 * @param   source       the characters being searched.
 * @param   sourceOffset offset of the source string.
 * @param   sourceCount  count of the source string.
 * @param   target       the characters being searched for.
 * @param   targetOffset offset of the target string.
 * @param   targetCount  count of the target string.
 * @param   fromIndex    the index to begin searching from.
 */
static int indexOf(char[] source, int sourceOffset, int sourceCount,
        char[] target, int targetOffset, int targetCount,
        int fromIndex) {
    if (fromIndex >= sourceCount) {
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        fromIndex = 0;
    }
    if (targetCount == 0) {
        return fromIndex;
    }

    char first = target[targetOffset];
    int max = sourceOffset + (sourceCount - targetCount);

    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }

        /* Found first character, now look at the rest of v2 */
        if (i <= max) {
            int j = i + 1;
            int end = j + targetCount - 1;
            for (int k = targetOffset + 1; j < end && source[j]
                    == target[k]; j++, k++);

            if (j == end) {
                /* Found whole string. */
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```

底层的字符串匹配的逻辑比较简单，就是普通的匹配模式：
1. 查找首字符，匹配target的第一个字符在source内的位置，若查找到max位置还找到，则返回-1；
2. 若在source匹配到了target的第一个字符，那么在依次比较srouce和target后面的字符，一直到target的末尾；
3. 如果target后面的字符与source都已经匹配，则返回在source上匹配到的第一个字符的**相对**下标，否则返回-1。

但是仔细读代码会发现一个问题，就是这里
```
int max = sourceOffset + (sourceCount - targetCount);
```
max的计算方式，max的作用是计算出最大的首字符匹配次数，取值范围应该是"max <= sourceCount"。
所以target字符串的长度是可以不用匹配的，故“sourceCount - targetCount”是没问题的。
关键的地方是这里加上了sourceOffset，sourceOffset是source字符串的起始匹配偏移量，即从source的哪个字符开始匹配。
所以，根据代码里的max计算方式，最终计算出来的max值是会有可能大于sourceCount。
看下测试代码：
```
package string;

/**
 * string test
 */
public class StringTest {

    static int indexOf(char[] source, int sourceOffset, int sourceCount,
                       char[] target, int targetOffset, int targetCount,
                       int fromIndex) {
        if (fromIndex >= sourceCount) {
            return (targetCount == 0 ? sourceCount : -1);
        }
        if (fromIndex < 0) {
            fromIndex = 0;
        }
        if (targetCount == 0) {
            return fromIndex;
        }

        char first = target[targetOffset];
        int max = sourceOffset + (sourceCount - targetCount);

        for (int i = sourceOffset + fromIndex; i <= max; i++) {
            /* Look for first character. */
            if (source[i] != first) {
                while (++i <= max && source[i] != first);
            }

            /* Found first character, now look at the rest of v2 */
            if (i <= max) {
                int j = i + 1;
                int end = j + targetCount - 1;
                for (int k = targetOffset + 1; j < end && source[j]
                        == target[k]; j++, k++);

                if (j == end) {
                    /* Found whole string. */
                    return i - sourceOffset;
                }
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        String source = "abcdefghigklmn";
        String target = "n";
        int sourceOffset = 5;
        int targetOffset = 0;

        int index = indexOf(source.toCharArray(), sourceOffset, source.length(), target.toCharArray(), targetOffset, target.length(), 0);
        System.out.println(index);
    }
}
```
如果target在source内可以匹配到返回正确结果8（结果8是相对于sourceOffset的结果，如果转换成source内的位置则是13）。
但是如果target在source内匹配不到，则会抛出java.lang.ArrayIndexOutOfBoundsException异常，如下：
```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 14
	at string.StringTest.indexOf(StringTest.java:27)
	at string.StringTest.main(StringTest.java:52)
```
可见报出越界的下标是14,这就是由于max = sourceOffset + (sourceCount - targetCount)引起，计算出的max值为：17。

所以，个人认为max计算这里是个潜在的BUG，应该改为 int max = sourceCount - targetCount;

不过这个方法是一个非public方法，只在String内部调用，同时也跟踪了所有对该方法的调用链，都是传入的默认0,在使用时不会出现数组越界问题。
不知这是开发者故意为之，还是其它我未知用意，欢迎大家交流讨论！！！