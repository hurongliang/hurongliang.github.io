---
layout: post
title: Java中的hashCode()和equals()
categories: java hashCode equals
excerpt: 这两个方法比较常见，深究下会发现很有意思。
---

这两个方法比较常见，深究下会发现很有意思。  

#  什么是hashCode？

他是根据对象的某些特性计算出的一个整数值。

#  hashCode有什么用？
一般没什么用，只有在查找时有用。比如对象放在哈希表里，存放位置是根据hascode计算出来的。这时候要查找某个对象，只要用这个对象的hashcode计算一下就能直接定位。

#  怎么得到一个对象的hashCode？

调用这个对象的hashCode方法。

hashCode方法定义在跟对象Object上，所以每个对象都有hashCode方法。

#  Object对象的hashCode是怎么算出来的？如何自己设计hashCode算法？

Object类的hashCode实现方法是：将对象的内存地址转换为整数作为hashCode。 
 
java.lang.Object中的hashCode()方法对hashCode做了如下规范：

*  在对象的生命周期内，每次调用hashCode方法返回的值必须一样。
*  不同的对象生命周期，其hashCode可以不一样。比如跑在服务器jvm里的一个固定日期的Date对象，他的hashCode和重启服务器后的hashCode可以不一样。
*  如果两个对象通过equals方法比较结果相同，则他们的hashCode方法放回的值必须是一样的。
*  如果两个对象通过equals方法比较结果不同，择他们的hashCode方法可以一样也可以不一样，没有硬性规定。但是通常建议不一样，目的是为hash table提供更好的查找性能。

一个较合理的设计是不同对象的hashCode应该是不同的。 

#  JDK中有哪些常用的类的hashCode被重写了？
*  **java.lang.Integer**  hashCode等于他的基本类型值。
*  **java.lang.String** 计算公式：s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]。所以只要代表的字符串值一样，hashCode就是一样的。

#  equals方法有什么用？

比较两个对象是否相同

#  Object的equals方法怎么实现的？自己如何设计equals方法？

Object的equals方法的实现是直接比较两个对象的引用地址（a==b）  

设计equals方法需遵循如下原则：

*  自反性：x.equals(x)应该为true
*  对称性：如果a.equals(b)为true，b.equals(a)==true
*  传递性：如果a.equals(b)为true，b.equals(c)为true，则a.equals(c)为true
*  一致性：如果a、b没有的内容没有改变，则多次调用a.equals(b)返回的结果应该相同
*  a.equals(null)应该返回false

#  什么样的情况需要重写hashCode和equals方法？有哪些要主意的？

需要根据对象的内容来确定两个对象是否相同时需要重写hashCode和equals方法。  

重写hashCode往往是为了配合重写equals，因为hashCode规范里规定，如果equals方法返回true，则两个对象的hashCode值必须一直。

#  判断两个对象需要同时调用他们的hashCode方法和equals方法吗？

一般只要调用equals方法就行了。  

如果要提高比较效率，可以先比较两个对象的hashCode值，如果相同再调用equals方法判断。这也是JDK里经常用的策略。这也是为什么要同时重写hashCode和equals方法的原因，只有遵守hashCode规范才能与JDK或这其他类库达成一致。

#  理解hashtable查询对象的内部逻辑有助于更好的理解hashCode和equals方法。

hashtable里维护着一个table数组，table数组里的每个元素是一个Entry链表，hashtable的对象实际存储在Entry链表里。  

**存储规则**：根据对象的hashCode找到table数组里对应地Entry链表（根据hashCode算出table数组里这个Entry元素的下标），将对象添加在Entry链表尾部。  

**判断是否存在的规则**：根据对象的hashCode先定位到table数组里对应地EntryEntry链表，逐个遍历Entry链表，调用数据的equals方法比较链表元素，找到比较结果为true的对象即为要找到对象。
