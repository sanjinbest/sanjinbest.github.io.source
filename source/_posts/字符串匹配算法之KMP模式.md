title: 字符串匹配算法之KMP模式
author: 木子三金
tags:
  - 算法
  - 数据结构
  - 字符串
categories:
  - 数据结构
date: 2018-12-20 18:16:00
---
这篇文章主要是介绍KMP模式匹配算法，在正式介绍KMP之前我们先看一下普通模式匹配，由普通模式匹配在进一步的推导KMP模式会更容易理解。

<!-- more -->

## 字符串的普通模式匹配

普通模式匹配的原理不进行说明了，简单来说就是两个字符串的每个字符依次进行匹配。

```
public int match(String S,String T){
    int i = 0;
    int j = 0;

    while(i < S.length() && j < T.length()){
        if(S.charAt(i) == T.charAt(j)){
            i++;
            j++;
        }else{
            i = i - j + 1;
            j = 0;
        }
    }

    if(j >= T.length()){
        return i - T.length();
    }else{
        return 0;
    }
}
```

普通模式匹配的时间复杂度最坏的情况（即T在S的末尾）时，时间复杂度为O（(m-n+1)*n）。这种算法的优点是实现简单，缺点也显而易见那就是效率较低。jdk String类内的静态方法indexOf底层使用的就是类似该种算法。



## KMP模式匹配算法推导



### 回溯推导

为了方便理解KMP模式，我们先看一下普通模式模式的流程：

栗子1：

主串：S = "abcdefgab"

子串：T = "abcdex"

![upload successful](/images/pasted-83.png)

观察下普通模式匹配算法，对于要匹配的子串T = “abcdex”来说

1. 首字母"a"与后面的串"bcdex"中的任意一个字符都不相等;
2. 串"bcde"与主串S的第2-5位相等，那么首字母"a"就不可能与主串S的第2-5位相等。

所以，第2 3 4 5 的判断都是不需要的，这是个理解KMP模式的关键。

那么问题来了，如果T串后面还含有首字母"a"的字符会怎么样呢？我们在看一个例子：

粟子2：

主串：S = "abcabcabc"

子串：T = “abcabx”

![upload successful](/images/pasted-84.png)

对于要比配的子串T = "abcabx"来说，

1. T的首字母"a"与T的第2个字符"b"、第3个字符"c"均不等，且T的第1-3位字符与主串S的第1-3相等，所以，步骤2和步骤3可以省略；
2. T的前2位串"ab"与T的第4-5串"ab"相等，且T的第4-5串"ab"与主串S的第4-5串"ab"相等，得出T的前2位串与S的第4-5也相等，步骤4 5可以省略。

最后，第6步的前两次比较也是不需要的，直接比较 主串S的第6位的"a"和子串T的第3位"c"即可，如下图：

![upload successful](/images/pasted-85.png)

对比这两个例子，可以发现普通模式主串S的游标每次都是回溯到i++的位置。而经过上面的分析后发现，这种回溯方式其实是不需要的，KMP模式就是解决了这些没有必要的回溯。

既然要避免i的回溯，那么我们就要在子串T的游标j上下工夫了。通过观察也可发现，如果T的首字母与自身后面的字符有相等的情况，那j值的变化就会不相同。

例子1内j的变化，由于首字母"a"与后面的字符都不相等，则j由6变回了1。

例子2内j的变化，由于前缀"ab"与后面的"ab"相等，因此j从6变回了3。

有没有看出什么规律？提示一下：

例子1内与前缀相同的字符个数为0，j变成了1。

例子2内与前缀相同的字符个数为2，j变成了3。

***j的变化取决也当前字符之前的串的前后缀的相似度。***

### 回溯公式

根据上一节的推导，我们可以将T串各个位置的j的变化定义为一个数组next，next的长度就是T串的长度，我们可以得到下面的函数定义（为了后面读程序方便理解，所以该函数是遵循数组下标从0开始的规范）：

![upload successful](/images/pasted-86.png)

我们来手工验证一下该函数。

T = "abcdex"

|||
| :-----: | :----: |
|    j    | 123456 |
| 模式串T | abcdex |
| next[j] | 000000 |

