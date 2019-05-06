---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——使用IDEA创建SpringBoot Gradle多项目工程        # 标题
subtitle:   使用IDEA创建SpringBoot Gradle多项目工程        #副标题
date:       2018-11-13          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
    - 技巧
---


# 使用IDEA创建SpringBoot Gradle多项目工程

最近想用Springboot做一个项目练手，但是发现在一个项目中运行两个Springboot的工程不会创建，在网上查了一些资料，记录一下创建多项目工程的步骤

### 1. 点击Idea下的File新建一个Project

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程0.jpg)

### 2. 由于使用的Gradle，所以此处选择Gradle项目创建

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程1.jpg)

### 3. GroupId可以不用填写，Artifactid随喜好自己起个名字

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程2.jpg)

### 4. 选择Gradle

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程3.jpg)

### 5. 此时一个Gradle的项目已经创建好了

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程4.jpg)

### 6. 然后如果点击创建一个Module

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程5.jpg)

### 7. 由于要创建SpringBoot项目，所以如下图进行选择

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程6.jpg)

### 8. 填写基本信息，此时注意Type选择Gradle

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程7.jpg)

### 9. 选择自己想要的SpringBoot初始化加载的Jar包

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程8.jpg)

### 10. 此时一个Module就已经创建好了，并且拥有自己的启动类

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程9.jpg)

### 11. 相同的办法创建另一个Module

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程10.jpg)

此时两个模块的项目已经创建完了，Gradle下载的Jar包的时候会特别慢，这时候可以在Gradle中将下载源给换了，换成阿里的国内镜像。镜像地址

```
maven {url 'http://maven.aliyun.com/nexus/content/groups/public/'}

```

在各个build.gradle文件中在`repositories`中加入此地址就行

![](/img/pageImg/使用IDEA创建SpringBoot-Gradle多项目工程11.jpg)


### [项目地址](http://github.com/modouxiansheng/SpringBoot-Practice)

