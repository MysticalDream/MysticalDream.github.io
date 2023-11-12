---
title: "String, StringBuffer, StringBuilder区别"
date:  2023-04-27T23:34:28+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
# categories:
# - category 1
# - category 2

tags:
- java
- java面经
- 面经

---

# String, StringBuffer, StringBuilder区别

>String, StringBuffer, StringBuilder是Java中用于处理字符串的三个类

## String 
>是一个不可变的类，即创建后不能被修改。每当对String类型的变量进行操作时，都会创建一个新的String对象，这会导致内存使用效率低下。因此，如果需要对字符串进行频繁的修改操作，使用 String 类会导致性能问题。

>String类内部保存这一个final修饰的value数组
>```java
> private final char value[];
>```

## StringBuffer
>是一个线程安全的可变类，适用于在多个线程之间共享的情况。每当对StringBuffer类型的变量进行操作时，不会创建新的String对象，而是在原有对象的基础上进行修改。

>其父类 `AbstractStringBuilder` 的value数组不是final类型
>```java
>char[] value;
>```

>该类的方法都加上了`synchronized`保证线程安全

## StringBuilder
>是可变类，与 StringBuffer 类似，但是 StringBuilder 不保证线程安全。因此，当不需要考虑多线程访问的情况下，建议使用 StringBuilder 而不是 StringBuffer，因为 StringBuilder 的性能比 StringBuffer 更高。

## 总结
>如果需要对字符串进行频繁的修改操作并且只有单个线程访问该对象，那么就应该使用 StringBuilder；如果需要对字符串进行频繁的修改操作并且多个线程可能访问该对象，那么就应该使用 StringBuffer。而如果不需要对字符串进行修改操作，那么就应该使用 String。