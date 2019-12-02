---
layout:     post                    # 使用的布局（不需要改）
title:      一文彻底搞懂Cookie、Session、Token到底是什么        # 标题
subtitle:   一文彻底搞懂Cookie、Session、Token到底是什么        #副标题
date:       2019-12-02          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-rwd.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Spring
    - 网络
---

> 笔者文笔功力尚浅，如有不妥，请慷慨指出，必定感激不尽


## Cookie

```
洛：大爷，楼上322住的是马冬梅家吧？

大爷：马都什么？

夏洛：马冬梅。

大爷：什么都没啊？

夏洛：马冬梅啊。

大爷：马什么没？

夏洛：行，大爷你先凉快着吧。

```

在了解这三个概念之前我们先要了解HTTP是**无状态**的Web服务器，什么是无状态呢？就像上面夏洛特烦恼中经典的一幕对话一样，一次对话完成后下一次对话完全不知道上一次对话发生了什么。如果在Web服务器中只是用来管理静态文件还好说，对方是谁并不重要，把文件从磁盘中读取出来发出去即可。但是随着网络的不断发展，比如电商中的购物车只有记住了用户的身份才能够执行接下来的一系列动作。所以此时就需要我们**无状态**的服务器记住一些事情。

那么Web服务器是如何记住一些事情呢？既然Web服务器记不住东西，那么我们就在外部想办法记住，相当于服务器给每个客户端都贴上了一个小纸条。上面记录了服务器给我们返回的一些信息。然后服务器看到这张小纸条就知道我们是谁了。那么`Cookie`是谁产生的呢？Cookies是由服务器产生的。接下来我们描述一下`Cookie`产生的过程

1. 浏览器第一次访问服务端时，服务器此时肯定不知道他的身份，所以创建一个独特的身份标识数据，格式为`key=value`，放入到`Set-Cookie`字段里，随着响应报文发给浏览器。
2. 浏览器看到有`Set-Cookie`字段以后就知道这是服务器给的身份标识，于是就保存起来，下次请求时会自动将此`key=value`值放入到`Cookie`字段中发给服务端。
3. 服务端收到请求报文后，发现`Cookie`字段中有值，就能根据此值识别用户的身份然后提供个性化的服务。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么0.jpg)

接下来我们用代码演示一下服务器是如何生成，我们自己搭建一个后台服务器，这里我用的是SpringBoot搭建的，并且写入SpringMVC的代码如下。

```
@RequestMapping("/testCookies")
public String cookies(HttpServletResponse response){
    response.addCookie(new Cookie("testUser","xxxx"));
    return "cookies";
}
 
```

项目启动以后我们输入路径`http://localhost:8005/testCookies`，然后查看发的请求。可以看到下面那张图使我们首次访问服务器时发送的请求，可以看到服务器返回的响应中有`Set-Cookie`字段。而里面的`key=value`值正是我们服务器中设置的值。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么1.jpg)

接下来我们再次刷新这个页面可以看到在请求体中已经设置了`Cookie`字段，并且将我们的值也带过去了。这样服务器就能够根据`Cookie`中的值记住我们的信息了。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么2.jpg)

接下来我们换一个请求呢？是不是`Cookie`也会带过去呢？接下来我们输入路径`http://localhost:8005`请求。我们可以看到`Cookie`字段还是被带过去了。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么3.jpg)

那么浏览器的`Cookie`是存放在哪呢？如果是使用的是`Chrome`浏览器的话，那么可以按照下面步骤。

1. 在计算机打开`Chrome`
2. 在右上角，一次点击`更多`图标->`设置`
3. 在底部，点击`高级`
4. 在`隐私设置和安全性`下方，点击网站设置
5. 依次点击`Cookie`->查看所有`Cookie和网站数据`

然后可以根据域名进行搜索所管理的`Cookie`数据。所以是浏览器替你管理了`Cookie`的数据，如果此时你换成了`Firefox`等其他的浏览器，因为`Cookie`刚才是存储在`Chrome`里面的，所以服务器又蒙圈了，不知道你是谁，就会给`Firefox`再次贴上小纸条。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么4.jpg)

