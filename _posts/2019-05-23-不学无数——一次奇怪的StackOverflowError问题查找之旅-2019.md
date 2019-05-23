---
layout:     post                    # 使用的布局（不需要改）
title:      一次奇怪的StackOverflowError问题查找之旅        # 标题
subtitle:   一次奇怪的StackOverflowError问题查找之旅        #副标题
date:       2019-05-23          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-BJJ.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Tomcat
    - JAVA
---

# 一次奇怪的StackOverflowError问题查找之旅

公司最近买了一套老代码，在测试环境部署的时候发生了`nested exception is java.lang.StackOverflowError`的异常，当时看到这个异常首先想到是栈内存溢出，网上给出的解决办法就是加栈内存大小就行。趁着这个机会也了解一下什么是Java虚拟机栈。

## Java虚拟机栈

我们想要解决`StackOverflowError`问题就得了解其内部工作机制，首先我们需要了解Java虚拟机栈是什么。了解Java虚拟机栈之前我们先看一段代码。

```
public class TestStack {

    public static void main(String[] args) {
        a();
    }

    public static void a(){
        int a = 10;
        b();
    }

    public static void b(){
        int b = 10;
    }
}

```

上面的一段代码非常简单，整体流程就是

1. 先执行main()方法
2. 执行main()方法调用的a()方法，并且赋值a=10
3. 执行a()方法调用的b()方法，并赋值b=10

现在让我们了解上面一段简单流程情况下栈的动作流程，首先我们需要知道栈是每个线程独有的，而存放在栈里面的叫做栈帧，每进入一个方法就会有一个栈帧入栈的动作，即下面的黄色方块就是栈帧。每个栈帧里面存放的是局部变量表（例如此时的int a=10 ，a的值就存放在局部变量表中）、操作数栈、动态链接、方法出口等等。

> 局部变量表中存放的是编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、double、long）、对象引用（是指向存放的堆中实例对象的地址）

![入栈](/img/pageImg/一次奇怪的StackOverflowError问题查找之旅0.jpg)

当每个方法执行完毕以后，就会将此栈帧弹出栈。

![出栈](/img/pageImg/一次奇怪的StackOverflowError问题查找之旅1.jpg)

## 什么造成了StackOverflowError

上面我们知道了每个线程都有自己独有的栈空间，而栈帧中存放的私有的变量、对象地址引用、返回值都占用这内存空间，如果线程栈的大小超过了规定的大小，那么就会抛出`StackOverflowError`。例如我将上面代码稍微改一下

```
public class TestStack {

    public static void main(String[] args) {
        a();
    }

    public static void a(){
        b();
    }

    public static void b(){
        a();
    }
}

```

让他一直循环调用，并且设置每个线程栈大小为160K`Xss160k`，就会发现不断的加入栈帧，总会达到栈的最大值的，就会抛出`StackOverflowError `

![栈溢出](/img/pageImg/一次奇怪的StackOverflowError问题查找之旅2.jpg)

## 如何解决StackOverflowError

首先回到我们开头的问题上，由于是买过来的代码，所以需要知道引起`StackOverflowError`的是代码问题还是设置的栈内存小了。经过排查看到有一块验证信息的代码中用了责任链设计模式，在xml中设置的引用链条长达五十多个。因此在初始化的时候如果设置的栈内存不够大的话那么就会发生`StackOverflowError `。

![责任链](/img/pageImg/一次奇怪的StackOverflowError问题查找之旅3.jpg)

所以就直接增大栈内存大小就行了，我们使用的是公司给提供的Tomcat包部署的项目，经网上查询在Tomcat中设置JVM参数只需要在`catalina.sh`中设置`JAVA_OPTS="$JAVA_OPTS -Xss500k"`参数即可，于是设置了此参数后又重新启动，发现还是`StackOverflowError`错误。



然后经过`ps -ef |grep tomcat`查看发现设置的栈大小没有生效。于是在Tomcat目录中`grep -r "Xss" *`查找看是哪里配置的栈大小，最后发现是在`setenv.sh`文件中配置JVM信息，而在`setenv.sh `文件中配置的是`CATALINA_OPTS="$CATALINA_OPTS -Xss256k"`。

> 配置只是该Tomcat使用的环境变量，使用`CATALINA_OPTS`，而如果配置其他Java应用程序也要使用的环境变量，比如JBoss，在`JAVA_OPTS`中设置

经过试验发现如果在配置文件中出现了相同的配置信息。例如

```
CATALINA_OPTS="$CATALINA_OPTS -Xss300k"
JAVA_OPTS="$JAVA_OPTS -Xss200k"
```
哪怕`JAVA_OPTS`是在后面，那么也是以`CATALINA_OPTS `所配置的为准，如果两个都是

```
CATALINA_OPTS="$CATALINA_OPTS -Xss300k"
CATALINA_OPTS ="$CATALINA_OPTS -Xss200k"
```

那么就以出现的先后顺序，后出现的配置生效。随后在`setenv.sh `文件中将栈空间大小增大问题也就随之解决了。
