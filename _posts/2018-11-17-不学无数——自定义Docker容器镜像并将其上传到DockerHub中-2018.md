---
layout:     post                    # 使用的布局（不需要改）
title:      自定义Docker容器镜像并将其上传到DockerHub中        # 标题
subtitle:   自定义Docker容器镜像并将其上传到DockerHub中        #副标题
date:       2018-11-17          # 时间
author:     不学无数                      # 作者
header-img: img/post-bg-hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 容器
---

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0f1o5o3rgj31500u0wtx.jpg)

# 自定义Docker容器镜像并将其上传到DockerHub中

Docker从2013年发布至今，一直是广受瞩目，所以我们或多或少也应该了解一些Docker的技术原理，而学习一项技术有了兴趣才能更好的让你持续学习下去。如果让你体会到Docker的神奇之处那么兴趣或许会大一点，接下来我们就先从自定义一个自己的Docker容器镜像来开启学习Docker的第一步。

## 自定义Docker容器镜像

在开始我们的实践之前，我们当然得在机器上安装Docker，请参考官方文档

* [Mac安装方法](https://docs.docker.com/docker-for-mac/install/)
* [Windows安装方法](https://docs.docker.com/docker-for-windows/install/)

安装完Docker以后我们就需要编写我们要部署的项目代码了，我们简单的部署一个用Python编写的Web应用，如果是之前部署到虚拟机中的话我们得在虚拟机中安装各种环境才能将此项目部署成功。而现在在Docker中我们只需要一个文件就能将环境全部打包成一个镜像并且运行。

```
from flask import Flask
import socket
import os

app = Flask(__name__)

@app.route('/')
def hello():
    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname())

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)

```

这个程序我们在本机启动的话在浏览器中输入` http://0.0.0.0:80/`即可访问到后台打印的信息。接下来我们就将其制作成镜像。

制作镜像的关键一步就是制作Dockerfile文件，在Dockerfile中通过Docker提供的一些命令编写一套执行流程来制作镜像。

```
# 使用官方提供的 Python 作为我们镜像的基础环境
FROM python:2.7-slim

# 将工作目录切换为 /Users/hupengfei/Downloads/docker/test
WORKDIR /Users/hupengfei/Downloads/docker/test

# 将当前目录下的所有内容复制到 /Users/hupengfei/Downloads/docker/test 下
ADD . /Users/hupengfei/Downloads/docker/test

# 使用 pip 命令安装这个应用所需要的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 允许外界访问容器的 80 端口
EXPOSE 80

# 设置环境变量
ENV NAME World

# 设置容器进程为：python app.py，即：这个 Python 应用的启动命令
CMD ["python", "app.py"]

```

下面介绍一些上面Dockerfile中用到的命令作用

* FROM：用于指定其后构建新镜像所使用的基础镜像。FROM 指令必是 Dockerfile 文件中的首条命令，启动构建流程后，Docker 将会基于该镜像构建新镜像，FROM 后的命令也会基于这个基础镜像。
* WORKDIR：用于在容器内设置一个工作目录。
* ADD：更高级的复制命令，将原路径文件赋值到目标路径中
* RUN：在容器中执行Shell命令，requirements.txt中放着所要下载依赖的名字Flask
* EXPOSE：为构建的镜像设置监听端口，使容器在运行时监听。
* ENV：设置环境变量而已，无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。
* CMD：用于指定在容器启动时所要执行的命令。

此时我们看到我们的目录中存放的文件为

```
hupengfei@localhost  ~/Downloads/docker  ls
Dockerfile       app.py           requirements.txt test

```

此时在存放Dockerfile目录下执行一下命令

```
docker build -t helloworld .

```

其中-t是为此镜像起一个名字，其中此命令会自动的加载本路径下的Dockerfile文件中内容，按顺序执行命令来创建镜像。此时可以通过以下命令查看已经创建的镜像

```
 hupengfei@localhost  ~/Downloads/docker  docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
helloworld              latest              c91cf8925eba        34 seconds ago      130MB

```

接下来通过`dokcer run`命令进行启动镜像

```
 hupengfei@localhost  ~/Downloads/docker  docker run -p 4000:80 helloworld
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)

```

其中`4000:80`意思是将宿主机的4000端口映射到镜像中的80端口上，此时我们用`docker ps `就可以看到启动的docker镜像了

```
 hupengfei@localhost  ~  docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED
1f1e5affd99e        helloworld          "python app.py"     23 seconds ago

```
此时访问页面也能够访问到

```
 hupengfei@localhost  ~  curl http://localhost:4000
<h3>Hello World!</h3><b>Hostname:</b> 1f1e5affd99e<br/>%

```

如果我们不指定绑定4000端口的话，那么就会在宿主机上随机分配一个端口与镜像进行连接

```
 hupengfei@localhost  ~/Downloads/docker  docker run -p :80 helloworld
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)

```

此时我们通过`docker port id号`能够查询到宿主机与镜像对应连接的端口号

```
 hupengfei@localhost  ~  docker port 1a93958d31b1
80/tcp -> 0.0.0.0:32769

```

## 上传镜像到DockerHub中

1. 在[Docker Hub](https://hub.docker.com/)上注册账号

2. 登录进去后在主页点击Create a Repository

	![](https://ws2.sinaimg.cn/large/006tKfTcly1g0ez7ukqztj314m0u0wiu.jpg)

3. 填写信息

	![](http://ws4.sinaimg.cn/large/006tKfTcly1g0ez9psvb7j30u00vin2d.jpg)

4. 在本机登录自己的Docker Hub 账号，命令是`docker login`

	![](http://ws4.sinaimg.cn/large/006tKfTcly1g0ezccdbbgj31f005igmz.jpg)

5. 用`docker tag`命令为自己的镜像取一个完整的名字包括版本号，v1为版本号

	```
	 docker tag helloworld 刚才创建的账户名字/helloworld:v1
	```

6. 将刚才所自定义的镜像上传上去，命令是`docker push buxuewushu/helloworld:v1`，其中`buxuewushu`是换成自己的账号，后面是刚才打的标签名

	![](https://ws2.sinaimg.cn/large/006tKfTcly1g0f09i6ezjj313s09m0v5.jpg)

7. 此时你就可以在[Docker Hub](https://hub.docker.com/)中看到自己上传的镜像了

	![](https://ws3.sinaimg.cn/large/006tKfTcly1g0f0a4xgszj31pq0i475y.jpg)

## 参考文章

本文的例子参考自极客时间中张磊写的深入剖析Kubernetes，有兴趣的可以查看

![](http://ws4.sinaimg.cn/large/006tKfTcly1g0f6kn6rsdj30u01hcdk2.jpg)