### Cookie中的参数设置

说到这里，应该知道了`Cookie`就是服务器委托浏览器存储在客户端里的一些数据，而这些数据通常都会记录用户的关键识别信息。所以`Cookie`需要用一些其他的手段用来保护，防止外泄或者窃取，这些手段就是`Cookie`的属性。

|参数名|作用|后端设置方法|
|--------|------|------|
|Max-Age|设置cookie的过期时间，单位为秒|`cookie.setMaxAge(10)`|
|Domain|指定了Cookie所属的域名|`cookie.setDomain("")`|
|Path|指定了Cookie所属的路径|`cookie.setPath("");`|
|HttpOnly|告诉浏览器此Cookie只能靠浏览器Http协议传输,禁止其他方式访问|`cookie.setHttpOnly(true)`|
|Secure|告诉浏览器此Cookie只能在Https安全协议中传输,如果是Http则禁止传输|`cookie.setSecure(true)`|

下面我就简单演示一下这几个参数的用法及现象。

#### Path

设置为`cookie.setPath("/testCookies")`，接下来我们访问`http://localhost:8005/testCookies`，我们可以看到在左边和我们指定的路径是一样的，所以`Cookie`才在请求头中出现，接下来我们访问`http://localhost:8005`，我们发现没有`Cookie`字段了，这就是`Path`控制的路径。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么5.jpg)

#### Domain

设置为`cookie.setDomain("localhost")`，接下来我们访问`http://localhost:8005/testCookies`我们发现下图中左边的是有`Cookie`的字段的，但是我们访问`http://172.16.42.81:8005/testCookies`，看下图的右边可以看到没有`Cookie`的字段了。这就是`Domain`控制的域名发送`Cookie`。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么6.jpg)

接下来的几个参数就不一一演示了，相信到这里大家应该对`Cookie`有一些了解了。

## Session

> Cookie是存储在客户端方，Session是存储在服务端方，客户端只存储`SessionId`

在上面我们了解了什么是`Cookie`，既然浏览器已经通过`Cookie`实现了有状态这一需求，那么为什么又来了一个`Session`呢？这里我们想象一下，如果将账户的一些信息都存入`Cookie`中的话，一旦信息被拦截，那么我们所有的账户信息都会丢失掉。所以就出现了`Session`，在一次会话中将重要信息保存在`Session`中，浏览器只记录`SessionId`一个`SessionId`对应一次会话请求。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么7.jpg)

```
@RequestMapping("/testSession")
@ResponseBody
public String testSession(HttpSession session){
    session.setAttribute("testSession","this is my session");
    return "testSession";
}


@RequestMapping("/testGetSession")
@ResponseBody
public String testGetSession(HttpSession session){
    Object testSession = session.getAttribute("testSession");
    return String.valueOf(testSession);
}

```

这里我们写一个新的方法来测试`Session`是如何产生的，我们在请求参数中加上`HttpSession session`，然后再浏览器中输入`http://localhost:8005/testSession`进行访问可以看到在服务器的返回头中在`Cookie`中生成了一个`SessionId`。然后浏览器记住此`SessionId`下次访问时可以带着此Id，然后就能根据此Id找到存储在服务端的信息了。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么8.jpg)

此时我们访问路径`http://localhost:8005/testGetSession`，发现得到了我们上面存储在`Session`中的信息。那么`Session`什么时候过期呢？

* 客户端：和`Cookie`过期一致，如果没设置，默认是关了浏览器就没了，即再打开浏览器的时候初次请求头中是没有`SessionId`了。
* 服务端：服务端的过期是真的过期，即服务器端的`Session`存储的数据结构多久不可用了，默认是30分钟。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么9.jpg)

既然我们知道了`Session`是在服务端进行管理的，那么或许你们看到这有几个疑问，`Session`是在在哪创建的？`Session`是存储在什么数据结构中？接下来带领大家一起看一下`Session`是如何被管理的。

`Session`的管理是在容器中被管理的，什么是容器呢？`Tomcat`、`Jetty`等都是容器。接下来我们拿最常用的`Tomcat`为例来看下`Tomcat`是如何管理`Session`的。在`ManageBase`的`createSession`是用来创建`Session`的。

