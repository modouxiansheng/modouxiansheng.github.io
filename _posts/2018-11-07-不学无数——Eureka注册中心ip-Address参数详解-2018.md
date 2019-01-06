---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——Eureka注册中心ip-Address参数详解        # 标题
subtitle:   Eureka注册中心ip-Address参数详解          #副标题
date:       2018-11-07           # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
    - 源码解析
---

# Eureka注册中心ip-Address参数详解


在Eureka中如果不指定任何的Ip参数的话，那么提供者注册到Eureka中，消费者进行消费的时候访问的Ip为部署Eureka服务器的Ip地址。

![Eureka不加参数调用图](http://upload-images.jianshu.io/upload_images/13146186-b60904e16a7f2177.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 那么正常情况下，服务器A调用将会调到服务B的身上，而此时如果在服务B中加入了一下的参数。那么就变了

 ```
eureka.instance.prefer-ip-address=true
eureka.instance.ip-address=172.12.9.0
 ```

 此时服务C和服务B是相同的服务，部署在不同的服务器中，Ip地址不同，那么此时调用的服务将会被Eureka转发到服务C中。

 ![image](http://upload-images.jianshu.io/upload_images/13146186-e206c09ad7fcfbb8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 源代码详解

我们可以在`EurekaClientAutoConfiguration`类中找到`eurekaInstanceConfigBean`方法。发现他在程序启动的时候就会加载一些eureka的配置信息

```
@Bean
	@ConditionalOnMissingBean(value = EurekaInstanceConfig.class, search = SearchStrategy.CURRENT)
	public EurekaInstanceConfigBean eurekaInstanceConfigBean(InetUtils inetUtils,
															 ManagementMetadataProvider managementMetadataProvider, ConfigurableEnvironment env) throws MalformedURLException {
		PropertyResolver environmentPropertyResolver = new RelaxedPropertyResolver(env);
		PropertyResolver eurekaPropertyResolver = new RelaxedPropertyResolver(env, "eureka.instance.");
		String hostname = eurekaPropertyResolver.getProperty("hostname");

		boolean preferIpAddress = Boolean.parseBoolean(eurekaPropertyResolver.getProperty("preferIpAddress"));
		String ipAddress = eurekaPropertyResolver.getProperty("ipAddress");
		boolean isSecurePortEnabled = Boolean.parseBoolean(eurekaPropertyResolver.getProperty("securePortEnabled"));
		String serverContextPath = environmentPropertyResolver.getProperty("server.contextPath", "/");
		int serverPort = Integer.valueOf(environmentPropertyResolver.getProperty("server.port", environmentPropertyResolver.getProperty("port", "8080")));

		Integer managementPort = environmentPropertyResolver.getProperty("management.port", Integer.class);// nullable. should be wrapped into optional
		String managementContextPath = environmentPropertyResolver.getProperty("management.contextPath");// nullable. should be wrapped into optional
		Integer jmxPort = environmentPropertyResolver.getProperty("com.sun.management.jmxremote.port", Integer.class);//nullable
		EurekaInstanceConfigBean instance = new EurekaInstanceConfigBean(inetUtils);

		instance.setNonSecurePort(serverPort);
		instance.setInstanceId(getDefaultInstanceId(environmentPropertyResolver));
		instance.setPreferIpAddress(preferIpAddress);
		instance.setSecurePortEnabled(isSecurePortEnabled);
		if (StringUtils.hasText(ipAddress)) {
			instance.setIpAddress(ipAddress);
		}

		if(isSecurePortEnabled) {
			instance.setSecurePort(serverPort);
		}

		if (StringUtils.hasText(hostname)) {
			instance.setHostname(hostname);
		}
		String statusPageUrlPath = eurekaPropertyResolver.getProperty("statusPageUrlPath");
		String healthCheckUrlPath = eurekaPropertyResolver.getProperty("healthCheckUrlPath");

		if (StringUtils.hasText(statusPageUrlPath)) {
			instance.setStatusPageUrlPath(statusPageUrlPath);
		}
		if (StringUtils.hasText(healthCheckUrlPath)) {
			instance.setHealthCheckUrlPath(healthCheckUrlPath);
		}

		ManagementMetadata metadata = managementMetadataProvider.get(instance, serverPort,
				serverContextPath, managementContextPath, managementPort);

		if(metadata != null) {
			instance.setStatusPageUrl(metadata.getStatusPageUrl());
			instance.setHealthCheckUrl(metadata.getHealthCheckUrl());
			if(instance.isSecurePortEnabled()) {
				instance.setSecureHealthCheckUrl(metadata.getSecureHealthCheckUrl());
			}
			Map<String, String> metadataMap = instance.getMetadataMap();
			if (metadataMap.get("management.port") == null) {
				metadataMap.put("management.port", String.valueOf(metadata.getManagementPort()));
			}
		}

		setupJmxPort(instance, jmxPort);
		return instance;
	}

```

此时我们看到`String ipAddress = eurekaPropertyResolver.getProperty("ipAddress");`就是在这进行设置的，此时如果没有设置`eureka.instance.ip-address`参数的时候，那么此ipAddress就是本机的Ip。这是如何设置的呢？我们可以看到代码中有这么一句

```
EurekaInstanceConfigBean instance = new EurekaInstanceConfigBean(inetUtils);
instance.setNonSecurePort(serverPort);
instance.setInstanceId(getDefaultInstanceId(environmentPropertyResolver));
instance.setPreferIpAddress(preferIpAddress);
instance.setSecurePortEnabled(isSecurePortEnabled);
if (StringUtils.hasText(ipAddress)) {
	instance.setIpAddress(ipAddress);
}

```
`EurekaInstanceConfigBean `先进行了初始化，我们进入`EurekaInstanceConfigBean `的构造参数中看

```
	public EurekaInstanceConfigBean(InetUtils inetUtils) {
		this.inetUtils = inetUtils;
		this.hostInfo = this.inetUtils.findFirstNonLoopbackHostInfo();
		this.ipAddress = this.hostInfo.getIpAddress();
		this.hostname = this.hostInfo.getHostname();
	}

```

可以看到在这进行了初始化`ipAddress`参数，其中`hostInfo`是如何得到的就进行讲解了，其本质就是为了获取本机的真实IP。到这给Eureka注册中心的注册信息已经准备好了

然后在`InstanceInfoReplicator`中调用run方法进行将自己信息注册到远程的Eureka中。

此时就可以看到如果设置`eureka.instance.prefer-ip-address`为false时，那么注册到Eureka中的Ip地址就是本机的Ip地址。如果设置了true并且也设置了`eureka.instance.ip-address`那么就将此ip地址注册到Eureka中。那么调用的时候，发送的请求目的地就是此Ip地址。
