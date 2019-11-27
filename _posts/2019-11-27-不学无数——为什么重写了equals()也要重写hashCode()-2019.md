---
layout:     post                    # 使用的布局（不需要改）
title:      为什么重写了equals()也要重写hashCode()        # 标题
subtitle:   为什么重写了equals()也要重写hashCode()        #副标题
date:       2019-11-27          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-mma-4.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Spring
    - 事务
---

# 为什么重写了equals()也要重写hashCode()

> 笔者文笔功力尚浅，如有不妥，请慷慨指出，必定感激不尽

在`Effective Java`中第九条规定在覆盖`equals()`方法时总要覆盖`hashCode()`方法。这是为什么呢？接下来我们就介绍一下这两个方法。

Java中的`equals()`方法和`hashCode()`方法都是在`Object`类中的方法，而在Java中所有的类都是`Obejct`类的子类，所以Java中所有的方法都会有这两个方法的默认实现。

## equals方法

`Object`类中的`equals()`方法定义如下

```
public boolean equals(Object obj) {
    return (this == obj);
}

```

我们发现在`equals()`方法中就关键的`==`，那么`==`在Java中有什么含义呢，我们都知道在Java中分为基本数据类型和引用数据类型。那么`==`在这两个类型中作用是不一样的。

* 基本数据类型：比较的是`==`两边值是否相等
* 引用数据类型：比较的是`==`两边内存地址是否相等

> 基本数据类型包括：`byte`,`short`,`char`,`int`,`long`,`float`,`double`,`boolean`

而通过Java文档中的`equals()`方法描述，所有要实现自己的`equals()`方法都要遵守下面几个规则

* 自反性：对于任何对象x，`x.equals(x)`应该返回`true`
* 对称性：对于任何两个对象x和y，如果`x.equals(y)`返回`true`，那么`y.equals(x)`也应该返回`true`
* 传递性：对于多个对象x,y,z，如果`x.equals(y)`返回`true`,`y.equals(z)`返回`true`，那么`y.equals(z)`也应该返回`true`
* 一致性：对于两个非空对象x,y，在没有修改此对象的前提下，多次调用返回的结果应该相同
* 对于任何非空的对象x，`x.equals(null)`都应该返回`false`

## hashCode方法

`Object`中的`hashCode()`方法是一个本地方法，返回一个`int`类型的哈希值。


```
public native int hashCode();

```

在`hashCode()`方法中也有一些规约

* 如果对象在使用`equals`方法中进行比较的参数没有修改，那么多次调用一个对象的`hashCode()`方法返回的哈希值应该是相同的。
* 如果两个对象通过`equals`方法比较是相等的，那么**要求**这两个对象的`hashCode`方法返回的值也应该是相等的。
* 如果两个对象通过`equals`方法比较是不同的，那么也**不要求**这两个对象的`hashCode`方法返回的值是相同的。但是我们应该知道对于不同对象产生不同的哈希值对于哈希表(HashMap等等)能够提高性能。

## equals方法和hashCode方法会在哪用到

这两个方法经常出现在Java中的哪个类里面呢？如果看过`HashMap`源码的应该了解这两个方法经常出现在`HashMap`中。网上介绍`HashMap`类的文章有很多了，这里就简单介绍一下`HashMap`。

> 当一个节点中的链表超过了8的时候就会变为红黑树，以解决链表长度过长以后查询速度慢的缺点。

![](/img/pageImg/为什么重写了equals()也要重写hashCode()0.jpg)

`HashMap`是由数组和链表组成的高效存储数据的结构。那么是如何确定一个数据存储在数组中的哪个位置呢？就是通过`hashCode`方法进行计算出存储在哪个位置，还记得我们上面讲`hashCode`方法说了有可能两个不同对象的`hashCode`方法返回的值相同，那么此时就会产生冲突，产生冲突的话就会调用`equals`方法进行比对，如果不同，那么就将其加入链表尾部，如果相同就替换原数据。

> 计算位置当然不是上面简单的一个`hashCode`方法就计算出来，中间还有一些其他的步骤，这里可以简单的认为是`hashCode`确定了位置。

## 什么时候去覆盖这两个方法呢？

如果你不将自定义的类定义为`HashMap`的key值的话，那么我们重写了`equals`方法而没有重写`hashCode`方法，编译器不会报任何错，在运行时也不会抛任何异常。

如果你想将自定义的类定义为`HashMap`的key值得话，那么如果重写了`equals `方法那么就必须也重写`hashCode`方法。

接下来我们可以看一下我们使用自定义的类作为`HashMap`的key，并且自定义的类不重写`equals`和`hashCode`方法会发生什么。

自定义的类

```
@Builder
@NoArgsConstructor
@AllArgsConstructor
class CustomizedKey{
    private Integer id;
    private String name;
}

```

接下来我们看使用自定义的类作为key


```
    public static void main(String[] args) {
        
        Map<CustomizedKey, Integer> data = getData();
        
        CustomizedKey key = CustomizedKey.builder().id(1).name("key").build();
        
        Integer integer = data.get(key);
        
        System.out.printf(String.valueOf(integer));
    }

    private static Map<CustomizedKey,Integer> getData(){
        Map<CustomizedKey,Integer> customizedKeyIntegerMap = new HashMap<>();
        CustomizedKey key = CustomizedKey.builder().id(1).name("key").build();
        customizedKeyIntegerMap.put(key,10);
        return customizedKeyIntegerMap;
    }

```

我们可以看到程序最后打印的是一个`null`值。原因正如上面我们说的一样。

* `hashCode`：用来计算该对象放入数组中的哪个位置，因为是两个都是new的对象，所以即使里面的值一样，但是对象所处的地址却不同，所以使用默认的`hashCode`也就不同，当然在`hashMap`中就不会认为两个是一个对象。

接下来我们就重写一下这两个方法。如果我们使用`IDEA`的话，那么直接使用快捷键即可。

![](/img/pageImg/为什么重写了equals()也要重写hashCode()1.jpg)

接下来我们看我们实现的两个方法

```
@Builder
@NoArgsConstructor
@AllArgsConstructor
class CustomizedKey{
    private Integer id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CustomizedKey that = (CustomizedKey) o;
        return Objects.equals(id, that.id) &&
                Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

```

然后我们再次运行上面的程序发现输出打印已经变成了`10`。

> 我们也能够使用`Lombock`提供的`@EqualsAndHashCode`注解简化代码



## [本文代码地址](https://github.com/modouxiansheng/Doraemon)

## 往期文章

* [学会这几道链表算法题，面试再也不怕手写链表了](https://juejin.im/post/5dd884936fb9a07a9323de6f)
* [Spring事务传播属性有那么难吗？看这一篇就够了](https://juejin.im/post/5da6eee2f265da5bb977d65c)
* [后端框架开发需要注意的几点](https://juejin.im/post/5d6e6077f265da039a28a49f)
* [压缩20M文件从30秒到1秒的优化过程](https://juejin.im/post/5d5626cdf265da03a65312be)

## 参考文章

* [Java Object 源码]()
* [Java equals() and hashCode()](https://www.journaldev.com/21095/java-equals-hashcode)