```

@Override
public Session createSession(String sessionId) {
    //首先判断Session数量是不是到了最大值，最大Session数可以通过参数设置
    if ((maxActiveSessions >= 0) &&
            (getActiveSessions() >= maxActiveSessions)) {
        rejectedSessions++;
        throw new TooManyActiveSessionsException(
                sm.getString("managerBase.createSession.ise"),
                maxActiveSessions);
    }

    // 重用或者创建一个新的Session对象，请注意在Tomcat中就是StandardSession
    // 它是HttpSession的具体实现类，而HttpSession是Servlet规范中定义的接口
    Session session = createEmptySession();


    // 初始化新Session的值
    session.setNew(true);
    session.setValid(true);
    session.setCreationTime(System.currentTimeMillis());
    // 设置Session过期时间是30分钟
    session.setMaxInactiveInterval(getContext().getSessionTimeout() * 60);
    String id = sessionId;
    if (id == null) {
        id = generateSessionId();
    }
    session.setId(id);// 这里会将Session添加到ConcurrentHashMap中
    sessionCounter++;
    
    //将创建时间添加到LinkedList中，并且把最先添加的时间移除
    //主要还是方便清理过期Session
    SessionTiming timing = new SessionTiming(session.getCreationTime(), 0);
    synchronized (sessionCreationTiming) {
        sessionCreationTiming.add(timing);
        sessionCreationTiming.poll();
    }
    return session
}

```

到此我们明白了`Session`是如何创建出来的，创建出来后`Session`会被保存到一个`ConcurrentHashMap`中。可以看`StandardSession`类。

```
protected Map<String, Session> sessions = new ConcurrentHashMap<>();

```

到这里大家应该对`Session`有简单的了解了。

> Session是存储在Tomcat的容器中，所以如果后端机器是多台的话，因此多个机器间是无法共享Session的，此时可以使用Spring提供的分布式Session的解决方案，是将Session放在了Redis中。


## Token

`Session`是将要验证的信息存储在服务端，并以`SessionId`和数据进行对应，`SessionId`由客户端存储，在请求时将`SessionId`也带过去，因此实现了状态的对应。而`Token`是在服务端将用户信息经过Base64Url编码过后传给在客户端，每次用户请求的时候都会带上这一段信息，因此服务端拿到此信息进行解密后就知道此用户是谁了，这个方法叫做JWT(Json Web Token)。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么10.jpg)

> `Token`相比较于`Session`的优点在于，当后端系统有多台时，由于是客户端访问时直接带着数据，因此无需做共享数据的操作。

### Token的优点

1. 简洁：可以通过`URL`,`POST`参数或者是在`HTTP`头参数发送，因为数据量小，传输速度也很快
2. 自包含：由于串包含了用户所需要的信息，避免了多次查询数据库
3. 因为Token是以Json的形式保存在客户端的，所以JWT是跨语言的
4. 不需要在服务端保存会话信息，特别适用于分布式微服务

### JWT的结构

实际的JWT大概长下面的这样，它是一个很长的字符串，中间用`.`分割成三部分

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么11.jpg)

JWT是有三部分组成的

#### Header

是一个Json对象，描述JWT的元数据，通常是下面这样子的

```
{
  "alg": "HS256",
  "typ": "JWT"
}

```

上面代码中，alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT。
最后，将上面的 JSON 对象使用 Base64URL 算法转成字符串。

> JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。

#### Payload

Payload部分也是一个Json对象，用来存放实际需要传输的数据，JWT官方规定了下面几个官方的字段供选用。

* iss (issuer)：签发人
* exp (expiration time)：过期时间
* sub (subject)：主题
* aud (audience)：受众
* nbf (Not Before)：生效时间
* iat (Issued At)：签发时间
* jti (JWT ID)：编号

当然除了官方提供的这几个字段我们也能够自己定义私有字段，下面就是一个例子

```
{
  "name": "xiaoMing",
  "age": 14
}

```

