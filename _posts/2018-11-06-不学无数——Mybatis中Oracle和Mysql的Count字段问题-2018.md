---
layout:     post                    # 使用的布局（不需要改）
title:      不学无数——Mybatis中Oracle和Mysql的Count字段问题       # 标题
subtitle:   Mybatis中Oracle和Mysql的Count字段问题          #副标题
date:       2018-11-06           # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Java
    - 源码解析
---

# Mybatis中Oracle和Mysql的Count字段问题

我们在进行项目开发时经常会碰到查询总数的问题，所以我们直接是用`select count(1) from table`来进行查询。那么在Mybatis通常情况下我们是这么写的

```
<select id="testCount" resultType="int">
    select count(1) as "totalCount" from ams.t_ams_ac_pmt_dtl
</select>
```
这样做是没问题的，无论是在Oracle还是Mysql，因为Mybatis中有类型处理器，当其检测到resultType时会将其值转化为Int类型的值。所以接收是没问题的。但是如果是如下的写法的话，将resultType变为Map，那么就会有问题。

```
<select id="testCount" resultType="Map">
    select count(1) as "totalCount" from ams.t_ams_ac_pmt_dtl
</select>

```
在Mybatis中如果resultType是Map的话，那么在接收结果参数的时候会实例化一个`Map<String,Object>`的Map，问题就出现在这，在之前的代码中是用`Map<String,BigDecimal>`来接收的，这在Oracle中是没有问题的，因为在Oracle中count函数获得的值在Java对应的类型是BigDecimal，但是在Mysql中就会出现问题。

在`ResultSetMetaData.getClassNameForJavaType()`的方法中可以看到Mysql字段对应的Java字段，我们可以得知在Mysql中查询的count得到的数据库类型是BigInt类型的对应的Java类型是Long

![image](/img/pageImg/Mybatis中Oracle和Mysql的Count字段问题0.jpg)

## 解决办法

* 改代码，将接收的`Map<String, BigDecimal>`改为`Map<String, Object>`，然后进行类型转换
* 改Sql

![image](/img/pageImg/Mybatis中Oracle和Mysql的Count字段问题1.jpg)

我们可以看到在Mysql中的Decimal和Numeric类型的都被转化为了BigDecimal，所以在Sql文件中进行类型转换就行

```
select CAST(count(1) as decimal(18,0)) as "totalCount" from table

```
