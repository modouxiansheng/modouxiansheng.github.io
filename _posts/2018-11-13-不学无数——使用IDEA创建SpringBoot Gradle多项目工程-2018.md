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

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz9t3abd5yj30ll04fdkb.jpg)

### 2. 由于使用的Gradle，所以此处选择Gradle项目创建

![](https://ws1.sinaimg.cn/large/006tNc79ly1fz9sl0aqznj30yz0u07ak.jpg)

### 3. GroupId可以不用填写，Artifactid随喜好自己起个名字

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz9snwnfkjj31c00jmjt1.jpg)

### 4. 选择Gradle

![](https://ws3.sinaimg.cn/large/006tNc79ly1fz9sohcgdbj31dc0kggp4.jpg)

### 5. 此时一个Gradle的项目已经创建好了

![](https://ws4.sinaimg.cn/large/006tNc79ly1fz9sorvtfaj30ih0bdgm9.jpg)

### 6. 然后如果点击创建一个Module

![](https://ws1.sinaimg.cn/large/006tNc79ly1fz9sq27544j30kx050myg.jpg)

### 7. 由于要创建SpringBoot项目，所以如下图进行选择

![](https://ws3.sinaimg.cn/large/006tNc79ly1fz9sqneuv5j31cy0eaju7.jpg)

### 8. 填写基本信息，此时注意Type选择Gradle

![](https://ws3.sinaimg.cn/large/006tNc79ly1fz9sw2urqoj317t0u042r.jpg)

### 9. 选择自己想要的SpringBoot初始化加载的Jar包

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz9st6i8b1j31ek0re7cg.jpg)

### 10. 此时一个Module就已经创建好了，并且拥有自己的启动类

![](https://ws3.sinaimg.cn/large/006tNc79ly1fz9tblldmxj30k005s74n.jpg)

### 11. 相同的办法创建另一个Module

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz9t0ec6yrj30i308i3z4.jpg)

此时两个模块的项目已经创建完了，Gradle下载的Jar包的时候会特别慢，这时候可以在Gradle中将下载源给换了，换成阿里的国内镜像。镜像地址

```
maven {url 'http://maven.aliyun.com/nexus/content/groups/public/'}

```

在各个build.gradle文件中在`repositories`中加入此地址就行

![](https://ws3.sinaimg.cn/large/006tNc79ly1fz9tn0map3j30yq09v75x.jpg)


### [项目地址](https://github.com/modouxiansheng/SpringBoot-Practice)

