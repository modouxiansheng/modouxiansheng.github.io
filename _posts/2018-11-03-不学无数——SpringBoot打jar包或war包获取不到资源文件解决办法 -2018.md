---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——SpringBoot打jar包或war包获取不到资源文件解决办法        # 标题
subtitle:   SpringBoot打jar包或war包获取不到资源文件解决办法          #副标题
date:       2018-11-03           # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
---

## 问题描述

在开发过程中我们经常会碰到要在代码中获取资源文件的情况，而我在最近将原有的Tomcat的原生项目迁移到SpringBoot项目中时碰到一个问题，就是在本地运行时，获取本地的xml资源文件是能够获取到的，但是项目打成war包然后将其部署到Tomcat中运行时，就会发生问题，报找不到资源文件的错误。然后经过寻找排查确定了是下面代码通过`ClassLoader`获取路径的时候出错了。

```
ExcelXmlModelFactory.class.getClassLoader().getResource("template/").getPath()

```
我的资源文件存放路径如下

![image](/img/pageImg/SpringBoot打jar包或war包获取不到资源文件解决办法0.jpg)

在本地中打印的日志路径为

```
/Users/hupengfei/git/lap/lap-service/out/production/resources/template

```

但是在将SpringBoot打包成war包部署到Tomcat中时打印的目录为

```
/home/app/lap/app/lap-service-1.0.0-SNAPSHOT.war!/WEB-INF/classes!/template/

```
可以看到在Linux中无法直接访问未经解压的文件，所以就会找不到文件。

## 解决办法

通过`ClassLoader`的`getResourceAsStream()`方法获取其流，就能够获取到

> 读取jar里面的文件，我们只能用流去读取，不能用File

## 获取资源的两种方式

通常在开发过程中会碰到读取配置文件的问题，一般有两种方式进行读取。一种是`Class.getResource(String path)`，一种是`ClassLoader.getResource(String path)`，这两种虽然都能读取文件，但是在`path`的填写上有一点点的不同。

### Class.getResource

* path以`/`开头：则是从ClassPath根下获取
* path不以`/`开头：默认是从此类所在的包下取资源

下面有个例子

```
public class Test {
    public static void main(String[] args) {
    	 System.out.println(Test.class.getResource("/"));
        System.out.println(Test.class.getResource(""));
    }
}

```

输出如下

```
file:/Users/hupengfei/git/Test/out/production/classes/
file:/Users/hupengfei/git/Test/out/production/classes/Practice/Day13/
```

那么如果在`resource`下有三个资源文件

![image](/img/pageImg/SpringBoot打jar包或war包获取不到资源文件解决办法1.jpg)

那么该怎么获取这三个文件呢，因为在class文件夹中的目录结构如下

```
-- classes
	-- Convience
	-- Practice
	-- Test
```
所以如果想要获取Test下的资源文件，就如下的获取方法

```
System.out.println(Test.class.getResource("../../Test/1.xml"));
System.out.println(Test.class.getResource("/Test/1.xml"));
```

### ClassLoader.getResource

> `ClassLoader.getResource`的path中不能以`/`开头，path是默认是从根目录下进行读取的

例子如下

```
System.out.println(Test.class.getClassLoader().getResource(""));
System.out.println(Test.class.getClassLoader().getResource("/"));
```
打印如下

```
file:/Users/hupengfei/git/Test/out/production/classes/
null
```
从上面例子我们可以看到

`Test.class.getClassLoader().getResource("")=Test.class.getResource("/")`

### 两个获取资源文件的差别

其实查看`Class.getResource`中可以看到

```
	public java.net.URL getResource(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResource(name);
        }
        return cl.getResource(name);
    }

```

他最后调用的还是`ClassLoader.getResource`这个方法，那么为什么会有`path`的差别呢，因为其`resolveName `方法中对传的`/`进行了解析，解析为了空字符串。

```
    private String resolveName(String name) {
        if (name == null) {
            return name;
        }
        if (!name.startsWith("/")) {
            Class<?> c = this;
            while (c.isArray()) {
                c = c.getComponentType();
            }
            String baseName = c.getName();
            int index = baseName.lastIndexOf('.');
            if (index != -1) {
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else {
            name = name.substring(1);
        }
        return name;
    }

```
可以看到在这穿进去的为`/`

![image](/img/pageImg/SpringBoot打jar包或war包获取不到资源文件解决办法2.jpg)

传出的是

![image](/img/pageImg/SpringBoot打jar包或war包获取不到资源文件解决办法3.jpg)


## 参考文章

* [http://www.cnblogs.com/yejg1212/p/3270152.html](http://www.cnblogs.com/yejg1212/p/3270152.html)
* [http://blog.csdn.net/xiaoxudong666/article/details/82192425](http://blog.csdn.net/xiaoxudong666/article/details/82192425)