默认情况下JWT是不加密的，任何人只要在网上进行Base64解码就可以读到信息，所以一般不要将秘密信息放在这个部分。这个Json对象也要用`Base64URL `算法转成字符串

#### Signature

Signature部分是对前面的两部分的数据进行签名，防止数据篡改。

首先需要定义一个秘钥，这个秘钥只有服务器才知道，不能泄露给用户，然后使用Header中指定的签名算法(默认情况是HMAC SHA256)，算出签名以后将Header、Payload、Signature三部分拼成一个字符串，每个部分用`.`分割开来，就可以返给用户了。

> HS256可以使用单个密钥为给定的数据样本创建签名。当消息与签名一起传输时，接收方可以使用相同的密钥来验证签名是否与消息匹配。

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么12.jpg)

### Java中如何使用Token

上面我们介绍了关于JWT的一些概念，接下来如何使用呢？首先在项目中引入Jar包

```
compile('io.jsonwebtoken:jjwt:0.9.0')

```

然后编码如下

```
// 签名算法 ，将对token进行签名
SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
// 通过秘钥签名JWT
byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary("SECRET");
Key signingKey = new SecretKeySpec(apiKeySecretBytes, signatureAlgorithm.getJcaName());
Map<String,Object> claimsMap = new HashMap<>();
claimsMap.put("name","xiaoMing");
claimsMap.put("age",14);
JwtBuilder builderWithSercet = Jwts.builder()
        .setSubject("subject")
        .setIssuer("issuer")
        .addClaims(claimsMap)
        .signWith(signatureAlgorithm, signingKey);
System.out.printf(builderWithSercet.compact());

```

发现输出的Token如下

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzdWJqZWN0IiwiaXNzIjoiaXNzdWVyIiwibmFtZSI6InhpYW9NaW5nIiwiYWdlIjoxNH0.3KOWQ-oYvBSzslW5vgB1D-JpCwS-HkWGyWdXCP5l3Ko

```

此时在网上随便找个Base64解码的网站就能将信息解码出来

![](/img/pageImg/一文彻底搞懂Cookie、Session、Token到底是什么13.jpg)

## 总结

相信大家看到这应该对`Cookie`、`Session`、`Token`有一定的了解了，接下来再回顾一下重要的知识点

* Cookie是存储在客户端的
* Session是存储在服务端的，可以理解为一个状态列表。拥有一个唯一会话标识`SessionId`。可以根据`SessionId`在服务端查询到存储的信息。
* Session会引发一个问题，即后端多台机器时Session共享的问题，解决方案可以使用Spring提供的框架。
* Token类似一个令牌，无状态的，服务端所需的信息被Base64编码后放到Token中，服务器可以直接解码出其中的数据。

## [GitHub代码地址](https://github.com/modouxiansheng/Doraemon)

## 猜你想读

* [学会这几道链表算法题，面试再也不怕手写链表了](https://juejin.im/post/5dd884936fb9a07a9323de6f)
* [Spring事务传播属性有那么难吗？看这一篇就够了](https://juejin.im/post/5da6eee2f265da5bb977d65c)
* [后端框架开发需要注意的几点](https://juejin.im/post/5d6e6077f265da039a28a49f)
* [压缩20M文件从30秒到1秒的优化过程](https://juejin.im/post/5d5626cdf265da03a65312be)
* [为什么重写了equals()也要重写hashCode()](https://juejin.im/post/5ddde269e51d4532c8596851)

## 参考文章

* [Cookies vs. Tokens: The Definitive Guide](https://dzone.com/articles/cookies-vs-tokens-the-definitive-guide)
* [彻底弄懂session，cookie，token](https://segmentfault.com/a/1190000017831088)
* [透视HTTP协议](https://time.geekbang.org/column/article/106034)
* [Manager组件：Tomcat的Session管理机制解析](https://time.geekbang.org/column/article/109635)
* [JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
* [SpringBoot集成JWT实现token验证](https://www.jianshu.com/p/e88d3f8151db)
* [JSON Web Token - 在Web应用间安全地传递信息](http://blog.leapoahead.com/2015/09/06/understanding-jwt/)