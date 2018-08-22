---
layout: post
title: Deploy web in docker!
---

在centOS中使用docker部署简单的web服务

本机环境为物理机中开启虚拟机，虚拟机中已经配置好网络，可以正常访问外网。  
先在物理机中创建项目代码，推送到github，然后在虚拟机中拉取代码，创建镜像，启动容器。也可以直接在centOS中创建项目和必要的脚本。

1. 先快速生成一个简单web项目，本次使用EggJS框架。  
[如何快速搭建EggJS框架](https://eggjs.org/zh-cn/intro/quickstart.html)
 <br>
 在生成的项目根目录下创建Dockerfile文件(对，没有扩展名)  
  Dockerfile内容：
  <pre>
    <code>
      FROM mc2labs/nodejs
      LABEL Name=mytest
      # RUN npm config set registry=https://registry.npm.taobao.org/ \
            && npm install egg-scripts -g  \
            && npm install \
            && npm install -g typescript 
      EXPOSE 7001
      #CMD egg-scripts start --port 80 --daemon --title=APP_API
      CMD npm run dev
    </code>
  </pre>
[各种命令的含义点这里](https://blog.csdn.net/mozf881/article/details/55798811)  
以上就完成了Dockerfile脚本的工作。
本地测试项目可以正常运行之后推送到代码仓库。
2. 在centOS中创建镜像并启动容器  
从代码仓库拉取项目代码到虚拟机中，cd进入项目根目录执行命令：  
`docker build -t mynodeapp .`  
注意最后那个点  
上述命令会逐步执行Dockerfile中的每行指令  
类似下图  
![_config.yml]({{ site.baseurl }}/images/docker_build.png)
完成之后使用命令`docker images`查看所有存在镜像  
![_config.yml]({{ site.baseurl }}/images/docker_images.png)
此时镜像已经创建好了，就需要运行镜像  

执行命令`docker run -d -p 7001:7001 <镜像id>`  
然后查看镜像是否成功运行`docker ps`。  
如果不成功，那应该是通过`docker ps -a`才能看到，此时可以通过日志查看原因。  

执行`docker logs <运行失败的容器id>`，此处运行失败的容器id跟镜像id是不一样的，需要注意，容器id是执行`docker ps -a`返回的列表中状态为exited的容器对应的container id。  

如果成功，`docker ps`命令会显示正在运行中的容器信息  
![_config.yml]({{ site.baseurl }}/images/docker_ps.png)

测试一下是否可以访问运行中的web服务  
>curl 127.0.0.1:7001  
>hi, egg  

成功。