1. 若  j = 0, next[0] = 0;
2. 若  j = 1, 'a' <> 'b' ,next[1] = 0;
3. 若  j = 2,  子串"abc"，也没有重复的字符子串，next[2] = 0;
4. 以后同理，所以最终T串的ntxt[j]为000000。

例子就列举到这里，现在放下KMP的Java实现，其它实例大家可以使用程序进行验证。

## KMP模式匹配算法实现

```java
public void getNext(String T,Integer[] next){
    next[0] = 0;    //当j = 0时，next[0] = 0

    int i = 1;
    int j = 0;
    int k = 0;
    while (i < T.length() && j < i){
        if(T.substring(0,j).equals(T.substring(i-j,i))){
            k = j;	//若有相同子串，则记录下对应在T串内的位置，最终得出最长匹配成功的位置
        }
        j++;

        if(j >= i){
            next[i] = k;	//若一直未匹配成功，将k = 0赋值给next[i]
            j = 1;
            k = 0;
            i++;
        }
    }
}
```

我们来测试 几个字符串：

```
input: abcdex
output:000000

input: abcabx
output:000012

input: aaaaaaaab
output:001234567
```

下面是完整的KMP模式匹配算法的代码：

```java
package string;
public class KMPMatch {
    public void getNext(String T,Integer[] next){
        next[0] = 0;    //当j = 0时，next[0] = 0

        int i = 1;
        int j = 0;
        int k = 0;
        while (i < T.length() && j < i){
            if(T.substring(0,j).equals(T.substring(i-j,i))){
                k = j;	//若有相同子串，则记录下对应在T串内的位置，最终得出最长匹配成功的位置
            }
            j++;

            if(j >= i){
                next[i] = k;	//若一直未匹配成功，将k = 0赋值给next[i]
                j = 1;
                k = 0;
                i++;
            }
        }
    }

    public int match(String S,String T){
        int i = 0;
        int j = 0;

        Integer[] next = new Integer[T.length()];
        getNext(T,next);

        while(i < S.length() && j < T.length()){
            if(j == 0 || S.charAt(i) == T.charAt(j)){
                i++;
                j++;
            }else{
                j = next[j];
            }
        }

        if(j >= T.length()){
            return i - T.length();
        }else{
            return 0;
        }
    }

    public static void main(String[] args) {
        String S = "aaaabcde";
        String T = "abcd";
        System.out.println(new KMPMatch().match(S,T));
    }
}

```

对于方法getNext来说，若T的长度为m，因只涉及简单的单循环，其时间复杂度为O(m)，由于i不进行回溯，while循环的时间复杂度为O(n)。因此，整个算法的时间复杂度为O(m+n)。

***对于KMP模式来说，仅当子串与主串之前存在许多“部分匹配”的时候才体现出它的优势，否则两者差异并不明显。***

## KMP模式匹配算法的优化

看下面这个例子：

S = "aaaabcde"

T = "aaaaax"

T的next分别为001234，两个串的匹配流程如下：

![upload successful](/images/pasted-87.png)

从流程中可发现，当中的步骤2 3 4 5都是多余的判断。由于T串的第2  3 4 5位置的字符都与首字符相等，那么就可以用首位next[0]值去取代与它相等的字符的next[j]，下面就是对getNext方法进行的改良，代码如下：

```java
public void getNextVal(String T,Integer[] nextVal){
    nextVal[0] = 0;    //当j = 0时，next[0] = 0

    int i = 1;
    int j = 0;
    int k = 0;
    while (i < T.length() && j < i){
        if(T.substring(0,j).equals(T.substring(i-j,i))){
            k = j;	//若有相同子串，则记录下对应在T串内的位置，最终得出最长匹配成功的位置
        }
        j++;

        if(j >= i){
            //当前字符与前缀字符相同中，则当前字符j为nextVal[i]
            if(T.charAt(i) == T.charAt(k)){
                nextVal[i] = nextVal[k];
            }else {
                nextVal[i] = k;
            }
            j = 1;
            k = 0;
            i++;
        }
    }
}
```

*学习是一个过程，对学习到的知识进行理解、吸收、整理，并将其按自己的理解进行输出。*

参考材料：《大话数据结构》