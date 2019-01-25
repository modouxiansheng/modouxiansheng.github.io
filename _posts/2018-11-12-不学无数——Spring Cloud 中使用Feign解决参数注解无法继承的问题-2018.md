---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——Spring Cloud 中使用Feign解决参数注解无法继承的问题        # 标题
subtitle:   Spring Cloud 中使用Feign解决参数注解无法继承的问题        #副标题
date:       2018-11-12          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
    - 源码解析
---

# Spring Cloud 中使用Feign解决参数注解无法继承的问题

在使用Feign的时候，通常先写一个接口类，然后再写实现类，根据官网的例子接下来编写一个简单的Feign的请求例子

```
@FeignClient("spring-cloud-eureka")
public interface FeignDemoApi {

    @RequestMapping("/testFeign")
    public String testSpringMvc(@RequestBody User user);
}

```

然后实现类如下

```
@RestController
public class FeignDemoApiImpl implements FeignDemoApi{
    @Override
    public String testSpringMvc(User user) {
        return user.getName();
    }
}

```
然后测试类编写如下

```
    @RequestMapping("/testSpringmvc")
    public void test6(){
        User user =new User();
        user.setName("test");
        user.setAge(18);
        feignDemoApi.testSpringMvc(user);
    }

```

发现在客户端进行接收的时候发现接收到的`User`为null

![](https://ws4.sinaimg.cn/large/006tNc79ly1fz7j28d4kfj30mc04uglz.jpg)

然后从网上查资料才知道需要在实现类的入参中也加入`@RequestBody `注解，这样才能接收到参数。但是不禁有个疑问，我们查看`@RequestMapping`和`@RequestBody`这两个注解的代码中都没有`@Inherited`这个可支持继承的注解，那么`@RequestMapping `为什么能发挥作用？而`@RequestBody `却不能发挥作用呢？

## @RequestMapping 注解

然后经过查资料了解到，SpringMvc在初始化时候会将带有`@Controller`初始化进Spring容器中，其实例化的类型为`RequestMappingInfo`类，此时在SpringMvc中会加载`@Controller `类注解下的所有带有`@ RequestMapping `的方法，将其`@RequestMapping`中的属性值也加载进来，如果在本类方法上找不到`@RequestMapping`注解信息的话，那么就会寻找父类相同方法名的`@RequestMapping`的注解。具体在`AnnotatedElementUtils`类中的`searchWithFindSemantics`方法中有下面的一段。表示在父类中寻找注解。

```
// Search on interfaces
for (Class<?> ifc : clazz.getInterfaces()) {
	T result = searchWithFindSemantics(ifc, annotationType, annotationName,
			containerType, processor, visited, metaDepth);
	if (result != null) {
		return result;
	}
}

```

此时就能知道为什么在子类中没有加`@RequestMapping`注解，但是却享有父类`@RequestMapping`注解的效果了。


## @RequestBody 注解

在上面的注解中我们了解到虽然`@RequestMapping`不支持继承，但是子类享有同样效果的原因就是在判断的时候如果子类没有就去父类找，但是在测试中我们发现`@RequestBody`是没有享受此效果的，所以我猜测在判断是否有注解的时候只是判断本类有没有此注解而没有判断父类。

经过查询资料，发现在SpringMvc中使用`RequestResponseBodyMethodProcessor `来进行入参和出参的解析，其中使用`supportsParameter `来判断是否有`@RequestBody`的注解

```
	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.hasParameterAnnotation(RequestBody.class);
	}

```

我们发现事实如我们猜测的一样。

## 解决方案

我们也可以像`@RequestMapping`注解一样进行判断如果本类没有的话，那么就对其父类进行判断。创建一个配置类然后将自定义的`ArgumentResolvers`放入到`RequestMappingHandlerAdapter`中

```
@Configuration
public class MyWebMvcConfigu implements BeanFactoryAware {
    private ConfigurableBeanFactory beanFactory;
    @Autowired
    private RequestMappingHandlerAdapter requestMappingHandlerAdapter;

	//判断其父类是否有注解
    public static <A extends Annotation> MethodParameter interfaceMethodParameter(MethodParameter parameter,
            Class<A> annotationType) {
        if (!parameter.hasParameterAnnotation(annotationType)) {
            for (Class<?> itf : parameter.getDeclaringClass().getInterfaces()) {
                try {
                    Method method = itf.getMethod(parameter.getMethod().getName(),
                            parameter.getMethod().getParameterTypes());
                    MethodParameter itfParameter = new MethodParameter(method, parameter.getParameterIndex());
                    if (itfParameter.hasParameterAnnotation(annotationType)) {
                        return itfParameter;
                    }
                } catch (NoSuchMethodException e) {
                    continue;
                }
            }
        }
        return parameter;
    }

    @PostConstruct
    public void modifyArgumentResolvers() {
        List<HandlerMethodArgumentResolver> list = new ArrayList<>(requestMappingHandlerAdapter.getArgumentResolvers());

        // RequestBody 支持接口注解
        list.add(0, new RequestResponseBodyMethodProcessor(requestMappingHandlerAdapter.getMessageConverters()) {
            @Override
            public boolean supportsParameter(MethodParameter parameter) {
                return super.supportsParameter(interfaceMethodParameter(parameter, RequestBody.class));
            }

            @Override
            // 支持@Valid验证
            protected void validateIfApplicable(WebDataBinder binder, MethodParameter methodParam) {
                super.validateIfApplicable(binder, interfaceMethodParameter(methodParam, Valid.class));
            }
        });

        // 修改ArgumentResolvers, 支持接口注解
        requestMappingHandlerAdapter.setArgumentResolvers(list);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (ConfigurableBeanFactory) beanFactory;
    }
}

```

然后我们就可以发现再重新调用以后子类没有加入`@RequestBody`注解也能够接收到实体类了

![](https://ws2.sinaimg.cn/large/006tNc79ly1fz8i3r4a98j31e80sk0wl.jpg)

## 参考文章

* [https://blog.csdn.net/phoebechen_gz/article/details/82500904](https://blog.csdn.net/phoebechen_gz/article/details/82500904)
* [https://blog.csdn.net/congcong68/article/details/40900713](https://blog.csdn.net/congcong68/article/details/40900713)

