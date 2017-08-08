---
title: java-基础之string
date: 2017-08-08 20:11:39
tags:
	- Java

categories: Java
---

## Java 基础-String

```
The class String includes methods for examining individual characters of the sequence, for comparing strings, for searching strings, for extracting substrings, and for creating a copy of a string with all characters translated to uppercase or to lowercase. Case mapping is based on the Unicode Standard version specified by the Character class.

`String`类包括了一系列字符的方法，比如：比较字符，查找字符，子字符串，创建一个全是大写或者全是小写的字符。匹配是在unicode标准版本规范在`Character` 类的基础上。

The Java language provides special support for the string concatenation operator ( + ), and for conversion of other objects to strings. String concatenation is implemented through the StringBuilder(or StringBuffer) class and its append method. String conversions are implemented through the method toString, defined by Object and inherited by all classes in Java. For additional information on string concatenation and conversion, see Gosling, Joy, and Steele, The Java Language Specification.

Java语言提供了对字符的操作符`+`做了特殊的支持,并且对其他的objects转换成string也做了相应支持。String的连接是StringBuilder(或者说StringBuffer)来实现的它的append方法的。String转换是通过实现定义在Object(所有的java方法都继承了它)的toString方法。更多的额外的字符操作和转换的特殊操作，看看gosling，joy，steele，写的《java语言规范》。

Unless otherwise noted, passing a null argument to a constructor or method in this class will cause a NullPointerException to be thrown.

除非有额外的标记，传递一个null参数给一个构造方法或者非构造方法在这个类中，会导致`NullPointerException`被抛出。

A String represents a string in the UTF-16 format in which supplementary characters are represented by surrogate pairs (see the section Unicode Character Representations in the Character class for more information). Index values refer to char code units, so a supplementary character uses two positions in a String.

一个String呈现的字符串是用UTF-16格式，不够的补位。

The String class provides methods for dealing with Unicode code points (i.e., characters), in addition to those for dealing with Unicode code units (i.e., char values).

这个String类提供方法处理Unicode代码点和unicode代码单元。

```
<!-- more-->

上面是摘抄自`String`类的介绍，翻译的地方有些不是特别理解。

### String 父类

`String`继承自Object,实现Serializable, CharSequence, Comparable<String>
重写了Object的toString，equals,hashCode方法;实现了compareTo

hashCode方法重写:
```
/**
 * Returns a hash code for this string. The hash code for a
 * {@code String} object is computed as
 * <blockquote><pre>
 * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
 * </pre></blockquote>
 * using {@code int} arithmetic, where {@code s[i]} is the
 * <i>i</i>th character of the string, {@code n} is the length of
 * the string, and {@code ^} indicates exponentiation.
 * (The hash value of the empty string is zero.)
 *
 * @return  a hash code value for this object.
 */
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

可以看到String重写了hashCode方法，计算规则中n为长度。i为坐标就是index.表明如果是空字符串那么hash值为0。hashCode中是31的幂次方。为什么是31不是其他的数，貌似有人说是2的5次方-1，也有说是可以稍微大点的质数都可以。


重写了equals方法:

```
/**
* Compares this string to the specified object.  The result is {@code
* true} if and only if the argument is not {@code null} and is a {@code
* String} object that represents the same sequence of characters as this
* object.
*
* @param  anObject
*         The object to compare this {@code String} against
*
* @return  {@code true} if the given object represents a {@code String}
*          equivalent to this string, {@code false} otherwise
*
* @see  #compareTo(String)
* @see  #equalsIgnoreCase(String)
*/
public boolean equals(Object anObject) {
  if (this == anObject) {
      return true;
  }
  if (anObject instanceof String) {
      String anotherString = (String)anObject;
      int n = value.length;
      if (n == anotherString.value.length) {
          char v1[] = value;
          char v2[] = anotherString.value;
          int i = 0;
          while (n-- != 0) {
              if (v1[i] != v2[i])
                  return false;
              i++;
          }
          return true;
      }
  }
  return false;
}
```
equals方法中先看到```this==anObject```,effective里面有说过重写对象的equal方法的几个注意点。在String中String也重写了equals方法，所以比较字符串是否相等的时候一般都要使用equals方法。

String也实现了compareTo:

```
/**
 * Compares two strings lexicographically.
 * The comparison is based on the Unicode value of each character in
 * the strings. The character sequence represented by this
 * {@code String} object is compared lexicographically to the
 * character sequence represented by the argument string. The result is
 * a negative integer if this {@code String} object
 * lexicographically precedes the argument string. The result is a
 * positive integer if this {@code String} object lexicographically
 * follows the argument string. The result is zero if the strings
 * are equal; {@code compareTo} returns {@code 0} exactly when
 * the {@link #equals(Object)} method would return {@code true}.
 * <p>
 * This is the definition of lexicographic ordering. If two strings are
 * different, then either they have different characters at some index
 * that is a valid index for both strings, or their lengths are different,
 * or both. If they have different characters at one or more index
 * positions, let <i>k</i> be the smallest such index; then the string
 * whose character at position <i>k</i> has the smaller value, as
 * determined by using the &lt; operator, lexicographically precedes the
 * other string. In this case, {@code compareTo} returns the
 * difference of the two character values at position {@code k} in
 * the two string -- that is, the value:
 * <blockquote><pre>
 * this.charAt(k)-anotherString.charAt(k)
 * </pre></blockquote>
 * If there is no index position at which they differ, then the shorter
 * string lexicographically precedes the longer string. In this case,
 * {@code compareTo} returns the difference of the lengths of the
 * strings -- that is, the value:
 * <blockquote><pre>
 * this.length()-anotherString.length()
 * </pre></blockquote>
 *
 * @param   anotherString   the {@code String} to be compared.
 * @return  the value {@code 0} if the argument string is equal to
 *          this string; a value less than {@code 0} if this string
 *          is lexicographically less than the string argument; and a
 *          value greater than {@code 0} if this string is
 *          lexicographically greater than the string argument.
 */
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}

