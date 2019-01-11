---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——如何使用Spring的FactoryBean接口        # 标题
subtitle:   如何使用Spring的FactoryBean接口         #副标题
date:       2018-11-10          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
    - Spring
---

# 如何使用Spring的FactoryBean接口

在Spring容器中有两类的Bean，一类是普通的Bean，一类是工厂Bean。这两种Bean都是被Spring的容器进行管理的。而Spring也提供了一个接口用于扩展工厂Bean，我们只要实现`org.springframework.beans.factory.FactoryBean`即可。

## 如何使用

首先我们看一下`FactoryBean `接口

```
public interface FactoryBean<T> {

	T getObject() throws Exception;

	Class<?> getObjectType();

	boolean isSingleton();

}

```

这三个方法的作用分别是

* `getObject `：返回本工厂创建的对象实例。此实例也许是共享的，依赖于该工厂返回的是单例或者是原型。
* `getObjectType `：返回对象类型
* `isSingleton `：表示被工厂创建的实例是否是单例

现在让我们开始使用，创建一个`UserFactory`用来创建实例`User`

```

public class User {

    private String name;
    private Integer age;
-----get set 方法
}

```

`UserFactory `工厂类

```
public class UserFactory implements FactoryBean<User> {
    private String name;
    private Integer age;

    @Override
    public User getObject() throws Exception {
        return new User(name,age);
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

```

添加配置类将`UserFactory`加入到容器中

```
@Configuration
public class FactoryBeanAppConfig {

    @Bean
    public UserFactory userFactory(){
        UserFactory userFactory = new UserFactory();
        userFactory.setAge(10);
        userFactory.setName("Grace");
        return userFactory;
    }

}

```

接下来我们就能通过`UserFactory `工厂类进行创建`User`了

```
public class BeanFactoryTest extends BaseTest{

    @Autowired
    private User user;

    @Test
    public void testBeanFactory(){
        assertEquals(new Integer(10),user.getAge());
        assertEquals("Grace",user.getName());
    }
}

```

此时我们Debug也能看到其我们自动注入的`User`类就是通过`UserFactory `创建的实例

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz1l23dq53j30sy0e8dhf.jpg)

## 加载顺序

有时候我们需要在工厂Bean创建之后，但是`getObject`方法执行之前执行一些动作，例如资源配置检查之类的动作的话，我们可以使用`@PostConstruct`注解在方法上，那么此方法就会在执行`getObject()`方法执行之前先执行。

此时我们在`UserFactory `工厂类中修改如下，增加`@PostConstruct`注解的方法，并且增加打印日志。

```
public class UserFactory implements FactoryBean<User> {
    private String name;
    private Integer age;

    @Override
    public User getObject() throws Exception {
        System.out.println("getObject Begin");
        return new User(name,age);
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    @PostConstruct
    public void Initialize(){
        System.out.println("Initialize Begin");
    }
}

```


此时我们运行就会发现执行顺序

![](https://ws4.sinaimg.cn/large/006tNc79ly1fz1lab5wihj30hh0a3jsv.jpg)

## `AbstractFactoryBean `类

Spring为我们提供了一个关于`FactoryBean `的抽象类`AbstractFactoryBean `为我们简化了操作。我们直接继承此类就能更加方便的创建单例或者非单例的实体类了。接下来我们就演示一下如何创建单例和非单例的类。首先先创建两个工厂，一个工厂`SingleUserFactory`负责创建单例，一个工厂`NonSingleUserFactory`负责创建非单例

```
public class SingleUserFactory extends AbstractFactoryBean<User> {

    private String name;
    private Integer age;

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    protected User createInstance() throws Exception {
        return new User(name,age);
    }
--get set方法
}

```

`NonSingleUserFactory`类

```
public class NonSingleUserFactory extends AbstractFactoryBean<User> {

    private String name;
    private Integer age;
	//设置为非单例模式
    public NonSingleUserFactory(){
        setSingleton(false);
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    protected User createInstance() throws Exception {
        return new User(name,age);
    }

---- get set方法
}

```

并且在配置文件中加入对这两个工厂类的配置

```
    @Bean(name = "singleUser")
    public SingleUserFactory getSingle(){
        SingleUserFactory singleUserFactory = new SingleUserFactory();
        singleUserFactory.setAge(12);
        singleUserFactory.setName("Single");
        return singleUserFactory;
    }

    @Bean(name = "nonSingleUser")
    public NonSingleUserFactory getNonSingle(){
        NonSingleUserFactory nonSingleUserFactory = new NonSingleUserFactory();
        nonSingleUserFactory.setAge(12);
        nonSingleUserFactory.setName("NonSingle");
        return nonSingleUserFactory;
    }

```

在测试类中测试如下

```
public class SingleBeanFactoryTest extends BaseTest{
    @Resource(name = "singleUser")
    private User user1;
    @Resource(name = "singleUser")
    private User user2;
    @Resource(name = "nonSingleUser")
    private User user3;
    @Resource(name = "nonSingleUser")
    private User user4;

    @Test
    public void testSingleBean(){
        assertTrue(user1 == user2);
        assertTrue(user3 != user4);
    }
}

```
我们查看类的路径就可以知道哪些类是一样的了

![](https://ws1.sinaimg.cn/large/006tNc79ly1fz1mkiw93qj30dm05ogmz.jpg)

从结果中我们可以看到`SingleUserFactory`工厂类创建的都是单例的对象，而`NonSingleUserFactory`创建的都是非单例的对象。如果是创建单例的那么就无需设置`singleton`的值，因为他是默认为`True`的。

> 使用`FactoryBean `能够在Spring中更好更便捷的创建管理一些有着复杂构造逻辑的实体类。


### 源码可在[github](https://github.com/modouxiansheng/Spring-Practice/tree/master/src/main/java/com/example/springpractice/springpractice/Spring/FactoryBean)中查看