```

可以看到hashCode，equals，compareTo 中都有提到value？那么value是什么？稍后看。现在回到compareTo,它是Comparable接口中唯一一个方法，约定俗成的方式是如果大返回1，如果等于返回0，如果小于返回-1。实现Comparable接口还有一个好处可以在集合中排序。

其中我们讲到`value`，看到String类中value的定义是:`private final char value[];`这说明我们的String类其实底层的实现也是char,这也是为啥String不属于基础类型的一个原因吧,但是语言开发者考虑到String用的比较多所以也做了很多特殊的处理.

### final ,可变不可变

可以在源码中看到String为定义为final,并且里面的值也是大多定义为final的.定义为final的类是不能被*继承*的.另外定义为final也就意味着值是不能改变的.既然不能改变值,那么String类其实可以说是安全的. 想想看为啥value不是public而是private,其实这是必须的,因为如果是public那么就可以改变里面的值,记住final是不能改变引用指向的对象,但是对象里面的值还是可以改变的:
```
public class StringMe
{
    public final static char[] aChar = new char[]{'1','2','3'};
}


public class StringTest
{
    public static void main(String[] args)
    {
        System.out.println(StringMe.aChar);
        StringMe.aChar[0] = 34;
        System.out.println(StringMe.aChar); //改变了achar[0]

        //StringMe.aChar = new char[]{'3'};//编译报错,想改变aChar指向的对象
    }
}
```

### String 特殊点

常常会见到下面这种情况,面试中常有:
```
//        String a = "a";
//        String a_new = new String("a");
//
//        System.out.println(a == a_new);//false
//        System.out.println(a.equals(a_new));//true
//
//        String abc = "ab" + "c";
//
//        String abc_new = new String("abc");
//        System.out.println(abc == abc_new);//fasle
//        System.out.println(abc.equals(abc_new));//true
//
//        abc_new = "abc";
//        System.out.println(abc == abc_new);//true
//        System.out.println(abc.equals(abc_new));//true
//
        final String ab = "ab";
        String abcd = ab + "cd";
        String abcd_origin = "abcd";
        System.out.println((abcd == abcd_origin)); // true

        final String abcde = "abcde";
        String abcdef = abcde + "f";
        String abcdef_origin = new String("abcdef");
        System.out.println(abcdef == abcdef_origin); //false


```
![String类反编译图片](/images/encode-java-stringtest.png)

```
String ab = "ab";
String cd = "cd";
String abcd1 = ab + cd;
String abcd2 = "ab" + cd;
String abcd3 = ab + "cd";
String abcd5 = "abcd";
String abcd6 = "ab"+"cd";
System.out.println((abcd1 == abcd2)); // false
System.out.println((abcd2 == abcd3)); // false
System.out.println((abcd3 == abcd5)); // false
System.out.println((abcd5 == abcd6)); // true
System.out.println((abcd1 == abcd5)); // false

```

![String类反编译图片2](/images/encode-java-stringtest2.png)

图中可以看出其实String底层的相加也是用的StringBuild.


....未完待续....
// 相似点，同类
// 常用方法
// hashCode，equals
// 是否同步，线程安全
// 不同步是否有同步的
